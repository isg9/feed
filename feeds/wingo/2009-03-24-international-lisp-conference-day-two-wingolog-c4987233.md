---
title: international lisp conference -- day two — wingolog
url: https://wingolog.org/archives/2009/03/24/international-lisp-conference-day-two
published: "2009-03-24T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/03/24/international-lisp-conference-day-two
---

# international lisp conference -- day two — wingolog

## [international lisp conference -- day two](/archives/2009/03/24/international-lisp-conference-day-two)

24 March 2009 2:15 PM

- [ilc](/tags/ilc)
- [lisp](/tags/lisp)
- [cambridge](/tags/cambridge)
- [guile](/tags/guile)
- [scheme](/tags/scheme)

So, picking up where I [left off](//wingolog.org/archives/2009/03/23/international-lisp-conference-day-one)...

[David Moon](http://en.wikipedia.org/wiki/David_Moon), about whom the internet knows surprisingly little, gave a wild talk on a language he's been working on: the [Programming Language for Old-Timers](http://users.rcn.com/david-moon/PLOT/). Actually the talk wasn't about the language, but its macro system. PLOT is a language with syntax, but whose grammar is defined in itself.

PLOT macros take token streams, produced by a fixed tokenizer, and perform LL(1) parsing on that stream, building up abstract syntax tree objects or returning other token streams. Actually, given that macro output can be AST objects, macros take AST fragments as well. The interesting thing is that macros are hygienic, that they track source locations, and that they have access to the full PLOT language themselves. It seems that you can actually implement Lisp macros on top of PLOT macros.

Anyway, a sharp tool, but I look forward to seeing what Moon carves with it.

Next up was a talk by Joe Marshall, a Scheme hacker now working at Google, about implementing continuations on machines with no continuation support, like the JVM or CLR. The basic idea was for functions to be modified to return an OK/Error flag, with the actual return value in an additional output argument. The compiler should do some live-variable analysis, and have a separate flow path that saves variable values all the way down the stack, as functions return "Error", and that can be restored when you reinstate a continuation.

It's nasty. Joe must really want Scheme on the CLR.

Next there was Alexey Radul, a grad student of Sussman's, giving a talk entitled "The Art of the Propagator". The concepts were interesting, although hindered by Radul's poor diction. The basic idea appears to be an approach to calculating constraints-based systems in the presence of backtracking and conflicting information, with electronics as a metaphor -- connecting boxes and wires, signals flowing back and forth. Not like any problem I have ever seen in practice, though.

Following that were a set of lightning talks, the last of which was legendary: one M. Greenberg (who was that guy?) talking about the history of an old macro system, TRAC. Apparently all computation in TRAC was done by macros. Macros could rewrite their definitions out of existence, and thus be recursive and terminate. It was startling, but Greenberg steeled our resolve with the words, "Don't be afraid of macros -- they'll sense your fear, and fuck you up."

Then, a panel discussion on the future of Lisp, the ostensible topic of the conference. From left to right there was Pascal Costanza, an academic from Brussels hacking mostly on CLOS-related things; Edi Weitz, an industrial Common Lisp hacker; Ravi Nanavati, an erstwhile Schemer but current Haskeller; Rich Hickey, Clojure language designer; Alan Bawden, former Scheme Steering Committee member; Duane Rettig, Allegro CL hacker; and Scott McKay, a former Scheme and Dylan hacker now working for ITA software. Kent Pitman moderated.

It started good, with the long view, then things devolved. Eventually the audience started to talk with each other instead of with the panel, and for some reason settled down onto the "why isn't Lisp popular" navel-gaze. Quite disappointing.

After dinner, there was a staged "debate" on "Macros: Are they a menace?" It was obvious that the panel (Steele, Gabriel, Costanza, and the scapegoat, someone whose name I forget) was having a good time. At one point, Gabriel wrote out the "(defmacro .....)" on the chalkboard, noted that the "..." is often large and thus heavy and must sink to the bottom, leaving the parenthesis before the "defmacro" looking like a frowny face: q.e.d.

More seriously, the arguments against macros centered on control: control by bosses on workers, with the idea that macros make custom languages, thus making individual programmers less replaceable. While I don't believe the premise, the conclusion to me comes from another world. I hack Free Software precisely because I like my trade, and want to practice it in an environment free from coercion and domination.

The "debate" had an interlude, in which Costanza asked Sussman why MIT had switched away from Scheme for their introductory programming course, 6.001. This was a gem. He said that the reason that happened was because engineering in 1980 was not what it was in the mid-90s or in 2000. In 1980, good programmers spent a lot of time thinking, and then produced spare code that they thought should work. Code ran close to the metal, even Scheme -- it was understandable all the way down. Like a resistor, where you could read the bands and know the power rating and the tolerance and the resistance and V=IR and that's all there was to know. 6.001 had been conceived to teach engineers how to take small parts that they understood entirely and use simple techniques to compose them into larger things that do what you want.

But programming now isn't so much like that, said Sussman. Nowadays you muck around with incomprehensible or nonexistent man pages for software you don't know who wrote. *You have to do basic science on your libraries to see how they work,* trying out different inputs and seeing how the code reacts. This is a fundamentally different job, and it needed a different course.

So the good thing about the new 6.001 was that it was robot-centered -- you had to program a little robot to move around. And robots are not like resistors, behaving according to ideal functions. Wheels slip, the environment changes, etc -- you have to build in robustness to the system, in a different way than the one SICP discusses.

And why Python, then? Well, said Sussman, it probably just had a library already implemented for the robotics interface, that was all.

\\* \\* \\*

Now it's Tuesday morning, which started with some guy talking about bombing people with Lisp. Or maybe it was about supplying parts to the military. At one point there was a slide with a little clip-art tank and a clip-art pistol and a clip-art fighter jet. Sheesh.

Finally, via [Richard Gabriel](http://dreamsongs.net/): [Cobol on Cogs](http://coboloncogs.org/INDEX.HTM). Enjoy.

## related articles

- [international lisp conference 2009, day three, part 1](/archives/2009/03/25/international-lisp-conference-day-three-part-)
- [international lisp conference 2009 -- day one](/archives/2009/03/23/international-lisp-conference-day-one)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [quasiconf 2012: lisp @ froscon](/archives/2012/08/15/quasiconf-2012-lisp-froscon)
- [the very merry month of may](/archives/2009/05/10/the-very-merry-month-of-may)
- [dispatch strategies in dynamic languages](/archives/2008/10/17/dispatch-strategies-in-dynamic-languages)

### 14 responses

1. [Boudewijn Rempt](http://www.valdyas.org) says:[24 March 2009 3:06 PM](#6ac9b0253b8cdcd99f8bf9e2378d2f077ddccd77)

   Thanks for these reports! Todays reports suggests why I, as a linguist, feel right at home programming: basically, I've been trained to do fieldwork on incomprehensible languages and make sense out of them. Doing basic science on a library is just the same thing :-)

2. [Boudewijn Rempt](http://www.valdyas.org) says:[24 March 2009 3:08 PM](#b2d6ba34d283ab6e512bf7c5053debab675bba1e)

   Btw, may I suggest doing a trim on the input to your web form? Putting a space in front of my email address doesn't make it invalid!

3. [wingo](http://wingolog.org) says:[24 March 2009 4:02 PM](#79085c48503cb2621aff46f022925a80f332e5d2)

   Thanks for the comments Boudewijn! And yes I should fix the validator, it's much too fascist.

4. fusss says:[24 March 2009 8:06 PM](#cfece3d3005913f57a57c5eed4bdb79b0807460e)

   They're not just "scheme hackers" or unknown characters. David Moon wrote his own version of the MacLisp manual, the Moonual, as part of the effort to port MacLisp to the Multics. He is credit in both CLtL and the HyperSpec, a Symbolics co-founder whose name appears attached to everything from NCOMPLR (the "new" compiler for MacLisp circa 1974), the Honeywell port of MacLisp. He INVENTED ephemeral garbage collection and implemented it in the Lisp Machine. He co-designed Flavors, the old, pre-CLOS, object system for Lisp. David Moon NAMED Common Lisp :-) in addition to coining the term "Sufficiently Smart Compiler" which is tossed around in the compiler hacking lore as a cute euphemism. He is part of the ANSI Common Lisp standardization committee.

   If the internet doesn't know much about David Moon, then the internet be damned.

5. fusss says:[24 March 2009 8:18 PM](#17cc5faae85bf5abb83c470aafa13cb4350c6373)

   "Next up was a talk by Joe Marshall, a Scheme hacker now working at Google"

   Joe Marshall is not JUST a Scheme hacker. He is a Lisper's lisper, the guy wrote the REBOL compiler.

   "Scott McKay, a former Scheme and Dylan hacker now working for ITA software"

   Not \_just\_ Scheme and Dylan. The man, along with Robert Fahlman and the CMU researchers created Spice Lisp and CMU Common Lisp. He designed CLIM, the Common Lisp Interface Manager, the standard GUI toolkit for CL which influenced his later DWIM project (Dylan User Interface Manager.) A former engineer for both Symbolics and Harlequin.

   Now your history people.

6. [Robert Fisher](http://web.fisher.cx/robert/) says:[24 March 2009 10:57 PM](#38c6e9007fd5100f30df33aca94a9ef8d3e6adb0)

   Re: 6.001: Did Sussman consider his description of software engineering today a natural evolution or a recognition of an unfortunate reality?

7. [Paddy3118](http://paddy3118.blogspot.com) says:[25 March 2009 3:50 AM](#74914c7606a61a6154a908e8cf9a0ac6f43e1945)

   Could you comment on how you thaught Sussman was aiming people to take his comment on Python:

   "it probably just had a library already implemented for the robotics interface, that was all."?

   Was it in jest?

8. Jeremy H. Brown says:[25 March 2009 4:05 AM](#be6cd02112380a38ba51046ddde261397218fb71)

   I was the "scapegoat," although I would have said my job was more rodeo clown. My slides and talk outline are [here,](http://people.csail.mit.edu/jhbrown/macros/) should any of your readers be interested.

9. A little schemer says:[25 March 2009 4:23 AM](#994a736cf41c9bb4528e73084424e11a983f53a2)

   @Robert Fisher: I was there for this panel, too, and I think it was probably a little bit of both.

10. [wingo](http://wingolog.org/) says:[25 March 2009 12:48 PM](#07e9e6027aff60c6fb1927b950900eac6db9b247)

    @Jeremy: Thanks for stopping by, and apologies for my ignorance!

    I actually liked your point that most uses of macros should be part of the language, like Python's `with`.

11. Jeremy H. Brown says:[26 March 2009 0:02 AM](#6b8480365f735e87ca40e97b54a0b8ccac8def1d)

    No problem. Thanks for taking an interest in the discussion.

    Wearing less of an absolutist hat than I took to the debate, macros are a nice mechanism for laboratory experimentation. You can use it to experiment with the syntax of while, or object definitions, or whatever. But once you've stabilized on some patterns, you should push the constructs into the core language.

    There's also some line of taste which is hard to put a finger on. Scheme has a more or less completely orthogonal set of core constructs, then derives some useful combinations on top of them with macros. But the utility of case, cond, etc. have been well-demonstrated "in the lab", and it'd be as reasonable for them to be "core" special forms rather than macros.

    What I'm really saying is, if you're writing crazy nonsense like (setf-if x y z) to save yourself the extra set of parens in (if x (setf y z)) you are tasteless and your license to build macrologies is revoked.

12. [Patrick Logan](http://patricklogan.blogspot.com) says:[11 April 2009 5:39 PM](#93e25b62bed06eb55be263a1c3c47d551b187083)

    "once you've stabilized on some patterns, you should push the constructs into the core language"

    um. what does this even mean for a lisp system? the compilers I am familiar with try to get to a simple core language that can be compiled without a lot of special cases. and so most syntax is implemented as macros whether you want to think of them as such or not.

    for other interesting syntax that comes along, what does this mean? you lobby everyone to adopt it into "the core"? that may work for java.

13. [Mark Miller](http://tekkie.wordpress.com) says:[11 April 2009 9:55 PM](#4a1f919f9dd4e49190621e852034f56adc751103)

    It was real interesting reading about Joe Marshall's talk on implementing continuations in languages that don't have them. I did something similar to what you described of his scheme, in 1995, practically just out of school. I wasn't going for continuations. I was trying desperately to build robustness into a C program I was maintaining, a script interpreter. One of the problems that plagued me was non-existent return values from functions (which came back as garbage). My memory is the compiler I used didn't detect this. So I set up a scheme where each function created for the interpreter returned two values (success or failure), and I took as disciplined an approach as I could in accounting for every contingency in my code (though I missed some things). I encased every function call to the interpreter's own functions inside case statements that would look for either of the two values, and I had a default case which would detect errant return values. In the failure or default case it spit out an error message and unwound the stack, ultimately exiting the program. All of my returned data values were "out" parameters from the functions called. The one thing I didn't put in was a listing of the stack, so I couldn't tell what led to the fault. What I had was enough for diagnosing problems.

    Along with this, after much trial and error, I had come up with a simple macro that would check ALL of my pointer references before I used them. If any pointers came up NULL (unless where intended) it would spit out an error message, locating the errant pointer, and exit the program.

    The whole inspiration for this was that I was working in MS-DOS (no Windows) which was a terrible development environment. If the program crashed (which happened at least a few times a day, because I was always changing things in it) it invariably locked up the computer, and I'd have to reboot it. I got real tired of this!

    It was a LOT of work to implement these schemes. Looking back on it now I wonder if it would've been better to create my own C runtime that would've done this stuff for me...that is if I had the knowledge of how to do that at the time.

14. vadim says:[10 March 2010 4:28 PM](#75a6bde216b76affd5f03c0d66c9998473f3c65f)

    fusss said on 24 March 2009: "\[Scott McKay\], along with Robert Fahlman

    and the CMU researchers created Spice Lisp and CMU Common Lisp."

    I have it on very good authority that at the time CMUCL was created,

    Scott McKay was not involved. He was busy doing some of the other

    things that you correctly attribute to him. You must've confused him

    with SCOTT Fahlman and Robert MACLACHLAN

    (http://www.cons.org/cmucl/credits-print.html)

Comments are closed.
