---
title: Legitimate-ish Use of alloca()
url: https://nullprogram.com/blog/2019/10/28/
published: "2019-10-28T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2019/10/28/
---

# Legitimate-ish Use of alloca()

## [Legitimate-ish Use of alloca()](/blog/2019/10/28/)

October 28, 2019

nullprogram.com/blog/2019/10/28/

*This article was discussed [on Hacker News](https://news.ycombinator.com/item?id=21374863)*.

Yesterday [I wrote about a legitimate use for variable length
arrays](/blog/2019/10/27/). While recently discussing this topic with [a
co-worker](/blog/2016/09/02/), I also thought of a semi-legitimate use for [`alloca()`](http://man7.org/linux/man-pages/man3/alloca.3.html), a non-standard “function” for dynamically allocating memory on the stack.

```
void *alloca(size_t);

```

I say “function” in quotes because it’s not truly a function and cannot be implemented as a function or by a library. It’s implemented in the compiler and is essentially part of the language itself. It’s a tool allowing a function to manipulate its own stack frame.

Like VLAs, it has the problem that if you’re able to use `alloca()` safely, then you really don’t need it in the first place. Allocation failures are undetectable and once they happen it’s already too late.

### Opaque structs

To set the scene, let’s talk about opaque structs. Suppose you’re writing a C library with [a clean interface](/blog/2018/06/10/). It’s set up so that changing your struct fields won’t break the Application Binary Interface (ABI), and callers are largely unable to depend on implementation details, even by accident. To achieve this, it’s likely you’re making use of *opaque structs* in your interface. Callers only ever receive pointers to library structures, which are handed back into the interface when they’re used. The internal details are hidden away.

```
/* opaque float stack API */
struct stack *stack_create(void);
void          stack_destroy(struct stack *);
int           stack_push(struct stack *, float v);
float         stack_pop(struct stack *);

```

Callers can use the API above without ever knowing the layout or even the size of `struct stack`. Only a pointer to the struct is ever needed. However, in order for this to work, the library must allocate the struct itself. If this is a concern, then the library will typically allow the caller to supply an allocator via function pointers. To see a really slick version of this in practice, check out [Lua’s `lua_Alloc`](https://www.lua.org/manual/5.3/manual.html#lua_Alloc), a single function allocator API.

Suppose we wanted to support something simpler: The library will advertise the size of the struct so the caller can allocate it.

```
/* API additions */
size_t stack_sizeof(void);
void   stack_init(struct stack *);  // like stack_create()
void   stack_free(struct stack *);  // like stack_destroy()

```

The implementation of `stack_sizeof()` would literally just be `return sizeof struct stack`. The caller might use it like so:

```
size_t len = stack_sizeof();
struct stack *s = malloc(len);
if (s) {
    stack_init(s);
    /* ... */
    stack_free(s);
    free(s);
}

```

However, that’s still a heap allocation. If this wasn’t an opaque struct, the caller could very naturally use automatic (i.e. stack) allocation, which is likely even preferred in this case. Is this still possible? Idea: Allocate it via a generic `char` array (VLA in this case).

```
size_t len = stack_sizeof();
char buf[len];
struct stack *s = (struct stack *)buf;
stack_init(s);

```

However, this is technically undefined behavior. While a `char` pointer is special and permitted to alias with anything, the inverse isn’t true. Pointers to other types don’t get a free pass to alias with a `char` array. Accessing a `char` value as if it were a different type just isn’t allowed. Why? Because the standard says so. If you want one of the practical reasons: the alignment might be incorrect.

Hmmm, is there another option? Maybe with `alloca()`!

```
size_t len = stack_sizeof();
struct stack *s = alloca(len);
stack_init(s);

```

Since `len` is expected to be small, it’s not any less safe than the non-opaque alternative. It doesn’t undermine the type system, either, since `alloca()` has the same semantics as `malloc()`. The downsides are:

- It’s not portable: `alloca()` is only a common extension, never standardized, and for good reason.
- This is still a dynamic stack allocation, so, like I showed in the last article, the function making this allocation becomes more complex. It must manage its own stack frame dynamically.

### Optimizing out `alloca()`?

The second issue can possibly be resolved if the size is available as a compile time constant. This starts to break the abstraction provided by opaque structs, but they’re still *mostly* opaque. For example:

```
/* API additions */
#define STACK_SIZE 24

/* In practice, this would likely be horrific #ifdef spaghetti! */

```

The caller might use it like this:

```
struct stack *s = alloca(STACK_SIZE);
stack_init(s);

```

Now the compiler can see the allocation size, and potentially optimize away the `alloca()`. As of this writing, Clang (all versions) can optimize these fixed-size `alloca()` usages, but GCC (9.2) still does not. Here’s a simple example:

```
#include

void
foo(void)
{
#ifdef ALLOCA
    volatile char *s = alloca(64);
#else
    volatile char s[64];
#endif
    s[63] = 0;
}

```

With the `char` array version, both GCC and Clang produce optimal code:

```
0000000000000000 :
   0:	c6 44 24 f8 00       	mov    BYTE PTR [rsp-0x1],0x0
   5:	c3                   	ret

```

Side note: This is on x86-64 Linux, which uses the System V ABI. The entire array falls within the [red zone](https://eli.thegreenplace.net/2011/09/06/stack-frame-layout-on-x86-64/), so it doesn’t need to be explicitly allocated.

With `-DALLOCA`, Clang does the same, but GCC does the allocation inefficiently as if it were dynamic:

```
0000000000000000 :
   0:	55                   	push   rbp
   1:	48 89 e5             	mov    rbp,rsp
   4:	48 83 ec 50          	sub    rsp,0x50
   8:	48 8d 44 24 0f       	lea    rax,[rsp+0xf]
   d:	48 83 e0 f0          	and    rax,0xfffffffffffffff0
  11:	c6 40 3f 00          	mov    BYTE PTR [rax+0x3f],0x0
  15:	c9                   	leave
  16:	c3                   	ret

```

It would make a slightly better case for `alloca()` here if GCC was better about optimizing it. Regardless, this is another neat little trick that I probably wouldn’t use in practice.

- [c](/tags/c/)
- [optimization](/tags/optimization/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Legitimate-ish%20Use%20of%20alloca()) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Legitimate-ish+Use+of+alloca%28%29).

« [Legitimate Use of Variable Length Arrays](/blog/2019/10/27/)

» [Infectious Executable Stacks](/blog/2019/11/15/)
