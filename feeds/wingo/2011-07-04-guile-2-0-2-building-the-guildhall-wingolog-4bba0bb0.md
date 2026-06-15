---
title: 'guile 2.0.2: building the guildhall — wingolog'
url: https://wingolog.org/archives/2011/07/04/guile-2-0-2-building-the-guildhall
published: "2011-07-04T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/07/04/guile-2-0-2-building-the-guildhall
---

# guile 2.0.2: building the guildhall — wingolog

## [guile 2.0.2: building the guildhall](/archives/2011/07/04/guile-2-0-2-building-the-guildhall)

4 July 2011 9:47 AM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [compilers](/tags/compilers)
- [gnu](/tags/gnu)
- [guildhall](/tags/guildhall)
- [dorodango](/tags/dorodango)

Hey tubes, we made another [Guile](http://www.gnu.org/software/guile/) release!

**what is guile**

Guile is the [GNU](http://www.gnu.org/) extension language. It's an implementation of Scheme. It's pretty fun to hack in, and on!

**guile 2.0.2: even faster!**

Here I'd like to highlight the thing that I really like about this release: performance. Relative to 2.0.1:

- The VM is 20% faster

- Compiled `.go` files are half the size

- Startup time is down from 16.2 milliseconds to 11.7 milliseconds (on my machine)

- Base memory use down from 3.8 MB of private dirty memory to 2.9MB (measured on a `guile -c '(sleep 100)'`)

In short, thundercats are totally on the loose! See the [release announcement](http://thread.gmane.org/gmane.lisp.guile.devel/12633) for all the details.

**where we're going**

It's hard to tell sometimes where this jalopy is headed. Sometimes the vehicle just drives itself, like the recent speed improvements, which were totally unplanned.

But that said, there are a few stops we want to make on this journey, in the near- and mid-term. Let's take a look.

**2.0: building the guildhall**

On the 2.0 branch, the current stable series, the big feature that we'd like to land soon will be a CPAN-like repository of installable Scheme modules.

We agonized over the name for a while. It needs to be short and memorable, so it can be used as a command-line tool. (The Git experience is pretty definitive here.) At the same time it needs to be linked to Guile, and follow some sort of theme.

What we've done now is to take our `guile-tools` script runner (equivalent to the `git` wrapper) and rename it to `guild`. Currently it's mostly used for compiling files, as in `guild compile -o foo.go foo.scm`, but in the future we want to use it as a part of this system for Guile wizards and journeyfolk to band together to share code---to form a guild, so to speak.

And so the package archive itself will be the "guild hall" (or "guildhall"). We'll see if we can get guildhall.gnu.org, as a place to host the packages.

As far as code, protocol, and formats go, we are looking at porting Andreas Rottmann's [Dorodango](http://home.gna.org/dorodango/) Scheme package manager to be Guile-specific, with `guild` commands. So `guild update`, `guild upgrade`, `guild install xml-rpc`, etc.

Dorodango is basically a port of Debian's `apt` to Scheme. It even has ports of the same dependency resolution algorithms, which is fun. It also allows for multiple sources, so we can have our Guile guildhall archive, but also have support for portable R6RS Scheme archives from Dorodango.

**2.2: bit mucking**

Guile is getting pretty good now, but there is more to be done before it is as cheap to run a program written in Scheme as it is to run a program written in C.

The first thing that we will do to help this is to switch to ELF as the format of our compiled code. We will do this on all platforms, and even before we do native compilation. The bytecode will simply be stored in an ELF section. This will also allow us to statically allocate more data, decreasing memory usage and increasing locality. Also our debugging information is currently interspersed with code -- not in the execution path, mind you, but it does make the memory profile of the code look more like emmental than comté cheese. Moving debug info out to a separate section will help with that.

ELF will also add flexibility. We can add sections to express dependencies of compiled code on source code. We can strip debug info and other things from compiled code. We can link multiple modules into one file, to avoid `stat` penalties on startup.

In addition ELF will allow us to add native code, in a separate section. Of course we have to actually produce native code, so that's the next big thing for Guile 2.2. The plan is to write an ahead-of-time code generator for x86 and x86-64 systems, while keeping the bytecode interpreter as well. Then once we know what we want, we can go and talk to the GCC folks and see if our use case can be supported there.

Anyway, that's the plan. I expect 2.2 to take another year or so. By the time we have native compilation, I hope also our Elisp implementation will be significantly faster than the one in Emacs, so it will really become attractive to people to finish the work that we have done to make the switch there.

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)

### 4 responses

1. Kelly Clowers says:[4 July 2011 9:24 PM](#2ce83868fcb76db1aa0b112b7e1dd9d55d17500b)

   > Guildhall

   > Dorodango is basically a port of Debian's apt to Scheme. It even has ports of the same dependency resolution algorithms, which is fun.

   Wow, sounds perfect. I have been on the fence about actually taking the time to learn a lisp, I think I will have to now.

2. guildroid says:[4 July 2011 10:35 PM](#5462f2d4056306e47e3f9a7d392d1693c89e6384)

   Guile REPL for Android ?

3. John says:[5 July 2011 3:23 AM](#6b496bf3c54ef6d8a345dbd1b0ee78845718313d)

   Very excited to hear that guildhall is on the way.

   Definitely motivates me to finally become familiar with Guile.

4. zorg says:[7 July 2011 4:02 AM](#f25929857ee1a67461a5c4237aa083cbd82dd789)

   There's already quite a few popular Scheme implementations around , Gambit, Chicken, Racket come to mind.

   How does Guite positions itself with the competition.

   In your opinion, what is Guile selling point ?

   Performances ?

   Libraries ?

   Easy to use FFI ?

   Something else ?

   Thanks

Comments are closed.
