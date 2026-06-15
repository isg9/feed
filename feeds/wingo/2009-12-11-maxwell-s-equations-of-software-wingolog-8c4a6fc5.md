---
title: maxwell's equations of software — wingolog
url: https://wingolog.org/archives/2009/12/11/maxwells-equations-of-software
published: "2009-12-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/12/11/maxwells-equations-of-software
---

# maxwell's equations of software — wingolog

## [maxwell's equations of software](/archives/2009/12/11/maxwells-equations-of-software)

11 December 2009 2:23 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [meta-circular](/tags/meta-circular)
- [evaluator](/tags/evaluator)
- [compiler](/tags/compiler)

There was some interesting feedback on my [last article](//wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty), and I ended up wanting to say too much in reply that here I am again, typing at the ether.

Stephen [reflects](//wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty#bd765f51279943578e7c6e6156e5664605803c3b) the common confusion that somehow this is just a little file that might or might not work. No man, this is Guile! That's the implementation of Guile's `eval`. So unit tests, yes, we have more than 12000 of them. Only some of them specifically test the evaluator, but all of them go through the evaluator.

But to be honest, I think unit tests help, but when I'm hacking the compiler the most useful test is to simply rebuild the compiler. If it successfully bootstraps, usually we're doing pretty well.

Zed [raises](http://twitter.com/zedshaw/status/6541845920) a more direct criticism:

> This is why I hate lisp: *\[ [link to my article](//wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty)\]* Long and dense, no comments, not semantic. That's "awesome code"?

I don't think I used the word "awesome", but yes, I believe so, in the sense of "inspiring awe" -- at least for me.

I'm going to call on my homie Alan Kay for some support here. There was an oft-cited [interview with Alan Kay](http://queue.acm.org/detail.cfm?id=1039523) a few years back, when he described `eval` as "Maxwell's equations of software". I quote:

> *\[Interviewer:\]* If nothing else, Lisp was carefully defined in terms of Lisp.
>
> *\[Alan Kay:\]* Yes, that was the big revelation to me when I was in graduate school—when I finally understood that the half page of code on the bottom of page 13 of the Lisp 1.5 manual was Lisp in itself. These were “Maxwell’s Equations of Software!” This is the whole world of programming in a few lines that I can put my hand over.
>
> I realized that anytime I want to know what I’m doing, I can just write down the kernel of this thing in a half page and it’s not going to lose any power. In fact, it’s going to gain power by being able to reenter itself much more readily than most systems done the other way can possibly do.

Just for context, here is that half-a-page -- Maxwell's equations of software:

> `evalquote` is defined by using two main functions, called `eval` and `apply`. `apply` handles a function and its arguments, while `eval` handles forms. Each of these functions also has another argument that is used as an association list for storing the values of bound variables and function names.
>
> ```
>  evalquote[fn;x] = apply[fn;x;NIL]
>
> ```
>
> where
>
> ```
>  apply[fn;x;a] =
>        [atom[fn] → [eq[fn;CAR] → caar[x];
>                     eq[fn;CDR] → cdar[x];
>                     eq[fn;CONS] → cons[car[x];cadr[x]];
>                     eq[fn;ATOM] → atom[car[x]];
>                     eq[fn;EQ] → eq[car[x];cadr[x]];
>                     T → apply[eval[fn;a];x;a]];
>         eq[car[fn];LAMBDA] → eval[caddr[fn];pairlis[cadr[fn];x;a]];
>         eq[car[fn];LABEL] → apply[caddr[fn];x;cons[cons[cadr[fn];
>                                                    caddr[fn]];a]]]
>
>  eval[e;a] =
>        [atom[e] → cdr[assoc[e;a]];
>         atom[car[e]] → [eq[car[e];QUOTE] → cadr[e];
>                         eq[car[e];COND] → evcon[cdr[e];a];
>                         T → apply[car[e];evlis[cdr[e];a];a]];
>         T → apply[car[e];evlis[cdr[e];a];a]]
>
> ```
>
> `pairlis` and `assoc` have been previously defined.
>
> ```
>  evcon[c;a] = [eval[caar[c];a] → eval[cadar[c];a];
>                T → evcon[cdr[c];a]]
>
> ```
>
> and
>
> ```
>  evlis[m;a] = [null[m] →  NIL;
>                T → cons[eval[car[m];a];evlis[cdr[m];a]]]
>
> ```
>
> *From the [LISP 1.5 Programmer's Manual](http://www.softwarepreservation.org/projects/LISP/book/LISP%201.5%20Programmers%20Manual.pdf)*

What a mess! Where are the unit tests? Where are the comments? A little bit of whitespace, please! They use "cdar", "caar", *and* "cadr". They should be using named accessors! What if you eval an atom that's not found in the association list? Did they really name a function "evlis"? Et cetera.

Now, I do think the LISP 1.5 (it wasn't yet spelled "Lisp" then) evaluator is awesome, as it was in McCarthy's first papers on the subject. If you disagree with that, that's cool, I see why one would "hate" my poor imitation.

But given the Lisp history of meta-circular evaluators, one doesn't need to comment every one as if it were the first. In the same way that one recognizes an `if` statement without the need for a [design pattern](http://people.csail.mit.edu/gregs/ll1-discuss-archive-html/msg01629.html) behind it, I would want anyone who's hacking on Guile's evaluator to have read or perhaps even written several evaluators; and once you've written one, you know the pattern.

More substantively, Charles [speculates](http://www.advogato.org/person/chalst/diary/256.html) on source-to-source transformations to ease interpretation. I admit almost total ignorance regarding PreScheme; I've been meaning to learn about it for years now. But the point of this evaluator was to have it use the same representation as VM-compiled procedures; ideally it should run fast, but the first priority is for it to run on the VM itself instead of on a separate stack. There's certainly many more clever things that can be done there, and thankfully since `eval` is implemented in Scheme and compiled like anything else, it will also reap advantages of an improving compiler.

Regarding Emacs, I hope to say more on that point within a week or so, when the video for my talk at the recent GNU Hackers Meeting gets put up.

Finally, Bubo [asks](//wingolog.org/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty#abfe59e953b9b71e25dc24ee3f9a53ca5882eb73) if this work is in a released tarball. Not yet is the answer, but it will be in Tuesday's 1.9.6 release.

Happy hacking!

## related articles

- [in which our protagonist forgoes modesty](/archives/2009/12/09/in-which-our-protagonist-forgoes-modesty)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [scheme quiz time!](/archives/2013/11/02/scheme-quiz-time)
- [visualizing statistical profiles with chartprof](/archives/2009/02/09/visualizing-statistical-profiles-with-chartprof)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### 6 responses

1. bubo says:[11 December 2009 4:02 PM](#15655b7b7d41cc6b726dbe4fdb640b43f6526472)

   thanks for the reply! it wasn't in the tarball but already in git. i need quite a bit of libraries on Debian testing to install the stuff in the git repo, so i'll give it a shot on the weekend. otherwise: tuesday == eval/apply-day :)! if this realy works it should give guile quite a boost in performance. if not at least debugging will be more fun in the future. by the way: IMHO scheme48 is one of the best scheme's out there in terms of design, power, features. but it's quite slow. so i don't think that the underlying prescheme (which is AFAIK r4rs) has helped it a lot in terms of program execution speed. however, thanks a lot for making guile \*better\*! bye

2. [wingo](http://wingolog.org/) says:[11 December 2009 6:02 PM](#f71ec60b436abb49d7e0cfcd6db3ba14e4574204)

   Thanks for stopping by. Regarding speed of `eval` however, I am afraid you will be disappointed; consider that the C evaluator that we have, for bootstrapping purposes uses the same algorithm as the Scheme evaluator. So you have the same algorithm implemented in two languages. The faster implementation is the one whose compiler produces the fastest code. Guile's compiler is not as fast as GCC, and probably will never be, though who knows.

   Where we win is by having a uniform semantics, and a uniform debugging interface (once eval.scm is compiled).

   That said, whether the evaluator is in Scheme or C doesn't affect the speed of compiled code, which is the vast majority of code executed by Guile. Even the REPL is really a read-compile-run-print loop, not a read-eval-print loop. But since the semantics are the same, and the compiler is zippy enough for REPL purposes, no one notices.

3. [rotty](http://rotty.yi.org/) says:[12 December 2009 0:04 AM](#a7d8f76886655a2008b7feee9fae050a3e61cd48)

   @bubo: Regarding PreScheme: IIRC, PreScheme is a Scheme-like language with quite heavy restrictions (e.g. no garbage collection) that can be compiled to C and is used to implement the Scheme48 virtual machine. So it's not really R4RS, or any Scheme (could a language without GC still be called Scheme?).

4. Thomas says:[12 December 2009 4:19 PM](#1414928e4d293d57cbc6af1f0f4a4367280dd784)

   Hi,

   thanks a lot for your great improvements on Guile. I am once again awed by your work ^^

   I just listened to your talk/skimmed through the slides and I must say I'm really excited about the idea of a Guile-based GNU Emacs.

   Keep up the good work!

   Thomas

5. bubo says:[14 December 2009 5:28 AM](#356468d4c937faf371a4cff009d9ce12f15a4365)

   @wingo: to be honest - i haven't thought this through entirely. i try to get proficient to hack in guile and c not on guile and c. i'm not god enough for that. i read the guile-devel mail but never post - i'd just hold you back...;)

   @rotty: hmmm. i remember olin shivers saying something like this in the talk, he gave on dan friedman's 60th birthday. i have a paper on prescheme anywhere in an archive, which means i can't find it. without gc it can't be r4rs, that's for sure... however: when is serveez coming back to squeeze, man? i'm really missing my favorite server! :)

6. John Cowan says:[13 April 2010 5:56 PM](#fd1f39aad2b641218f7bdbfc630657faeb44724d)

   If you're not familiar with how the Chicken Scheme bootstrap works, you may be interested. Chicken is primarily meant for creating libraries and stand-alone applications, not as an extension language, so it's not directly comparable to Guile.

   Chicken's compiler generates C code which is then compiled with gcc. There is also some run-time code written in C, and a single assembler routine that obviates the need for libffi in simple cases. The interpreter is a library function written in Chicken and compiled with the Chicken compiler and gcc; there is an application wrapper around it that provides a stand-alone REPL for Chicken.

   So then, how does Chicken compile its compiler? In normal operation, with a pre-existing Chicken compiler, provided it is not too old. If it is too old, or if you have no existing Chicken compiler on the system, you download the C sources of the latest stable Chicken compiler and compile them with gcc.

   The resulting bootstrap compiler is then used to compile the new Chicken compiler, which then compiles itself in order to take advantage of any recent improvements in the compiler. Given that, all the Chicken libraries and utilities can be compiled, and there you are. You can then throw away both binary and source of the bootstrap compiler, as normal updating will employ the compiler you just created.

Comments are closed.
