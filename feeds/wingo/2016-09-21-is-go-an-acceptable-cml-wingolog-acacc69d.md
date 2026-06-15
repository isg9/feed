---
title: is go an acceptable cml? — wingolog
url: https://wingolog.org/archives/2016/09/21/is-go-an-acceptable-cml
published: "2016-09-21T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2016/09/21/is-go-an-acceptable-cml
---

# is go an acceptable cml? — wingolog

## [is go an acceptable cml?](/archives/2016/09/21/is-go-an-acceptable-cml)

21 September 2016 9:29 PM

- [go](/tags/go)
- [concurrency](/tags/concurrency)
- [ml](/tags/ml)
- [ignorance](/tags/ignorance)
- [guile](/tags/guile)
- [clojure](/tags/clojure)
- [core.async](/tags/core.async)

Yesterday I tried to summarize the things I know about Concurrent ML, and I came to the [tentative conclusion](https://wingolog.org/archives/2016/09/20/concurrent-ml-versus-go) that Go (and any Go-like system) was an acceptable CML. Turns out I was both wrong and right.

**you were wrong when you said everything's gonna be all right**

I was wrong, in the sense that programming against the CML abstractions lets you do more things than programming against channels-and-goroutines. Thanks to Sam Tobin-Hochstadt to pointing this out. As an example, consider a little process that tries to receive a message off a channel, and times out otherwise:

```
func withTimeout(ch chan int, timeout int) (result int) {
  var timeoutChannel chan int;
  var msg int;
  go func() {
    sleep(timeout)
    timeoutChannel <- 0
  }()
  select {
    case msg = <-ch: return msg;
    case msg = <-timeoutChannel: return 0;
  }
}

```

I think that's the first Go I've ever written. I don't even know if it's syntactically valid. Anyway, I think we see how it should work. We return the message from the channel, unless the timeout happens before.

But, what if the message is itself a composite message somehow? For example, say we have a transformer that reads a value from a channel and adds 1 to it:

```
func onePlus(in chan int) (result chan int) {
  var out chan int
  go func () { out <- 1 + <-in }()
  return out
}

```

What if we do a `withTimeout(onePlus(numbers), 0)`? Assume the timeout fires first and that's the result that `select` chooses. There's still that `onePlus` goroutine out there trying to read from `in` and at some point probably it will succeed, but nobody will read its value. At that point the number just vanishes into the ether. Maybe that's OK in certain domains, but certainly not in general!

What CML gives you is the ability to express an *event* (which is kinda like a possibility of sending or receiving a message on a channel) in such a way that we don't run into this situation. Specifically with the `wrap` combinator, we would make an event such that receiving on `numbers` would run a function on the received message and return that as the message value -- which is of course the same as what we have, except that in CML the `select` wouldn't actually read the message off unless it `select`'d that channel for input.

Of course in Go you could just rewrite your program, so that the `select` statement looks like this:

```
select {
  case msg = <-ch: return msg + 1;
  case msg = <-timeoutChannel: return 0;
}

```

But here we're operating at a lower level of abstraction; we were forced to intertwingle our concerns of adding 1 and our concerns of timeout. CML is more expressive than Go.

**you were right when you said we're all just bricks in the wall**

However! I was right in the sense that you can build a CML system on top of Go-like systems (though possibly not Go in particular). Thanks to Vesa Karvonen for [this comment](http://wingolog.org/archives/2016/09/20/concurrent-ml-versus-go#2a65df52fda7ed2cd2f31a55ff530becfc21d819) and the link to their [proof-of-concept CML implementation in Clojure's `core.async`](https://github.com/polytypic/poc.cml). I understand Vesa also has an implementation in F# as well.

Folks should read Vesa's code, after reading the Reppy papers of course; it's delightfully short and expressive. The basic idea is that event composition operators like `choose` and `wrap` build up data structures instead of doing things. The `sync` operation then grovels through those data structures to collect a list of channels to pass on to `core.async`'s equivalent of `select`. When `select` returns, `sync` determines which event that chosen channel and message corresponds to, and proceeds to "activate" the event (and, as a side effect, possibly issue NACK messages to other channels).

Provided you can map from the chosen `select` channel/message back to the event, (something that `core.async` can mostly do, with a caveat; see the code), then you can build CML on top of channels and goroutines.

**o/~ yeah you were wrong o/~**

On the other hand! One advantage of CML is that its events are not limited to channel sends and receives. I understand that timeouts, thread joins, and maybe some other event types are first-class event kinds in many CML systems. Michael Sperber, current Scheme48 maintainer and functional programmer, tells me that simply wrapping events in channels+goroutines works but can incur a big performance overhead relative to supporting those event types natively, due to the need to make the new goroutine and channel and the scheduling costs. He quotes 10X as the overhead!

So although CML and Go appear to be inter-expressible, maybe a proper solution will base the simple channel send/receive interface on CML rather than the other way around.

Also, since these events are now second-class, it must be OK to lose these events, for the same reason that the naïve `withTimeout` could lose a message from `numbers`. This is the case for timeouts usually but maybe you have to think about this more, and possibly provide an infinite stream of the message. (Of course the wrapper goroutine would be collected if the channel becomes unreachable.)

**you were right when you said this is the end**

I've long wondered how contemporary musicians deal with the enormous, crushing weight of recorded music. I don't really pick any more but hoo am I feeling this now. I think for Guile, I will continue hacking on fibers in a separate library, and I think that things will remain that way for the next couple years and possibly more. We need more experience and more mistakes before blessing and supporting any particular formulation of highly concurrent programming. I will say though that I am delighted that we are able to actually do this experimentation on a library level and I look forward to seeing what works out :)

Thanks again to Vesa, Michael, and Sam for sharing their time and knowledge; all errors are of course mine. Happy hacking!

## related articles

- [concurrent ml versus go](/archives/2016/09/20/concurrent-ml-versus-go)
- [tree-shaking, the horticulturally misguided algorithm](/archives/2023/11/24/tree-shaking-the-horticulturally-misguided-algorithm)
- [An incomplete history of language facilities for concurrency](/archives/2016/10/12/an-incomplete-history-of-language-facilities-for-concurrency)
- [guile lab notebook: on the move!](/archives/2025/07/08/guile-lab-notebook-on-the-move)
- [on taking advantage of ragged stops](/archives/2024/09/06/on-taking-advantage-of-ragged-stops)
- [coarse or lazy?](/archives/2022/08/07/coarse-or-lazy)

### One response

1. Brandon says:[8 October 2016 2:54 AM](#5a9a4cf5d2c303872dd2a9d47a19159297bdce64)

   You seem to have some spam in the comments on this article (and the last one).

Comments are closed.
