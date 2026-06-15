---
title: Is That a Compiler Bug?
url: https://blog.regehr.org/archives/26
published: "2010-02-14T19:34:27Z"
feed: regehr
guid: http://blog.regehr.org/?p=26
---

# Is That a Compiler Bug?

It’s common for a computer program to display behavior seemingly unrelated to its source code.  Additionally, programs written in C/C++ frequently change their behavior when the compiler optimization options are changed.  This post is a quick guide to figuring out if the compiler is at fault.  This post was motivated by [a blog post by Eric Raymond](http://esr.ibiblio.org/?p=1705).  Eric is addressing a more general topic — heisenbugs — but I felt that the compiler side of the story needed clarification.

# Not Compiler Bugs

It’s best to start out assuming that the code being compiled is broken, not the compiler.  Close to 200 kinds of “undefined behavior” are mentioned in the C and C++ standards.  For example, dividing by zero, shifting an integer by an amount not less than its bitwidth, accessing past the end of an array, and updating an object multiple times between sequence points are all undefined behaviors.  The compiler is permitted to assume that these never happen, and its behavior is unconstrained when they do happen.  Additionally, C/C++ leave some important behaviors, such as the order of evaluation of arguments to a function, “unspecified.”  Unspecified means that some ordering must be chosen, but that the compiler can choose a different order every time based on the optimization options, the phase of the moon, etc.  Undefined and unspecified behavior are at the root of most suspected compiler bugs.  There are many ways to avoid these: use tools like valgrind, enable and heed compiler warnings, write code carefully and defensively, etc.

Timing dependencies and their close cousins, race conditions, are another very common reason why programs change their behavior when compiler options are changed.  Tool support for catching races is reasonably good these days.  For timing problems, you’re more or less on your own, especially when programming embedded systems.  Fortunately, most programs have a limited number of timing dependencies and they’re not too hard to spot by hand, usually.

# Compiler Bugs

Most experienced developers have run into some compiler bugs.  However, as mentioned above, their incidence is far lower than other problems.  Compilers tend to be extremely reliable when the following conditions are met:

- the compiler is heavily used
- the optimization options chosen are heavily used
- the code being compiled “looks like” most other code

In other words, if you’re compiling vanilla C code using a released version of gcc at -O or -O2 on an x86 machine, you are unlikely to run into a compiler bug.  Fundamentally, your code is exercising very well-tested code paths in the compiler.  Of course, bugs in vanilla code are always possible.  For example I discovered that the version of gcc that shipped with Ubuntu 8.04 for x86 miscompiled this C function at all optimization levels:

```
int foo (void) {
  signed char x = 1;
  unsigned char y = -1;
  return x > y;
}
```

The correct return value is 0, the Ubuntu compiler returned 1.  The base version of gcc was fine, but the Ubuntu people applied about 5 MB of patches and one of them broke the compiler.

Compiler bugs where the wrong code is emitted at all optimization levels are relatively uncommon, as are bugs when optimizations are disabled.  However, both happen.  In general, bugs can be expected to be more common as the optimizer is asked to be more aggressive.

If any of the conditions above is not met, the incidence of compiler bugs can go up dramatically.  Consider these situations:

- you are using an cross-compiler for a new, obsolete, or unpopular embedded processor
- you are using strange combinations of optimization flags
- you are compiling machine-generated C code or otherwise straying outside the compiler’s comfort zone

In these situations, compiler bugs can become a frequent hazard instead of an occasional novelty.  Workarounds are needed.  Common workarounds include:

- use a different version of the compiler
- turn off optimizations for the module that is being miscompiled
- edit the code to avoid triggering the bug (for example, if 64-bit adds are being miscompiled, you could do the add in assembly or else piece together a 64-bit add from 32-bit operations)

None of these is particularly satisfactory, but most large embedded projects rely on the second and third of these workarounds to avoid compiler bugs.  Once a big embedded system is working and tested, developers never, ever upgrade the compiler.  The devil you know.

Embedded software developers can experience nightmarish compiler bugs for several reasons.  There are many embedded compilers, many embedded targets, and often many versions of each target chip.  There tend to be far fewer developers (even if there are a couple billion Nokia phones, all the software is developed by a few hundred people– compare that with x86-gcc’s user base).  The upshot is that embedded compilers tend to be less well-tested, and some embedded compilers are really, really buggy.  Of course some are extremely good– but the average quality is not great.  Bugs in embedded software may actually matter, as Toyota recently realized.

# Assigning Blame

When attempting to blame the compiler, people often turn to the assembly code too quickly.  It is easy and comforting to look at assembly code because its semantics are so simple and clear, compared to C/C++.  The problem is that doing so diverts attention from where the problem usually lies: in undefined behavior at the source code level.  Let me give an example.  Let’s say that I believe the following code is being miscompiled:

```
int foo (int x) {
  return (x+1) > x;
}

```

My evidence is as follows.  At -O1, gcc-4.3.2 emits good code:

```
$ gcc-4 -O1 -S foo.c -o - -fomit-frame-pointer
 _foo:
     movl    4(%esp), %edx
     leal    1(%edx), %eax
     cmpl    %edx, %eax
     setg    %al
     movzbl  %al, %eax
     ret
```

However, at -O2 the same compiler suddenly believes that the function returns constant true:

```
$ gcc-4 -O2 -S foo.c -o - -fomit-frame-pointer
 _foo:
     movl    $1, %eax
     ret
```

When looking at these two fragments of assembly, it’s easy see a smoking gun and start blaming the compiler.  In fact, the compiler is not wrong here.  Instead, at -O2 gcc is exploiting the undefined nature of signed overflow in C.  In detail, the compiler is performing the following case analysis:

- Case 1: (x+1) â‰¤ INT\_MAX  :  (x+1) > x evaluates to true
- Case 2: (x+1) > INT\_MAX  :  undefined behavior

Since the compiler’s behavior is unconstrained when the program executes an operation with undefined behavior, it can do anything it wants to: crash the program, cause it to send email to your grandmother, or whatever.  In this case, the compiler simply chose the most efficient implementation for Case 2: the same behavior as Case 1, permitting the cases to be collapsed into a single “return 1.”

As a programmer, it’s easy to look at this kind of compiler behavior, where undefined behavior is cunningly exploited to produce faster code, and conclude that the compiler is essentially evil.  This is probably not far from the truth, but this is the worldview promoted by the C and C++ standards.  For people who really dislike it, Java and most assembly languages are refreshingly free of undefined behaviors.

# Reporting a Bug

Once you’re sure you’ve found a compiler bug, you should try to find out if it’s a known bug or not.  If not, please consider reporting it. If you report a serious wrong-code bug in LLVM or GCC, it is extremely likely to be fixed in the next release, saving yourself and others from future frustration.

Creating a good bug report can be challenging.  The GCC developers, for example, happen to be somewhat merciless when it comes to dealing with bogus bug reports (probably for good reason– they must receive a lot of these).  Making matters worse, there is a substantial gray zone of behaviors not covered by the C standard but that are expected of real compilers.  This includes the behavior of extensions such as inline assembly and attributes.  Arguments about bugs in these extensions can be frustrating for everyone.  OS code and embedded systems tend to live in this gray zone a lot.

To show people that you have found a compiler bug, you need to create a bug report that is convincing, reproducible, and actionable.  You need a small test case (2-3 lines is ideal) that is obviously free of undefined behavior, obviously independent of unspecified behavior, and obviously miscompiled. Creating small failure-inducing test cases is hard work.  You also need to provide the expected result, and perhaps give a list of compilers and options that produce that result.  Third, you need to clearly identify the version of the compiler that produces the wrong result, including the target CPU and OS, the options used to build the compiler, etc.  If the compiler is a patched one supplied by an OS vendor (as in the Ubuntu example above), you should say so.

# Creds

So far I’ve tried to present the facts.  This last bit is self-promotion.  For the last few years I’ve run a research project in finding C compiler defects; so far we’ve reported around 190 bugs to various compiler development teams.  Each bug report contained a new way to crash a compiler, or cause it to generate wrong code.  A bit more info is [here](http://www.cs.utah.edu/~regehr/research/) and [here](http://www.cs.utah.edu/~eeide/emsoft08/).  The motivation for this project came from roughly a decade of hands-on experience with compilers for embedded systems.  Between working on TinyOS and teaching embedded systems classes, I saw an appalling number of compiler bugs and finally got sick of it.  How many safety-critical, mission-critical, and security-critical systems are being compiled with buggy tools?  Too many.  In summary: **We’re finding compiler bugs so you don’t have to.**

Here are [our GCC bugs](http://gcc.gnu.org/bugzilla/buglist.cgi?query_format=advanced&short_desc_type=allwordssubstr&short_desc=&known_to_fail_type=allwordssubstr&known_to_work_type=allwordssubstr&long_desc_type=substring&long_desc=&bug_file_loc_type=allwordssubstr&bug_file_loc=&gcchost_type=allwordssubstr&gcchost=&gcctarget_type=allwordssubstr&gcctarget=&gccbuild_type=allwordssubstr&gccbuild=&keywords_type=allwords&keywords=&bug_status=UNCONFIRMED&bug_status=NEW&bug_status=ASSIGNED&bug_status=SUSPENDED&bug_status=WAITING&bug_status=REOPENED&bug_status=RESOLVED&bug_status=VERIFIED&bug_status=CLOSED&resolution=FIXED&resolution=WONTFIX&resolution=WORKSFORME&resolution=MOVED&resolution=---&bug_severity=blocker&bug_severity=critical&bug_severity=major&bug_severity=normal&bug_severity=minor&bug_severity=trivial&bug_severity=enhancement&priority=P1&priority=P2&priority=P3&priority=P4&priority=P5&emailreporter1=1&emailcc1=1&emailtype1=substring&email1=regehr%40cs.utah.edu&emailassigned_to2=1&emailreporter2=1&emailcc2=1&emailtype2=substring&email2=&bugidtype=include&bug_id=&votes=&chfieldfrom=&chfieldto=Now&chfieldvalue=&cmdtype=doit&order=Bug+Number&field0-0-0=noop&type0-0-0=noop&value0-0-0=) and [our LLVM bugs](http://www.llvm.org/bugs/buglist.cgi?query_format=advanced&short_desc_type=allwordssubstr&short_desc=&long_desc_type=allwordssubstr&long_desc=&bug_file_loc_type=allwordssubstr&bug_file_loc=&keywords_type=allwords&keywords=&bug_status=UNCONFIRMED&bug_status=NEW&bug_status=ASSIGNED&bug_status=REOPENED&bug_status=RESOLVED&bug_status=VERIFIED&bug_status=CLOSED&resolution=FIXED&resolution=WONTFIX&resolution=LATER&resolution=REMIND&bug_severity=blocker&bug_severity=critical&bug_severity=major&bug_severity=normal&bug_severity=minor&bug_severity=trivial&bug_severity=enhancement&priority=P1&priority=P2&priority=P3&priority=P4&priority=P5&rep_platform=All&rep_platform=DEC&rep_platform=HP&rep_platform=Macintosh&rep_platform=PC&rep_platform=SGI&rep_platform=Sun&rep_platform=Other&op_sys=All&op_sys=Windows+2000&op_sys=Windows+NT&op_sys=Windows+XP&op_sys=MacOS+X&op_sys=Linux&op_sys=DragonFly+BSD&op_sys=FreeBSD&op_sys=NetBSD&op_sys=OpenBSD&op_sys=AIX&op_sys=AuroraUX&op_sys=HP-UX&op_sys=Solaris&op_sys=other&emailreporter1=1&emailcc1=1&emailtype1=substring&email1=regehr%40cs.utah.edu&emailassigned_to2=1&emailreporter2=1&emailcc2=1&emailtype2=substring&email2=&bugidtype=exclude&bug_id=&chfieldfrom=&chfieldto=Now&chfieldvalue=&cmdtype=doit&order=Bug+Number&field0-0-0=noop&type0-0-0=noop&value0-0-0=).

It’s common for computer programs to display behavior seemingly

unrelated to their source code.  Additionally, programs written in

C/C++ frequently change their behavior when the compiler optimization

options are changed.  This post is a quick guide to figuring out if

the compiler is at fault.

Not Compiler Bugs

It’s best to start out assuming that the code being compiled is

broken, not the compiler.  Close to 200 kinds of “undefined behavior”

are mentioned in the C and C++ standards.  For example, dividing by

zero, shfiting an integer by an amount not less than its bitwidth,

accessing past the end of an array, and updating an object multiple

times between sequence points are all undefined behaviors.  The

compiler is permitted to assume that these never, ever happen, and its

behavior is unconstrained when they do happen.  Additionally, C/C++

leave some important behaviors, such as the order of evaluation of

arguments to a function, “unspecified.”  Unspecified means that some

ordering must be chosen, but that the compiler can choose a different

order every time based on the optimization options, the phase of the

moon, etc.  Undefined and unspecified behavior are at the root of most

suspected compiler bugs.  There are many ways to avoid these: use

tools like valgrind, enable and heed compiler warnings, write code

carefully and defensively, etc.

Timing errors and their close cousins, race conditions, are another

very common reason why programs change their behavior when compiler

options are changed.  Tool support for catching races is reasonably

good these days.  For timing problems, you’re more or less on your

own, especially when programming embedded systems.  Fortunately, most

programs have a limited number of timing dependencies and they’re not

too hard to spot by hand, usually.

Compiler Bugs

Most experienced developers have run some compiler bugs.  However, as

mentioned above, their incidence tends to be far lower than other

problems.  Compilers tend to be extremely reliable when the following

conditions are met:

– the compiler is heavily used

– the optimization options chosen are heavily used

– the code being compiled “looks like” most other code

In other words, if you’re compiling vanilla C code using a released

version of gcc at -O or -O2 on an x86 machine, you are very unlikely

to run into a compiler bug.  Fundamentally, your code is exercising

very well-tested code paths in the compiler.  Of course, bugs are

always possible.  For example I discovered that the version of gcc

that shipped with Ubuntu 8.04 for x86 miscompiled this C function

at all optimization levels:

int foo (void) {

signed char x = 1;

unsigned char y = -1;

return x > y;

}

The correct return value is 0, the Ubuntu compiler returned 1.  The

base version of gcc was fine, but the Ubuntu people applied about 5 MB

of patches and one of them broke the compiler.

Compiler bugs where the wrong code is emitted at all optimization

levels are relatively uncommon, as are bugs when optimizations are

disabled.  However, both happen.  In general, bugs can be expected to

be more common as the optimizer is asked to be more aggressive.

If any of the conditions above is not met, the incidence of compiler

bugs can be expected to go up dramatically.  Consider these

situations:

– you are using an cross-compiler for a new, obsolete, or unpopular

embedded procesor

– you are using strange combinations of optimization flags

– you are compiling machine-generated C code or otherwise straying

outside the compiler’s comfort zone

In these situations, compiler bugs can become a frequent hazard

instead of an occasional novelty.  Workarounds are needed.  Common

workarounds include:

– use a different version of the compiler

– turn off optimizations for the module that is being miscompiled

– edit the code to avoid triggering the bug (for example, if 64-bit

adds are being miscompiled, you could do the add in assembly or

else piece together a 64-bit add from 32-bit operations)

None of these is particularly satisfactory, but most large embedded

projects rely on the second and third of these workarounds to avoid

compiler bugs.  Once a big embedded system is working and tested,

developers never, ever upgrade the compiler.  The devil you know.

Embedded systems are a nightmare zone for compiler bugs for several

reasons.  There are many embedded compilers, many embedded targets,

and often many versions of each target chip.  There tend to be far

fewer developers (even if there are a couple billion Nokia phones, all

the software is developed by a few hundred people– compare that with

x86-gcc’s user base).  The upshot is that embedded compilers tend to

be less well-tested, and some embedded compilers are really, really

buggy.  Of course some are extremely good– but the average quality is

not great.  Bugs in embedded software may actually matter, as Toyota

recently realized.

Assigning Blame

When attempting to blame the compiler, people often turn to the

assembly code too quickly (I hate to say this because when teaching

classes, I spend so much time exhorting students to look at the

assembly).  It is easy and comforting to look at assembly code because

its semantics are so simple and clear, compared to C/C++.  The problem

is that doing so diverts attention from where the problem ususally

lies: in undefined behavior at the source code level.  Let me give an

example.  Let’s say that I believe the following code is being

miscompiled:

int foo (int x) {

return (x+1)>x;

}

My evidence is as follows.  At -O1, gcc-4.3.2 emits good code:

$ gcc-4 -O1 -S foo.c -o – -fomit-frame-pointer

\_foo:

movl    4(%esp), %edx

leal    1(%edx), %eax

cmpl    %edx, %eax

setg    %al

movzbl  %al, %eax

ret

However, at -O2 the same compiler falsely believes that the

function returns constant true:

$ gcc-4 -O2 -S foo.c -o – -fomit-frame-pointer

\_foo:

movl    $1, %eax

ret

When looking at these two fragments of assembly, it’s easy see a

smoking gun and start blaming the compiler.  In fact, the compiler is

not wrong here.  Instead, at -O2 gcc is exploiting the undefined

nature of signed overflow in C.  In detail, the compiler is performing

the following case analysis:

Case 1: (x+1) <= INT\_MAX : (x+1)>x evaluates to true

Case 2: (x+1) > INT\_MAX : undefined behavior

Since the compiler’s behavior is unconstrainted when the program

executes an operation with undefined behavior, it can do anything it

wants to, including crash the program, cause it to send email to your

grandmother, or whatever.  In this case, the compiler elects to do the

most efficient thing: the same behavior as Case 1.  As a programmer,

it’s easy to look at this kind of compiler behavior, where undefined

behavior is cunningly exploited to produce faster code, and conclude

that the compiler is essentially evil.  This is probably not far form

the truth, but this is the worldview promoted by the C standard.  For

people who really dislike it, Java and most assembly languages are

refreshingly free of undefined behaviors.

Reporting a Bug

Once you’re sure you’ve found a compiler bug, you should try to find

out if it’s a known bug or not.  If not, please consider reporting it.

If you report a serious wrong-code bug in LLVM or GCC, it is extremely

likely to be fixed in the next release, saving yourself and others

from future frustration.

Creating a good bug report can be challenging.  The GCC developers,

for example, happen to be somewhat merciless when it comes to dealing

with bogus bug reports (probably for good reason– they must receive a

lot of these).  Making matters worse, there is a substantial gray zone

of behaviors not covered by the C standard but that are expected of

real compilers.  This includes the behavior of extensions such as

inline assembly and attributes.  Arguments about bugs in these

extensions can be frustrating for everyone.  Embedded systems tend to

live in this gray zone a lot.

To show people that you have found a compiler bug, you need to create

a bug report that is convincing, reproducible, and actionable.  First,

you need a small test case (2-3 lines is ideal) that is obviously free

of undefined behavior and independent of unspecified behavior.

Second, you need to provide the expected result, and perhaps give a

list of compilers and options that produce that result.  Third, you

need to identify the version of the compiler that produces the wrong

result, including the target CPU and OS, the options used to build the

compiler, etc.  If the compiler is a patched one supplied by an OS

vendor (as in the Ubuntu example above), you should say so.
