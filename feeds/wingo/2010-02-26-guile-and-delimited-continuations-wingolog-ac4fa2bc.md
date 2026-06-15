---
title: guile and delimited continuations — wingolog
url: https://wingolog.org/archives/2010/02/26/guile-and-delimited-continuations
published: "2010-02-26T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2010/02/26/guile-and-delimited-continuations
---

# guile and delimited continuations — wingolog

## [guile and delimited continuations](/archives/2010/02/26/guile-and-delimited-continuations)

26 February 2010 8:39 PM

- [guile](/tags/guile)
- [continuations](/tags/continuations)
- [scheme](/tags/scheme)
- [delimited continuations](/tags/delimited%20continuations)

Guile now does delimited continuations.

Ahem! I say: *Guile now does delimited continuations!* Whoop whoop!

Practically speaking, this means that Guile implements `prompt` and `abort`, as described in Dorai Sitaram's 1993 paper, [Handling Control](http://www.ccs.neu.edu/scheme/pubs/pldi93-sitaram.pdf). (Guile's `abort` is the operator that Sitaram calls `control`.)

I used to joke that all of this Guile compilation work was to bring Guile back into the 2000s, from being stuck in the 1990s. But delimited continuations are definitely a twenty-teens topic: besides PLT Scheme, I don't know of any other production language implementation that has delimited continuations as part of its core language. (Please correct my ignorance.) If this works out, and the implementation proves to be sufficiently bug-free, this is a great step forward not only for Guile but for the concept itself of delimited continuations.

*Edit: Readers suggest that I add Scheme48, OCaml, and Scala to the list. Still, too few implementations for such a lovely construct.*

Furthermore, I retrofitted Guile's existing `catch` and `throw` APIs, implementing them in terms of `prompt` and `control`, and providing compatible shims for Guile's C API.

[Thanks again](//wingolog.org/archives/2010/02/14/sidelong-glimpses) to [Matthew Flatt](http://www.cs.utah.edu/~mflatt/) for blazing the trail here.

**rambling**

So, for the benefit of those dozen or so people that will probably implement delimited continuations after me, here's a few strategies and pitfalls. For the rest of you, I recommend [rancid wine](//wingolog.org/archives/2010/02/26/lunches-and-lunches).

First, I should make a disclaimer. Here, my focus is on a so-called "direct" implementation of delimited continuations; that is to say, I don't rely on a global continuation-passing style (CPS) transform of the source code.

There was recently an article posted to the [Scheme reddit](http://scheme.reddit.com) about [Femtolisp](http://code.google.com/p/femtolisp/), a Lisp implementation by Jeff Benzason. I thought it was pretty neat. I was especially pleased that he decided to support `shift` and `reset`; but bummed when I found out that he did so by local CPS-transformation. So you couldn't reset from arbitrary functions.

A local CPS-rewrite strategy doesn't work very well, because prompts are fundamentally dynamic. There is something fundamentally dynamic about them, that they are part of the dynamic environment (like dynamic-wind). So to support that via CPS, you end up needing some kind of [double- or triple-barrelled CPS](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.7.9526&rep=rep1&type=pdf), with abort continuations as well. I think? I think.

So for me, direct implementation means that you use the language's native stack representation, not requiring global or local transformations.

Also I should confess that I didn't glean anything from Gasbichler and Sperber's [Final Shift](http://citeseer.ist.psu.edu/gasbichler02final.html) paper; in all likelihood, again due to my ignorance. So if I say things that they've said better, well, stories have to be retold to live, and it won't hurt if I add my interpretation.

**the seitan of the matter**

So, *Digresso [Riddikulus](http://en.wikipedia.org/wiki/Boggart_(Harry_Potter)#Boggarts)*: poof! Where were we? And what's up with this jack-in-the-box? Yes, direct implementations of delimited continuations.

Here's a C program:

```
int baz ();
int bar () { return baz (); }
int foo () { return bar (); }
int main () { return foo (); }

```

And here's a C stack:

![](//wingolog.org/pub/delimc-0.png)

Right! So you see that function calls are pushing frames on the stack.

When you're programming normally, you have a top-down view of the stack. It's like standing on a ladder and looking down -- you see the top rung clearly, but the rest is obscured by your toolbelt.

Laying out the frames flat like this allows us to talk instead about the whole future of this program -- what it's doing, and what it's going to do. The future of the program is known as its *continuation* \-\- a funny word, but I hope the meaning is clear in this context.

Now, consider a C program that calls into some hypothetical Scheme, assuming appropriate Scheme definitions of `foo`, `bar`, and `baz`:

```
int main () { return eval ("(foo)"); }

```

And the corresponding pixels:

![](//wingolog.org/pub/delimc-1.png)

It should be clear that there are logically two stacks at work here. However they both correspond, in this case, to the same logical control-flow stack -- `eval` doesn't return until `foo`, `bar`, and `baz` have returned.

Also note that the "C" stack has been renamed the "foreign" stack, indicating that Scheme is actually where you want to be, and whatever is not in the Scheme world is heathen. This model maintains its truthiness regardless of the implementation strategy of the Scheme -- whether it reuses the C stack, or evaluates Scheme expressions on their own stack.

So! Delimited continuations, right? Let's try one. Here's some Scheme code that begins a prompt, and calls `baz`.

```
(% (baz) (lambda (k) k))

```

If `baz` returns normally, well that's the value; but if it aborts, the partial continuation will be saved, and returned. Verily, pixels:

![](//wingolog.org/pub/delimc-2.png)

The red dot and line indicates that that position in the continuation has been saved, and if we abort, we'll abort back to that line. It's like grabbing a cherry from the tree, and then falling down a couple of rungs on the ladder. Yes? Yes.

Expanding the example with an implementation of `baz`, we have:

```
(define (baz) (abort) 3)

```

So, `baz` will abort to the nearest prompt, and if the abort comes back, it will return 3. Like this:

![](//wingolog.org/pub/delimc-3.png)

Abort itself needs to capture a partial continuation -- that part of the continuation that is delimited by `prompt` and `abort`. (That's why they're called delimited continuations.)

In practice, for the implementor, this has a number of fine points. The one I'll mention here is that you don't actually capture the `prompt` frame itself, or the presence of the prompt on the dynamic stack. I tried to indicate that in my drawings by making the red line disjoint from the red box. The red box is what's captured by the continuation.

See Dybvig's "A monadic framework for delimited continuations" for a more complete discussion of what to capture.

```
(define (qux)
  (+ (k) 38))

```

OK! Assume that we save away that partial continuation somewhere, let's say, in `k`. Now the evaluator ends up calling `qux`, which calls the partial continuation.

![](//wingolog.org/pub/delimc-4.png)

Calling the partial continuation in this context means splatting it on the stack, and then fixing things up so that it looks like we actually called `baz` from `qux`.

I'll get to the precise set of things you have to fix up, but let's pause a moment with these pixels, and make one thing clear: you can't splat foreign frames back on the stack. Maybe you can in some circumstances, but not in the general case.

The practical implication of this is twofold: one, delimited continuation support needs to be a core part of your language. You can't evaluate the "body" part of the prompt by recursing through a foreign function, for example.

The second (and deeper) implication is that if any intervening code does recurse through a foreign function, the resulting partial continuation cannot be reinstated.

One could simply error in this case, but it's more useful to allow nonlocal exits, and then error on an attempted re-entry.

So yes, things you need to fix up: obviously the frame pointer stack. The dynamic winds (and more generally, the dynamic environment). And, any captured prompts themselves. Consider:

![](//wingolog.org/pub/delimc-5.png)

Here we see a prompt, which has a prompt inside it, which calls `baz`, which then aborts back to the first prompt. (You will need prompt tags for this.) Now, note: there are two horizontal lines here: two prompts active in the dynamic environment.

Let's see what happens when we reinstate the continuation, represented by the red dotted box:

![](//wingolog.org/pub/delimc-6.png)

Here we see that the blue line corresponding to the *captured* prompt is in a different place (relative to the base of the stack). That's because when you reinstate a prompt, any captured prompts are logically made new: one must recapture their registers, both real (via setjmp) and virtual (in the case of a virtual machine) when rewinding the stack.

And that's all!

**postre**

Meanwhile, back at the ranch: while searching for the correct spelling of *riddikulus*, I stumbled upon a delightful interview with JK Rowling, [in](http://www.mugglenet.com/jkrinterview.shtml) [three](http://www.mugglenet.com/jkrinterview2.shtml) [parts](http://www.mugglenet.com/jkrinterview3.shtml). Enjoy!

## related articles

- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [stack overflow](/archives/2014/03/17/stack-overflow)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [scheme quiz time!](/archives/2013/11/02/scheme-quiz-time)

### 5 responses

1. Sam TH says:[26 February 2010 8:48 PM](#c8a57130fb9552884f83879d0a06b3fd8c01fa3f)

   I think both OCaml and Scheme48 currently have delimited continuations.

2. [matthias felleisen](http://goglemyfirstname.com) says:[26 February 2010 9:03 PM](#712ae992ec1e34c2687ad574a6dbaeae303e0de4)

   I suggest googling 'delimited continuations'. You will find that even Wikipedia

   has a badly written article on the idea.

   You will also find on that first page that

   \-\- Scheme 48

   \-\- OCaml

   \-\- Scala

   come with these things.

   When I put Dorai to work on this idea, he came up with the novel idea

   of implementing it all via 'macros' (with one small caveat) and so we added

   it all to Chez Scheme and taught prompt-ed continuations in this setting

   starting in 1989.

   (My first prototype was implemented in Scheme 84 in the fall of 1984.

   We used it at IU for a couple of years to run Dan Friedman's course on

   Coordinate computing but the engine's based implementation (threads)

   was way too slow.)

3. [Andy Wingo](http://wingolog.org/) says:[26 February 2010 9:29 PM](#489794e3a3d155129110e33e69e257b087adeaeb)

   Thanks for stopping by again Sam, and I'm humbled to have your visit, Matthias. Cheers.

   Regarding Scheme48, Flatt 07 (which Matthias co-authored) seemed to indicate that delimited continuations were somehow not integrated -- due to interactions with dynamic-wind, perhaps. I left it off the list for that reason, though perhaps things have changed.

   I had seen something about OCaml, a paper by Oleg that suggested that it operated on the bytecode itself; and a blog post about delimited continuations maybe being added to Scala. Nevertheless I'll edit the post and add them.

   I think the thing is not simply to have implemented delimited continuations in a Scheme, it's to have integrated them well -- as the fine PLT folk have done. Guile is not there yet, but it's making progress.

4. Tristan Colgate says:[2 March 2010 9:25 AM](#fc017abef201532758affc1b579afbd620fa60e9)

   I get an error with guile master if the call to abort is to be used as a value passed to something else (it works at the end of a begin)...

   (define cont (% (begin (newline)(+ 2 (abort (fluid-ref %default-prompt-tag)))) (lambda(k)k)))

   Throw to key \`misc-error':

   ERROR: unknown binding type val

   This works though...

   (define cont (% (begin (newline)(abort (fluid-ref %default-prompt-tag))) (lambda(k)k)))

   scheme@(guile-user)> (cont)

   scheme@(guile-user)> (cont 3)

   $4 = 3

5. [Andy Wingo](http://wingolog.org/) says:[3 March 2010 11:18 AM](#2b72e18779fe0d45dccab0bea72c295023506c19)

   @Tristan: Interesting, thanks for the report!

   (The reason why the test suite doesn't pick it up is that the test suite gets run through the evaluator and not the compiler, so some compiler cases aren't exercised.)

Comments are closed.
