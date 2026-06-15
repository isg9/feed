---
title: guile bar mitzvah — wingolog
url: https://wingolog.org/archives/2008/11/02/guile-bar-mitzvah
published: "2008-11-02T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/11/02/guile-bar-mitzvah
---

# guile bar mitzvah — wingolog

## [guile bar mitzvah](/archives/2008/11/02/guile-bar-mitzvah)

2 November 2008 1:50 AM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [compilation](/tags/compilation)

Friends, neighbors, &c: ahem! May I present a new compiler!

**checkout**

Check out and build the `vm` branch of the Guile repository:

```
git clone git://git.sv.gnu.org/guile.git
git checkout -b vm origin/vm
./autogen.sh && ./configure && make

```

If you haven't ever run Guile before, do yourself a favor and enable readline and the value history:

```
cat > ~/.guile <the unboxingFire it up, foolios!$ ./pre-inst-guile
Guile Scheme interpreter 0.5 on Guile 1.9.0
Copyright (C) 2001-2008 Free Software Foundation, Inc.

Enter `,help' for help.
scheme@(guile-user)>
I must say, Guile starts up in a lovely and friendly fashion these days.The scheme part of the prompt indicates the current language. Those of you that are a bit longer in the tooth may recall the TCL wars, in which Guile was born as an attempt to be a common language runtime for all of free software. Part of the idea was that many languages could be compiled to Guile Scheme, which could run them all together in some kind of Harmony.So, the idea failed, for many reasons. But the basics remain sound: if you have an open, layered compiler, and a VM that you control, there's no reason why you shouldn't be able to replace the top layer or two of the compiler with something that accepts, for example, Emacs Lisp.I digress. The prompt tells us that we are in the scheme language, in the (guile-user) module -- and it's waiting with bated breath for us to type in an expression. I'll just show the prompt as > from now on.> 1.0
$1 = 1.0
OK, it does react to us! The $1 =  part of the output is what (ice-9 history) does for us, by the way. But what to do now? Hm... Better see what was going on with that help thing...> ,help

Help Commands [abbrev]:

 ,help [GROUP]                 [,h] - List available meta commands.
 ,apropos REGEXP               [,a] - Find bindings/modules/packages.
 ,describe OBJ                 [,d] - Show description/documentation.
 ,option [KEY VALUE]           [,o] - List/show/set options.
 ,quit                         [,q] - Quit this session.

Command Groups:

 ,help all                          - List all commands
 ,help module                       - List module commands
 ,help language                     - List language commands
 ,help compile                      - List compile commands
 ,help profile                      - List profile commands
 ,help debug                        - List debug commands
 ,help system                       - List system commands

Type `,COMMAND -h' to show documentation of each command.
See, this is what I really like about GNU. There is a culture of the command line in all GNU software. Here we see that there is loads of interactive documentation and functionality available to us, in a system which has the Emacs nature: self-describing, and extensible within its own language.One of my favorite commands is ,disassemble, abbreviated as ,x:> (define (hello) (display "Hi mom!") (newline))
> ,x hello
Disassembly of #:

Bytecode:

   0    (late-variable-ref 0)           ;; `display'
   2    (object-ref 1)                  ;; "Hi mom!"
   4    (call 1)                                              at (unknown file):1:16
   6    (drop)
   7    (late-variable-ref 2)           ;; `newline'
   9    (goto/args 0)                                         at (unknown file):1:36
In this particular case there's not much going on. The thing to note is that since we haven't run the procedure yet, the toplevel variables haven't yet been looked up (the late-variable-ref calls above), so they print as symbols. The locations corresponding to those symbols will be cached when the procedure is first called, relative to the module that was current when `hello' was defined.As you can see, the VM that the compiler is targetting is a simple stack machine. There are only three registers: the stack pointer, the current frame pointer, and the instruction pointer. This simple model maps fairly directly to Scheme's semantics.Programs, the name given to compiled procedures, are code with data: they have a static array associated with all invocations of the procedure (as in the object-ref above), and a list of heap-allocated values, the external list:> ,x (let ((x 10)) (lambda () (set! x (* 2 x)) x))
Disassembly of #:

Bytecode:

   0    (make-int8 2)                   ;; 2
   2    (external-ref 0)                ;; (closure variable)
   4    (mul)
   5    (external-set 0)                ;; (closure variable) at (unknown file):0:28
   7    (external-ref 0)                ;; (closure variable)
   9    (return)

Externals:

   0    10
It's interesting, it seems you need to allocate lexicals on the heap in two circumstances: one, if they are lexically referenced by some procedure, or two, if they are ever mutated. The latter point is a bit subtle, but in the presence of call/cc, you need to capture the identity of variables, not their values.So it turns out that what you think is "closer to the metal", viz. set!, actually forces the compiler to take the less efficient route. (Of course the Haskell people know this already.)There are some optimizations that can be made, but in general if you call out to a separately compiled procedure, you have no idea what that procedure will do -- so you need to allocate mutated variables on the heap.statusOf course, anyone can make a toy compiler for a toy language. So what's the big deal here? Well to me, for my own code, the beauty is that it's not a "toy compiler": there is a large corpus of Guile code out there, and this compiler aims to interoperate with just about all of it.So anything that is out there that is valid Guile should run just fine with compiled code. It's taken a bit of work, over the past six months or so, but I think we're pretty much at that state.Furthermore, we permit the intermixing of the existing routines that are hard-coded in C, with the existing code that runs in the memoizer/interpreter, and with whatever subset of the existing code that has been run through the compiler. You can call seamlessly from one to the other.Currently Guile's compiler is only slightly faster than its evaluator -- which is to be expected in many ways. The evaluator was tuned over many years to reach its current speed, and the VM still has many paranoid checks enabled in its source code. I expect that over time, VM code will be faster than the evaluator by a factor of about 5 or 10.More than that, having compiled source to bytecodes gives us a nice low-level format to work with, highly amenable to JIT compilation to native code. This is where I really want to go -- it won't much matter in cases like the ones given above, but in highly computational algorithms this will be the difference between the delight of coding in Scheme and the drudgery of C.historyI started on this project because I had a social investment in Guile, and wanted my own code to run faster. Luckily, Keisuke Nishida already did all of the hard work for me.Working with Kei's code has been incredibly enlightening, we've been having conversations, displaced in time: I come up with the questions, then I notice the answers already in the source. I have never actually corresponded with Keisuke in the conventional sense. This is a beautiful and strange thing about free software.Since then, the code passed through Ludovic Courtès' hands for some years, then I unearthed it and brought it to where it is today. It's been a pleasing task, having all of the right decisions already made for you, just having to fill in the missing bits.I should also mention that I had no idea how to write a compiler -- Ghuloum's beautiful paper notwithstanding. But seeing code that breaks it down into levels that I understand has empowered me as a programmer to create things that I did not think possible.what rough beast, its hour come round at lastThe way I reckon it, Guile is at least 13, so a coming-of-age ceremony is appropriate. Once the VM branch gets merged to master, we'll start to get the gears in motion for a Guile 2.0 -- source-compatible, but much faster.If you, dear reader, have been piqued by any of this: well by all means subscribe to guile-devel, where the wonderful things happen.The pleasant thing about all this is that it's a small group of people, and that one individual working hard and working smart can make an enormous difference. So come along and play out those compiler optimization fantasies that you've been keeping secret!
```

## related articles

- [scheme modules vs whole-program compilation: fight](/archives/2024/01/05/scheme-modules-vs-whole-program-compilation-fight)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [((lambda (x) ((compile x) x)) '(lambda (x) ((compile x) x)))](/archives/2008/11/14/lambda-x-compile-x-x-lambda-x-compile-x-x)
- [breaking silence, innocuously](/archives/2008/09/18/breaking-silence-innocuously)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)

### 2 responses

1. djcb says:[2 November 2008 10:15 AM](#f039a38be5104de7d372836975b5f069ccba1ca6)

   wow, my congratulations, great work andy!

2. julian says:[5 November 2008 11:19 PM](#6152ba3b77d6580686ae9bb636a7894ba72905ee)

   Andy Wingo is my hero!

Comments are closed.
