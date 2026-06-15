---
title: on-stack replacement in v8 — wingolog
url: https://wingolog.org/archives/2011/06/20/on-stack-replacement-in-v8
published: "2011-06-20T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/06/20/on-stack-replacement-in-v8
---

# on-stack replacement in v8 — wingolog

## [on-stack replacement in v8](/archives/2011/06/20/on-stack-replacement-in-v8)

20 June 2011 1:14 PM

- [igalia](/tags/igalia)
- [osr](/tags/osr)
- [v8](/tags/v8)
- [javascript](/tags/javascript)
- [goops](/tags/goops)
- [guile](/tags/guile)
- [compilers](/tags/compilers)

A [recent post](//wingolog.org/archives/2011/06/08/what-does-v8-do-with-that-loop) looked at what V8 does with a simple loop:

```
function g () { return 1; }
function f () {
  var ret = 0;
  for (var i = 1; i < 10000000; i++) {
    ret += g ();
  }
  return ret;
}

```

It turns out that calling `f` actually inlines `g` into the body, at runtime. But how does V8 do that, and what was the dead code about? V8 hacker Kasper Lund kindly chimed in with [an explanation](//wingolog.org/archives/2011/06/08/what-does-v8-do-with-that-loop#f6afc00bfdaf85a93bb13be0217251519ecc0cd3), which led me to write this post.

When V8 compiled an optimized version of `f` with `g`, it decided to do so when `f` was already in the loop. It needed to stop the loop in order to replace the slow version with the fast version. The mechanism that it uses is to reset the stack limit, causing the overflow handler in `g` to be called. So the computation is stuck in a call to `g`, and somehow must be transformed to a call to a combined `f+g`.

*\[Ed: It seems I flubbed some of the details here; the OSR of `f` is actually triggered by the interrupt check in the loop back-edge in `f`, not the method prologue in `g`. Thanks to V8 hacker Vyacheslav Egorov for pointing this out. OSR on one activation at a time makes the problem a bit more tractable, but unfortunately for me it invalidates some of the details of my lovely drawing. Truth, truly a harsh mistress!\]*

The actual mechanics are a bit complicated, so I'm going to try a picture. Here we go:

[![](//wingolog.org/pub/osr.png)](//wingolog.org/pub/osr.svg)

As you can see, V8 replaces the top of the current call stack with one in which the frames have been merged together. This is called *on-stack replacement* (OSR). OSR takes the contexts of some contiguous range of function applications and the variables that are live in those activations, and transforms those inputs into a new context or contexts and live variable set, and splats the new activations on the stack.

I'm using weasel words here because OSR can go both ways: optimizing (in which N frames are combined to 1) and deoptimizing (typically the other way around), as Hölzle notes in "Optimizing Dynamically-Dispatched Calls with Run-Time Type Feedback". More on deoptimization some other day, perhaps.

The diagram above mentions "loop restart entry points", which is indeed the "dead code" that I mentioned previously. That code forms part of an alternate entry point to the function, used only once, when the computation is restarted after on-stack replacement.

**details, [godly or otherwise](http://www.johndcook.com/blog/2008/03/30/god-is-in-the-details/)**

Given what I know now about OSR, I'm going to try an alternate decompilation of V8's `f+g` optimized function. This time I am going to abuse C as a high-level assembler. ( [I know, I know.](http://blog.llvm.org/2011/05/what-every-c-programmer-should-know.html) Play along with me :))

Let us start by defining our types.

```
typedef union { uint64_t bits; } Value;
typedef struct { Value map; uint64_t data[]; } Object;

```

All JavaScript values are of type Value. Some Values encode small integers (SMIs), and others encode pointers to Object. Here I'm going to assume we are on a 64-bit system. The `data` member of Object is a tail array of slots.

```
inline bool val_is_smi (Value v)
{ return !(v.bits & 0x1); }

inline Value val_from_smi (int32 i)
{ return (Value) { ((uint64_t)i) << 32 }; }

inline int32 val_to_smi (Value v)
{ return v.bits >> 32; }

```

Small integers have 0 as their least significant bit, and the payload in the upper 32 bits. Did you see the union literal syntax here? `(Value){ ((uint64_t)i) << 32 }`? It's a C99 thing that most folks don't know about. Anyway.

```
inline bool val_is_obj (Value v)
{ return !val_is_smi (v); }

inline Value val_from_obj (Object *p)
{ return (Value) { ((uint64_t)p) + 1U }; }

inline Object* val_to_obj (Value v)
{ return (Object*) (v.bits - 1U); }

```

Values that are Object pointers have 01 as their least significant bits. We have to mask off those bits to get the pointer to Object.

Numbers that are not small integers are stored as Object values, on the heap. All Object values have a `map` field, which points to a special object that describes the value: what type it is, how its fields are laid out, etc. [Much like GOOPS](//wingolog.org/archives/2008/04/11/allocate-memory-part-of-n), actually, though not as reflective.

In V8, the double value will be the only slot in a heap number. Therefore to access the double value of a heap number, we simply check whether the Object's map is the heap number map, and in that case return the first slot, as a double.

There is a complication though; what is value of the heap number map? V8 actually doesn't encode it into the compiled function directly. I'm not sure why it doesn't. Instead it stores the heap number map in a slot in the root object, and stores a pointer into the middle of the root object in `r13`. It's as if we had a global variable like this:

```
Value *root;    // r13

const int STACK_LIMIT_IDX = 0;
const int HEAP_NUMBER_MAP_IDX = -7; // Really.

inline bool obj_is_double (Object* o)
{ return o->map == root[HEAP_NUMBER_MAP_IDX]; }

```

Indeed, the offset into the root pointer is negative, which is a bit odd. But hey, details! Let's add the functions to actually get a double from an object:

```
union cvt { double d; uint64_t u; };

inline double obj_to_double (Object* o)
{ return ((union cvt) { o->data[0] }).d; }

inline Object* double_to_obj (double d)
{
  Object *ret = malloc (sizeof Value * 2);
  ret->map = root[HEAP_NUMBER_MAP_IDX];
  ret->data[0] = ((union cvt) { d }).u;
  return ret;
}

```

I'm including `double_to_obj` here just for completeness. Also did you enjoy the union literals in this block?

So, we're getting there. Perhaps the reader recalls, but V8's calling convention specifies that the context and the function are passed in registers. Let's model that in this silly code with some global variables:

```
Value context;  // rsi
Value function; // rdi

```

Recall also that when `f+g` is called, it needs to check that `g` is actually bound to the same value. So here we go:

```
const Value* g_cell;
const Value expected_g = (Value) { 0x7f7b205d7ba1U };

```

Finally, `f+g`. I'm going to show the main path all in one go. Take a big breath!

```
Value f_and_g (Value receiver)
{
  // Stack slots (5 of them)
  Value osr_receiver, osr_unused, osr_i, osr_ret, arg_context;
  // Dedicated registers
  register int64_t i, ret;

  arg_context = context;

  // Assuming the stack grows down.
  if (&arg_context > root[STACK_LIMIT_IDX])
    // The overflow handler knows how to inspect the stack.
    // It can longjmp(), or do other things.
    handle_overflow ();

  i = 0;
  ret = 0;

restart_after_osr:
  if (*g_cell != expected_g)
    goto deoptimize_1;

  while (i < 10000000)
    {
      register uint64_t tmp = ret + 1;
      if ((int64_t)tmp < 0)
        goto deoptimize_2;

      i++;
      ret = tmp;

      // Check for interrupt.
      if (&arg_context > root[STACK_LIMIT_IDX])
        handle_interrupt ();
    }

  return val_from_smi (ret);

```

And exhale. The receiver object is passed as an argument. There are five stack slots, none of which are really used in the main path. Two locals are allocated in registers: `i` and `ret`, as we mentioned before. There's the stack check, locals initialization, the check for `g`, and then the loop, and the return. The first test in the loop is intended to be a jump-on-overflow check.

In the original version of this article I had forgotten to include the interrupt handler; thanks again to Vyacheslav for pointing that out. Note that the calling the interrupt handler in a loop back-edge is more expensive than the handle\_overflow from the function's prologue; see line 306 of the assembly in my original article.

**`restart_after_osr` what?**

But what about OSR, and what's that label about? What happens is that when `f+g` is replaced on the stack, the computation needs to restart. It does so from the `osr_after_inlining_g` label:

```
osr_after_inlining_g:
  if (val_is_smi (osr_i))
    i = val_to_smi (osr_i);
  else
    {
      if (!obj_is_double (osr_i))
        goto deoptimize_3;

      double d = obj_to_double (val_to_obj (osr_i));

      i = (int64_t)trunc (d);
      if ((double) i != d || isnan (d))
        goto deoptimize_3;
    }

```

Here we take the value for the loop counter, as stored on the stack by OSR, and unpack it so that it is stored in the right register.

Unpacking a SMI into an integer register is simple. On the other hand unpacking a heap number has to check that the number has an integer value. I tried to represent that here with the C code. The actual assembly is just as hairy but a bit more compact; see the [article](//wingolog.org/archives/2011/06/08/what-does-v8-do-with-that-loop) for the full assembly listing.

The same thing happens for the other value that was live at OSR time, `ret`:

```
  if (val_is_smi (osr_ret))
    ret = val_to_smi (osr_ret);
  else
    {
      if (!obj_is_double (osr_ret))
        goto deoptimize_4;

      double d = obj_to_double (val_to_obj (osr_ret));

      ret = (int64_t)trunc (d);
      if ((double) ret != d || isnan (d))
        goto deoptimize_4;

      if (ret == 0 && signbit (d))
        goto deoptimize_4;
    }
  goto restart_after_osr;

```

Here we see the same thing, except that additionally there is a check for `-0.0` at the end, because V8 could not prove to itself that `ret` would not be `-0.0`. But if all succeeded, we jump back to the `restart_after_osr` case, and the loop proceeds.

Finally we can imagine some deoptimization bailouts, which would result in OSR, but for deoptimization instead of optimization. They are implemented as tail calls (jumps), so we can't represent them properly in C.

```
deoptimize_1:
  return deopt_1 ();
deoptimize_2:
  return deopt_2 ();
deoptimize_3:
  return deopt_3 ();
deoptimize_4:
  return deopt_4 ();
}

```

**and that's that.**

I think I've gnawed all the meat that's to be had off of this bone, so hopefully that's the last we'll see of this silly loop. Comments and corrections very much welcome. Next up will probably be a discussion of how V8's optimizer works. Happy hacking.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [v8: a tale of two compilers](/archives/2011/07/05/v8-a-tale-of-two-compilers)
- [what does v8 do with that loop?](/archives/2011/06/08/what-does-v8-do-with-that-loop)
- [pre-tenuring in v8](/archives/2026/01/05/pre-tenuring-in-v8)
- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [bigint shipping in firefox!](/archives/2019/05/23/bigint-shipping-in-firefox)

### 4 responses

1. Joe says:[20 June 2011 4:37 PM](#78a2fb2969b1b3f6cb102c7e271aa8777e31407f)

   You mentioned two PH.D dissertation on v8 in your previous post. I have been looking into v8, and have not seen these, do you have links to the two PH.D papers on V8?

   Great two posts, enjoy seeing lower level stuff, on such an important piece of software as V8.

   Thanks!

2. [Vyacheslav Egorov](http://mrale.ph) says:[20 June 2011 10:30 PM](#61c7ffe18717bd25ae30d5929110fb9e9bd6c15d)

   Minor correction:

   \> So the computation is stuck in a call to g, and somehow must be transformed to a call to a combined f+g

   V8 does not do OSR for several frames, it only does OSR for a single frame. So OSR is triggered while execution is in f on the back edge of the loop.

3. [Andy Wingo](http://wingolog.org) says:[21 June 2011 8:59 AM](#5b0668d54c764ad0bc7cdebf7ea5f0d4ce2a0a90)

   Thanks for stopping by, Vyacheslav, and thanks for the pointer. Indeed somehow I had forgotten the stack check from the original assembly code. I'll edit the post to put it in, and fix the error.

   Joe, I was actually referring to the Self papers. See the [self papers page](http://labs.oracle.com/self/papers/papers.html), and in particular, [Hölzle's thesis](http://research.sun.com/self/papers/hoelzle-thesis.ps.gz).

4. Willy says:[15 May 2015 7:19 AM](#79941b0c0f746af6533488594c0b22b1e84b5609)

   Thanks for your clear explanation.

   But one place I can't understand is

   // Stack slots (5 of them)

   Value osr\_receiver, osr\_unused, osr\_i, osr\_ret, arg\_context;

   Could you explain what is osr\_receiver, osr\_unused, ... used for ?

Comments are closed.
