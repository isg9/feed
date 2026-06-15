---
title: Comparing Compiler Optimizations
url: https://blog.regehr.org/archives/320
published: "2010-12-11T21:08:20Z"
feed: regehr
guid: http://blog.regehr.org/?p=320
---

# Comparing Compiler Optimizations

\[Update from Dec 14: Some of these problems have already been fixed! [Details here](../archives/324).\]

\[This is a guest post by my grad student [Yang Chen](http://www.cs.utah.edu/~chenyang/), describing one of the projects he’s been working on lately. I elaborated on some of Yang’s points, edited, and did some formatting for WordPress.\]

Our goal is to help C compiler developers find areas where their optimizer could stand to be improved. We do this by taking some open source applications and, for each application:

1. Select a number of benchmark functions. For now, all benchmarks are leaves: they make no function calls. There are a few other restrictions too.
2. Instrument the source code so that all inputs to benchmark functions (including those accessed through pointers) are logged.
3. Run whatever test suite comes with the application, logging inputs to benchmark functions. Logging stops after a configurable maximum number of distinct inputs has been seen.
4. Extract standalone versions of the benchmark functions.
5. For each compiler under test, compile each extracted benchmark with a test harness. Run the resulting code on an otherwise quiescent machine in order to measure the performance of the benchmark function in isolation.
6. Look for outlier functions where one compiler created significantly faster code than another compiler — these are potentially interesting and we inspect them manually. Some examples are below.

When we report the number of cycles required to execute a function, this is an average over all logged inputs.

The test machine is built around an Intel Core i7-920 processor, which has 4 cores running at 2.67GHz and 8MB of shared L3 cache. It has 6 GB of RAM. It runs the x86-64 version of Ubuntu 9.10. Performance is measured using the RDTSC instruction. The test process is pinned to a single processor and all CPU and OS power management functionality is turned off.

The compiler options we used are intended to:

1. Optimize for maximum speed
2. Give the tools a fighting chance at making equivalent assumptions about the hardware platform

# Clang

Svn head built on Dec 4, 2010:

> $ clang -v
>
> clang version 2.9 (trunk 120838)
>
> Target: x86\_64-unknown-linux-gnu
>
> Thread model: posix

Invocation:

> clang -O3 -mssse3 -march=core2 -mtune=core2 -fomit-frame-pointer -fno-stack-protector -w

# GCC

Svn head built on Dec 4, 2010:

> $ current-gcc -v
>
> Using built-in specs.
>
> COLLECT\_GCC=./compilers/bin/current-gcc
>
> COLLECT\_LTO\_WRAPPER=/uusoc/exports/scratch/chenyang/performance\_tests/compilers/libexec/gcc/x86\_64-unknown-linux-gnu/4.6.0/lto-wrapper
>
> Target: x86\_64-unknown-linux-gnu
>
> Configured with: ../configure –prefix=/uusoc/exports/scratch/chenyang/performance\_tests/compilers –program-prefix=current- –enable-lto –enable-languages=c,c++,lto
>
> Thread model: posix
>
> gcc version 4.6.0 20101204 (experimental) (GCC)

Invocation:

> current-gcc -O3 -mfpmath=sse -mssse3 -march=core2 -mtune=core2 -fno-stack-protector -fomit-frame-pointer -w

# Intel

Version is:

> $ icc -V
>
> Intel(R) C Intel(R) 64 Compiler Professional for applications running on Intel(R) 64, Version 11.1 Build 20100414
>
> Copyright (C) 1985-2010 Intel Corporation. All rights reserved.
>
> FOR NON-COMMERCIAL USE ONLY

Invocation:

> icc -O3 -static -not-prec-div -xHost -mssse3 -fno-stack-protector -fomit-frame-pointer -w

# Example 1

(Here and in other examples, we have edited the assembly code for clarity. Also note that we are showing the source code as seen by the compilers being tested. It is not only preprocesssed, but also has been parsed by our benchmark extraction tool which is based on [CIL](http://www.cs.berkeley.edu/~necula/cil/). CIL does a few simple transformations that make C code a lot uglier sometimes.)

This function is setboundaries() from zic.c in postgresql-9.0.0.

```
 1 typedef long int64;
 2 typedef int64 zic_t;
 3 void setboundaries (void);
 4 zic_t max_time;
 5 zic_t min_time;
 6 void
 7 setboundaries (void)
 8 {
 9   int i;
10
11   min_time = -1L;
12   i = 0;
13   while (i < 63)
14     {
15       min_time *= 2L;
16       i++;
17     }
18   max_time = -(min_time + 1L);
19   return;
20 }

```

GCC and Intel CC generate code that executes in 124 cycles and 160 cycles, respectively. Clang’s output, on the other hand, requires only 4 cycles because it can statically evaluate the loop. GCC and ICC can statically evaluate many loops, but not this one.

GCC output:

```
setboundaries:
        movl    $63, %eax
        movq    $-1, %rdx
.L2:    addq    %rdx, %rdx
        subl    $1, %eax
        jne     .L2
        movq    %rdx, min_time(%rip)
        notq    %rdx
        movq    %rdx, max_time(%rip)
        ret

```

Clang output:

```
setboundaries:
        movabsq $-9223372036854775808, %rax # imm = 0x8000000000000000
        movq    %rax, min_time(%rip)
        movabsq $9223372036854775807, %rax # imm = 0x7FFFFFFFFFFFFFFF
        movq    %rax, max_time(%rip)
        ret

```

# Example 2

This function is mad\_synth\_mute() from synth.c in mad-0.14.2b from the [MiBench](http://www.eecs.umich.edu/mibench/) embedded benchmark suite.

```
 1 typedef int mad_fixed_t;
 2 struct mad_pcm
 3 {
 4   unsigned int samplerate;
 5   unsigned short channels;
 6   unsigned short length;
 7   mad_fixed_t samples[2][1152];
 8 };
 9 struct mad_synth
10 {
11   mad_fixed_t filter[2][2][2][16][8];
12   unsigned int phase;
13   struct mad_pcm pcm;
14 };
15 void mad_synth_mute (struct mad_synth *synth);
16 void
17 mad_synth_mute (struct mad_synth *synth)
18 {
19   unsigned int ch;
20   unsigned int s;
21   unsigned int v;
22
23   ch = 0U;
24   while (ch < 2U)
25     {
26       s = 0U;
27       while (s < 16U)
28 	{
29 	  v = 0U;
30 	  while (v < 8U)
31 	    {
32 	      synth->filter[ch][1][1][s][v] = 0;
33 	      synth->filter[ch][1][0][s][v] = 0;
34 	      synth->filter[ch][0][1][s][v] = 0;
35 	      synth->filter[ch][0][0][s][v] = 0;
36 	      v++;
37 	    }
38 	  s++;
39 	}
40       ch++;
41     }
42   return;
43 }

```

The questions here are: Can the compiler recognize that the loops are equivalent to a bzero()? And, how fast of a bzero() can be generated?

GCC’s code executes in 256 cycles because the loop is completely unrolled and SSE instructions are used. Intel CC’s code uses SSE but does less unrolling, and takes 308. Clang produces an obvious translation of the source code — locality is poor and 1032 cycles are required.

GCC’s code:

```
mad_synth_mute:
        pxor    %xmm0, %xmm0
        movdqu  %xmm0, (%rdi)
        movdqu  %xmm0, 16(%rdi)
        movdqu  %xmm0, 32(%rdi)
        movdqu  %xmm0, 48(%rdi)
        movdqu  %xmm0, 64(%rdi)
        ... (obvious sequence of movdqu instructions elided) ....
        movdqu  %xmm0, 3936(%rdi)
        movdqu  %xmm0, 3952(%rdi)
        movdqu  %xmm0, 3968(%rdi)
        movdqu  %xmm0, 3984(%rdi)
        movdqu  %xmm0, 4064(%rdi)
        movdqu  %xmm0, 4080(%rdi)
        ret

```

# Example 3

This is an initialization function taken from pyexpat.c in Python 2.7.

```
 1 char template_buffer[257];
 2 void
 3 init_template_buffer (void)
 4 {
 5   int i;
 6
 7   i = 0;
 8   while (i < 256)
 9     {
10       template_buffer[i] = (char) i;
11       i++;
12     }
13   template_buffer[256] = (char) 0;
14   return;
15 }

```

Here Clang generates the obvious looping code requiring 532 cycles: 2 cycles per loop iteration plus some overhead. GCC generates very good SSE code requiring 86 cycles. Intel CC generates code that executes in 18 cycles:

```
init_template_buffer:
        movl      $269488144, %eax                              #6.3
        movd      %eax, %xmm0                                   #6.3
        pshufd    $0, %xmm0, %xmm1                              #6.3
        movdqa    _2il0floatpacket.0(%rip), %xmm0               #6.3
        xorl      %eax, %eax                                    #6.3
..B1.2:                         # Preds ..B1.2 ..B1.1
        movdqa    %xmm0, template_buffer(%rax)                  #7.5
        paddb     %xmm1, %xmm0                                  #7.5
        movdqa    %xmm0, 16+template_buffer(%rax)               #7.5
        paddb     %xmm1, %xmm0                                  #7.5
        movdqa    %xmm0, 32+template_buffer(%rax)               #7.5
        paddb     %xmm1, %xmm0                                  #7.5
        movdqa    %xmm0, 48+template_buffer(%rax)               #7.5
        paddb     %xmm1, %xmm0                                  #7.5
        movdqa    %xmm0, 64+template_buffer(%rax)               #7.5
        paddb     %xmm1, %xmm0                                  #7.5
        movdqa    %xmm0, 80+template_buffer(%rax)               #7.5
        paddb     %xmm1, %xmm0                                  #7.5
        movdqa    %xmm0, 96+template_buffer(%rax)               #7.5
        paddb     %xmm1, %xmm0                                  #7.5
        movdqa    %xmm0, 112+template_buffer(%rax)              #7.5
        paddb     %xmm1, %xmm0                                  #7.5
        addq      $128, %rax                                    #6.3
        cmpq      $256, %rax                                    #6.3
        jb        ..B1.2        # Prob 99%                      #6.3
        movb      $0, 256+template_buffer(%rip)                 #10.3
        ret                                                     #11.3
```

Here ICC has performed an impressive 16-way vectorization and 16-way unrolling; the result takes a bit of work to understand.

The 269488144 loaded into eax is 0x10101010. The pshufd instruction ( [shuffle packed doublewords](http://siyobik.info/index.php?module=x86&id=254)) loads the value 0x10101010 10101010 10101010 10101010 into xmm1. The next instruction loads xmm0 with 0x0f0e0d0c 0b0a0908 07060504 03020100.

Now the purpose of the first movdqa in the loop is clear: it loads the values {0,1,2, …, 15} into the first 16 bytes of  template\_buffer. The paddb (packed add, byte-wise) adds 0x10 to each byte of xmm0, leaving its contents 0x1f1e1d1c 1b1a1918 17161514 13121110. These are moved into the next 16 bytes of template\_buffer, and so on. The loop body executes only twice before the entire array is initialized.

Although this translation seems very fragile, icc can generate equivalent object code for most versions of this code where N, X, and Y are constants:

> ```
> for (i=0; i<N; i++) a[i] = iX+Y;
> ```

# Example 4

This function is tf\_bH() from \_ctypes\_test.c in Python 2.7.

```
 1 unsigned long long last_tf_arg_u;
 2 unsigned short
 3 tf_bH (signed char x, unsigned short c)
 4 {
 5   last_tf_arg_u = (unsigned long long) c;
 6   return ((unsigned short) ((int) c / 3));
 7 }

```

Here GCC and Intel CC both generate code that executes in 6 cycles, whereas Clang’s takes 18.

GCC’s output:

```
tf_bH:
	movzwl	%si, %eax
	movq	%rax, last_tf_arg_u(%rip)
	movzwl	%si, %eax
	imull	$43691, %eax, %eax
	shrl	$17, %eax
	ret

```

Clang’s:

```
tf_bH:                                  # @tf_bH
	movq	%rsi, last_tf_arg_u(%rip)
	movw	%si, %ax
	movw	$-21845, %cx            # imm = 0xFFFFFFFFFFFFAAAB
	mulw	%cx
	andl	$65534, %edx            # imm = 0xFFFE
	movl	%edx, %eax
	shrl	%eax
	ret

```

At first glance, both compilers have done the right thing: turned the divide-by-constant into a multiply. The difference in execution times is explained by the Intel Optimization Reference Manual, Assembly/Compiler Coding Rule 21:

> Favor generating code using imm8 or imm32 values instead of imm16 values. If imm16 is needed, load equivalent imm32 into a register and use the word value in the register instead.

We verified that this is the problem by changing the imm16 value to imm32 by hand and re-running the performance test.

# Example 5

This function is get\_domain() from tree-call-cdce.c in GCC 4.5.0.

```
 1 struct input_domain
 2 {
 3   int lb;
 4   int ub;
 5   unsigned char has_lb;
 6   unsigned char has_ub;
 7   unsigned char is_lb_inclusive;
 8   unsigned char is_ub_inclusive;
 9 };
10 typedef struct input_domain inp_domain;
11 inp_domain
12 get_domain (int lb, unsigned char has_lb, unsigned char lb_inclusive,
13 	    int ub, unsigned char has_ub, unsigned char ub_inclusive)
14 {
15   inp_domain domain;
16
17   domain.lb = lb;
18   domain.has_lb = has_lb;
19   domain.is_lb_inclusive = lb_inclusive;
20   domain.ub = ub;
21   domain.has_ub = has_ub;
22   domain.is_ub_inclusive = ub_inclusive;
23   return (domain);
24 }

```

This function simply loads its arguments into a struct. GCC pushes the values through memory, taking 35 cycles. Intel CC generates code that uses memory 6 times (as opposed to 8 for GCC), requiring 24 cycles. Clang assembles the struct using registers, requiring only 12 cycles.

GCC’s code:

```
get_domain:
	movl	%edi, -24(%rsp)
	movl	%ecx, -20(%rsp)
	movb	%sil, -16(%rsp)
	movq	-24(%rsp), %rax
	movb	%r8b, -15(%rsp)
	movb	%dl, -14(%rsp)
	movb	%r9b, -13(%rsp)
	movl	-16(%rsp), %edx
	ret

```

Clang’s:

```
get_domain:
	movl	%edi, %eax
	shlq	$32, %rcx
	leaq	(%rcx,%rax), %rax
	shll	$16, %edx
	orl	%esi, %edx
	shll	$8, %r8d
	orl	%edx, %r8d
	shll	$24, %r9d
	movl	%r9d, %edx
	orl	%r8d, %edx
	ret

```

# Example 6

This function is php\_filter\_parse\_int() from logical\_filters.c in PHP 5.3.3.

```
  1 int
  2 php_filter_parse_int (char const *str, unsigned int str_len, long *ret)
  3 {
  4   long ctx_value;
  5   int sign;
  6   int digit;
  7   char const *end;
  8   int tmp;
  9   char const *tmp___0;
 10   char const *tmp___1;
 11
 12   sign = 0;
 13   digit = 0;
 14   end = str + str_len;
 15   switch ((int) *str)
 16     {
 17     case 45:
 18       sign = 1;
 19     case 43:
 20       str++;
 21     default:;
 22       break;
 23     }
 24   if ((unsigned long) str < (unsigned long) end)
 25     {
 26       if ((int const) *str >= 49)
 27 	{
 28 	  if ((int const) *str <= 57)
 29 	    {
 30 	      if (sign)
 31 		{
 32 		  tmp = -1;
 33 		}
 34 	      else
 35 		{
 36 		  tmp = 1;
 37 		}
 38 	      tmp___0 = str;
 39 	      str++;
 40 	      ctx_value = (long) (tmp * (int) ((int const) *tmp___0 - 48));
 41 	    }
 42 	  else
 43 	    {
 44 	      return (-1);
 45 	    }
 46 	}
 47       else
 48 	{
 49 	  return (-1);
 50 	}
 51     }
 52   else
 53     {
 54       return (-1);
 55     }
 56   if (end - str > 19)
 57     {
 58       return (-1);
 59     }
 60   while ((unsigned long) str < (unsigned long) end)
 61     {
 62       if ((int const) *str >= 48)
 63 	{
 64 	  if ((int const) *str <= 57)
 65 	    {
 66 	      tmp___1 = str;
 67 	      str++;
 68 	      digit = (int) ((int const) *tmp___1 - 48);
 69 	      if (!sign)
 70 		{
 71 		  if (ctx_value <=
 72 		      (9223372036854775807L - (long) digit) / 10L)
 73 		    {
 74 		      ctx_value = ctx_value * 10L + (long) digit;
 75 		    }
 76 		  else
 77 		    {
 78 		      goto _L;
 79 		    }
 80 		}
 81 	      else
 82 		{
 83 		_L:
 84 		  if (sign)
 85 		    {
 86 		      if (ctx_value >=
 87 			  ((-0x7FFFFFFFFFFFFFFF - 1) + (long) digit) / 10L)
 88 			{
 89 			  ctx_value = ctx_value * 10L - (long) digit;
 90 			}
 91 		      else
 92 			{
 93 			  return (-1);
 94 			}
 95 		    }
 96 		  else
 97 		    {
 98 		      return (-1);
 99 		    }
100 		}
101 	    }
102 	  else
103 	    {
104 	      return (-1);
105 	    }
106 	}
107       else
108 	{
109 	  return (-1);
110 	}
111     }
112   *ret = ctx_value;
113   return (1);
114 }

```

The assembly is long and not that interesting. The upshot is that Clang and Intel CC are able to turn the divisions by 10 at lines 72 and 87 into integer multiplies, whereas GCC leaves division instructions in the generated code. It is not clear why this happens; as we saw above, other divisions-by-constant are optimized successfully by this compiler. The result is that Clang’s and ICC’s code executes in 41 cycles whereas GCC’s takes 133.

# Example 7

This function is crud() from ident.c in git-1.7.3.2.

```
1 int crud (unsigned char c)
2 {
3   return (((((((((((int) c <= 32 || (int) c == 46) || (int) c == 44)
4 		 || (int) c == 58) || (int) c == 59) || (int) c == 60)
5 	      || (int) c == 62) || (int) c == 34) || (int) c == 92)
6 	   || (int) c == 39) != 0);
7 }

```

GCC and Clang generate the obvious cascaded comparisons and their code executes in 21 and 24 cycles, respectively. Intel CC is far more clever and its code requires only 8 cycles:

```
crud:
        movzbl    %dil, %edi
        cmpl      $32, %edi
        jle       ..B1.4
        addl      $-34, %edi
        cmpl      $64, %edi
        jae       ..B1.5
        movl      $1, %eax
        movl      %edi, %ecx
        shlq      %cl, %rax
        movq      $0x400000017001421, %rdx
        testq     %rdx, %rax
        je        ..B1.5
..B1.4:
        movl      $1, %eax
        ret
..B1.5:
        xorl      %eax, %eax
        ret

```

Since the range of characters being checked for is less than 64, ICC is able to pre-compute a bitmap with 1s in positions corresponding to the desired characters.

# Conclusions

Our idea is that using a small collection of compilers and a large collection of realistic benchmark functions, we can get the compilers to reveal performance limitations in each other. Once the basic infrastructure is in place, results can be recompiled with almost no manual effort. Although there are obvious limitations of this approach — it cannot spot an optimization missed by all compilers, it does not take the larger execution context of a benchmark function into account, etc. — we hope that it will become a useful part of compiler developers’ arsenal against performance problems.

These are very early results, selected from only about 1400 benchmark functions. As we remove limitations of the tool, it should become possible to harvest and test hundreds of thousands of benchmark functions. At that point, it may become useful to incorporate profile data into our metric for interesting functions, in order to direct compiler developers towards missed optimizations that are causing the most real slowdown.
