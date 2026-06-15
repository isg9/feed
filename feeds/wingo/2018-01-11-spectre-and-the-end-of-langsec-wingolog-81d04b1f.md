---
title: spectre and the end of langsec — wingolog
url: https://wingolog.org/archives/2018/01/11/spectre-and-the-end-of-langsec
published: "2018-01-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2018/01/11/spectre-and-the-end-of-langsec
---

# spectre and the end of langsec — wingolog

## [spectre and the end of langsec](/archives/2018/01/11/spectre-and-the-end-of-langsec)

11 January 2018 1:44 PM

- [compilers](/tags/compilers)
- [security](/tags/security)
- [spectre](/tags/spectre)
- [meltdown](/tags/meltdown)

I remember in 2008 seeing Gerald Sussman, creator of the Scheme language, resignedly describing a sea change in the MIT computer science curriculum. In response to a question from the audience, [he said](https://wingolog.org/archives/2009/03/24/international-lisp-conference-day-two):

> The work of engineers used to be about taking small parts that they understood entirely and using simple techniques to compose them into larger things that do what they want.
>
> But programming now isn't so much like that. Nowadays you muck around with incomprehensible or nonexistent man pages for software you don't know who wrote. *You have to do basic science on your libraries to see how they work*, trying out different inputs and seeing how the code reacts. This is a fundamentally different job.

Like many I was profoundly saddened by this analysis. I want to believe in constructive correctness, in math and in proofs. And so with the rise of functional programming, I thought that this historical slide from reason towards observation was just that, historical, and that the "safe" languages had a compelling value that would be evident eventually: that "another world is possible".

In particular I found solace in "langsec", an approach to assessing and ensuring system security in terms of constructively correct programs. One obvious application is parsing of untrusted input, and indeed the [langsec.org website](http://langsec.org/) appears to emphasize this domain as one in which a programming languages approach can be fruitful. It is, after all, [a truth universally acknowledged, that a program with good use of data types, will be free from many common bugs.](https://www.gnu.org/software/guile/manual/html_node/Types-and-the-Web.html#Types-and-the-Web) So far so good, and so far so successful.

The basis of language security is starting from a programming language with a well-defined, easy-to-understand semantics. From there you can prove (formally or informally) interesting security properties about particular programs. For example, if a program has a secret *k*, but some untrusted subcomponent *C* of it should not have access to *k*, one can prove if *k* can or cannot leak to *C*. This approach is taken, for example, by [Google's Caja compiler](https://developers.google.com/caja/) to isolate components from each other, even when they run in the context of the same web page.

But the [Spectre](https://spectreattack.com/) and [Meltdown](https://meltdownattack.com/) attacks have seriously set back this endeavor. One manifestation of the Spectre vulnerability is that code running in a process can now read the entirety of its address space, bypassing invariants of the language in which it is written, even if it is written in a "safe" language. This is currently being used by JavaScript programs to exfiltrate passwords from a browser's password manager, or bitcoin wallets.

Mathematically, in terms of the semantics of e.g. JavaScript, these attacks should not be possible. But practically, they work. Spectre shows us that the building blocks provided to us by Intel, ARM, and all the rest are no longer "small parts understood entirely"; that instead now we have to do "basic science" on our CPUs and memory hierarchies to know what they do.

What's worse, we need to do basic science to come up with adequate mitigations to the Spectre vulnerabilities (side-channel exfiltration of results of speculative execution). [Retpolines](https://support.google.com/faqs/answer/7625886), [poisons and masks](https://webkit.org/blog/8048/what-spectre-and-meltdown-mean-for-webkit/), et cetera: none of these are *proven* to work. They are simply *observed* to be effective on current hardware. Indeed mitigations are anathema to the correctness-by-construction: if you can prove that a problem doesn't exist, what is there to mitigate?

Spectre is not the first crack in the edifice of practical program correctness. In particular, [timing side channels](https://wingolog.org/archives/2014/12/02/there-are-no-good-constant-time-data-structures) are rarely captured in language semantics. But I think it's fair to say that Spectre is the most devastating vulnerability in the langsec approach to security that has ever been uncovered.

Where do we go from here? I see but two options. One is to attempt to make the behavior of the machines targetted by secure language implementations behave rigorously as architecturally specified, and in no other way. This is the approach taken by all of the deployed mitigations (retpolines, poisoned pointers, masked accesses): modify the compiler and runtime to prevent the CPU from speculating through vulnerable indirect branches (prevent speculative execution), or from using fetched values in further speculative fetches (prevent this particular side channel). I think we are missing a model and a proof that these mitigations restore target architectural semantics, though.

However if we did have a model of what a CPU does, we have another opportunity, which is to incorporate that model in a semantics of the target language of a compiler (e.g. micro-x86 versus x86). It could be that this model produces a co-evolution of the target architectures as well, whereby Intel decides to disclose and expose more of its microarchitecture to user code. Cacheing and other microarchitectural side-effects would then become explicit rather than transparent.

Rich Hickey has this thing where he talks about "simple versus easy". Both of them sound good but for him, only "simple" is good whereas "easy" is bad. It's the sort of subjective distinction that can lead to an endless string of [Worse Is Better Is Worse](https://www.dreamsongs.com/Files/worse-is-worse.pdf) Bourbaki papers, according to the perspective of the author. Anyway transparent caching in the CPU has been marvelously easy for most application developers and fantastically beneficial from a performance perspective. People needing constant-time operations have complained, of course, but that kind of person always complains. Could it be, though, that actually there is some other, better-is-better kind of simplicity that should replace the all-pervasive, now-treacherous transparent cacheing?

I don't know. All I will say is that an ad-hoc approach to determining which branches and loads are safe and which are not is not a plan that inspires confidence. Godspeed to the langsec faithful in these dark times.

## related articles

- [on "binary security of webassembly"](/archives/2020/10/15/on-binary-security-of-webassembly)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [security implications of jit compilation](/archives/2011/06/21/security-implications-of-jit-compilation)
- [wastrelly wabbits](/archives/2026/03/31/wastrelly-wabbits)
- [six thoughts on generating c](/archives/2026/02/09/six-thoughts-on-generating-c)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)

### 6 responses

1. RobT says:[11 January 2018 4:13 PM](#ef8eb7b32d8e71471a3213169fefcfa2dd6d4a40)

   Check out riscv-formal

2. Thomas Dullien says:[11 January 2018 4:25 PM](#db2f2f430e99b7655a4707ca549de7c4260fefd1)

   Hey there,

   I think this is too dark a post, but it shows a useful shock: Computer Science likes to live in proximity to pure mathematics, but it lives between EE and mathematics. And neglecting the EE side is dangerous - which not only Spectre showed, but which should have been obvious at the latest when Rowhammer hit.

   There's actual physics happening, and we need to be aware of it.

   Cheers,

   Thomas

3. Nahuel says:[11 January 2018 5:37 PM](#5c8599e270c49e0b7094f747bfa0629772461885)

   http://wiki.c2.com/?LeakyAbstraction

4. [Mark S. Miller](https://research.google.com/pubs/author35958.html) says:[12 January 2018 9:55 AM](#9150b8770a3cdb9989aa613e1896cf4f96c5e865)

   This is hardly the end of langsec. The confidentiality claims of Caja are indeed threatened, need to be reevaluated, and may now be invalid. However, we never made strong confidentiality claims, precisely because we had no formal basis for believing we were secure against leakage via non-overt (covert and side) channels.

   The integrity claims of Caja are intact and undiminished. See https://groups.google.com/d/msg/friam/WBr0-UBLm50/iGwjh00sAAAJ and the thread around it.

5. [Arne Babenhauserheide](http://draketo.de) says:[21 January 2018 9:11 PM](#283f504ef2718e0e5160d2260e95b4b938cf06a8)

   Once there is co-evolution between hardware and language creators, the languages become part of the competition between hardware creators - you can see the effects of that in CUDA vs. OpenCL and OpenMP vs. MPI vs. OpenACC. It can be beneficial, but it ties your program to a specific hardware platform for years - sometimes for ever. It takes you to a point where a program only works well if you can figure out the correct compile flags to activate all the required hardware specific libraries (like the math kernel library (mkl) for intel).

   You’d need a way to isolate the low level details of the language interface with the hardware from the high level constructs programmers use in their actual programs.

6. Bob Bishop says:[23 January 2018 2:50 PM](#acac781216a207dc77198f45b7b0985ef4c51eb9)

   Langsec: "let's avoid vulnerabilities in the programs we write", this post's title and mood: "there are vulnerabilities elsewhere, langsec is dead". Non sequitur, and such an argument didn't even need Spectre. The post itself doesn't quite say what the title says though, did I get click-bitten?

Comments are closed.
