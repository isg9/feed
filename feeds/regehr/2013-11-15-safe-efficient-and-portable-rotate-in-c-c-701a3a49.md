---
title: Safe, Efficient, and Portable Rotate in C/C++
url: https://blog.regehr.org/archives/1063
published: "2013-11-15T20:10:44Z"
feed: regehr
guid: http://blog.regehr.org/?p=1063
---

# Safe, Efficient, and Portable Rotate in C/C++

Rotating a computer integer is just like shifting, except that when a bit falls off one end of the register, it is used to fill the vacated bit position at the other end. Rotate is used in encryption and decryption, so we want it to be fast. The obvious C/C++ code for left rotate is:

```
uint32_t rotl32a (uint32_t x, uint32_t n)
{
  return (x<<n) | (x>>(32-n));
}

```

Modern compilers will recognize this code and emit a rotate instruction, if available:

**```** **rotl32a:** **movb %sil, %cl** **roll %cl, %edi** **movl %edi, %eax** **ret**

**```**

The source and assembly code for right rotate are analogous.

Although this x86-64 rotate instruction will have the expected behavior (acting as a nop) when asked to rotate by 0 or 32 places, this C/C++ function must only be called with `n` in the range 1-31. Outside of that range, the code has undefined behavior due to shifting a 32-bit value by 32 places. In general, crypto codes are designed to avoid rotating 32-bit words by 32 places. However, as seen in my [last post](https://blog.regehr.org/archives/1054), the current versions of five of the 10 open source crypto libraries that I examined perform rotations by 0 places — a potentially serious bug because C/C++ compilers have unpredictable results when shifting past bitwidth. Bear in mind that the well-definedness of the eventual instruction does not rescue the situation: it is the source code that places (or in this case, fails to place) obligations upon the compiler.

Here’s a safer rotate function:

```
uint32_t rotl32b (uint32_t x, uint32_t n)
{
  assert (n<32);
  if (!n) return x;
  return (x<<n) | (x>>(32-n));
}

```

This one protects against the expected case where `n` is zero by avoiding the erroneous shift and also against the disallowed case of rotating by too many places. If we don’t want the overhead of the assert for production compiles, we can define NDEBUG in the preprocessor. The problem with this code is that (even with the assert compiled out) it contains a branch, which is sort of ugly. Clang generates the obvious branching code whereas GCC chooses to use a conditional move:

**```** **rotl32b:** **movl %edi, %eax** **movb %sil, %cl** **roll %cl, %eax** **testl %esi, %esi** **cmove %edi, %eax** **ret**

**```**

I figured that was the end of the story but then I saw this version:

```
uint32_t rotl32c (uint32_t x, uint32_t n)
{
  assert (n<32);
  return (x<<n) | (x>>(-n&31));
}

```

This one is branchless, meaning that it results in a single basic block of code, potentially making it easier to optimize. At the moment it defeats Clang’s rotate recognizer:

**```** **rotl32c:** **movl %esi, %ecx** **movl %edi, %eax** **shll %cl, %eax** **negl %ecx** **shrl %cl, %edi** **orl %eax, %edi** **movl %edi, %eax** **ret**

**```**

However, [they are working on this](http://llvm.org/bugs/show_bug.cgi?id=17332).

GCC (built today) has the desired output:

**```** **rotl32c:** **movl %edi, %eax** **movb %sil, %cl** **roll %cl, %eax** **ret**

**```**

Here is [their discussion of that issue](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=57157).

**Summary:** If you want to write portable C/C++ code (that is, you don’t want to use intrinsics or inline assembly) and you want to correctly deal with rotating by zero places, the `rotl32c()` function from this post, and its obvious right-rotate counterpart, are probably the best choices. GCC already generates excellent code and the LLVM people are working on it.
