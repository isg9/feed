---
title: Sometimes Compilers are Cute
url: https://blog.regehr.org/archives/1055
published: "2013-10-26T22:06:52Z"
feed: regehr
guid: http://blog.regehr.org/?p=1055
---

# Sometimes Compilers are Cute

```
#include <stdint.h>

uint32_t foo1 (uint32_t x, int r)
{
  return (x << r) | (x >> (32 - r));
}

uint32_t foo2 (uint32_t x, int r)
{
  return (x >> r) | (x << (32 - r));
}

uint32_t foo3 (uint32_t x)
{
  return
    (x << 24) |
    ((x & 0x0000ff00) <<  8) |
    ((x & 0x00ff0000) >>  8) |
    (x >> 24);
}

uint32_t foo4 (uint32_t x)
{
  char *xp = (char *)&x;
  uint32_t y;
  char *yp = (char *)&y;
  yp[0] = xp[3];
  yp[1] = xp[2];
  yp[2] = xp[1];
  yp[3] = xp[0];
  return y;
}

```

**```** **regehr@john-home ~ $ clang -Os -S -o - foo.c** **foo1:** **movb %sil, %cl** **roll %cl, %edi** **movl %edi, %eax** **ret**

**foo2:** **movb %sil, %cl** **rorl %cl, %edi** **movl %edi, %eax** **ret**

**foo3:** **bswapl %edi** **movl %edi, %eax** **ret**

**foo4:** **bswapl %edi** **movl %edi, %eax** **ret**

**regehr@john-home ~ $ gcc -Os -S -o - foo.c** **foo1:** **movl %edi, %eax** **movb %sil, %cl** **roll %cl, %eax** **ret**

**foo2:** **movl %edi, %eax** **movb %sil, %cl** **rorl %cl, %eax** **ret**

**foo3:** **movl %edi, %eax** **bswap %eax** **ret**

**foo4:** **movl %edi, -20(%rsp)** **movb -17(%rsp), %al** **movb %al, -4(%rsp)** **movb -18(%rsp), %al** **movb %al, -3(%rsp)** **movb -19(%rsp), %al** **movb %al, -2(%rsp)** **movb -20(%rsp), %al** **movb %al, -1(%rsp)** **movl -4(%rsp), %eax** **ret**

**```**

I’ve edited the asm to remove extraneous lines. I expect there’s probably a tweak I could use to get GCC to recognize the pointer-based endian swap but I didn’t find it.
