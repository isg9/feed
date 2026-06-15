---
title: Pfmatch, a packet filtering language embedded in Lua — wingolog
url: https://wingolog.org/archives/2015/07/03/pfmatch-a-packet-filtering-language-embedded-in-lua
published: "2015-07-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2015/07/03/pfmatch-a-packet-filtering-language-embedded-in-lua
---

# Pfmatch, a packet filtering language embedded in Lua — wingolog

## [Pfmatch, a packet filtering language embedded in Lua](/archives/2015/07/03/pfmatch-a-packet-filtering-language-embedded-in-lua)

3 July 2015 11:05 AM

- [lua](/tags/lua)
- [pflua](/tags/pflua)
- [bpf](/tags/bpf)
- [pflang](/tags/pflang)
- [igalia](/tags/igalia)
- [snabb](/tags/snabb)
- [compilers](/tags/compilers)
- [dsl](/tags/dsl)
- [edsl](/tags/edsl)

Greets, hackers! I just finished implementing a little embedded language in Lua and wanted to share it with you. First, a bit about the language, then some notes on how it works with Lua to reach the high performance targets of [Snabb Switch](https://github.com/SnabbCo/snabbswitch).

**the pfmatch language**

[Pfmatch](https://github.com/Igalia/pflua/blob/master/doc/pfmatch.md) is a language designed for filtering, classifying, and dispatching network packets in Lua. Pfmatch is built on the well-known [pflang](https://github.com/Igalia/pflua/blob/master/doc/pflang.md) packet filtering language, using the fast [pflua](https://github.com/Igalia/pflua/blob/master/README.md) compiler for LuaJIT.

Here's an example of a simple pfmatch program that just divides up packets depending on whether they are TCP, UDP, or something else:

```
match {
   tcp => handle_tcp
   udp => handle_udp
   otherwise => handle_other
}

```

Unlike pflang filters written for such tools as `tcpdump`, a pfmatch program can dispatch packets to multiple handlers, potentially destructuring them along the way. In contrast, a pflang filter can only say "yes" or "no" on a packet.

Here's a more complicated example that passes all non-IP traffic, drops all IP traffic that is not going to or coming from certain IP addresses, and calls a handler on the rest of the traffic.

```
match {
   not ip => forward
   ip src 1.2.3.4 => incoming_ip
   ip dst 5.6.7.8 => outgoing_ip
   otherwise => drop
}

```

In the example above, the handlers after the arrows ( `forward`, `incoming_ip`, `outgoing_ip`, and `drop`) are Lua functions. The part before the arrow ( `not ip` and so on) is a pflang expression. If the pflang expression matches, its handler will be called with two arguments: the packet data and the length. For example, if the `not ip` pflang expression is true on the packet, the `forward` handler will be called.

It's also possible for the handler of an expression to be a sub-match:

```
match {
   not ip => forward
   ip src 1.2.3.4 => {
      tcp => incoming_tcp(&ip[0], &tcp[0])
      udp => incoming_udp(&ip[0], &ucp[0])
      otherwise => incoming_ip(&ip[0])
   }
   ip dst 5.6.7.8 => {
      tcp => outgoing_tcp(&ip[0], &tcp[0])
      udp => outgoing_udp(&ip[0], &ucp[0])
      otherwise => outgoing_ip(&ip[0])
   }
   otherwise => drop
}

```

As you can see, the handlers can also have additional arguments, beyond the implicit packet data and length. In the above example, if `not ip` doesn't match, then `ip src 1.2.3.4` matches, then `tcp` matches, then the `incoming_tcp` function will be called with four arguments: the packet data as a `uint8_t*` pointer, its length in bytes, the offset of byte 0 of the IP header, and the offset of byte 0 of the TCP header. An argument to a handler can be any arithmetic expression of pflang; in this case `&ip[0]` is actually an [extension](https://github.com/Igalia/pflua/blob/master/doc/extensions.md). More on that later. For language lawyers, check the [syntax and semantics](https://github.com/Igalia/pflua/blob/master/doc/pfmatch.md#syntax) over in our source repo.

Thanks especially to my colleague Katerina Barone-Adesi for long backs and forths about the language design; they really made it better. Fistbump!

**pfmatch and lua**

The challenge of designing pfmatch is to gain expressiveness, compared to writing filters by hand, while not endangering the performance targets of Pflua and Snabb Switch. These days [Snabb is on target to give ASIC-driven network appliances a run for their money](https://groups.google.com/d/msg/snabb-devel/_vKQgCU29_Q/ngyMU2cM6qsJ), so anything we come up with cannot sacrifice speed.

In practice what this means is *compile, don't interpret*. Using the pflua compiler allows us to generalize the good performance that we have gotten on pflang expressions to a multiple-dispatch scenario. It's a pretty straightword strategy. Naturally though, the interface with Lua is more complex now, so to understand the performance we should understand the interaction with Lua.

How does one make two languages interoperate, anyway? With pflang it's pretty clear: you compile pflang to a Lua function, and call the Lua function to match on packets. It returns true or false. It's a thin interface. Indeed with pflang and pflua you could just match the clauses in order:

```
not_ip = pf.compile('not ip')
incoming = pf.compile('ip src 1.2.3.4')
outgoing = pf.compile('ip dst 5.6.7.8')

function handle(packet, len)
   if not_ip(packet, len) then return forward(packet, len)
   elseif incoming(packet, len) then return incoming_ip(packet, len)
   elseif outgoing(packet, len) then return outgoing_ip(packet, len)
   else return drop(packet, len) end
end

```

But not only is this tedious, you don't get easy access to the packet itself, and you're missing out on opportunities for optimization. For example, if the packet fails the `not_ip` check, we don't need to check if it's an IP packet in the `incoming` check. Compiling a pfmatch program takes advantage of pflua's optimizer to produce good code for the match expression as a whole.

If this were Scheme I would make the right-hand side of an arrow be an expression and implement pfmatch as a macro; see [Racket's `match` documentation](http://docs.racket-lang.org/reference/match.html) for an example. In Lua or other languages that's harder to do; you would have to parse Lua, and it's not clear which parts of the production as a whole are the host language (Lua) and which are the embedded language (pfmatch).

Instead, I think embedding host language snippets by function name is a fine solution. It seems fairly clear that `incoming_ip`, for example, is some kind of function. It's easy to parse identifiers in an embedded language, both for humans and for programs, so that takes away a lot of implementation headache and cognitive overhead.

We are left with a few problems: how to map names to functions, what to do about the return value of match expressions, and how to tie it all together in the host language. Again, if this were Scheme then I'd use macros to embed expressions into the pfmatch term, and their names would be scoped into whatever environment the match term was defined. In Lua, the best way to implement a name/value mapping is with a table. So we have:

```
local handlers = {
   forward = function(data, len)
      ...
   end,
   drop = function(data, len)
      ...
   end,
   incoming_ip = function(data, len)
      ...
   end,
   outgoing_ip = function(data, len)
      ...
   end
}

```

Then we will pass the handlers table to the matcher function, and the matcher function will call the handlers by name. LuaJIT will mostly take care of the overhead of the table dispatch. We compile the filter like this:

```
local match = require('pf.match')

local dispatcher = match.compile([[match {
   not ip => forward
   ip src 1.2.3.4 => incoming_ip
   ip dst 5.6.7.8 => outgoing_ip
   otherwise => drop
}]])

```

To use it, you just invoke the dispatcher with the handlers, data, and length, and the return value is whatever the handler returns. Here let's assume it's a boolean.

```
function loop(self)
   local i, o = self.input.input, self.output.output
   while not link.empty() do
      local pkt = link.receive(i)
      if dispatcher(handlers, pkt.data, pkt.length) then
         link.transmit(o, pkt)
      end
   end
end

```

Finally, we're ready for an example of a compiled matcher function. Here's what pflua does with the match expression above:

```
local cast = require("ffi").cast
return function(self,P,length)
   if length < 14 then return self.forward(P, len) end
   if cast("uint16_t*", P+12)[0] ~= 8 then return self.forward(P, len) end
   if length < 34 then return self.drop(P, len) end
   if P[23] ~= 6 then return self.drop(P, len) end
   if cast("uint32_t*", P+26)[0] == 67305985 then return self.incoming_ip(P, len) end
   if cast("uint32_t*", P+30)[0] == 134678021 then return self.outgoing_ip(P, len) end
   return self.drop(P, len)
end

```

The result is a pretty good dispatcher. There are always things to improve, but it's likely that the function above is better than what you would write by hand, and it will continue to get better as pflua improves.

Getting back to what I mentioned earlier, when we write filtering code by hand, we inevitably end up writing *interpreters* for some kind of filtering language. Network functions are essentially linguistic in nature: static appliances are no good because network topologies change, and people want solutions that reflect their problems. Usually this means embedding an interpreter for some embedded language, for example BPF bytecode or iptables rules. Using pflua and pfmatch expressions, we can instead *compile* a filter suited directly for the problem at hand -- and while we're at it, we can forget about worrying about pesky offsets, constants, and bit-shifts.

**challenges**

I'm optimistic about pfmatch or something like it being a success, but there are some challenges too.

One challenge is that pflang is pretty weird. For example, [attempting to access ip\[100\] will abort a filter immediately on a packet that is less than 100 bytes long](https://github.com/Igalia/pflua/blob/master/doc/pflang.md#packet-access), not including L2 encapsulation. It's wonky semantics, and in the context of pfmatch, aborting the entire pfmatch program would obviously be the wrong thing. That would abort too much. Instead it should probably just fail the pflang test in which that packet access appears. To this end, in pfmatch we turn those aborts into local expression match failures. However, this leads to an inconsistency with pflang. For example in `(ip[100000] == 0 or (1==1))`, instead of `ip[100000]` causing the whole pflang match to fail, it just causes the local test to fail. This leaves us with `1==1`, which passes. We abort too little.

This inconsistency is probably a bug. We want people to be able to test clauses with vanilla pflang expressions, and have the result match the pfmatch behavior. Due to limitations in some of pflua's intermediate languages, it's likely to persist for a while. It is the only inconsistency that I know of, though.

Pflang is also underpowered in many ways. It has terrible IPv6 support; for example, `tcp[0]` only matches IPv4 packets, and at least as implemented in libpcap, most [payload access on IPv6 packets does the wrong thing regarding chained extension headers](https://github.com/Igalia/pflua/issues/239). There is no facility in the language for binding names to intermediate results, there is no linguistic facility for talking about fragmentation, no ability to address IP source and destination addresses in arithmetic expressions by name, and so on. We can solve these in pflua with [extensions](https://github.com/Igalia/pflua/blob/master/doc/extensions.md) to the language, but that introduces incompatibilities with pflang.

You might wonder why to stick with pflang, after all of this. If this is you, Juho Snellman wrote a great article on this topic, just for you: [What's wrong with pcap filters](https://www.snellman.net/blog/archive/2015-05-18-whats-wrong-with-pcap-filters/).

Pflua's optimizer has mostly helped us, but there have been places where it could be more helpful. When compiling just one expression, you can often end up figuring out which branches are dead-ends, which helps the rest of the optimization to proceed. With more than one successful branch, we had to make a few improvements to the optimizer to actually get decent results. We also had to relax one restriction on the optimizer: usually we only permit transformations that make the code smaller. This way we know we're going in the right direction and will eventually terminate. However because of reasons™ we did decide to allow tail calls to be duplicated, so instead of having just one place in the match function that tail-calls a handler, you can end up with multiple calls. I suspect using a tracing compiler will largely make this moot, as control-flow splits effectively lead to trace duplication anyway, and making sure control-flow joins later doesn't effectively counter that. Still, I suspect that the resulting trace shape will rejoin only at the loop head, instead of in some intermediate point, which is probably OK.

**future**

With all of these concerns, is pfmatch still a win? Yes, probably! We're going to start using it when [building Snabb apps](https://github.com/Igalia/pflua/blob/master/doc/pfmatch.md#using-pfmatch), and will see how it goes. We'll probably end up adding a few more pflang extensions before we're done. If it's something you're in to, [`snabb-devel`](https://groups.google.com/forum/#!forum/snabb-devel) is the place to try it out, and see you on the [bug tracker](https://github.com/Igalia/pflua/issues). Happy packet hacking!

## related articles

- [high-performance packet filtering with pflua](/archives/2014/09/02/high-performance-packet-filtering-with-pflua)
- [lightweight concurrency in lua](/archives/2018/05/16/lightweight-concurrency-in-lua)
- [an early look at p4 for software networking](/archives/2017/06/26/an-early-look-at-p4-for-software-networking)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [six thoughts on generating c](/archives/2026/02/09/six-thoughts-on-generating-c)

Comments are closed.
