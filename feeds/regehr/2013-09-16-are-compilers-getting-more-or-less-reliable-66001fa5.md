---
title: Are Compilers Getting More or Less Reliable?
url: https://blog.regehr.org/archives/1036
published: "2013-09-16T03:56:44Z"
feed: regehr
guid: http://blog.regehr.org/?p=1036
---

# Are Compilers Getting More or Less Reliable?

Earlier this summer, Miod Vallat of the OpenBSD project [posted some thoughts](http://marc.info/?l=openbsd-misc&m=137530560232232&w=2) about compiler reliability. His thesis is that in the good old days of GCC 2.5 through 2.7, the compiler pretty much just worked and you could trust it to emit correct code. Subsequently, the level of code churn in GCC increased for several reasons:

- new standards appeared, such as C++98

- more effort was made to exploit platform-specific performance opportunities

- more people started working on the compiler

The result is that “gcc had bugs, and you were expected to accept that and cope with them.” Upgrading the compiler is one way to get bugs fixed, but this is a mixed bag since new bugs are also added. What is needed is compiler versions with long-term support.

This post is a collection of responses to Miod’s post and also a bit of an attempt to figure out if open-source compilers have really gotten less reliable over the past 20 years.

First of all, to my eye the GCC project already does a good job in supporting their compilers. For example, 4.4.0 was released in April 2009 and 4.4.7 was released in March 2012 — that’s almost three years’ worth of bug fix releases. As a frequent reporter of compiler bugs, I can confirm that it is common for the GCC developers to take a bug report against trunk and backport the fix to at least one of the maintenance branches. So what else is needed here– Longer maintenance periods? More fixes being backported? I’d appreciate hearing from people about this. Miod’s mail indicates that he has some specific complaints:

> … gcc developers have been eager to release “bug free” new versions by enforcing a policy that only “regressions” would get fixed, and spending more time changing their definition of “regression” or trying to explain why regressions weren’t, so as not need to fix them in any stable release.

I would tend to respect the opinions of an OpenBSD port maintainer, but on the other hand my observation is that the GCC people are in fact putting quite a bit of effort into the maintenance branches. Of course one could always argue that they are not doing enough.

The overall context for Miod’s mail is an ongoing discussion in the OpenBSD community about switching over to Clang as the default system compiler. One problem with this idea is that at present, the LLVM project does not have any maintenance branches at all. So realistically this switch would seem to be a step backward in terms of compiler stability. I’m not complaining about the reliability of Clang (see below for more details on this); rather, my group writes code that interfaces with LLVM/Clang and our experience is that these projects move forward *very* quickly.

The second thing I wanted to say is that I think we can add even more items to Miod’s list of reasons why GCC might be getting buggier over time:

- GCC is now significantly larger than it was: 1300 KLOC for 4.8.1 vs. 331 KLOC for GCC 2.7.2.3 — that’s a million new lines of code for bugs to hide in, and much of this code is as hairy as you’ll see anywhere

- the codes being compiled now are significantly larger and more complex than they were 20 years ago during the GCC 2.5 through 2.7 era; large and complex codes are more likely to trigger compiler bugs

- the scope of compiler optimizations, in terms of how much code each optimization considers, has increased substantially in the last 20 years — this makes it harder to get the optimizations right

- automatically generating code that uses SIMD instructions is tricky, and we do a lot more of that now

After writing this list I feel like we’re lucky to have compilers that work at all. But are there any trends indicating that GCC should be getting more reliable? I think so:

- vastly improved compiler warnings, static analyzers, and dynamic tools such as Valgrind support easy detection of some kinds of bugs that used to be difficult

- GCC’s move to an SSA-based internal representation, which caused many bugs at first, has permitted optimizations to be cleaned up

- GCC’s test suite has gotten better

- GCC attempts to fail rapidly when something goes wrong by using assertions much more heavily than it used to; I count about 12,000 assertions now, compared to fewer than 500 in 2.7.2.3 (these counts are very rough, using grep)

So now we’re ready to tackle the original question: Was GCC 2.7 really more reliable than our modern open-source compilers? Answering this kind of question is hard and in fact I don’t think anyone — in industry or in academia — has published quantitative information comparing the reliability of various compilers. It just isn’t done. My method (which has some obvious shortcomings that I’ll discuss later) was to take three compilers — GCC 2.7.2.3 for x86, a recent Clang snapshot for x86-64, and a recent GCC snapshot for x86-64 — and use them to compile 100,000 random programs generated by [Csmith](http://embed.cs.utah.edu/csmith/). Each program was compiled at `-O0`, `-O1`, `-O2`, `-Os` (if supported — GCC 2.7 did not have this flag yet), and one of `-O3` or `-Ofast`. If any compiler crashed or failed to terminate within three minutes, this was noted. Then the compiled programs were run and their checksums were compared. These checksums are a feature of programs generated by Csmith; they are computed by looking at the values in all global variables just before the randomly-generated program terminates. The intent of the checksum is to support easy differential testing where we don’t know the correct result for a test case but we do know that the result is not supposed to change across compilers or optimization options. If changing the optimization level changed a checksum, the compiler was considered to have emitted incorrect code. For more details see the [Csmith paper](http://www.cs.utah.edu/~regehr/papers/pldi11-preprint.pdf).

Here are the results:

crashestimeoutswrong codeGCC 2.7.2.31386668239GCC r202341403Clang r190166007

Here’s an example of a program that, when compiled by GCC 2.7.2.3 at `-O2`, causes the compiler to run for more than three minutes (in fact, it runs for more than 60 minutes — at which point I got bored and killed it):

```
int x0;
int x1() {
  for (;;) {
    x0 = 0;
    for (; x0;) {
      int x2 = 1;
      if (x2) return 0;
    }
  }
}

```

I didn’t investigate the crash bugs or wrong-code bugs in GCC 2.7, these are probably of very little interest now. It’s impressive that Clang never crashed in 100,000 test cases. The four crashes in the current GCC found during this testing run were all variants of [this known problem](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=57393); a patch exists, but it hasn’t been applied to trunk yet. The instances of incorrect code generation in Clang were due to some previously-unknown bugs that I reported as [PR 17158](http://llvm.org/bugs/show_bug.cgi?id=17158) and [PR 17179](http://llvm.org/bugs/show_bug.cgi?id=17179), which are triggered respectively by the following programs.

PR 17158:

```
int printf(const char *, ...);
struct x0 {
  int x1;
  char x2;
};
struct x3 {
  struct x0 x4;
  char x5;
} x6;
int main() {
  struct x3 x7 = { { 0, 0 }, 1 };
  x6 = x7;
  x7.x4.x2;
  printf("%d\n", x6.x5);
  return 0;
}

```

PR 17179:

```
int printf(const char *, ...);
int x0, x1, x2;
int main (void) {
  int x3;
  for (x1 = 0; x1 != 52; x1++) {
    for (x2 = 0; x2 <= 0; x2 = 1) {
      int x4 = x0 ^ x1;
      x3 = x4;
    }
  }
  printf("%d\n", x3);
  return 0;
}

```

The GCC miscompilations were also due to previously unknown bugs; I reported them as [PR 58364](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=58364), [PR 58365](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=58365), and [PR 58385](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=58385). Here are the two shorter ones.

PR 58364:

```
int printf(const char *, ...);
int x3 (int x4) {
  return x4 < 0 ? 1 : x4;
}
int x0 = 1, x1, x2;
int main (void) {
  if (x3(x0 > x2 == (x1 = 0))) {
    printf("x\n");
  }
  return 0;
}

```

PR 58385:

```
int printf(const char *, ...);
int x0, x1 = 1;
int x2() {
  x1 = 0;
  return 0;
}
int main() {
  ((0 || x0) & x2() >= 0) <= 1 && 1;
  printf("%d\n", x1);
  return 0;
}

```

The three new wrong-code bugs in GCC were all fixed within a few days of being reported. So, I ran another 100,000 random programs against an updated snapshot and here are the results:

crashestimeoutswrong codeGCC r202599900

Out of the nine crashes, six were due to the PR 57393 problem mentioned above and the other three were due to null pointer dereference issues. The GCC bug database already contains a number of open bugs for segfaults in the compiler so I’m not in any big rush to report these.

I did not run another Clang test since neither of the new bugs in that compiler has been fixed yet.

Let’s get back to the original question: Has the reliability of open source compilers decreased since GCC 2.7? This does not appear to be the case. In fact, this experiment indicates that the opposite might be true. Of course we need to be careful in interpreting these results, since randomly generated C programs are not necessarily representative: they could trigger bugs that no real C program triggers, while also missing all of the bugs that Miod is complaining about. On the other hand, take a look at the C programs above that trigger the new wrong-code bugs and try to convince yourself that these patterns will never occur during compilation of real code. A different source of bias in these results is that GCC 2.7 was never tested with Csmith, but the current versions of GCC and Clang have been subjected to extensive random testing.

Getting back to Miod’s point about the need for LTS versions of compilers, I completely agree: this should be done. Five or even 10 years of support would not be too long, especially in the embedded systems world where old compilers often persist long after their expire-by date. I suspect that the main problem is that compiler maintenance is too difficult and unsexy to attract lots of volunteers. Of course people could be paid to do it, but as far as I know nobody has stepped up to fund this sort of effort.

Miod’s mail received some [comments at Hacker News](https://news.ycombinator.com/item?id=6141319).

Updates from Sep 16:

- good reading: [“FreeBSD 10 Alpha Built With clang”](http://www.infoq.com/news/2013/09/freebsd10-clang)
- I added a bit of explanation of Csmith’s checksumming strategy

- I added the test case text for four of the five new wrong-code bugs just because I think they’re interesting
