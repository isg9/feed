---
title: Design and Evolution of C-Reduce (Part 1)
url: https://blog.regehr.org/archives/1678
published: "2019-07-09T22:01:23Z"
feed: regehr
guid: http://blog.regehr.org/?p=1678
---

# Design and Evolution of C-Reduce (Part 1)

\[This piece is posted in parallel on the IEEE Software blog. Karim Ali copyedited.\]

Since 2008, my colleagues and I have developed and maintained C-Reduce, a tool for programmatically reducing the size of C and C++ files that trigger compiler bugs. C-Reduce also usually does a credible job reducing test cases in languages other than C and C++; we’ll return to that later.

## Why Reduce Test Cases?

Here’s a typical C-Reduce output, [as found in the LLVM bug database](https://bugs.llvm.org/show_bug.cgi?id=39199):

```
int a[1];
int b;
void c() {
  void *d = b = 0;
  for (;; b++)
    a[b] = (int) d++;
}

```

Compiling this code at the -O2 optimization level causes LLVM to crash. The bug report doesn’t contain the original, unreduced test case, but most likely it was larger.

A reduced test case is preferable because:

- it usually gets the compiler to misbehave quickly, reducing the number of function calls, memory allocations, etc. that the compiler developer has to step through and reason about while debugging

- the reduced file contains little code not directly related to triggering the bug, reducing the likelihood that compiler developers will be distracted by extraneous features of the test case

- reduced test cases for the same bug often look similar to each other, whereas this is not normally true for unreduced files that trigger the same bug

- there is often little or no discernible similarity between an unreduced source file and its reduced version, making it easier for compiler bugs triggered by proprietary code to be reported externally

The minimum-sized input triggering any particular compiler bug can be found using a trivial algorithm: exhaustive search of text strings of increasing size. This is, of course, almost always intractable. In practice, test case reduction proceeds in the other direction: starting with a large, failure-inducing test case, incrementally making it smaller until a local minimum is reached.

## A Bit of Background

The history of automated test case reduction does not seem to be well documented, but several examples can be found in software testing papers from the 1990s, such as [Differential Testing for Software](https://www.cs.dartmouth.edu/~mckeeman/references/DifferentialTestingForSoftware.pdf) and [Massive Stochastic Testing of SQL](https://pdfs.semanticscholar.org/98bc/afcf68dcaac784c7d4271656783bb676cb51.pdf). Test-case reduction was first studied in its own right in 2000 when [Hildebrandt and Zeller introduced Delta Debugging](https://www.st.cs.uni-saarland.de/publications/files/hildebrandt-issta-2000.pdf): a general-purpose technique for test case reduction. Their algorithm uses a greedy search where a series of “variants” (our term, not theirs, for partially-reduced test case candidates) is produced by removing chunks of the test case. As reduction progresses, the chunk size is reduced, until it reaches some minimum-sized unit, such as a single line, token, or character. When no minimum-sized chunk can be removed from the test case without breaking the property that it triggers the bug, the Delta Debugger terminates. Almost all subsequent test-case reduction work, including ours, builds upon this work.

## Towards C-Reduce

I became interested in test case reduction when my colleagues and I began to find a lot of bugs in C compilers using random testing. We found so many bugs that reporting them became bottlenecked on reducing the bug-triggering programs. Since I was the one reporting the bugs we found, I was the one who felt the pain of manual test-case reduction, and it quickly got old. I eventually reported around 500 compiler bugs and I could not have done this without first creating C-Reduce.

At the time, the [best open-source implementation of Delta Debugging, from UC Berkeley](http://delta.tigris.org/), was line-based and contained a significant innovation over the original algorithm: it could reorganize a file in such a way that all nested curly braces deeper than a configurable level would appear on a single line. Thus, at level zero, entire functions would be placed on the same line, enabling the line-based reducer to remove an entire function at once. At higher nesting levels, functions would be split across lines, enabling finer-grained reduction. This worked well but the tool ended up being inadequate for my needs: it got stuck at local minima that were often orders of magnitude larger than what could be achieved when reducing by hand.

The limiting factor in this existing tool (“Delta” from now on) was obvious: it was not able to exploit enough of the structure of the file being reduced. For example, it could usually not do much to simplify arithmetic expressions. These sorts of simplifications tend to have a cascading effect: eliminating the last use of a variable allows its definition to be eliminated, etc. The obvious path forward was to write a new tool that solved a reduction problem that Delta could not solve, and then to alternate running this tool and Delta until a global fixpoint was reached. I did this, adding more and more reduction techniques over time. At some point I implemented a line-elimination pass in my new reducer, at which point Delta was subsumed and could be dropped.

We ended up keeping two elements of Delta’s design. First, the configurable hierarchical reformatting of a test case based on curly brace nesting. This technique, followed by removing contiguous chunks of code, is still one of C-Reduce’s most useful first lines of attack on a test case. Second, Delta’s mechanism for determining whether a given variant is “interesting.” An interesting variant is used as the basis for further reduction steps; an uninteresting variant is a dead end, and is discarded. Delta determined interestingness by invoking a user-supplied program — typically a shell script — whose process exit code determines the interestingness of the current variant. The flexibility afforded by this small element of user extensibility ends up being extremely useful. For example, the interestingness test can discard test cases that trigger certain compiler warnings, it can attempt to disambiguate different crash bugs, etc.

It is more challenging to reduce test cases that cause the compiler to emit incorrect object code than it is to reduce test cases that merely cause the compiler to crash. C-Reduce itself is agnostic about the character of the bug of interest: we push all of the difficulties in reducing miscompilation triggers into the interestingness test, which should try to answer questions such as:

- is the variant well-defined by the C or C++ standard?

- does the variant avoid depending on behaviors that are unspecified by the C or C++ standard?

- does the buggy compiler turn the variant into an executable?

- does this executable terminate within a specified time?

- does the reference compiler (assumed to not contain the bug of interest) turn the variant into an executable?

- does this executable also terminate within a specified time?

- does the behavior of the two executables differ in a way that indicates that a miscompilation occurred?

The variant is interesting if the answer to all of these questions is “yes.”

The hardest part of reducing programs that trigger miscompilation bugs is ensuring that variants avoid undefined behaviors (such as invalid pointer operations) and do not rely on unspecified behaviors (such as the order of evaluation of function arguments). A test case doing one of these things is ill-formed and can accomplish nothing beyond annoying compiler developers. Empirically, if undefined behavior is not actively avoided during test-case reduction, C-Reduce will almost certainly introduce it. The practical solution is to employ suitable static and dynamic analysis tools, in order to avoid these ill-formed variants. Since no single tool detecting all undefined behaviors in C and C++ programs exists, a hybrid approach involving multiple tools is typically used in practice. This is not completely satisfying, but it works well enough that C-Reduce can reliably produce useful reduced test cases for miscompilation bugs in C and C++ compilers.

Writing good interestingness tests for miscompilations takes a bit of practice. More than one user has described C-Reduce as being something like the sorcerer’s apprentice: it does an excellent job reducing according to the criteria it is given, but if these criteria contain any kind of loophole, C-Reduce is likely to find it. For example it is easy to accidentally write a test which believes that the empty file is interesting. Additionally, a good interestingness test is written in such a way that its expected-case runtime is minimized by asking the quickest and most-likely-to-fail questions first.

From the start, C-Reduce was designed to produce a very small final reduced test case, even when this would make C-Reduce run for longer than we liked. This is based on the premise that we should burn cycles instead of human time, and that reporting a compiler bug is rarely on the critical path; we can often afford to wait for a better result. The consequences of this decision can be seen in [Tables 1 and 2 of this paper](https://people.inf.ethz.ch/suz/publications/perses.pdf) that evaluates several test-case reduction methods: C-Reduce produces the smallest final output, but takes more time to do so.

## A Modular, Domain-Independent Reducer Core

Although C-Reduce started out as a pet project solving a specific problem, it evolved into a research project involving a number of my colleagues, whose top-level goal was to produce an effective and usable reducer for C and C++ code as found in the wild. The first research contribution to come out of this effort was a way to achieve a clean mechanism/policy separation in a test case reducer. Previous reduction techniques had all baked specific transformations into the overall search strategy. This impedes extensibility, which we believe to be crucial. The structure that we ended up with is a small core that invokes a collection of pluggable transformation passes until a global fixpoint is reached.

The API for C-Reduce passes is simple but — like many simple things — required a lot of iterations before it felt finished. It is based on the ideas that transformation passes should be stateless and that every pass should implement a linear sequence of transformations, each of which results in a variant that may or may not be interesting. The interface is as follows:

`state new(filename, option)` : Return a fresh state object. Each pass uses this state to keep track of where it is in the sequence of transformations that it is capable of performing. These states may contain arbitrary data items; the C-Reduce core treats them as opaque. A typical pass stores some kind of cursor — a byte offset, token offset, line number, position in a tree traversal, etc. — in the state object.

The file referred to by filename is logically part of the state object even though it resides in the filesystem instead of memory. Of course it would not be difficult to pass the contents of the file around as a memory object but this would be slow when these objects are large (C-Reduce is frequently invoked on multi-megabyte preprocessed C++ files).

The “option” is used to select among different behaviors implemented by a composite pass.

`state advance(filename, option, state)` : Return a new state object referring to the next transformation opportunity following the one referenced by the state object passed as a parameter.

`result transform(filename, option, state)` : Modify the file in-place, selecting the transformation instance referred to by the state object. The result takes one of three values:

- OK : the transformation succeeded

- STOP : no more transformation instances remain for this pass

- ERROR : something went wrong; for example, an external tool crashed, a working file or directory could not be created, etc.

(The API contains one additional method, which checks whether a pass’s external dependencies are satisfied, that doesn’t matter here.)

Our experience has been that every transformation pass that we wanted has been easy to implement behind this API.

The C-Reduce core implements this algorithm:

```
current = original_test_case
do
  size_at_start = size(current)
  foreach (p, option) in pass_list
    state = p::new(current, option)
    do
      variant = current         // this is a file copy operation
      result = p::transform(variant, option, state)
      if result == ERROR
	report_problem_in_pass(p, option)
      if result == OK
        if is_interesting(variant)
          current = variant     // also a file copy
	else
          state = p::advance(current, option, state)
    while result == OK
while size(current) < size_at_start

```

The termination argument for C-Reduce is:

1. Since the outermost loop requires the size of the test case to decrease monotonically, it can only execute as many times as the size (in bytes) of the unreduced test case. In practice it executes many fewer times than this.

2. The loop over passes terminates because the pass list is immutable once C-Reduce is initialized.

3. Each iteration of the innermost loop either advances the state object or else (by selecting an interesting variant) removes one transformation opportunity. Either way, the number of remaining transformations decreases by one.

4. The interestingness test is, at worst, terminated (using OS support for killing all processes in a process group) after a configurable timeout.

In practice the weak link in this argument is case 3, which is vulnerable to bugs in passes. C-Reduce terminates robustly by abandoning passes when they appear to be behaving unreasonably.

The C-Reduce core does not insist that transformations make the test case smaller, and in fact quite a few of its passes potentially increase the size of the test case, with the goal of eliminating sources of coupling within the test case, unblocking progress in other passes.

The sequence of transformation passes is carefully orchestrated such that passes that are likely to give the biggest wins -- such as those that remove entire functions -- run first; otherwise the tool would end up spending days or weeks doing silly things such as trying to shrink numeric constants in a huge source file. Shrinking numbers is useful, and it should be done, but only after many other reduction mechanisms have run to completion.

C-Reduce's collection of cooperating passes, with heavy phase-ordering constraints, is highly reminiscent of how a modern optimizing compiler works. However, only a small proportion of the transformation passes is intended to be semantics-preserving in the sense that a compiler's optimization passes must be. In this domain, we only want to preserve enough semantics that we can probabilistically avoid breaking whatever property makes a test case interesting.

A consequence of writing a modular reducer is that once we came up with the right API for writing passes, we were free to write a lot of passes. My colleagues and I spent several years doing this and we ended up with:

- 35 passes, implemented in Perl, that include heuristics such as removing lines, removing various kinds of matched delimiters (and perhaps also the text between them), and shrinking integer values

- 6 passes that invoke external utilities such as [unifdef](https://dotat.at/prog/unifdef/), a partial evaluator for the C preprocessor language, a lexer for C and C++ that supports various token-level reduction transformations, and pretty-printing utilities that make the reduced test case more pleasant to look at

- 69 passes, implemented in C++, that use LLVM's Clang front end as a library for source-to-source transformation of C and C++ code; these include function inlining, partial template instantiation, scalar replacement of aggregates, copy propagation, and eliminating levels of a class hierarchy.

The actual number of dynamic passes is larger than the total of these numbers since some passes can be invoked in different modes using the "option" parameter mentioned above.

In this piece, we looked at why we had to create C-Reduce and at the modular structure that was key to making it solve the problems that we wanted to solve. In [Part 2](https://blog.regehr.org/archives/1679), I'll describe how C-Reduce improves reduction times using multiple cores and why C-Reduce usually does a good job reducing test cases in languages other than C and C++; finally, I'll discuss a few open research problems in test case reduction.
