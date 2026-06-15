---
title: security implications of jit compilation — wingolog
url: https://wingolog.org/archives/2011/06/21/security-implications-of-jit-compilation
published: "2011-06-21T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/06/21/security-implications-of-jit-compilation
---

# security implications of jit compilation — wingolog

## [security implications of jit compilation](/archives/2011/06/21/security-implications-of-jit-compilation)

21 June 2011 5:00 PM

- [igalia](/tags/igalia)
- [jit](/tags/jit)
- [compilers](/tags/compilers)
- [v8](/tags/v8)
- [dalvik](/tags/dalvik)
- [android](/tags/android)
- [guile](/tags/guile)
- [security](/tags/security)

Periodically the [tubes alight](http://daringfireball.net/2011/03/nitro_ios_43) with statements that just-in-time (JIT) compilers are somehow bad for security. At [Igalia](http://igalia.com/) we work with many companies that have put or are interested in putting JIT compilers on mobile devices, and obviously security is one of their concerns. In this article I'd like to take a closer look at the risks and myths surrounding JIT and security.

**executive summary**

JIT compilers do not cause a significant security risk in and of themselves.

However, JIT compilers do allow flaws in unsafe native libraries to be more easily exploited.

The problem is not JIT compilation. The problem is code written in unsafe languages.

**a background: interpreters and security**

It used to be that JavaScript implementations all used interpreters to execute JavaScript code, usually in bytecode form. An interpreter is a program, usually written in C or C++, which interprets program code as data.

(I will take JavaScript as the example in this article, though similar considerations apply to other language implementations.)

JavaScript implementations also includes a language runtime, which implements all of the "primitive" objects of the language. In this case of JavaScript this might include Date objects and such. The runtime also includes a garbage collector, a parser, and other such things.

Sometimes interpreters are used to run "external" code, as is the case with JavaScript. In that case the language implementation is a fairly clear attack target: the attacker tries to produce an input which triggers a flaw in the language implementation, allowing takeover of the target system.

Note that in this case, the interpreter itself is only part of the target. Indeed the interpreter is typically only a small part of a language implementation, and is very well-tested. Usually the vulnerability is in some bit of the runtime, and often stems from having the runtime written in an unsafe language like C or C++.

So the interpreter part itself typically imposes no risk, beyond the obvious ones of interpreting data provided by an attacker.

**compilers have more moving pieces**

Instead of interpreting a program as data, compilers produce native code. A compiler is bigger than an interpreter, and so exposes a larger attack surface. A compiler usually has more corner cases than an interpreter, which often are not tested properly. Compilers are usually specific to particular target architectures, which makes bugs in less-used architectures more likely. Additionally, by generating native code, a compiler operates outside whatever meager safety guarantees are provide by the implementation language (C or C++).

So yes, compilers are a risk, JIT or otherwise. However with proper testing it is possible to have confidence in a compiler. First of all, a compiler is written by compiler writers, not ordinary programmers, and not attackers. Secondly a compiler is generally a functional process, so it's easy to test. Modern JS implementations have thousands and thousands of tests. It's still a cross-your-fingers-and-pray thing, [usually](http://compcert.inria.fr/), but the fact is that exploits these days come from bugs present in the source of unsafe languages, not bugs in code generated from safe languages.

**jit compilation allows performant mobile code**

JIT compilation can actually increase the overall security of a system, if one of the design goals of that system is user-installable "apps". For example, Android apps are (typically) written in Java, targetting the Dalvik VM. This choice of safe language and safe runtime largely eliminates the need to make a tight sandbox around apps. If it weren't for the presence of unsafe code in the libraries available to an Android app, things would be even easier.

Note that Dalvik's JIT has other positive security characteristics, in that its generated code can be verified against the bytecodes that it is supposed to implement. I recommend [this video](http://www.youtube.com/watch?v=Ls0tM-c4Vfo) for a good discussion; thanks to my colleague [Xan](http://blogs.gnome.org/xan/) for pointing it out.

In contrast, Apple's decision to allow natively-compiled applications to be installed by users leaves them scrambling to try to mitigate the effects of what happens *when* an app has a buffer overflow. Again, the problem is unsafe code, not JIT compilation.

**attempts to contain security vulnerabilities in unsafe code**

That said, unsafe languages are part of the sordid reality, so let's look at how attackers compromise such applications.

In the old days, once you got a stack buffer overflow, you just wrote whatever code you wanted onto the stack, and arranged for a function return to execute that code. [Smashing the stack for fun and profit](http://www.phrack.org/issues.html?id=14&issue=49), you see.

These days it is more difficult. One of the things that makes it more difficult is making it impossible to write code which is later executed. This technique goes under various names: the NX bit, DEP, W⊕X, etc. Basically it means you can't write shellcode. But, you can [return to libc](http://en.wikipedia.org/wiki/Return-to-libc_attack), which uses existing functions in libc (like `system`).

Some systems also prevent you from running code on disk unless it is signed. Dion Blazakis wrote a [nice overview on the sandbox in iOS devices](http://dl.packetstormsecurity.net/papers/general/apple-sandbox.pdf) (subtitle: using Scheme for evil). Besides not letting you write code for your own device -- a terrible thing IMO, but not related to security -- this technique does prevent attackers from getting around the NX restriction by writing code to disk, then executing it.

But one need not write code to perform arbitrary computation. I assume everyone's seen the following papers, and if not, I highly recommend them:

- [The geometry of innocent flesh on the bone: Return-into-libc without function calls (on the x86)](http://cseweb.ucsd.edu/~hovav/dist/geometry.pdf), Hovav Shacham, CCS 2007

- [When good instructions go bad: generalizing return-oriented programming to RISC](http://www.erikbuchanan.com/brss_ccs2008.pdf) Erik Buchanan et al, CCS 2008

- [Return-oriented programming without returns](http://cseweb.ucsd.edu/~hovav/dist/noret-ccs.pdf), Stephen Checkoway et al, CCS 2010

Finally, instead of loading libc always at the same place, the latest thing to happen is to randomize the positions at which the various segments are loaded in memory. This is called Address Space Layout Randomization (ASLR). To effectively execute a return to libc under ASLR, you have to figure out where libc is first; not a trivial task.

I think I need to mention again that these are not security measures *per se*. If you are really interested in security, don't write systems software in unsafe languages. These measures are designed to contain the effects of a vulnerability.

I should also note that all of these measures are actively harmful to people writing code. ASLR increases load time by disallowing prelinking of various sorts, and forcing the generation of position-independent code. Disallowing writable, executable memory means that you can't do JIT compilation, not to mention [quajects](http://valerieaurora.org/synthesis/SynthesisOS/). Disallowing unsigned binaries means that you can't run your code on your own device.

**jit compilation is not compatible with these restrictions**

Having a JIT compiler by definition means that you are writing code and then going to run it. This makes it easier for an attacker who has found a bug in a native library to exploit it.

These exploits do happen, though currently not very often. A good example of such a paper is [Interpreter Exploitation: Pointer Inference And JIT Spraying](http://www.semantiscope.com/research/BHDC2010/BHDC-2010-Paper.pdf) by Dion Blazakis.

Blazakis uses characteristics of the language runtime (Flash, in that case) to allow him to determine the address ranges to write his code. The actual code is written by taking advantage of the predictable output of the particular JIT in question to spray a big [NOPsled](http://en.wikipedia.org/wiki/NOP_slide) onto the heap, which if I understand it correctly, is implemented by starting the IP in the middle of x86 instructions, in lovely contrapuntal style. Very clever stuff.

But again, the vulnerability is present not in code related to the JIT, but in native code. Having executable pages makes the vulnerability easier to exploit, but is not itself the security risk.

**specific jit implementations**

I called out Dalvik as an example of a JIT implementation that is particularly security-conscious. As much as I like V8, though, I cannot recommend it particularly, from the security perspective. It is simply changing too fast. Practically every commit touches code generators. The individual architectures share some pieces but duplicate a lot. I cannot speak for the other JavaScript implementations, as I do not know them as well.

V8's primary use case in the Chromium browser does have some security safeguards though, benefitting from the process sandbox and rapid release cycle. But I would be hesitant to put it in a device, lacking the same safeguards.

**mitigations for jit vulnerabilities**

These days all signs are pointing to the end of sandboxing as a security strategy, replacing it instead with verification and software fault isolation (SFI). Google's Native Client group is leading the way here. For more, see [Adapting software fault isolation to contemporary CPU architectures](http://www.usenix.org/events/sec10/tech/full_papers/Sehr.pdf) by Sehr et al. But it does seem a poor thing to try to turn an unsafe language into a "fault-isolated" runtime; better to start with safe code to begin with.

Adapting SFI to JIT compilers could protect against flaws in JIT compilers, but it has [abysmal results](http://people.csail.mit.edu/jansel/papers/2011pldi-nacljit.pdf). The authors of that paper adapted V8 to produce the code sequences needed to preserve the SFI invariants, and got killed in speed. They somehow managed to claim success though, which is beyond me.

**in closing**

If you've already decided that you're going to ship remotely exploitable libraries written in unsafe languages, disallowing JIT can indeed reduce your attack surface. I understand it in some cases. But it is a shame to make safe languages pay the sins of their unsafe brethren. If you can, be Android, and not iOS.

## related articles

- [just-in-time code generation within webassembly](/archives/2022/08/18/just-in-time-code-generation-within-webassembly)
- [lightening run-time code generation](/archives/2019/05/24/lightening-run-time-code-generation)
- [design notes on inline caches in guile](/archives/2018/02/07/design-notes-on-inline-caches-in-guile)
- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [effects analysis in guile](/archives/2014/05/18/effects-analysis-in-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)

### 3 responses

1. [Brion Vibber](http://brionv.com) says:[22 June 2011 1:02 AM](#e45caaee36c8a43a722bb5019373421907fd48bb)

   I'm generally in agreement, but beware that Android does in fact allow users to install apps with native code libraried called through JNI - this is how Firefox for Android is implemented.

   Actual app security is enforced through process and user separation, not unlike iOS; but an app written in Java will naturally avoid classes of errors that one with a lot of native code.

2. [chris](http://www.matasano.com/research/) says:[5 July 2011 5:06 PM](#34d1353d48a186689e340d9a25a5b7a8080cf037)

   Great blog post. Completely disagree sandboxes are on their way out as a security strategy, they are just getting started. Unless of course you mean language sandboxes, even then I disagree. JITs raise a number of security risks to applications that use them. Relying on the 'only native code is bad' argument is not a very strong argument IMO. JIT's themselves are often written in C/C++, so they count as attack surface if that argument is true. We will be talking about JIT security at Blackhat this year https://www.blackhat.com/html/bh-us-11/bh-us-11-briefings.html

3. [wingo](http://wingolog.org/) says:[6 July 2011 8:33 AM](#6d97160fb848b73238edd59d31281c38635a3e59)

   Thanks for stopping by, Chris. I look forward to reading your paper.

Comments are closed.
