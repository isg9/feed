---
title: Nine ways to break your systems code using volatile
url: https://blog.regehr.org/archives/28
published: "2010-02-27T02:49:00Z"
feed: regehr
guid: http://blog.regehr.org/?p=28
---

# Nine ways to break your systems code using volatile

The volatile qualifier in C/C++ is a little bit like the C preprocessor: an ugly, blunt tool that is easy to misuse but that — in a very narrow set of circumstances — gets the job done.  This article will first briefly explain volatile and its history and then, through a series of examples about how not to use it, explain how to most effectively create correct systems software using volatile.  Although this article focuses on C, almost everything in it also applies to C++.

# What does a C program mean?

The [C standard](http://www.open-std.org/JTC1/SC22/wg14/www/docs/n1124.pdf) defines the meaning of a C program in terms of an “abstract machine” that you can think of as a simple, non-optimizing interpreter for C.  The behavior of any given C implementation (a compiler plus target machine) must produce the same side effects as the abstract machine, but otherwise the standard mandates very little correspondence between the abstract machine and computation that actually happens on the target platform.  In other words, the C program can be thought of as a specification of desired effects, and the C implementation decides how to best go about making those effects happen.

As a simple example, consider this function:

```
int loop_add3 (int x) {
  int i;
  for (i=0; i<3; i++) x++;
  return x;
}
```

The behavior of the abstract machine is clear: it creates a function-scoped variable named i which loops from 0 through 2, adding 1 to x on each iteration of the loop.  On the other hand, a good compiler emits code like this:

```
loop_add3:
  movl 4(%esp), %eax
  addl $3, %eax
  ret
```

In the actual machine, i is never incremented or even instantiated, but the net effect is the same as if it had been.  In general, this gap between the abstract and actual semantics is considered to be a good thing, and the “as if” wording in the standard is what gives the optimizer the freedom to generate efficient code from a high-level specification.

# What does volatile mean?

The problem with the gap between the abstract and concrete semantics is that C is a low-level language that was designed for implementing operating systems.  Writing OS code often requires that the gap between the abstract and actual semantics be narrowed or closed.  For example, if an OS wants to create a new page table, the abstract semantics for C fails to capture an important fact: that the programmer requires an actual page table that sits in RAM where it can be traversed by hardware.  If the C implementation concludes that the page table is useless and optimizes it away, the developers’ intent has not been served.  The canonical example of this problem is when a compiler decides that a developer’s code for zeroing sensitive data is useless and optimizes it away.  Since the C abstract machine was not designed to consider cases where this data may be snooped later, the optimization is legitimate (though obviously undesirable).

The C standard gives us just a few ways to establish a connection between the abstract and actual machines:

- the arguments to main()
- the return value from main()
- the side-effecting functions in the C standard library
- volatile variables

Most C implementations offer additional mechanisms, such as inline assembly and extra library functions not mandated by the standard.

The way the volatile connects the abstract and real semantics is this:

> For every read from a volatile variable by the abstract machine, the actual machine must load from the memory address corresponding to that variable.  Also, each read may return a different value.  For every write to a volatile variable by the abstract machine, the actual machine must store to the corresponding address.  Otherwise, the address should not be accessed (with some exceptions) and also accesses to volatiles should not be reordered (with some exceptions).

In summary:

- Volatile has no effect on the abstract machine; in fact, the C standard explicitly states that for a C implementation that closely mirrors the abstract machine (i.e. a simple C interpreter), volatile has no effect at all.
- Accesses to volatile-qualified objects obligate the C implementation to perform certain operations at the level of the actual computation.

# Where did volatile come from?

Historically, the connection between the abstract and actual machines was established mainly through accident: compilers weren’t good enough at optimizing to create an important semantic gap.  As optimizers improved, it became increasingly clear that a systematic solution was needed.  In an [excellent USENET post](http://groups.google.com/group/comp.std.c/msg/7709e4162620f2cd) 20 years ago Doug Gwyn explained how volatile came about:

> To take a specific example, UNIX device drivers are almost always coded entirely in C, and on the PDP-11 and similar memory-mapped I/O architectures, some device registers perform different actions upon a “read-byte”, “read-word”, “write-byte”, “write-word”, “read-modify-write”, or other variations of the memory-bus access cycles involved.  Trying to get the right type of machine code generated while coding the driver in C was quite tricky, and many hard-to-track-down bugs resulted.  With compilers other than Ritchie’s, enabling optimization often would change this behavior, too.  At least one version of the UNIX Portable C Compiler (PCC) had a special hack to recognize constructs like
>
> ```
> ((struct xxx *)0177450)->zzz
> ```
>
> as being potential references to I/O space (device registers) and would avoid excessive optimization involving such expressions (where the constant lay within the Unibus I/O address range).  X3J11 decided that this problem had to be faced squarely, and introduced “volatile” to obviate the need for such hacks.  However, although it was proposed that conforming implementations be required to implement the minimum possible access “width” for volatile-qualified data, and that is the intent of requiring an implementation definition for it, it was not practical to insist on it in every implementation; thus, some latitude was allowed implementors in that regard.

It’s hard to overstate how bad an idea it is for a compiler to use strange heuristics about code structure to guess the developer’s intent.

# Nine ways to break your systems code using volatile

## 1\. Not enough volatile

The most obvious kind of volatile error is to leave it out when it is required.  Let’s look at a specific example.  Suppose we’re developing software for an AVR 8-bit embedded processor, which (on some models) has no hardware multiplier.  Since multiplies are going to happen in software, we’re probably interested in seeing how slow they are, so we know how hard to try to avoid them.  So we write a little benchmark program like this:

```
#define TCNT1 (*(uint16_t *)(0x4C))
```

```
signed char a, b, c;
```

```
uint16_t time_mul (void) {
  uint16_t first = TCNT1;
  c = a * b;
  uint16_t second = TCNT1;
  return second - first;
}
```

Here TCNT1 points to a hardware register living at address 0x4C.  This register provides access to Timer/Counter 1: a free-running 16-bit timer that we assume is configured to run at some rate appropriate for this experiment.  We read the register before and after the multiply operation, and subtract to find the duration.  Side note: although at first glance this code looks like it fails to account for the case where TCNT1 overflows from 65535 to 0 during the timing run, it actually works properly for all durations between 0 and 65535 ticks.

Unfortunately, when we run this code, it always reports that the multiply operation required zero clock ticks.  To see what went wrong, let us look at the assembly language:

```
$ avr-gcc -Os -S -o - reg1.c
time_mul:
  lds r22,a
  lds r24,b
  rcall __mulqi3
  sts c,r24
  ldi r24,lo8(0)
  ldi r25,hi8(0)
  ret
```

Now the problem is obvious: both reads from the TCNT1 register have been eliminated and the function is simply returning the constant zero (avr-gcc returns a 16-bit value in the r24:r25 register pair).

How can the compiler get away with never reading from TCNT1?  First, let’s remember that the meaning of a C program is defined by the abstract machine described in the C standard.  Since the rules for the abstract machine say nothing about hardware registers (or concurrent execution) the C implementation is permitted to assume that two reads from an object, with no intervening stores, both return the same value.  Of course, any value subtracted from itself is zero.  So the translation performed by avr-gcc here is perfectly correct; it is our program that’s wrong.

To fix the problem, we need to change the code so that TCNT1 points to a volatile location.

```
#define TCNT1 (*(volatile uint16_t *)(0x4C))
```

Now, the C implementation is not free to eliminate the reads and also it cannot assume the same value is read both times.  This time the compiler outputs better code:

```
$ avr-gcc -Os -S -o - reg2.c
time_mul:
  in r18,0x2c
  in r19,0x2d
  lds r22,a
  lds r24,b
  rcall __mulqi3
  sts c,r24
  in r24,0x2c
  in r25,0x2d
  sub r24,r18
  sbc r25,r19
  ret
```

Although this assembly code is correct, our C code still contains a latent error.  We’ll explore it later.

Normally, you will find definitions for device registers in system header files.  If so, you will not need to use volatile in this case.  But it may be worth checking that the definitions are correct, they aren’t always.

Let’s look at another example.  In an embedded system you are implementing, some computation must wait for an interrupt handler to fire.  Your code looks like this:

```
int done;
```

```
__attribute((signal)) void __vector_4 (void) {
  done = 1;
}
```

```
void wait_for_done (void) {
  while (!done) ;
}
```

Here wait\_for\_done() is designed to be called from the non-interrupt context, whereas \_\_vector\_4() will be invoked by the interrupt controller in response to some external event.  We compile this code into assembly:

```
$ avr-gcc -Os wait.c -S -o -
__vector_4:
  push r0
  in r0,__SREG__
  push r0
  push r24
  ldi r24,lo8(1)
  sts done,r24
  pop r24
  pop r0
  out __SREG__,r0
  pop r0
  reti
```

```
wait_for_done:
  lds r24,done
.L3:
  tst r24
  breq .L3
  ret
```

The code for the interrupt handler looks good: it stores to done as intended.  The rest of the interrupt handler is just AVR interrupt boilerplate.  However, the code for wait\_for\_done() contains an important flaw: it is spinning on r24 instead of spinning on a RAM location.  This happens because the C abstract machine has no notion of communication between concurrent flows (whether they are threads, interrupts, or anything else).  Again, the translation is perfectly correct, but does not match the developer’s intent.

If we mark done as a volatile variable, the interrupt handler code does not change, but wait\_for\_done() now looks like this:

```
wait_for_done:
.L3:
  lds r24,done
  tst r24
  breq .L3
  ret
```

This code will work.  The issue here is one of visibility.  When you store to a global variable in C, what computations running on the machine are guaranteed to see the store?  When you load from a global variable, what computations are assumed to have produced the value?  In both cases, the answer is “the computation that performs the load or store is assumed to be the only computation that matters.”  That is, C makes no visibility guarantees for normal variable references.  The volatile qualifier forces stores to go to memory and loads to come from memory, giving us a way to ensure visibility across multiple computations (threads, interrupts, coroutines, or whatever).

Again, our C code contains a latent bug that we’ll investigate later.

A few other legitimate uses of volatile, including making variables in UNIX programs visible to signal handlers, are [discussed by Hans Boehm](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2006/n2016.html).

**Summary: The abstract C machine is connected to the actual machine in only a few places.  The memory behavior of the actual machine may be very different from the operations specified in source code.  If you require additional connections between the two levels of abstraction, for example to access device registers, the volatile qualifier can help.**

## 2\. Too much volatile

In a well-designed piece of software, volatile is used exactly where it is needed.  It serves as documentation, saying in effect “this variable does not play by the C rules: it requires a strong connection with the memory subsystem.”  In a system that uses too much volatile, variables will be indiscriminately labeled as volatile, without any technical justification.  There are three reasons why this is bad.  First, it’s bad documentation and will confuse subsequent maintainers.  Second, volatile sometimes has the effect of hiding program bugs such as race conditions.  If your code needs volatile and you don’t understand why, this is probably what is happening.  Far better to actually fix the problem than to rely on a hack you do not understand to solve a problem you do not understand.  Finally, volatile causes inefficiency by handicapping the optimizer.  The overhead that it introduces is hard to track down since it is spread out all over the system– a profiler will be little help in finding it.

Using volatile is a little like deciding what kind of insurance policy to buy.  Too little insurance and you may run into problems down the road.  Too much insurance and you’re covered but in the long run you end up paying too much.

**Summary: Use volatile only when you can provide a precise technical justification.  Volatile is not a substitute for thought ( [Nigel Jones said this](http://www.netrino.com/Embedded-Systems/How-To/C-Volatile-Keyword)).**

## 3\. Misplaced qualification

At the level of C syntax, volatile is a type qualifier.  It can be applied to any type, following rules that are similar to, but not quite the same as, the rules for the const qualifier.  The situation can become confusing when qualified types are used to build up more complex types.  For example, there are four possible ways to qualify a single-level pointer:

```
int *p;                              // pointer to int
volatile int *p_to_vol;              // pointer to volatile int
int *volatile vol_p;                 // volatile pointer to int
volatile int *volatile vol_p_to_vol; // volatile pointer to volatile int
```

In each case, either the pointer is volatile or not, and the pointer target is volatile or not.  The distinction is crucial: if you use a “volatile pointer to regular int” to access a device register, the compiler is free to optimize away accesses to the register.  Also, you will get slow code since the compiler will not be free to optimize accesses to the pointer.  This problem comes up pretty often on embedded mailing lists; it’s an easy mistake to make.  It’s also easy to overlook when vetting code since your eye may just be looking for a volatile somewhere.

For example, this code is wrong:

```
int *volatile REGISTER = 0xfeed;
*REGISTER = new_val;
```

To write clear, maintainable code using volatile, a reasonable idea is to build up more complex types using typedefs (of course this is a good idea anyway).  For example we could first make a new type “vint” which is a volatile int:

```
typedef volatile int vint;
```

Next, we create a pointer-to-vint:

```
vint *REGISTER = 0xfeed;
```

Members of a struct or union can be volatile, and structs/unions can also be volatile.  If an aggregate type is volatile, the effect is the same as making all members volatile.

We might ask, does it make sense to declare an object as both const and volatile?

```
const volatile int *p;
```

Although this initially looks like a contradiction, it is not.  The semantics of const in C are “I agree not to try to store to it” rather than “it does not change.”  So in fact this qualification is perfectly meaningful and would even be useful, for example, to declare a timer register than spontaneously changes value, but that should not be stored to (this example is specifically pointed out in the C standard).

**Summary: Since C’s type declaration syntax is not particularly readable or intuitive, volatile qualifiers must be placed with care.  Typedefs are a useful way to structure complex declarations.**

## 4\. Inconsistent qualification

The last version of Linux 2.2 was 2.2.26.  In that version, in the file arch/i386/kernel/smp.c at line 125, we find this definition:

```
volatile unsigned long ipi_count;
```

So far, no problem: we’re declaring a long to store the number of inter-processor interrupts and making it volatile.  However, in the header file include/asm-i386/smp.h at line 178 we find this definition:

```
extern unsigned long ipi_count;
```

C files that include this header will not treat ipi\_count as volatile, and this could easily cause problems.  Kernels in the 2.3 series also contain this error.

Recent versions of gcc treat this kind of inconsistent qualification as a compile-time error, so these problems have disappeared.  However, it’s a good bet that some embedded compilers (obviously including those based on older versions of gcc) will permit you to make this mistake.

Another way to get inconsistent qualification is through typecasts.  Casts can be implicit, for example passing a pointer-to-volatile to a function expecting an unqualified pointer.  The compiler will warn about this; these warnings should never be ignored or suppressed.  Explicit typecasts that remove qualifiers should be avoided, these generally do not cause any warnings.  The C standard explicitly states that a program’s behavior is undefined if you access a volatile object through an unqualified pointer.

**Summary: Never use inconsistent qualification.  If a variable is declared as volatile, then all accesses to it, direct or indirect, must be through volatiles and pointers-to-volatile.**

## 5\. Expecting volatile to enforce ordering with non-volatile accesses

Next we come to an issue that even some experts in embedded software development get wrong, and that even experts in C language semantics have arguments about.

The question is: What was wrong with the fixed C code examples above, where we added volatile to the TCNT1 register handle and to the done flag?  The answer, depending on who you believe, is either “nothing” or else “the compiler may reorder the operations in such a way as to create broken output.”

One school of thought is that compilers may not move accesses to global variables around accesses to volatile variables.  There seems to be a consistent reading of the standard that backs this up.  The problem with this reading is that important compilers are based on a different interpretation, which says that accesses to non-volatile objects can be arbitrarily moved around volatile accesses.

Take this simple example (which originated with Arch Robison):

```
volatile int ready;
int message[100];
```

```
void foo (int i) {
  message[i/10] = 42;
  ready = 1;
}
```

The purpose of foo() is to store a value into the message array and then set the ready flag so that another interrupt or thread can see the value.  From this code, GCC, Intel CC, Sun CC, and Open64 emit very similar assembly:

```
$ gcc -O2 barrier1.c -S -o -
foo:
  movl 4(%esp), %ecx
  movl $1717986919, %edx
  movl $1, ready
  movl %ecx, %eax
  imull %edx
  sarl $31, %ecx
  sarl $2, %edx
  subl %ecx, %edx
  movl $42, message(,%edx,4)
  ret
```

Obviously the programmer’s intent is not respected here, since the flag is stored prior to the value being written into the array.  As of this writing LLVM does not do this reordering but, as far as I know, this is a matter of chance rather than design.  A number of embedded compilers refuse to do this kind of reordering as a deliberate choice to prefer safety over performance.  I’ve heard, but not checked, that recent Microsoft C/C++ compilers also take a very conservative stance on volatile accesses.  This is probably the right choice, but it doesn’t help people who have to write portable code.

One way to fix this problem is to declare message as a volatile array.  The C standard is unambiguous that volatile side effects must not move past sequence points, so this will work.  On the other hand, adding more volatile qualifiers may suppress interesting optimizations elsewhere in the program.  Wouldn’t it be nice if we could force data to memory only at selected program points without making things volatile everywhere?

The construct that we need is a “compiler barrier.”  The C standard does not provide this, but many compilers do.  For example, GCC and sufficiently compatible compilers (including LLVM and Intel CC) support a memory barrier that looks like this:

```
asm volatile ("" : : : "memory");
```

It means roughly “this inline assembly code, although it contains no instructions, may read or write all of RAM.”  The effect is that the compiler dumps all registers to RAM before the barrier and reloads them afterwards.  Moreover, code motion is not permitted around the barrier in either direction.  Basically a compiler barrier is to an optimizing compiler as a memory barrier is to an out-of-order processor.

We can use a barrier in the code example:

```
volatile int ready;
int message[100];
```

```
void foo (int i) {
  message[i/10] = 42;
  asm volatile ("" : : : "memory");
  ready = 1;
}
```

Now the output is what we wanted:

```
$ gcc -O2 barrier2.c -S -o -
foo:
  movl 4(%esp), %ecx
  movl $1717986919, %edx
  movl %ecx, %eax
  imull %edx
  sarl $31, %ecx
  sarl $2, %edx
  subl %ecx, %edx
  movl $42, message(,%edx,4)
  movl $1, ready
  ret
```

What about compilers that fail to support memory barriers?  One bad solution is to hope that this kind of compiler isn’t aggressive enough to move accesses around in a harmful way.  Another bad solution is to insert a call to an external function where you would put the barrier.  Since the compiler doesn’t know what memory will be touched by this function, it may have a barrier-like effect.  A better solution would be to ask your compiler vendor to fix the problem and also to recommend a workaround in the meantime.

**Summary: Most compilers can and will move accesses to non-volatile objects around accesses to volatile objects, so don’t rely on the program ordering being respected.**

## 6\. Using volatile to get atomicity

Earlier we saw a case where volatile was used to make a value visible to a concurrently running computation.  This was — in limited circumstances — a valid implementation choice.  On the other hand it is never valid to use volatile to get atomicity.

Somewhat surprisingly for a systems programming language, C does not provide guarantees about atomicity of its memory operations, regardless of the volatility of objects being accessed.  Generally, however, individual compilers will make guarantees such as “aligned accesses to word-sized variables are atomic.”

In most cases, you use locks to get atomicity.  If you’re lucky, you have access to well-designed locks that contain compiler barriers.  If you’re programming on bare metal on an embedded processor, you may not be so lucky.  If you have to devise your own locks, it would be wise to add compiler barriers.  For example, older versions of [TinyOS](http://tinyos.net/) for AVR chips used these functions to acquire and release the global interrupt lock:

```
char __nesc_atomic_start (void) {
  char result = SREG;
  __nesc_disable_interrupt();
  return result;
}
```

```
void __nesc_atomic_end (char save) {
  SREG = save;
}
```

Since these functions can be (and generally are) inlined, it was always possible for the compiler to move code outside of a critical section.  We changed the locks to look like this:

```
char__nesc_atomic_start(void) {
  char result = SREG;
  __nesc_disable_interrupt();
  asm volatile("" : : : "memory");
  return result;
}
```

```
void __nesc_atomic_end(char save) {
  asm volatile("" : : : "memory");
  SREG = save;
}
```

Perhaps interestingly, this had no effect on TinyOS efficiency and even made the code smaller in some cases.

**Summary: Volatile has nothing to do with atomicity.  Use locks.**

## 7\. Using volatile on a modern machine

Volatile is of very limited usefulness on a machine that is out-of-order, multiprocessor, or both.  The problem is that while volatile forces the compiler to emit memory operations on the actual machine, these loads and stores by themselves do not constrain the hardware’s behavior very much.  It is vastly preferable to find good locking primitives and use them, as opposed to rolling them yourself.  Consider this spin-unlock function from the ARM port of Linux:

```
static inline void arch_spin_unlock(arch_spinlock_t *lock) {
  smp_mb();
  __asm__ __volatile__("str %1, [%0]\n" : : "r" (&lock->lock), "r" (0) : "cc");
}
```

Before unlocking, smp\_mb() is executed, which boils down to something like this:

```
__asm__ __volatile__ ("dmb" : : : "memory");
```

This is both a compiler barrier and a memory barrier.

**Summary: Without help from properly designed synchronization libraries, writing correct concurrent code on an out-of-order machine or multiprocessor is extremely hard, and volatile is of little help.**

## 8\. Using volatile in multi-threaded code

This issue overlaps with the previous two but it’s important enough to be worth repeating.  Arch Robison says that [volatile is almost useless for multi-threaded programming](http://software.intel.com/en-us/blogs/2007/11/30/volatile-almost-useless-for-multi-threaded-programming/).  And he’s right.  If you have threads then you should also have locks, and should use them.  There’s a wonderful result showing that properly synchronized code — where shared variables are always accessed from within critical sections — executes in a sequentially consistent fashion (provided that the locks are properly implemented, and you shouldn’t have to worry about that).  This means that if you use locks, you don’t have to worry about compiler barriers, memory system barriers, or volatile.  None of it matters.

**Summary: To write correct multi-threaded code, you need primitives providing (at least) atomicity and visibility.  On modern hardware volatile provides neither.  You should write properly synchronized multi-threaded code whenever possible.  Idioms like double-checked locking are best avoided.**

## 9\. Assume volatile accesses are translated correctly

Compilers are not totally reliable in their translation of accesses to volatile-qualified objects.  I’ve [written extensively about this subject elsewhere](http://www.cs.utah.edu/~regehr/papers/emsoft08-preprint.pdf), but here’s a quick example:

```
volatile int x;
```

```
void foo (void) {
  x = x;
}
```

The proper behavior of this code on the actual machine is unambiguous: there should be a load from x, then a store to it.  However, the port of GCC to the MSP430 processor behaves differently:

```
$ msp430-gcc -O vol.c -S -o -
foo:
  ret
```

The emitted function is a nop.  It is wrong.  In general, compilers based on gcc 4.x are mostly volatile-correct, as are recent versions of LLVM and Intel CC.  Pre-4.0 versions of gcc have problems, as do a number of other compilers.

**Summary: If your code makes correct use of volatiles and still does not work, consider reading the compiler’s output to make sure it has emitted the proper memory operations.**

# What about the Linux people?

You can find various rants, screeds, and diatribes against volatile on Linux mailing lists and web pages.  These are largely correct, but you have to keep in mind that:

1. Linux often runs on out-of-order multicores where volatile by itself is nearly useless.
2. The Linux kernel provides a rich collection of functions for synchronization and hardware access that, properly used, eliminate almost all need for volatile in regular kernel code.

If you are writing code for an in-order embedded processor and have little or no infrastructure besides the C compiler, you may need to lean more heavily on volatile.

# Summary

Optimizing compilers are tricky to reason about, as are out-of-order processors.  Also, the C standard contains some very dark corners.  The volatile qualifier invites all of these difficulties to come together in one place and interact with one another.  Furthermore, it provides weaker guarantees than people commonly assume.  Careful thought is required to create correct software using volatile.

Happily, most programmers who write user-mode C and C++ can get away without ever using volatile.  Also, the vast majority of kernel mode programmers will seldom if ever need volatile.  It is primarily needed by people working near bare metal; for example, working on an embedded microcontroller or porting an OS to a new platform.

# Credentials

I’ve been programming computers for 26 years and embedded systems for 17 years.  For the last eight years I’ve taught operating systems, embedded systems, compilers, and related courses to undergrads and graduate students.  I’ve tried to educate them about volatile and have a pretty good idea about where people go wrong with it.

In February 2010 I gave some lectures at RWTH Aachen, including about an hour on getting the volatile qualifier wrong.  This post expands on that material.
