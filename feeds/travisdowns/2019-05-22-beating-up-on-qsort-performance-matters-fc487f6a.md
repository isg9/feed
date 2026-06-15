---
title: Beating Up on Qsort | Performance Matters
url: https://travisdowns.github.io/blog/2019/05/22/sorting.html
published: "2019-05-22T00:00:00Z"
feed: travisdowns
guid: https://travisdowns.github.io/blog/2019/05/22/sorting.html
---

# Beating Up on Qsort | Performance Matters

May 22, 2019

• [algorithms](/tags/algorithms.html)[performance](/tags/performance.html)[perf](/tags/perf.html)

Recently, Daniel Lemire [tackled the topic](https://lemire.me/blog/2019/05/07/almost-picking-n-distinct-numbers-at-random/) of selecting N *distinct* numbers at random. In the case we want sorted output, an obvious solution presents itself: sorting randomly chosen values and de-duplicating the list, which is easy since identical values are now adjacent.[1](#fn:distinct)

While Daniel suggests a clever method of avoiding a sort entirely[2](#fn:danmethod), I’m also interested in they *why* for the underlying performace of the sort method: it takes more than 100 ns per element, which means 100s of CPU clock cycles and usually even more instructions than that (on a superscalar processor)! As a sanity check, a quick benchmark ( `perf record ./bench && perf report`) shows that more than 90% of the time spent in this approach is in the sorting routine, [qsort](https://devdocs.io/c/algorithm/qsort) \- so we are right to focus on this step, rather than say the de-duplication step or the initial random number generation. This naturally, this raises the question: how fast is qsort when it comes to sorting integers and can we do better?

All of the code for this post [is available on GitHub](https://github.com/travisdowns/sort-bench), so if you’d like to follow along with the code open in an editor, go right ahead (warning: there are obviously some spoilers if you dig through the code first).

## Benchmarking Qsort

First, let’s take a look at what `qsort` is doing, to see if there is any delicous low-hanging performance fruit. We use `perf record ./bench qsort` to capture profiling data, and `perf report --stdio` to print a summary[3](#fn:long-tail):

```
# Samples: 101K of event 'cycles:ppp'
# Event count (approx.): 65312285835
#
# Overhead  Command  Shared Object      Symbol
# ........  .......  .................  ..............................................
#
    64.90%  bench    libc-2.23.so       [.] msort_with_tmp.part.0
    21.45%  bench    bench              [.] compare_uint64_t
     8.65%  bench    libc-2.23.so       [.] __memcpy_sse2
     0.87%  bench    libc-2.23.so       [.] __memcpy_avx_unaligned
     0.83%  bench    bench              [.] main
     0.41%  bench    [kernel.kallsyms]  [k] clear_page_erms
     0.34%  bench    [kernel.kallsyms]  [k] native_irq_return_iret
     0.31%  bench    bench              [.] bench_one

```

The assembly for the biggest offender, `msort_with_tmp` looks like this[4](#fn:annotate-command) :

```
 Percent | Address      | Disassembly
--------------------------------------------------
   30.55 :   39200:       mov    rax,QWORD PTR [r15]
    0.61 :   39203:       sub    rbp,0x1
    0.52 :   39207:       add    r15,0x8
    7.30 :   3920b:       mov    QWORD PTR [rbx],rax
    0.39 :   3920e:       add    rbx,0x8
    0.07 :   39212:       test   r12,r12
    0.09 :   39215:       je     390e0   ; merge finished
    1.11 :   3921b:       test   rbp,rbp
    0.01 :   3921e:       je     390e0   ; merge finished
    5.24 :   39224:       mov    rdx,QWORD PTR [rsp+0x8]
    0.42 :   39229:       mov    rsi,r15
    0.19 :   3922c:       mov    rdi,r13
    6.08 :   3922f:       call   r14
    0.59 :   39232:       test   eax,eax
    3.52 :   39234:       jg     39200
   32.69 :   39236:       mov    rax,QWORD PTR [r13+0x0]
    1.31 :   3923a:       sub    r12,0x1
    1.01 :   3923e:       add    r13,0x8
    1.09 :   39242:       jmp    3920b

```

Depending on your level of assembly reading skill, it may not be obvious, but this is basically a classic merge routine: it is merging two lists by comparing the top elements of each list (pointed to by `r13` and `r15`), and then storing the smaller element (the line `QWORD PTR [rbx],rax`) and loading the next element from that list. There are also two checks for termination ( `test r12,r12` and `test rbp,rbp`). This hot loop corresponds directly to this code from `glibc` (from the file `msort.c` [5](#fn:msort-note)) :

```
while (n1 > 0 && n2 > 0)
{
    if ((*cmp) (b1, b2, arg) <= 0)
    {
        *(uint64_t *) tmp = *(uint64_t *) b1;
        b1 += sizeof (uint64_t);
        --n1;
    }
    else
    {
        *(uint64_t *) tmp = *(uint64_t *) b2;
        b2 += sizeof (uint64_t);
        --n2;
    }
    tmp += sizeof (uint64_t);
}

```

This loop suffers heavily from branch mispredictions, since the “which element is larger” branch is highly unpredictable (at least for random-looking input data). Indeed, we see roughly 128 million mispredicts while sorting ~11 million elements: close to 12 mispredicts per element.

We also note the presence of the indirect call at the `call r14` line. This corresponds to the `(*cmp) (b1, b2, arg)` expression in the source: it is calling the user provided comparator function through a function pointer. Since the `qsort()` code is compiled ahead of time and is found inside the shared libc binary, there is no chance that the comparator, passed as a function pointer, can be inlined.

The comparator function I provide looks like:

```
int compare_uint64_t(const void *l_, const void *r_) {
    uint64_t l = *(const uint64_t *)l_;
    uint64_t r = *(const uint64_t *)r_;
    if (l < r) return -1;
    if (l > r) return  1;
    return 0;
}

```

which on gcc compiles to branch-free code:

```
mov    rax,QWORD PTR [rsi]
mov    edx,0xffffffff
cmp    QWORD PTR [rdi],rax
seta   al
movzx  eax,al
cmovb  eax,edx
ret

```

Note that the comparator has to redundantly load from memory the two locations to compare, something the merge loop already did (the merge loop reads them because it is responsible for moving the elements).

How much better could things get if we inline the comparator into the merge loop? That’s what we do in `qsort-inlined` [6](#fn:inline-hard), and here’s the main loop which now includes the comparator function[7](#fn:cmdline1) :

```asm
 0.07 :   401dc8:       test   rbp,rbp
 0.66 :   401dcb:       je     401e0c (msort_param const*, void*, unsigned long, CompareU64)+0xbc>
 3.51 :   401dcd:       mov    rax,QWORD PTR [r9]
 5.00 :   401dd0:       lea    rdx,[rbx+0x8]
 1.62 :   401dd4:       mov    rcx,QWORD PTR [rbx]
 0.24 :   401dd7:       lea    r8,[r9+0x8]
 6.96 :   401ddb:       cmp    rax,rcx
20.83 :   401dde:       cmovbe r9,r8
 8.88 :   401de2:       cmova  rbx,rdx
 0.27 :   401de6:       cmp    rcx,rax
 6.23 :   401de9:       sbb    r8,r8
 0.74 :   401dec:       cmp    rcx,rax
 4.93 :   401def:       sbb    rdx,rdx
 0.24 :   401df2:       not    r8
 6.69 :   401df5:       add    rbp,rdx
 0.44 :   401df8:       cmp    rax,rcx
 5.34 :   401dfb:       cmova  rax,rcx
 5.96 :   401dff:       add    rdi,0x8
 7.48 :   401e03:       mov    QWORD PTR [rdi-0x8],rax
 0.00 :   401e07:       add    r15,r8
 0.71 :   401e0a:       jne    401dc8 (msort_param const*, void*, unsigned long, CompareU64)+0x78>

```

A key difference is that the core of the loop is now branch free. Yes, there are still two conditional jumps, but they are both just checking for the termination condition (that one of the lists to merge is exhausted), so we expect this loop to be free of branch mispredictions other than the final iteration. Indeed, we measure with `perf stat` that the misprediction rate has dropped from to close to 12 mispredicts per element to around 0.75 per element. The loop has only two loads and one store, so the memory access redundancy between the merge code and the comparator has been eliminated[8](#fn:load-redundancy). Finally, the comparator does a three-way compare (returning distrinct results for `<`, `>` and `==`), but the merge code only needs a two-way compare ( `<=` or `>`) \- inlining the comparator manages to remove extra code associated with distinguishing the `<` and `==` cases.

What’s the payoff? It’s pretty big:

![Effect of comparator inlining](/assets/2019-05-22/fig2.svg)

The speedup hovers right around 1.77x. Note that this is much larger than simply eliminating all the time spent in the separate comparator function in the original version (about 17% of the time implying a speedup of 1.2x if all the function time disapeared). This is a good example of how inlining isn’t just about removing function call overhead but enabling further *knock on* optimizations which can have a much larger effect than just removing the overhead associated with function calls.

## What about C++?

Short of copying the existing glibc (note: LGPL licenced) sorting code to allow inlining, what else can we do to speed things up? I’m writing in C++, so how about the C++ sort functions available in the `` header? Unlike C’s `qsort` which is generic by virtue of taking a function pointer and information about the object size, the C++ sort functions use templates to achieve genericity and so are implemented directly in header files. Since the sort code and the comparator are being compiler together, we expect the comparator to be easily inlined, and perhaps other optimizations may occur.

Without further ado, let’s just throw `std::sort`, `std::stable_sort` and `std::partial_sort` into the mix:

![C vs C++ sort functions](/assets/2019-05-22/fig3.svg)

The C++ sort functions, other than perhaps `std::partial_sort` [9](#fn:partial-sort), put in a good showing. It is interesting that `std::stable_sort` which has *stricly more requirements* on its implementation than `std::sort` (i.e., any stable sort is also suitable for `std::sort`) ends up faster. I re-wrote this paragaph several times, since sometimes after a reboot `stable_sort` was slower and sometimes it was faster (as shown above). When it was “fast” it had less than 2% branch mispredictions, and when it was slow it was at 15%. So perhaps there was some type of aliasing issue in the branch predictor which depends on the physical addresses assigned, which can vary from run to run, I’m not sure. See [10](#fn:stablesort) for an old note from when `std::stable_sort` was slower.

## Can we do better?

So that’s as fast as it gets, right? We aren’t going to beat `std::sort` or `std::stable_sort` without a huge amount of effort, I think? After all, these are presumably highly optimized sorting routines written by the standard library implementors. Sure, we might expect to be able to beat `qsort()`, but that’s mostly because of built-in disadvantages that `qsort` has, lacking the ability to inline the comparator, etc.

### Radix Sort Attempt 1

Well, one thing we can try is a non-comparison sort. We know we have integer keys, so why stick to comparing numbers pairwise - maybe we can use something like [radix sort](https://en.wikipedia.org/wiki/Radix_sort) to stick them directly in their final location.

We can pretty much copy the description from the wikipedia article into C++ code that looks like this:

```
const size_t    RADIX_BITS   = 8;
const size_t    RADIX_SIZE   = (size_t)1 << RADIX_BITS;
const size_t    RADIX_LEVELS = (63 / RADIX_BITS) + 1;
const uint64_t  RADIX_MASK   = RADIX_SIZE - 1;

using queuetype = std::vector<uint64_t>;

void radix_sort1(uint64_t *a, size_t count)
{
    for (size_t pass = 0; pass < RADIX_LEVELS; pass++) {
        uint64_t shift = pass * RADIX_BITS;
        std::array<queuetype, RADIX_SIZE> queues;

        // copy each element into the appropriate queue based on the current RADIX_BITS sized
        // "digit" within it
        for (size_t i = 0; i < count; i++) {
            size_t value = a[i];
            size_t index = (value >> shift) & RADIX_MASK;
            queues[index].push_back(value);
        }

        // copy all the queues back over top of the original array in order
        uint64_t* aptr = a;
        for (auto& queue : queues) {
            aptr = std::copy(queue.begin(), queue.end(), aptr);
        }
    }
}

```

That’s about as simple as it gets. We decide to use one byte (i.e., radix-256) as the size of our “digit” (although it’s easy to change by adjusting the `RADIX_BITS` constant) and so we make 8 passes over our `uint64_t` array from the least to most significant byte. At each pass we assign the current value to one of 256 “queues” (vectors in this case) based on the value of the current byte, and once all elements have been processed we copy each queue in order back to the original array. We’re done - the list is sorted.

How does it perform against the usual suspects?

![Radix 1](/assets/2019-05-22/fig4.svg)

Well it’s not *terrible*, and while it certainly has some issues at low element counts, it actaully squeezes into first place at 1,000,000 elements and is competitive at 100,000 and 10,000,000. Not bad for a dozen lines of code.

A quick check of `time ./bench Radix1` shows something interesting:

```
real    0m1.099s
user    0m0.552s
sys     0m0.548s

```

We spent about the same amount of time in the kernel ( `sys` time) as in user space. The other algorithms spend only a few lonely % in the kernel, and almost most of that is in the setup code, not in the actual sort.

A deeper look with `perf record && perf report` shows:

```
# Samples: 4K of event 'cycles:ppp'
# Event count (approx.): 2858148287
#
# Overhead  Command  Shared Object        Symbol
# ........  .......  ...................  ................................................
#
    29.02%  bench    bench                [.] radix_sort1
    26.16%  bench    libc-2.23.so         [.] __memmove_avx_unaligned
     4.93%  bench    [kernel.kallsyms]    [k] clear_page_erms
     4.46%  bench    [kernel.kallsyms]    [k] native_irq_return_iret
     3.61%  bench    [kernel.kallsyms]    [k] error_entry
     3.04%  bench    [kernel.kallsyms]    [k] swapgs_restore_regs_and_return_to_usermode
     2.99%  bench    [kernel.kallsyms]    [k] sync_regs
     1.94%  bench    bench                [.] main
     1.91%  bench    [kernel.kallsyms]    [k] get_page_from_freelist
     1.64%  bench    libc-2.23.so         [.] __memcpy_avx_unaligned
     1.59%  bench    [kernel.kallsyms]    [k] release_pages
     1.40%  bench    [kernel.kallsyms]    [k] __handle_mm_fault
     1.37%  bench    [kernel.kallsyms]    [k] _raw_spin_lock
     1.01%  bench    [kernel.kallsyms]    [k] __pagevec_lru_add_fn
     0.88%  bench    [kernel.kallsyms]    [k] handle_mm_fault
     0.86%  bench    [kernel.kallsyms]    [k] __alloc_pages_nodemask
     0.83%  bench    [kernel.kallsyms]    [k] unmap_page_range
     0.75%  bench    [kernel.kallsyms]    [k] try_charge
     0.74%  bench    [kernel.kallsyms]    [k] get_mem_cgroup_from_mm
     0.70%  bench    bench                [.] bench_one
     0.63%  bench    [kernel.kallsyms]    [k] __do_page_fault
     0.63%  bench    [kernel.kallsyms]    [k] __mod_zone_page_state
     0.49%  bench    [kernel.kallsyms]    [k] free_pcppages_bulk
     0.45%  bench    [kernel.kallsyms]    [k] page_add_new_anon_rmap
     0.43%  bench    [kernel.kallsyms]    [k] up_read
     0.40%  bench    [kernel.kallsyms]    [k] page_remove_rmap
     0.36%  bench    [kernel.kallsyms]    [k] __mod_node_page_state

```

I’m not even going to try to explain what `__pagevec_lru_add_fn` does, but the basic idea here is that we are spending a lot of time in the kernel, and we are doing that because we are allocating and freeing *a lot* of memory. Every pass we `push_back` every element into one of 256 vectors, which will be constantly growing to accomodate new elements, and then finally all the now-giant vectors are freed at the end of every allocation. That’s a lot of stress on the memory allocation paths in the kernel[11](#fn:memalloc).

### Radix Sort Attempt 2

Let’s try the first-thing-you-do-when-vector-is-involved-and-performance-matters; that is, let us `reserve()` memory for each vector before we start adding elements. Just throw this at the start of each pass:

```
for (auto& queue : queues) {
    queue.reserve(count / RADIX_SIZE * 1.2);
}

```

Here, `1.2` is an arbitrary fudge factor to account for the fact that some vectors will get more than the average number of elements. The exact value doesn’t matter much as long as it’s not too small (0.9 is a bad value, almost evey vector needs a final doubling). This gives use [`radix_sort2`](https://github.com/travisdowns/sort-bench/blob/master/radix2.cpp) and let’s jump straight to the results (I’ve removed a couple of the less interesting sorts to reduce clutter):

![Radix 2](/assets/2019-05-22/fig5.svg)

I guess it’s a bit better? It does better for small array sizes, probably because the overhead of constantly resizing the small vectors is more significant there, but it is actually a bit slower for the middle sizes. System time is lower but still quite high:

```
real	0m0.904s
user	0m0.523s
sys	0m0.380s

```

What we really want is to stop throwing away the memory we allocated every pass. Let’s move the queues outside of the loop and just clear them every iteration:

```
void radix_sort3(uint64_t *a, size_t count)
{
    using queuetype = std::vector<uint64_t>;

    std::array<queuetype, RADIX_SIZE> queues;

    // we keep the reservation code (now outside the loop),
    // although it matters less now since the resizing will
    // generally only happen in the first iteration
    for (auto& queue : queues) {
        queue.reserve(count / RADIX_SIZE * 1.2);
    }

    for (size_t pass = 0; pass < RADIX_LEVELS; pass++) {
        // ... as before

        // copy all the queues back over top of the original array in order
        uint64_t* aptr = a;
        for (auto& queue : queues) {
            aptr = std::copy(queue.begin(), queue.end(), aptr);
            queue.clear();  // <--- this is new, clear the queues
        }
    }
}

```

Yes another graph with Radix3 included this time:

![Radix 3](/assets/2019-05-22/fig6.svg)

That looks a lot better! This radix sort is always faster than our earlier attempts and the fastest overall for sizes 10,000 and above. It still falls behind the `std::` algorithms for the 1,000 element size, where the `O(n)` vs `O(n*log(n))` difference doesn’t play as much of a role. Despite this minor victory, and while system is reduced, we are *still* spending 30% of our time in the kernel:

```
real    0m0.612s
user    0m0.428s
sys     0m0.184s

```

### Pre-sizing the Queues

Sorting should be about my code, not the kernel - so let’s get rid of this kernel time for good.

To do that, we’ll move away from `std::vector` entirely and just allocate one large temporary region for all of our queues. Although we know the *total* final size of all the queues (it’s the same size as the input array), we don’t know how big any *particular* queue will be. This means we don’t know exactly how to divide up the region. A well-known solution to this problem is to first count the number of number of values that will fall into each queue so they can be sized appropriately (also known as taking the histogram of the data). As a bonus, we can count the frequencies for all radix passes in a single trip over the data, so we expect this part to be much cheaper than the radix sort proper which needs a separate pass for each “digit”.

Knowing the size of each queue allows us to pack all the values exactly within a single temporary region. The copy at the end of each stage is just a single linear copy. The code is longer now since we need to implement the frequency counting gives us [radix\_sort4](https://github.com/travisdowns/sort-bench/blob/f05c53d02f8f374486c0f445ef519c1f47be95ce/radix4.cpp#L31). Results:

![Radix 4](/assets/2019-05-22/fig7.svg)

It’s a significant speedup over Radix 3, especially at small sizes (speedup about 3x) but still good at large sizes (about 1.3x for 10m elements). The speedup over poor `qsort` ranges from 3.7x to 5.45x, increasing at larger sizes. Even compared to the best contented from the standard library, `std::stable_sort`, the speedup averages about 2x.

### Are We Done Yet?

So are we done yet? Can we squeeze out some more performance?

One little trick is to note that the temporary “queue area” and the original array are now of the same size and type, so rather than always performing the radix passes from the original array to the temporary area (which requires a copy back to the original array each time), we can instead copy back and forth between these two areas, alternating the “from” and “to” areas each time. This saves a copy each pass.

The results in [radix\_sort5](https://github.com/travisdowns/sort-bench/blob/f05c53d02f8f374486c0f445ef519c1f47be95ce/radix5.cpp#L31) and it provides a small but measurable benefit:

![Radix 5](/assets/2019-05-22/fig8.svg)

It’s also interesting how *small* the improvement is. This change actually cuts the memory bandwidth requirements of the algorithm almost exactly in half: rather than reading and writing each element twice during each pass (once during the sort and once in the final copy), we read them only once (it’s not *exactly* half because the single histogramming pass adds another read). Yet the overall speedup is small, in the range of 1.05x to 1.2x. From this we can conclude that we are not approaching the memory bandwidth limits in the radix passes.

There is a catch here: at the end of the sort, if we have done an *odd* number of passes, the final sorted results will be in the temporary area, not in the original array, so we need to copy back to the original array - but 1 extra copy is better than 8! In any case, with `RADIX_BITS == 8` as we’ve chosen, there are an even number of copies, so this code never executes in our benchmark.

### Pointless Work is Pointless

Another observation we can make is that for this input (and many inputs in the real world), many of the radix passes do nothing. All the input values are less than 40,000,000,000. In 64-bit hex that looks like `0x00000009502F9000` \- the top 28 bits are always zero. Any radix pass that uses these all-zero bits is pointless: every element will be copied to the first queue entry, one by one: essentially it’s a slow, convoluted `memcpy`.

We can simply skip these “trivial” passes by examining the frequency count: if all counts are zero except a single entry, the pass does nothing. This gives us [radix\_sort6](https://github.com/travisdowns/sort-bench/blob/master/radix6.cpp), which ends up cutting out 3 of the 8 radix passes leading to performance like this (I’ve changed the scale to emphasize the faster algorithms as they were getting crowed down at the bottom):

![Radix 6](/assets/2019-05-22/fig9.svg)

In relative terms this provides a significant speedup ranging from 1.2x to 1.5x over `radix_sort5`. The theoretical speedup from skipping 3 of the 8 passes is 1.6x, but we don’t achieve that because there is work outside of the core passes (counting the frequencies, for example) and also because the 3 trivial passes were actually slightly faster than the non-trivial ones because of better caching behavior.

### Unpointless Prefetch

So how much more juice can we squeeze from this performance orange? Will this post ever come to an end? Has anyone even made it this far?

As it turns out we’re not done yet, and the next change is perhaps the easiest one yet, a one-liner. First, let us observe that the core radix sort loop does a linear read through the elements (very prefetch and cacheline locality friendly), and then a *scattered store* to one of 256 locations depending on the value. We might expect that those scattered stores are problematic, since we don’t expect the prefetcher to track 256 different streams. On the other hand, stores are somewhat “fire and forget”, because after we execute the store, they can just sit around in the store buffer for as long as needed while the associated cache lines are fetched: they don’t participate in any dependency chains[12](#fn:not-sfw). So maybe they aren’t causing a problem?

Let’s check that theory using the `resource_stalls.sb` event, which tells us how many cycles we stalled store buffer was full, using this magical invocation:

```
for i in {3..8}; do s=$((10**$i)); rm -f bench.o; echo "SIZE=$s, KB=$(($s*8/1000))"; make EFLAGS=-DSIZE=$s; perf stat -e cycles,instructions,resource_stalls.any,resource_stalls.sb ./bench Radix6; done

```

This tests a variety of different sizes and here’s typical output when the array to sort has 1 million elements (8 MB):

```
SIZE=1000000, KB=8000
...
 Performance counter stats for './bench Radix6':

     5,544,710,461      cycles
     8,219,301,287      instructions              #    1.48  insn per cycle
     2,917,340,919      resource_stalls.any
     2,454,399,834      resource_stalls.sb

```

out of 5.54 billion cycles, we are stalled because the store buffer is full in 2.45 billion of them. So … a lot.

One fix for this is a one-liner in the main radix-sort loop:

```
for (size_t i = 0; i < count; i++) {
    size_t value = from[i];
    size_t index = (value >> shift) & RADIX_MASK;
    *queue_ptrs[index]++ = value;
    __builtin_prefetch(queue_ptrs[index] + 1); // <-- the magic happens here
}

```

That’s it. We prefetch the next plus one position in the queue after writing to the queue. 87.5% of the time this does nothing, since the next position is either in the same cache line (6 out of 8 times) or we already prefetched it the last time we wrote to this queue (1 out of 8 times).

The other 12.5% of the time it helps, producing results like this:

![Radix 7](/assets/2019-05-22/fig10.svg)

The speedup is zero at the smallest size (1000 elements, aka 8 KB) which fits in L1, but ranges between 1.31x and 1.45x as soon as the data set exceeds L1. In principle, I wouldn’t expect prefetch to help here: we expect it to help for loads if we can start the load early, but for stores, with a full store buffer the CPU can already pick from 50+ stores to start prefetching. That is, the CPU already *knows* what stores are coming becaue the store buffer is full of them. However, in practice, theory and practice are different and in particular Intel CPUs seem to struggle when loads that hit in L1 are interleaved with loads that don’t, [especially with recent microcode](/blog/2019/03/19/random-writes-and-microcode-oh-my.html) [13](#fn:microcode).

That interleaved scenario will happen all the time with this type of scattered write pattern, and as noted in the earlier post, prefetch is one way of mitigating this. For the 8 MB working set we now have the following `perf stat` results:

```
SIZE=1000000, KB=8000
...
 Performance counter stats for './bench Radix7':

     4,342,298,986      cycles
     8,719,315,140      instructions              #    2.01  insn per cycle
     1,555,574,009      resource_stalls.any
       623,342,154      resource_stalls.sb

       1.675302704 seconds time elapsed

```

About a four-fold reduction in store-buffer stalls. Reductions are even more dramatic for other sizes: the 100,000 element (8 KB) size has an even larger reduction, from being stalled 43% of the time down to 5% after prefetching is added.

## What’s Next?

What’s next is that I have to eat. So while we’re not done here yet, the remaining part of this trip down the radix sort hole will have to wait for part 2.

If you liked this post, check out the [homepage](/) for others you might enjoy.

Boxing photo by [Hermes Rivera](https://unsplash.com/@hermez777) on [Unsplash](https://unsplash.com).

---

---

1. Note that wanting *distinct* values here is key. Without that restriction, the problem is simple: just use the output of any decent random number generator. Returning only distinct entries is trickier: you have to find a way to avoid or remove duplicate elements. Well it’s not all that tricky: you can simply remember existing elements using a `std::set` and reject any duplicate elements. What is tricky is doing it quickly. [↩](#fnref:distinct)

2. In particular, the method he suggests picks random values one-at-a-time, already in sorted order, deciding on the gap to the next number by drawing randomly from a geometric distribution. Read his post for full details, but the one sentence summary is that Dan finds that the geometric distribution approach has favorable performace results compared to the sorted one (more than 2x as fast). Both this method and the sorting method have the little problem that the resulting list may be smaller than requested. For example in the case of the sorting method, the output after de-duplication is smaller if there are any collisions (and due to the [Birthday Problem](https://en.wikipedia.org/wiki/Birthday_problem) that is a lot likelier than you might think). One could cope with this by generating slightly more numbers than required, and then removing randomly selected elements until the list reaches its desired size. [↩](#fnref:danmethod)

3. There is a long tail of other functions here, but they add up to only about 2% of the runtime, so you can safely ignore them. [↩](#fnref:long-tail)

4. I get the annotated assembly with `perfc annotate -Mintel --stdio --symbol=msort_with_tmp.part.0` \- this only shows assmebly because the function is in glibc and there are no debug symbols for that library on this host. In any case, assembly is probably what we want. [↩](#fnref:annotate-command)

5. `msort.c` implements a mergesort algoirhtm. There is also a file `qsort.c` which implements a traditional partition-around-a-pivot based quicksort, but it seems not used to implement `qsort()` on recent gcc versions. [↩](#fnref:msort-note)

6. It turns out that inlining the comparator is not simply a matter of compiling `msort.c` and the comparator in the same translation unit while still passing the comparator as a function pointer, so the compiler can see the definition. This doens’t pan out because msort (a) is a recursive function, which means unlimited inlining isn’t possible (so at some point there is an out-of-line call where the identity of the comparator will be lost) and (b) the comparator function is saved to memory (in the `msort_param` struct) and used from there, which makes it harder for compilers to prove the comparator is always the one originally passed in. Instead, I use a template version of msort which takes the comparator of arbitrary type `C`, and in the case that a functor object is passed, the comparator is built right into the signature of the function, making inlining the comparator basically automatic for any compiler. [↩](#fnref:inline-hard)

7. We obtain this output with `perf record ./bench qsort-inlined && perfc annotate -Mintel --stdio --no-source --symbol='msort_with_tmp'`, if your version of `perf` supports de-mangling names (otherwise you’ll have to use the mangled name). [↩](#fnref:cmdline1)

8. Technically, there is still some redundancy since we load *two* elements every iteration, whereas we really only need to load one: the new element from whichever list an element was removed from. You can still do that in a branch-free way, but the transformation is aparently beyond the capability of the compiler. [↩](#fnref:load-redundancy)

9. It’s hard to blame `std::partial_sort` here, after all it is a specialized sort for cases where you want need sort only a subset of the input to be sorted, e.g., the first 100 elements of a 100,000 element sequence. However, we can use it as a full sort simply by specifying that we want the full range, but one would not expect the algorithm to be optimized for that case. [↩](#fnref:partial-sort)

10. I had to edit this part of the blog entry because something weird happened: originally, my results always showed that `std::stable_sort` was faster than `std::sort` across all input sizes. After installing a bunch of OS packages and restarting, however, `std::stable_sort` performance came back down to earth, around 1.4x slower than before and now slower than `std::sort` across all input sizes. I don’t know what changed. I did find that before the change `std::stable_sort` had very few (< 1%) branch mispredictions, while after it had many mispredictions (about 15%). [↩](#fnref:stablesort)

11. This behavior actually depends pretty heavily on your memory allocator. The default glibc allocator I’m using likes to give memory allocations above a certain size back to the OS whenever they are freed, which means this workload turns into a `mmap` and `munmap` workout for the kernel. Using an allocator that wasn’t too worried about memory use and kept these pages around for its own use would result in a very different profile. [↩](#fnref:memalloc)

12. Stores *never* participate in normal “register carried” dependency chains, since they do not write any registers, but they can still participate in chains through memory, e.g., if a store is followed by a load that reads the same location, the load depends on the store (and efficiently this dependency is handled depends on a lot on the hardware). This case doesn’t apply here because we don’t read the recently written queue locations any time soon: our stores are truly “fire and forget”. [↩](#fnref:not-sfw)

13. In fact, if you have an older microcode on your machine, you will see different results: there will be very little difference between Radix6 and Radix7 because Radix6 is considerably faster if you don’t update your microcode. [↩](#fnref:microcode)

---

## Comments

Simon • [March 31th, 2026 20:23](#comment-6bb29010-2d3f-11f1-9fc0-1550c9c5cbc6 "Permalink to this comment")

I really enjoyed your article after messing around with optimising a mergesort program. This is just the kind of quality content I want to see on the internet, but Google thinks you only deserve to appear on page 6 after mostly high level dross comparing big Os. It would be great to see part2, but I just wanted to say thanks for sharing this. I’m going to take a look at your other posts….

↪︎ Reply to Simon

---

Robert Clausecker • [August 4th, 2025 11:07](#comment-2c410ca0-7123-11f0-8b75-994618a9dce8 "Permalink to this comment")

You may also be interested in Jan Wassenberg’s 2010 paper “Faster radix sort via virtual memory and writecombining” (arXiv 1008.2849) which proposes a method of software-defined write-combining to avoid the read-to-own loads entirely.

↪︎ Reply to Robert Clausecker

---

Brian Green • [May 24th, 2022 16:22](#comment-c0335fb0-db7d-11ec-8183-83c617056737 "Permalink to this comment")

Do you have any insight into why the prefetch is to the next element *plus one*? Naively, it seems like we should prefetch the next output position instead of skipping one element. But then if +1 works, then why not +7 or +8 to try to fetch the next cache line?

It feels like secret magic to me.

↪︎ Reply to Brian Green

Author Travis Downs • [May 28th, 2022 23:35](#comment-d9c7d1e0-dede-11ec-af05-b7bfec67ee79 "Permalink to this comment")

Hi Brian,

I can’t give you a very principled reason why the prefetch is +2 from the last written element. It is possible that I intended to do +1 from the last written element and this is simply a bug. More likely, I tried a few values and settled on 1 as providing a good result empirically.

This “just experiment” approach is typical of prefetch distance: you can try to determine the “best” distance by modelling the expected memory behavior but this is in general very difficult without a detailed model of the CPU and memory subsystem, and even then it would only apply to that specific hardware configuration. So it ends up being more practical just to try a bunch of different prefetch values and settle on the best one.

If I recall correctly, the exact value here didn’t make much difference: the key is that the next line is brought in at some point before we start using it, so +1, +2, +3, all serve mostly the same purpose because they’ll all fall into the next line as we reach the end of the previous one: the main difference is that larger offsets bring in the next line slightly earlier, with the upside of more latency hiding if several elements are written in a burst to that same bucket, and the downside of a higher chance of a “useless prefetch” where the line is evicted before we use it.

---

Observer • [May 8th, 2021 19:08](#comment-bf577230-b030-11eb-ac11-970eadd5eac5 "Permalink to this comment")

Part 2 when?

↪︎ Reply to Observer

Author Travis Downs • [May 11th, 2021 04:07](#comment-5ef93a10-b20e-11eb-98b6-21cb6d5229ef "Permalink to this comment")

Good question. It’s half written but I haven’t made progress on it in over a year. I’ve experienced some changes in my life that means that “maybe never” might be the most realistic answer.

I may already release an abbreviated version based mostly on what I have now, but it would be a much weaker post in that case.

---

psionl0 • [January 20th, 2021 20:32](#comment-af4c2ab0-5b5e-11eb-89df-efcf6a0d3088 "Permalink to this comment")

Fancy that! It’s called “qsort” but it actually does a merge sort.

↪︎ Reply to psionl0

---

Noah Goldstein • [January 13th, 2021 05:01](#comment-731b4010-555c-11eb-bfb5-01430c7bddbd "Permalink to this comment")

If its just sorting integers your after you might find [this](https://github.com/goldsteinn/SIMD-sorting-network) cool

↪︎ Reply to Noah Goldstein

Author Travis Downs • [January 29th, 2021 05:45](#comment-232ebf00-61f5-11eb-8073-2b87c96cc065 "Permalink to this comment")

Thanks for your comment Noah. I have looked into sorting networks and I may have more to write on this in the future. Your link is handy, I’m filing it away for when I look at this again.

‍ • [July 3th, 2021 08:12](#comment-61b331b0-dbd6-11eb-9689-b9e2acca48b3 "Permalink to this comment")

You might even be interested in https://github.com/simd-sorting/fast-and-robust which is a Quicksort implementation using vectorized sorting networks for small arrays.

---

Gautham • [August 21th, 2020 12:51](#comment-fbb93340-e3ac-11ea-9955-9556da4b6cf3 "Permalink to this comment")

It is possible to do radix sort in-place, as given in [Wikipedia](https://en.wikipedia.org/wiki/Radix_sort). Instead of having `queue_ptrs` consist of pointers to a second array, one can just use the existing array and swap in-place, with a check to avoid out-of-bounds for each bucket.

An additional trade-off is that the in-place algorithm is not [stable](https://en.wikipedia.org/wiki/Sorting_algorithm#Stability), ie “equal” elements may not end up in the same order as they were in the input (I found this out the hard way, as I wanted to speed up a sorting of `vector>` after reading this post).

↪︎ Reply to Gautham

---

Jip • [May 3th, 2020 11:25](#comment-c02de000-8d30-11ea-9db0-e9d644cafe33 "Permalink to this comment")

Very interesting article !!! You seem to suggest that qsort is indeed weak in performance because of the iterative call of the compare function. If I understood the paper well, this function could be inlined, but it’s not!! You also suggest that the C++ standard sort is better than C qsort. Now, sorry, I’m not a C++ man. I’m just trying to learn C and I find it difficult enough !!! What would be nice would be to publish a better qsort than the one we find in stdlib. Could you do that ?? And also (if possible) explain all the improvements. I read a lot of articles about qsort. Some guys say : “don’t worry ! qsort is a very good general purpose sorting algorithm. It has been done by very good programmers. Just use it and be happy!”. Other guys say : “qsort could be much better. It has been done many years ago and no one bothered about improving this built-in function. It’s time to do it !!” Who is right ??? From what I read, you should be able to provide a better qsort function. Of course, I’m interested by the C version of it (no C++ please !!!). Thanks in advance. Although I didn’t understand everything, I liked the article.

↪︎ Reply to Jip

Author Travis Downs • [May 7th, 2020 18:35](#comment-85e9a140-9091-11ea-9170-4181cea7519a "Permalink to this comment")

> Very interesting article !!! You seem to suggest that qsort is indeed weak in performance because of the iterative call of the compare function. If I understood the paper well, this function could be inlined, but it’s not!! You also suggest that the C++ standard sort is better than C qsort.

Yes, more or less. Essentially, the C sort comparator function cannot be inlined, since the C sort is implemented in a separate translation unit (C file) and compiled once and it lives in your libc shared object. So the comparator, which is called many times in the hottest part of the sort, is necessarily a function call, which adds overhead in various ways: especially because the comprands are passed by pointer so end up being loaded multiple times rather than passed in registers.

On the other hand, C++ std::sort, by its very template nature is implemented in a header and so gets specialized for every type and every comparator function. This generally makes it faster, but also bloats code size. So it is not a pure win-win, but something of a tradeoff.

> What would be nice would be to publish a better qsort than the one we find in stdlib.

Well I do plan to publish some usable radix sorts when this is done, but I don’t think is is very obvious how to write a C qsort that improves significantly on the existing qsort. In relation to the inlining aspect, you could do is a qsort implementation that is implemented in a header in C, just like C++ and rely on the optimizer to inline the comparator function. This isn’t totally straightforward, as [described in a footnote](https://travisdowns.github.io/blog/2019/05/22/sorting.html#fn:inline-hard) but I’m sure it could be done.

This comes with all the code bloat problems of the C++ version, however.

In any case, I don’t have specific plans to try to improve on that aspect of qsort: I’m mostly interested in improving radix sort.

Nathan Myers • [January 15th, 2021 03:05](#comment-79a1afe0-56de-11eb-9052-e35cdcba2ae4 "Permalink to this comment")

C is quite a lot harder to code (correctly) in than C++. So, it is a grave error to avoid C++.

Noah Goldstein • [October 20th, 2021 20:36](#comment-5936acf0-31e5-11ec-b62a-5b9ea25a934c "Permalink to this comment")

@Travis Downs (there seems to be no way to reply to a reply)

> Well I do plan to publish some usable radix sorts when this is done, but I don’t think is is very obvious how to write a C qsort that improves significantly on the existing qsort. In relation to the inlining aspect, you could do is a qsort implementation that is implemented in a header in C, just like C++ and rely on the optimizer to inline the comparator function. This isn’t totally straightforward, as described in a footnote but I’m sure it could be done.

There is a bit of work being down to optimize GLIBC’s qsort in [this patchset](https://patchwork.sourceware.org/project/glibc/list/?series=2745).

The reality is most of the optimization is for faster swap and to make qsort safe against the `O(n^2)` worst case. In many ways [GLIBC’s qsort is still living in the 1990s….](https://sourceware.org/git/?p=glibc.git;a=blob;f=stdlib/qsort.c;h=0f6d7ff787b87b3a432177d8bf662b3a47cdc64d;hb=HEAD#l42).

We explored writing an optimized version for specific types by providing type specific compare functions then checking on `qsort` entry if the compare function used was one of the known types i.e

```
qsort(...) {
    if(cmpfunc == INT_CMP) {
        optimized_int_qsort(...);
    }
...
}

```

We have implementation for `uint32_t` (was able to beat your radix7 up to size==8192 for random and to IIRC ~2^20 for semi-sorted data) but decided these pure typed qsort’s are not particularly common in real code and mostly just used for benchmarking so it wasn’t worth the code size.

---

## Add Comment

Name

E-mail

Cancel Reply

Submit

Close
