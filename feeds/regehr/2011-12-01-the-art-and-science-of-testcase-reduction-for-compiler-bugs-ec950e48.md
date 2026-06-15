---
title: The Art and Science of Testcase Reduction for Compiler Bugs
url: https://blog.regehr.org/archives/639
published: "2011-12-01T06:04:07Z"
feed: regehr
guid: http://blog.regehr.org/?p=639
---

# The Art and Science of Testcase Reduction for Compiler Bugs

![](https://blog.regehr.org/wp-content/uploads/2011/11/bug.jpg) Test case reduction is a common problem faced by programmers where some large input (or more generally, some complicated set of circumstances) causes a program to fail, but we wish to know the smallest input (or the simplest circumstances) that causes the same failure. Reduced test cases are important because:

- Since the bulk of the reduced test case is usually relevant to the problem at hand, a good guess about the underlying problem can often by made simply by inspecting the test case.
- Usually, the system under test doesn’t spend very long executing the reduced test case, making debugging much easier.
- A reduced test case makes a good regression test.
- Even if the original failure-inducing test case is unreportable because it is proprietary, the reduced test case may be simple enough that it is reportable.

Test case reduction is particularly difficult when parts of the input are implicit — time, context switches, cache misses, etc. Reducing inputs for compiler bugs — the topic of this post — is both easier and harder than other common test case reduction problems. They are easier because compiler behavior (usually) depends only on the contents of a collection of source code files comprising its input (this may not be the case when the compiler executes non-deterministically or when it reads hidden state like precompiled header files). Test case reduction for compilers is harder because the inputs are highly structured. Thus, it may be impossible to get rid of some piece of the input (e.g. a struct field) because it is referenced by other parts of the input.

The GCC developers have written about testcase reduction [here](http://gcc.gnu.org/bugs/minimize.html) and [here](http://gcc.gnu.org/wiki/A_guide_to_testcase_reduction). LLVM comes with its own test case reducer called [Bugpoint](http://llvm.org/docs/Bugpoint.html), and reduction is also discussed [here](http://llvm.org/docs/HowToSubmitABug.html). The most commonly used automated test case reduction tool for compiler inputs is [delta](http://delta.tigris.org/), an implementation of the [ddmin](http://www.st.cs.uni-saarland.de/papers/tse2002/tse2002.pdf) algorithm. However, delta improves upon the original by exploiting the natural hierarchy found in block-structured programming languages.

A basic strategy for reducing a test case has been known for a long time. It is:

1. Remove part of the input.
2. See if it still causes the failure.
3. If yes, go to step 1. If no, backtrack to the last version that caused the failure and then go to step 1.

This is boring and easy. The tricky thing is that when a fixpoint is reached (no part of the input can be removed without breaking the test case), the resulting test case is probably not the smallest one that triggers the bug. There are several reasons for this:

- The definition of “part of the input” may not capture necessary transformations. For example, the delta tool does not understand how to remove an argument from a function and also from all calls to the function (no language-independent tool could be expected to do this).
- The search is greedy; it may have missed a branch much earlier in the search tree that lead to a smaller input.
- The actual smallest input triggering the bug may bear no resemblance to the testcase that we have.

Escaping local minima requires at least some domain specificity, and at most a deep understanding of the system under test. Let’s look at an example. A few weeks ago I reported a bug where this program causes GCC to crash:

```
struct S0
{
  int f1:1;
};
int g_155;
int g_229;
int g_360;
struct S0
func_93 (int p_94, int p_95, int p_96)
{
  struct S0 l_272 = {
  };
  for (; g_155;)
    if (0)
      {
      }
    else
      l_272.f1 = 1;
  for (;;)
    {
      unsigned l_663 = 94967295;
      if (g_229++, g_360 = p_94 || (l_272.f1 &= l_663))
        {
        }
    }
}

```

As reduced testcases go, this is perfectly fine. However, Jakub Jelinek, a GCC developer, posted a better version:

```
struct S { int s : 1; };
int a;

void
foo (int x, int y)
{
  struct S s;
  s.s = !!y;
  while (1)
    {
      unsigned l = 94967295;
      a = x || (s.s &= l);
    }
}

```

While this is clearly derived from the testcase that I reported, it was not produced simply by deleting stuff. Jakub made changes such as:

- Simplifying identifiers
- Changing a function to return void and to eliminate an argument
- Factoring code out of a conditional
- An interesting refactoring of some loop code into `s.s = !!y;`

Why does this matter?  Because it would be awesome if we could automate these transformations and save Jakub some time. The first three are clearly possible, but I’m not sure there’s any general version of the fourth transformation.

It turns out that Jakub is kind of an unsung testcase reduction hero. Consider [this awesome example](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=50911), where his reduced test case is not even in the same programming language as the original. Other good ones are [PR 48305](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=48305), [PR 47139](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=47139), [PR 47141](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=47141), and [PR 45059](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=45059). All of these make my reducers look kind of bad. Many more excellent reduced GCC testcases have been created by people like Richard Guenther, Volker Reichelt, and Andrew Pinski.

What other tricks do people use to create the smallest possible test cases?

- Inlining a function body
- Declaring multiple variables in a single line
- Removing a level of indirection from a pointer or a dimension from an array
- Replacing a struct or union with one or more scalars

A final reduction technique is to deeply understand what is going wrong in the compiler and trigger that behavior in some different way. For example, see the introduction of \_\_UINTPTR\_TYPE\_\_ in [PR 47141](http://gcc.gnu.org/bugzilla/show_bug.cgi?id=47141).

In summary, testcase reduction is both an art and a science. The science part is about 98% of the problem and we should be able to automate all of it. Creating a test case from scratch that triggers a given compiler bug is, for now at least, not only an art, but an art that has only a handful of practitioners.
