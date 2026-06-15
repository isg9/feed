---
title: Async DNS Resolution
url: https://ziglang.org/devlog/2025/#2025-10-15
published: "2025-10-15T00:00:00Z"
feed: zig-devlog
guid: https://ziglang.org/devlog/2025/#2025-10-15
---

# Async DNS Resolution

October 15, 2025

# [Async DNS Resolution](\#2025-10-15)

Author: Andrew Kelley

Without libc in the picture, DNS resolution is a happy async utopia. We can blast out multiple queries and handle responses as they arrive, using standard system primitives such as file descriptors and `poll`.

Perturbingly, a pernicious pair of party poopers plot to puncture perfection:

1. The libc API for DNS resolution is `getaddrinfo`, which is ass.
2. Operating systems hide important details such as the configured nameservers behind libc, so you have to use it.

For example, if you bypass libc on a glibc system in order to get the good API, then you end up with people complaining that your application does not integrate properly with [NSS](https://sourceware.org/glibc/manual/).

But `getaddrinfo` is crap! It blocks until *all* the DNS queries return, so the best you can do is run it in a utility thread and then wait until it’s done. And even then there’s a footgun… it reads `environ` so if any code anywhere in the process calls `setenv` then you have corrupt memory access. This alone makes me not take POSIX seriously. If they want to regain my respect they have to fix this along with making `close` infallible. Rumor has it they made `close` fallible because they thought that was an appropriate place to rewind tape drives. 🤦

Anyway, thankfully some systems have non-standard alternatives that support asynchrony and cancellation:

- glibc has `getaddrinfo_a` / `gai_cancel`
- Windows has `GetAddrInfoExW` / `GetAddrInfoExCancel`
- OpenBSD has `getaddrinfo_async` / `asr_abort`
- macOS has `CFHostStartInfoResolution` / `CFHostCancelInfoResolution`
- FreeBSD has `dnsres_getaddrinfo`, but no cancellation support

And that’s it. If your OS is not in this list, your DNS resolution API is crap! Fix it!

I didn’t take advantage any of those APIs yet, but I did implement the case without pesky libc. I want to share that code here because it’s a pretty nifty example of the new `std.Io` interface in action:

```zig
pub const ConnectError = LookupError || IpAddress.ConnectError;

pub fn connect(
    host_name: HostName,
    io: Io,
    port: u16,
    options: IpAddress.ConnectOptions,
) ConnectError!Stream {
    var connect_many_buffer: [32]ConnectManyResult = undefined;
    var connect_many_queue: Io.Queue(ConnectManyResult) = .init(&connect_many_buffer);

    var connect_many = io.async(connectMany, .{ host_name, io, port, &connect_many_queue, options });
    var saw_end = false;
    defer {
        connect_many.cancel(io);
        if (!saw_end) while (true) switch (connect_many_queue.getOneUncancelable(io)) {
            .connection => |loser| if (loser) |s| s.close(io) else |_| continue,
            .end => break,
        };
    }

    var aggregate_error: ConnectError = error.UnknownHostName;

    while (connect_many_queue.getOne(io)) |result| switch (result) {
        .connection => |connection| if (connection) |stream| return stream else |err| switch (err) {
            error.SystemResources,
            error.OptionUnsupported,
            error.ProcessFdQuotaExceeded,
            error.SystemFdQuotaExceeded,
            error.Canceled,
            => |e| return e,

            error.WouldBlock => return error.Unexpected,

            else => |e| aggregate_error = e,
        },
        .end => |end| {
            saw_end = true;
            try end;
            return aggregate_error;
        },
    } else |err| switch (err) {
        error.Canceled => |e| return e,
    }
}

pub const ConnectManyResult = union(enum) {
    connection: IpAddress.ConnectError!Stream,
    end: ConnectError!void,
};

/// Asynchronously establishes a connection to all IP addresses associated with
/// a host name, adding them to a results queue upon completion.
pub fn connectMany(
    host_name: HostName,
    io: Io,
    port: u16,
    results: *Io.Queue(ConnectManyResult),
    options: IpAddress.ConnectOptions,
) void {
    var canonical_name_buffer: [max_len]u8 = undefined;
    var lookup_buffer: [32]HostName.LookupResult = undefined;
    var lookup_queue: Io.Queue(LookupResult) = .init(&lookup_buffer);
    var group: Io.Group = .init;

    group.async(io, lookup, .{ host_name, io, &lookup_queue, .{
        .port = port,
        .canonical_name_buffer = &canonical_name_buffer,
    } });

    while (lookup_queue.getOne(io)) |dns_result| switch (dns_result) {
        .address => |address| group.async(io, enqueueConnection, .{ address, io, results, options }),
        .canonical_name => continue,
        .end => |lookup_result| {
            results.putOneUncancelable(io, .{
                .end = if (group.wait(io)) lookup_result else |err| err,
            });
            return;
        },
    } else |err| switch (err) {
        error.Canceled => |e| {
            group.cancel(io);
            results.putOneUncancelable(io, .{ .end = e });
        },
    }
}

```

This code has the following properties:

- It asynchronously sends out DNS queries to each configured nameserver
- As each response comes in, it immediately, asynchronously tries to TCP connect to the returned IP address.
- Upon the first successful TCP connection, all other in-flight connection attempts are canceled, including DNS queries.
- Regardless of whether `Io` is implemented via threads, or via an event loop, this code behaves optimally.
- The code also works when using single-threaded, blocking `Io` even though the operations happen sequentially.
- No heap allocation.

I’m pleased that, on top of all this, it reads like standard, idiomatic Zig code. We still use all the reguar control flow: `try`, `defer`, `while`, etc.

The code is flat, not much nesting. And just like non-async code, we can insert `try foo` almost anywhere and rest assured that the same resource management patterns have the error handling taken care of.

So hopefully that gives you a taste of what these changes bring to the table:

[std: Introduce Io Interface](https://github.com/ziglang/zig/pull/25592)
