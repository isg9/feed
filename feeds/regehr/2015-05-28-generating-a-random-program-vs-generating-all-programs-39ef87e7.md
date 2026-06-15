---
title: Generating a Random Program vs. Generating All Programs
url: https://blog.regehr.org/archives/1246
published: "2015-05-28T18:32:57Z"
feed: regehr
guid: http://blog.regehr.org/?p=1246
---

# Generating a Random Program vs. Generating All Programs

Generating all possible inputs — up to some maximum length — to a software system is one way of creating test cases, and this technique even has a name: bounded exhaustive testing. Back when we were doing Csmith, my then-student Yang Chen spent a while on a bounded exhaustive C program generator which was in fact pretty decent at finding compiler bugs and had the added advantage of not requiring a test case reducer: if you try test cases in order of increasing size, you’re guaranteed that the first input that triggers a bug is already minimal. In the end, however, Csmith worked better and we dropped the exhaustive generator.

Lately I’ve been working on a little [exhaustive LLVM function generator](https://github.com/regehr/opt-fuzz). (NOTE: be careful about running this program, it not only writes a lot of files but is also a barely-controlled forkbomb. It would be better to just download a [tarball of its output](http://www.cs.utah.edu/~regehr/funcs-2-3.tar.bz2).) Here I don’t want to write about what we’ve learned using it, but rather about writing such a tool.

The first observation is obvious: the difference between a random generator and an exhaustive generator is simply the implementation of the choice operator: the random tool makes a random choice and the exhaustive generator makes every choice. Here [you can see my choice implementation](https://github.com/regehr/opt-fuzz/blob/blogpost/opt-fuzz.cpp#L97) which uses fork() since it is simple and fairly efficient.

The rest of this piece is about the less obvious differences between writing a random generator and writing an exhaustive generator. First, most random program generators contain code like this:

```
restart:
  ch1 = choose(I);
  ch2 = choose(J);
  if (!compatible(ch1, ch2))
    goto restart;

```

In other words, when the generator gets into a bind, it simply restarts an entire operation. As long as this will eventually succeed, there’s no problem, and in practice a generator that can use this idiom will be moderately simpler than one that doesn’t use it. For exhaustive generation there are two problems. First, wasting choices increases the generator’s branching factor, potentially creating a huge increase in the number of states that must be explored. Second, the exhaustive generator might not even terminate, depending on what kind of “fuel” it uses. There’s no problem if the fuel is the number of choices, but if we want to use a different, user-meaningful kind of fuel (LLVM instructions, for my generator above) then this idiom leads to an infinite fork loop.

To write the same sequence of choices without a restart loop, we end up with this kind of code:

```
  ch1 = choose(I);
  ch2 = choose(J - incompatible_choices(ch1));
  ch2 = expand(ch1, ch2);

```

In other words, we need to always choose from a densely populated range of choices and then expand the choice out to the original, sparser choice space. Why does this sort of thing happen in practice? Let’s say that a function in the program we’re generating takes two arguments that should be distinct. It’s easy to generate two random arguments and then restart if they happen to be the same, but slightly harder to generate a random argument and then a second random argument that is guaranteed to be different from the first. The painfulness increases rapidly with the sophistication of the constraints that need to be respected. I’m not sure that it would have been possible to implement Csmith in the non-restart style.

The next issue is the balancing of probabilities that is required to create a good random program generator. This is really hard, we spent an enormous amount of time tuning Csmith so that it often generates something useful. Doing a bad job at this means that a fuzzer — even if it is in principle capable of generating interesting outputs — almost never does that. For example, the main [value generation function](https://github.com/regehr/opt-fuzz/blob/blogpost/opt-fuzz.cpp#L146) in my LLVM generator is structured as a sequence of coin tosses. This is fine for exhaustive generation but creates an extremely skewed distribution of programs when used in fuzzer mode.

The final difference between random and exhaustive generation is that exhaustive generators end up being highly sensitive to incidental branching factors. For example, if my LLVM function generator is permitted to simply return one of its arguments (with the actual contents of the function being dead) the number of programs that it generates (for a given number of instructions) increases significantly, with no added testing benefit. Similarly, if we generate instructions that fold (for example, “add 1, 1”) we get another massive increase in the number of outputs (though these are not useless since they test the folder — something that we might or might not be interested in doing). A massive blowup comes from the necessity of choosing every possible value for each literal constant. This is a bit of a showstopper when generating C programs but is not so bad in LLVM since we can limit inputs to be, for example, three bits wide.

In summary, although in principle a fuzzer and an exhaustive generator are the same program with different implementations of the choice operator, in practice the concerns you face while writing one are significantly different from the other. Neither is strictly easier than the other to create.
