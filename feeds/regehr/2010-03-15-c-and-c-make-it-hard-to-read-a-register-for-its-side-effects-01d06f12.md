---
title: C and C++ Make It Hard to Read a Register for Its Side Effects
url: https://blog.regehr.org/archives/41
published: "2010-03-15T19:15:09Z"
feed: regehr
guid: http://blog.regehr.org/?p=41
---

# C and C++ Make It Hard to Read a Register for Its Side Effects

*\[ This post was co-written with Nigel Jones, who maintains an excellent embedded blog [Stack Overflow](http://embeddedgurus.com/stack-overflow/).  Nigel and I share an interest in volatile pitfalls in embedded C/C++ and this post resulted from an email discussion we had.  Since we both have blogs, we decided to both post it.   However, since comments are not enabled here, [all discussion should take place at Nigel’s post](http://embeddedgurus.com/stack-overflow/2010/03/reading-a-register-for-its-side-effects-in-c-and-c/). \]*

Once in awhile one finds oneself having to read a device register, but without needing nor caring what the value of the register is. A typical scenario is as follows. You have written some sort of asynchronous communications driver. The driver is set up to generate an interrupt upon receipt of a character. In the ISR, the code first of all examines a status register to see if the character has been received correctly (e.g. no framing, parity or overrun errors). If an error has occurred, what should the code do? Well, in just about every system we have worked on, it is necessary to read the register that contains the received character — even though the character is useless. If you don’t perform the read, then you will almost certainly get an overrun error on the next character. Thus you find yourself in the position of having to read a register even though its value is useless. The question then becomes, how does one do this in C? In the following examples, assume that SBUF is the register holding the data to be discarded and that SBUF is understood to be *volatile*. The exact semantics of the declaration of SBUF vary from compiler to compiler.

If you are programming in C and if your compiler correctly supports the volatile qualifier, then this simple code suffices:

```
void cload_reg1 (void)
{
  SBUF;
}
```

This certainly looks a little strange, but it is completely legal C and should generate the requisite read, and nothing more. For example, at the -Os optimization level, the MSP430 port of GCC gives this code:

```
cload_reg1:
   mov &SBUF, r15
   ret
```

Unfortunately, there are two practical problems with this C code. First, quite a few C compilers incorrectly translate this code, although the C standard gives it an unambiguous meaning. We tested the code on a variety of general-purpose and embedded compilers, and present the results below. These results are a little depressing.

The second problem is even scarier. The problem is that the C++ standard is not 100% clear about what the code above means. On one hand, the standard says this:

> In general, the semantics of volatile are intended to be the same in C++ as they are in C.

A number of C++ compilers, including GCC and LLVM, generate the same code for cload\_reg1() when compiling in C++ mode as they do in C mode. On the other hand, several high-quality C++ compilers, such as those from ARM, Intel, and IAR, turn the function cload\_reg1() into object code that does nothing. We discussed this issue with people from the compiler groups at Intel and IAR, and both gave essentially the same response. Here we quote (with permission) from the Intel folks:

> The operation that turns into a load instruction in the executable code is what the C++ standard calls the lvalue-to-rvalue conversion; it converts an lvalue (which identifies an object, which resides in memory and has an address) into an rvalue (or just value; something whose address can’t be taken and can be in a register). The C++ standard is very clear and explicit about where the lvalue-to-rvalue conversion happens. Basically, it happens for most operands of most operators – but of course not for the left operand of assignment, or the operand of unary ampersand, for example. The top-level expression of an expression statement, which is of course not the operand of any operator, is not a context where the lvalue-to-rvalue conversion happens.
>
> In the C standard, the situation is somewhat different. The C standard has a list of the contexts where the lvalue-to-rvalue conversion doesn’t happen, and that list doesn’t include appearing as the expression in an expression-statement.
>
> So we’re doing exactly what the various standards say to do. It’s not a matter of the C++ standard allowing the volatile reference to be optimized away; in C++, the standard requires that it not happen in the first place.

We think the last sentence sums it up beautifully. How many readers were aware that the semantics for the volatile qualifier are significantly different between C and C++? The additional implication is that as shown below GCC, the Microsoft compiler, and Open64, when compiling C++ code, are in error.

We asked about this on the GCC mailing list and received only one response which was basically “Why should we change the semantics, since this will break working code?” This is a fair point. Frankly speaking, the semantics of volatile in C are a bit of mess and C++ makes the situation much worse by permitting reasonable people to interpret it in two totally different ways.

# Experimental Results

To test C and C++ compilers, we compiled the following two functions to object code at a reasonably high level of optimization:

```
extern volatile unsigned char foo;
```

```
void cload_reg1 (void)
{
  foo;
}
```

```
void cload_reg2 (void)
{
  volatile unsigned char sink;
  sink = foo;
}
```

For embedded compilers that have built-in support for accessing hardware registers, we tested two additional functions where as above, SBUF is understood to be a hardware register defined by the semantics of the compiler under test:

```
void cload_reg3 (void)
{
  SBUF;
}

void cload_reg4 (void)
{
  volatile unsigned char sink;
  sink = SBUF;
}
```

The results were as follows.

## GCC

We tested version 4.4.1, hosted on x86 Linux and also targeting x86 Linux, using optimization level -Os. The C compiler loads from foo in both cload\_reg1() and cload\_reg2() . No warnings are generated. The C++ compiler shows the same behavior as the C compiler.

## Intel Compiler

We tested icc version 11.1, hosted on x86 Linux and also targeting x86 Linux, using optimization level -Os. The C compiler emits code loading from foo for both cload\_reg1() and cload\_reg2(), without giving any warnings. The C++ compiler emits a warning “expression has no effect” for cload\_reg1() and this function does not load from foo. cload\_reg2() does load from foo and gives no warnings.

## Sun Compiler

We tested suncc version 5.10, hosted on x86 Linux and also targeting x86 Linux, using optimization level -O. The C compiler does not load from foo in cload\_reg1(), nor does it emit any warning. It does load from foo in cload\_reg2(). The C++ compiler has the same behavior as the C compiler.

## x86-Open64

We tested opencc version 4.2.3, hosted on x86 Linux and also targeting x86 Linux, using optimization level -Os. The C compiler does not load from foo in cload\_reg1(), nor does it emit any warning. It does load from foo in cload\_reg2(). The C++ compiler has the same behavior as the C compiler.

## LLVM / Clang

We tested subversion rev 98508, which is between versions 2.6 and 2.7, hosted on x86 Linux and also targeting x86 Linux, using optimization level -Os. The C compiler loads from foo in both cload\_reg1() and cload\_reg2() . A warning about unused value is generated for cload\_reg1(). The C++ compiler shows the same behavior as the C compiler.

## CrossWorks for MSP430

We tested version 2.0.8.2009062500.4974, hosted on x86 Linux, using optimization level -O. This compiler supports only C. foo was not loaded in cload\_reg1(), but it was loaded in cload\_reg2().

## IAR for AVR

We tested version 5.30.6.50191, hosted on Windows XP, using maximum speed optimization. The C compiler performed the load in all four cases. The C++ compiler did not perform the load for cload\_reg1() or cload\_reg3(), but did for cload\_reg2() and cload\_reg4().

## Keil 8051

We tested version 8.01, hosted on Windows XP, using optimization level 8, configured to favor speed. The Keil compiler failed to generate the required load in cload\_reg1() (but did give at least give a warning), yet did perform the load in all other cases including cload\_reg3() suggesting that for the Keil compiler, its IO register (SFR) semantics are treated differently to volatile variable semantics.

## HI-TECH for PIC16

We tested version 9.70, hosted on Windows XP, using Global optimization level 9, configured to favor speed. This was very interesting in that the results were almost a mirror image to the Keil compiler. In this case the load was performed in all cases except cload\_reg3(). Thus the HI-TECH semantics for IO registers and volatile variables also appears to be different – just the opposite to Keil! No warnings was generated by the Hi-TECH compiler when it failed to generate code.

## Microchip Compiler for PIC18

We tested version 3.35, hosted on Windows XP, using full optimization level. This rounded out the group of embedded compilers quite nicely in that it didn’t perform the load in either cload\_reg1() or cload\_reg3() – but did in the rest. It also failed to warn about the statements having no effect. This was the worst performing of all the compilers we tested.

# Summary

The level of non-conformance with the C compilers, together with the genuine uncertainty as to what the C++ compilers should do, provides a real quandary. If you need the most efficient code possible, then you have no option other than to investigate what your compiler does. If you are looking for a generally reliable and portable solution, then the methodology in cload\_reg2() is probably your best bet. However it would be just that: a bet. Naturally, we (and the other readers of this blog) would be very interested to hear what your compiler does. So if you have a few minutes, please run the sample code through your compiler and let us know the results.

# Acknowledgments

We’d like to thank Hans Boehm at HP, Arch Robison at Intel, and the compiler groups at both Intel and IAR for their valuable feedback that helped us construct this post. Any mistakes are, of course, ours.
