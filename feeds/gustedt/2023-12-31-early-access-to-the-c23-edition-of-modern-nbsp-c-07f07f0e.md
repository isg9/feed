---
title: Early access to the C23 edition of Modern&nbsp;C
url: https://gustedt.wordpress.com/2023/12/31/early-access-to-the-c23-edition-of-modern-c/
published: "2023-12-31T20:56:00Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=4016
---

# Early access to the C23 edition of Modern&nbsp;C

![](https://gustedt.wordpress.com/wp-content/uploads/2024/01/dotd_newmeap_gustedt21.png?w=600)

Manning’s early access program (MEAP) for the new edition is now open

at

[https://www.manning.com/books/modern-c-third-edition](https://www.manning.com/books/modern-c-third-edition)

> There is a special code `mlgustedt2` to get 45% off of the official price. This is currently limited until Jan 31.

The previous edition already has been largely successful and is considered by some as one of the reference books on C. This new edition has been the occasion to overhaul the presentation in many places, but its main purpose is the update to the new C standard, C23. The goal is to publish this new edition of Modern C at the same time as the new C standard goes through the procedure of ISO publication and as new releases of major compilers will implement all the new features that it brings.

Among the most noticeable changes and additions that we handle are those for integers: there are new bit-precise types coined `_BitInt(N)`, new C library headers `(for arithmetic with overflow check) and` (for bit manipulation), possibilities for 128 bit types on modern architectures, and substantial improvements for enumeration types. Other new concepts in C23 include a `nullptr` constant and its underlying type, syntactic annotation with attributes, more tools for type generic programming such as type inference with `auto` and `typeof`, default initialization with `{}`, even for variable length arrays, and `constexpr` for name constants of any type. Furthermore, new material has been added, discussing compound expressions and lambdas, so-called “internationalization”, a comprehensive approach for program failure.

During MEAP we will also add an appendix and continue work on a temporary include header for an easy transition to C23 on existing platforms, that will allow you to start off with C23 right away.

We also encourage you to post any questions or comments you have about the content in the [liveBook Discussion forum](https://livebook.manning.com/forum?product=gustedt2&page=1). We appreciate knowing where we can make improvements and increase your understanding of the material.

Note that the “about this book” section in MEAP is a bit hidden under the front page. I think that reading this even before going in details is important so I repeat some of this information here.

## About this book

The C programming language has been around since the early seventies. Since then, C has been used in an incredible number of applications. Programs and systems written in C are all around us: in personal computers, phones, cameras, set-top boxes, refrigerators, cars, mainframes, satellites … basically in any modern device that has a programmable interface.

In contrast to the ubiquitous presence of C programs and systems, good knowledge of and about C is much more scarce. Even experienced C programmers often appear to be stuck in some degree of self-inflicted ignorance about the modern evolution of the C language. A likely reason for this is that C is seen as an “easy to learn” language, allowing a programmer with little experience to quickly write or copy snippets of code that at least appear to do what it’s supposed to. In a way, C fails to motivate its users to climb to higher levels of knowledge.

This book is intended to change that general attitude, so it is organized in *levels* that reflect familiarity with the C language and programming in general. This structure may go against some habits of the book’s readers; in particular, it splits some difficult subjects (such as pointers) across levels in order to not swamp readers too early with the wrong information. We’ll explain the book’s organization in more detail shortly.

Generally, although many universally applicable ideas will be presented, that would also be valid for other programming languages (such as Java, Python, Ruby, C# or C++) the book primarily addresses concepts and practices that are unique to C or are of particular value when programming in the C language.

### C revisions

As the title of this book suggests, today’s C is not the same language as the one originally designed by its creator. Right from the start, C has been in a continuous process of adjustment and improvement. Usually, early C is referred to as K&R C (Kernighan and Ritchie C) after the first book that made the language popular. Since then, it has undergone an important standardization and extension process, now driven by ISO, the International Standards Organization. This led to the publication of a series of C standards in 1989, 1999, 2011, 2018 and 2024, commonly referred to as C89, C99, C11, and C17. The C standards committee puts a lot of effort into guaranteeing backward compatibility such that code written for earlier revisions of the language, say C11, should compile to a semantically equivalent executable with a compiler that implements a newer revision. Unfortunately, this backward compatibility has had the unwanted side effect of not motivating projects that could benefit greatly from the new features to update their code base. To emphasize on this progress of revisions, we indicate for newer features which standard revision introduced them.

### This edition

This edition presents a considerable rework in view of the latest revision, C23, of the C standard. A lot of new material has been added and many expositions have been straightened out to reflect the new capabilities of the C programming language. So, in this book we will mainly refer to C23, but at the time of this writing compilers don’t yet implement this standard completely. If you want to compile the examples in this book, you will need at least a compiler that implements most of C17. For the novelties that introduces we provide a compatibility header and discuss how to possibly generate a suitable C compiler and C library platform on POSIX systems as a fallback in an annex. Beware, that this is not meant as a permanent tool, but only a crutch while platforms adopt.

### C and C++

Programming has become a very important cultural and economic activity, and C remains an important element in the programming world. As in all human activities, progress in C is driven by many factors: corporate or individual interest, politics, beauty, logic, luck, ignorance, selfishness, ego, sectarianism, … (add your primary motivation here). Thus the development of C has not been and cannot be ideal. It has flaws and artifacts that can only be understood with their historical and societal context.

An important part of the context in which C developed was the early appearance of its sister language, C++. One common misconception is that C++ evolved from C by adding its particular features. Although this is historically correct (C++ evolved from a very early C), it is not particularly relevant today. In fact, C and C++ separated from a common ancestor more than 30 years ago and have evolved separately ever since. But this evolution of the two languages has not taken place in isolation; they have exchanged and adopted each other’s concepts over the years. Some new features, such as the addition of atomics and threads, have been designed in close collaboration between the C and C++ standard committees.

Nevertheless, many differences remain, and generally all that is said in this book is about C, not C++. Many code examples that are given will not even compile with a C++ compiler. So we should not mix sources of both languages.

\> C and C++ are different: don’t mix them, and don’t mix them up.

### Requirements

To be able to profit from this book, you need to fulfill some minimal requirements. If you are uncertain about any of these, please obtain or learn them first; otherwise, you might waste a lot of time.

First, you can’t learn a programming language without practicing it, so you *must* have a decent programming environment at your disposal (usually on a PC or laptop), and you *must* master it to some extent. This environment can be integrated (an IDE) or a collection of separate utilities. Platforms vary widely in what they offer, so it is difficult to advise on specifics. On Unix-like environments such as Linux and Apple’s macOS, you will find editors such as `emacs` and `vim`, and compilers such as `c99`, `c17` `gcc`, and `clang`.

If you have never programmed before, this book will be tough. Knowing some of the following will help: Basic, C (historical revisions), C++, Fortran, R, bash, JavaScript, Java, MATLAB, Perl, Python, Scilab, and so on. But perhaps you have had some other programming experience, maybe even without noticing. Many technical specifications actually come in some sort of specialized language that can be helpful as an analogy: for example, *HTML* for web pages and *LaTeX* for document formatting.

### Source code

Many of the programming code snippets that are presented in this book are publicly available, see . This allows you to view them in context and to compile them and try them out. The archive also contains a `Makefile` with a description of the components that are needed to compile these files. It is centered around Linux or, more generally, POSIX systems, but it may also help you to find out what you need when you are on a different system.

### Exercises and challenges

Throughout this book, you’ll see exercises that are meant to get you thinking about the concepts being discussed. These are probably best done directly along with your reading. Then there is another category called “challenges.” These are generally more demanding. You will need to do some research to even understand what they are about, and the solutions will not come all by themselves: they will require effort. They will take more time, sometimes hours or, depending on your degree of satisfaction with your work, even days. The subjects covered in these challenges are the fruit of my own personal bias toward “interesting questions” from my personal experience. If you have other problems or projects in your studies or your work that cover the same ground, they should do equally well. The important aspect is to train yourself by first searching for help and ideas elsewhere, and then to get your hands dirty and get things done. You will only learn to swim if you jump into the water.

### Organization

This book is organized in *levels*, numbered from 0 to 3. The starting level 0, named “Encounter”, will summarize the very basics of programming with C. Its principal role is to remind you of the main concepts we have mentioned and familiarize you with the special vocabulary and viewpoints that C applies. By the end of it, even if you don’t have much experience in programming with C, you should be able to understand the structure of simple C programs and start writing your own.

The “Acquaintance” level 1 details most principal concepts and features such as control structures, data types, operators, and functions. It should give you a deeper understanding of the things that are going on when you run your programs. This knowledge should be sufficient for an introductory course in algorithms and other work at that level, with the notable caveat that pointers are not yet fully introduced.

The “Cognition” level 2 goes to the heart of the C language. It fully explains pointers, familiarizes you with C’s memory model, and allows you to understand most of C’s library interface. Completing this level should enable you to write C code professionally; it therefore begins with an essential discussion about the writing and organization of C programs. I personally would expect anybody who graduated from an engineering school with a major related to computer science or programming in C to master this level. Don’t be satisfied with less.

The “Experience” level 3 then goes into detail about specific topics, such as performance, reentrancy, atomicity, threads, and type-generic programming. These are probably best discovered as you go, which is when you encounter them in the real world. Nevertheless, as a whole, they are necessary to round off the discussion and to provide you with full expertise in C. Anybody with some years of professional programming in C or who heads a software project that uses C as its main programming language should master this level.
