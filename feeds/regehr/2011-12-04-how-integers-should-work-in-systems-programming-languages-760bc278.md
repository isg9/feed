---
title: How Integers Should Work (In Systems Programming Languages)
url: https://blog.regehr.org/archives/642
published: "2011-12-04T15:13:20Z"
feed: regehr
guid: http://blog.regehr.org/?p=642
---

# How Integers Should Work (In Systems Programming Languages)

My [last post](https://blog.regehr.org/archives/641) outlined some of the possibilities for integer semantics in programming languages, and asked which option was correct. This post contains my answers. Just to be clear: I want practical solutions, but I’m not very interested by historical issues or backwards compatibility with any existing language, and particularly not with C and C++.

We’ll start with:

> **Premise 1: Operations on default integer types return the mathematically correct result or else trap.**

This is the same premise that [as-if infinitely ranged](http://www.sei.cmu.edu/library/abstracts/reports/10tn008.cfm) begins with, but we’ll perhaps end up with some different consequences. The things I like about this premise are:

- It appears to be consistent with the principle of least astonishment.
- Since it is somewhat harsh — ruling out many semantics such as 2’s complement wraparound, undefined overflow, saturation, etc. — it gives a manageably constrained design space.
- I just don’t like two’s complement wraparound. It’s never what I want.

Of course there are specialized domains like DSP that want saturation and crypto codes that want wraparound; there’s nothing wrong with having special datatypes to support those semantics. However, we want the type system to make it difficult for those specialized integers to flow into allocation functions, array indices, etc. That sort of thing is the root of a lot of serious bugs in C/C++ applications.

Despite the constraints of premise 1, we have considerable design freedom in:

- When to trap vs. giving a correct result?
- Do traps occur early (as soon as an overflow happens) or late (when an overflowed value is used inappropriately)?

For high-level languages, and scripting languages in particular, traps should only occur due to resource exhaustion or when an operation has no sensible result, such as divide by zero. In other words, these language should provide arbitrary precision integers by default.

The tricky case, then, is systems languages — those used for implementing operating systems, virtual machines, embedded systems, etc. It is in these systems where overflows are potentially the most harmful, and also they cannot be finessed by punting to a bignum library. This leads to:

> **Premise 2: The default integer types in systems languages are fixed size.**

In other words, overflow traps will occur. These will raise exceptions that terminate the program if uncaught. The rationale for fixed-size integers is that allocating more storage on demand has unacceptable consequences for time and memory predictability. Additionally, the possibility that allocation may happen at nearly arbitrary program points is extraordinarily inconvenient when implementing the lowest levels of a system — interrupt handlers, locks, schedulers, and similar.

Side note: The Go language provides arbitrary precision math at compile time. This is a good thing.

Overflows in intermediate results are not fun to deal with. For example, check out the first few comments on a [PHP bug report](https://bugs.php.net/bug.php?id=52550) I filed a while ago. The code under discussion is:

> ```
> result = result * 10 + cursor - '0';
> ```

It occurs in an atoi() kind of function. This function — like many atoi functions (if you’ve written one in C/C++, please go run it under [IOC](http://embed.cs.utah.edu/ioc/)) — executes an integer overflow while parsing 2147483647. The overflow is a bit subtle; the programmer seems to have wanted to write this code:

> ```
> result * 10 + (cursor - '0')
> ```

However, the code actually evaluates as:

> ```
> (result * 10 + cursor) - '0'
> ```

and this overflows.

How can we design a language where programmers don’t need to worry about transient overflows? It’s easy:

> **Premise 3: Expression evaluation always returns the mathematically correct result. Overflow can only occur when the result of an expression is stored into a fixed-size integer type.**

This is more or less what Mattias Engdegí¥rd said in a comment to the previous post. Clearly, eliminating transient overflows is desirable, but how costly is it? I believe that in most cases, mathematical correctness for intermediate results is free. In a minority of cases math will need to be performed at a wider bitwidth, for example promoting 32-bit operands to 64 bits before performing operations. Are there pathological cases that require arbitrary precision? This depends on what operators are available inside the expression evaluation context. If the language supports a builtin factorial operator and we evaluate `y = x!;` there’s no problem — once the factorial result is larger than can fit into y, we can stop computing and trap. On the other hand, evaluating `b = !!(x! % 1000000);` is a problem — there’s no obvious shortcut in the general case (unless the optimizer is especially smart). We can sidestep this kind of problem by making factorial into a library function that returns a fixed-width integer.

A consequence of premise 3 that I like is that it reduces the number of program points where math exceptions may be thrown. This makes it easier to debug complex expressions and also gives the optimizer more freedom to rearrange code.

The next question is, should overflows trap early or late? Early trapping — where an exception is thrown as soon as an overflow occurs — is simple and predictable; it may be the right answer for a new language. But let’s briefly explore the design of a late-trapping overflow system for integer math. Late trapping makes integers somewhat analogous to pointers in C/C++/Java where there’s an explicit null value.

We require a new signed integer representation that has a NaN (not-a-number) value. NaN is contagious in the sense that any math operation which inputs a NaN produces NaN as its result. Integer NaNs can originate explicitly (e.g., `x = iNaN;`) and also implicitly due to uninitialized data or when an operation overflows or otherwise incurs a value loss (e.g., a narrowing type cast).

Integer NaN values propagate silently unless they reach certain program operations (array indexing and pointer arithmetic, mainly) or library functions (especially resource allocation). At that point an exception is thrown.

The new integer representation is identical to two’s complement except that the bit pattern formerly used to represent INT\_MIN now represents NaN. This choice has the side effect of eliminating the inconvenient asymmetry where the two’s complement INT\_MIN is not equal to -INT\_MAX. Processor architectures will have to add a family of integer math instructions that properly propagate NaN values and also a collection of addressing modes that trap when an address register contains a NaN. These should be simple ALU hacks.

A few examples:

> ```
> int x; // x gets iNaN
> char c = 1000; // c gets iNaN
> cout << iNaN; // prints "iNaN"
> p = &x + iNaN; // throws exception
> x = a[iNaN]; // throws exception
> ```

Good aspects of this design are that an explicit “uninitialized” value for integers would be useful (see [here](http://rusty.ozlabs.org/?p=232) for example), execution would be efficient given hardware support, programs would tolerate certain kinds of benign overflows, and the stupid -INT\_MAX thing has always bothered me. It’s also possible that a more relaxed overflow system leads to some nice programming idioms. For example, [iterating over the full range](https://blog.regehr.org/archives/536) is awkward in C. But now that we have a sentinel value we can write:

> ```
> for (x = INT_MIN; x != iNaN; x++) { use (x); }
> ```

The main disadvantage of late trapping is that it would sometimes mask errors and make debugging more difficult in the same way that null pointers do. Type system support for “valid integers”, analogous to non-null pointers, would mitigate these problems. On the other hand, perhaps we’d have been better off with early trapping.

Finally, we could design an integer system using the iNaN integer representation and also early trapping. In this system, iNaN would exclusively be used to represent uninitialized or garbage values. Any operation on iNaN other than simple copying would result in a trap.
