---
title: What is io_uring?
url: https://matklad.github.io/2024/09/23/what-is-io-uring.html
published: "2024-09-23T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2024/09/23/what-is-io-uring.html
---

# What is io_uring?

Sep 23, 2024

An attempt at concise explanation of what io\_uring is.

`io_uring` is a new Linux kernel interface for making system calls. Traditionally, syscalls are submitted to the kernel **individually** and **synchronously**: a syscall CPU instruction transfers control from the application to the kernel; control returns to the application only when the syscall is completed. In contrast, `io_uring` is a **batched** and **asynchronous** interface. The application submits several syscalls by writing their codes & arguments to a lock-free shared-memory ring buffer. The kernel reads the syscalls from this shared memory and executes them at its own pace. To communicate results back to the application, the kernel writes the results to a second lock-free shared-memory ring buffer, where they become available to the application asynchronously.

You might want to use `io_uring` if:

- you need extra performance unlocked by amortizing userspace/kernelspace context switching across entire batches of syscalls,

- you want a unified asynchronous interface to the entire system.

You might want to avoid `io_uring` if:

- you need to write portable software,

- you want to use only old, proven features,

- and in particular you want to use features with a good security track record.
