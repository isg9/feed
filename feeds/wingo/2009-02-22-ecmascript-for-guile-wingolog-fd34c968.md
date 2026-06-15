---
title: ecmascript for guile — wingolog
url: https://wingolog.org/archives/2009/02/22/ecmascript-for-guile
published: "2009-02-22T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/02/22/ecmascript-for-guile
---

# ecmascript for guile — wingolog

## [ecmascript for guile](/archives/2009/02/22/ecmascript-for-guile)

22 February 2009 4:45 PM

- [ecmascript](/tags/ecmascript)
- [guile](/tags/guile)
- [compiler](/tags/compiler)
- [goops](/tags/goops)
- [vm](/tags/vm)

Ladies, gentlemen: behold, an [ECMAScript](http://en.wikipedia.org/wiki/ECMAScript) compiler for Guile!

```
$ guile
scheme@(guile-user)> ,language ecmascript
Guile ECMAScript interpreter 3.0 on Guile 1.9.0
Copyright (C) 2001-2008 Free Software Foundation, Inc.

Enter `,help' for help.
ecmascript@(guile-user)> 42 + " good times!";
$1 = "42 good times!"
ecmascript@(guile-user)> [0,1,2,3,4,5].length * 7;
$2 = 42
ecmascript@(guile-user)> var zoink = {
                           qux: 12,
                           frobate: function (x) {
                              return random(x * 2.0) * this.qux;
                           }
                         };
ecmascript@(guile-user)> zoink.frobate("4.2")
$3 = 37.3717848761822

```

The REPL above parses ECMAScript expressions from the current input port, compiling them for Guile's virtual machine, then runs them -- just like any other Guile program.

Above you can see some of the elements of ECMAScript in action. The compiler implements most of ECMAScript 3.0, and with another few days' effort should implement the whole thing. (It's easier to implement a specification than to document differences to what people expect.)

The "frobate" example also shows integration with Guile -- the `random` function comes from current module, which is helpfully printed out in the prompt, `(guile-user)` above.

In addition, we can import other Guile modules as JavaScript, oops, I mean ECMAScript objects:

```
ecmascript@(guile-user)> require ('cairo');
$1 = #< b7192810>
ecmascript@(guile-user)> $1['cairo-version']();
$2 = 10800

```

I could automatically rename everything to names that are valid ES identifiers, but I figured that it's less confusing just to leave them as they are, and require `['%strange-names!']` to be accessed in brackets. Of course if the user wants, she can just rename them herself.

Neat hack, eh?

**what the hell?**

I realize that my readers might have a number of questions, especially those that have other things to do than to obsessively refresh my weblog. Well, since I had myself at my disposal, I decided to put some of these questions to me.

*So, Andy, why did you implement this?*

Well, I've been hacking on a compiler for Guile for the last year or so, and realized at some point that the [compiler tower](http://article.gmane.org/gmane.lisp.guile.devel/7984) I had implemented gave me multi-language support for free.

But obviously I couldn't claim that Guile supported multiple languages without actually implementing another language, so that's what I did.

I chose ECMAScript because it's a relatively clean language, and one that doesn't have too large of a standard library -- because implementing standard libraries is a bit of a drag. Even this one isn't complete yet.

*How does it perform? Is it as fast as those [arachno](http://www.mozilla.org/js/spidermonkey/)- [fish](http://trac.webkit.org/wiki/SquirrelFish) implementations I keep hearing about?*

It's tough to tell, but it seems to be good enough. It's probably not as fast as compilers that produce native code, but because it hooks into Guile's compiler at a high level, as Guile's compiler improves and eventually gets native code compilation, it will be plenty fast. For now it feels snappy.

There is another way in which it feels much faster though, and that's development time -- having a real REPL with readline, a big set of library functions (Guile's), and fast compilation all make it seem like you're talking directly with the demon on the other side of the cursor.

*It actually implements ECMAScript? And what about ES4?*

ES3 is the current goal, though there are some bits that are lacking -- unimplemented parts of the core libraries, mainly. Probably there are some semantic differences as well, but those are considered bugs, not features. I'm just one man, except in interviews!

Well, there is one difference: how could you deny the full numeric tower to a language?

```
ecmascript@(guile-user)> 2 / 3 - 1 / 6;
$3 = 1/2

```

And regarding [future standards](http://en.wikipedia.org/wiki/ECMAScript#Future_development) of ECMAScript, who knows what will happen. ES4 looks like a much larger language. Still, Guile is well-positioned to implement it -- we already have a powerful object system with multimethod support, and a compiler and runtime written in a high-level language, which count for a lot.

*Awesome! So I can run my jQuery comet stuff on Guile!!1!!*

You certainly could, in theory -- if you implemented XMLHttpRequest and the DOM and all the other things that JavaScript-in-a-web-browser implements. But that's not what I'm interested in, so you won't get that implementation from me!

*Where do you see this going?*

I see this compiler leading to me publishing a paper at some conference!

More seriously, I think it will have several effects. One will be that users of applications with Guile extensions will now be able to extend their applications in ECMAScript in addition to Scheme. Many more people know ECMAScript than Scheme, so this is a good thing.

Also, developers that want to support ES extensions don't have to abandon Scheme to do so. There are actually many people like this, who prefer Scheme, but who have some users that prefer ECMAScript. I'm a lover, not a fighter.

The compiler will also be an example for upcoming Elisp support. I think that Guile is the best chance we have at modernizing Emacs -- we can compile Elisp to Guile's VM, write some C shims so that all existing C code works, then we replace the Elisp engine with Guile. That would bring Scheme *and* ECMAScript *and* whatever other languages are implemented to be peers of Elisp -- and provide better typing, macros, modules, etc to Emacs. Everyone wins.

*So how do I give this thing a spin?*

Well, it's on a branch at the moment. Either you wait the 3 or 6 months for a Guile 2.0 release, or you check it out from git:

```
git clone git://git.sv.gnu.org/guile.git guile
cd guile
git fetch
git checkout -b vm origin/vm
./autogen.sh && ./configure && make
./pre-inst-guile

```

Once you're in Guile, type `,language ecmascript` to switch languages. This will be better integrated in the future.

*Why haven't you answered my mail?*

Mom, I've been hacking on a compiler! I'll call tonight ;-)

## related articles

- [visualizing statistical profiles with chartprof](/archives/2009/02/09/visualizing-statistical-profiles-with-chartprof)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [on generators](/archives/2013/02/25/on-generators)
- [inline cache applications in scheme](/archives/2012/05/29/inline-cache-applications-in-scheme)
- [eval, that spectral hound](/archives/2012/02/01/eval-that-spectral-hound)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)

### 13 responses

1. Thomas says:[22 February 2009 5:10 PM](#206935e42ea946d21f1350f1ea0a287abdacefb3)

   Regardless of the performance of the implementation, this wins easily as best performance for the announcement! Congratulations.

2. [djcb](http://emacs-fu.blogspot.com) says:[22 February 2009 6:58 PM](#7ca15ce17e4bae3d544a6a701e83cb723d46195e)

   Congrats Andy - this is great news. It seems the grand visions for Guile may come true after all.

3. Zeeshan Ali (Khattak) says:[22 February 2009 7:10 PM](#22177fb1af58519d09cab8bf4350d72d2a979376)

   Doing great cool stuff, that is Andy Wingo. :)

4. [fraggle](http://fragglet.livejournal.com/) says:[22 February 2009 7:23 PM](#f5920f9bc5a5cc7a99c131845c4a3e99f2aa329e)

   Nice work, but hasn't this already been done? I'm sure I already saw a JS -> Guile compiler a few years ago. Or maybe it was Python. Hmm.

5. [Dodji](http://www.seketeli.org/dodji) says:[22 February 2009 7:41 PM](#7f6dd31e2e06eb7d8b0c80436554e043f4740ff2)

   Oh boy, you are crazy. And I like that :)

   Keep up your rockarific crazyness :)

6. Thomas Kappler says:[22 February 2009 8:05 PM](#b65f5b9a2862817005dc8341a63c63472da3ff6a)

   You're my favorite rockstar hacker. Emacs extensible in Scheme and Javascript? Heaven.

7. Miguel de Icaza says:[23 February 2009 5:13 AM](#3fbb2a2a12b4fd658e7c8459f1292441933254a8)

   Beautiful hack!

8. [Andrea Russo](http://www.salug.it) says:[23 February 2009 6:42 PM](#389d1cc1160756f71fbd5f36d80197a0cb0582cd)

   Fantastic. This is an exciting news. Thanks Andy for your great work!

9. [Ian McKellar](http://ian.mckellar.org/) says:[23 February 2009 7:24 PM](#60674cf0b551a355b96a136267622bccadb0b0d7)

   I love this awesome cracksmoke! That's really really neat.

10. Thomas says:[14 March 2009 8:48 PM](#7801ae68871ad68f6b917fe30371c45812e1485e)

    Wow, this is excellent in just so many ways that I don't know how to fit them into one short comment!

11. [Julian Graham](http://undecidable.net/joolean/) says:[27 March 2009 11:40 PM](#105665b61e4aa9ef0bac15e020caf4910488c0df)

    You know, theoretically, Guile \*does\* have a DOM implementation.

12. TheGZeus says:[19 April 2010 11:59 AM](#1d821ffbbab0b2f8aa4344409562f9ac40cdcc1c)

    Neat. Very cool, in fact. The Emacs Lisp thing looks particularly awesome.

    The existing code would probably end up in one thread(and obviously one namespace) unless converted, I assume?

13. Mom says:[6 September 2012 2:01 AM](#e25e9e4fb44602f9ce97b1a7a7040ebf79ea4bed)

    Andrew, why haven't you called yet?

Comments are closed.
