---
title: 'Alive2 Part 1: Introduction'
url: https://blog.regehr.org/archives/1722
published: "2020-03-12T20:15:32Z"
feed: regehr
guid: http://blog.regehr.org/?p=1722
---

# Alive2 Part 1: Introduction

*\[This piece is co-authored by [Nuno P. Lopes](http://web.ist.utl.pt/nuno.lopes/) and John Regehr.\]*

Compiler bugs threaten the correctness of almost any computer system that uses compiled code. Translation validation is a path towards reliably correct compilation that works by checking that an individual execution of the compiler did the right thing. We created a tool, Alive2, that takes an unoptimized function in LLVM intermediate representation (IR) and either proves that it is refined by the optimized version of the function, or else shows a set of inputs that illustrate a failure of refinement.

We work in terms of refinement, rather than equivalence, because compilers often change the meaning of the code they are optimizing — but in allowed ways. For example, adding two variables of “int” type in a C or C++ program means something like “add the two quantities, returning the mathematical result as long as it is within an implementation-specified range that is no smaller than -32767..32767.” When integer addition is lowered from C or C++ to LLVM IR, this very restrictive range for validity will typically be refined to be -2147483648..2147483647. Then, when LLVM IR is lowered to a target platform such as ARM or x86-64, the behavior of additions whose results fall out of bounds is refined from “undefined” to “two’s complement wraparound.” Both of these refinements are perfectly legal, but would be incorrect if the transformation went in the other direction. Non-trivial refinement is ubiquitous during compilation and we have to take it into account in a translation validation tool.

Let’s look at an example of refinement and also learn how to invoke Alive2 and interpret its results. Here’s a pair of functions in LLVM IR:

```
define i16 @src(i16 %x, i16 %y) {
    %r = sdiv i16 %x, %y
    ret i16 %r
}

define i16 @tgt(i16 %x, i16 %y) {
    %z = icmp eq i16 %y, 0
    br i1 %z, label %zero, label %nonzero
zero:
    ret i16 8888
nonzero:
    %q = sdiv i16 %x, %y
    ret i16 %q
}

```

The first function, src, simply returns the quotient of its arguments. The second function, tgt, returns the quotient of its arguments unless the second argument is zero, in which case it returns 8888. It turns out that src is refined by tgt. This works because division by zero in src has undefined behavior: LLVM makes no promise whatsoever about the behavior of a program that divides a number by zero. tgt, on the other hand, has a specific behavior when its second argument is zero, and behaves the same as src otherwise. This is the essence of a refinement relationship. If we run the refinement check in the other direction, Alive2 will tell us that src does not refine tgt.

To see these results for yourself, you can build [Alive2](https://github.com/AliveToolkit/alive2) (making sure to enable translation validation at CMake time) and then invoke our alive-tv tool like this:

```
alive-tv foo.ll

```

Alternatively, you can simply visit our [Compiler Explorer (CE) instance](https://alive2.llvm.org/ce/). Let’s talk about this user interface for a second. When you load the page you should see something like this:

![](../extra_files/compiler-explorer1.png)

The large pane on the left is where you type or paste LLVM IR. The large pane on the right shows alive-tv’s output in the case where it is able to parse the LLVM code successfully. The “share” pulldown at the top provides a short link to what you are currently seeing: this is an excellent way to show other people your results. If your LLVM code contains an error, the right pane will say “Compilation failed” and the text at the bottom of the right pane will change from “Output (0/0)” to something like “Output (0/3)”. Click on this text and a third pane will open showing the error output. You should then see something like this:

![](../extra_files/compiler-explorer2.png)

After fixing the error you can close this new pane.

Returning to our example, here’s [Compiler Explorer with the example already loaded](https://alive2.llvm.org/ce/z/D-Dwdj). We’ve also given alive-tv the `--bidirectional` command line option, which causes it to run the refinement check in both directions. The output therefore has two parts; this is the first one:

```
----------------------------------------
define i16 @src(i16 %x, i16 %y) {
%0:
  %r = sdiv i16 %x, %y
  ret i16 %r
}
=>
define i16 @tgt(i16 %x, i16 %y) {
%0:
  %z = icmp eq i16 %y, 0
  br i1 %z, label %zero, label %nonzero

%nonzero:
  %q = sdiv i16 %x, %y
  ret i16 %q

%zero:
  ret i16 8888
}
Transformation seems to be correct!

```

“Transformation seems to be correct!” indicates that Alive2 thinks that src is refined by tgt. The arrow ( `=>`) points in the direction of the refinement relation. The second part of the output is:

```
----------------------------------------
define i16 @tgt(i16 %x, i16 %y) {
%0:
  %z = icmp eq i16 %y, 0
  br i1 %z, label %zero, label %nonzero

%nonzero:
  %q = sdiv i16 %x, %y
  ret i16 %q

%zero:
  ret i16 8888
}
=>
define i16 @src(i16 %x, i16 %y) {
%0:
  %r = sdiv i16 %x, %y
  ret i16 %r
}
Reverse transformation doesn't verify!
ERROR: Source is more defined than target

Example:
i16 %x = undef
i16 %y = #x0000 (0)

Source:
i1 %z = #x1 (1)
i16 %q = #xffff (65535, -1)	[based on undef value]

Target:
i16 %r = #xffff (65535, -1)

```

Notice that now the arrow points from tgt to src. This time Alive2 says:

```
Reverse transformation doesn't verify!
ERROR: Source is more defined than target

```

(Here in the reverse direction, “source” unfortunately refers to “tgt” and “target” refers to “src”.)

Interpreting a counterexample from Alive2 isn’t always easy. Part of the blame falls on LLVM’s tricky undefined behavior semantics, but also there are some UX details that we hope to improve over time. The main thing to remember is that a counterexample is an example: it represents a single execution which illustrates the failure of refinement. The most important part of a counterexample is the set of input values to the IR being checked:

```
i16 %x = undef
i16 %y = #x0000 (0)

```

These are, of course, the arguments to the functions. Given a set of input values, you should ideally be able to execute both the unoptimized and optimized functions in your head or on paper, in order to see the difference. Alas, this depends on the functions being fairly small and also on you having a good understanding of LLVM’s undefined behavior model.

The next part of the output is designed to help you think through the execution of the code; it provides intermediate values produced as the source and target execute the counterexample:

```
Source:
i1 %z = #x1 (1)
i16 %q = #xffff (65535, -1)	[based on undef value]

Target:
i16 %r = #xffff (65535, -1)

```

Now that we’re seen all of the parts of Alive2’s output, let’s work through this specific counterexample. The key is passing 0 as the argument to the %y parameter; the value of %x doesn’t matter at all. Then, tgt branches to %zero and returns 8888. src, on the other hand, divides by zero and becomes undefined. This breaks that rule that optimizations can make code more defined, but not less defined. We plan to augment Alive2’s counterexamples with some indication of where control flow went and at what point the execution became undefined. Also, we should suppress printing of values that occur in dead code. Alternately, and perhaps preferably, we could feed Alive2’s counterexample to a separate UB-aware LLVM interpreter (a tool that doesn’t yet exist, but should).

In general, when refinement fails, it means that there exists at least one valuation of the inputs to the function such that one of these conditions holds:

1. the source and target return different concrete answers
2. the source function returns a concrete answer but the target function is undefined
3. there is a failure of refinement in the memory locations touched by the function

We’ll explain memory refinement in a later post in this series.

Alive2’s predecessor, [Alive](http://www.cs.utah.edu/~regehr/papers/pldi15.pdf), was also a refinement checker. It was based on the idea that a user-friendly domain-specific language could make it easier for compiler developers to try out new optimizations. This language supported omitting some details found in real LLVM IR, such as how wide, in bits, an operation being optimized is. Alive2 still supports this domain-specific language, but much of our recent work has focused on handling LLVM IR directly. This means that we are working at a slightly lower level of abstraction than before, but it allows us to run our tool on large amounts of code found in the wild.

One of our main targets for Alive2 has been LLVM’s unit test suite: a collection of about 22,000 files containing LLVM IR. Each file specifies that one or more LLVM optimization passes should be run, and then checks that the file gets optimized as expected. When we run Alive2 over the test suite, we ignore the expected result and instead simply check that the optimized code refines the original code. Perhaps surprisingly, this has caught quite a few bugs: 27 so far. Why would the test suite itself, when all tests are in a passing state, be capable of triggering bugs in LLVM? This can happen for several reasons, but the most common one is that LLVM’s test oracles — which decide whether the compiler behaved correctly or not — are generally coarse-grained. For example, a unit test about optimizing division by power-of-two into right shift might simply test for the presence of a shift instruction in the optimized code. Alive2, on the other hand, acts as a fine-grained oracle: if the optimized code does not preserve the meaning of the original code for even one choice of input values, this fact will be detected and reported. [Here’s an example of a bug](https://bugs.llvm.org/show_bug.cgi?id=42198) we found this way.

Alive2 handles a large subset of LLVM, but not all of it. Right now there are three main limitations. First, loops are only partially analyzed. Alive2 unrolls loops once, so any miscompilation that only manifests when the loop runs for more than one iteration cannot be detected. Second, interprocedural optimizations are not supported. Although Alive2 can handle function calls in the code being analyzed, it does not look across functions. So a function inlining transformation cannot be analyzed, nor can something like removing an unused global variable. Third, some kinds of code will reliably cause the underlying SMT solver, Z3, to take too long to return a result. Floating point operations, wide integer divisions, and complex memory operations are common culprits.

This post has introduced Alive2 and shown off some of its basic capabilities. Subsequent posts in this series will cover more detailed technical topics such as Alive2’s memory model, verifying vector and floating point optimizations, and finding bugs in LLVM backends and also binary lifters that turn object code into LLVM IR.
