---
title: Helping Generative Fuzzers Avoid Looking Only Where the Light is Good, Part 1
url: https://blog.regehr.org/archives/1700
published: "2019-11-04T15:50:37Z"
feed: regehr
guid: http://blog.regehr.org/?p=1700
---

# Helping Generative Fuzzers Avoid Looking Only Where the Light is Good, Part 1

Let’s take a second to recall this [old joke](https://en.wikipedia.org/wiki/Streetlight_effect):

> A policeman sees a drunk man searching for something under a streetlight and asks what the drunk has lost. He says he lost his keys and they both look under the streetlight together. After a few minutes the policeman asks if he is sure he lost them here, and the drunk replies, no, and that he lost them in the park. The policeman asks why he is searching here, and the drunk replies, “this is where the light is.”

Using a generative fuzzer — which creates test cases from scratch, rather than mutating a collection of seed inputs — feels to me a lot like being the drunk guy in the joke: we’re looking for bugs that can be triggered by inputs that the generator is likely to generate, because we don’t have an obviously better option, besides doing some hard work in improving the generator. This problem has bothered me for a long time.

One way to improve the situation is [swarm testing](https://www.cs.utah.edu/~regehr/papers/swarm12.pdf), which exploits coarse-grained configuration options in the fuzzer to reach diverse parts of the search space with higher probability. Swarm is a good idea in the sense that it costs almost nothing and seems to give good results, but it’s not remotely the end of the story. Today we’ll work on a different angle.

Let’s start out with a high-level view of the situation: we have a system under test, and its inputs come from a vast, high-dimensional space. If we confine ourselves to generating inputs up to N bits long, it is often the case that only a very small fraction of the 2N possibilities is useful. (Consider testing Microsoft Word by generating uniform random bit strings of 100 KB. Very nearly 0% of these would be enough like a Word document to serve as an effective test case.) This is the reason generative fuzzers exist: they avoid creating invalid inputs. (Of course mutation-based fuzzers work on a related principle, but this piece isn’t about them.)

Now we can ask: out of the space of inputs to the system under test, which parts of the space will trigger bugs? I would argue that unless we are doing some sort of focused testing, for example to evaluate the hypothesis “formulas containing trig functions often make Excel fall over,” we should assume that bugs are distributed uniformly. (And even if we are restricting our attention to formulas containing a trig function, we probably want to assume that this subset of Excel formulas contains uniformly distributed bugs.) It follows that a good generative fuzzer should explore the space of inputs with uniform probability, or at least it should be capable of doing this. Another way to state the desired property is: if the fuzzer is capable of producing X different test cases, each of them should be generated with probability 1/X. I don’t know that this property is optimal in any meaningful sense, but I do believe it to be a good and reasonable design goal for a generative fuzzer. Alas, a non-trivial generative fuzzer is very unlikely to have this property by accident.

Why is a generative fuzzer not going to naturally emit each of its X outputs with probability 1/X? This follows from how these tools are implemented: as decision trees. A generative fuzzer makes a series of decisions, each determined by consulting a PRNG, on the way to emitting an actual test case. In other words, this kind of tool starts at the root of a decision tree and walks to a randomly chosen leaf. Choices made near the root of the decision tree are effectively given more weight than choices made near the leaves, making it possible for some leaves to have arbitrarily low probabilities of being emitted by a decision tree. Let’s consider this very simple decision tree:

![](../extra_files/tree.png)

If we use it to generate output a bunch of times, we’ll end up getting outputs with percentages something like this:

```
A 49.979
B 25.0018
C 12.509
D 6.26272
E 3.1265
F 3.12106

```

If you are thinking to yourself “this example is highly contrived” then please consider how structurally similar it is to [the core of a simple LLVM IR generator](https://github.com/regehr/opt-fuzz/blob/master/opt-fuzz.cpp#L314-L706) that I wrote, which has been used to find a number of optimizer bugs. (At the moment it only works as an enumerator, I ripped out its randomized mode because of the problems described in this post.)

So what can we do about this? How can we hack our decision tree to generate its leaves uniformly? A few solutions suggest themselves:

- Enumerate all leaves and then perform uniform sampling of the resulting list.

- Skew the decision tree’s probabilities so that they are biased towards larger subtrees.

- Structure the generator so that its decision tree is flattened, to stop this problem from happening in the first place.

Unfortunately, the first solution fails when the tree is too large to enumerate, the second fails when we don’t know subtree sizes, and the third presents software engineering difficulties. In Part 2 we’ll look at more sophisticated solutions. Meanwhile, if you’d like to play with a trivial generative fuzzer + leaf enumerator based on this decision tree, [you can find a bit of code here](https://gist.github.com/regehr/24560ea4352e11bf6c74a0bc70b8f0e1).

Finally I want to briefly explain why generative fuzzers are structured as decision trees: it’s because they are effectively inverted parsers. In other words, while a parser takes lexical elements and builds them into a finished parse tree, a generative fuzzer — using the same grammar — starts at the root node and recursively walks down to the leaves, resulting in test cases that always parse successfully.
