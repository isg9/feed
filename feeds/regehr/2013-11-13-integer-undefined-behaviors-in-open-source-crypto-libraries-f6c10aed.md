---
title: Integer Undefined Behaviors in Open Source Crypto Libraries
url: https://blog.regehr.org/archives/1054
published: "2013-11-13T16:52:44Z"
feed: regehr
guid: http://blog.regehr.org/?p=1054
---

# Integer Undefined Behaviors in Open Source Crypto Libraries

Crypto libraries should be beyond reproach. This post investigates integer-related undefined behaviors found in crypto code written in C/C++. Undefined behavior (UB) is bad because according to the standards, it destroys the meaning of any program that executes it. In practice, over the last decade compilers have become pretty smart about exploiting integer undefined behaviors to generate fast but surprising object code. Plenty of [examples are here](https://blog.regehr.org/archives/759).

This post was motivated by a [discussion on the C++ standards committee’s mailing list for undefined behavior](http://www.open-std.org/pipermail/ub/2013-October/000272.html) where I proposed making signed left-shift work just like unsigned left-shift. In contrast, in C99, C11, and C++11, it is illegal to shift a 1 bit into, out of, or through the sign bit. Many developers are unaware of this restriction. This seemed to me like a pretty safe proposal since it isn’t clear that any existing compiler implements anything other than two’s complement semantics for signed left shifts in the first place — so the change would essentially just mandate the behavior that is already implemented. Chandler Carruth from Google disagreed with this, stating that not only do the LLVM people believe that these UBs can result in useful optimizations but also that developers were appreciative of the bug reports that come from statically and dynamically detecting signed left-shift problems. Of course, if the UBs disappear due to a tighter language standard, then the capability for error detection disappears too. Chandler’s messages contain some good details so I’ll just link to them: [1](http://www.open-std.org/pipermail/ub/2013-October/000273.html), [2](http://www.open-std.org/pipermail/ub/2013-October/000275.html), [3](http://www.open-std.org/pipermail/ub/2013-October/000278.html). Based on that conversation, my goal here is to try to understand where these integer undefined behaviors come from and whether they correspond to “real bugs” or not.

To make matters just a little more complicated, there’s a [C++ defect report](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3367.html#1457) stating that it should be permissible to shift a 1 into the sign bit, but not out of or through it. The defect report has no effect upon any of the standards until a TC is released, but it does indicate that this particular issue is considered to have been fixed by the committee. There is no corresponding defect report for C, as far as I know.

Let’s get on with the testing. The methodology used here was, for each of a number of open source crypto libraries that include a substantial amount of C/C++ code:

1. Grab a recent version
2. Build it using Clang with the `-fsanitize=integer` flag on an x86-64 machine running Linux
3. Run the library’s built-in test suite
4. Examine the resulting undefined behaviors

Note that if you do this kind of work yourself, you’re most likely better off using the `-fsanitize=undefined` flag because it doesn’t warn about (well-defined) unsigned overflows and it warns about many undefined behaviors not relating to integers.

# [BeeCrypt](http://beecrypt.sourceforge.net/) 4.2.1

This library executes no integer undefined behaviors while running its test suite. Nice!

# [Botan](http://botan.randombit.net/) 1.11.4

Two of Botan’s rotate functions, found in `rotate.h`, take an unsigned int and shift it by 32 places. The code looks like this:

```
template inline T rotate_left(T input, size_t rot)
{
  return static_cast((input << rot) | (input >> (8*sizeof(T)-rot)));;
}

```

The problem occurs not when rotating by 32, but rather when rotating by 0. Rotate by 0 is undefined because it causes the second shift operator to shift by 32 – 0. The fix (applied by the code’s author already) is to add a check for rotate-by-0 and in that case return the unmodified input. This is a little bit annoying because in the expected case where the rotate code is translated into a rotate instruction ( [both GCC and Clang will do this](https://blog.regehr.org/archives/1055)) the test just adds overhead to the generated code since the machine instruction is perfectly well defined for rotating by zero. The compiler could go ahead and remove the test and exit, but neither has been taught to do so.

Shifting by bitwidth (or more) is a serious kind of undefined behavior that can lead to portability problems across compilers, platforms, and even optimization levels. On the version of Clang from Xcode 5, a little shift-past-bitwidth program that I wrote has different results each time it is run.

# [HELib](https://github.com/shaih/HElib/) from Github on 10/26/13

This library executes numerous integer undefined behaviors, but all of them occur inside code from [NTL](http://www.shoup.net/ntl/). However, NTL can be configured using the `NTL_CLEAN_INT` option, which makes the undefined behaviors go away. In other words, the NTL author (who I had a short discussion with) is aware of the undefined behaviors but prefers the version that executes them since it results in faster code and compilers do not currently (as far as we know) exploit the undefined behaviors to break NTL. My own take is that if NTL were distributed as a library that could be solidly unit tested, this might not be a bad bet to make. However, a lot of NTL code gets included in applications via header files, giving the compiler plenty of opportunity to do tricky things. Personally, I would not trust a modern compiler to fail to exploit an undefined integer overflow when it can see all of the code and propagate constants through it.

I didn’t report anything to the HELib people since the UBs aren’t in their code, and their code appears to be intended for research purposes anyhow.

# [LibTomCrypt](http://libtom.org/?page=features&newsitems=5&whatfile=crypt) 1.17

This library contains an interesting undefined behavior in a line of code (anubis.c:934) that shifts a value left by 24:

```
    for (i = 0, pos = 0; i < N; i++, pos += 4) {
      kappa[i] =
         (key[pos    ] << 24) ^
         (key[pos + 1] << 16) ^
         (key[pos + 2] <<  8) ^
         (key[pos + 3]      );
    }

```

No undefined behaviors can occur when shifting an unsigned value left by 24. However, in this code `key` is a pointer to unsigned char. The char value is promoted to int before being shifted; if the value in the int is >127, a 1 bit will end up being shifted into the sign bit, which is undefined. This example nicely illustrates how the integer promotion rules in C/C++ can have surprising consequences. There’s another instance of the same problem in the same file at line 1051. I reported these issues but haven’t heard back from the developers yet.

# [Libgcrypt](http://www.gnu.org/software/libgcrypt/) 1.5.3

Ok, at this point things start to get a bit repetitive so I’ll start leaving out the details. This version of Libgcrypt contains:

- 4 locations in blowfish.c, 8 locations in cast5.c, 3 locations in des.c, and 8 locations in twofish.c where an unsigned char is promoted to int and then left-shifted by 24 places, resulting in undefined behavior
- 2 locations in cast5.c where a 32-bit integer is shifted by 32 places, again as part of a rotate operation

Keep in mind that the former problems are probably benign for now, whereas the latter ones should definitely be fixed. These have been reported (see [developer’s response here](https://bugs.g10code.com/gnupg/issue1567)).

# [Crypto++](http://www.cryptopp.com/) 5.6.2

3 locations in misc.h where a 32-bit unsigned value is shifted by 32 places, again in rotate functions. Reported.

# [Sodium](https://github.com/jedisct1/libsodium) 0.4.5

3 locations in aes256-ctr.c where an unsigned char is promoted to signed int and shifted left by 24. Reported, and fixed within hours.

# [Libmcrypt](http://mcrypt.sourceforge.net/) 2.5.8

Summary:

- 2 locations in cast-128.c where the rotate operation leads to a shift by 32 places of a 32-bit integer

- 26 locations in cast-256.c with very large shift exponents up to almost 232; the code is heavily macroized so I didn’t dig in too deeply

- 2 signed multiplication overflows in enigma.c

- 1 location in des.c, 1 in gost.c, and 2 in loki97.c where a 1 is left-shifted into the sign bit

- 1 location in tripledes.c where a 1 is left-shifted out of the sign bit

I didn’t bother reporting since libmcrypt was last released in 2007 and it has a number of long-open, serious-looking bugs. Hopefully most people have phased out their use of this library, though I see that it is still provided as a package in Ubuntu 13.10.

# [Nettle](http://www.lysator.liu.se/~nisse/nettle/) 2.7.1

1 location in blowfish.c and 1 in twofish.c where an unsigned char is promoted to signed int and the left-shifted by 24. 2 locations in cast128.c where a 32-bit value is shifted by 32 places in a rotate function. Reported and the rotate has been fixed already.

# [OpenSSL](http://www.openssl.org/) SNAP-20131112

At a\_int.c:397 and obj\_dat.c:143 there are undefined signed left-shifts. These — unlike most of what we’ve seen so far — are the result of variables that were declared as signed, as opposed to being the result of implicit conversions. These shifts should be done more carefully, or should be done using unsigned operations.

In c\_enc.c there are 20 sites that shift a 32-bit variable by 32 places, all resulting from a macro called E\_CAST.

gost89.c has 8 locations where a 1 is left-shifted into the sign bit, all are due to promotion of an unsigned char to int. Same thing happens at 2 locations in gost\_crypt.c.

These have been reported.

# The Bad News

The bad news, obviously, is that nearly all of the crypto libraries I tested execute integer undefined behaviors. There appear to be several root causes:

1. C’s weird integer promotion rules practically guarantee that signed types will be introduced into computations that start out with only unsigned types. Signed math is hard because its operators have much stricter rules to follow if we are to avoid undefined behavior. Unsightly explicit casts to unsigned are the solution.

2. Some developers continue to purposefully use signed math under the faulty assumption that the C/C++ languages are built around two’s complement arithmetic

3. Reasoning about function preconditions is hard even for experienced developers. In my opinion, some of these libraries could have used a lot more assertions to go along with their (generally perfectly adequate) test suites.

# The Good News

1. Most of the undefined behaviors seen here stem from a few simple patterns, most notably “rotate by zero” and “unsigned char is promoted to signed int.” Crypto developers can and should learn to recognize and deal correctly with these. In general, the problems seen here are not difficult to fix.

2. The developers who have responded to my bug reports seem very on the ball and generally happy that people are vetting their software.

3. Clang’s undefined behavior sanitizer revealed all of these problems trivially. Use it, folks.

# Next Steps

The undefined behaviors reported here are a subset of all undefined behaviors executed by these libraries, in three senses:

1. I only checked for integer undefined behaviors, whereas Clang’s undefined behavior sanitizers can find other problems such as memory safety bugs.
2. Even Clang’s full set of undefined behavior sanitizers checks only a subset of all undefined behaviors in C/C++. More work on checkers is needed if we are to keep using C/C++ in security-critical roles.
3. Even if we had a checker for all undefined behaviors, we would still be limited by the quality of each library’s builtin test suite. Static analysis is one solution and better testing is another.

If our security relies on crypto libraries operating correctly, it would not be a bad idea for someone to spend some time and energy vetting crypto code using a tool like [Frama-C](http://frama-c.com/). Crypto code is nice and mathy: a perfect target for formal methods.

Finally, if I’ve missed your favorite crypto library, please let me know and I’ll take a look. I am interested only in open source code which contains a substantial amount of C/C++ and which can be built on Linux or OS X.
