---
title: V8 is faster than GCC — wingolog
url: https://wingolog.org/archives/2011/06/10/v8-is-faster-than-gcc
published: "2011-06-10T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/06/10/v8-is-faster-than-gcc
---

# V8 is faster than GCC — wingolog

## [V8 is faster than GCC](/archives/2011/06/10/v8-is-faster-than-gcc)

10 June 2011 3:57 PM

- [v8](/tags/v8)
- [gcc](/tags/gcc)
- [compilers](/tags/compilers)
- [inlining](/tags/inlining)
- [guile](/tags/guile)

Do you like the linkbait title? Neither do I, but it is true in this case. Check it out.

After my [last post](//wingolog.org/archives/2011/06/08/what-does-v8-do-with-that-loop), [Benjamin](http://blogs.gnome.org/otte/) noted that GCC would reduce my simple test to a `mov rax, $10000000; ret` sequence. Well yes, that's true, and GCC does do that: but only if GCC is able and allowed to do the inlining. So the equivalent of the test, for GCC, is to compile the `g` in a separate file:

```
unsigned int g (void) { return 1; }

```

Then in the main file, we have:

```
extern unsigned int g (void);

int main (int argc, char *argv) {
  unsigned int i, ret = 0;
  for (i = 0; i < NU; i++)
    ret += g ();
  return ret;
}

```

Here we replace *N* with the number of iterations we're interested in testing.

I decided to run such a test; the harness is [here](//wingolog.org/pub/v8-vs-gcc.scm). I ran both GCC and V8 for the above program, for *N* from 20 to 231, and plotted the *total* time taken by V8 as compared to GCC.

Naturally this includes the GCC compile times, for which we expect GCC to lose at low iterations, but the question is, when does GCC start being faster than V8?

**answer: never!**

![](//wingolog.org/pub/v8-vs-gcc.png)

That's right, V8 is always faster than GCC, right up to the point at which its fixnums start failing. For the record, only the points on the right of the graph are really worth anything, as the ones on the left only run for a few milliseconds. Raw data [here](//wingolog.org/pub/v8-vs-gcc.data).

Why is V8 faster, even for significant iteration counts? Because it can inline hot function calls at runtime. It can see more than GCC can, and it takes advantage of it: but still preserving the dynamic characteristics of JavaScript. In this context the ELF symbol interposition rules that GCC has to deal with are not such a burden!

## related articles

- [cross-module inlining in guile](/archives/2021/05/13/cross-module-inlining-in-guile)
- [design notes on inline caches in guile](/archives/2018/02/07/design-notes-on-inline-caches-in-guile)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [two paths, one peak: a view from below on high-performance language implementations](/archives/2015/11/03/two-paths-one-peak-a-view-from-below-on-high-performance-language-implementations)
- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)

### 20 responses

1. AdamK says:[10 June 2011 5:14 PM](#4e74d768c082b58018f5e01e29b69b9a3e2b62ea)

   Seems logical. GCC deals with the code only once, ant after it's done there is no room for improvement. V8 deals with the same code over and over and can improve optimization over time.

   But...

   You usually compile code with GCC only once. Noone compiles everything everytime he wants to use something. So, comparing compile+run time between V8 and GCC is not fair - GCC has much harder job to do to create good code, because it can't improve it over time.

2. [Andy Wingo](http://wingolog.org) says:[10 June 2011 5:23 PM](#76bec7590305cf5e6c59127fa7ae6051a18032a5)

   Points well taken, Adam; but note that for significant workloads in GCC, the runtime is longer than the compile time, and so it is a win to do some adaptive compilation at runtime.

   That said, I am working on an AOT compiler for Guile, not a JIT. I'll write more about why I'm doing that sometime soon.

3. [Andy Wingo](http://wingolog.org) says:[10 June 2011 5:24 PM](#47e0a671034fa2e3f81f9d39fd8df05a59f77629)

   (Specifically: that for more than 2^23 iterations in this case, GCC spends more time running the code than compiling it.)

4. tobias3 says:[10 June 2011 6:01 PM](#83a789a0e20fd581db5069db5d3deacf2326edcd)

   Use the -flto compiler flag for gcc please!!! (And correct the title to: GCC compiled binaries are faster than V8)

5. [cloudhead](http://cloudhead.io) says:[10 June 2011 7:19 PM](#a5285fb326f3db491bd766e9f6de77e4bc57950e)

   Did you just benchmark a JIT compiler vs a static compiler? \*stabs self\*

6. [Andy Wingo](http://wingolog.org/) says:[10 June 2011 7:26 PM](#e2e1c94b83ba2411b8a89e6e1f16634700a51d30)

   I thought it would be obvious, but I'll say it anyway: LTO is a good step. But besides being quite new, can't help users of a shared library.

7. [Tonio](http://loewald.com/blog/) says:[10 June 2011 7:49 PM](#7674b84fcb8c320c024d0cced3fe6fdacd3eae6b)

   Nice piece. It's worth considering that a lot of code is never going to run enough times for the time saved running the code to outweigh the time spent compiling the code, especially if you take iterative development into consideration.

   In essence, one takeaway message is that — as usual — premature optimization is the root of all evil, including using a low level language when a short level language will do.

8. [Christopher Lord](http://christopher.lord.ac) says:[10 June 2011 8:11 PM](#10852d709302de768937c68c7221d9051b623cb4)

   Taking this at face value:

   GCC has a lot of overhead associated with supporting everything under the sun, including most static languages and most possible optimizations.

   V8 would have to support all dynamic languages (like ruby, perl, etc) in order to make this more comparable. In fact, V8 is specific to Javascript.

   To be fair, throw TCC or clang into the mix and see how a more minimal C-only compiler will compare.

9. Mathias Hasselmann says:[10 June 2011 10:26 PM](#4f17f4c6c56e83dcbca0527c35861c39e8365747)

   Put the code on 1 million devices.

   How is gcc compile time significant now?

   Facepalm.

   Nice troll. Attempt.

10. Eki says:[10 June 2011 10:36 PM](#2004f172602474b9d40b38344fae78f6470bb243)

    You say:

    "Well yes, that's true, and GCC does do that: but only if GCC is able and allowed to do the inlining."

    and then:

    "Because it can inline hot function calls at runtime. It can see more than GCC can, and it takes advantage of it: but still preserving the dynamic characteristics of JavaScript."

    As much as I would like to believe your test, you're comparing the V8 and GCC under unequal circunstances, why is V8 allowed to inline if you forbid GCC to do the same?

11. codebeard says:[10 June 2011 10:43 PM](#c8d6ddf9c50db004eede561aac14002d989b211b)

    Does anyone actually care if such a simple loop can be optimised better by V8? How about comparing some more practical code e.g. sorting linked lists or doing some crypto?

12. Brennan says:[10 June 2011 10:57 PM](#710356a557450d5fe16fce2f454c146fbbc632af)

    First off, the code you are running is not indicative of anything that would actually happen in the real world.

    Secondly, you are binding gcc in multiple ways and then letting V8 run optimally. By your own writing gcc compiles the entire routine to a SINGLE OP-CODE that executes in a SINGLE CLOCK CYCLE but that doesn't support your argument so you force gcc to do it in a way that is less than optimal to support your original assertion.

    Third, why are you including compile time in the benchmark? For any significant number of iterations compile time is moot.

    What you've just written is one of the most contrived benchmarks I have ever read. At no point in your rambling, incoherent argument were you even close to anything that could be considered an objective comparison. Everyone in this room is now dumber for having read it. I award you no points, and may God have mercy on your soul.

13. Jim Gagnon says:[11 June 2011 4:56 AM](#8a4580151e6b33753aaad4894f229badcc84f7bf)

    Could you please post the raw timings? I'm actually curious about the Javascript run times for higher order N.

    I hear what the compiled geeks are saying, and they're correct, but consider that V8 is Google's first stab at this. The point that V8 can have greater insight into any given code than gcc must be given weight; once could imagine a V9 that, when seeing excessive execution times, does a actual reanalysis and compile of the section of Javascript most beneficial, giving you the best of JIT and compiled languages in one package.

14. [Andy Wingo](http://wingolog.org/) says:[12 June 2011 4:58 PM](#6c81ceca01bd495e103df19a5804242389a2ccbb)

    Um, OK! I'm not sure why this pushed a bunch of people's buttons. I can only assume that I didn't explain myself clearly enough.

    There was a concern that GCC's compile-once, run-many-times characteristic is somehow being disrespected (Adam, Matthias, Brennan). That's not the case at all -- as you can see at significant iteration counts, compile time is a small fraction of run time (30ms versus about 2500ms). The fact is that V8's compiled result is simply better.

    I included GCC's compile time to be fair to V8 at low numbers of iterations, but expected GCC to be faster once compile time was a small fraction of run time (which, as no one noted, already included starting V8 from scratch each time).

    Folks seem to feel a need to evince GCC as a better compiler (Tobias, Christopher, Matthias, Eki). Of course it's better, that's not in dispute. But because it compiles early, it can't do some things that runtime compilers do, and do well. That's all I'm saying here. No need to get all defensive!

    Tobias notes that I should use -flto, and Eki asks why I have forbidden GCC to inline. Brennan also suggests that I have allowed V8 to run optimally. That is not the case. The point for me was to foil static inlining attempts in both. (See, I come from the Scheme world, where people know how to inline lexically, but generally don't know V8's dynamic inlining trick.) So in V8 I also compiled f and g in separate compilation units. -flto is just another opportunity to statically inline. Anyway GCC did not decide to do it at -O2, the flags that I used, and can't do it for a tight loop calling into a shared library.

    Oddly enough there was the criticism that this isn't real enough code. Of course it's a silly thing, and of course GCC is better for most real code. I didn't intend to benchmark it when I started looking into it for my previous article, but it was indeed amusing. That said, the fact that most current systems don't really handle tight loops across compilation units affects the way that people write real systems, providing an artificial constraint to program construction.

    Finally... the tone has really gotten terrible here. Please think before you write. Thanks!

15. [Brandon](http://dotsony.blogspot.com) says:[14 June 2011 5:01 PM](#9fc9fc3c6cdca9c5a5d275015029d3d4d4270985)

    The title is very much a button-pusher. I assumed it was it was meant to be tongue-in-cheek, but if you're serious then I'm afraid even I feel you've may have made a quantum leap to the conclusion. It seems like v8 does some really clever things, and I agree with almost everything you've mentioned, except the title itself.

    On the flip side, I am weary of C/C++ apologists that get worked up at the suggestion that other languages have their merits. It poisons discourse. To those people all I can say is, "calm down." C and C++ will be with us for a long time to come, even if advances in compiler technology are making languages like javascript "fast enough" for an increasing variety of purposes.

16. Bo says:[2 February 2012 5:20 PM](#113627852a8d2ff64c7b745faf5714d7bb74b00c)

    Oh, man, have a sense of humor. I love this post. It just showed that in some cases, JIT may be faster than a static compiler like GCC. What's wrong with this opinion? The data explained all. The author never said "let's forget C/C++ and use JS from now on."

17. JimJJewett says:[23 August 2012 6:25 PM](#ffe494ca0bf489efb668aca639bbea336d496364)

    If I'm not mistaken, there were several points (2^24 through 2^30) where V8 was faster than the gcc-compiled code, even excluding compilation time. If you assume an already running V8, there are a few more.

    Would the compiled runtimes catch up if they were compiled using profile-guided optimization? (Obviously, that makes the comparison even more favorable to V8 in the run-only-a-few-times case.)

18. Chris says:[20 September 2012 8:22 AM](#e89d119d474b70ba285882c64280d2eea3e6fb12)

    Similar things have been done with other jits before.

    http://morepypy.blogspot.com/2011/02/pypy-faster-than-c-on-carefully-crafted.html

    http://morepypy.blogspot.com/2011/08/pypy-is-faster-than-c-again-string.html

19. [Aaron Griffith](http://gammalevel.com/) says:[3 January 2013 11:27 PM](#12b84d4f8258b48640e8e2907bd02b2cbecccf0a)

    I would like to point out that your choice of data to graph, and the axes you chose to graph it on, is extremely hard to read. The data does support your statement (I was relieved to see, since hard-to-read graphs are usually a sign of funny business) but at first glance it looks like it \*refutes\* the statement.

    Why did you choose to graph \*total\* v8 time and gcc \*compile\* time together? Why not totals for both? Or at least, run and compile times for both, preferably stacked on top of each other. You can still keep the relative % scale, it's just confusing to use two different metrics for data that shows up side-by-side.

    Also at first glance it's easy to miss that the X axis is logarithmic. It'd be better to use an axis with logarithmic tick marks and true value labels, or at the very least label with 2^N directly instead of hiding that detail in the label.

    Because of both of these, I seriously stared at this and thought "wow, GCC is faster at only 24 iterations" for a few minutes.

20. [Arne Babenhauserheide](http://draketo.de) says:[29 January 2013 8:50 AM](#bec38cb68f13d5db5b48e5b8dc520a3a1bca6896)

    Just found this graphpost, and wonder if you’re serious.

    Did you want to see whether GCC compilation would be a better fit for optimizing JS on the web?

Comments are closed.
