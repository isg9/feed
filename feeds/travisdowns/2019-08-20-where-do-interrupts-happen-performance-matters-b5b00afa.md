---
title: Where Do Interrupts Happen? | Performance Matters
url: https://travisdowns.github.io/blog/2019/08/20/interrupts.html
published: "2019-08-20T00:00:00Z"
feed: travisdowns
guid: https://travisdowns.github.io/blog/2019/08/20/interrupts.html
---

# Where Do Interrupts Happen? | Performance Matters

Aug 20, 2019

• [performance](/tags/performance.html)[benchmarking](/tags/benchmarking.html)

On Twitter, Paul Khuong [asks](https://twitter.com/pkhuong/status/1162832557030948864): *Has anyone done a study on the distribution of interrupt points in OOO processors?*

Personally, I’m not aware of any such study for modern x86, and I have also wondered the same thing. In particular, when a CPU receives an externally triggered[1](#fn:triggered) interrupt, at what point in the instruction stream is the CPU interrupted?

For a simple 1-wide in-order, non-pipelined CPU the answer might be as simple as: the CPU is interrupted either before or after instruction that is currently running[2](#fn:twoc). For anything more complicated it’s not going to be easy. On a modern out-of-order processor there may be hundreds of instructions in-flight at any time, some waiting to execute, a dozen or more currently executing, and others waiting to retire. From all these choices, which instruction will be chosen as the victim?

Among other reasons, the answer is interesting because it helps us understand how useful the exact interrupt position is when profiling via interrupt: can we extract useful information from the instruction position, or should we only trust it at a higher level (e.g., over regions of say 100s of instructions).

So let’s go figure out how interruption works, at least on my Skylake i7-6700HQ, by compiling a bunch of small pure-asm programs and running them. The source for all the tests is available in the associated [git repo](https://github.com/travisdowns/interrupt-test) so you can follow along or write your own tests. All the tests are written in assembly because we want full control over the instructions and because they are all short and simple. In any case, we can’t avoid assembly-level analysis when talking about what instructions get interrupted.

First, let’s take a look at some asm that doesn’t have any instruction that sticks out in any way at all, just a bunch of `mov` instructions[3](#fn:whymov). The key part of the [source](https://github.com/travisdowns/interrupt-test/blob/master/indep-mov.asm) looks like this[4](#fn:srcnotes):

```
.loop:

%rep 10
	mov  eax, 1
	mov  ebx, 2
	mov  edi, 3
	mov  edx, 4
	mov  r8d, 5
	mov  r9d, 6
	mov r10d, 7
	mov r11d, 8
%endrep

	dec rcx
	jne .loop

```

Just constant moves into registers, 8 of them repeated 10 times. This code executes with an expected and measured[5](#fn:ipc)IPC of 4.

Next, we get to the meat of the investigation. We run the binary using `perf record -e task-clock ./indep-mov`, which will periodically interrupt the process and record the IP. Next, we examine the interrupted locations with `perf report` [6](#fn:acommand). Here’s the output (hereafter, I’m going to cut out the header and just show the samples):

```
 Samples |	Source code & Disassembly of indep-mov for task-clock (1769 samples, percent: local period)
-----------------------------------------------------------------------------------------------------------
         :
         :            Disassembly of section .text:
         :
         :            00000000004000ae <_start.loop>:
         :            _start.loop():
         :            indep-mov.asm:15
      16 :   4000ae:       mov    eax,0x1
      15 :   4000b3:       mov    ebx,0x2
      22 :   4000b8:       mov    edi,0x3
      25 :   4000bd:       mov    edx,0x4
      14 :   4000c2:       mov    r8d,0x5
      19 :   4000c8:       mov    r9d,0x6
      25 :   4000ce:       mov    r10d,0x7
      18 :   4000d4:       mov    r11d,0x8
      22 :   4000da:       mov    eax,0x1
      24 :   4000df:       mov    ebx,0x2
      20 :   4000e4:       mov    edi,0x3
      29 :   4000e9:       mov    edx,0x4
      28 :   4000ee:       mov    r8d,0x5
      18 :   4000f4:       mov    r9d,0x6
      21 :   4000fa:       mov    r10d,0x7
      19 :   400100:       mov    r11d,0x8
      26 :   400106:       mov    eax,0x1
      18 :   40010b:       mov    ebx,0x2
      29 :   400110:       mov    edi,0x3
      19 :   400115:       mov    edx,0x4

```

The first column shows the number of interrupts received for each instruction. Specially, the number of times an instruction would be the next instruction to execute following the interrupt.

Without doing any deep statistical analysis, I don’t see any particular pattern here. Every instruction gets its time in the sun. Some columns have somewhat higher values than others, but if you repeat the measurements, the columns with higher values don’t necessarily repeat.

We can try the exact same thing, but with `add` instructions [like this](https://github.com/travisdowns/interrupt-test/blob/master/indep-add.asm):

```
	add  eax, 1
	add  ebx, 2
	add  edi, 3
	add  edx, 4
	add  r8d, 5
	add  r9d, 6
	add r10d, 7
	add r11d, 8

```

We expect the execution behavior to be similar to the `mov` case: we *do* have dependency chains here but 8 separate ones (for each destination register) for a 1 cycle instruction so there should be little practical impact. Indeed, the results are basically identical to the last experiment so I won’t show them here (you can see them yourself with the `indep-add` test).

Let’s get moving here and try something more interesting. This time we will again use all `add` instructions, but two of the adds will depend on each other, while the other two will be independent. So the chain shared by those two adds will be twice as long (2 cycles) as the other chains (1 cycle each). [Like this](https://github.com/travisdowns/interrupt-test/blob/master/add-2-1-1.asm):

```
    add  rax, 1 ; 2-cycle chain
    add  rax, 2 ; 2-cycle chain
    add  rsi, 3
    add  rdi, 4

```

Here the chain through `rax` should limit the throughput of the above repeated block to 1 per 2 cycles, and indeed I measure an IPC of 2 (4 instructions / 2 cycles = 2 IPC).

Here’s the interrupt distribution:

```
       0 :   4000ae:       add    rax,0x1
      82 :   4000b2:       add    rax,0x2
     112 :   4000b6:       add    rsi,0x3
       0 :   4000ba:       add    rdi,0x4
       0 :   4000be:       add    rax,0x1
      45 :   4000c2:       add    rax,0x2
     144 :   4000c6:       add    rsi,0x3
       0 :   4000ca:       add    rdi,0x4
       0 :   4000ce:       add    rax,0x1
      44 :   4000d2:       add    rax,0x2
     107 :   4000d6:       add    rsi,0x3

(pattern repeats...)

```

This is certainly something new. We see that *all* the interrupts fall on the middle two instructions, one of which is part of the addition chain and one which is not. The second of the two locations also gets about 2-3 times as many interrupts as the first.

## A Hypothesis

Let us form a hypothesis now so we can design more tests.

Let’s guess that interrupts *select* instructions which are the oldest unretired instruction, and that this *selected* instruction is allowed to complete hence samples fall on the next instruction (let us call this next instruction the *sampled* instruction). I am making the distinction between *selected* and *sampled* instructions rather than just saying “interrupts sample instructions that follow the oldest unretired instruction” because we are going to build our model almost entirely around the *selected* instructions, so we want to name them. The characteristics of the ultimately sampled instructions (except their positioning after *selected* instructions) hardly matters[7](#fn:untrue).

Without a more detailed model of instruction retirement, we can’t yet explain everything we see - but the basic idea is instructions that take longer, hence are more likely to be the oldest unretired instruction, are the ones that get sampled. In particular, if there is a critical dependency chain, instructions in that chain are likely[8](#fn:likely) be sampled at some point[9](#fn:notonly).

Let’s take a look at some more examples. I’m going to switch using `mov rax, [rax]` as my long latency instruction (4 cycles latency) and `nop` as the filler instruction not part of any chain. Don’t worry, `nop` has to allocate and retire just like any other instruction: it simply gets to skip execution. You can build all these examples with a real instruction like `add` and they’ll work in the same way[10](#fn:whynop).

Let’s [take a look](https://github.com/travisdowns/interrupt-test/blob/master/load-nop10.asm) at a load followed by 10 `nops`:

```
.loop:

%rep 10
    mov  rax, [rax]
    times 10 nop
%endrep

    dec rcx
    jne .loop

```

The result:

```
       0 :   4000ba:       mov    rax,QWORD PTR [rax]
      33 :   4000bd:       nop
       0 :   4000be:       nop
       0 :   4000bf:       nop
       0 :   4000c0:       nop
      11 :   4000c1:       nop
       0 :   4000c2:       nop
       0 :   4000c3:       nop
       0 :   4000c4:       nop
      22 :   4000c5:       nop
       0 :   4000c6:       nop
       0 :   4000c7:       mov    rax,QWORD PTR [rax]
      15 :   4000ca:       nop
       0 :   4000cb:       nop
       0 :   4000cc:       nop
       0 :   4000cd:       nop
      13 :   4000ce:       nop
       1 :   4000cf:       nop
       0 :   4000d0:       nop
       0 :   4000d1:       nop
      35 :   4000d2:       nop
       0 :   4000d3:       nop
       0 :   4000d4:       mov    rax,QWORD PTR [rax]
      16 :   4000d7:       nop
       0 :   4000d8:       nop
       0 :   4000d9:       nop
       0 :   4000da:       nop
      14 :   4000db:       nop
       0 :   4000dc:       nop
       0 :   4000dd:       nop
       0 :   4000de:       nop
      31 :   4000df:       nop
       0 :   4000e0:       nop
       0 :   4000e1:       mov    rax,QWORD PTR [rax]
      22 :   4000e4:       nop
       0 :   4000e5:       nop
       0 :   4000e6:       nop
       0 :   4000e7:       nop
      16 :   4000e8:       nop
       0 :   4000e9:       nop
       0 :   4000ea:       nop
       0 :   4000eb:       nop
      24 :   4000ec:       nop
       0 :   4000ed:       nop

```

The *selected* instructions are the long-latency `mov`-chain, but also two specific `nop` instructions out of the 10 that follow: those which fall 4 and 8 instructions after the `mov`. Here we can see the impact of retirement throughput. Although the `mov`-chain is the only thing that contributes to *execution* latency, this Skylake CPU can [only retire](https://www.realworldtech.com/forum/?threadid=161978&curpostid=162222) up to 4 instructions per cycle (per thread). So when the `mov` finally retires, there will be two cycles of retiring blocks of 4 `nop` instructions before we get to the next `mov`:

```
;                            retire cycle
mov    rax,QWORD PTR [rax] ; 0 (execution limited)
nop                        ; 0
nop                        ; 0
nop                        ; 0
nop                        ; 1 <-- selected nop
nop                        ; 1
nop                        ; 1
nop                        ; 1
nop                        ; 2 <-- selected nop
nop                        ; 2
nop                        ; 2
mov    rax,QWORD PTR [rax] ; 4 (execution limited)
nop                        ; 4
nop                        ; 4

```

In this example, the retirement of `mov` instructions are “execution limited” - i.e., their retirement cycle is determined by when they are done executing, not by any details of the retirement engine. The retirement of the other instructions, on the other hand, is determined by the retirement behavior: they are “ready” early but cannot retire because the in-order retirement pointer hasn’t reached them yet (it is held up waiting for the `mov` to execute).

So those selected `nop` instructions aren’t particularly special: they aren’t slower than the rest or causing any bottleneck. They are simply selected because the pattern of retirement following the `mov` is predictable. Note also that it is the *first* `nop` that in the group of 4 that would otherwise retire that is selected[11](#fn:already).

This means that we can construct [an example](https://github.com/travisdowns/interrupt-test/blob/master/load-add2.asm) where an instruction on the critical path never gets selected:

```
%rep 10
    mov  rax, [rax]
    nop
    nop
    nop
    nop
    nop
    add  rax, 0
%endrep

```

The results:

```
       0 :   4000ba:       mov    rax,QWORD PTR [rax]
      94 :   4000bd:       nop
       0 :   4000be:       nop
       0 :   4000bf:       nop
       0 :   4000c0:       nop
      17 :   4000c1:       nop
       0 :   4000c2:       add    rax,0x0
       0 :   4000c6:       mov    rax,QWORD PTR [rax]
      78 :   4000c9:       nop
       0 :   4000ca:       nop
       0 :   4000cb:       nop
       0 :   4000cc:       nop
      18 :   4000cd:       nop
       0 :   4000ce:       add    rax,0x0

```

The `add` instruction is on the critical path: it increases the execution time of the block from 4 cycles to 6 cycles[12](#fn:4to6), yet it is never selected. The retirement pattern looks like:

```
                     ;  /- scheduled
                     ; |  /- ready
                     ; |  |  /- complete
                     ; |  |  |  /- retired
    mov  rax, [rax]  ; 0  0  5  5 <-- selected
    nop              ; 0  0  0  5 <-- sample
    nop              ; 0  0  0  5
    nop              ; 0  0  0  5
    nop              ; 1  1  1  6 <-- selected
    nop              ; 1  1  1  6 <-- sampled
    add  rax, 0      ; 1  5  6  6
    mov  rax, [rax]  ; 1  6 11 11 <-- selected
    nop              ; 2  2  2 11 <-- sampled
    nop              ; 2  2  2 11
    nop              ; 2  2  2 11
    nop              ; 2  2  2 12 <-- selected
    nop              ; 3  3  3 12 <-- sampled
    add  rax, 0      ; 3 11 12 12
    mov  rax, [rax]  ; 3 12 17 17 <-- selected
    nop              ; 3  3  3 17 <-- sampled

```

On the right hand side, I’ve annotated each instruction with several key cycle values, described below.

The *scheduled* column indicates when the instruction enters the scheduler and hence could execute if all its dependencies were met. This column is very simple: we assume that there are no front-end bottlenecks and hence we schedule (aka “allocate”) 4 instructions every cycle. This part is in-order: instructions enter the scheduler in program order.

The *ready* column indicates when all dependencies of a scheduled instruction have executed and hence the instruction is ready to execute. In this simple model, an instruction always begins executing when it is ready. A more complicated model would also need to model contention for execution ports, but here we don’t have any such contention. Instruction readiness occurs *out of order*: you can see that many instructions become ready before older instructions (e.g., the `nop` instructions are generally ready before the preceding `mov` or `add` instructions). To calculate this column take the maximum of the *ready* column for this instruction and the *completed* column for all previous instructions whose outputs are inputs to this instruction.

The *complete* column indicates when an instruction finishes execution. In this model it simply takes the value of the *ready* column plus the instruction latency, which is 0 for the `nop` instructions (they don’t execute at all, so they have 0 effective latency), 1 for the `add` instruction and 5 for the `mov`. Like the *ready* column this happens out of order.

Finally, the *retired* column, what we’re really after, shows when the instruction retires. The rule is fairly simple: an instruction cannot retire until it is complete, and the instruction before it must be retired or retiring on this cycle. No more than 4 instructions can retire in a cycle. As a consequence of the “previous instruction must retired” part, this column is only increasing and so like the first column, retirement is *in order*.

Once we have the *retired* column filled out, we can identify the `<-- selected` instructions[13](#fn:possibly): they are the ones where the retirement cycle increases. In this case, selected instructions are always either the `mov` instruction (because of its long latency, it holds up retirement), or the fourth `nop` after the `mov` (because of the “only retire 4 per cycle” rule, this `nop` is at the head of the group that retires in the cycle after the `mov` retires). Finally, the *sampled* instructions which are those will actually show up in the interrupt report are simply the instruction following each selected instruction.

Here, the `add` is never selected because it executes in the cycle after the `mov`, so it is eligible for retirement in the next cycle and hence doesn’t slow down retirement and so doesn’t behave much differently than a `nop` for the purposes of retirement. We can change the position of the `add` slightly, so it falls in the same 4-instruction retirement window as the `mov`, [like this](https://github.com/travisdowns/interrupt-test/blob/master/load-add3.asm):

```
%rep 10
    mov  rax, [rax]
    nop
    nop
    add  rax, 0
    nop
    nop
    nop
%endrep

```

We’ve only slid the `add` up a few places. The number of instructions is the same and this block executes in 6 cycles, identical to the last example. However, the `add` instruction now always gets selected:

```
      21 :   4000ba:       mov    rax,QWORD PTR [rax]
      54 :   4000bd:       nop
       0 :   4000be:       nop
       0 :   4000bf:       add    rax,0x0
      15 :   4000c3:       nop
       0 :   4000c4:       nop
       0 :   4000c5:       nop
       0 :   4000c6:       mov    rax,QWORD PTR [rax]
      88 :   4000c9:       nop
       0 :   4000ca:       nop
       0 :   4000cb:       add    rax,0x0
      14 :   4000cf:       nop
       0 :   4000d0:       nop
       0 :   4000d1:       nop
       0 :   4000d2:       mov    rax,QWORD PTR [rax]
      91 :   4000d5:       nop
       0 :   4000d6:       nop
       0 :   4000d7:       add    rax,0x0
      13 :   4000db:       nop

```

Here’s the cycle analysis and retirement pattern for this version:

```
                     ;  /- scheduled
                     ; |  /- ready
                     ; |  |  /- complete
                     ; |  |  |  /- retired
    mov  rax, [rax]  ; 0  0  5  5 <-- selected
    nop              ; 0  0  0  5 <-- sample
    nop              ; 0  0  0  5
    add  rax, 0      ; 0  5  6  6 <-- selected
    nop              ; 1  1  1  6 <-- sampled
    nop              ; 1  1  1  6
    nop              ; 1  1  1  6
    mov  rax, [rax]  ; 1  6 11 11 <-- selected
    nop              ; 2  2  2 11 <-- sampled
    nop              ; 2  2  2 11
    add  rax, 0      ; 2 11 12 12 <-- selected
    nop              ; 2  2  2 12 <-- sampled
    nop              ; 3  3  3 12
    nop              ; 3  3  3 12
    mov  rax, [rax]  ; 3 12 17 17 <-- selected
    nop              ; 3  3  3 17 <-- sampled

```

Now might be a good time to note that we also care about the actual sample counts, and not just their presence or absence. Here, the samples associated with the `mov` are more frequent than the samples associated with the `add`. In fact, there are about 4.9 samples for `mov` for every sample for `add` (calculated over the full results). That lines up almost exactly with the `mov` having a latency of 5 and the `add` a latency of 1: the `mov` will be the oldest unretired instruction 5 times as often as the `add`. So the sample counts are very meaningful in this case.

Going back to the cycle charts, we know the *selected* instructions are those where the retirement cycle increases. To that we add that the *size* of the increase determines their *selection weight*: the `mov` instruction has a weight of 5, since it jumps (for example) from 6 to 11 so it is the oldest unretired instruction for 5 cycles, while the `nop` instructions have a weight of 1.

This lets you measure *in-situ* the latency of various instructions, as long as you have accounted for the retirement behavior. For example, measuring the following block ( `rdx` is zero at runtime):

```
    mov  rax, [rax]
    mov  rax, [rax + rdx]

```

Results in samples accumulating in a 4:5 ratio for the first and second lines: reflecting the fact that the second load has a latency of 5 due to complex addressing, while the first load takes only 4 cycles.

### Branches

What about branches? I don’t find anything special about branches: whether taken or untaken they seem to retire normally and fit the pattern described above. I am not going to show the results but you can play with the [`branches` test](https://github.com/travisdowns/interrupt-test/blob/master/branches.asm) yourself if you want.

### Atomic Operations

What about atomic operations? Here the story does get interesting.

I’m going to use `lock add QWORD [rbx], 1` as my default atomic instruction, but the story seems similar for all of them. Alone, this instruction has a “latency”[14](#fn:atomiclat) of 18 cycles. Let’s put it in parallel with a couple of SIMD instructions that have a total latency of 20 cycles alone:

```
    vpmulld xmm0, xmm0, xmm0
    vpmulld xmm0, xmm0, xmm0
    lock add QWORD [rbx], 1

```

This loop *still takes 20 cycles* to execute. That is, the atomic costs nothing in runtime: the performance is the same if you comment it out. The `vpmulld` dependency chain is long enough to hide the cost of the atomic. Let’s take a look at the interrupt distribution for this code:

```
       0 :   4000c8:       vpmulld xmm0,xmm0,xmm0
      12 :   4000cd:       vpmulld xmm0,xmm0,xmm0
      10 :   4000d2:       lock add QWORD PTR [rbx],0x1
     244 :   4000d7:       vpmulld xmm0,xmm0,xmm0
       0 :   4000dc:       vpmulld xmm0,xmm0,xmm0
      26 :   4000e1:       lock add QWORD PTR [rbx],0x1
     299 :   4000e6:       vpmulld xmm0,xmm0,xmm0
       0 :   4000eb:       vpmulld xmm0,xmm0,xmm0
      35 :   4000f0:       lock add QWORD PTR [rbx],0x1
     277 :   4000f5:       vpmulld xmm0,xmm0,xmm0
       0 :   4000fa:       vpmulld xmm0,xmm0,xmm0
      33 :   4000ff:       lock add QWORD PTR [rbx],0x1
     302 :   400104:       vpmulld xmm0,xmm0,xmm0
       0 :   400109:       vpmulld xmm0,xmm0,xmm0
      33 :   40010e:       lock add QWORD PTR [rbx],0x1
     272 :   400113:       vpmulld xmm0,xmm0,xmm0
       0 :   400118:       vpmulld xmm0,xmm0,xmm0
      31 :   40011d:       lock add QWORD PTR [rbx],0x1
     280 :   400122:       vpmulld xmm0,xmm0,xmm0
       0 :   400127:       vpmulld xmm0,xmm0,xmm0
      40 :   40012c:       lock add QWORD PTR [rbx],0x1
     277 :   400131:       vpmulld xmm0,xmm0,xmm0
       0 :   400136:       vpmulld xmm0,xmm0,xmm0
      21 :   40013b:       lock add QWORD PTR [rbx],0x1
     282 :   400140:       vpmulld xmm0,xmm0,xmm0
       0 :   400145:       vpmulld xmm0,xmm0,xmm0
      35 :   40014a:       lock add QWORD PTR [rbx],0x1
     291 :   40014f:       vpmulld xmm0,xmm0,xmm0
       0 :   400154:       vpmulld xmm0,xmm0,xmm0
      35 :   400159:       lock add QWORD PTR [rbx],0x1
     270 :   40015e:       dec    rcx
       0 :   400161:       jne    4000c8 <_start.loop>

```

The `lock add` instructions are selected by the interrupt close to 90% of the time, despite not contributing to the execution time. Based on our mental model, these instructions should be able to run ahead of the `vpmulld` loop and hence be ready to retire as soon as they are the head of the ROB. The effect that we see here is because `lock`-prefixed instructions are *execute at retire*. This is a special type of instruction that waits until it at the head of the ROB before it executes[15](#fn:execretire).

So this instruction will *always* take a certain minimum amount of time as the oldest unretired instruction: it never retires immediately regardless of the surrounding instructions. In this case, it means retirement spends most of its time waiting for the locked instructions, while execution spends most of its time waiting for the `vpmulld`. Note that the retirement time added by the locked instructions wasn’t additive with that from `vpmulld`: the time it spends waiting for retirement is subtracted from the time that would otherwise be spent waiting on the multiplication retirement. That’s why you end up with a lopsided split, not 50/50. We can see this more clearly if we double the number of multiplications to 4:

```
    vpmulld xmm0, xmm0, xmm0
    vpmulld xmm0, xmm0, xmm0
    vpmulld xmm0, xmm0, xmm0
    vpmulld xmm0, xmm0, xmm0
    lock add QWORD [rbx], 1

```

This takes 40 cycles to execute, and the interrupt pattern looks like:

```
       0 :   4000c8:       vpmulld xmm0,xmm0,xmm0
      18 :   4000cd:       vpmulld xmm0,xmm0,xmm0
      49 :   4000d2:       vpmulld xmm0,xmm0,xmm0
     133 :   4000d7:       vpmulld xmm0,xmm0,xmm0
     152 :   4000dc:       lock add QWORD PTR [rbx],0x1
     263 :   4000e1:       vpmulld xmm0,xmm0,xmm0
       0 :   4000e6:       vpmulld xmm0,xmm0,xmm0
      61 :   4000eb:       vpmulld xmm0,xmm0,xmm0
     160 :   4000f0:       vpmulld xmm0,xmm0,xmm0
     168 :   4000f5:       lock add QWORD PTR [rbx],0x1
     251 :   4000fa:       vpmulld xmm0,xmm0,xmm0
       0 :   4000ff:       vpmulld xmm0,xmm0,xmm0
      59 :   400104:       vpmulld xmm0,xmm0,xmm0
     166 :   400109:       vpmulld xmm0,xmm0,xmm0
     162 :   40010e:       lock add QWORD PTR [rbx],0x1
     267 :   400113:       vpmulld xmm0,xmm0,xmm0
       0 :   400118:       vpmulld xmm0,xmm0,xmm0
      60 :   40011d:       vpmulld xmm0,xmm0,xmm0
     160 :   400122:       vpmulld xmm0,xmm0,xmm0
     155 :   400127:       lock add QWORD PTR [rbx],0x1
     218 :   40012c:       vpmulld xmm0,xmm0,xmm0
       0 :   400131:       vpmulld xmm0,xmm0,xmm0
      58 :   400136:       vpmulld xmm0,xmm0,xmm0
     144 :   40013b:       vpmulld xmm0,xmm0,xmm0
     154 :   400140:       lock add QWORD PTR [rbx],0x1

pattern continues...

```

The sample counts are similar for `lock add` but they’ve now increased to a comparable amount (in total) for the `vmulld` instructions[16](#fn:uneven). In fact, we can calculate how long the `lock add` instruction takes to retire, using the ratio of the times it was selected compared to the known block throughput of 40 cycles. I get about a 38%-40% rate over a couple of runs which corresponds to a retire time of 15-16 cycles, only slightly less than then back-to-back latency[14](#fn:atomiclat) of this instruction.

Can we do anything with this information?

Well one idea is that it lets us fairly precisely map out the retirement timing of instructions. For example, we can set up an instruction to test, and a parallel series of instructions with a known latency. Then we observe what is selected by interrupts: the instruction under test or the end of the known-latency chain. Whichever is selected has longer retirement latency and the known-latency chain can be adjusted to narrow it down exactly.

Of course, this sounds way harder than the usual way of measuring latency: a long series of back-to-back instructions, but it does let us measure some things *in situ* without a long chain, and we can measure instructions that don’t have an obvious way to chain (e.g,. have no output like stores or different-domain instructions).

### Some Things That I Didn’t Get To

- Explain the variable (non-self synchronizing) results in terms of retire window patterns
- Check interruptible instructions
- Check `mfence` and friends
- Check the execution effect of atomic instructions (e.g., blocking load ports)

### Comments

Feedback of any type is welcome. I don’t have a comments system[17](#fn:comments) yet, so as usual I’ll outsource discussion to [this HackerNews thread](https://news.ycombinator.com/item?id=20751186).

### Thanks and Attribution

Thanks to HN user rrss for pointing out errors in my cycle chart.

Thanks to Alex Dowad for pointing out a typo in the text.

[Intel 8259 image](https://commons.wikimedia.org/wiki/File:Intel_8259.svg) by Wikipedia user German under [CC BY-SA 3.0](https://creativecommons.org/licenses/by-sa/3.0).

---

---

1. By *externally triggered* I mean pretty much any interrupt that doesn’t result directly from the code currently running on the logical core (such as an `int` instruction or some fault/trap). So it includes includes timer interrupts, cross CPU inter-processor interrupts, etc. [↩](#fnref:triggered)

2. The two choices basically correspond to letting the current instruction finish, or aborting it in order to run the interrupt. [↩](#fnref:twoc)

3. I use `mov` here to avoid creating any dependency chains, i.e., the destination of `mov` is *write-only* unlike most x86 integer instructions which both read and write their first operand. [↩](#fnref:whymov)

4. Here I show the full loop, and the `%rep` directive which repeats (unrolls) the contained block the specified number of times, but from this point on I may just show the inner block itself with the understanding that it is unrolled with `%rep` some number of times (to reduce the impact of the loop) and contained with a loop of ~1,000,000 iterations so we get a reasonable number of samples. For the exact code you can always refer to the [repo](https://github.com/travisdowns/interrupt-test). [↩](#fnref:srcnotes)

5. In general I’ll just say “this code executes like this” or “the bottleneck is this”, without further justification as the examples are usually simple with one obvious bottleneck. Even though I won’t always mentioned it, I *have* checked that the examples perform as expected, usually with a `perf stat` to confirm the IPC or cycles per iteration or whatever. [↩](#fnref:ipc)

6. The full command I use is something like `perfc annotate -Mintel --stdio --no-source --stdio-color --show-nr-samples`. For interactive analysis just drop the `--stdio` part to get the tui UI. [↩](#fnref:acommand)

7. Of course we will see that in some cases the *selected* and *sampled* instructions can actually be the same, in the case of interruptible instructions. [↩](#fnref:untrue)

8. Originally I thought it was guaranteed that instructions on the critical path would be eligible for sampling, but it is not: later we construct an example where a critical instruction never gets selected. [↩](#fnref:likely)

9. Note that I am specifically *not* saying the only selected instructions will be from the chain. As we will see soon, retirement limits mean that unless the chain instructions are fairly dense (i.e., at least 1 every 4 instructions), other instructions may appear also. [↩](#fnref:notonly)

10. I use `nop` because it has no inputs or outputs so I don’t have to be careful not to create dependency chains (e.g., ensuring each `add` goes to a distinct register), and it doesn’t use any execution units, so we won’t steal a unit from the chain on the critical path. [↩](#fnref:whynop)

11. We have already seen this effect in all of the earlier examples: it is why the long-latency `add` is selected, even though that in the cycle that it retires the three subsequent instructions also retire. This behavior is nice because it ensures that longer latency instructions are generally selected, rather than some unrelated instruction (i.e., the usual skid is +1 not +4). [↩](#fnref:already)

12. The increase is from 4 to 6, rather than 4 to 5, because of +1 cycle for the `add` itself (obvious) and +1 cycle because any ALU instruction in the address computation path of a pointer chasing loop disables the 4-cycle load fast path (much less obvious). [↩](#fnref:4to6)

13. Of course this doesn’t mean the instruction *will* be selected, we only have a few thousand interrupts over one million or more interrupts, so usually nothign happens at all. These are just the locations that are highly likely to be chosen when an interrupt does arrive. [↩](#fnref:possibly)

14. Latency is in scare quotes here because the apparent latency of atomic operations doesn’t behave like most other instructions: if you measure the throughput of back-to-back atomic instructions you get higher performance if the instructions form a dependency chain, rather than if they don’t! The latency of atomics doesn’t fit neatly into an input-to-output model as their *execute at retirement* behavior causes other bottlenecks. [↩](#fnref:atomiclat) [↩2](#fnref:atomiclat:1)

15. That’s the simple way of looking at it: reality is more complex as usual. Locked instructions may execute some of their instructions before they are ready to retire: e.g., loading the the required value and the ALU operation. However, the key “unlock” operation which verifies the result of the earlier operations and commits the results, in an atomic way, happens at retire and this is responsible for the majority of the cost of this instruction. [↩](#fnref:execretire)

16. The uneven pattern among the `vpmulld` instructions is because the `lock add` instruction eats the retire cycles of the first `vpmulld` that occur after (they can retire quickly because they are already complete), so only the later ones have a full allocation. [↩](#fnref:uneven)

17. If anyone has a recommendation or a if anyone knows of a comments system that works with static sites, and which is not Disqus, has no ads, is free and fast, and lets me own the comment data (or at least export it in a reasonable format), I am all ears. [↩](#fnref:comments)

---

## Comments

Noah Goldstein • [February 3th, 2025 23:53](#comment-009d0770-e28a-11ef-98ee-e12241f4a0ad "Permalink to this comment")

Edit to my first comment about “oldest unretired instruction”. I had youngest and oldest mixed up… the oldest instruction to retire in a given cycle is the first one to retire :)

Makes sense. Feel free to kill that comment and this one.

Great article as always.

↪︎ Reply to Noah Goldstein

---

Noah Goldstein • [February 3th, 2025 23:44](#comment-c80f72e0-e288-11ef-98ee-e12241f4a0ad "Permalink to this comment")

re:

> I am not going to show the results but you can play with the branches test yourself if you want.

Is the sampled instruction the next in straight line code or the next instruction executed?

↪︎ Reply to Noah Goldstein

---

Noah Goldstein • [February 3th, 2025 23:39](#comment-2466a0f0-e288-11ef-98ee-e12241f4a0ad "Permalink to this comment")

Using the “oldest unretired instruction” model, why is the *selected* instruction the **first** in a new retirement block i.e in the first example:

```
;                            retire cycle
mov    rax,QWORD PTR [rax] ; 0 (execution limited)
nop                        ; 0
nop                        ; 0
nop                        ; 0 <-- oldest unretired on cycle 0
nop                        ; 1 <-- selected nop
nop                        ; 1
nop                        ; 1
nop                        ; 1 <-- oldest unretired on cycle 1
nop                        ; 2 <-- selected nop
nop                        ; 2
nop                        ; 2 <-- oldest unretired on cycle 2
mov    rax,QWORD PTR [rax] ; 4 (execution limited)
nop                        ; 4
nop                        ; 4 <-- oldest unretired on cycle 4

```

↪︎ Reply to Noah Goldstein

---

Lewis Chan • [November 23th, 2024 04:59](#comment-b44def30-a957-11ef-ae4a-93a4ae268d6a "Permalink to this comment")

nice article, but quite hard for me to understand. I know some concepts of computer architecture, but I feel some preliminary knowledge is lacking.

```
So when the mov finally retires, there will be two cycles of retiring blocks of 4 nop instructions before we get to the next mov:

the retirement of mov instructions are “execution limited” - i.e., their retirement cycle is determined by when they are done executing, not by any details of the retirement engine. The retirement of the other instructions, on the other hand, is determined by the retirement behavior: they are “ready” early but cannot retire because the in-order retirement pointer hasn’t reached them yet

```

Such words are unfamiliar to me, could you provide some introductory materials for new learners ?

↪︎ Reply to Lewis Chan

---

kolya • [June 23th, 2021 08:27](#comment-d2e42b00-d3fc-11eb-b2f8-5d7287282354 "Permalink to this comment")

Oh, I mean “more similar to add”, not indep-add. indep-add looks like indep-mov, as I’d expect.

↪︎ Reply to kolya

---

kolya • [June 23th, 2021 08:15](#comment-31d13ba0-d3fb-11eb-b2f8-5d7287282354 "Permalink to this comment")

Running indep-mov test (#0) on my Nehalem: perf record -e task-clock ./indep-mov \| perf annotate -Mintel –stdio –no-source –stdio-color –show-nr-samples shows pattern which looks more similar to indep-add but there is non-zero (but still very small compared to max/avg) values. Like 45 0 6 40 22 35 0 0 \[cycle\] 7 33 0 9 1 26 32 0 \[cycle\]… This is definitely not your distribution. Of course, there is an absence of any prioritized instruction (if I sum values for all cycles I will likely get the same picture), but for you it works at per-cycle basis.

↪︎ Reply to kolya

---

Alex Dowad • [January 15th, 2021 15:56](#comment-3ed7ab50-574a-11eb-971c-e3dfa8f2b075 "Permalink to this comment")

Typo: “instructions” -> “instrucitons”

↪︎ Reply to Alex Dowad

Author Travis Downs • [January 29th, 2021 05:42](#comment-cdad8e30-61f4-11eb-8073-2b87c96cc065 "Permalink to this comment")

Thanks Alex, fixed and credited.

---

## Add Comment

Name

E-mail

Cancel Reply

Submit

Close
