---
title: Use of Assertions
url: https://blog.regehr.org/archives/1091
published: "2014-02-03T00:16:18Z"
feed: regehr
guid: http://blog.regehr.org/?p=1091
---

# Use of Assertions

Assertions are great because they:

- support better testing,
- make debugging easier by reducing the distance between the execution of a bug and the manifestation of its effects,
- serve as executable comments about preconditions and postconditions,
- can act as a gateway drug to formal methods.

Assertions are less than great because they:

- slow down our code,
- make our programs incorrect — when used improperly,
- might trick some of us lazy programmers into using them to implement error handling,
- are commonly misunderstood.

This post will go through the care and feeding of assertions with the goal of helping developers maximize the benefits and minimize the costs.

# Definitions

The best definition of assertion that I’ve seen is from the [C2 wiki](http://c2.com/cgi/wiki?WhatAreAssertions):

> An assertion is a Boolean expression at a specific point in a program which will be true unless there is a bug in the program.

This definition immediately tells us that assertions are not to be used for error handling. In contrast with assertions, errors are things that go wrong that do not correspond to bugs in the program. For example, we might ask if this assertion is a good one:

```
int result = open (filename, O_RDWR);
assert (result != -1);

```

The definition tells us that the expression **`result != -1`** will be true unless there is a bug in the program. Clearly this is not (generally) the case: the call to open() will fail depending on the contents of the filesystem, which are not wholly under the control of our program. Therefore, this assertion is a poor one. The ability to make a clear distinction between program bugs and externally-triggered error conditions is the most important prerequisite for effective use of assertions.

[This post on assertions](http://nedbatchelder.com/text/assert.html) by Ned Batchelder gives an alternate definition:

> ASSERT(expr)
>
> Asserts that an expression is true. The expression may or may not be evaluated.
> - If the expression is true, execution continues normally.
> - If the expression is false, what happens is undefined.

Notice the difference between the first definition — which told us what an assertion actually means in terms of program correctness — and this one, which is operational and doesn’t mention bugs at all. This is a language designer’s and compiler writer’s view of assertions. It is useful because it makes it clear that while the assertion might be executed, we must not count on its being executed. This foreshadows an issue that I’ll discuss below, which is that assertions must not change the state of the program.

“What happens is undefined” when an assertion fails is taking a pretty strong view of things. First, this says that assertion failure might result in a message being logged to a console, a fatal signal being sent to the application, an exception being thrown, or no effect at all. Second, “undefined behavior when p is false” is equivalent to “p may be assumed to be true”. Therefore, the compiler should feel free to optimize the program under the assumption that the asserted condition holds. Although this might be what we want — in fact it would be really cool if adding assertions made our code faster rather than slower — it’s not an interpretation that is universally useful. As developers, we might want to count on a certain kind of behavior when an assertion fails. For example, Linux’s BUG\_ON() is defined to trigger a kernel panic. If we weaken Linux’s behavior, for example by logging an error message and continuing to execute, we could easily end up adding exploitable vulnerabilities.

# Where Do Assertions Come From?

When we write an assertion, we are teaching a program to diagnose bugs in itself. The second most common problem that students have with assertions (the first most common being a failure to make the distinction between bugs and error conditions) is figuring out what to assert. This is not so easy at first, but (for many people, at least) it soon becomes second nature. The short answer to where assertions come from is “outside of the core program logic.” Specifically, assertions come from:

**Math.** Basic math gives us simple ways — such as casting out nines — to check for errors in a complex operation. In a mathematical computer program there are typically many opportunities to exploit similar tricks. If we’re computing the angles between the sides of a triangle, it would be reasonable to assert that the angles sum to 180 degrees (or close to 180 if we’re using floating point). If we’re implementing an involved method of computing square roots, we might assert that the result squared is equal to the original argument. If we’re implementing a fast and tricky cosine function, asserting that its output is in \[-1,1\] would be a useful sanity check. In general, CS theory tells us that there are likely to be many problems that are inherently much more difficult to solve than to verify. Asserting the verification of the solution to any hard problem (whether it is in NP or not) would be an excellent thing to do.

**Preconditions.** When something has to be true for our code to execute correctly, it’s often worth asserting the condition up-front. This documents the requirement and makes it easier to diagnose failures to meet it. Examples include asserting that a number is non-negative before computing its square root, asserting that a pointer is non-null before dereferencing it, and asserting that a list does not contain duplicate elements before relying on that fact.

**Postconditions.** Often, near the end of a function, some kind of non-trivial but easy-to-check guarantee will be made, such as a search tree being balanced or a matrix being upper triangular. If we think there’s even a remote chance that our logic does not respect this guarantee, we can assert the postcondition to make sure.

**Invariants.** Data and data structures often have properties that need to hold in non-buggy executions. For example, in a doubly-linked list, things have gone wrong if there exists a node where **`n->next->prev != n`**. Similarly, binary search trees often require that the value attached to the left node is not greater than the value attached to the right node. Of course these are easy examples: programs such as compilers that do a lot of data structure manipulation can have almost arbitrarily elaborate data structure invariants.

**The spec.** Since a program is buggy by definition if it doesn’t implement the spec, specifications can be a powerful source of conditions to assert. For example, if our financial program is designed to balance the books, we might assert that no money has been created or destroyed after the day’s transactions have all been processed. In a typesetting program, we might assert that no text has been placed outside of the page.

# Assertions vs. Contracts

As shown in the previous section, assertions fill a variety of roles. Assertions about preconditions and postconditions that hold at module boundaries are often called contracts. Programming languages such as [Eiffel](http://www.eiffel.com/developers/design_by_contract.html), [D](http://dlang.org/dbc.html), [Racket](http://docs.racket-lang.org/guide/contract-boundaries.html), and [Ada](http://www.drdobbs.com/architecture-and-design/ada-2012-ada-with-contracts/240150569#) provide first-class support for contracts. In other languages, contract support is available via libraries or we can simply use assertions in a contract-like fashion.

# How About Some Examples?

Let’s look at assertions in action in two sophisticated code bases. Here’s [a super badass assertion in Mozilla](https://bugzilla.mozilla.org/show_bug.cgi?id=framedest) that was triggered by 66 different bugs. Jesse Ruderman adds that “about half of the bugs that trigger the assertion CAN lead to exploitable crashes, but without a specially crafted testcase, they will not crash at all.” Here’s [another awesome Mozilla assertion](https://bugzilla.mozilla.org/show_bug.cgi?id=compartment-mismatch) with 33 bugs that could trigger it.

The LLVM project (not including Clang, and not including the “examples” and “unittests” directories) contains about 500,000 SLOC in .cpp files and about 7,000 assertions. Does one assertion per 70 lines of code sound high or low? It seems about right to me.

I arbitrarily picked the C++ file in LLVM that is at the 90th percentile for number of assertions (the file with the median number of assertions contains just one!). This file, [which can be found here](http://llvm.org/svn/llvm-project/llvm/trunk/lib/Bitcode/Reader/BitcodeReader.cpp), contains 18 assertions and it’s not too hard to get a sense for their meaning even without seeing the surrounding code:

```
assert(BlockAddrFwdRefs.empty() && "Unresolved blockaddress fwd references");
assert(Ty == V->getType() && "Type mismatch in constant table!");
assert((Ty == 0 || Ty == V->getType()) && "Type mismatch in value table!");
assert(It != ResolveConstants.end() && It->first == *I);
assert(isa<ConstantExpr>(UserC) && "Must be a ConstantExpr.");
assert(V->getType()->isMetadataTy() && "Type mismatch in value table!");
assert((!Alignment || isPowerOf2_32(Alignment)) && "Alignment must be a power of two.");
assert((Record[i] == 3 || Record[i] == 4) && "Invalid attribute group entry");
assert(Record[i] == 0 && "Kind string not null terminated");
assert(Record[i] == 0 && "Value string not null terminated");
assert(ResultTy && "Didn't read a type?");
assert(TypeList[NumRecords] == 0 && "Already read type?");
assert(NextBitCode == bitc::METADATA_NAMED_NODE); (void)NextBitCode;
assert((CT != LandingPadInst::Catch || !isa<ArrayType>(Val->getType())) &&
        "Catch clause has a invalid type!");
assert((CT != LandingPadInst::Filter || isa<ArrayType>(Val->getType())) &&
        "Filter clause has invalid type!");
assert(DFII != DeferredFunctionInfo.end() && "Deferred function not found!");
assert(DeferredFunctionInfo.count(F) && "No info to read function later?");
assert(M == TheModule && "Can only Materialize the Module this BitcodeReader is attached to.");

```

The string on the right side of each conjunction is a nice touch; it makes assertion failure messages a bit more self-documenting than they otherwise would have been. Jesse Ruderman points out that a variadic assert taking an optional explanation string would be less error prone, and he also points to [Mozilla’s implementation of this](http://mxr.mozilla.org/mozilla-central/source/mfbt/Assertions.h#248), which burned my eyes.

Do the assertions in LLVM ever fail? Of course they do. As of right now, [422 open bugs in the LLVM bug database](http://llvm.org/bugs/buglist.cgi?quicksearch=assertion) match the search string “assertion.”

GCC (leaving out its testsuite directory) contains 1,228,865 SLOC in its .c files, with about 9,500 assertions, or about one assertion per 130 lines of code.

Things I’ve be interested in hearing from readers:

- Your favorite assertion

- The assertion ratio of your favorite code base, if you have the data available and can share it

# Mistakes in Assertions

There’s only one really, really bad mistake you can make when writing an assertion: changing the state of the program while evaluating the Boolean condition that is being asserted. This is likely to come about in one of two ways. First, in a C/C++ program we sometimes like to accidentally write this sort of code:

```
assert (x = 7);

```

This can be avoided using Yoda conditions or — better yet — just by being careful. The second way to accidentally change the state of the program is like this:

```
assert (treeDepth() == 7);

```

but unfortunately treeDepth() changes the value in some variable or heap cell, perhaps via a longish call chain.

In case it isn’t totally clear, the problem with side-effects in assertions is that we’ll test our program for a while, decide it’s good, and do a release build with assertions turned off and of course suddenly it doesn’t work. Or, it might be the release version that works but our debug build is broken by a side-effecting assertion. Dealing with these problems is highly demoralizing since assertions are supposed to save time, not eat it up. I feel certain that there are static analyzers that warn about this kind of thing. In fact, [the original paper about the work that became Coverity’s tool](http://www.stanford.edu/~engler/mc-osdi.pdf) mentions exactly this analysis in Section 4.1, and also gives plenty of examples of this bug. This is an area where language support for controlling side effects would be useful. Such support is extremely primitive in C/C++.

I feel compelled to add that of course every assertion changes the state of the machine, for example by using clock cycles, flipping bits in the branch predictor, and by causing cache lines to be loaded or evicted. In some circumstances, these changes will feed back into the program logic. For example, a program that runs 2% slower when compiled with assertions may run afoul of network timeouts and behave totally differently than its non-assert cousin. As developers, we hope not to run into these situations too often.

Other assertion mistakes are less severe. We might accidentally write a vacuous assertion that gives us a false sense of confidence in the code. We might accidentally write an assertion that is too strict; it will spuriously fail at some point and need to be dialed back. As a rule of thumb, we don’t want to write assertions that are too obviously true:

```
if (b) {
  x = 3;
} else {
  x = 17;
}
assert (x==3 || x==17);

```

It is also not useful to gunk up a program with layers and layers of redundant assertions. This makes code slow and hard to read. The 1-in-70 number from LLVM is a reasonable target. Some codes will naturally want more assertions than that; some codes will not need nearly as many.

[Ben Titzer’s comment](https://blog.regehr.org/archives/1091#comment-13980) illustrates some ways that assertions can be misused to break modularity and make code more confusing.

One danger of assertions is that they don’t compose well when their behavior is to unconditionally terminate a process. Tripping over assertions in buggy library code is extremely frustrating. On the other hand, it’s not immediately obvious that compiling a buggy library with NDEBUG is a big improvement since now incorrect results are likely to percolate into application code. In any case, if we are writing library code, we must be even more careful to use assertions correctly than if we are writing a standalone application. The only time to assert something in library code is when the code truly cannot continue executing without dire consequences.

Finally — and I already said this — assertions are not for error handling. Even so, I often write code asserting that calls to close() return 0 and calls to malloc() return a non-null pointer. I’m not proud of this but I feel that it’s better than the popular alternatives of (1) ignoring the fact that malloc() and close() can fail and (2) writing dodgy code that pretends to handle their failure. Anyhow, as a professor I’m allowed to write academic-quality code sometimes. In a high-quality code base it might still be OK to crash upon encountering an error that we don’t care to handle, but we’d want to do it using a separate mechanism. Jesse mentions that Mozilla has a MOZ\_CRASH(message) construct that serves this purpose.

# Misconceptions About Assertions

A few years ago, in a class where the assignments were being written in Java, I noticed that the students weren’t putting very many assertions in their code, so I asked them about it. Most of them sort of looked at me blankly but one student spoke up and said that he thought that exceptions were an adequate replacement for assertions. I said something like “exceptions are for error handling and assertions are for catching bugs” but really that wasn’t the best answer. The best answer would have been something like this:

> Exceptions are a low-level control flow mechanism in the same category as method dispatch and switch statements. A common use for exceptions is to support structured error handling. However, exceptions could also be used to implement assertion failure. Both assertions and structured error handling are higher-level programming tasks that need to be mapped onto lower-level language features.

Another misconception (that I’ve never heard from a student, but have seen on the net) is that unit testing is a superior replacement for assertions. This would be true only in the case where our unit tests are perfect because they catch all bugs. In the real world neither unit tests nor assertions find all bugs, so we should use both. In fact, there is a strong synergy between assertions and unit tests, as I have tried to [illustrate in a previous post](https://blog.regehr.org/archives/896). Here’s a blog post talking about how [assertions interfere with unit testing](http://googletesting.blogspot.com/2009/02/to-assert-or-not-to-assert.html). My opinion is that if this happens, you’re probably doing something wrong, such as writing tests that are too friendly with the implementation or else writing bogus assertions that don’t actually correspond to bug conditions. This page asks if [unit tests or assertions are more important](http://programmers.stackexchange.com/questions/18288/are-asserts-or-unit-tests-more-important) and here’s [some additional discussion](http://c2.com/cgi/wiki?DoNotUseAssertions).

# The checkRep() Idiom

I believe it is a good idea, when implementing any non-trivial data structure, to create a representation checker — often called checkRep() — that walks the entire structure and asserts everything it can. Alternatively, a method called repOK() can be implemented; it does not contain any assertions but rather returns a Boolean value indicating whether the data structure is in a consistent state. We would then write, for example:

```
assert (tree.repOK());

```

The idea is that while unit-testing the data structure, we can call checkRep() or assert repOK() after every operation in order to aggressively catch bugs in the data structure’s methods. For example, here’s a checkRep() that I implemented for a red-black tree:

```
void checkRep (rb_red_blk_tree *tree)
{
  /* root is black by convention */
  assert (!tree->root->left->red);
  checkRepHelper (tree->root->left, tree);
}

/*
 * returns the number of black nodes along any path through
 * this subtree
 */
int checkRepHelper (rb_red_blk_node *node, rb_red_blk_tree *t)
{
  /* by convention sentinel nodes point to nil instead of null */
  assert (node);
  if (node == t->nil) return 0;

  /* the tree order must be respected */
  /* parents and children must point to each other */
  if (node->left != t->nil) {
    int tmp = t->Compare (node->key, node->left->key);
    assert (tmp==0 || tmp==1);
    assert (node->left->parent == node);
  }
  if (node->right != t->nil) {
    int tmp = t->Compare (node->key, node->right->key);
    assert (tmp==0 || tmp==-1);
    assert (node->right->parent == node);
  }
  if (node->left != t->nil && node->right != t->nil) {
    int tmp = t->Compare (node->left->key, node->right->key);
    assert (tmp==0 || tmp==-1);
  }

  /* both children of a red node are black */
  if (node->red) {
    assert (!node->left->red);
    assert (!node->right->red);
  }

  /* every root->leaf path has the same number of black nodes */
  int left_black_cnt = checkRepHelper (node->left, t);
  int right_black_cnt = checkRepHelper (node->right, t);
  assert (left_black_cnt == right_black_cnt);
  return left_black_cnt + (node->red ? 0 : 1);
}

```

Hopefully this code makes some sense even if you’ve never implemented a red-black tree. It checks just about every invariant I could think of. The [full code is here](https://github.com/regehr/rb_tree_demo/blob/master/red_black_tree.c).

# The UNREACHABLE() Idiom

Often, a switch or case statement is meant to be cover all possibilities and we don’t wish to fall into the default case. Here’s one way to deal with the problem:

```
switch (x) {
  case 1: foo(); break;
  case 2: bar(); break;
  case 3: baz(); break;
  default: assert (false);
}

```

Alternatively, some code bases use an UNREACHABLE() macro that is equivalent to **`assert(false)`**. LLVM, for example, uses [llvm\_unreachable()](http://llvm.org/docs/doxygen/html/ErrorHandling_8h.html#ace243f5c25697a1107cce46626b3dc94) about 2,500 times (however, their unreachable construct has a pretty strong semantics — in non-debug builds, it is turned into a compiler directive indicating dead code).

# Light vs. Heavy Assertions

In many cases, assertions naturally divide into two categories. Light assertions execute in small constant time and tend to touch only metadata. Let’s take the example of a linked list that maintains a cached list length. A light assertion would ensure that the length is zero when the list pointer is null, and the length is non-zero when the list pointer is non-null. In contrast, a heavy assertion would check that the length is actually equal to the number of elements. Heavy assertions for a sort routine would ensure that the output is sorted and maybe also that no element of the array has been modified during the sort. A checkRep() almost always includes heavy assertions.

Generally speaking, heavy assertions are most useful during testing and probably have to be disabled in production builds. Light assertions might be enabled in deployed software. It is not uncommon for large software bases such as LLVM and Microsoft Windows to support both “checked” and “release” builds where the checked build — which is customarily not used outside of the organization that develops the software — executes heavy assertions and is substantially slower than the release build. The release build — if it does include the light assertions — is generally only a tiny bit slower than it would be if the assertions were omitted entirely.

# Are Assertions Enabled in Production Code?

This is entirely situational. Let’s look at a few examples. Jesse passes along [this example](https://bugzilla.mozilla.org/show_bug.cgi?id=810718#c22) where a useful check in Mozilla was backed out due to a 3% performance hit. On the other hand [Julian Seward said](http://www.techrepublic.com/article/open-source-awards-2004-julian-seward-for-valgrind/):

> Valgrind is loaded with assertion checks and internal sanity checkers which periodically inspect critical data structures. These are permanently enabled. I don’t care if 5 percent or even 10 percent of the total run-time is spent in these checks–automated debugging is the way to go. As a result, Valgrind almost never segfaults–instead it emits some kind of a useful error message before dying. That’s something I’m rather proud of.

The Linux kernel generally wants to keep going no matter what, but even so it contains more than 11,000 uses of the BUG\_ON() macro, which is basically an assertion — on failure it prints a message and then triggers a kernel panic without flushing dirty buffers. The idea, I assume, is that we’d rather lose some recently-produced data than risk flushing corrupted data to stable storage. Compilers such as GCC and LLVM ship with assertions enabled, making the compiler more likely to die and less likely to emit incorrect object code. On the other hand, I heard from a NASA flight software engineer that some of the tricky Mars landings have been done with assertions turned off because an assertion violation would have resulted in a system reboot and by the time the reboot had completed, the spacecraft would have hit the planet. The question of whether it is better to stop or keep going when an internal bug is detected is not a straightforward one to answer. I’d be interested to hear stories from readers about situations where assertions or lack of assertions in deployed software lead to interesting results.

# Limitations of Assertions

Some classes of program bugs cannot be effectively detected using assertions. There’s no obvious way to assert the absence of race conditions or infinite loops. In C/C++, it is not possible to assert the validity of pointers, nor is it possible to assert that storage has been initialized. Since assertions live within the programming language, they cannot be used to express mathematical concepts such as quantifiers — useful, for example, because they support a concise specification of part of the postcondition for a sort routine:

∀ i, j : 0 ≤ i ≤ j < length ⇒ array\[i\] ≤ array\[j\]

The other half of the postcondition for a sort routine — which is often left out — requires that the output array is a permutation of the input array.

Some bugs can in principle be detected by assertions, but in practice are better detected other ways. One good example of this is undefined integer operations in C/C++ programs: asserting the precondition for every math operation would clutter up a program unacceptably. Compiler-inserted instrumentation is a much better solution. Assertions are best reserved for conditions that mean something to humans.

# Assertions and Formal Methods

Recall my preferred definition of assertion: an expression at a program point that, if it ever evaluates to false, indicates a bug. Using predicate logic we can flip this definition around and see that if our program contains no bugs, then no assertion can fail. Thus, at least in principle, we should be able to prove that each assertion in our program cannot fail. Doing these proofs for non-trivial code is difficult, although it is far easier than proving an entire program to be correct. Also, we have tools that are specifically designed to help us reason about assertions; my favorite one for C code is [Frama-C](http://frama-c.com/). If we write our assertions in [ACSL](http://frama-c.com/download/acsl.pdf) (Frama-C’s annotation language) instead of in C, we get some advantages. First, ACSL is considerably more expressive than C; for example, it provides quantifiers, mathematical integers, and a way to assert that a pointer refers to valid storage. Second, if we can prove that our ACSL annotations are consistent with the source code, then we have a very strong result — we know that the assertion holds on all program paths, not just the ones that we were able to execute during testing. Additionally, as a backup plan, a subset of ACSL (called [E-ACSL](http://frama-c.com/eacsl.html)) can be translated into regular old assertions.

# Conclusion

Assertions live at a sweet spot where testing, documentation, and formal methods all come together to help us increase the quality of a code base.

Many thanks to Robby Findler and Jesse Ruderman for providing feedback on drafts of this article.

**UPDATES:**

- As I hoped (and expected!) there is a lot of good information in the comments.

- See [Ben Titzer’s comment](https://blog.regehr.org/archives/1091#comment-13980) about bad use of assertions.

- Jesse Ruderman just published a [great post about the synergy between fuzzing and assertions](http://www.squarefree.com/2014/02/03/fuzzers-love-assertions/) with data and examples from Mozilla.

- Jack Ganssle passed along a pointer to [this paper from Microsoft Research](http://research.microsoft.com/pubs/70290/tr-2006-54.pdf) that provides some data about the effectiveness of assertions.

- Assertions are one of Phil Guo’s [code carabiners](http://pgbovine.net/code-carabiners.htm).

- I added a paragraph about assertions in library code to the “Mistakes in Assertions” section based on some [discussion at Reddit](http://www.reddit.com/r/programming/comments/1wupqt/embedded_in_academia_use_of_assertions/cf5sxcd). Thanks folks.
