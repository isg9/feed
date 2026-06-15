---
title: Probabilities in Random Testing
url: https://blog.regehr.org/archives/380
published: "2011-02-10T17:10:57Z"
feed: regehr
guid: http://blog.regehr.org/?p=380
---

# Probabilities in Random Testing

A typical real computer system has an extremely large input space and no testing method can cover more than an infinitesimal part of it. On the other hand, broad regions of the input space will trigger bugs. The problem is that we do not know the shapes or locations of the buggy parts of the space. Random testing is a shotgun approach to finding these regions; empirically, it works well.

I’ve written, or supervised the writing of, about 10 random test case generators. The most difficult and unsatisfactory part of engineering a good random tester is setting the probabilities properly. For example, if I’m generating JavaScript programs to stress-test a browser, I need to decide how often to create a new variable vs. referencing an existing one, how often to generate a loop vs. a conditional, etc. The shape of the eventual JS program depends strongly on dozens of little decisions like these.

A few more observations:

- If the probabilities are chosen poorly, the tester will be far less effective than it could have been, and may not find any bugs at all
- Probabilities alone are an insufficient mechanism for engineering good test cases, and need to be combined with other mechanisms such as heuristic limits on the size of various parts of a test case
- Equivalent to the previous bullet, probabilities sometimes want to be a function of the test case being constructed; for example, the probability of creating a new function may decrease as the number of generated functions increases
- The connection between probabilities and high-level properties of test cases may be very indirect
- Alternatives to playing probability games, such as uniform sampling of the input space or bounded exhaustive testing, are generally harder or less effective than just getting the probabilities right

In practice, probability engineering looks like this:

1. Think hard about what kind of test cases will drive the system under test into parts of its state space where its logic is weak
2. Tweak the probabilities to make the generated tests look more like they need to look
3. Look at a lot of generated test cases to evaluate the success of the tweaking
4. Go back to step 1

Random testing almost always has a big payoff, but only when combined with significant domain knowledge and a lot of hard work.
