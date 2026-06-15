---
title: Automated Reasoning About LLVM Optimizations and Undefined Behavior
url: https://blog.regehr.org/archives/1122
published: "2014-03-27T14:54:17Z"
feed: regehr
guid: http://blog.regehr.org/?p=1122
---

# Automated Reasoning About LLVM Optimizations and Undefined Behavior

Following up on [last week’s toy use of Z3](https://blog.regehr.org/archives/1120) and my [LLVM superoptimizer post](https://blog.regehr.org/archives/1109) from a few weeks ago, I wanted to try to prove things about optimizations that involve undefined behavior.

We’ll be working with the following example:

```
int triple (int x)
{
  return x + x + x;
}

```

Clang’s -O0 output for this code is cluttered but after running mem2reg we get a nice literal translation of the C code:

> **```** **define i32 @triple(i32 %x) #0 {** **%1 = add nsw i32 %x, %x** **%2 = add nsw i32 %1, %x** **ret i32 %2** **}**
>
> **```**

Next we can run instcombine, with this result:

> **```** **define i32 @triple(i32 %x) #0 {** **%1 = mul i32 %x, 3** **ret i32 %1** **}**
>
> **```**

The optimization itself (replacing x+x+x with x\*3) is uninteresting and proving that it is correct is trivial in Z3 or any similar tool that supports reasoning about bitvector operations. To see this, go to the [Z3 site](http://rise4fun.com/z3py/tutorial) and paste this code into the Python window on the right:

```
x = BitVec('x', 32)
slow = x+x+x
fast = x*3
prove (slow == fast)

```

The interesting aspect of this optimization is that the original C code and LLVM code are not using two’s complement addition. Rather, `+` in C and `add nsw` in LLVM are only defined when the addition doesn’t overflow. On the other hand, the multiplication emitted by instcombine is not qualified by `nsw`: it wraps in the expected fashion when the multiplication overflows ( [see the docs for more detail](http://llvm.org/docs/LangRef.html#mul-instruction)). My goal was to use Z3 to automatically prove that instcombine dropped the ball here: it could have emitted `mul nsw` which has weaker semantics and is therefore more desirable: compared to the unqualified `mul` it gives subsequent optimizations more freedom.

When we talk informally about compiler optimizations, we often say that the compiler should replace code with (faster or smaller) code that is functionally equivalent. Actually I just did this above while showing that `x+x+x` and `x*3` are equivalent. But in fact, a compiler optimization does not need to replace code with functionally equivalent code. Rather, the new code only needs to refine the old code. Refinement occurs many times during a typical compilation. For example this C code:

```
foo (bar(), baz());

```

can be refined to either this:

```
tmp1 = bar();
tmp2 = baz();
foo (tmp1, tmp2);

```

or this:

```
tmp2 = baz();
tmp1 = bar();
foo (tmp1, tmp2);

```

These refinements are not (in general) equivalent to each other, nor are they equivalent to the original non-deterministic C code. Analogously, the LLVM instruction `mul %x, 3` refines `mul nsw %x, 3` but not the other way around.

So how do we teach Z3 to do refinement proofs? I was a bit worried by some wording in the [Z3 documentation](http://rise4fun.com/z3/tutorialcontent/guide) that says “in Z3 all functions are total” and then I got stuck but [Alexey Solovyev](http://www.cs.utah.edu/~monad/), a formal methods postdoc at Utah, helped me out with the following solution. The idea is to use Z3’s ML-style option types to turn functions with undefined behavior into total functions:

```
BITWIDTH = 32

Opt = Datatype('Opt')
Opt.declare('none')
Opt.declare('some', ('value', BitVecSort(BITWIDTH)))

Opt = Opt.create()
none = Opt.none
some = Opt.some
value = Opt.value

(t, r) = Consts('t r', Opt)

s = Solver()

# print "sat" if f is refined by g, else print "unsat"
# equivalently, you can say "f can be implemented by calling g"
def check_refined(f, g):
    s.push()
    s.add(ForAll([t, r], Implies(f(t, r) != none, f(t, r) == g(t, r))))
    print s.check()
    s.pop()

```

In other words, f() is refined by g() iff g() returns the same value as f() for every member of f()’s domain. For inputs not in the domain of f(), the behavior of g() is a don’t-care.

Here are some LLVM operations that let us reason about the example code above:

```
INT_MIN = BitVecVal(1 << BITWIDTH - 1, BITWIDTH)
INT_MAX = BitVecVal((1 << BITWIDTH - 1) - 1, BITWIDTH)
INT_MIN_AS_LONG = SignExt(BITWIDTH, INT_MIN)
INT_MAX_AS_LONG = SignExt(BITWIDTH, INT_MAX)

def signed_add_overflow(y, z):
    ylong = SignExt(BITWIDTH, y)
    zlong = SignExt(BITWIDTH, z)
    rlong = ylong + zlong
    return Or(rlong > INT_MAX_AS_LONG, rlong < INT_MIN_AS_LONG)

def signed_mul_overflow(y, z):
    ylong = SignExt(BITWIDTH, y)
    zlong = SignExt(BITWIDTH, z)
    rlong = ylong * zlong
    return Or(rlong > INT_MAX_AS_LONG, rlong < INT_MIN_AS_LONG)

def add(t, r):
    return If(Or(t == none, r == none),
              none,
              some(value(t) + value(r)))

def add_nsw(t, r):
    return If(Or(t == none, r == none),
              none,
              If(signed_add_overflow(value(t), value(r)),
                 none,
                 some(value(t) + value(r))))

def mul(t, r):
    return If(Or(t == none, r == none),
              none,
              some(value(t) * value(r)))

def mul_nsw(t, r):
    return If(Or(t == none, r == none),
              none,
              If(signed_mul_overflow(value(t), value(r)),
                 none,
                 some(value(t) * value(r))))

```

Hopefully this code is pretty clear: `add_nsw` and its friend `mul_nsw` act like `add` and `mul` as long as there’s no overflow. If an overflow occurs, they map their arguments to `none`.

At this point I might as well mention a dirty little secret: formal methods code isn’t any easier to get right than regular code, and often it is considerably more difficult. The solution is testing. This is more than a little bit ironic since working around inadequacies of testing is the point of using formal methods in the first place. So here’s some test code. To make it easier to understand I’ve set BITWIDTH to 8 and also put the results of the print statements in comments.

```
def opt_str(o):
    p = simplify(o)
    if is_const(p):
        return "none"
    else:
        return simplify(value(p)).as_signed_long()

ONE = Opt.some(1)
TWO = Opt.some(2)
NEG_ONE = Opt.some(-1)
NEG_TWO = Opt.some(-2)
INT_MIN = Opt.some(INT_MIN)
INT_MAX = Opt.some(INT_MAX)

print opt_str(mul_nsw(ONE, ONE))           #    1
print opt_str(mul_nsw(INT_MAX, ONE))       #  127
print opt_str(mul_nsw(INT_MIN, ONE))       # -128
print opt_str(mul_nsw(INT_MAX, NEG_ONE))   # -127
print opt_str(mul_nsw(INT_MIN, NEG_ONE))   # none
print opt_str(mul_nsw(INT_MAX, TWO))       # none
print opt_str(mul_nsw(INT_MIN, TWO))       # none
print opt_str(mul_nsw(INT_MAX, NEG_TWO))   # none
print opt_str(mul_nsw(INT_MIN, NEG_TWO))   # none

print opt_str(add_nsw(ONE, ONE))           #    2
print opt_str(add_nsw(INT_MAX, ONE))       # none
print opt_str(add_nsw(INT_MIN, ONE))       # -127
print opt_str(add_nsw(INT_MAX, NEG_ONE))   #  126
print opt_str(add_nsw(INT_MIN, NEG_ONE))   # none

check_refined(add, add)                    # sat
check_refined(add, add_nsw)                # unsat

check_refined(add_nsw, add)                # sat
check_refined(add_nsw, add_nsw)            # sat

```

The formalization passes this quick smoke test. I’ve done some more testing and won’t bore you with it. None of this is conclusive of course.

Finally, we return to the example that motivated this post. Recall that LLVM’s instcombine pass translates two `add nsw` instructions into a single `mul`. Let’s make sure that was correct. Again, for convenience, I’ll put outputs in comments.

```
(a, b) = Consts('a b', Opt)

def slow(a, b):
    return add_nsw(a, add_nsw(a, a))

def fast1(a, b):
    return mul(a, Opt.some(3))

check_refined(slow, fast1)           # sat
check_refined(fast1, slow)           # unsat

```

As expected, LLVM’s transformation is correct: `x+x+x` where both `+` operators have undefined overflow can be refined by `x*3` where the `*` operator has two’s complement wraparound. The converse is not true.

Next, let’s explore the idea that instcombine could emit a `mul nsw` instruction.

```
def fast2(a, b):
    return mul_nsw(a, Opt.some(3))

check_refined(slow, fast2)           # sat
check_refined(fast2, slow)           # sat

```

Not only does this optimization work, but the refinement property also holds for the converse, indicating that the two formulations are equivalent.

We can also ask Z3 to prove equivalence of these different formulations:

```
prove (slow(a, b) == fast1(a, b))    # counterexample [a = some(137)]
prove (slow(a, b) == fast2(a, b))    # proved

```

As expected, `fast1` is not equivalent to the original code and additionally Z3 has given us a value for `a` that demonstrates this (for the 8-bit case). The second equivalence check passes.

What have we learned here? First, automated proofs about compiler optimizations in the presence of integer undefined behaviors are not too difficult. This is nice because manual reasoning about undefined behavior is error-prone. Second, there’s room for at least a little bit of improvement in LLVM’s instcombine pass (I’m sure the LLVM people know this). One idea that I quite like is using a solver to improve LLVM’s peephole optimizations by automatically figuring out when qualifiers like “nsw”, “nuw”, and “exact” can be used correctly.

Here’s the [Python code all in one piece](https://blog.regehr.org/extra_files/z3_llvm.txt). You can run it locally after installing Z3 or else you can paste it into the Z3 website’s Python interface.
