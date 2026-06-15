---
title: The Central Limit Theorem Makes Random Testing Hard
url: https://blog.regehr.org/archives/660
published: "2012-06-19T16:12:41Z"
feed: regehr
guid: http://blog.regehr.org/?p=660
---

# The Central Limit Theorem Makes Random Testing Hard

I believe that the central limit theorem provides a partial explanation for why it can be very difficult to create an effective random tester for a software system.

Random testing is carpet bombing for software: the more of the system you can hit, the better it works. The central limit theorem, however, tells us that the average of a number of random choices is going to follow a normal distribution. This is important because:

1. Sophisticated random test-case generators take as input not one but many random numbers, effectively combining them in order to create a random test case.
2. A normal distribution, whose probability decays exponentially as we move away from its center, is a very poor choice for carpet bombing.

# An Example

Let’s say we’re testing a container data structure such as a bounded queue. It has a capacity bug that fires only when an element is added to a full queue. The obvious way to generate random test cases for a container is to randomly generate a sequence of API calls. Let’s assume that only the enqueue and dequeue operations are available, and that we generate each with 50% probability. How likely is it that our random tester will discover the capacity bug? The number of elements in the queue is just a 1-dimensional random walk, telling us that the probability will decrease exponentially as the capacity of the bounded queue increases.

What does this mean, concretely? If the queue is 64 elements in size and a random test case is 10,000 enqueue/dequeue operations, each test case has about a 90% chance of finding the queue bug by attempting to add an element to a full queue. This drops to 50% if the queue holds 128 elements, and to just a few percent if the queue holds 256 elements. Larger queues make it quite unlikely that this corner case will be tested.

In this trivial example, the central limit theorem is not so likely to bite us. We’ll probably notice (using a code coverage tool or similar) that the “enqueue on full” condition is not tested, and we’ll cover this behavior by hand.

# Another Example

A while ago I [posted this bit puzzle](https://blog.regehr.org/archives/651). I won’t repeat the problem description, but the quick summary is that the goal is to find an extremely efficient implementation for some fairly simple functions, each of which operates on a pair of bit vectors. The obvious way to test a function like this is to generate two random bit vectors and then inspect the output. In general “inspect the output” is tricky but it’s trivial here since my slow example code serves as an oracle.

But now let’s ask a simple question: given a pair of random 64-bit numbers, what is the Hamming distance between them? This is the distribution:

![](https://blog.regehr.org/wp-content/uploads/2012/06/dist2.png)

This is, of course, the normal distribution centered at 32. This distribution shows up here because each bit is effectively a separate random number, for purposes of the distance computation.

But why do we care? This matters because the functions of interest in my earlier blog post have a special case when the Hamming distance is 0. That is, when the input bit vectors are equal it’s not totally clear what to do unless you look at my reference implementation. In fact, if you read the comments you can see that a couple of people wrote buggy code due to missing this case. As the graph above makes clear, the probability of generating two 64-bit vectors whose Hamming distance is zero is quite low.

# More Examples

Can we find more examples where the central limit theorem makes it hard to do good random testing? Definitely. Consider fuzzing a filesystem. If the API calls in a test case are equally likely to open and close a file, we’re not likely to reach the maximum number of open files. If a sequence of API calls is equally likely to create and remove a directory, we’re not likely to test the maximum number of subdirectories. Similar arguments can be made for almost any resource limit or edge case in a filesystem.

I suspect that normal distributions in test cases limit the effectiveness of Csmith. For example, since the number of local variables in a function is determined by many calls to the PRNG, we might expect this quantity to be normally distributed. However, certain numbers of local variables (I’m simplifying of course) cause interesting code in the register allocator, such as its spilling and rematerialization logic, to be called. Is Csmith adequately stress-testing that kind of logic for all compilers that we test? Probably not.

My hypothesis is that many similar effects occur at larger scale as we apply random testing to big, complex software systems. The larger the test case, the more likely it will be that some derived property of that test case follows a normal distribution.

# Solutions

Here are some possible ways to prevent normally distributed properties from making random testing less effective.

First, we could try to execute an extremely large number of random test cases in order to saturate even parts of the space of test cases that are generated with low probability. But this is not the right solution: the rapid falling off of the normal distribution means we’ll need to increase the number of test cases exponentially.

Second, we can impose (small) arbitrary limits on system parameters. For example, if we test queues of size 4, we’re likely to reach the overflow condition. If we test bit vectors of size 8, we’re likely to test the case where the inputs are equal. This is an extremely powerful testing technique: by reducing the size of the state space, we are more likely to reach interesting edge cases. However, it has a couple of drawbacks. For one thing, it requires modifying the software under test. Second, it requires whitebox knowledge, which may be hard to come by.

Third, we can hack the random tester to reshape its distributions. For the queue example, we might generate enqueue operations 75% of the time and dequeue operations 25% of the time. For the bit functions, we can start with a single random 64-bit number and then create the second input by flipping a random number of its bits (this actually makes the graph of frequencies of occurrences of Hamming distances completely flat). This will fix the particular problems above, but it is not clear how to generalize the technique. Moreover, any bias that we insert into the test case generator to make it easier to find one problem probably makes it harder to find others. For example, as we increase the bias towards generating enqueue operations, we become less and less likely to discover the opposite bug of the one we were looking for: a bug in the “dequeue from empty” corner case.

Fourth, we can bias the probabilities that parameterize the random tester, randomly, each time we generate a test case. So, for the queue example, one test case might get 2% enqueue operations, the next might get 55%, etc. Each of these test cases will be vulnerable to the constraints of the central limit theorem, but across test cases the centers of the distributions will change. This is perhaps more satisfactory than the previous method but it ignores the fact that the probabilities governing a random tester are often very finely tuned, by hand, based on intuition and experience. In Csmith we tried randomly setting all parameters and it did not work well at all (the code is still there and can be triggered using the `--random-random` command line option) due to complex and poorly understood interactions between various parameters. [Swarm testing](http://www.cs.utah.edu/~regehr/papers/swarm12.pdf) represents a more successful attempt to do the same thing, though our efforts so far were limited to randomly toggling binary feature inclusion switches—this seems much less likely to “go wrong” than does what we tried in Csmith. My current hypothesis is that some combination of hand-tuning and randomization of parameters is the right answer, but it will take time to put this to the test and even more time to explain why it works.

*(Just to be clear, in case anyone cares: Swarm testing was Alex’s idea. The “random-random” mode for Csmith was my idea. The connection with the central limit theorem was my idea.)*

# Summary

A well-tuned random tester (such as the one I [blogged about yesterday](https://blog.regehr.org/archives/731)) can be outrageously effective. However, creating these tools is difficult and is more art than science.

As far as I know, this connection between random testing and the central limit theorem has not yet been studied—if anyone knows differently, please say. I’d also appreciate hearing any stories about instances where this kind of issue bit you, and how you worked around it. This is a deep problem and I hope to write more about it in the future, either here or in a paper.

# Update from evening of June 19

Thanks for all the feedback, folks! I agree with the general sentiment that CLT applies a lot more narrowly than I tried to make it sound. This is what you get from reading hastily written blog posts instead of peer reviewed papers, I guess!

Anyway, the tendency for important properties of random test cases to be clumped in center-heavy distributions is a real and painful effect, and the solutions that I outline are reasonable (if partial) ones. I’ll try to reformulate my arguments and present a more developed version later on.

Also, thanks to the person on HN who mentioned the law of the iterated logarithm. I slept through a lot of college and apparently missed the day they taught that one!
