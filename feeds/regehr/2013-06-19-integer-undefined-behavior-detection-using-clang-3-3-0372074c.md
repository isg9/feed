---
title: Integer Undefined Behavior Detection using Clang 3.3
url: https://blog.regehr.org/archives/963
published: "2013-06-19T22:36:02Z"
feed: regehr
guid: http://blog.regehr.org/?p=963
---

# Integer Undefined Behavior Detection using Clang 3.3

Undefined behaviors in C/C++ are harmful to developers:

- There are many kinds of undefined behavior

- They can be hard to understand

- Their effect changes depending on which compiler version you use, which compiler options you use, and they get worse every time an optimizer gets smarter

- Plenty of them aren’t reliably detected by any tool that I know of

Until these languages die, which isn’t going to happen anytime soon, our best defense against undefined behaviors is to write better checking tools. Recently, Clang has started to accumulate a nice collection of such tools, many of which can be enabled using the compiler flag **`-fsanitize=undefined`**. [The Clang manual](http://clang.llvm.org/docs/UsersManual.html#controlling-code-generation) has more details.

Our modest contribution has been a collection of checks for integer undefined behaviors like signed overflow and shift-past-bitwidth. These checks have been part of LLVM for a while and finally now they are part of the 3.3 release which comes in variety of [convenient pre-compiled packages](http://llvm.org/releases/download.html#3.3).

To find integer undefined behaviors in a code base that you are about, there are three steps:

1. Install Clang/LLVM 3.3 from a binary package or from source and make sure that clang and clang++ are in your PATH. If compiling from source, you will need to build compiler-rt. [Full instructions are here](http://clang.llvm.org/get_started.html).

2. Build your code base using clang or clang++ and a flag such as **`-fsanitize=undefined`**

3. Test the compiled code as thoroughly as possible; if any sanitizer output appears then you have probably found one or more bugs. The lines you care about will contain the string “runtime error”.

Let’s go through a quick example. I did this on a Linux machine but it should be more or less the same on other platforms. Grab the latest stable version of Perl, untar it, and run its configure script:

> **`wget http://www.cpan.org/src/5.0/perl-5.18.0.tar.gz** **tar xvf perl-5.18.0.tar.gz** **cd perl-5.18.0/** **./Configure** **`**

When the configure script asks for a C compiler, respond with `clang -fsanitize=undefined`. Then build Perl, run its test suite, and look for problems:

> **`make -j4** **make test > make.out 2>&1** **grep 'runtime error' make.out | sort | uniq** **`**

At this point you should see several hundred lines of undefined behavior errors. [Here’s the full output](https://blog.regehr.org/extra_files/perl-test-output.txt).

Since modern C compilers actually exploit undefined integer behaviors in order to generate code that you didn’t expect or want, this whole exercise is probably worth doing for codes whose correctness you care about.
