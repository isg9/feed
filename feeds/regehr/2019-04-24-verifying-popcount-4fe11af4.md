---
title: Verifying Popcount
url: https://blog.regehr.org/archives/1667
published: "2019-04-24T16:16:35Z"
feed: regehr
guid: http://blog.regehr.org/?p=1667
---

# Verifying Popcount

Popcount is the function that returns the number of set bits in its argument. Showing that a popcount implementation does what it claims to do has become one of my favorite examples to use when I need to quickly show students how we can reason about programs mathematically. Something like a selection sort is probably the more standard example here, but sorting gets into some subtleties that I’d rather reserve for a second example. Popcount is so easy that this whole example can be done in a half hour.

The programming language and the particular algorithm don’t matter too much, here we’ll use what is perhaps the simplest popcount implementation in C or C++:

```
int popcount(uint64_t x) {
  int c = 0;
  for (int i = 0; i < 64; i++) {
    c += x & 1;
    x >>= 1;
  }
  return c;
}

```

This works, of course, because it scans the entire argument, incrementing the counter each time it finds a set bit. To show that it works in a more formal fashion, we can use methods more or less out of [A Discipline of Programming](https://www.amazon.com/Discipline-Programming-Edsger-W-Dijkstra/dp/013215871X), which allow us to combine obvious effects of individual statements into (hopefully) the desired high-level property, which we might as well annotate the program with right now:

```
int popcount(uint64_t x) {
  int c = 0;
  for (int i = 0; i < 64; i++) {
    c += x & 1;
    x >>= 1;
  }
  // c = POPCOUNT(orig_x)
  return c;
}

```

Here we’ve used a couple of tricks that need to be fully explained when giving this example to students who aren’t used to this kind of reasoning. First, since `x` is modified as the program executes, we’ve used `orig_x` as a shorthand for the original value of `x` when it was passed as an argument. (This syntax is a trick I learned from [ACSL](https://frama-c.com/download/acsl_1.9.pdf).) Second, here we’ve used the POPCOUNT function, which is the mathematical specification of popcount, which is a completely different thing than the C code we are trying to verify. Consistently applying the correct rules to mathematical and programmatic elements isn’t as straightforward as we would hope and in fact quite a few results from the bad old days of non-mechanized program verification [are wrong in subtle ways](https://ai.googleblog.com/2006/06/extra-extra-read-all-about-it-nearly.html).

To make our proof go, we’ll need to find the right loop invariant: a statement that is true before the loop executes and also true after every loop iteration. The trick when explaining this task to students is helping them realize that (for a simple example like this) if they understand how the algorithm works, they effectively already know the loop invariant. They just need some help in teasing it out into a usable form. One of the stumbling blocks is that every loop has many invariants, but most of them (for example `c ≥ 0`) aren’t helpful. We can avoid a lot of silly loop invariants by observing that the right invariant will be strong enough to imply the correctness criterion for the program while also being weak enough to already be true before the first loop iteration, but I’ve found that negative constraints like these are not very helpful for students. The trick, of course, is to capture the abstract fact that is being held in balance as the loop modifies pieces of program state, and to make a lecture like this work, we need to avoid the appearance of pulling this fact out of thin air. It defeats the entire purpose of this discussion if students walk away feeling like there was a magical step in the middle.

So anyway, by hook or by crook we need to get the audience to realize that the key to this algorithm is the loop invariant `c + POPCOUNT(x) == POPCOUNT(orig_x)`. At this point we’re over the hump and we should mark up the code some more:

```
int popcount(uint64_t x) {
  int c = 0;
  // c + POPCOUNT(x) == POPCOUNT(orig_x) ???
  for (int i = 0; i < 64; i++) {
    // c + POPCOUNT(x) == POPCOUNT(orig_x)
    c += x & 1;
    x >>= 1;
    // c + POPCOUNT(x) == POPCOUNT(orig_x) ???
  }
  // c + POPCOUNT(x) == POPCOUNT(orig_x) => c = POPCOUNT(orig_x) ???
  return c;
}

```

Here I’ve used question marks to indicate a proposition that is still in doubt. Now we’ve broken up the problem into manageable pieces and can visit them in turn:

- The loop invariant is true before the loop executes because `c = 0` and `x = orig_x`.

- To see that the loop invariant is preserved by the loop body, we analyze two cases: `x & 1` is either zero or one. If it is zero, then `c` is unchanged and also, clearly, shifting `x` right by one place does not change `POPCOUNT(x)`. If it is 1 then `c` is incremented by one and shifting `x` right by one place has decreased its popcount by one. So we can see that either way, the invariant is maintained.

- To prove that the loop invariant implies the overall correctness condition, we must observe that `x = 0` after being shifted right 64 times. This is the sort of situation where we better be clear about the rules for shifting signed vs. unsigned values in C or C++.

Of course we’ve skipped over a lot of smaller steps, but this argument captures the essence of why the algorithm works. When presenting this particular algorithm, we can safely skip the termination proof since `i` is not modified in the loop body.

An entertaining variant is Kernighan’s popcount:

```
int popcount(uint64_t x) {
  int c = 0;
  while (x) {
    x &= x - 1;
    c++;
  }
  return c;
}

```

I didn’t use it in this post because I feel like it’s useful to show the analysis by cases in the simpler algorithm. [Wikipedia has some faster and much harsher algorithms](https://en.wikipedia.org/wiki/Hamming_weight#Efficient_implementation), which are the ones used in practice on processors that do not have a native popcount instruction.

Anyway, I presented this material in a software engineering class yesterday, and realized that I’d used this example a couple of times before, so it seemed worth writing up here.
