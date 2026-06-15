---
title: Pluggable Domain-Specific Testcase Reducers
url: https://blog.regehr.org/archives/645
published: "2011-12-09T04:31:41Z"
feed: regehr
guid: http://blog.regehr.org/?p=645
---

# Pluggable Domain-Specific Testcase Reducers

An important part of effective random testing is providing developers with actionable testcases, and a big part of creating actionable testcases is testcase reduction. Delta debugging, as exemplified by the [delta tool](http://delta.tigris.org/), is a good generic reduction technique. However, in many cases, as I discussed the other day [here](https://blog.regehr.org/archives/639), a domain-specific reducer can do a much better job.

Today’s question is: How to build a domain-specific testcase reduction tool? Of course, one option is to build this kind of tool from scratch, but that’s not very fun. After building a few testcase reducers from scratch, and after watching my students build a few more, I discovered a nice API for isolating the transformation part of a reducer — the main part that is actually domain specific — from the rest of the tool. But first, let’s look at what has already been accomplished in terms of factoring a testcase reducer into reusable parts.

The original delta debugging algorithm, [ddmin](http://www.st.cs.uni-saarland.de/papers/tse2002/tse2002.pdf), mixes all of its concerns together. To reuse ddmin, you reimplement from scratch. The [delta tool](http://delta.tigris.org/) improves the situation by factoring the “interestingness test” out into an external script. This test decides whether a freshly produced delta variant should be accepted or discarded. This is really useful and I’ve written probably well over a dozen different interestingness tests. However, the transformation part of ddmin remains hard-coded in the delta tool. The only thing it knows how to do is remove one or more contiguous lines from a text file. This is nice but not nearly good enough for demanding reduction tasks.

This post is about factoring out the reduction transformations. This is necessary because you need one set of transformations to reduce C++ code (template expansion, namespace flattening, etc.) and different ones to reduce XML or Java or whatever. Furthermore, even if we consider testcase reduction for a single language, some transformations can be expressed as simple string manipulations — there’s no real need to factor these out of a reduction tool — whereas other transformations (inlining a C function or performing copy propagation) are far more involved. We want to maintain some separation between these compiler-like tools and the program reducer.

The transformation tool API is very simple; it provides a single point of entry:

> **reduce (filename, command, number)**

It works like this:

- *filename* refers to the current test case, which the transformation tool modifies in place
- *command* specifies which transformation to perform; it doesn’t matter for tools that only perform one kind of transformation
- *number* identifies which instance of the transformation to perform
- the return value of a *reduce* call is either *success*, indicating that the transformation succeeded, or *failure*, indicating that *number* did not correspond to any transformation

The only trick here is properly enumerating transformation opportunities. For any given test case, it is required that there exists an n such that *number* in 0 .. n-1 will result in successful transformations and *number* â‰¥ n will fail. In my experience, assigning numbers to transformation opportunities is straightforward. For example, consider a transformation that tries to replace a constant in a test case with the value “1”. We could number constants by their order of appearance in the token stream, we could number them by their order of occurrence in a preorder traversal of the AST, or whatever — it doesn’t matter as long as the above property holds.

It is not required that the transformation produces a valid testcase, nor is it required that the transformation always makes the test case smaller. There are not, strictly speaking, any requirements at all on the transformation; it could randomly change the test case or even generate a whole new test case from scratch. On the other hand, in practice, it’s nice if the transformation:

- is deterministic,
- for any given input, eventually runs out of transformation opportunities when repeatedly applied, and
- produces a valid, improved test case reasonably often.

The first property makes testcase reduction repeatable. The second property means that the reducer will eventually terminate when it reaches a fixpoint, as opposed to requiring a timeout or other termination criterion. To return to the example of a transformation that replaces constants with one, we would need to put a special case in the transformer code that prevents it from replacing one with one in order to satisfy this property. In practice it is often the case that a successful transformation decreases the number of possible transformations by one — but it’s easy to find examples where this does not hold (and it doesn’t need to). The third property means that the reducer is reasonably effective and fast.

Other parts of the testcase reducer — its search strategy, interestingness test, and fitness function — do not matter here. Let’s look at an example of plugging the “replace constant with one” transformer into a generic reducer. Let’s pretend we have a compiler that crashes with a math exception when it tries to constant-fold the division by zero. We’ll begin with this unreduced test case:

```
int foo (void) {
  int x = 33;
  int y = x / 0;
  return y + 66;
}

```

We invoke the reducer and it first issues the call “reduce (foo.c, REPLACE\_WITH\_ONE, 0)”. The call succeeds and gives:

```
int foo (void) {
  int x = 1;
  int y = x / 0;
  return y + 66;
}

```

This test case is still interesting (it still triggers the compiler crash) and it is smaller than the previous one, so the reducer accepts it as the current test case. The reducer heuristically assumes that the successful transformation eliminated an opportunity; therefore it again issues “reduce (foo.c, REPLACE\_WITH\_ONE, 0)”. Again the call succeeds and gives:

```
int foo (void) {
  int x = 1;
  int y = x / 1;
  return y + 66;
}

```

This test case is not interesting — it will not trigger a divide-by-zero bug in the compiler. This variant is discarded and the reducer now issues “reduce (foo.c, REPLACE\_WITH\_ONE, 1)”. Recall that constants with value 1 are skipped over when the transformer enumerates its possibilities. The transformation succeeds and the result is:

```
int foo (void) {
  int x = 1;
  int y = x / 0;
  return y + 1;
}

```

This is again interesting. However, the reducer’s next call “reduce (foo.c, REPLACE\_WITH\_ONE, 1)” fails because transformation opportunity 1 does not now exist. To be sure that it has reached a fixpoint, the reducer will need to again test “reduce (foo.c, REPLACE\_WITH\_ONE, 0)” which again succeeds but leaves an uninteresting testcase. At this point the REPLACE\_WITH\_ONE transformation cannot make progress. If the reducer has no other transformations to try, it terminates.

It would be fairly straightforward to reimplement the delta tool in this framework; I haven’t bothered yet because delta works perfectly well. Something interesting about the delta tool is that it doesn’t really reach a fixpoint: if you run it twice using the same parameters, the second run sometimes improves the testcase. Basically the authors decided to go for quicker termination at the expense of testcase quality. Another quirk is that it iterates backwards through its list of transformations whereas in the example above, we iterated forwards. Presumably the delta authors observed that backwards iteration is faster; I haven’t done the experiments to confirm this. To support backwards iteration, we’d have to add another call to the API:

> **get\_count (filename, command)**

This call returns the total number of possible instances of a transformation there are in a test case. To go backwards we’d start by issuing a reduce call at one less than this number and work down to zero.

A minor variant of the *reduce* interface that is used by my [c\_delta tool](https://github.com/csmith-project/csmith/blob/master/utah/scripts/reduce/c_delta.pl) replaces the *number* argument with a byte count. The tool then doesn’t even bother searching for transformation opportunities in bytes preceding the specified starting point. Revising c\_delta’s regex-based transformations to use numbers instead of byte counts would not be hard but I don’t have plans to do this soon.

The idea of decomposing testcase reduction into a collection of modular parts is one that I’ve [explored before](https://blog.regehr.org/archives/527). This post takes a detailed look at one aspect of the overall design of testcase reducers: the reduction transformation. The API is simple and (I think) generically useful. I’m embarrassed to say that it took me a few years of dinking around with various testcase reducers before I discovered it. My guess is that it’s not sufficiently complicated or researchy to be publishable, so the writeup here will have to suffice.

Something that I hope comes out of this line of work is that after factoring reducers into reusable parts, we’ll be able to do a better job improving those parts. For example, comments on my [previous post about reduction for C code](https://blog.regehr.org/archives/639) indicated some transformations that I hadn’t thought of. Once we implement these, for example as Clang or [Rose](http://rosecompiler.org/) modules, then they can be trivially plugged into any testcase reducer supporting this API. Similarly, now that the overall reducer doesn’t need to understand interestingness tests or transformations, we’re free to start evaluating better search strategies, figuring out how to parallelize the reducer, etc.

**Help Wanted:** If you have a transformation tool for C or C++ that can be modified to fit the interface described in this post, and if it implements some transformation that is useful (or if you are willing to implement one), please leave a comment or drop me a line. I’m currently working on a “best of breed” reduction tool for C/C++ programs and am willing to try out any plausible transformation. I report a lot of compiler bugs and expect this tool to be super useful in practice.
