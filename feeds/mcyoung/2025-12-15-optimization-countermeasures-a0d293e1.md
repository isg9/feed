---
title: Optimization Countermeasures
url: /2025/12/15/value-barriers
published: "2025-12-15T00:00:00Z"
feed: mcyoung
guid: /2025/12/15/value-barriers
---

# Optimization Countermeasures

Silicon designers are bad at designing secure hardware. [Embarrassingly so](https://en.wikipedia.org/wiki/Spectre_(security_vulnerability)), sometimes. This means that low-level cryptography, as well as code which directly handles key material, often needs to be written in a particularly delicate style called “constant-time”.

“Constant-time” is a bit of a misnomer. It does not mean that the code’s time complexity is O(1) (although this is a closely related property). Constant-time is a threat model for side-channel timing attacks like Spectre, which ensures that key material is not leaked through the microarchitecture of CPUs.

Although constant-time is a powerful countermeasure against the silicon designers leaking our keys, the compiler can still screw us. However, there are magic incantations that can be offered to the compiler to make it behave correctly in many relevant situations.

First: what is constant-time?

## [The Constant-Time Threat Model](\#the-constant-time-threat-model)

The actual threat model is a series of assumptions about the most advanced attacker we wish to defeat. The assumptions, as applied to a cryptography software library, are as follows:

1. The attacker has access to both the source code of the library, the compiled artifact linking in the the library, the toolchain used to build it, and any relevant compiler flags (this is true, for example, for an Internet browser).

2. The attacker has a complete trace of every program counter value visited by the program. This is not the same as an instruction trace, which will usually also record software-visible architectural state at each instruction. Attackers can often obtain this information by directly timing the software, since the relevant information is mostly branches-taken. This includes instructions executed in a privileged mode, such as within the kernel.

3. The attacker knows the address of every pointer stored or loaded by the program. That is, each program counter value in the above is annotated with the values of registers containing pointers relevant to that instruction, such as the pointer for a load instruction. This data can be obtained through data cache side-channels such as Spectre.

Programming against this model defeats virtually all known timing side-channel attacks, although it does not protect against other side-channel attacks, such as thermal and power analysis.

### [New Footguns](\#new-footguns)

All cryptography libraries that are safe to use in 2025 implement all of their critical code in constant-time. This model has broad consequences what kinds of programs are allowed.

What’s wrong with this code?

```cpp highlight
int8* key = ...
for (int i = 0; i < n; ++i) {
  if (auto& k = key[i]; k < 0) {
    k = -k;
  }
}

```

[C++](#code:1)

Because we are looping over `n`, we immediately leak `n`, because the attacker can count the number of loop iterations. If `n` is secret, this is a problem.

We also leak one eight of the bits in the key: the attacker can see which loop iterations contain a negation instruction; those iterations correspond to bytes which had their sign bit set.

To protect the value of `n`, we would have to ensure that there is some maximum value `N` of `n`, and that `key` was allocated to an at-least- `N`-byte buffer; the loop would then be over `N`, making no reference to `n`. This is a relatively uncommon situation, since the length of a buffer is almost never a secret in practice.

Protecting the sign bits is simpler: rather than branching to determine if the value should be negated, we can mask off the sign bit: `key[i] &= 0x7f`.

### [Buffer Comparisons](\#buffer-comparisons)

Many standard library functions are not constant-time. For example, `memcmp`’s runtime is not constant in the size of its inputs; most implementations break out early when encountering an unequal value:

```cpp highlight
int memcmp(const char* a, const char* b, int n) {
  for (int i = 0; i < n; ++i) {
    auto diff = a[i] - b[i];
    if (diff != 0) {
      return diff
    }
  }
  return 0;
}

```

[C++](#code:2)

Now suppose we have the following code to verify a signature:

```cpp highlight
Message msg = ...;
char* expected = compute_signature(msg.body, private_key);
if (memcmp(msg.signature, expected) == 0) {
  // Assume msg is authentic.
}

```

[C++](#code:3)

The attacker can use this as a signing oracle. By providing sending the desired message to sign with a bad signature, they can determine the first byte of the signature which is incorrect by timing alone, and brute force that byte. They can then proceed to the next byte, and so on. The maximum number of queries to forge a signature is `256` times the number of bits, reducing the cost from O(2n) to O(n)!

To defeat this attack, we need to use constant-time `memcmp`. This means that every byte must be compared, and the comparison must be accumulated without branching.

The typical implementation is something like this:

```cpp highlight
bool ct_memeq(const char* a, const char* b, int n) {
  char acc = 0;
  for (int i = 0; i < n; ++i) {
    acc |= a[i] ^ b[i];
  }
  return acc == 0;
}

```

[C++](#code:4)

If at least one byte differs between `a` and `b`, their xor will be non-zero, and so `acc` will be nonzero. This function only compares for exact equality, not lexicographic equality, but the latter is never an operation you want or need on keys and other sensitive data.

There are many variants on this implementation, but the xor one is the most popular. Subtraction will achieve a similar result (although signed overflow is UB in C++).

### [Constant-Time Select](\#constant-time-select)

A similar trick is used to select one of two values. The ternary operator cannot be used because it can be compiled into a branch.

```cpp highlight
template <typename T>
T ct_select(bool flag, T a, T b) {
  auto mask = -T(flag);
  return (a & mask) | (b & ~mask);
}

```

[C++](#code:ct_select)

If `flag` is true, `mask` is the all-ones representation for `T`, so `a & mask` is `a` and `b & ~mask` is `0`, so their or is `a`. If `flag` is false, the opposite is true.

On x86, Clang recognizes this idiom and produces a conditional move instruction, which does not violate the threat model. However, on architectures without conditional move, such as RISC-V, Clang still recognizes the pattern… but produces a branch.

```x86 highlight
_Z9ct_selectIiET_bS0_S0_:
  mov     eax, esi
  test    edi, edi
  cmove   eax, edx
  ret

```

[x86 Assembly](#code:5)

```riscv highlight
_Z9ct_selectIiET_bS0_S0_:
  bnez    a0, .LBB0_2
  mv      a1, a2
.LBB0_2:
  mv      a0, a1
  ret

```

[RISC-V Assembly](#code:6)

You might want to point the finger at the `bool`, but this actually goes much deeper than that. This type of unwanted optimization happens all the time in constant-time code, and preventing it is essential to ensure that the countermeasure works.

The “easy” solution is to just write the assembly directly, which is the case for performance-critical parts of cryptography implementations, but this is error-prone and not portable.

Thankfully, all modern native compilers provide a hidden feature to block optimizations inimical to security, and what this post is really about: the *value barrier*.

## [An Incantation](\#an-incantation)

The value barrier is a special secret syntax construct that instructs the compiler to ignore information it could use to prove the correctness of optimizations.

For example, the reason Clang is able to “defeat” our [`ct_select` implementation](#code:ct_select) is that it knows from `-T(flag)` that `mask` must be either `0` or `-1`. If we can hide this fact from Clang, it will be forced to not emit the branch.

One way we can do this is to “encrypt” `mask` with a value that Clang cannot see through, such as a global variable:

```cpp highlight
template <typename T>
inline T key;

template <typename T>
T ct_select(bool flag, T a, T b) {
  auto mask = -T(flag);
  mask ^= key<T>;
  return (a & mask) | (b & ~mask);
}

```

[C++](#code:xor-barrier)

Clang cannot know what the value of `key` will be at runtime, because code in another translation unit might set it. It must therefore assume it knows nothing about `mask` and the return value could be an arbitrary bit-mix of `a` and `b`. However, because we never actually set this global, it will always be zero, so `mask ^= key<T>;` is a no-op at runtime.

On RISC-V, this does what we need:

```riscv highlight
_Z9ct_selectIiET_bS0_S0_:
.Lpcrel_hi0:
  auipc   a3, %pcrel_hi(_Z3keyIiE)
  lw      a3, %pcrel_lo(.Lpcrel_hi0)(a3)
  neg     a0, a0
  xor     a0, a0, a3
  xor     a1, a1, a2
  and     a0, a0, a1
  xor     a0, a0, a2
  ret

_Z3keyIiE:
  .word   0

```

[RISC-V Assembly](#code:7)

Unfortunately, this does perform a load, which is a performance hiccup we’d like to avoid. Also, if whole-program optimization, such as LTO, BOLT, or similar notices the global is never written to, it can replace all of its loads with immediates.

Another option is to send the value into an assembly function. Not even LTO can see into assembly functions, and BOLT will not attempt to optimize hand-written assembly.

```cpp highlight
#include <stdint.h>

asm(R"(
  .intel_syntax
  __asm_eyepatch:
    mov rax, rsi
    ret
)");

extern "C" uint64_t __asm_eyepatch(uint64_t);

template <typename T>
T ct_select(bool flag, T a, T b) {
  auto mask = -T(flag);
  mask = __asm_eyepatch(mask);
  return (a & mask) | (b & ~mask);
}

```

[C++](#code:8)

This works but now the cost is a non-inlineable function call rather than a pointer load. Not ideal. It’s also again, not portable across targets. The function call is also treated as having side-effects by the compiler, which impedes desirable optimizations.

But a small modification will eliminate this problem: we can simply use an inline assembly block with zero instructions.

```cpp highlight
template <typename T>
T ct_select(bool flag, T a, T b) {
  auto mask = -T(flag);
  asm("" : "+r"(mask));
  return (a & mask) | (b & ~mask);
}

```

[C++](#code:9)

This is called the value barrier: an empty assembly block which modifies a single register-sized value in-place as a no-op. The `"+r"` constraint indicates that the assembly takes `mask` as an input, and outputs a result onto `mask`, in the same register.

This has the same effect as the xor-with-key solution (prevents optimizations that depend on the value of `mask`) without the load from a global. It’s architecture-independent, because `""` is a valid inline assembly block regardless of underlying assembly language.

The value barrier is often written as its own function (with a massive comment explaining what it does), to be used in key points in cryptographic algorithms.

```cpp highlight
template <typename T>
[:nodiscard:] T value_barrier(T x) {
  asm("" : "+r"(x));
  return x;
}

```

[C++](#code:10)

A use of the value barrier looks like this:

```cpp highlight
template <typename T>
T ct_select(bool flag, T a, T b) {
  auto mask = -T(flag);
  mask = value_barrier(mask);
  return (a & mask) | (b & ~mask);
}

```

[C++](#code:11)

`value_barrier` is trivially inlineable and compiles down to zero instructions, but has a profound effect on optimizations.

## [Dataflow](\#dataflow)

A better name for the value barrier is the “dataflow barrier”, because that more accurately captures what the instruction does.

If you read my [introduction to SSA](${site.Page(\), you’ll know that every optimization compiler today puts a heavy emphasis on dataflow analysis. To figure out how to optimize some operation, we look at its inputs’ definitions.

Let’s look at what LLVM sees when we compile [`ct_select`](#code:ct_select), setting `T = int` (I have manually lifted everything into registers).

```llvm highlight
define i32 @_Z9ct_selectIiET_bS0_S0_(i1 zeroext %flag, i32 %a, i32 %2) {
  %1 = zext i1 %flag to i32  ; int(flag)
  %2 = sub nsw i32 0, %1     ; mask = 0 - int(flag)

  %3 = and i32 %a, %2     ; a & mask
  %4 = xor i32 %2, -1     ; mask ^ -1
  %5 = and i32 %b, %4     ; b & (mask ^ -1)
  %6 = or i32 %3, %5      ; (a & mask) | (b & (mask ^ -1))

  ret i32 %6
}

```

[LLVM IR](#code:12)

LLVM contains pattern-matching code that matches this code (and various permutations of it) as a “select” idiom. LLVM contains an instruction that selects one of two values based on an `i1`, called `select`. It is essentially the C ternary with SSA register arguments.

The pattern-matching code looks for an `or` whose arguments are both `and` s, where one argument to the `and` is the complement of the other (i.e, xor with `-1`). Call this argument is the “mask”. What LLVM has found is a general “bit-mixing” operations which selects bits from the other operands to the `and` s, depending on which bits of the mask are set.

LLVM then wants to prove that the mask is either `0` or `-1` (all ones). There are a number of ways LLVM can discover this, but all of them essentially boil down to “is the mask a `sext i1`, i.e., sign-extending a one-bit value”. That does not occur in this code, but a peephole optimization can rewrite `sub nsw i32 0, %1` into `sext i1 %0 to i32`, allowing this more complicated pattern to be detected.

LLVM rewrites this into a `select`, which on x86 turns into a `cmov`, but on RISC-V forces a branch.

All we need to do is prevent LLVM from looking through to the definition of the “mask”. That is precisely what the value barrier accomplishes: it inserts a new SSA register whose value is, at runtime, equivalent to the mask, but produced by an instruction that LLVM doesn’t have implement dataflow for. That is the value barrier: an intentional hole left in the compiler’s analysis.

In theory, LLVM can actually see that the inline assembly block is empty and optimize it out. However, it does not by design, because cryptography depends on it remaining an optimization barrier. In fact, LLVM (and GCC) emit special annotations around inline assembly to stop downstream tools, such as linker optimizations and post-link optimizers like BOLT, from accidentally optimizing sensitive sequences. In an assembly dump from LLVM, those regions look like this:

```cpp highlight
_Z9ct_selectIiET_bS0_S0_:
  mov     eax, esi
  neg     edi
  #APP
  #NO_APP
  xor     eax, edx
  and     eax, edi
  xor     eax, edx
  ret

```

[C++](#code:13)

These are just comments; the actual sensitive regions are recorded elsewhere in the resulting object code.

### [What Does the Barrier Do?](\#what-does-the-barrier-do)

The programming model for the value barrier is simple: it produces an arbitrary value that, at runtime, happens to be bitwise-identical to its input. The compiler may still make assumptions about the input value, but it cannot connect them to the output of the barrier through dataflow.

In other words, the value barrier is simply a register copy that also severs the dataflow link from the destination to the source operand.

The compiler is still allowed to optimized based on the assumption that this is some unknown concrete value. For example:

```cpp highlight
int random() {
  int x = value_barrier(42);
  return x - x;
}

```

[C++](#code:14)

`x - x` is always zero, so LLVM can optimize away the value barrier and its input altogether. Critically, the value barrier *does not have side effects*, so if its result is not used, the value barrier will be deleted through dead code elimination. The following does not work:

```cpp highlight
template <typename T>
T ct_select(bool flag, T a, T b) {
  auto mask = -T(flag);
  value_barrier(mask); // Unused result warning.
  return (a & mask) | (b & ~mask);
}

```

[C++](#code:15)

Because it is side-effect-free, it can also be hoisted out of loops and conditionals, which is actually a desirable optimization.

### [Benchmark Black Box](\#benchmark-black-box)

There is a related function that appears in benchmarking code, which is not equivalent to the value barrier:

```cpp highlight
template <typename T>
void black_box(const T& v) {
  asm volatile("" :: "m"(v))
}

```

[C++](#code:16)

This function guarantees that its input is treated as “used” by the compiler, even if it is side-effect free. Rather than blocking dataflow, it blocks dead code elimination.

It works because `asm volatile` must be treated as having observable side-effects that depend on all of its inputs. This means that it cannot be deleted, hoisted our of loops, or executed speculatively.

A benchmark black box is most commonly used to force the result of a function being benchmarked to not be deleted by the compiler, so that the runtime of that function can be accurately measured. It *can* in place of the value barrier, because the `"m"` constraint passes a pointer to the argument into the assembly block, which it could potentially mutate. However, this is not guaranteed to work in the same way that the value barrier does: it depends on whether the surface language (C++ or Rust) considers mutating through that pointer to be UB.

Using the actual value barrier avoids this altogether.

### [Unintended Consequences](\#unintended-consequences)

The value barrier works very well at blocking undesirable optimizations, but it obstructs a few important ones, making its use problematic in performance-sensitive scenarios.

A particularly notable one is automatic vectorization. Consider this function:

```cpp highlight
uint64_t sum(uint64_t* p, int n) {
  n &= ~31; // Round n to the nearest multiple of 32.
  u64 a = 0;
  for (int i = 0; i < n; i++) {
    a += p[i];
  }
  return a;
}

```

[C++](#code:17)

When we push this through Clang at `-O3 -march=zenv5`, the loop gets unrolled 32 times over into four parallel AVX512 operations.

```x86 highlight
_Z3sumPyi:
  cmp     esi, 32
  jl      .LBB1_1
  mov     eax, 6661
  vpxor   xmm0, xmm0, xmm0
  xor     ecx, ecx
  vpxor   xmm1, xmm1, xmm1
  vpxor   xmm2, xmm2, xmm2
  vpxor   xmm3, xmm3, xmm3
  bextr   eax, esi, eax
  shl     rax, 8
.LBB1_3:
  vpaddq  zmm0, zmm0, zmmword ptr [rdi + rcx]
  vpaddq  zmm1, zmm1, zmmword ptr [rdi + rcx + 64]
  vpaddq  zmm2, zmm2, zmmword ptr [rdi + rcx + 128]
  vpaddq  zmm3, zmm3, zmmword ptr [rdi + rcx + 192]
  add     rcx, 256
  cmp     rax, rcx
  jne     .LBB1_3
  vpaddq  zmm0, zmm1, zmm0
  vpaddq  zmm2, zmm3, zmm2
  vpaddq  zmm0, zmm2, zmm0
  vextracti64x4   ymm1, zmm0, 1
  vpaddq  zmm0, zmm0, zmm1
  vextracti128    xmm1, ymm0, 1
  vpaddq  xmm0, xmm0, xmm1
  vpshufd xmm1, xmm0, 238
  vpaddq  xmm0, xmm0, xmm1
  vmovq   rax, xmm0
  vzeroupper
  ret
.LBB1_1:
  xor     eax, eax
  ret

```

[x86 Assembly](#code:18)

Pretty cool, right? But what if we need to stick a value barrier into the loaded values for some reason?

```cpp highlight
uint64_t sum(uint64_t* p, int n) {
  n &= ~31; // Round n to the nearest multiple of 32.
  u64 a = 0;
  for (int i = 0; i < n; i++) {
    a += value_barrier(p[i]);
  }
  return a;
}

```

[C++](#code:slow-sum)

Surprisingly, this seems to completely wreck vectorization, forcing each loop iteration to load only 8 bytes instead of 256.

```cpp highlight
_Z3sumPyi:
  cmp     esi, 32
  jl      .LBB2_1
  mov     eax, 6661
  xor     edx, edx
  bextr   ecx, esi, eax
  xor     eax, eax
  shl     rcx, 8
.LBB2_4:
  mov     rsi, qword ptr [rdi + rdx]
  add     rax, rsi
  add     rdx, 8
  cmp     rcx, rdx
  jne     .LBB2_4
  ret
.LBB2_1:
  xor     eax, eax
  ret

```

[C++](#code:19)

The reason for this is kind of nasty. From LLVM’s perspective, an inline assembly block is a `call asm` instruction to a special function whose name is the inline assembly string. Although the lack of `volatile` marks this function as pure, making it reorderable, there is no way to tell LLVM that it can be merged.

If we try to unroll the loop 32 times, we wind up with 32 calls to a pure function with the signature `i64 -> i64`, and LLVM doesn’t know that it’s safe to merge them into an almost identical function call with a signature `<32 x i64> -> <32 x i64>`. Because one of the loop operations cannot be vectorized (namely, this inline assembly block), vectorization fails.

There is no workaround for this. Allowing the inline assembly block to use an SSE register using the `"+x"` constraint doesn’t work, because the failure is *not* the assembly constraints: it’s that the `call asm` instruction does not support vectorization[1](#fn:1).

## [Is This Really a Language Feature?](\#is-this-really-a-language-feature)

For all intents and purposes, this magic incantation is part of C++ (at least, the dialect that GCC and Clang implement, which are the only compilers that matter for security[2](#fn:2)).

This is because BoringSSL[3](#fn:3), the most widely-deployed cryptography library in the world, [relies on this trick](https://boringssl.googlesource.com/boringssl/+/15b1f9c6a4f9656e7c172e03064fe7f8e03c666d/crypto/internal.h#343). This feature is critical to the security posture of the two biggest orgs that fund LLVM: Google and Apple. Tools which break the value barrier have historically been quickly patched to respect it, usually due to the consternation of a professional cryptographer.

The optimization black box is not quite as critical, but its correctness is a side-effect of the value barrier’s: namely, that the optimizer must never peek into an inline assembly block.

Of course, any professional cryptographer who has to implement constant-time primitives would tell you that this is a crummy workaround, and that we really need actual language-level intrinsics for manipulating values in a constant-time way. The tricky part is specifying semantics for them in a way that makes sure we don’t allow “bad” optimizations, a notoriously difficult problem in compiler IR design.

### [Postscript](\#postscript)

Shortly after publishing this piece today, I got an email informing me that Clang 22 is planning to land `__builtin_ct_select`, an intrinsic that implements constant-time select in an optimization-friendly way. It will never be emitted as a branch, but it lowers to a `cmov` on x86 and a `csel` on aarch64. More on this new intrinsic can be found [here](https://blog.trailofbits.com/2025/12/02/introducing-constant-time-support-for-llvm-to-protect-cryptographic-code/). This posts suggests that there may be future improvements, such as a true “secret integer” type that always guarantees constant-time operations. I’m surprised that neither I nor David Benjamin (the maintainer of BoringSSL) knew about this LLVM RFC.

While discussing this article with David, I also realized that the [version of `sum` with a barrier](#code:slow-sum) can actually be written in such a way that the barrier is present without breaking vectorization. If we use the [xor-with-key](#code:xor-barrier) variant of the value barrier, we can still get the optimizations we want:

```cpp highlight
uint64_t __ct_key; // Must not be static or const!

uint64_t sum(uint64_t* p, int n) {
  n &= ~31; // Round n to the nearest multiple of 32.
  u64 a = 0;
  for (int i = 0; i < n; i++) {
    a += p[i] ^ __ct_key;
  }
  return a;
}

```

[C++](#code:20)

However, I’m fairly certain that LTO and post-link optimization such as BOLT can potentially shred this barrier, and it comes at the cost of a vectorized xor each unrolled loop iteration[4](#fn:4).

---

1. This could be fixed by making `asm("")` vectorizable, because it *is* in the sense that a no-op is vectorizable. However, it’s not immediately clear to me if this is safe, even though it feels like it aught to be. [↩︎](#fnref:1)

2. For example: all Internet browsers today are built with Clang, because Chromium only supports Clang builds and Firefox and Safari use Clang because Rust and Apple, respectively. [↩︎](#fnref:2)

3. The only browsers not shipping BoringSSL are Firefox and Safari; Chromium and all of its derivatives use it. It is also deployed on every Android device (not just the most popular mobile OS: horrible little Android devices, like POS equipment, are everywhere), and, of course, within all of Google’s data centers. Being on every Android device is probably enough to make the claim of “most widely deployed”, but being on 90%+ of consumer equipment, by virtue of Google Chrome being so popular, definitely makes it enough. [↩︎](#fnref:3)

4. The load from `[rip + __ct_key@GOTPCREL]` gets hoisted out of the loop, thanks to the fact that C++ can safely assume that no writes to `__ct_key` occur on other threads, thanks to `__ct_key` being non-atomic and data races resulting in Undefined Behavior. [↩︎](#fnref:4)
