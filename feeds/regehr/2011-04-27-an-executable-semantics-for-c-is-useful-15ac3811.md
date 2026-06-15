---
title: An Executable Semantics For C Is Useful
url: https://blog.regehr.org/archives/523
published: "2011-04-27T05:26:32Z"
feed: regehr
guid: http://blog.regehr.org/?p=523
---

# An Executable Semantics For C Is Useful

The goal of a C/C++ compiler is to turn every sequence of ASCII characters into executable instructions. OK, not really — though it does seem that way sometimes. The real goal of a C/C++ compiler is to map every conforming input into executable instructions that correspond to a legal interpretation of that input. The qualifiers “conforming” and “legal interpretation” are very important. First, the compiler has extremely weak requirements about what it should do with non-conforming inputs, such as programs that contain undefined behaviors (array bounds violations, etc.). Second, all realistic C/C++ programs have a large number of possible interpretations, for example corresponding to different integer sizes, different orders of evaluation for function arguments, etc. The compiler chooses a convenient or efficient one, and the remaining interpretations are latent. They may emerge later on if the compiler options are changed, if the compiler is upgraded, or if a different compiler is used. The point is that the compiler has no obligation to tell us whether the input is conforming or not, nor how many possible interpretations it has.

Thus, while C/C++ compilers are very good at turning conforming programs into efficient executables, they are just about useless for other answering other kinds of questions:

- Does the program ever execute undefined behaviors, causing it (in principle) to have no meaning and (in practice) to execute attack code or crash?
- Does the program rely on unspecified behaviors, making it non-portable across compilers, compiler versions, and changes in compiler options?
- Does the program rely on implementation-defined behaviors, affecting its portability to other compilers and platforms?
- Why does the program behave in a certain way? In other words, what part of the standard forced that interpretation?

To answer these questions, a wide variety of static analyzers, model checkers, runtime verifiers, and related tools have been developed. These tools are great. However, even taken all together, they are incomplete: there exist plenty of bad (or interesting) program behaviors that few or none of them can find. For example:

- Very few tools exist that can [reliably detect uses of uninitialized storage](https://blog.regehr.org/archives/519).
- Few, if any, tools can correctly diagnose problems resulting from C/C++’s unspecified order of evaluation of function arguments.
- An lvalue must not be modified multiple times, or be both read and written, in between sequence points. I’m not aware of many tools that can correctly detect that evaluating this function results in undefined behavior when p1 and p2 are aliases:

> ```
> int foo (int *p1, int *p2) {
>   return (*p1)++ % (*p2)++;
> }
> ```

- There exists C code subtle enough that [multiple independent compiler teams get it wrong](https://blog.regehr.org/archives/482). Miscompilations are, by definition, out of reach for source-level analysis tools.

# The Missing Tool

The missing tool (or one of them, at any rate) is an executable semantics for C. An executable semantics is an extremely careful kind of interpreter where every action it takes directly corresponds to some part of the language standard. Moreover, an executable semantics can be designed to tell us whether the standard assigns any meaning at all to the program being interpreted. In other words, it can explicitly check for all (or at least most) undefined, unspecified, and implementation-defined behaviors. For example, when an executable semantics evaluates (\*p1)++ % (\*p2)++, it won’t assign a meaning to the expression until it has checked that:

- both pointers are valid
- neither addition overflows (if the promoted types are signed)
- p1 and p2 are not aliases
- \*p2 is not 0
- either \*p1 is not INT\_MIN or \*p2 is not -1

Moreover, the tool should make explicit all of the implicit casts that are part of the “usual arithmetic conversions.” And it needs to do about 500 other things that we usually don’t think about when dealing with C code.

# Who Needs an Executable Semantics?

Regular programmers won’t need it very often, but they will occasionally find it useful for settling the kinds of annoying arguments that happen when people don’t know how to read the all-too-ambiguous English text in the standard. Of course, the executable semantics can only settle arguments if we agree that it has captured the sense of the standard. Better yet, we would treat the executable semantics as definitive and the document as a derivative work.

Compiler developers need an executable semantics. It would provide a simple, automated filter to apply to programs that purportedly trigger compiler bugs. [A web page at Keil](http://www.keil.com/support/bugreport.asp) states that “Fewer than 1% of the bug reports we receive are actually bugs.” An executable semantics would rapidly find code fragments that contain undefined or unspecified behaviors — these are a common source of bogus bug reports. Currently, compiler developers do this checking by hand. The GCC bug database contains 4966 bug reports that have been marked as INVALID. Not all of these could be automatically detected, but some of them certainly could be.

People developing safety-critical software may get some benefit from an executable semantics. Consider CompCert, a verified compiler that provably preserves the meaning of C code when translating it into assembly. CompCert’s guarantee, however, is conditional on the C code containing no undefined behaviors. How are we supposed to verify the absence of undefined behaviors when existing tools don’t reliably check for initialization and multiple updates to lvalues? Moreover, CompCert is free to choose any legitimate interpretation of a C program that relies on unspecified behaviors, and it does not need to tell us which one it has chosen. We need to verify up-front that (under some set of implementation-defined behaviors) our safety-critical C program has a single interpretation.

My students and I need an executable semantics, because we are constantly trying to figure out whether random C functions are well-defined or not. This is surprisingly hard. We also need a reliable, automated way to detect undefined behavior because this enables automated test case reduction.

# An Executable Semantics for C Exists

I spent a few years lamenting the non-existence of an executable C semantics, but no longer: as of recently, the tool exists. It was created by [Chucky Ellison](http://www.freefour.com/), a PhD student at the University of Illinois working under the supervision of [Grigore Rosu](http://fsl.cs.uiuc.edu/index.php/Grigore_Rosu). They have written a [TR about it](http://fsl.cs.uiuc.edu/pubs/ellison-rosu-2010-tr.pdf) and also [the tool can be downloaded](http://code.google.com/p/c-semantics/). Hopefully Chucky does not mind if I provide this link — the tool is very much a research prototype (mainly, it is not very fast). But it works:

> ```
> regehr@home:~/svn/code$ cat lval.c
> int foo (int *p1, int *p2) {
>   return (*p1)++ % (*p2)++;
> }
>
> int main (void) {
>   int a = 1;
>   return foo (&a, &a);
> }
> regehr@home:~/svn/code$ kcc lval.c
> regehr@home:~/svn/code$ ./a.out
> =============================================================
> ERROR! KCC encountered an error while executing this program.
> =============================================================
> Error: 00003
> Description: Unsequenced side effect on scalar object with value computation of same object.
> =============================================================
> File: /mnt/z/svn/code/lval.c
> Function: foo
> Line: 2
> ============================================================
> ```

As I mentioned earlier, very few tools for analyzing C code find this error. Chucky’s tool can also perform a state space exploration to find order of evaluation problems and problems in concurrent C codes. Finally, it can run in profile mode. Unlike a regular profiler, this one profiles the rules from the C semantics that fire when the program is interpreted. This is really cool and we plan to use it to figure out what parts of the C standard are not exercised by Csmith.

Chucky’s tool is already an integral part of one of our test case reducers. This reducer takes as input a huge, ugly, bug-triggering C program generated by Csmith. It then uses Delta debugging to output a much smaller bug-triggering program that (ideally) can be included in a compiler bug report without further manual simplification. Before Chucky’s tool arrived, we had spent several years trying to deal with the fact that Delta always introduces undefined behavior. We now seem to have a bulletproof solution to that problem.

The benefits of executable semantics have long been recognized in the PL community. The new thing here is a tool that actually works, for the C language. Hopefully, as Chucky’s tool matures people will find more uses for it, and perhaps it can even evolve into a sort of de facto litmus test for ascertaining the meaning — or lack thereof — of difficult C programs.
