---
title: A Quick Update to Comparing Compiler Optimizations
url: https://blog.regehr.org/archives/324
published: "2010-12-14T19:51:36Z"
feed: regehr
guid: http://blog.regehr.org/?p=324
---

# A Quick Update to Comparing Compiler Optimizations

Saturday’s post on [Comparing Compiler Optimizations](https://blog.regehr.org/archives/320) featured some work done by my student Yang Chen. Since then:

- There has been some discussion of these problems on the GCC mailing list; some of the problems are already in the Bugzilla and a [new PR was filed](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=46928). A patch fixing the problem is already available!
- On Sunday night Chris Lattner left a comment saying that two of the Clang performance problems have been fixed (the other two are known problems and will take more time to fix).

Have I ever said how great open source is? Of course, there’s nothing stopping a product group from fixing a bug just as quickly — but the results would not be visible to the outside world until the next release. When improvements to a piece of software depend on a tight feedback loop between developers and outsiders, it’s pretty difficult to beat the open source model.

Yang re-tested Clang and here are the results.

# Example 4

[Previously](https://blog.regehr.org/archives/320#example4), Clang’s output required 18 cycles to execute. The new output is both faster (4 cycles) and smaller than GCC’s or Intel CC’s:

```
tf_bH:
        movq    %rsi, last_tf_arg_u(%rip)
        imull   $43691, %esi, %eax      # imm = 0xAAAB
        shrl    $17, %eax
        ret

```

Nice!

# Example 7

[Previously](https://blog.regehr.org/archives/320#example7), Intel CC’s 8-cycle code was winning against Clang’s 24 and GCC’s 21 cycles. Clang now generates this code:

```
crud:
	cmpb	$33, %dil
	jb	.LBB0_4
	addb	$-34, %dil
	cmpb	$58, %dil
	ja	.LBB0_3
	movzbl	%dil, %eax
	movabsq	$288230376537592865, %rcx
	btq	%rax, %rcx
	jb	.LBB0_4
.LBB0_3:
	xorl	%eax, %eax
	ret
.LBB0_4:
	movl	$1, %eax
	ret

```

Although this is several instructions shorter than Intel CC’s output, it still requires 10 cycles to execute on our test machine, as opposed to ICC’s 8 cycles. Perhaps this is in the noise, but Yang spent a while trying to figure it out anyway. It seems that if you replace the btq instruction with a testq (and adjust the subsequent branch appropriately), Clang’s output also executes in 8 cycles. I won’t pretend to understand this.

# Example 6 (Updated Dec 17)

This has been fixed in GCC, see the [discussion here](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=46939). The failure to optimize was pretty interesting: some hacking in GCC’s profile code caused some breakage that lead it to infer that the division instructions in this function are “cold” and not worth optimizing to multiplies. This is a great example of how the lack of good low-level metrics makes it difficult to detect subtle regressions. Of course this would have been noticed eventually by someone who reads assembly, but there are probably other similar problems that are below the noise floor when we look only at results from high-level benchmarks.
