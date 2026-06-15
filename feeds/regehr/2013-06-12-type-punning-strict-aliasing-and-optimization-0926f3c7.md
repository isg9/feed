---
title: Type Punning, Strict Aliasing, and Optimization
url: https://blog.regehr.org/archives/959
published: "2013-06-12T03:38:21Z"
feed: regehr
guid: http://blog.regehr.org/?p=959
---

# Type Punning, Strict Aliasing, and Optimization

One of the basic jobs of a low-level programming language like C or C++ is to make it easy to peek and poke at data representations, as opposed to providing opaque high-level abstractions. Access to representations supports grungy tasks such as JIT compiling, setting up page tables, driving peripherals, and communicating with machines that use different representations.

As an example, we’ll look at taking an array of four 16-bit quantities and viewing it as a single 64-bit quantity. The basic observation is that this operation doesn’t require moving any data around, it only requires looking at some data in a different way. In C/C++ it’s not possible to directly type-cast an array, but it is possible to cast between kinds of pointers, so we might write something like this:

```
uint64_t c1 (const uint16_t *buf)  {
  return *(uint64_t *)buf;
}

```

This code is appealing because it is short and also it captures the idea that we are simply looking at data in a different way, as opposed to moving it around. And in fact, at `-O1` and higher, recent versions of Clang and GCC (on x86-64 Linux) compile `c1` into perfect assembly:

```
c1:
        movq    (%rdi), %rax
        ret

```

Here the `movq` instruction moves 64 bits from where `%rdi`, the argument, points to, into `%rax`, which is where 64-bit return values live. On a 32-bit platform the output is also good, though not quite as pretty due to the calling convention and the lack of a 64-bit move instruction:

```
c1:
        movl    4(%esp), %eax
        movl    4(%eax), %edx
        movl    (%eax), %eax
        ret

```

The problem with `c1` is that it violates C/C++’s “strict aliasing” rule which basically says that if you cast a pointer to a different kind of pointer and then dereference it, then your program has executed an [undefined behavior](https://blog.regehr.org/archives/213). The pernicious thing about undefined behavior is that it leads to problems only sometimes, and obviously, nothing bad happened here.

There do exist, however, simple programs where violations of strict aliasing lead to problems (this program is adapted from one I found on the net somewhere, I think on Stack Overflow):

```
#include <stdio.h>

void check (int *h, long *k) {
  *h = 5;
  *k = 6;
  printf("%d\n", *h);
}

int main (void) {
  long k;
  check((int *)&k, &k);
  return 0;
}

```

The problem here is that `k` is aliased by a pointer to long and also a pointer to int. The result:

```
$ gcc -O1 cast3.c -o cast3 ; ./cast3
6
$ gcc -O2 cast3.c -o cast3 ; ./cast3
5
$ clang -O1 cast3.c -o cast3 ; ./cast3
6
$ clang -O2 cast3.c -o cast3 ; ./cast3
5

```

As you can imagine, a lot of code that previously worked broke when type-based alias analysis was turned on in GCC. A type-based alias analysis is one that assumes pointers to different types are not aliases. In a type-safe programming language, this is generally a solid assumption. In C/C++, the responsibility for avoiding problems is placed on developers, resulting in the strict aliasing rules. There is, by the way, a severe lack of tools which check for violations of strict aliasing (compilers warn about some violations, but these warnings are highly unreliable).

Ok, back to the example: `c1` contains code that we must not use. The next most obvious way to change representations is to define a union like this:

```
union u {
  uint16_t buf[4];
  uint64_t l;
};

```

Now we use the type punning feature of unions to read out the other representation:

```
uint64_t c2 (const uint16_t *buf) {
  union u tp;
  tp.buf[0] = buf[0];
  tp.buf[1] = buf[1];
  tp.buf[2] = buf[2];
  tp.buf[3] = buf[3];
  return tp.l;
}

```

Unfortunately this code is also undefined by modern C/C++ dialects (or maybe just unspecified, I’m not totally sure). On the other hand, [GCC explicitly provides the intuitive semantics for this code](http://gcc.gnu.org/onlinedocs/gcc/Structures-unions-enumerations-and-bit_002dfields-implementation.html). The [Clang user’s manual](http://clang.llvm.org/docs/UsersManual.html) does not include the string “union” anywhere, but since Clang more or less emulates GCC, we might expect that it provides the same behavior. However, other compilers [such as Sun’s do not](http://blog.qt.digia.com/blog/2011/06/10/type-punning-and-strict-aliasing/) (and yes, [this compiler still exists](http://www.oracle.com/technetwork/server-storage/solarisstudio/overview/index.html)).

Also, compilers such as GCC 4.8.1 and Clang ~3.3 (both for x86-64 Linux) fail to generate good code for `c2`. Here is Clang’s output:

```
c2:
        movzwl  (%rdi), %ecx
        movzwl  2(%rdi), %eax
        shlq    $16, %rax
        orq     %rcx, %rax
        movzwl  4(%rdi), %ecx
        shlq    $32, %rcx
        orq     %rax, %rcx
        movzwl  6(%rdi), %eax
        shlq    $48, %rax
        orq     %rcx, %rax
        ret

```

Perhaps oddly, both GCC and Clang generate optimal object code from this variant:

```
uint64_t c3 (const uint16_t *buf) {
  union u tp;
  memcpy (&tp, buf, 8);
  return tp.l;
}

```

Of course `c3` is still undefined/unspecified for other compilers.

Now let’s try to write some well-defined C code:

```
uint64_t c4 (const uint16_t *buf) {
  return
      (((uint64_t)buf[0]) << (0*16))
    | (((uint64_t)buf[1]) << (1*16))
    | (((uint64_t)buf[2]) << (2*16))
    | (((uint64_t)buf[3]) << (3*16))
    ;
}

```

This works but results in basically the same crappy object code that we got from `c2`.

Finally, let’s try this:

```
uint64_t c5 (const uint16_t *buf) {
  uint64_t num;
  memcpy(&num, buf, 8);
  return num;
}

```

Again, although we might expect that adding a function call would make the code harder to optimize, both compilers understand `memcpy` deeply enough that we get the desired object code:

```
c5:
        movq    (%rdi), %rax
        ret

```

In my opinion `c5` is the easiest code to understand out of this little batch of functions because it doesn’t do the messy shifting and also it is totally, completely, obviously free of complications that might arise from the confusing rules for unions and strict aliasing. It became my preferred idiom for type punning a few years ago when I discovered that compilers could see through the `memcpy` and generate the right code.
