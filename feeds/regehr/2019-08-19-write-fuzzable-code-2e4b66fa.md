---
title: Write Fuzzable Code
url: https://blog.regehr.org/archives/1687
published: "2019-08-19T16:09:50Z"
feed: regehr
guid: http://blog.regehr.org/?p=1687
---

# Write Fuzzable Code

Fuzzing is sort of a superpower for locating vulnerabilities and other software defects, but it is often used to find problems baked deeply into already-deployed code. Fuzzing should be done earlier, and moreover developers should spend some effort making their code more amenable to being fuzzed.

This post is a non-comprehensive, non-orthogonal list of ways that you can write code that fuzzes better. Throughout, I’ll use “fuzzer” to refer to basically any kind of randomized test-case generator, whether mutation-based (afl, libFuzzer, etc.) or generative (jsfunfuzz, Csmith, etc.). Not all advice will apply to every situation, but a lot of it is sound software engineering advice in general. I’ve bold-faced a few points that I think are particularly important.

## Invest in Oracles

A test oracle decides whether a test case triggered a bug or not. By default, the only oracle available to a fuzzer like afl is provided by the OS’s page protection mechanism. In other words, it detects only crashes. We can do much better than this.

[Assertions](https://blog.regehr.org/archives/1091) and their compiler-inserted friends — sanitizer checks — are another excellent kind of oracle. You should fuzz using as many of these checks as possible. Beyond these easy oracles, many more possibilities exist, such as:

- function-inverse pairs: does a parse-print loop, compress-decompress loop, encrypt-decrypt loop, or similar, work as expected?

- differential: do two different implementations, or modes of the same implementation, show the same behavior?

- metamorphic: does the system show the same behavior when a test case is modified in a semantics-preserving way, such as adding a layer of parentheses to an expression?

- resource: does the system consume a reasonable amount of time, memory, etc. when processing an input?

- domain specific: for example, is a lossily-compressed image sufficiently visually similar to its uncompressed version?

**Strong oracles are worth their weight in gold, since they tend to find application-level logic errors rather than the lower-level bugs that are typically caught by looking for things like array bounds violations.**

I [wrote a bit more about this topic](https://blog.regehr.org/archives/856) a few years ago. Finally, a twitter user suggested “If you’re testing a parser, poke at the object it returns, don’t just check if it parses.” This is good advice.

## Interpose on I/O and State

Stateless code is easier to fuzz. Beyond that, you will want APIs for taking control of state and for interposing on I/O. For example, if your program asks the OS for the number of cores, the current date, or the amount of disk space remaining, you should provide a documented method for setting these values. It’s not that we necessarily want to randomly change the number of cores, but rather that we might want to fuzz our code when set to single-core mode and then separately fuzz it in 128-core mode. Important special cases of taking control of state and I/O include making it easy to reset the state (to support persistent-mode fuzzing) and avoiding hidden inputs that lead to non-deterministic execution. **We want as much determinism as possible while fuzzing our code.**

## Avoid or Control Fuzzer Blockers

Fuzzer blockers are things that make fuzzing gratuitously difficult. The canonical fuzzer blocker is a checksum included somewhere in the input: the random changes made to the input by a mutation-based fuzzer will tend to cause the checksum to not validate, resulting in very poor code coverage. There are basically two solutions. First, turn off checksum validation in builds intended for fuzzing. Second, have the fuzzer generate inputs with valid checksums. A generation-based fuzzer will have this built in; with a mutation-based fuzzer we would write a little utility to patch up the test case with a valid checksum after it is generated and before it is passed to the program being fuzzed. [afl has support for this](https://github.com/mirrorer/afl/blob/master/experimental/post_library/post_library.so.c).

Beyond checksums, hard-to-satisfy validity properties over the input can be a serious problem. For example, if you are fuzzing a compiler for a strongly typed programming language, blind mutation of compiler inputs may not result in valid compiler inputs very often. I like to think of validity constraints as being either soft (invalid inputs waste time, but are otherwise harmless) or hard (the system behaves arbitrarily when processing an invalid input, so they must be avoided for fuzzing to work at all). When we fuzz a C++ compiler to look for wrong code bugs, we face a hard validity constraint because compiler inputs that have UB will look like wrong code bugs. There is no simple, general-purpose solution to this kind of problem, but rather a family of techniques for explicitly taking validity properties into account. The most obvious solution — but often not the right one — is to write a new generational fuzzer. The problem is that if you do this, you cannot take advantage of modern coverage-driven fuzzing techniques, which are amazing. To fit into a coverage-driven fuzzing framework you have a couple of options. First, write a custom mutator that respects your validity constraints. **Second, [structure-aware fuzzing](https://github.com/google/fuzzer-test-suite/blob/master/tutorial/structure-aware-fuzzing.md), which basically means taking the mutated data from the fuzzer and translating it into something like what the program being fuzzed expects to see.** There’s a lot of research left to be done in making coverage-driven fuzzers work well in the presence of validity constraints without requiring a lot of manual effort. There are some significant subtleties here, maybe I’ll go into them another time. Putting something like a SAT solver into the fuzzer is not, generally speaking, the answer here, first because some validity constraints like checksums are specifically difficult for solvers, and second because some validity constraints (such as UB-freedom in a C++ program) are implicit and cannot be inferred, even in principle, by looking at the system being fuzzed.

A lot of code in a typical system cannot be fuzzed effectively by feeding input to public APIs because access is blocked by other code in the system. For example, if you use a custom memory allocator or hash table implementation, then fuzzing at the application level probably does not result in especially effective fuzzing of the allocator or hash table. These kinds of APIs should be exposed to direct fuzzing. **There is a strong synergy between unit testing and fuzzing: if one of these is possible and desirable, then the other one probably is too. You typically want to do both.**

Sanitizers and fuzzers often require tweaks or even significant changes to the build process. To make this easier, keep the build process as clean and simple as possible. Make it easy to switch out the compiler and modify the compiler options. Depend on specific tools (and versions of tools) as little as possible. Routinely build and test your code with multiple compilers. Document special build system requirements.

Finally, some fuzzer blockers are sort of silly and easy to avoid. If your code leaks memory or terminates its process with a deep call stack, it will be painful to test using a persistent-mode fuzzer, so don’t do these things. Avoid handling SIGSEGV or, if you really must do this, have a way to disable the handler for fuzzer builds. If your code is not compatible with ASan or UBSan, then these extremely useful oracles are harder to use. In particular, if your code uses a custom memory allocator you should consider turning it off for fuzzer builds, or else [adapting it to work with ASan](https://github.com/google/sanitizers/wiki/AddressSanitizerManualPoisoning), or else you’ll miss important bugs.

## Unblock Coverage-Driven Fuzzers

Because coverage-driven fuzzers refocus their effort to try to hit uncovered branches, they can be blocked in certain specific ways. For example, if a coverage-driven fuzzer is presented with too many uncoverable branches, it can spend so much time on them that it becomes less likely to hit more interesting branches elsewhere in the program. For example, one time I compared afl’s coverage on a program compiled with and without UBSan, and found that (in whatever time limit I used) it covered quite a lot less of the sanitized program, compared to the unsanitized build. On the other hand, we definitely want our fuzzer to look for sanitizer failures. My advice is to fuzz both sanitized and unsanitized builds of your program. I don’t know how to budget fuzzing resources for these different activities and don’t know of any principled work on that problem. It may not matter that much, since fuzzing is all about overkill.

Sometimes your program will call branchy, already-heavily-fuzzed code early in its execution. For example, you might decompress or decrypt input before processing it. This is likely to distract the coverage-driven fuzzer, causing it to spend a lot of time trying to fuzz the crypto or compression library. If you don’t want to do this, provide a way to disable crypto or compression during fuzzing.

Any interpreter in your program is likely to make life difficult for a coverage-driven fuzzer, since the relevant program paths are now encoded in the data being interpreted, which is generally opaque to the fuzzer. If you want maximum mileage out of coverage-driven fuzzers, you may want to try to avoid writing interpreters, or at least keep them extremely simple. An obvious way to deal with embedded interpreters — which someone must have thought of and tried, but I don’t have any pointers — would be to have an API for teaching the fuzzer how to see coverage of the language being interpreted.

## Enable High Fuzzing Throughput

Fuzzing is most effective when throughput is very high; this seems particularly the case for feedback-driven fuzzers that may take a while to learn how to hit difficult coverage targets. An easy throughput hack is to make it possible to disable slow code (detailed logging, for example) when it is not germane to the fuzzing task. Similarly, interposing on I/O can help us avoid speed hacks such as running the fuzzer in a ramdisk.

## “But I Want Fuzzing My Code to be Harder, Not Easier”

I don’t have a lot of sympathy for this point of view. Instead of aiming for security through obscurity, we would do better to:

- fuzz early and thoroughly, eliminating fuzzable defects before releasing code into the wild

- write code in a programming language with as strong of a type system as we are willing to tolerate — this will statically eliminate classes of bad program behaviors, for example by stopping us from putting the wrong kind of thing into a hashmap

- aggressively use assertions and sanitizers to get dynamic checking for properties that the type system can’t enforce statically

Anti-fuzzing techniques are a thing, but I don’t think it represents a useful kind of progress towards better software.

## Conclusion

Randomized testing is incredibly powerful and there’s no point avoiding it: if you don’t fuzz your code, someone else will. This piece has described some ways for you, the software developer, to make fuzzing work better. Of course there are plenty of other aspects, such as choosing a good corpus and writing a good fuzzer driver, that are not covered here.

**Acknowledgments:** Pascal Cuoq and Alex Groce provided feedback on a draft of this piece, and it also benefited from suggestions I received on Twitter. You can [read the conversation here](https://twitter.com/johnregehr/status/1154888675810934784); it contains some suggestions and nuances that I did not manage to capture.
