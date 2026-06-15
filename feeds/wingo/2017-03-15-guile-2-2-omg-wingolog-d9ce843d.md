---
title: guile 2.2 omg!!! — wingolog
url: https://wingolog.org/archives/2017/03/15/guile-2-2-omg
published: "2017-03-15T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2017/03/15/guile-2-2-omg
---

# guile 2.2 omg!!! — wingolog

## [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)

15 March 2017 10:56 PM

- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [gnu](/tags/gnu)
- [igalia](/tags/igalia)
- [compilers](/tags/compilers)
- [academia](/tags/academia)
- [relief](/tags/relief)

Oh, good evening my hackfriends! I am just chuffed to share a thing with yall: tomorrow we release Guile 2.2.0. Yaaaay!

I know in these days of version number inflation that this seems like a very incremental, point-release kind of a thing, but it's a big deal to me. This is a project I have been working on since soon after [the release of Guile 2.0](https://wingolog.org/archives/2011/02/16/guile-is-out) some 6 years ago. It wasn't always clear that this project would work, but now it's here, going into production.

In that time I have worked on JavaScriptCore and V8 and SpiderMonkey and so I got a feel for what a state-of-the-art programming language implementation looks like. Also in that time I ate and breathed optimizing compilers, and really hit the wall until finally paging in what Fluet and Weeks were saying so many years ago about [continuation-passing style and scope](https://sourceforge.net/p/mlton/mailman/message/31023809/), and eventually came through with a solution that was still CPS: [CPS soup](https://vimeo.com/182585166). At this point Guile's "middle-end" is, I think, totally respectable. The backend targets a [quite good virtual machine](https://www.gnu.org/software/guile/manual/html_node/A-Virtual-Machine-for-Guile.html#A-Virtual-Machine-for-Guile).

The virtual machine is still a bytecode interpreter for now; native code is a next step. Oddly my journey here has been precisely opposite, in a way, to [An incremental approach to compiler construction](http://scheme2006.cs.uchicago.edu/11-ghuloum.pdf); incremental, yes, but starting from the other end. But I am very happy with where things are. Guile remains very portable, bootstrappable from C, and the compiler is in a good shape to take us the rest of the way to register allocation and native code generation, and [performance is pretty ok](https://ecraven.github.io/r7rs-benchmarks/benchmark.html), even better than some natively-compiled Schemes.

For a "scripting" language (what does that mean?), I also think that Guile is breaking nice ground by using [ELF as its object file format](https://wingolog.org/archives/2014/01/19/elf-in-guile). Very cute. As this seems to be a "Andy mentions things he's proud of" segment, I was also pleased with how [we were able to completely remove the stack size restriction](https://wingolog.org/archives/2014/03/17/stack-overflow).

**high fives all around**

As is often the case with these things, I got the idea for removing the stack limit after talking with Sam Tobin-Hochstadt from Racket and the PLT group. I admire Racket and its makers very much and look forward to stealing fromworking with them in the future.

Of course the ideas for the [contification and closure optimization passes](https://wingolog.org/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure) are in debt to Matthew Fluet and Stephen Weeks for the former, and Andy Keep and Kent Dybvig for the the latter. The intmap/intset representation of CPS soup itself is highly endebted to the late Phil Bagwell, to Rich Hickey, and to Clojure folk; persistent data structures were an amazing revelation to me.

Guile's virtual machine itself was initially heavily inspired by JavaScriptCore's VM. Thanks to WebKit folks for writing so much about the early days of Squirrelfish! As far as the actual optimizations in the compiler itself, I was inspired a lot by V8's Crankshaft in a weird way -- it was my first touch with fixed-point flow analysis. As most of yall know, I didn't study CS, for better and for worse; for worse, because I didn't know a lot of this stuff, and for better, as I had the joy of learning it as I needed it. Since starting with flow analysis, Carl Offner's [Notes on graph algorithms used in optimizing compilers](http://www.cs.umb.edu/~offner/files/flow_graph.pdf) was invaluable. I still open it up from time to time.

While I'm high-fiving, large ups to two amazing support teams: firstly to my colleagues at [Igalia](https://igalia.com) for supporting me on this. Almost the whole time I've been at Igalia, I've been working on this, for about a day or two a week. Sometimes at work we get to take advantage of a Guile thing, but Igalia's Guile investment mainly pays out in the sense of keeping me happy, keeping me up to date with language implementation techniques, and attracting talent. At work we have a lot of language implementation people, in JS engines obviously but also in other niches like the networking group, and it helps to be able to transfer hackers from Scheme to these domains.

I put in my own time too, of course; but my time isn't really my own either. My wife Kate has been really supportive and understanding of my not-infrequent impulses to just nerd out and hack a thing. She probably won't read this (though maybe?), but it's important to acknowledge that many of us hackers are only able to do our work because of the support that we get from our families.

**a digression on the nature of seeking and knowledge**

I am jealous of my colleagues in academia sometimes; of course it must be this way, that we are jealous of each other. Greener grass and all that. But when you go through a doctoral program, you know that you [push the boundaries of human knowledge](http://matt.might.net/articles/phd-school-in-pictures/). You know because you are acutely aware of the state of recorded knowledge in your field, and you know that your work expands that record. If you stay in academia, you use your honed skills to continue chipping away at the unknown. The papers that this process reifies have a huge impact on the flow of knowledge in the world. As just one example, I've read all of Dybvig's papers, with delight and pleasure and avarice and jealousy, and learned loads from them. (Incidentally, I am given to understand that all of these are proper academic reactions :)

But in my work on Guile I don't actually know that I've expanded knowledge in any way. I don't actually know that anything I did is new and suspect that nothing is. Maybe CPS soup? There have been some similar publications in the last couple years but you never know. Maybe some of the multicore Concurrent ML stuff I haven't written about yet. Really not sure. I am starting to see papers these days that are similar to what I do and I have the feeling that they have a bit more impact than my work because of their medium, and I wonder if I could be putting my work in a more useful form, or orienting it in a more newness-oriented way.

I also don't know how important new knowledge is. Simply being able to practice language implementation at a state-of-the-art level is a valuable skill in itself, and releasing a quality, stable free-software language implementation is valuable to the world. So it's not like I'm negative on where I'm at, but I do feel wonderful talking with folks at academic conferences and wonder how to pull some more of that into my life.

In the meantime, I feel like (my part of) Guile 2.2 is my master work in a way -- a savepoint in my hack career. It's fine work; see [A Virtual Machine for Guile](https://www.gnu.org/software/guile/manual/html_node/A-Virtual-Machine-for-Guile.html#A-Virtual-Machine-for-Guile) and [Continuation-Passing Style](https://www.gnu.org/software/guile/manual/html_node/Continuation_002dPassing-Style.html#Continuation_002dPassing-Style) for some high level documentation, or many of these bloggies for the nitties and the gritties. OKitties!

**getting the goods**

It's been a joy over the last two or three years to see the growth of [Guix](https://www.gnu.org/software/guix/), a packaging system written in Guile and inspired by [GNU stow](https://www.gnu.org/software/stow/) and [Nix](http://nixos.org/nix/). The laptop I'm writing this on runs GuixSD, and Guix is up to some 5000 packages at this point.

I've always wondered what the right solution for packaging Guile and Guile modules was. At one point I thought that [we would have a Guile-specific packaging system](https://wingolog.org/archives/2011/07/04/guile-2-0-2-building-the-guildhall), but one with stow-like characteristics. We had problems with C extensions though: how do you build one? Where do you get the compilers? Where do you get the libraries?

Guix solves this in a comprehensive way. From the four or five bootstrap binaries, Guix can download and build the world from source, for any of its supported architectures. The result is a farm of weirdly-named files in `/gnu/store`, but the transitive closure of a store item works on *any* distribution of that architecture.

This state of affairs was clear from the Guix [binary installation instructions](https://www.gnu.org/software/guix/manual/html_node/Binary-Installation.html) that just have you extract a tarball over your current distro, regardless of what's there. The process of building this weird tarball was always a bit ad-hoc though, geared to Guix's installation needs.

It turns out that we can use the same strategy to distribute reproducible binaries for any package that Guix includes. So if you download [this tarball](https://ftp.gnu.org/gnu/guile/guile-2.2.0-pack-x86_64-linux-gnu.tar.lz), and extract it as root in `/`, then it will extract some paths in `/gnu/store` and also add a `/opt/guile-2.2.0`. Run Guile as `/opt/guile-2.2.0/bin/guile` and you have Guile 2.2, before any of your friends! That pack was made using `guix pack -C lzip -S /opt/guile-2.2.0=/ guile-next glibc-utf8-locales`, at Guix git revision `80a725726d3b3a62c69c9f80d35a898dcea8ad90`.

(If you run that Guile, it will complain about not being able to install the locale. Guix, like Scheme, is generally a statically scoped system; but locales are dynamically scoped. That is to say, you have to set `GUIX_LOCPATH=/opt/guile-2.2.0/lib/locale` in the environment, for locales to work. See the [GUIX\_LOCPATH docs](https://www.gnu.org/software/guix/manual/html_node/Application-Setup.html#Application-Setup) for the gnarlies.)

Alternately of course you can install Guix and just `guix package -i guile-next`. Guix itself will migrate to 2.2 over the next week or so.

Welp, that's all for this evening. I'll be relieved to push the release tag and announcements tomorrow. In the meantime, happy hacking, and yes: this blog is served by Guile 2.2! :)

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [a lambda is not (necessarily) a closure](/archives/2016/02/08/a-lambda-is-not-necessarily-a-closure)

### 14 responses

1. rain1 says:[15 March 2017 11:19 PM](#80de1f97e046b37789421577be9eea06dac4109c)

   Congratulations! Thank you for keeping the wonderful scheme language alive and well! The guile compiler is a fantastic tool both for the practical getting work done and the theoretical - learning scheme/programming/compiler implementation details.

2. [John Cowan](http://vrici.lojban.org/~cowan) says:[16 March 2017 0:27 AM](#c12fdadb390b117d48c442a44b7a32b0d13446e8)

   This is good news, because I am about to reinstall my large suite of Schemes on my home system using Ubuntu on Windows, so I'll hold off on installing Guile. I do hope I don't have to wait hours, like I did the last time I installed Guile from source.

3. [Julian Graham](http://undecidable.net/joolean) says:[16 March 2017 2:07 AM](#76e3e3e274d1b2a6175c0569ebe5f41a76e31615)

   Andy, you rock! Thanks so much for all your hard work on this. Can't wait to start using the official release.

4. [Javier Sancho](http://jsancho.org/) says:[16 March 2017 6:57 AM](#75ea6a589182d33ade37ec6715f88831004e7704)

   Thank you so much, Andy! Your work and your posts are undoubtedly taking us to the top.

5. gasche says:[16 March 2017 7:02 AM](#5e14491321e8b5cf2e4c0bb4db9d3e8881bbf1bd)

   I think you should seriously consider writing an academic article about Guile, as a way indeed to present and transfer your knowledge and design choices for the system. For example, the deadline for article submission for the next "Trends of Functional Programming" symposium is on May 5 (the conference is June 19-22 in Canterbury, UK), and looking at last year's programme ( http://www.tifp.org/ ) could give you an idea of what the conference feels like -- it is more open to implementation-oriented works than some other conferences, which is why I thought of it as a reasonable first step. Surely you could enlist the help of some colleagues that were recently in academia or still there for the "how do I structure/write a damn paper?" and "how do I get feedback on my draft?" parts.

6. gasche says:[16 March 2017 7:03 AM](#d82b348f6a087d0c990b6d3971bfa61161412ab3)

   Sigh, frames; http://tfp2016.org/accepted.html

7. [Charles](http://charles.stanho.pe) says:[16 March 2017 1:24 PM](#53e38fcfbf4b173baaac686259edf195fa93030d)

   Congratulations on 2.2! It's been a pleasure to watch Guile improve under your watch and see the new applications it has found as a result. I look forward to trying out the latest. Also, thank you for all the informative and edifying blog posts on language design and implementation.

8. [amz3](http://hyperdev.fr) says:[16 March 2017 3:28 PM](#eda5c54e3bb75b99f09a4b472c1622a591980824)

   Thanks for this awesome release. I am not not a compiler/VM dev just a user but AFAIK Guile offers what's required to build software. It's seems to me Guile 2.2 is similar in terms of features compared to CPython 3.6 except that Guile doesn't have a GIL.

   Guile only lakes more users :)

9. RobertX says:[16 March 2017 5:57 PM](#f6a63cf5f1db800e1cda1d03d254d6c365a38b39)

   How does Windows fit into this? Is it even a target?

10. Johnny says:[19 March 2017 7:47 AM](#969eaabf2564eae655aeba96b641c31d88f82b00)

    I just looked through ecraven's benchmarks. Sure they are benchmarks, and they onlt show a part of the truth, but consistently being AT LEAST 2x faster must mean something :)

    Well done!

11. Adam says:[22 March 2017 3:18 PM](#3c388074794764cb02a10a8e374e59e06bcd5b03)

    When you talk about "the multicore Concurrent ML stuff" that you have yet to write about, is that stuff that is publicly available in your fibers repoo, or something else? In the ML world, this has been the subject of ongoing research but I'm curious what you've done.

12. [wingo](http://wingolog.org/) says:[28 March 2017 11:28 AM](#efd50429c893e476ea286a19ea1a8de217af0129)

    Heya Adam,

    I understand that Manticore is a working multicore CML system. The work I have is in Fibers and is mostly in [operations.scm](https://github.com/wingo/fibers/blob/master/fibers/operations.scm), and pretty much follows Manticore. See also channels.scm in that same directory, and internal.scm for the scheduler implementation. In Fibers we coalesce the "poll" and "do" phases into a single "try" phase; otherwise it's pretty much the same as Manticore, though the channel protocol is atomic (no spinlocks). I need to do some more benchmarking and will blog about that sometime soon :)

    Andy

13. Adam says:[2 April 2017 10:01 PM](#fdda1ce1b995957b499ec2280cd1c4ebd60753fb)

    Andy - Thanks! Really interesting stuff, hope you blog about it soon. I'm hoping more people learn about CML since it solves a lot of the problems I have as a UI programmer.

14. mats says:[27 April 2017 7:52 AM](#c009477f067b74f1818fe36b05f88f6b69b00fc7)

    I looked through the benchmarks by ecraven, and the speedup is incredible! Ported a bunch of python things (project euler solutions) to guile2.2 and had an instant 3-10x speedup.

Comments are closed.
