---
title: Fun With Saturating Arithmetic
url: https://blog.regehr.org/archives/277
published: "2010-10-06T05:34:35Z"
feed: regehr
guid: http://blog.regehr.org/?p=277
---

# Fun With Saturating Arithmetic

An assignment that I often give my Advanced Embedded Systems class early in the semester is to implement saturating versions of signed and unsigned addition and subtraction. Saturating operations “stick” at the maximum or minimum value. (INT\_MAX +sat 1) therefore evaluates to INT\_MAX. The only wrinkle in the assignment is that solutions must be capable of being compiled (via a typedef) for integers of 8, 16, 32, or 64 bits. I give this assignment for several reasons:

- I like people to see saturating arithmetic since it is useful in some embedded applications
- it should be easy and fun, but ends up being a bit tricky
- the (typically) poor performance of the class helps motivate later course material on testing and on integer problems

Over the years I’ve ended up with nearly 100 submissions from students. Tonight I ran them all through my grading script and found that 23 students got all 4 functions right, for all integer widths. Also I rediscovered a few gems:

# Segfaults From Integer Code

Two students managed to create solutions that segfault. How is this impressive result even possible, you ask? It was accomplished by implementing addition and subtraction as mutually recursive functions and failing to stop the recursion. Thus, the crash is a stack overflow.

# Exponential Running Time

Two more students implemented iterative solutions with running time exponential in the size of the inputs. This can be accomplished for unsigned addition by, for example, repeatedly incrementing one of the arguments and decrementing the other, stopping when the first saturates or the second reaches zero. While spectacularly inefficient, one of these solutions was correct, as far as I could tell. I had to give the student full points because the assignment failed to specify that the functions should terminate in a reasonable time in the 64-bit case.

# The Most Creative Way To Pass Tests

My test harness invokes the students’ saturating operations with random inputs as well as values at or near corner cases. By far the most creative wrong solution came from a student who implemented three of the four functions correctly, but forgot to return a value from the fourth. Of course this is an undefined behavior in C, but not a compile time error. Astoundingly, Intel CC, a recent Clang snapshot, and GCC 3.4 all report zero errors from this student’s code, when compiling with optimizations. The reason for this is entertaining:

1. The compiler, seeing a function that always relies on undefined behavior (I call these [Type 3 functions](https://blog.regehr.org/archives/213)), turns it into a nop.
2. My test harness calls the reference version of the saturating function, which leaves its result in eax.
3. The value is copied into ebx, but eax still retains a copy.
4. The student’s code is called. It is supposed to store its result in eax, but remember that it is now a nop.
5. The test harness compares eax against ebx. Of course they are equal, and so the test passes.

Have I said before that the way a C compiler silently destroys Type 3 functions is effectively evil? I have, and I’m saying it again.

# A Corner Case

Each year a few people implement saturating signed subtraction like this:

> ```
> mysint sat_signed_sub (mysint a, mysint b)
> {
>   return sat_signed_add (a, -b);
> }
> ```

This is tantalizingly simple, but wrong.

# Big Code == Wrong Code

After being compiled (for the 32-bit integer case, using a recent version of gcc for x86 at -Os) solutions I’ve received range from 76 to 1159 bytes of object code, totaled across all four functions. The largest correct solution is 226 bytes. Bigger solutions tend to be more complicated, and complicated solutions are usually wrong.

# The Best Code

I tried quite a few compilers and could not manage to generate a correct solution smaller than 91 bytes of x86 code (again, for the 32-bit case). This is from a recent GCC snapshot. The C code for this solution was produced by one of the brightest undergrads I’ve had the pleasure of teaching; it succeeds by using bit operators instead of arithmetic. I’m sure a good assembly programmer could shave some bytes off of this, but I suspect that the optimal solution is not a whole lot smaller. I’d love to include the code here but would like to avoid invalidating this entertaining assignment.
