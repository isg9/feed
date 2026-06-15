---
title: More Saturating Arithmetic
url: https://blog.regehr.org/archives/278
published: "2010-10-08T05:28:27Z"
feed: regehr
guid: http://blog.regehr.org/?p=278
---

# More Saturating Arithmetic

In [a previous post](https://blog.regehr.org/archives/277) I guessed that 91 bytes was close to the minimum size for implementing signed and unsigned saturating 32-bit addition and subtraction on x86. A commenter rightly pointed out that it should be possible to do better. Attached is my best shot: 83 bytes. Given the crappy x86 calling convention I’m going to guess it’s hard to do much better unless the saturating SSE instructions offer a way. Or can the branches in the signed functions be avoided? It seems like it, but I couldn’t figure out how.

> ```
> sat_unsigned_add:
> 	movl	4(%esp), %eax
> 	xor     %edx, %edx
> 	not     %edx
> 	addl	8(%esp), %eax
> 	cmovb	%edx, %eax
> 	ret
>
> sat_unsigned_sub:
> 	movl	4(%esp), %eax
> 	xor     %edx, %edx
> 	subl	8(%esp), %eax
> 	cmovb	%edx, %eax
> 	ret
>
> sat_signed_add:
> 	movl	4(%esp), %eax
> 	movl	$2147483647, %edx
> 	movl	$-2147483648, %ecx
> 	addl	8(%esp), %eax
> 	jno     out1
> 	cmovg	%edx, %eax
> 	cmovl	%ecx, %eax
> out1:	ret
>
> sat_signed_sub:
> 	movl	4(%esp), %eax
> 	movl	$2147483647, %edx
> 	movl	$-2147483648, %ecx
> 	subl	8(%esp), %eax
> 	jno     out2
> 	cmovg	%edx, %eax
> 	cmovl	%ecx, %eax
> out2:	ret
> ```

Another question: How difficult is it to create a superoptimizer that turns obvious C formulations of saturating operations into the code shown here? This is beyond the capabilities of current superoptimizers, I believe, while being obviously feasible.

Feedback is appreciated.
