---
title: Does Portable Byte-Order Code Optimize?
url: https://blog.regehr.org/archives/702
published: "2012-04-04T18:14:45Z"
feed: regehr
guid: http://blog.regehr.org/?p=702
---

# Does Portable Byte-Order Code Optimize?

When reading data whose byte-ordering may differ from the host computer’s, [Rob Pike advocates writing portable code](http://commandcenter.blogspot.com/2012/04/byte-order-fallacy.html) as opposed to using #ifdefs to optimize the case where the encoded data’s byte ordering matches the host. His arguments seem reasonable and it’s definitely a win if the compiler can transparently recognize the useless data extraction operations and optimize them away. Rob’s code is basically this:

```
int read_little_endian_int (unsigned char *data)
{
  int i =
    (data[0]<<0) | (data[1]<<8) |
    (data[2]<<16) | (data[3]<<24);
  return i;
}

```

On a little-endian host, this code can be simplified to just a 32-bit load. But do compilers actually perform this optimization? It turns out not. Clang (built today from trunk) gives this:

> ```
> read_little_endian_int:
>  movzbl (%rdi), %ecx
>  movzbl 1(%rdi), %eax
>  shll $8, %eax
>  orl %ecx, %eax
>  movzbl 2(%rdi), %ecx
>  shll $16, %ecx
>  orl %eax, %ecx
>  movzbl 3(%rdi), %eax
>  shll $24, %eax
>  orl %ecx, %eax
>  ret
> ```

GCC (built a few days ago) and Intel’s compiler (12.0.5) produce very similar code.

Does the sub-optimality of this code matter? Probably not if we’re reading from a slow device, but it does matter if we’re reading from RAM. My Core i7 920 (running in 64-bit mode) can copy a large block of data using 32-bit operations at about 7 GB/s, but reaches only about 2 GB/s when running the data through the code emitted by Clang. The results for GCC and Intel CC are similar.

But now the plot thickens slightly. If we wish to check that the function above is implemented and compiled correctly, we might write a program with this main function:

```
int main (void)
{
  int i = INT_MIN;
  while (1) {
    int i2 = read_little_endian_int ((unsigned char *)&i);
    if (i != i2) {
      printf ("oops %d != %d\n", i, i2);
    }
    if (i == INT_MAX) break;
    i++;
  }
  return 0;
}

```

GCC and Intel CC compile this in the obvious way, but Clang (at -O3) turns this main function into a nop:

> ```
>  main:
>  xorl %eax, %eax
>  ret
>
> ```

Now why would Clang understand that read\_little\_endian\_int is a nop when it is used in this context, but fail to emit the obvious code for the function itself? I’m not sure what to make of this. Maybe someone familiar with LLVM’s optimizers can help out here. The full code is [here](https://blog.regehr.org/extra_files/endian2.c) if anyone wants to play with it.
