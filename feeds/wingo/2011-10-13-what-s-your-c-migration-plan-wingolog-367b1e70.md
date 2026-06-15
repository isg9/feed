---
title: what's your C migration plan? — wingolog
url: https://wingolog.org/archives/2011/10/13/whats-your-c-migration-plan
published: "2011-10-13T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2011/10/13/whats-your-c-migration-plan
---

# what's your C migration plan? — wingolog

## [what's your C migration plan?](/archives/2011/10/13/whats-your-c-migration-plan)

13 October 2011 8:44 PM

- [c](/tags/c)
- [compilers](/tags/compilers)
- [scheme](/tags/scheme)
- [gnu](/tags/gnu)

I spend a lot of time in my neighborhood café. It has character. Some of it is generic: like in many other bars, the round tables are marble, the chairs are rickety, the customers and the barmen all shout at each other.

But it has its own thing. The owner inherited it from his father, but he was never sure whether he wanted a record store or a bar, so it's half and half. In practice the CDs in the changer don't change very often, and the rest are all in a side room that is otherwise packed full of junk.

In the morning, everyone gets *[tallats](http://ca.wikipedia.org/wiki/Cafè_tallat)*. Old ladies get theirs with pastries. Old men get theirs *amb gotes*, with liquor. Morning workers rush in, slam one back, slap coins on the counter and get out. *Adéu!*

Those with more time, like myself, get a *cafè amb llet*. Most read the paper. I hack. I spend a couple mornings a week there. It's right pleasant to [work](http://www.theatlantic.com/business/archive/2011/04/working-best-at-coffee-shops/237372/) there, without internet. My most productive hours of the week are there in the bar.

I do chitchat a bit, though: with designers, mothers, the barmen, retirees, random folk. There's one fellow in particular I like talking with, a designer. Turns out he wants to learn how to program. He told me yesterday that he wanted to learn [C](http://en.wikipedia.org/wiki/C_(programming_language)).

**on c**

Now, I spend a lot of time in C. I've written loads of it. I continue to write it at times. I'm fond of it, and it has served me well.

But I have come to believe that writing new programs in C is the hacker equivalent of malpractice. C is just too dangerous to use. The risks are not worth the benefits.

Let's be clear about the benefits of writing in C, before looking at its flaws. I want to be really fair here. C is fast. It has great compilers, everywhere. And they are everywhere: C is ubiquitous. It is very flexible, also. You can do anything in C. It is great for programming drivers. It is possible to write big e-mail clients in it. It has great low-level bit operations (though C++ is getting better at it, with its value types). It is a power tool, and it puts you in control. There are loads of libraries written in it, and for it. It encourages minimalism.

In short, C has been a wild success, and with good reason.

But it's time to stop using it.

The essence of the problem with C is that the behavior of erroneous programs is not specified. This doesn't sound like a big deal, but it is. Let's make an example. What is the meaning of this Python program?

```
def array_set(a, n, x):
  a[n] = x

```

I think we would all say that it sets the *n* th element of *a* to *x*. But what about this one in C:

```
void array_set (void **a, size_t n, void *x) {
  a[n] = x;
}

```

This one, you really can't tell. You can't use Python's definition, because there are no guarantees about the valid indices to set in the array. In fact there is no array, just a pointer.

In the Python case, if you pass an out-of-bounds array index, you will get an exception. In the C case, you get undefined behavior. Lately what "undefined behavior" means is that the state or organized crime gets to take control of your computer: read your email, log your keystrokes, take pictures of you.

I'm deadly serious. Let's take a better example. Last year [Liu Xiaobo](http://en.wikipedia.org/wiki/Liu_Xiaobo) won the Nobel Peace Prize for "his long and non-violent struggle for fundamental human rights in China." A couple weeks later, it was found that someone [cracked the Nobel Prize website](http://krebsonsecurity.com/2010/10/nobel-peace-prize-site-serves-firefox-0day/) and was serving a Firefox zero-day that took over users' machines.

Apparently, *someone* was interested in what was on the computers of visitors to the Nobel prize site -- interested enough to use a fresh zero-day, something that probably sells on the black market for some $20,000 or more.

Lately I've been quoting Eben Moglen a lot, especially from the talk he gave at this year's FOSDEM conference. One thing he said there is that "software is the steel of the 21st century". As steel shaped the social relations of the last century, what we hack now forms the material conditions of tomorrow: the next Arab Spring, or the next [Green Scare](http://en.wikipedia.org/wiki/Green_Scare).

Almost all software is connected to the net these days, and so it is all under attack. If you write software in C these days -- software with bugs, like any software -- you are writing software with undefined behavior, and thus, software that enables powerful state and organized crime actors to take advantage of your users.

That sounds a bit exaggerated, no? It is true, though. Look at what is happening with browser vulnerabilities, or what is the same, PDF or Flash vulnerabilities, or PNG or MP4, or what-have-you. The commonality here is that powerful interests exploit unsuspecting users due to flaws in the C and C++ language. Writing more C these days is malpractice.

**migration**

I still write C. I work on implementations of safe languages -- languages that don't have the same kinds of fundamental vulnerabilities that C and C++ have. Eventually the amount of C in the world will stop growing, and decline as pieces that are now written in C will be written in Python, in JavaScript, in Guile: in short, in languages that don't launch the missiles when you try to write beyond the end of an array.

C has had a great run; we should celebrate it. But its time has passed. What is your migration strategy? How are you going to stop writing C?

## related articles

- [the half strap: self-hosting and guile](/archives/2016/01/11/the-half-strap-self-hosting-and-guile)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a baseline compiler for guile](/archives/2020/06/03/a-baseline-compiler-for-guile)
- [a new concurrent ml](/archives/2017/06/29/a-new-concurrent-ml)
- [growing fibers](/archives/2017/06/27/growing-fibers)
- [guile 2.2 omg!!!](/archives/2017/03/15/guile-2-2-omg)

### 44 responses

1. [Danilo](http://danilo.segan.org/) says:[13 October 2011 9:05 PM](#7667c1be0052d42151434790a6b6df8e9e74b56a)

   I am not planning to stop writing C. I actually want to write more of it. Probably not as much as Python, but there is a rightful, irreplaceable spot for C, both in practice and in my heart. ;)

   And try as they might, "safe" languages like the ones you list above have had their own share of vulnerabilities: it's not languages that are vulnerable, it's the code that is.

   PS. Typing out a number like "seven" was not accepted by the blog submission form (oh, I did it on purpose ;).

2. Pieter says:[13 October 2011 9:09 PM](#93347ee0d375a22f1b5546d2f882eda0b3d3657c)

   Reading this post and "But its time has passed" in particular on the day that Dennis Ritchie has passed away makes me wonder how appropriate your post is. Out of respect for Dennis Ritchie I would have waited a week or so before posting this.

3. David says:[13 October 2011 9:44 PM](#a0b505087d495c8687f8fd14ab4d2758484d6b06)

   There is a growing use of C in the deeply embedded world. Instead of a custom ASIC with large amounts of dedicated Verilog/VHDL that would require a chip spin to fix a bug, more companies are trying to move all parts of the design that are not clock cycle critical into software. I've worked on a number of chips which contain a deeply embedded CPU with only a few kilobytes of RAM available to it. There is no chance any language other than C will be used on these systems.

   Sometimes a custom ASIC may include a dozen processors, all with their own dedicated memory space, and may all be different. IE 1 CPU may be a 25 000 gate simple 5-stage CPU, and another might be a faster 250 000 gate 12-stage CPU, depending on the particular need of any block within the ASIC.

4. Patrick says:[13 October 2011 9:46 PM](#2713794b1e3e2b7e7ca58c0ad60fa9fc1d87fdc6)

   I see Go as the clear natural successor to C, in no small part because two of its creators were long-time colleagues of Dennis Ritchie at Bell Labs -- Ken Thompson and Rob Pike. Among its features, Go provides garbage collection, type safety and memory safety.

5. [Edd](http://eddporter.com) says:[13 October 2011 9:49 PM](#7daeedca4f5685cbff7cf3fa94398f2015da1c60)

   So what is your suggestion for the language that we migrate to from C? Sure, there are loads of new and interesting higher-level languages that all have their places, but most are interpreted or are run in a VM.

   Don't get me wrong. I too have felt that perhaps C has had its time, but there seems to be nothing to replace it. It was a language designed to write an OS and it is still practically the only one that can (coupled with assembler, but that is even more dangerous). Everyone goes on about its "undefined behaviour", but show me a language \_at the same level\_ that doesn't. Python and JavaScript have their places, but they are not in the same league. If I wanted to write an OS tomorrow, what language should I use? It has to be C, surely?

6. brad says:[13 October 2011 9:51 PM](#c6682e07f14b57e4099d132f212499f03ff931bc)

   Go has many characteristics that make it appealing for people used to working with c, but i also use it to write many programs i would otherwise write in perl. Go isn't a radical departure, but because much of it seems (superficially) familiar, it can be sold as a migration path

7. Tomás Oliveira says:[13 October 2011 10:01 PM](#37a87004bfa24e0102929c24ce7f156a39a5fbee)

   Nope. C is king where performance is needed. In the case that C indeed falls out, probably going to Go.

8. programmerX says:[13 October 2011 10:01 PM](#ac08e5ef8b96e70f5a11d019387639b15b8f61b9)

   Vala for Gnome programming, Go for network and cloud programming. Javascript for client programming.

9. [Aleix](http://proli.net) says:[13 October 2011 10:02 PM](#92242a6b0d6f8f98599909801860be3a5828ceb3)

   I don't think there's much reason to move C projects to other languages. C++ does have a quite smooth porting flow, but you'll get stuck with weird constructions.

   We'll keep having C around for a while, like we have had tallats... forever. :)

10. [Roman Elizarov](http://www.devexperts.com) says:[13 October 2011 10:11 PM](#9e76272a9dd954448f97f69a3f3ba9b383fc0aeb)

    Java serves us well as a C and C++ replacement. With modern JVM you get C-like performance (unless you do a lof of floating point, that is). We process and forward over the network millions of option quotes per second on a commondity 1U box with ~10% CPU usage with 100% Java code. We could have written it all in C, but doing so we would have only wasted more developement effort writing and debugging it, without getting any noticeable performance benefits (quote processing is all integer stuff).

11. [Natan Yellin](http://natanyellin.com) says:[13 October 2011 10:33 PM](#7a3c99d684005c6e52ba86b259de0222761b0e9f)

    Java is \*not\* a C/C++ replacement. Runtime performance is often comparable but memory consumption is not. That is partly due to the JVM's overhead, but the primary factor is that garbage collecting will always be less efficient than explicit memory management done right. Period.

    It's ironic that you suggest replacing C/C++ with the JVM given that the JVM itself is written mostly in C.

12. Fred Fish says:[13 October 2011 11:47 PM](#29428fa78248f1dda4e030b4aeacccc88dfc81ae)

    Like any programming language it requires a discipline from the programmer. Your example is flawed, designed intentionally to obscure. If you don't use void\* and use typed variables, the compiler will pick up errors. If you don't check conditions on entry and exit, you run the risk of overrun. If you don't teach programmers to manage memory allocations and deallocations they cannot appreciate memory leaks. C is important so we don't end up with billions of bloated and repetitive class libraries that come with the language.

13. Dave says:[14 October 2011 2:59 AM](#2338d179793e09fc53f27f9aad21e006705006ba)

    Have you looked at lua? www.lua.org

    It's fast and light and embeds easily with C. You get the best of both worlds imho.

14. Sindisil says:[14 October 2011 4:16 AM](#b2c5a69a159efe803ebd7e24a63d380ddc6f93d4)

    My "C Migration Plan" is to use more and more C every day.

    I've transitioned several projects that were once Java or C++ based, or multi-language, over to C or multi-language with C as the primary language.

    I intend to continue to do so.

    I seriously doubt that I will ever stop writing C, at least not until I stop writing all together.

15. [Ramakrishnan Muthukrishnan](http://rkrishnan.org) says:[14 October 2011 5:03 AM](#366bfeaffa03fd1d18d62de8e5e1ade9123ab00b)

    I would rather not migrate. That's because I don't have a choice for the tasks I do with C. And that will be my general strategy towards any programming task - choose the right language for a task rather than having a blanket plan which encompass everything.

16. Mathias Hasselmann says:[14 October 2011 6:01 AM](#2358e00666ad668728e439325f084190b1c02c38)

    Problem with C less is the language itself, but bad coding practice. For instance libpng is a horrible piece of \*\*\*\* full of longjmp and so on. Used with some reasonable framework like glib or even all the gobject stuff, that damn old language shows its beauty, by being easy to grok and only few mistakes to make. Hey, actually with that freaking gobject stuff(1) you even get two-phase construction and destruction, some feature all serious language designers but the C++ gurus understood to be important. Actually when talking about that pet problem of mine, I honestly hope people would stop using that marvelous C++ monster first.

    After all I'd expect C to stay around for quite some while, after all you need a language for writing kernels and virtual machines: You don't want to use assembler for that.

    (1) Yes, those horribly long method method names are a high price to pay for getting a reasonable OOP framework. Learn Vala (instead of C++) if that method names scare you.

17. Mukund says:[14 October 2011 8:40 AM](#2f4b6a392eae5a3d3a0697029c71f80b36dfb57e)

    I don't follow what the point of your post is. Is it to promote an alternative language, or to call out C programmers as evil?

    Those who use C know what it's strengths and weaknesses are, and how to use it effectively. Calling it malpractice is trolling. Surely you do it to be controversial, expecting a response for it?

    Are you going to ask the English speaking world to stop speaking English because it has many fundamental issues? Would it be malpractice?

    I took your posts about Guile pretty seriously, but I wonder why you have written this post. Sure this is your blog and you can say whatever. But ranting simply is no good.

18. [Andy Wingo](http://wingolog.org/) says:[14 October 2011 8:59 AM](#225f45a2ce84dd27675dc7d5c1ad375e53e2a0b1)

    Hi Mukund. I must have not expressed myself clearly enough.

    What I'm saying is that C has benefits, and C has risks. Those of us that use it usually ignore the risks. If we actually look at the expected effects of writing network-facing software in C, in many cases the risks outweigh the benefits.

    To put it another way, in university I studied nuclear engineering. That's inherently dangerous stuff. The reason there have been so few accidents (relatively) is because engineers looked at all of the ways that the systems could fail, assessed their risk, assessed their impact, and mitigated the issues. (I still consider it to be too dangerous, but that is another point.)

    Manual memory management and writing beyond the bounds of an array are two common failure modes of C programs. They have potentially enormous impact. Wouldn't it be nice to completely rid yourself of these problems? Wouldn't it be irresponsible not to even consider it?

19. [Danilo](http://danilo.segan.org/) says:[14 October 2011 9:16 AM](#429effe8653982283319fda38bb08a294f262d17)

    Andy, you are right, these are some of the problems that are too easy to introduce with C. But, when you are doing network programming, you are dealing with untrusted input (well, not just with network programming), and you are going to have problems with it no matter the language.

    XSSs and SQL injection are common problems these days (relative paths accepted from untrusted sources as well, though perhaps not so much anymore) but if those problems hit us, it's not a language to blame, it's the programmer. Sure, language can make some things easier and other things harder, but it ultimately dives down to people writing crappy code.

    Yes, I agree majority of today's (and tomorrow's) programmers will not be able to understand the intricacies of memory management (for better or for worse), but just as much as they won't be able to understand/anticipate SQL injection bugs either.

    That should not, in my (not so) humble opinion, lead to a conclusion of "let's drop C". Because when applied generically, it'll mean we should drop all the languages :)

20. [Andy Wingo](http://wingolog.org/) says:[14 October 2011 9:26 AM](#c3774f9c997d5e7c9a3274dc3f46ab98a17159c8)

    Thanks for stopping by, Danilo. I agree with you, broadly speaking; there are still programs now that have to be written in C. But still, we need a plan, about how to get to a point where we are causing less damage to our users.

    But, regarding the particular examples of XSS and SQL injection, [programming languages can help here!](http://www.gnu.org/software/guile/manual/html_node/Types-and-the-Web.html) Proper use of programming language technologies can completely eradicate entire classes of bugs.

21. Pavel says:[14 October 2011 10:25 AM](#3243b27e4932f65e1194e4a46e0d83f2f6d20212)

    my migration plan for c? adding a -fno-undefined-behaviour to GCC. Sure OOB checking will make code slower, but we wont have to recode everything. Besides garbage collection is already coming to C++. (N2670)

22. Mukund says:[14 October 2011 11:16 AM](#b0029d0179fc08b6c5395e34b006c84a61b904ae)

    Hi Andy

    No offense taken. :) I think the quality of your posts is good enough when you describe how managed or functional languages are good. Clever programmers will investigate, and stupid programmers will stay stupid. If you look at recent job opportunities (GitHub, Stack Overflow, etc.), most of them \_are\_ in managed languages. That's what the majority of the world seems to be using now. C has its place and those who use it want to use it.

    Btw, regarding your nuclear comment, you're spot on, except.. C is the nuclear reactor, and good programming practice, traps, Valgrind and static analysis are the ways to mitigate the issue. The other languages are like 'clean energy'.. new and relatively expensive but popular nevertheless. :)

    But it's all nuclear anyway.

23. rixed says:[14 October 2011 12:02 PM](#e39206991b14bd742daa5a1006d79f8ae9797d25)

    "Manual memory management and writing beyond the bounds of an array are two common failure modes of C programs." :

    I'm writing C programs for a living for many years and in my own limited experience this statement is just plain wrong. I wish it were true, though. How easy debugging would be!

    To me, bugs are first and foremost due to holes in specifications, and no language (that I know of) can help with that. Second, bugs are due to race conditions. Again, I don't know of a language that helps with that (I'm speaking about languages that does indeed provide parallel execution).

    I wouldn't say the problem with C is that it's unsafe, but that it does not allow abstraction, that it's verbose, etc.

24. userulluipeste says:[14 October 2011 12:53 PM](#33d62ab36703e6dfd8bc87654c14f32bb23c8c41)

    Like Fred Fish said, your example is flawed. If mingling with pointers is too much, you may create data structures for that (that is what that array in Python is). You may "migrate" to C++ with STL for your soul peace!

25. [Andy Wingo](http://wingolog.org/) says:[14 October 2011 1:41 PM](#01c3492f4b13197a9ea817426521f478b0bcc655)

    Userulluipeste, std::vector is just as unsafe as C arrays, in this regard.

    The problem is that C's standard data structure is simply an array 2^32 or 2^64 entries wide and 8 bits deep! (Indeed, check the `HEAP` array residualized by programs compiled from C to JS by Emscripten...)

26. John says:[14 October 2011 2:13 PM](#d585d8cd2a5f66f4dbd2ae1a7d9212e15a8f2814)

    What are some alternatives to C for writing things like language implementations, drivers, and operating systems?

27. Don says:[14 October 2011 3:14 PM](#5f4ff031753e60e3d2f60c927e248052390de7da)

    Not sure I agree with the article, but I will agree with those suggesting Go. I've been using it for some side projects and one production project at work, and I love it.

    http://golang.org/

28. Guess Who says:[14 October 2011 3:29 PM](#0c518f0af7f30464176bd489b6ae6291c979ffb7)

    \> What is your migration strategy? How are you going to stop writing C?

    Vala

29. xavi conde says:[14 October 2011 3:31 PM](#bc88de334e0ca6c6fa97c5c0c22d2842702d9eae)

    You said:

    "

    void array\_set (void \*\*a, size\_t n, void \*x) {

    a\[n\] = x;

    }

    This one, you really can't tell. You can't use Python's definition, because there are no guarantees about the valid indices to set in the array. In fact there is no array, just a pointer."

    Well, what actually does it what it was designed to do, assign the value x (n\*width of type a) times far from a. And it allows to do so because this array construction is useful for direct access to memory, which some software does need. So this code is perfectly ok, and you couldn't do it with Python (I guess).

    So C is yet better than python. Ha!

30. jk says:[14 October 2011 4:48 PM](#ef809cbcffe4a1697c5fa5a001ad5a1445036a69)

    @Andy: "Userulluipeste, std::vector is just as unsafe as C arrays, in this regard."

    Huh? Can you elaborate? std::vector is completely safe as long as you use begin/end/size etc. If you're really concerned about safe indexing you use at().

31. Conrad Steenberg says:[14 October 2011 5:11 PM](#c90096b24c2bf8cd2e902ea8dcaf594bcf1911dc)

    My migration strategy is to stop using both Python and C, and help out development of the Crack language :-)

    http://code.google.com/p/crack-language/

32. ecc says:[14 October 2011 5:48 PM](#3894481f8e6d2f88ead04541a5ee4b1a1b279b1b)

    Your Python-vs-C example actually makes just the opposite point from the one you intended.

    Despite the suggestive "array\_set" name, the subscript operation might have been overridden to do anything whatsoever, and can vary from call to call, depending on the argument "a". In the C function, on the other hand, the only thing that can go wrong is an out-of-bounds index, which can be trivially checked with an assertion.

33. [Joel J. Adamson](http://trashbird1240.wordpress.com) says:[17 October 2011 4:19 PM](#5f4408f8cbdb27cad0095eb72551b2108607dd11)

    Hello Andy,

    As many others have commented, I really like programming in C. I like the language itself, even as someone who uses Lisp on a regular basis. I'd say C is my favorite non-Scheme language :)

    I like it way better than Python, but for many tasks, shell is appropriate.

    Another thing that I've been thinking about since first reading this entry was that it's still very useful to learn C. Many newer languages offer automatic memory management of some form, but it's a good idea to learn memory management in C. For example, I didn't know the difference in scope between dynamically allocated and statically allocated variables until well into a large project in C (which is not network aware). Since Scheme always took care of that, I wasn't sure why I couldn't just return an array and have it persist in the caller.

    Educate me: if I'm building an application which will not offer any networking to the user and C is the best language ceteris paribus, should I still not use C?

    Joel (@trashbird1240 on identi.ca)

34. [Andy Wingo](http://wingolog.org/) says:[17 October 2011 5:26 PM](#28ffd91453228f7a0c21a646c99b35693a81d6a7)

    The polemics continue! :)

    Jk, std::vector is unsafe in practice because most people use operator\[\], not at().

    But TBH that is beside the point. My basic point is that all programs have bugs, and that bugs in C and C++ code are dangerous to the user, whereas the same is not true in safe languages. This isn't to say of course that programs in e.g. Scheme don't have catastrophic failure modes. It is rather that they do not have the specific flaws that C and C++ have which lead to a large number of the real-world program failures that are \*commonly\* exploited, maliciously, to harm users.

    Here's a remote code execution vulnerability from Thursday: [CVE-2011-3208 cyrus-imapd: nntpd buffer overflow in split\_wildmats()](https://bugzilla.redhat.com/show_bug.cgi?id=734926). (Though that report was a month ago, the fix only hit Fedora last week.) What is the root cause of that vulnerability? It is in C's poor string (and, in general, array) support.

    Joel, C does have advantages, as I said, and it's almost enough for me to really like it! I am simply mentioning the disadvantages here, and concluding that for me they are more important. It is a rare piece of software that is not ultimately connected to the network -- which, by that, I mean "operates on attacker-controlled data". Witness all the file-format vulnerabilities of late.

    So now I sound totally paranoid and anal and all, as if the original article wasn't bad enough. But you know, this is stuff to think about IMO. Everyone has to make their choice about these things; this article just argues to make it with your eyes open. Think about it!

35. peter says:[17 October 2011 7:15 PM](#14d976ab022bc8194f7e5f824e00728af9be9ade)

    rixed, I agree that Concurrency is hard, and especially so in languages (like C) which support Concurrency in the form of threads.

    There are language solutions to this though -- especially shared nothing, message passing Actor-like concurrency as for example implemented in Erlang or Go are much safer from races and deadlocks than their threading cousins.

36. [Rich](http://piran.com.au) says:[18 October 2011 8:53 AM](#779b25bb41e7532c429b29de6ed8beab30c7cebd)

    I've played a wee bit with Go, and I think it makes a great candidate for a successor to C.

37. Brandon says:[19 October 2011 9:08 PM](#935321295c01282e28f08eba35814a89f28af7be)

    Andy,

    As always, there seem to be far too many people out there for whom the idea that one should stop using C in real-world development is a novel concept. I wish this wasn't such a controversial idea, but you're doing the right thing by beating people over the head with it. Say it enough times and people bight even start to believe you at some point.

    I think part of the problem is the lack of an obvious replacement for the niche which C itself still fills, i.e. low-level systems programming. It's not that there aren't alternatives: it's that there are many, each with their own set of flaws that are different and less-familiar from those of C.

    As Douglas Crockford, author of "Javascript: the Good Parts", has wryly observed on several occasions, programmers are often the last people to accept advances in programming language and compiler technology. Programmers seem to form their programming ethos at an early stage in development, and seldom change that ethos during the course of their career. It seems to take a generational shift to drive the state of the art forward -- and even then, those on the forefront of each technological revolution are often in the minority: we acquire our skills in a social mileu steeped in the traditions of the past. And we, as programmers, have an emotional bond with their language communities that is deeper than many people really appreciate. You're on the right track with your use of the tribal metaphor.

    I've skimmed the comments on this, and similar, blog posts, and the pro-C rhetoric seems to boil down to only a few different flavors.

    My favorite when people look at your contrived example exhibiting undefined behavior and go "well, duh, of course that's bad. Everybody know's you're not supposed to do that! Programming is hard. Be careful", forgetting that this is a contrived example, and that in the real world an equivalent bug will be much harder to spot.

    I like to call this the "NRA" argument. It's a lot like the NRA slogan, "Guns don't kill people, people do" (recast as a statement about programming, "C doesn't cause buffer overrun exploits, programmers do!".) Well, of course this is true at a fairly literal level, it's missing the point.

    Framing the debate as a matter of public safety and consumer protection is a step in the right direction. We're not necessarily the consumers of our own products, and some happless end user suffers the consequences of our design decisions. Would you buy a new car without airbags? Would you want your kids playing with one of those old-fashioned metal fans you could easily stick your finger into? Do you trust your credit card numbers to a web application written in C? If you like C, fine, use it for recreation. Stop forcing it on your users.

38. Craig Barnes says:[22 October 2011 4:15 AM](#4fe7383d25adf3765c722f159577cffcd36d43ab)

    Comparing the dangers of a highly regulated industry that poses inherent risks to human life with the dangers of a programming language is disingenuous.

    Some data is very important. Most isn't. Some software has the potential to affect human wellbeing. Most doesn't.

    Nuclear engineering is highly regulated, nuclear engineers highly trained and at any given moment we are potentially hours from a catastrophic meltdown. Let's face it, software is completely unregulated, chock full of charlatans and \*mostly\* harmless.

    Anyway it's not the tools that expose us to risk in the software business - it's the idiots who use them. Validating and bound checking really isn't that hard.

    Sorry but for performance intensive applications, the benefits of using C vastly outweigh the risks and will continue to do so for a long time to come. No migration plan required.

39. [David Banks](http://www.solasistim.net/) says:[24 October 2011 10:19 AM](#7b7c51d5eac655dd43b661cc611054ca32359f09)

    IMO this should be uncontroversial by now. It's just common sense.

    Agreeing the successor language, though...

40. [exa](http://e-x-a.org/) says:[26 October 2011 7:12 PM](#bf639e27aaa08b68aca2aef688483fb93856feec)

    Dear sir,

    please do not look on the programming languages as systems that have 'features', 'advantages', 'disadvantages' and 'vulnerabilities'.

    Programming languages are only the tools that allow us to express our ideas in something that is better for the actual expression than assembler, yet the computer still understands it. The whole thing is actually \_just\_ like making a sculpture: you can get a big hammer, make it fast and efficiently, but if you're not experienced enough, you'll almost surely fuck it up. You can get a small hammer, program a bot that handles it, and make somehow sure it finally gets the statue out of the stone. Or you can get smart, and use both hammers - big at first, and smaller for details.

    From this side of view, C is an enormously big hammer that needs experienced individual to carry, but it totally delivers the power. If you don't need the power, you're welcome with more advanced and expressive languages. That is where you should make a choice.

    My choice is C/C++ for products and python for prototypes and stuff that is needed to be scriptable. If your choice is to leave C, please be sure to consider all the possibilities that you've chosen to ignore and discard.

    -exa

41. Jan says:[14 November 2011 6:08 PM](#739014494d676103b1df0147e0f4a5e111fee714)

    Seriously? After 20 years of regular buffer overflow bugs and messed up memory management in countless applications we still have to discuss wether or not C/C++ is still the right choice for large real world applications? Don't get me wrong: There are problems where C/C++ is just right. But as with any dangerous technology it should be used very carefully and only for the parts that \*really\* need it.

    Why? Because the ramifications of bugs are far more serious and most software still has bugs (and will probably always have), so judging by the "developers are the problem not the language" argument: Most developers don't seem to be good enough to handle the power and danger of C/C++. Chances are you too are not as good as you think. I know I'm not, so I try to avoid writing \*everything\* in C/C++ for 3% more speed.

42. [Christopher Browne](http://linuxdatabases.info) says:[7 December 2011 6:13 AM](#cdc4232995905d94453ce4d9e9778be3fc81ecee)

    A "replacement" for C seems, to me, to need to be usable to implement low level "platform-y" stuff.

    So Python, Lua, and, for that matter, Scheme, need not apply.

    Java's not much of an answer, either, as it's not self-hosting.

    The only language that strikes me as plausible is Go, which \*does\* have a self-hosting implementation. (As well as a GCC-based implementation, which is nice from a diversity perspective.) It seems as though it's a "better C," though I haven't built anything interesting in it yet.

    It doesn't have enough library infrastructure yet, and there isn't yet enough diversity of applications using it. I rather hope that this changes.

    The other three possibilities, in principle would be:

    \- If SBCL self-hosted onto enough platforms, that would be interesting, though I don't think there are enough "believers" in Common Lisp for it to be realistic.

    \- Erlang's mighty interesting. Though writing things in near-Prolog tends to be pretty irritating... And it doesn't look as though it's progressing towards self-hosting.

    \- If OCAML or a Haskell implementation headed to self-hosting, or perhaps self-hosting via native compiling atop Go, that would be interesting.

    I think you need to have meaningful libraries implemented in \[whichever\] of these, that's usable by a Python implementation (for good measure!) implemented atop whichever language you're looking at. (People keep implementing Python in different languages, so that's pretty plausible.)

    Properties that are probably implicit in this include such things as...

    \- Needs to compile to native code on a goodly variety of platforms

    \- Needs to be able to be used as the location where significant libraries to be used by other systems built atop the language may be used and reused

    For something to be a replacement for C, you'd have to be prepared to...

    \- Implement a successor to Postgres in \[NewLang\]

    \- Implement a successor to Tk in \[NewLang\], complete with bindings for a bunch of languages (and not just Tcl!)

    \- Implement an operating system in \[NewLang\]

    \- Implement a scripting language (Bash, rc, SCM, Python, Ruby) in \[NewLang\]

    Given 5 more years of development, I suspect Go \*might\* be ready for that; I don't think it is today.

    The other "waggish" answer would be Ada, as there are now mature and reasonably performant implementations of Ada, capable of the sorts of implementation tasks I describe above. (A buddy built a shell language in Ada, called "BUSH", so that bit's done!)

43. Craig says:[19 December 2011 9:58 PM](#27da455e0f16a9b435e7d47e86077231e726d40f)

    @Christopher Browne

    Nice handwaving, bro.

44. Jeremie Pelletier says:[6 November 2012 0:47 AM](#481832ff71b2ed4dde4370f7645e4194a5bdd6d7)

    The D programming language is so far the best imperative language I've ever worked with. No other systems level language as come close in the fun/pain ratio to fun as D, to the point I sometimes miss using a debugger. It kept all of C's features, and is in fact ABI-compatible with C, and to that added the good parts of C++, Java, Ruby and other languages to the mix. The result is a surprisingly balanced multi-paradigm C making it very, very hard to shoot your own foot without making it impossible; having inline assembly is the very proof of that!

    It has all the close-to-metal you'd expect from C, a template system far more powerful and cleaner than C++'s, compile-time function evaluation which provides more or less an equivalent to lisp's macros and mixins which I can only describe as JavaScript's eval but compile-time and type-safe.

    The language is certainly more complex and high-level than C (unless restricting yourself to that subset), but it manages to keeps you sane, unlike C++. At the same time, it does so without assuming you're a child programmer and forcing you to wrap everything with toy classes, unlike Java.

    It has pointers and optional garbage collection as well as the notions of data immutability, shared memory, function purity, memory-safety, type inference, design-by-contract, conditional compilation, and quite a lot more. My programs are as performant as they used to in C, but the code is as almost as short as the Ruby equivalent.

    In some cases D even performs better; having a GC means there are no pointer management and scattered deletes, having data immutability allows for copy-on-write strings and having a productive language lets you get the prototype up running much faster to spend the rest of the time optimizing the hot parts.

    While Go looks like a promising language, it doesn't offer a lot of D's features and I got myself addicted to quite a few of them.

Comments are closed.
