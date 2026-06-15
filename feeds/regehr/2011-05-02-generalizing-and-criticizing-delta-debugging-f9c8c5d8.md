---
title: Generalizing and Criticizing Delta Debugging
url: https://blog.regehr.org/archives/527
published: "2011-05-02T21:06:05Z"
feed: regehr
guid: http://blog.regehr.org/?p=527
---

# Generalizing and Criticizing Delta Debugging

Delta debugging is a search-based technique for taking an input to a program that triggers a bug and making that input smaller. For example, you might have a sequence of GUI operations that causes Thunderbird to crash. Assuming the crash is deterministic and the input can be replayed automatically, you can iteratively remove UI actions until you end up at a local minimum: a collection of events where removing even one of them makes the crash go away. If Delta is implemented well and properly takes advantage of the structure of the input, the resulting failure-inducing input is not only small, but also most of it is relevant to the failure. Debugging a failure triggered by a small, highly-relevant input is generally pretty easy. The definitive Delta debugging reference is [*Simplifying and Isolating Failure-Inducing Input*](http://www.st.cs.uni-saarland.de/publications/details/zeller-tse-2002/) by Andreas Zeller and Ralf Hildebrandt (ZH02 from now on).

Delta is fantastically useful, particularly in the context of random testing. As part of my group’s compiler bug-finding project we implemented three new test case minimizers using Delta-like ideas, and we also use an [existing line-based Delta implementation](http://delta.tigris.org/). Each of the four reducers has different strengths and weaknesses and in fact local minima can often by escaped by running the implementations one after the other.

Significantly, none of the four test case reducers is a straightforward implementation of an algorithm from the original Delta paper — each of them generalizes it in one or more ways. After spending a lot of time working on test case reduction and thinking about this, I got the idea of writing a paper perhaps called “Generalized Delta Debugging” which weakens many of the assumptions found in the original work. The problem was that the more parts of Delta debugging I generalized, the more the result looked just like a generic greedy search. Thus, it started to look extremely doubtful whether there was any research component to generalizing Delta debugging. This piece explores the consequences of that observation.

# Delta == Greedy Search

Just to be clear, by “greedy search” I mean the class of optimization algorithms that are based on a transformation operator and a fitness function. They work by repeatedly transforming the current solution to arrive at a new solution, and replacing the current with the new if the fitness level has increased. No doubt I’m butchering the accepted terminology, but the ideas here are really simple.

The “minimizing delta debugging algorithm” from ZH02 is an instance of greedy search where the fitness of a failure-inducing input is the inverse of its size and the fitness of all non-failure-inducing inputs is zero. The transformation operator removes a contiguous chunk of the input. When the transformation gets stuck — by running out of choices of chunks to remove — it reduces the chunk size. When it gets stuck at the minimum chunk size (a single line, character, or other atom of input) the search is finished.

The “general delta debugging algorithm” from ZH02 is very similar but its goal is to minimize the difference between the current solution and a given input, instead of simply minimizing size. Since I haven’t found many uses for this algorithm in practice, and since it’s not all that different from the minimizing Delta, I won’t discuss it further. Whenever I mention the “Delta algorithm” or similar, it is the minimizing Delta to which I refer.

Which parts of the Delta algorithms from ZH02 can be usefully generalized? As it turns out, pretty much all of them. Let’s look at the different elements in turn.

# Generalizing the Transformation Operator

The Delta transformer that deletes contiguous chunks of input at ever-finer levels of granularity is reasonably generic and efficient. However, when attacking highly-structured test cases it often gets stuck at a local maximum long before the test case is fully reduced. (Sorry if I keep switching between minimum and maximum. When discussing size the goal is minimization, when discussing fitness in a generic context, I’ll stick to the convention that the goal is maximization.) [Hierarchical delta debugging](http://www.cs.ucdavis.edu/~su/publications/icse06-hdd.pdf) is a variant that improves performance by operating on sub-trees of tree-structured inputs.

Another generalization is to use a transformer that replaces a chunk of input with something else, instead of simply deleting it. For example, one of our new reducers for C code tries to replace uses of variables with constants. Another replaces function calls with their results, including side effects. These are very effective in practice.

It is also useful to delete parts of the input in a non-local way. For example, to remove an argument to a function in a C program, we must delete it from the definition, declaration, and all uses. Making this transformation work requires a painful degree of friendliness with the C code, but again it’s very useful in practice.

Finally, we sometimes use transformations that don’t even make the test case smaller. For example it may be desirable to replace a small, complex construct (like a call to a trig function in a C program) with a larger but simpler construct (a math expression approximating the trig function’s behavior, perhaps). Similarly, it may be desirable to replace an array with a collection of scalars or a struct assignment with a collection of assignments to members. The scalars or the assignments are then vulnerable to subsequent reduction.

All of these examples point towards a more general idea which is that there is a strong synergy between test case reduction and compiler optimization (which I [wrote about earlier](https://blog.regehr.org/archives/356)).

# Generalizing the Fitness Function

ZH02’s minimizing Delta uses 1/size as its fitness function and its general Delta uses the inverse of the string distance between current solution and goal. There are plenty of other useful fitness functions. As I mentioned in the previous paragraph, considering the complexity of different program constructs is useful. We’ve also experimented with using Delta techniques to minimize the number of instructions executed by the test case. The insight is that the complexity of a failing execution depends not only on syntactic characteristics of the failure-inducing input, but also on the dynamic behavior induced by the test case.

A major gap in the ZH02 paper is that it does not address the validity problem: does the transformed test case satisfy the constraints imposed on test inputs? For some uses of Delta no validity test is required because the system under test can detect invalid inputs. On the other hand, the validity problem for C programs is extremely difficult to deal with (in theory, it’s undecidable; in practice, no fun at all) and this has been a major stumbling block in our C compiler bug-finding work — but [now solved](https://blog.regehr.org/archives/523) (thank goodness not by us). Sometimes it is desirable to test software with invalid inputs, but for the C compiler work we want to say that all invalid test cases have zero fitness.

# Generalizing the Search Framework

The third element of the Delta debugging algorithms from ZH02 that can be usefully generalized is the search algorithm itself. Backtracking can be used to avoid getting stuck too early, as can other techniques such as simulated annealing or genetic algorithms. Basically, test case reduction can be based on any search algorithm, greedy or not.

# A Verdict on Delta

Now I’d like to ask a couple of specific questions about the ZH02 paper. First, why weren’t the algorithms presented as “plugins” to the greedy search framework? This would have improved the presentation by making it clear which elements of the algorithms are important vs. boilerplate. Also, it would have made it easier for readers to understand how subsequent improvements to delta should be structured.

Second, given that delta debugging is a fairly straightforward application of greedy search, how significant is its research contribution? The answer that I lean towards is that the main contribution of delta is giving a nice name to an excellent idea that was, in 2002, somewhat obvious.

Since it is dangerous to criticize an idea by saying, in retrospect, that it was obvious, I’ll provide a bit of justification. First, two of the previously published papers cited by Zeller and Hildebrand (references 8 and 9 in the paper) apply ideas that are basically the same as Delta debugging, but without calling it that. Additionally, a paper they missed — [Differential Testing for Software](http://netilium.org/~mad/dtj/DTJ/DTJT08/DTJT08HM.HTM), published in 1998 — described a search-based, automated test case reducer. So it’s clear that by 2002 the ideas forming the core of Delta debugging were floating around.

My opinion is that the two Delta algorithms from the ZH02 paper have little enduring value because they simply do not work very well without modification. At least, we couldn’t make them work without significant generalization. The enduring value of the paper is to popularize and assign a name to the idea of using a search algorithm to improve the quality of failure-inducing test cases.

# Moving Forward

As the complexity of computer systems continues to increase, the advantages derived from deterministic execution and automated test case reduction will also continue to increase. Delta debugging provides a conceptual framework for doing so. Unfortunately, it seems that few useful papers on reducing the size of failure-inducing programs that have appeared since the original Delta work. A notable exception is the hierarchical delta debugging work I mentioned earlier. [Iterative Delta Debugging](http://staff.aist.go.jp/c.artho/papers/idd-full.pdf) is interesting but solves a slightly different problem. [Deriving Input Syntactic Structure](http://www.google.com/url?sa=t&source=web&cd=3&ved=0CDkQFjAC&url=http%3A%2F%2Fciteseerx.ist.psu.edu%2Fviewdoc%2Fdownload%3Fdoi%3D10.1.1.157.2822%26rep%3Drep1%26type%3Dpdf&ei=Nui-TaxLj9OBB7nlidcG&usg=AFQjCNGRAvu3sNRm98QGeZJEag9Mm2ULwA) is a technique that can make hierarchical delta easier to implement. If anyone knows of more work along these lines, I’d like to hear about it.
