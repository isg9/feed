---
title: 3Hz Computer, Hold the Transistors
url: /2022/07/24/curta
published: "2022-07-24T00:00:00Z"
feed: mcyoung
guid: /2022/07/24/curta
---

# 3Hz Computer, Hold the Transistors

I’m not really one to brag publicly about expensive toys, but a few weeks ago I managed to get one that’s really something special. It is a *Curta Type II*, a mechanical digital[1](#fn:1) calculator manufactured in Liechtenstein between the 50s and 70s, before solid-state calculators killed them and the likes of slide-rules.

I have wanted one since I was a kid, and I managed to win an eBay auction for one.

![The Curta](/assets/images/curta/with_case.jpg)

The Curta Type II (and Solomon the cat)

It’s a funny looking device, somewhere between a peppermill and a scifi grenade. Mine has serial number 544065, for those keeping score, and comes in a cute little bakelite pod (which has left hand thread?!).

I wanna talk about this thing because unlike something like a slide rule, it shares many features with modern computers. It has operations, flags, and registers. Its core primitive is an adder, but many other operations can be built on top of it: it is very much a platform for complex calculations.

I’m the sort of person who read *Hacker’s Delight* for fun, so I really like simple numerical algorithms. This article is a survey of the operation of a Curta calculator and algorithms you can implement on it, from the perspective of a professional assembly programmer.

Many of the algorithms I’m going to describe here exist online, but I’ve found them to be a bit difficult to wrap my head around, so this article is also intended as a reference card for myself.

Let’s dive in!

## [A Well-Lubricated ALU](\#a-well-lubricated-alu)

There are two Curta models, Type I and Type II, which primarily differ in the sizes of their registers. I have a Type II, so I will focus on the layout of that one.

The Curta is not a *stored program* computer like the one you’re reading this article on. An operator needs to manually execute operations. It is as if we had taken a CPU and pared it down to two of its most basic components: a register file and an arithmetic logic unit (ALU).

### [The Register File](\#the-register-file)

The Curta’s register file consists of three digital registers, each of which contains a decimal integer (i.e., each digit is from `0` to `9`, rather than `0` to `1` like on a binary computer):

- `sr`, the *setting register*, is located on the side of the device. The value in `sr` can be set manually by the operator using a set of knobs on the side of the device. The machine will never write to it, only read from it. It has 11 digits.
- `rr`, the *results register*, is located at the top of the device along the black part of the dial. It is readable and writable by the machine, but not directly modifiable by the operator. It has 15 digits.
- `cr`, the *counting register*, is located next to `rr` along the silver part of the dial. Like `rr`, it is only machine-modifiable. It has 8 digits.

![](/assets/images/curta/sr.jpg)

`sr`, set to `1997`.

![](/assets/images/curta/cr_rr.jpg)

`rr` is the black dial; `cr` is the silver one.

There are also two settings on the device that aren’t really registers, but, since they are changed as part of operation, they are a lot like the control registers of a modern computer.

The *carriage* (there isn’t an abbreviation for this one, so I’ll call it `ca`) is the upper knurled ring on the machine. It can be set to a value from `0` to `7` [2](#fn:2). To set it, the operator lifts the ring up (against spring tension), twists it, and lets it spring back into the detent for the chosen value. This is a one-hand motion.

There is a small triangle in the middle of the top of the device that points at which of the digits in `cr` will get incremented.

![](/assets/images/curta/ca.jpg)

`ca` raised and in motion.

Finally, `rl`, the *reversing lever*, is a small switch near the back of the device that can be in the up or down position. This is like a flag register: up is cleared, down is set.

![](/assets/images/curta/rl.jpg)

`rl` in the up position.

### [The Instruction Set](\#the-instruction-set)

We have all this memory, but the meat of a machine is what it can *do*. I will provide an *instruction set* for the Curta to aid in giving rigorous descriptions of operations you can perform with it.

The core operation of the Curta is “add-with-shift-and-increment”. This is a mouthful. At the very top of the machine is the handle, which is analogous to a clock signal pin. Every clockwise turn of this handle executes one of these operations. Internally, this is implemented using a variation on the [Leibniz gear](https://en.wikipedia.org/wiki/Leibniz_wheel), a common feature of mechanical calculators.

![](/assets/images/curta/pturn.jpg)

The handle in “addition” mode.

This operation is not that complicated, it just does a lot of stuff. It takes the value of `sr`, left-shifts it (in decimal) by the value in `ca`, and adds it to `rr`. Also, it increments `CR` by `1` shifted by `ca`. In other words:

```text highlight
rr += sr << ca
cr += 1 << ca

```

[Plaintext](#code:1)

Recall that this is a decimal machine, so `<<` is the same as multiplication by a power of 10, not a power of 2.

Addition can overflow, and it wraps around as expected: adding one to `999_999_999_999_999_999` already in `rr` will fill it with zeroes.

Pulling the handle up reveals a red ring, indicating the machine is in *subtraction mode*. This flips the signs of both the `rr` and `cr` modifications:

```text highlight
rr -= sr << ca
cr -= 1 << ca

```

[Plaintext](#code:2)

![](/assets/images/curta/mturn.jpg)

The handle in “subtraction” mode.

The Curta cannot handle negative numbers, so it will instead display the ten’s complement[3](#fn:3) of a negative result. For example, subtracting `1` from `0` will produce all-nines.

You can detect when underflow or overflow occurs when the resulting value is unexpectedly larger or smaller than the prior value in `rr`, respectively. (This trick is necessary on architectures that lack a carry flags register, like RISC-V.)

Setting `rl` will reverse the sign of the operation done on `cr` during a turn of the handle. In addition mode, it will cause `cr` to be subtracted from, while in subtraction mode, it will cause it to be added to. Some complex algorithms make use of this.

Finally, the *clearing lever* can be used to clear (to zero) `sr` or `rr`, independently. It is a small ring-shaped lever that, while the carriage is raised, can be wiped past digits to clear them. Registers cannot be partially

![](/assets/images/curta/pturn.jpg)

The clearing lever.

### [Notation](\#notation)

Let’s give names to all the instructions the operator needs to follow, so we can write some assembly:

- `mr`, or *Machine Ready!*, means to clear/zero every register. All Curta instructions use the term “Machine Ready” to indicate the beginning of a calculation session.
- `pturn` is the core addition operation, a “plus turn”.
- `mturn` is its subtraction twin, a “minus turn”.
- `set <flag>` requests the operator set one of `rl` or `sm`.
- `clr <flag>` is the opposite of `set`.
- `zero <reg>` request a clear of one of `rr` or `cr` using the clearing lever.
- `add <reg>, <imm>` requests manual addition of an immediate to `sr` or `ca`. This is limited by what mental math we can ask of the operator.
- `copy <reg>, sr` requests a copy of the value in `rr` or `cr` to `sr`.
- `wrnp <reg>, <symbol>` indicates we need to write down a value in any register to a handy notepad (hence `wr` ite `n` ote `p` ad), marked with `<symbol>`.
- `rdnp <reg>, <symbol>` asks the operator to `re` ad a value recorded with `wrnp`.
- `if <cond>, <label>` asks the operator to check a condition (in terms of `cr`, `rr`, and `sr`) and, if true, proceed to the instruction at the given `label:`. Here’s some examples of conditions we’ll use:
  - `rr == 42`, i.e., `rr` equals some constant value.
  - `rr.ovflow`, i.e., `rr` overflowed/underflowed due to the most recent `pturn`/ `mturn`.
  - `cr[1] == 9`, i.e. `cr`’s second digit (zero-indexed, not like the physical device!) equals `9`.
  - `cr[0..ca] < sr[0..ca]`, i.e., `cr`, considering only the digits up to the setting of `ca`, is less than those same digits in `sr`.
- `goto <label>` is like `if` without a condition.
- `done` means we’re done and the result can be read off of `rr` (or `cr`).

Note that there is a lot of mental math in some of the conditions. Algorithms on the Curta are aimed to minimize what work the operator needs to do to compute a result, but remember that it is only an ALU: all of the control flow logic needs to be provided by the human operator.

None of this is real code, and it is specifically for the benefit of readers.

## [Some Algorithms](\#some-algorithms)

So, addition and subtraction are easy, because there are hardware instructions for those. There is, however, no direct way to do multiplication or division. Let’s take a look at some of our options.

Given that a Curta is kinda expensive, you can try out an online simulator if you want to follow along. [This one](https://www.cailliau.org/en/Alphabetical/C/Computing/Curta/Simulator/) is pretty simple and runs in your browser.

### [Multiplication](\#multiplication)

The easiest way to do multiplication is by repeated addition; `cr` helps us check our work.

Given a value like `8364`, we can multiply it by `5` like so:

```curta highlight
mul_by_5:
  mr
  add   sr, 8364
loop:
    if    cr == 5, end
    pturn
    goto  loop
end:
  done

```

[Curta "Assembly"](#code:3)

Here, we input the larger factor into `sr`, and then keep turning until `cr` contains the other factor. The result is `41820`:

![](/assets/images/curta/mul.jpg)

`8364 * 5 == 41820`

Of course, this does not work well for complex products, such as squaring `41820`. You could sit there and turn the handle forty thousand times if you wanted to, or you might decided that you should get a better hobby, since modern silicon can do this in nanoseconds.

We can speed this up exponentially by making use of the distributive property and the fact that `turn` can incorporate multiplication by a power of `10`.

Consider:

```highlight
41820 * 41820
= 41820 * (40000 + 1000 + 800 + 20)
= 41820 * 40000 + 41820 * 1000 + 41820 * 800 + 41820 * 20

```

[Plaintext](#code:4)

Each nice round number here can be achieved in `cr` by use of `ca`. Our algorithm will look a bit like this:

```curta highlight
square:
  mr
  add   sr, 41820
loop:
    // Check if we're done.
    if    cr == 41820, end
  inner:
      // Turn until the first `ca` digits of `cr` and the
      // other factor match.
      if    cr[1..ca] == 41802[1..ca], inner_end
      pturn
      goto  inner
  inner_end:
    // Increment `ca` and repeat until done.
    add   ca, 1
    goto  loop
end:
  done

```

[Curta "Assembly"](#code:5)

There are two loops. The inner loop runs as many turns as is necessary to get the next prefix of the factor into `cr`, then incrementing `ca` to do the next digit, and on and on until `cr` contains the entire other factor, at which point we can read off the result.

The actual trace of operations (omitting control flow), and the resulting contents of the registers `sr/rr/mr/ca` at each step, looks something like this:

```curta highlight
mr
// 00000000000/000000000000000/00000000/0
add   sr, 41820
// 00000041820/000000000000000/00000000/0
add   ca, 1
// 00000041820/000000000000000/00000000/1
pturn
// 00000041820/000000000418200/00000010/1
pturn
// 00000041820/000000000083640/00000020/1
add   ca, 1
// 00000041820/000000000083640/00000020/2
pturn
// 00000041820/000000005018400/00000120/2
pturn
// 00000041820/000000009200400/00000220/2
pturn
// 00000041820/000000013382400/00000320/2
pturn
// 00000041820/000000017564400/00000420/2
pturn
// 00000041820/000000021746400/00000520/2
pturn
// 00000041820/000000025928400/00000620/2
pturn
// 00000041820/000000030110400/00000720/2
pturn
// 00000041820/000000034292400/00000820/2
add   ca, 1
// 00000041820/000000034292400/00000820/3
pturn
// 00000041820/000000076112400/00001820/3
add   ca, 1
// 00000041820/000000494312400/00011820/4
pturn
// 00000041820/000000912512400/00021820/4
pturn
// 00000041820/000001330712400/00031820/4
pturn
// 00000041820/000001748912400/00041820/4
pturn

```

[Curta "Assembly"](#code:6)

The result can be read off from `rr`: `1748912400`. In the trace, you can see `cr` get built up digit by digit, making this operation rather efficient.

![](/assets/images/curta/square.jpg)

`41820 * 41820 == 1748912400`

We can do even better, if we use subtraction. For example, note that `18 = 20 - 2`; we can build up `18` in `cr` by doing only 4 turns rather than nine, according to this formula. Here’s the general algorithm for `n * m`:

```curta highlight
mul:
  mr
  add   sr, n
loop:
    if    cr == m, end
    // Same as before, but if the next digit is large,
    // go into subtraction mode.
    if    m[ca] > 5, by_sub
  inner:
      if    cr[0..ca] == m[0..ca], inner_end
      pturn
      goto  inner
  by_sub:
    // Store the current `ca` position.
    wrnp  ca,   sub_from
    // Find the next small digit (eg. imagine n * 199, we
    // want to find the 1).
  find_small:
    add   ca,   1
    if    m[ca] > 5, find_small
    // Set the digit to one plus the desired value for that
    // digit.
  outer_turns:
    pturn
    if    cr[ca] != m[ca] + 1, outer_turns
    // Store how far we need to re-advance `ca`.
    wrnp  ca,   continue_from
    // Go back to the original `ca` position and enter
    // subtraction mode.
    rdnp  ca,   sub_from
  subs:
  subs_inner:
      // Perform subtractions until we get the value we want.
      if    cr[ca] == m[ca],  subs_end
      mturn
      goto  subs_inner
  subs_end:
    // Advance `ca` and keep going until we're done.
    add   ca,   1
    if    ca != continue_from, subs
    goto  loop
  inner_end:
    add   ca,   1
    goto  loop
end:
  done

```

[Curta "Assembly"](#code:7)

Although more complicated, if we execute it step by step, we’ll see we get to our answer in fewer turns:

```curta highlight
mr
// 00000000000/000000000000000/00000000/0
add   sr, 41820
// 00000041820/000000000000000/00000000/0
add   ca, 1
// 00000041820/000000000000000/00000000/1
pturn
// 00000041820/000000000418200/00000010/1
pturn
// 00000041820/000000000835400/00000020/1
add   ca, 2
// 00000041820/000000000835400/00000020/3
pturn
// 00000041820/000000042656400/00001020/3
pturn
// 00000041820/000000084476400/00002020/3
add   ca, -1
// 00000041820/000000084476400/00002020/2
mturn
// 00000041820/000000080294400/00001920/2
mturn
// 00000041820/000000076112400/00001820/2
add   ca, 2
// 00000041820/000000494312400/00011820/4
pturn
// 00000041820/000000912512400/00021820/4
pturn
// 00000041820/000001330712400/00031820/4
pturn
// 00000041820/000001748912400/00041820/4
pturn

```

[Curta "Assembly"](#code:8)

In exchange for a little overhead, the number of turns drops from 15 to 10. This is the fastest *general* algorithm, but some techniques from *Hacker’s* *Delight* can likely be applied here to make it faster for some products.

#### [Cubes](\#cubes)

As a quick note, computing the cube of a number without taking extra notes is easy, so long as the number is already written down somewhere you can already see it. After computing `n^2` by any of the methods above, we can do

```curta highlight
cube:
  mr
  add   sr,   n
  // Perform a multiplication by `n`, then copy the result
  // into `sr`.
  copy  sr,   rr
  zero  rr
  zero  cr
  // Perform another multiplication by `n`, but now with
  // its square in `sr`.
  done

```

[Curta "Assembly"](#code:9)

This sequence can be repeated over and over to produce higher powers, and is only limited by the size of `rr`.

### [Division](\#division)

Division is way more interesting, because it can be *inexact*, and thus produces a *remainder* in addition to the quotient. There are a few different algorithms, but the simplest one is division by repeated subtraction. Some literature calls this “division by breaking down”.

For small numbers, this is quite simple, such as `21 / 4`:

```curta highlight
div_by_4:
  mr
  add   sr,   21
  pturn
  zero  cr
  zero  sr
  add   sr,   4
  set   rl
loop:
    if    rr.oflow, end
    mturn
    goto  loop
end:
  pturn
  done

```

[Curta "Assembly"](#code:10)

This works by first getting the dividend into `rr` and resetting the rest of the machine. Then, with `rl` set, we subtract the divisor from `rr` until we get overflow, at which point we add to undo the overflow. The quotient will appear in `cr`: we set `rl`, so each subtraction *increments* `cr`, giving us a count of `mturn` s executed. The remainder appears in `rr`.

In this case, we get down to `1` before the next `mturn` underflows; the result of that underflow is to `99...97`, the ten’s complement of -3. We then undo the last operation by `pturn` ing, getting `5` in `cr`: this is our quotient. `1` in `rr` is the remainder.

The same tricks from earlier work here, using `ca` to make less work, effectively implementing decimal long division of `n/m`:

```curta highlight
div:
  // Set up the registers.
  mr
  add   sr,   n
  pturn
  zero  cr
  zero  sr
  add   sr,   m
  set   rl
  // Move `ca` to be such that the highest digit of
  // `sr` lines up with the highest digit of `rr`.
  add   ca,   log(m) - log(n) + 1
loop:
  // Make subtractive turns until we underflow.
  inner:
    mturn
    if    !rr.ovflow, inner
  // Undo the turn that underflowed by doing an addition.
  // Because `rl` is set, this will also conveniently subtract
  // from `cr`, to remove the extra count from the
  // underflowing turn.
  pturn
  // We're done if this is the last digit we can be subtracting.
  // Otherwise, decrement `ca` and start over.
  if    ca == 0, done
  add   ca,   -1
  goto  loop
end:
  done

```

[Curta "Assembly"](#code:11)

Let’s execute this on `3141592653 / 137`, with an instruction trace as before.

```curta highlight
mr
// 00000000000/000000000000000/00000000/0
add   sr, 3141592653
// 03141592653/000000000000000/00000000/0
pturn
// 03141592653/000003141592653/00000001/0
zero  cr
// 03141592653/000003141592653/00000000/0
zero  sr
// 00000000000/000003141592653/00000000/0
add   sr,   137
// 00000000137/000003141592653/00000000/0
add   ca,   7
// 00000000137/000003141592653/00000000/7
mturn
// 00000000137/000001771592653/10000000/7
turn
// 00000000137/000000401592653/20000000/7
turn
// 00000000137/999990031592653/30000000/7
pturn
// 00000000137/000000401592653/20000000/7
add   ca,   -1
// 00000000137/000000401592653/20000000/6
mturn
// 00000000137/000000264592653/21000000/6
mturn
// 00000000137/000000127592653/22000000/6
mturn
// 00000000137/999999990592653/23000000/6
pturn
// 00000000137/000000127592653/22000000/6
add ca,   -1
// 00000000137/000000127592653/22000000/5
// More turns...
add ca,   -1
// 00000000137/000000004292653/22900000/4
// More turns...
add ca,   -1
// 00000000137/000000000182653/22930000/3
// ...
add ca,   -1
// 00000000137/000000000045653/22931000/2
// ...
add ca,   -1
// 00000000137/000000000004553/22931300/1
// ...
add ca,   -1
// 00000000137/000000000000443/22931330/0
// ...
done
// 00000000137/000000000000032/22931333/0

```

[Curta "Assembly"](#code:12)

For a quotient this big, you’ll need to work through all eight `cr` digits, which is a ton of work. At the end, we get a quotient of `22931333` and reminder `32`.

![](/assets/images/curta/quot.jpg)

`3141592653 / 137 == 22931333, rem 32`

Unfortunately, we can’t as easily “cheat” with subtraction as we did with multiplication, because we don’t know the value that needs to appear in `cr`.

### [Square Roots](\#square-roots)

Computing square roots by approximation is one of the premiere operations on the Curta. There’s a number of approaches. Newton’s method is the classic, but requires a prior approximation, access to lookup tables, or a lot of multiplication.

A slower, but much more mechanical approach is to use *Töpler’s method*. This consists of observing that the sum of the first `n` odd numbers is the square of `n`. Thus, we can use an approach similar to that for division, only that we now subtract off consecutive odd numbers. Let’s take the square root of `92`:

```curta highlight
sqrt_of_92:
  mr
  add   sr,   92
  pturn
  zero  cr
  zero  sr
  add   sr,   1
  set   rl
loop:
  mturn
  if    rr.ovflow, end
  add   sr,
  goto  loop
end:
  pturn
  done

```

[Curta "Assembly"](#code:13)

We get `9` as our result, but that’s pretty awful precision. We can improve precision by multiplying `92` by a large, even power of ten, and then dividing the result by that power of ten’s square root (half the zeroes).

Unfortunately, this runs into the same problem as naive multiplication: we have to turn the handle *a lot*. Turning this algorithm into something that can be done exponentially faster is a bit fussier.

One approach (which I found on <curta.org>) allows us to compute the root by shifting. Several programmers appear to have independently discovered this in the 70s or 80s.

It is based on the so-called [“digit-by-digit”](https://en.wikipedia.org/wiki/Methods_of_computing_square_roots#Digit-by-digit_calculation) algorithm, dating back to at least the time of Napier. Wikipedia provides a good explanation of why this method works. However, I have not been able to write down a proof that this specific version works, since it incorporates borrowing to compute intermediate terms with successive odd numbers in a fairly subtle way. I would really appreciate a proof, if anyone knows of one!

The algorithm is thus, for a radicand `n`:

```curta highlight
sqrt:
  mr
  // Put `ca` as far as it will go, and then enter
  // the radicand as far right as it will go, so you
  // get as many digits as possible to work with.
  add   ca,   8
  add   sr,   n << (8 - log(n))
  pturn
  zero  cr
  zero  sr
  // Put a 1 under the leftmost pair of digits. This
  // assumes a number with an even number of digits.
  add   sr,   1 << (ca - 1)
  set   rl
loop:
  sqrt_loop:
      // Add an odd number (with a bunch of zeros
      // after it.)
      mturn
      if    rr.ovflow,  sqrt_end
      // Increment sr by 2 (again, with a bunch of
      // zeros after it). This gives us our next odd
      // number.
      add   sr,   2 << (ca - 1)
      goto  sqrt_loop
  sqrt_end:
    // Note that we do NOT undo the increment of `sr`
    // that caused overflow, but we do undo the last
    // mturn.
    pturn
    // If `ca` is all the way to the right, we're out of
    // space, so these are all the digits we're getting.
    // Zeroing out `rr` also means we're done.
    if    ca == 1 || rr == 0, end
    // Subtract ONE from the digit in `sr` we were
    // incrementing in the loop. This results in an even
    // number.
    add   sr,   -(1 << (ca - 1))
    // Decrement `ca` and keep cranking.
    add   ca,   -1
    add   sr,   1 << (ca - 1)
    goto loop
end:
  done

```

[Curta "Assembly"](#code:14)

Let’s compute some digits of `sqrt(2)`. Here’s the instruction trace.

```curta highlight
mr
// 00000000000/000000000000000/00000000/0
add   ca,   7
// 00000000000/000000000000000/00000000/7
add   sr,   2 << (8 - log(n))
// 00020000000/000000000000000/00000000/7
pturn
// 00020000000/200000000000000/10000000/7
zero  cr
// 00020000000/200000000000000/00000000/7
zero  sr
// 00000000000/200000000000000/00000000/7
add   sr,   1 << (ca - 2)
// 00010000000/200000000000000/00000000/7
mturn
// 00010000000/100000000000000/10000000/7
add   sr,   2 << (ca - 2)
// 00030000000/100000000000000/10000000/7
mturn
// 00030000000/800000000000000/10000000/7
pturn
// 00030000000/100000000000000/10000000/7
add   sr,   -(1 << (ca - 2))
// 00020000000/100000000000000/10000000/7
add   ca,   -1
// 00020000000/100000000000000/10000000/6
add   sr,   1 << (ca - 2)
// 00021000000/100000000000000/10000000/6
mturn
// 00021000000/079000000000000/11000000/6
add   sr,   2 << (ca - 2)
// 00023000000/079000000000000/11000000/6
mturn
// 00023000000/056000000000000/12000000/6
add   sr,   2 << (ca - 2)
// 00025000000/056000000000000/12000000/6
mturn
// 00025000000/031000000000000/13000000/6
add   sr,   2 << (ca - 2)
// 00027000000/031000000000000/13000000/6
mturn
// 00027000000/004000000000000/14000000/6
add   sr,   2 << (ca - 2)
// 00029000000/004000000000000/14000000/6
mturn
// 00029000000/975000000000000/15000000/6
pturn
// 00029000000/004000000000000/14000000/6
add   sr,   -(1 << (ca - 2))
// 00028000000/004000000000000/14000000/6
add   ca,   -1
// 00028000000/004000000000000/14000000/5
// More of the same...

```

[Curta "Assembly"](#code:15)

Over time, the digits `14121356` will appear in `cr`. This is the square root (although we do need to place the decimal point; the number of digits before it will be half of what we started with, rounded up).

![](/assets/images/curta/sqrt.jpg)

`sqrt(2) ~ 1.4121356`

## [Wrap-up](\#wrap-up)

There’s a quite a few other algorithms out there, but most of them boil down to clever use of lookup tables and combinations of the above techniques. For example, the so-called “rule of 3” is simply performing a multiplication to get a product into `rr`, and then using it as the dividend to produce a quotient of the form `a * b / c` in `cr`.

I hope that these simple numeric algorithms, presented in a style resembling assembly, helps illustrate that programming at such a low level is not *hard*, but merely requires learning a different bag of tricks.

---

1. Although this seems like an oxymoron, it is accurate! The Curta contains no electrical or electronic components, and its registers contain discrete symbols, not continuous values. It is *not* an analog computer! [↩︎](#fnref:1)

2. The Curta is a one-indexed machine, insofar as the values engraved on `ca` are not `0` to `7` but `1` to `8`. However, as we all know, zero-indexing is far more convenient. Any place where I say “set `ca` to `n`”, I mean the `n + 1` th detent.

Doing this avoids a lot of otherwise unnecessary `-1` s in the prose. [↩︎](#fnref:2)

3. The *ten’s complement* of a number `x` is analogous to the two’s complement (i.e., the value of `-x` when viewed as an unsigned integer on a binary machine). It is equal to `MAX_VALUE - x + 1`, where `MAX_VALUE` is the largest value that `x` could be. For example, this is `999_999_999_999_999_999` (fifteen nines) for `rr`. [↩︎](#fnref:3)
