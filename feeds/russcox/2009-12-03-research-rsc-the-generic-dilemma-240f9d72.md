---
title: 'research!rsc: The Generic Dilemma'
url: https://research.swtch.com/generic
published: "2009-12-03T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/generic
---

# research!rsc: The Generic Dilemma

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# The Generic Dilemma Russ Cox December 3, 2009 *research.swtch.com/generic* Posted on Thursday, December 3, 2009.

Generic data structures (vectors, queues, maps, trees, and so on) seem to be the hot topic if you are evaluating a new language. One of the [most frequent questions](https://golang.org/doc/faq#generics) we've had about Go is where the generics are. It seems like there are three basic approaches to generics:

1. (The C approach.) Leave them out.

This slows programmers.

But it adds no complexity to the language. 2. (The C++ approach.) Compile-time specialization or macro expansion.

This slows compilation.

It generates a lot of code, much of it redundant, and needs a good linker to eliminate duplicate copies. The individual specializations may be efficient but the program as a whole can suffer due to poor use of the instruction cache. I have heard of simple libraries having text segments shrink from megabytes to tens of kilobytes by revising or eliminating the use of templates.

3. (The Java approach.) Box everything implicitly.

This slows execution.

Compared to the implementations the C programmer would have written or the C++ compiler would have generated, the Java code is smaller but less efficient in both time and space, because of all the implicit boxing and unboxing. A vector of bytes uses significantly more than one byte per byte. Trying to hide the boxing and unboxing may also complicate the type system. On the other hand, it probably makes better use of the instruction cache, and a vector of bytes can be written separately.

The generic dilemma is this: *do you want* *slow programmers, slow compilers and bloated binaries,* *or slow execution times?*

I would be happy to learn about implementations that somehow manage to avoid all three of these bad outcomes, especially if there were good written descriptions (papers, blog posts, etc.) about what was hard and why it's a good approach. I'd also be interested to see good written descriptions of trying one of those approaches and what went wrong (or right).

Please leave comments with pointers. Thanks.

(Comments originally posted via Blogger.)

- [yuku](http://www.blogger.com/profile/15226462280828485140)(December 3, 2009 9:54 AM) How about C# implementation? It lets you use primitive types.

- [Barry Kelly](http://www.blogger.com/profile/10559947643606684495)(December 3, 2009 10:00 AM) The massive gaping hole in your list of approaches is .NET: reified generics, but in a reference-type oriented language.

C++ suffers because it has lots of support for value-oriented programming, with "clever" tricks like copy constructors, assignment operators, implicit conversions, etc. The lack of GC encourages flirting with almost-but-not-quite-good-enough smart pointers, and storing smart pointers in collections means every collection needs to call the appropriate copy constructors, destructors, etc. for internal operations. It makes it hard to share code.

.NET has reified generics, but all generic instantiations with reference types use the same actual instantiation behind the scenes, using the hidden type "System.\_\_Canon" as the type argument. This requires smuggling in the actual type argument list in certain locations - e.g. a hidden argument to generic method calls, a field in the type descriptor for distinct instantiated types, etc.

.NET generics are also instantiated at runtime, which means it can share instantiations across multiple dynamically-linked libraries. That's somewhat harder to get right in a precompiled environment; a few indirections and you may be able to get most of the benefit by having only one instantiation active and in caches, leaving the others cold and not paged in.

- [andrewducker](http://andrewducker.livejournal.com/)(December 3, 2009 10:15 AM) http://msdn.microsoft.com/en-us/library/ms379564(VS.80).aspx talks about the .Net Generic implementation in a bit more detail.

- [Paul Snively](http://www.blogger.com/profile/01862832764185349715)(December 3, 2009 10:51 AM) I would also take a look at BitC's [polyinstantiation](http://bitc-lang.org/docs/bitc/polyinst.pdf) system. In fact, in general, I would take as much inspiration from BitC as you can without sacrificing whatever specific goals you may have (e.g. blinding compilation speed) that BitC doesn't necessarily share.

- [newt0311](http://www.blogger.com/profile/00275501056310821335)(December 3, 2009 11:11 AM) Super-compilers would solve this problem but making them work effectively requires some major developments in compiler research. Its one of the few areas of academic CS that is useful.

- [CCs](http://www.blogger.com/profile/00577573410818964827)(December 3, 2009 11:19 AM) C# don't let you use "int" or other basic type as a parameter for generics.

I think C++ choose the "least bad" solution - you compile once your app, but run it many times.

- [Mikhail](http://www.blogger.com/profile/11722347898057818306)(December 3, 2009 11:46 AM) @CC: not sure if I understand you correctly but with C# you can do following:

// Declare a list of type int.

GenericList list1 = new GenericList();

- [David](http://www.blogger.com/profile/04628223255570601432)(December 3, 2009 11:52 AM) Yes it does, e.g. Action\[int\], a type representing a function that takes an integer argument and returns no value.

- [drhodes](http://www.blogger.com/profile/10482944964078869642)(December 3, 2009 11:53 AM) This paper might be worth a gander:

http://www.osl.iu.edu/publications/prints/2003/comparing\_generic\_programming03.pdf

A snippet from the abstract: ... a comprehensive comparison of generics in six programming languages: C++, Standard ML, Haskell, Eiffel, Java (with its proposed generics extension), and Generic C#. ...

There's a newer paper (2007) though it's behind a paywall.

A cursory glance at the table on page 3 shows how comprehensive haskell's feature set is, with respect to every other language listed.

Composable typeclasses would fit Go's orthogonal philosophy like a glove, IMHO.

- [andrewducker](http://andrewducker.livejournal.com/)(December 3, 2009 12:55 PM) That paper is based on a prototype of generics in C#, before the final version was completed - implicit typing is mentioned as lacking, whereas the final version supported it (for instance).

- [David Andersen](http://www.blogger.com/profile/03996590425188586871)(December 3, 2009 1:28 PM) Hi, Russ - not a comment about your questions, but a meta-point: Your post didn't address the question of whether the containers are statically type-safe. While one could imagine this being done orthogonally to how the code works (e.g., static type checking, but then box/unbox with the same impl), in practice, it seems that most of the time the generics also provide the type safety, and the boxing/unboxing approaches provide only runtime type checking. In practice, the question seems to be: slow compilers, bloated binaries, but more static type safety. Still a question, of course.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 3, 2009 1:53 PM) \[2-part post\]

Go should definitely try and support the Generic Programming paradigm as envisioned by Stepanov, which most closely is accomplished with C++ templates, overloading, and specialization (which is why the STL was implemented in C++ to begin with). His recent book "Elements of Programming" does a great job of walking through Generic Programming, which I highly recommend all programmers read, no matter their language background.

That's not to say that Go needs templates in the C++ sense, all it needs is type functions, a way to specify *true* Generic Programming concepts (including associated types and what would have been C++0x concept\_maps, etc. not just the basic name/function signature matching of Go interfaces), and a way to specialize/overload algorithms based on concept refinements and specific models of concepts.

Keep in mind that C++'s templates with respect to generic algorithms and datastructures have much more of an advantage than being "fast", they figure out dispatching at compile-time and are able to deal with type/concept information at compile-time meaning more errors caught before the program is even run. As well, as you bring more relationships to compile-time, certain complicated operations such as a subset of the occasions you'd resort to multi-dispatch in a Java or C# world are instead resolved at compile-time via simple overloading and/or specialization. Being generic and being fast are not mutually exclusive as other languages seem to make people think. It is possible to get abstraction with no extra runtime overhead.

To Barry Kelly, I argue that C++ does not "suffer" because of value-oriented programming, by which I believe you mean that it emphasizes Regular Types, but it's really its strongest suit and an area that most other mainstream languages simply get wrong. Copy-constructors, assignment operators, move operations, and deterministic destruction are not "clever tricks", they are the natural and generic way to represent operations on Regular Types. Garbage collection and *nondeterministic* disposal are the "clever tricks" of the programming world used because, despite being more complex for a compiler implementor, are easier for more programmers to work with and provide results that are acceptable for many applications. What's more is, Regular Types don't make it harder to share and write generic code, they make it easier, which is actually the entire point. It is why you are able to have generic containers and libraries that work with built-in types and user-defined types just as easily and without sacrificing speed. This is already a needless disability of Go with respect to maps.

\[continued in next post\]

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 3, 2009 1:54 PM) \[continued from previous post\]

Now it's true that I'm a C++ programmer and enthusiast, but I'll let you in on a little secret. We use template metaprogramming, exploit SFINAE, and do all sorts of the idiomatic C++ stuff that outsiders think of as voodoo and overly complicated *not* because we like the challenge, but because despite the complexity it more fully, more generically, more safely, and more efficiently represents concepts, types and generic algorithms than just about any other language out there. The secret is that you can get that end result *without* a complex feature like templates and without many of the hairier parts of C++. Generic Programmers program in C++ because we *have* to. There simply aren't other options that allow you to express generic libraries as fully and efficiently in a statically-typed world.

Languages like Java and C# miss the boat on that and I fear that Go may do so as well. Generics and even Go interfaces in their current form do not let you employ the mathematical approach that Stepanov and Boost programmers use all of the time to create generic code. If you can't program a library as generic and efficient as Boost's Graph Library then you are simply missing something if your goal is to create a language that supports Generic Programming.

In my opinion, if Go is to be relevant to the Generic Programmers of C++, it needs to support the Generic Programming paradigm more concisely than C++ templates without sacrificing functionality, which while is difficult, is far from an impossible task. This is particularly important as C++0x has dropped direct support for concepts meaning that if Go beats C++ to its own game it will truly be in a league of its own. As I mentioned early, really all you need to satisfy most Generic Programmers needs are a way to represent concepts, type functions, a way to specialize/overload algorithms based on concept requirements, and preferably something like what would have been C++0x concept\_maps. These are all that it takes to make a statically-typed language that supports libraries like the STL, and yet other mainstream languages miss that.

Again, Stepanov's "Elements of Programming" is a great place to look to get a more mathematical view of Generic Programming and I encourage you to please read it. Go's interfaces are a tiptoe in the right direction and if they can be expanded to encompass concepts you will begin to see many more programmers migrating over to Go from C++.

- [CCs](http://www.blogger.com/profile/00577573410818964827)(December 3, 2009 2:43 PM) C# generics with basic type (like int)

Yes, you can declare it.

But T = T+1; will not work in C#. It works in C++.

(There are some workarounds, but no real solution.)

- [NoiseEHC](http://www.blogger.com/profile/13086247873171140062)(December 3, 2009 2:46 PM) There is a 4. option you did not think about:

Implement some meta-programming instead and leave it to the programmer whether he use it to create a 2. or a 3. solution... In this case the interfaces to dynamically loaded modules (like core libraries) will be like 1.

- [Levi](http://www.blogger.com/profile/13615410426974691647)(December 3, 2009 3:25 PM) @inproceedings{ harper95compiling,

author = "Robert Harper and Greg Morrisett",

title = "Compiling Polymorphism Using Intensional Type Analysis",

booktitle = "Conference Record of {POPL} '95: 22nd {ACM} {SIGPLAN}-{SIGACT} Symposium on Principles of Programming Languages",

address = "San Francisco, California",

pages = "130--141",

year = "1995",

url = "citeseer.ist.psu.edu/harper95compiling.html" }

@article{ jones98transformationbased,

author = "Simon L. {Peyton Jones} and Andr{\\'e} L. M. Santos",

title = "A transformation-based optimiser for {Haskell}",

journal = "Science of Computer Programming",

volume = "32",

number = "1--3",

pages = "3--47",

year = "1998",

url = "citeseer.ist.psu.edu/peytonjones98transformationbased.html" }

- [Andrew](http://www.blogger.com/profile/11083609636462293755)(December 3, 2009 5:51 PM) Have you looked at how D does templates? It has fast compile times with C++ style templates. I don't know well it does with code bloat, but are there any actual examples of C++ code bloat causing slowdowns outside of extremely contrived cases?

- [Joe Schafer](http://www.blogger.com/profile/13327224520746535437)(December 3, 2009 8:00 PM) Maybe take a peek at [Ada Generics](http://en.wikibooks.org/wiki/Ada_Programming/Generics). The generic instances are shared, and there is no boxing of types.

- [malkia](http://www.blogger.com/profile/07408170618420648277)(December 3, 2009 9:18 PM) How about this, if you have @ in front of a type, then delay compilation of the function, until it's first being used, then generate for the compiler the same body but with ?type changed with the real-type.

func sum(a @type, b @type) @type

{

return a + b;

}

func test()

{

a, b int8;

c, d int16;

sum(a, b);

sum(c, d);

sum(a, d); // error, but look at the the other way to do sum below

sum(a, int16(d)); //ok

}

func sum(a @typeA, b @typeB) @typeC

{

ca := @typeC(a); // if possible

cb := @typeC(b); // if possible

return ca + cb;

}

not complete as in C++, but still

- [Dethe Elza](http://www.blogger.com/profile/04312659663582995721)(December 3, 2009 10:07 PM) The Self language, which inspired Java, Javascript, and others would compile specialized versions of methods and structures as needed. Kind of super-generics or dynamic type safety. There is a good paper on it here: http://research.sun.com/self/papers/implementation.html

- [rog peppe](http://www.blogger.com/profile/00839344798030831980)(December 4, 2009 4:27 AM) Vita Nuova's Limbo does generics, but only on pointer types. This can be frustrating, but they're still very useful and it doesn't result in any code bloat.

This can be a starting point - if you can do this well (and it's not easy to get it totally right), then it's "just" a matter of generating code to deal with value-type instantiations. There I don't think there's any way around the huge-code vs. slow-speed tradeoff, but you could generate code for N optimised cases (e.g. pointer, 32-bit word) and one general case (arbitrary sized pointer-int layout) so at least it would all work, and there's the capability to generate more optimised code if necessary.

- [Julian Morrison](http://www.blogger.com/profile/01115506275519545033)(December 4, 2009 9:37 AM)This post has been removed by the author.

- [Julian Morrison](http://www.blogger.com/profile/01115506275519545033)(December 4, 2009 9:45 AM) Ignore my previous comment if you saw it, it was a bad idea.

I think I support the "leave it out" camp. You can box explicitly with interface{} parameters, and unbox safely with a type switch. If you need the extra speed of a specialized implementation, write one.

- [diabolikmachine](http://www.blogger.com/profile/06774308600854594267)(December 4, 2009 11:05 AM) instead of having a generic system that you can use anywhere in your code, what about a generics that you instantiate?

type string\_hash is new Generic\_Hash(string);

this is sort of how Ada does it, one of the few things in ada that makes sense.

- [Jonathan Amsterdam](http://www.blogger.com/profile/00724441893784184431)(December 5, 2009 10:13 PM) First of all, option 3 doesn't make sense for a language with Go's aspirations to efficiency. I don't even think it makes sense for Java -- it was forced on them by the need to keep both the language and the VM architecture backwards-compatible with tons of existing code. That illustrates the danger of waiting too long to add generics to a language.

Option 1, leaving out generics, is going to result in one or more of the following: greatly reduced market share; use of unsafe pointers to simulate the void\* style of generics you often see in C; and a variety of marginally useful and incompatible macro-based solutions.

So effort should be focused on option 2, correcting C++'s deficiencies.

First, speed: is C++ template instantiation slow because headers are constantly being reparsed and templates are instantiated multiple times with the same arguments? Go doesn't have those problems: it doesn't need to reparse headers, and it can adopt a repository-based solution from the outset, where instantiated templates are remembered somewhere and only generated once. Or is C++ slow because templates are so heavily used, e.g. for smart pointers, strings, etc? Go already has built-in features that obviate the need for some common C++ generics (e.g. GC makes smart pointers pointless), and by making Go's generics weaker than C++'s -- eliminating the ability to do meta-programming, for instance -- you'll reduce the number of templates that actually get written.

Second, space: can the number of instantiations be reduced? It seems that for at least some templates, you could generate only one version for all pointer types, and possibly one additional version per size of value type. C# does something like this, and this paper on Ada would probably show how the Intermetrics compiler dealt with the problem: "Implementation implications of Ada generics," by Gary Bray. (http://doi.acm.org/10.1145/989971.989974) Unfortunately, the ACM wants you to pay them to read it. (BTW, Joe Schafer: "The generic instances are shared, and there is no boxing of types" -- I don't think that's true, I suspect there would be two distinct instances for 32-bit ints and 64-bit floats.)

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 6, 2009 2:49 PM) I agree with Jonathan except for a couple of issues:

*"Go already has built-in features that obviate the need for some common C++ generics (e.g. GC makes smart pointers pointless)"*

The above is not entirely true. If programmers want deterministic disposal they are still going to have to resort to smart-pointers when shared ownership is desired.

*"...by making Go's generics weaker than C++'s -- eliminating the ability to do meta-programming, for instance -- you'll reduce the number of templates that actually get written."*

The issue is that compile-time metaprogramming is an emergent property of fully capable concepts and templates. Merely by having a complete concept or template system *implies* being able to do metaprogramming in some form. The only way around that is to handicap the system in a way which would affect the originally intended use.

As well, even if it were doable, I don't agree with the rationale of eliminating a useful feature for the fact that it can affect compile-times when used. If a programmer needs fast compiles then he simply won't use the feature. Even in C++ there are programmers who don't do metaprogramming because of compile-times, but (thankfully) their choice does not impact those who require it.

- [Jonathan Amsterdam](http://www.blogger.com/profile/00724441893784184431)(December 6, 2009 7:23 PM) Rivorus - I understand what you mean when you talk about a complete concept/template system implying metaprogramming. (Haven't read Stepanov yet, but I'm eager to do so.) But it doesn't follow that any template system that falls short is "handicapped." The generics of C#, Haskell, Eiffel and Ada relate to the theory of parametric polymorphism, where the idea is simply to parameterize types over types. In a sense, any such system is weaker than full metaprogramming, just as regular expressions are weaker than Turing machines. But that doesn't mean that they are deficient. They are just solving a different problem.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 6, 2009 11:41 PM) True, I suppose "handicapped" is a more harsh term than I intended. What I mean to say is that it rules out a particular approach to programming that is becoming more common, and for good reason, particularly in generic libraries of modern C++. There is a lot of potential for creating extremely powerful and generic libraries when you have facilities for type functions and ways to specialize algorithms and datastructures based on compile-time type and concept information.

The Boost Graph Library is a great example of that, and to a similar but lesser extent the STL. If languages are to advance in the direction of supporting more generic code I think that we need to focus on taking those ideas and translating them into something simpler than what currently exists as C++ templates but without removing overall functionality.

The paper linked by drhodes is a good reference to go by. If Go can be made to support the programming of a library like the Boost Graph Library better than all the languages listed it will be a step ahead for generic programming. In my opinion, anything else is a step back or at the very least unnecessary stagnation.

- [sambeau](http://www.blogger.com/profile/14572755603419046111)(December 7, 2009 3:44 PM) >"Composable typeclasses would fit Go's orthogonal philosophy like a glove, IMHO."

+1

- [ncm](http://www.blogger.com/profile/00831355954619691739)(December 11, 2009 2:44 PM) *"Go already has built-in features that obviate the need for some common C++ generics (e.g. GC makes smart pointers pointless)"*

Built-in features that duplicate library constructs in another language indicate weakness, not strength. They imply that the language is not strong enough to address those needs without special-casing. We might therefore point out instead that C++'s destructors make GC pointless, and (further) illustrate GC's weakness. GC can only manage memory, but not the numerous other resources that are also routinely managed with destructors. (We might add that exceptions are weakened, as an architectural aid, by the lack of destructors, to the point of near uselessness.)

It betrays fundamental confusion to group Haskell among C#, Eiffel, and Ada. C++ and Haskell can express what these other languages cannot. Regular expressions really are deficient, unless you have resolved to ignore any problem your language is not up to addressing. "Handicapped" is simply accurate.

A simpler language that is as powerful as C++ is long overdue. Go, sadly, isn't it. Go will grow, but it's fatally handicapped by unfortunate early design choices. C++ will get concepts and modules in 2015. Those won't make it a simpler language, but it will be simpler to program in, and compile times will come down. That will carry us until a plausible successor emerges.

- [Julian Morrison](http://www.blogger.com/profile/01115506275519545033)(December 11, 2009 3:06 PM) ncm, you forget that GC also manages ownership and allows all code to share the same ownership/destruction policy, which is vital for Java-style glue code that links together assortments of libraries.

"finally" blocks or Go's "defer" take care of deterministic resource reclamation.

- [Russ Cox](http://swtch.com/~rsc/)(December 11, 2009 3:12 PM) *C++ will get concepts and modules in 2015.*

Is your suggestion that we sit on our hands until then?

- [ncm](http://www.blogger.com/profile/00831355954619691739)(December 11, 2009 3:42 PM) "Finally" blocks are the inadequate response to the fatal weakening of exceptions. They wipe out the centralization of error handling that was the architectural purpose of exceptions in the first place.

rsc: You may sit on your hands if you like, or start developing a simpler language powerful to do what people are doing with C++ today. We haven't got one, and Go won't evolve to be it. Few people are equipped to develop a powerful new language, either intellectually or institutionally.

- [Russ Cox](http://swtch.com/~rsc/)(December 11, 2009 3:44 PM) *Go won't evolve to be it.*

This would be a more useful comment if you explained why.

- [ncm](http://www.blogger.com/profile/00831355954619691739)(December 11, 2009 9:31 PM) Ask *them*.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 12, 2009 1:11 AM) *"Is your suggestion that we sit on our hands until then?"*

Not at all, we'd hope that Go can do it right and get it done before then since you guys don't have to deal with a long standardization process as we do (neither concepts nor modules have made the cut for C++0x). Go is still in its infancy and if you can get as much as you can correct now you're going to save yourself from becoming irrelevant or in need of duct tape in the future. It's true that C++ is a mess but the reason we use it is because it can more fully express very high level, generic, and efficient libraries.

If you want Go to truly be a great language and pull more C++ users you're going to have to understand why C++ is as powerful as it is. Much moreso than the fact that it allows you to be as low level as C, it provides some of the best support for generic programming, especially when compared to other mainstream languages and Go. The C++ community has a love-hate relationship with the language. We love what it can do and we hate that it's a mess. Making a language as powerful as C++ is a matter of understanding generic programming and preferably being able to accomplish what it can do in a much more concise way.

As the paradigm has become better understood and libraries like the STL, Adobe/Boost's Generic Image Library, and the Boost Graph Library have come about, we have learned what fundamental functionality should exist to allow proper generic programming. Despite the onslaught of features that get thrown into languages like C# they still simply don't get it. In particular, you should have the ability to express concepts with associated types, concept refinement, overloading/specialization of algorithms based on which concepts arguments model, and the ability to produce type functions. C++ strives because it is powerful enough to hackishly represent many of these features through templates to some degree. All you have to do is support this functionality directly and you will be a step ahead with respect to generic programming.

- [Russ Cox](http://swtch.com/~rsc/)(December 12, 2009 6:47 AM) I am going to try one more time.

*Go, sadly, isn't it. Go will grow, but it's fatally handicapped by unfortunate early design choices.*

*Go won't evolve to be it.*

These comments would be more helpful if you explained what you perceive those unfortunate early desgin choices to be and why Go can't evolve to be "it".

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 12, 2009 12:55 PM) \[multipart post\]

*"These comments would be more helpful if you explained what you perceive those unfortunate early desgin choices to be and why Go can't evolve to be "it"."*

I'm not ncm and I'm a little more optimistic than he is. I do think that Go *can* evolve to be a solid language for generic programming, although certain things as they already exist would need to change. It's a matter of whether or not you as the language designers want to take it in that direction. ncm is probably just saying that Go has already gone in the "wrong" direction with a flawed rationale for certain fundamental ideas of the language, and while I do agree, from the looks of the FAQs you sound very open to change. Anyway, I don't want to put words in ncm's mouth, I'm sure he'll respond soon.

Here's my personal take on things, in no particular order. One "problem" that jumps out at me personally, in reference to the language FAQ, is in the question [*"Why are maps, slices, and channels references while arrays are values?"*](http://golang.org/doc/go_lang_faq.html#references). The rationale provided is very weak -- it talks about how early on the implementation of maps were "syntactically" pointers and that you couldn't create a non-dynamic instance. Further, it goes on to say *"...we struggled with how arrays should work."* Neither of these are answers to the question.

I'll try to shed some light on this from a C++ perspective. In C++, maps, and all other containers, are values. In fact, with only some embarrassing exceptions, all object types in C++ have proper semantics of regular types (those exceptions notably being legacy C-arrays, which is one reason why the TR1/boost array template exists, and std::auto\_ptr which exists as-is because of the lack of move semantics in the current standard and the need to efficiently and safely return dynamically allocated data from functions). Consistent semantics for regular types are extremely important to generic programming and that is why generic datastructures in C++ are as powerful as they are. You need a generic way of efficiently working with the fundamental capabilities of all types with a consistent high level menaing (for example, a single interface for copyability and what that means with respect to equality).

Similarly, the FAQ states:

*"Map lookup requires an equality operator, which structs and arrays do not implement. They don't implement equality **because equality is not well defined on such types**; there are multiple considerations involving shallow vs. deep comparison, pointer vs. value comparison, how to deal with recursive structures, and so on. We may revisit this issue—and implementing equality for structs and arrays will not invalidate any existing programs—but without a clear idea of what equality of structs and arrays should mean, it was simpler to leave it out for now."*

As I touched on in the previous paragraph, you need to represent the concept of regular types. You have a notion of copyability of arrays already and that should give hint as to how equality should be implemented. By very definition, making a copy of something means making another instance with the same value. Referential equality is a separate concern and should not be confused. The act of copying implies that the values of each object are now equal as should be reflected by a well-defined equality operator. If after a copy the two objects do not compare as equal either the copy operation wasn't truely a copy operation or the equality operation wasn't correctly defined.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 12, 2009 12:58 PM) \[multipart post continued\]

Another decision that I am in much disagreement with is the lack of what C++ programmers know as destructors. This is very briefly mentioned [here](http://golang.org/doc/go_for_cpp_programmers.html), and a rationale is virtually nonexistent. I am left to believe that such functionality was left out simply because the designers of Go do not understand how important destructors really are. [Herb Sutter](http://blogs.msdn.com/hsutter/archive/2004/07/31/203137.aspx) has covered this topic already fairly well and explains why destructors should be a preferred default over features like Go's "defer." That's not to say that defer is bad, it's just that it is no replacement for destructors. Using "defer" has to be done explicitly whenever a type is used rather than being specified once in a type's definition and used implicitly from then on.

In the [Go language specification](http://golang.org/doc/go_spec.html#Defer_statements) you can see a great example of where destructors are better suited than defer statements. When you create a lock, it would be a very bad idea to never unlock or to unlock at some nondeterministic time. A destructor makes this automatic and implicit. There is no defer statement that you can forget to write because unlocking is already implied. In fact, this is exactly how locks are implemented in Boost and how they are implemented in the next C++ standard. A similar need for destructors exists with any type that manages resources, which is many if not most non-trivial types. RAII is important and constructors/destructors are the key way these are implemented in C++. There is currently no simple way to do this implicitly in Go.

That alone is already a solid argument for destructors, but what does that lack of destructors mean specifically with respect to generic programming? A lot, actually. Whenever you use a type with a generic datastructure or algorithm and that code is responsible for creating instances of your type, you should hope that resources are deterministically cleaned up when an instance of the type leaves scope. You can't write a defer statement in generic code if you don't know what exactly needs to be cleaned up. This is an implementation detail that generic code doesn't (and shouldn't) know about. With features like destructors, the correct behavior happens implicitly.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 12, 2009 1:04 PM) \[multipart post continued\]

Finally, there is the issue of overloading. Once again, the [FAQ](http://golang.org/doc/go_lang_faq.html#methods_on_basics) has a very weak rationale, talking about the fact that it makes implementation of the language simpler and that it can be "confusing." Certain things might be tough to understand at first but that doesn't mean that they aren't *important* to understand. For many programmers, overloading is already second nature. Not only that, but it is extremely important to generic programming and leaving it out means not being able to create efficient and generic libraries.

In a statically-typed language, overloads dispatch based on data types, and in an ideal world, on what concepts they model. In a well-designed generic program in a statically-typed language you have high level generic algorithms built around smaller and smaller algorithms, each operating on models of concepts. If there is a more refined concept or a particular type that can have a more efficient implementation of a given algorithm, then that algorithm is overloaded for that type or concept. In C++, since all of the overloading is based purely on type/concept data, which functions are dispatched is decided at compile time and you get zero overhead.

The reason why this type of overloading is so necessary to generic programming and why simply naming functions differently doesn't cut it is because of the hierarchical nature of algorithms. A calls B which calls C which calls D and E, etc. each one with a generic implementation that can work on anything that models a particular concept, for instance, any container or range with access to any element taking linear time. Now imagine that algorithm "C" can be written in a way that is more efficient when it is passed a range that allows accessing of arbitrary elements in constant time rather than linear time. If you can overload C based on this type information, then you in effect make it so that algorithms A and B are also each automatically more efficient when you pass in a range with constant-time access to any element.

Without this ability to overload based on the type system you cannot get composable and efficient generic code. In reference to the example above, making a differently named "C" for the special case optimized situation rather than overloading implies having to make a differently named "A" and "B" as well if you want them to take advantage of your optimized code. This means that more algorithms have to be written if they need to be efficient rather than getting it for free and anyone using your algorithms now has to know to call your special case algorithm with the new name if he wants the more optimal implementation. If their code is already written, that potentially means they'd have to rewrite it if they need it to be faster simply because an implementation detail changed at some arbitrary location down the chain. It's just bad news for everyone. All of that is possible in C++ because of the type system, overloading and SFINAE. While the concept-like facility in Go is nice, without overloading and associated types it's only half-way there.

Those are just some of the issues I personally have with Go as it is. A lot of that can be changed and the FAQ makes it sound like you are very open to the possibility of such changes. The fact that you have taken the bold stride towards what you call "interfaces" (which again are basically just stripped down versions of "concepts" in generic programming) and away from traditional religious OOP shows a lot. I have faith that you guys are trying hard to provide a solid language -- one that compares to C++ in efficiency and the ability to write generic code. If that actually is your goal, I think you need to definitely pay attention to the things that C++ has done right and how that can be altered or improved to provide something better. Right now it seems you are not learning the lessons that C++ and its generic libraries have taught.

- [Jonathan Amsterdam](http://www.blogger.com/profile/00724441893784184431)(December 12, 2009 5:47 PM) Rivorus, I admire your passion but your optimism is misplaced. It is extremely unlikely that the Go designers are unaware of your arguments. I imagine they just feel that the benefit of generic programming is not proportional to the weight it adds to the language. Or, to use a word that they use a lot, it would not make the language more "fun". It's about as likely you'll see these features in Go as it is that you'd see static typing added to Scheme. Everyone gets the issues; it comes down to a matter of taste. I think that the best one could hope for is a simple generic type facility.

You'll notice that the Go designers did add complexity where they felt that it was worth it. For example, their reflection facility is much more powerful than C++'s, and one could argue that for many typical modern programming tasks, like XML serialization (see Russ's post later in this blog), that power is more helpful than the power of generic programming. (I've little doubt that you could find a way to parse XML into C++ structs using template metaprogramming, but I don't think it would be pretty, and of course the I/O time would dominate anyway, so it wouldn't be measurably more efficient.)

A technical point: you can achieve destructor-style resource management in Go like so:

\-\-\----

interface Disposable { Dispose() }

func using(r Disposable, f func()) {

defer r.Dispose();

f();

}

// example

conn := db.NewConnection(connString);

using(conn, func() {

... conn ...

});

\-\-\------

Thanks to the interface, this could be done generically as well. It also illustrates another powerful feature of Go, anonymous closures, that C++ is only just getting.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 12, 2009 6:07 PM) *"I've little doubt that you could find a way to parse XML into C++ structs using template metaprogramming, but I don't think it would be pretty, and of course the I/O time would dominate anyway, so it wouldn't be measurably more efficient.*

Very true, however with the Boost.Spirit library in C++ you can do it very nicely and it's surprisingly intuitive as well. Of course, you don't want to look at all of the library code that makes it possible unless you are really into metaprogramming as you have pointed out.

*"A technical point: you can achieve destructor-style resource management in Go like so:"*

That's actually exactly what I am talking about. What you posting is exactly the type of thing that destructors accomplish implicitly and with less code. As well, with just defer the "incorrect" solution happens by default if you forget. The code you posted is analogous to the C# or Java code mentioned in the post by Herb Sutter that I linked to. Everything he states when talking about Java and C# there applies to Go with respect to defer, minus the part about exceptions since Go doesn't \[currently\] have exceptions.

- [Jonathan Amsterdam](http://www.blogger.com/profile/00724441893784184431)(December 13, 2009 8:57 AM) First, let's keep hold of the thread of the argument. Your claim was that defer interfered with generic programming. I demonstrated that that was false, assuming everyone adopted a certain convention. (You could now argue that Go has done nothing to foster such a convention, and I would agree with you.)

Now let's talk about destructors vs. using for resource management:

Some minor points about Sutter's post: He writes the Java version of Transfer incorrectly. In a correct version, the commenter's point about exceptions thrown by Dispose would not apply. (You'd nest the try blocks.) And I don't know why he thinks you'd have to wait for a GC to release the queues' resources. Dispose() would do it immediately.

But I agree with Sutter's main point: destructors are better when applicable. Most of my day-to-day programming is in C++, and I love this aspect of the language. But we should consider how often they are applicable. Definitely for file I/O, since the most common pattern is to read or write completely. Not at all for locks, because locks are always longer-lived than the scope in which you lock them; so in C++ you need a wrapper class for RAII, and the code is about the same as you'd get with "using". Sometimes for network connections; but often you want to hold on to a connection for a while before tearing it down, and you may want to free it asynchronously, say after it has been idle for a while. You end up with some sort of connection pool, and then I don't think the code looks very different in C++ vs. C#.

It's worth pointing out that garbage-collected languages have the disadvantage that there is no delete, and typical garbage collectors may run at some arbitrarily distant point in time after a resource is released. I think Limbo had a great solution to that: it used reference counting for non-cyclic structures, and guaranteed that they would be freed immediately upon becoming unreferenced. As a practical matter, resource-holding objects are almost never cyclic, so the idea should work well (though I have no practical experience with it).

As for Go, I don't see it adopting destructors, because that would lead down the path of constructors, assignment operators and the like, and the Go designers are very clear that they don't want to go down that route. It's plausible that Go will adopt GC finalizers, but beyond that I think we should not expect any help from the language.

- [Julian Morrison](http://www.blogger.com/profile/01115506275519545033)(December 13, 2009 11:37 AM) I really don't see the advantage in immediate deallocation of memory. "free" isn't free. You are shuffling memory in itty bitty slices, when a compacting GC could clear and release it in bulk.

If you need to access a resource which needs cleanup through an interface, perhaps you should have a "Release()" method as part of the interface, which you can defer.

- [ncm](http://www.blogger.com/profile/00831355954619691739)(December 13, 2009 11:39 AM) Thank you, Rivorus and Jonathan, for taking the time to express more thoroughly and diplomatically, and at least as thoughfully, what I would have tried to write.

Jonathan, I think you miss the point about the limits of "defer". In R.'s ABCD example, if something in C calls for defer code, there's no way for the person coding A to know that, so it can't be written. A destructor is the only place where it could be put.

The point that language built-ins enforce cross- library conventions is a good one, although I disagree on the implications.

We've seen examples in the C++ Standard Library of conventions imposed on user code dictated not by the language, but by compatibility with that library, e.g. no throwing from destructors. The more powerful a language is, and the more of what might have been built-ins are instead put in the standard library, the more it is necessary for somebody to be able to dictate such conventions. This isn't an argument for built-ins, it's an argument for formalizing cross-library requirements.

The C++ community has been on the forefront of the effort to learn how to express such requirements. In a more powerful language than C++, more of what we consider built-ins (argument lists, loop control structures, vtables?) could be extracted to the standard library, and depend correspondingly more on such compatibility requirements.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 13, 2009 12:40 PM) \[2-part again\]

*"Your claim was that defer interfered with generic programming. I demonstrated that that was false, assuming everyone adopted a certain convention."*

You did nothing of the sort. You showed exactly what I was saying, you have to jump through hoops and be explicit every single time you create an object with nontrivial disposal when in actuality you should get the *correct* behavior *implicitly* by default and with no chance of error. The language is Turing complete, no one is saying that you can't reach the same end goal for a given application both with and without destrutors, only that it is needlessly more difficult without them and more error prone.

*"I don't know why he thinks you'd have to wait for a GC to release the queues' resources. Dispose() would do it immediately."*

He didn't say that Dispose wouldn't do it, he said that if you as the programmer do not explicitly Dispose then you get resources released at a noneterministic time after the object leaves scope (finalization). This simply cannot happen when you use destructors since the correct behavior happens automatically.

*"He writes the Java version of Transfer incorrectly. In a correct version, the commenter's point about exceptions thrown by Dispose would not apply. (You'd nest the try blocks.)"*

He is writing the simplest code whose end result is comparable to the corresponding C++/CLI code that he posted to illustrate his point about having to be explicit. He could have nested try blocks but it would have just made the code even more complicated than it already is. If you want to really be pedantic, you are also running into the fundamental differences between how exceptions are dealt with in C++ and in Java -- in C++, exceptions thrown but never caught in the destructor do not propagate, they cause a terminate, meaning that if you really wanted to make the code as closely comparable to C++ as possible, not only would you need all those try/cathes, but you'd be terminating rather than propagating if a dispose let an exception leak through. All of this is beyond the point of his post but only further argues in favor of the approach with destructors. Again, destructors are automatically called whereas "defer" or the dispose pattern of Java and C# require you to be explicit every time a type with non-trivial disposal is instantiated.

*"But we should consider how often they are applicable. Definitely for file I/O, since the most common pattern is to read or write completely. Not at all for locks, because locks are always longer-lived than the scope in which you lock them; so in C++ you need a wrapper class for RAII, and the code is about the same as you'd get with "using". Sometimes for network connections; but often you want to hold on to a connection for a while before tearing it down, and you may want to free it asynchronously, say after it has been idle for a while. You end up with some sort of connection pool, and then I don't think the code looks very different in C++ vs. C#. "*

All of those cases are perfect examples of where destructors are applicable, and in fact I use RAII in those situations all of the time and would certainly not trade it for "defer". In the hypothetical cases you are talking about, for instance where locks are unlocked after the locking object goes out of scope (which I don't know why you claim is always the case, it's actually almost never the case), you are exhibiting times where you either declare the lock object at one scope and lock later on, or, less likely, you want shared ownership of the lock object. With C++ you are able to very easily and automatically handle both cases and still get deterministic unlocking immediately after all of the objects end their lifetime with virtually no chance of error. Further, I'm not sure what you are even trying to point out here since you are providing no example of a better approach with "defer".

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 13, 2009 12:42 PM) \[continued\]

Keep in mind I never said "defer" wasn't useful, only that destructors are much better suited for almost all common cases and are automatic rather than explicit and error prone. The times you'd want to use a "defer" like feature over destructors are when you need to do something unique to that particular scope. That does happen and it is why "defer" is very useful. The two features are in no way mutually exclusive.

Since you seem adamant that RAII is not applicable to threading or networking, please see [Boost.Thread](http://www.boost.org/doc/libs/1_41_0/doc/html/thread.html) and [Boost.ASIO](http://www.boost.org/doc/libs/1_41_0/doc/html/boost_asio.html) (asynchronous input output library), which are both fantastic libraries that correctly make use of RAII like most other C++ programs in the domains where you claim it is generally not applicable.

*"As for Go, I don't see it adopting destructors, because that would lead down the path of constructors, assignment operators and the like, and the Go designers are very clear that they don't want to go down that route. It's plausible that Go will adopt GC finalizers, but beyond that I think we should not expect any help from the language."*

Well then I'm sorry but I feel they are making a very fundamental mistake in the design of the language as I can't condone decisions that make it harder to program correctly rather than easier because they want to make the language "simple". Simplify as much as possible but no further.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 13, 2009 12:44 PM) To Julian:

*"I really don't see the advantage in immediate deallocation of memory. "free" isn't free. You are shuffling memory in itty bitty slices, when a compacting GC could clear and release it in bulk."*

We are not specifically talking about freeing memory, we are talking about releasing important resources, closing files, closing connections, releasing locks, etc.

*"If you need to access a resource which needs cleanup through an interface, perhaps you should have a "Release()" method as part of the interface, which you can defer."*

Again, that is explicit and incorrect by default if not used. Destructors are automatic and correct.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 13, 2009 1:10 PM) Back to Russ, [here](http://www.osl.iu.edu/publications/prints/2005/siek05:_fg_pldi.pdf) is another good paper concerning generic programming. In particular, it talks much more about "concepts".

- [Julian Morrison](http://www.blogger.com/profile/01115506275519545033)(December 13, 2009 1:13 PM) You're complaining it's explicit. I don't think that's a bug, I think it's a feature.

If I am looking at C++ (and to some extent Java, Ruby, etc) code, and I want to ask "where can it return? where can it suddenly decide to do an expensive operation?" The answer is *anywhere*. And if it doesn't do that now, who knows what the next library version will bring?

If I am looking at Go code, and I want to ask the same questions, the answers are: it returns where it says "return", or at the end of the function. It does cleanup at the end of a function in which you said "defer x.Cleanup()". *That's it*. The search can end. No need to dig arbitrarily deep or (as I've had to in Java in a huge codebase) rely blindly on an IDE that can dig for you.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 13, 2009 2:34 PM) *"You're complaining it's explicit. I don't think that's a bug, I think it's a feature."*

What you're saying is it's a "feature" that you can incorrectly use your objects. Call it what you want, that doesn't change what it is. If you want it explicit then fine, go and make it explicit, but also make it a compiler error if you don't have it written. What little have you now gained?

*"If I am looking at C++ (and to some extent Java, Ruby, etc) code, and I want to ask "where can it return? where can it suddenly decide to do an expensive operation?" The answer is anywhere. And if it doesn't do that now, who knows what the next library version will bring?"*

It can go into an "expensive operation" in the same places. When a function is run. The only difference is that functions are correctly run automatically when the lifetime of an object ends. If an essential function that should be run is not run there then that is a mistake. Forcing the programmer to explicitly write a "defer" statement only means that they may forget to or do so incorrectly.

If you're saying the programmer looks for "defer" and the corresponding end of scope, a programmer can just as easily do the same with respect to the instantiation of types. If I see no "defer" when a type other than something like a built-in arithmetic type is instantiated, instead of saying "there's no 'defer' statement, nothing is going on at the end of the function," I'd be more likely to say "there's no 'defer' statement, is disposal trivial or did the programmer forget to write 'defer'." With automatic destruction the correct behavior happens in either case.

There is nothing horribly complicated about this -- what *is* complicated is trying to correctly simulate that behavior explicitly every time you instantiate a type. That is what creates bugs.

Here is a better example since you say:

*"... who knows what the next library version will bring"*

The situation with destructors is both automatic and much safer with respect to future development. Imagine that type "A" has trivial disposal so the programmer decides to have no "defer" statement when it is instantiated. Later on in development -- weeks, months, or years -- the type "A" grows and implementation details change to where it now should close down resources. All of a sudden there should be a deferred disposal when you instantiate "A".

So what happens to that previous code that was written? Since "defer" wasn't used, you now have to go back and insert it everywhere, otherwise nothing is properly cleaned up.

The way around this horrible scenario would be to have made the original type "A" have an empty "dispose" method and the places that instantiate it have a "defer" statement that calls the empty dispose, postulating that sometime in the future "A" may evolve to have nontrivial disposal. If you abide by this pattern you now you have an explicit "defer" everywhere you instantiate a scope-bound instance of your type even though its dispose currently does nothing.

What then is the benefit of that over having it automatic? If every time you instantiate your type you should be defering disposal, you lose absolutely nothing by combining the operations implicitly. You have now made the type impossible to misuse.

This does not even cover places with shared ownership, where disposal happens deterministically after it has no more owners. You still have to do extra management -- management which is done automatically in C++ because of RAII with respect to smart pointers. Doing this with only a feature like "defer" is far from feasible.

It's very easy to stray down the road of oversimplification as many languages do. Everyone wants their language to be simple but you are only losing by taking away features that make tedious and error prone operations automatic. You can have a simple language without it being gimped.

- [Julian Morrison](http://www.blogger.com/profile/01115506275519545033)(December 13, 2009 3:04 PM) "x = NewX(); defer x.Destroy()" is an idiom - the defer immediately following the instantiation. Anyone who's read some example code would be surprised to see no defer. It would be a "loud silence".

There are reasons you might not want to destroy x. It might be captured in a closure. It might be included in a return value. It might be passed down a channel. Seeing no defer, one would know to look for x's escape.

- [Rivorus](http://www.blogger.com/profile/03014588465653251422)(December 13, 2009 3:30 PM) The times you don't want it to be destroyed are when you are logically dealing through indirection. You either have multiple objects "owning" the same object through indirection, possibly with some that reference it weakly, or you may always have exactly one owner where that ownership is traded between objects, etc.

In all of these cases RAII is still perfectly fine and still provides automatic and safe solutions and in all cases you are still trying to achieve the goal of deterministic disposal. What's more is, because of the fact that how life time is managed is a property of the type through which you are achieving the indirection, you provide readers of the code with much more information than "there is no defered dispose, someone else disposes of it", you tell them "this is precisely how lifetime of this object is managed". That is much more telling than the "loud silence" you describe and there's no chance that the programmer simply forgot to dispose.

As horrible as it sounds to those who dislike C++, this is all exemplified by smart pointers.

- [ncm](http://www.blogger.com/profile/00831355954619691739)(December 13, 2009 8:33 PM) By the way, Russ, it weakens the whole discussion to bring up the old "bloat" slander about templates. It wasn't true in the old days (it seemed like it ought to be, and that's good enough for slander), and it isn't true today. Not only "good" linkers fold together duplicate instantiations. *Every* linker does it, and they have since the beginning.

There certainly are plenty of bloated programs about, in Java and C# as much so as anywhere, but the bloat doesn't come from template instantiation. We will always have bloat, because bloat comes from the same place it always has -- bad coders -- and we will always have far too many of them.

- [Jonathan Amsterdam](http://www.blogger.com/profile/00724441893784184431)(December 14, 2009 12:30 PM) @ ncm: Russ mentioned "libraries," not executables. I think the compiler would output multiple instantiations of vector, one per .o, and this would result in the library being larger. However, I don't know why this would be a concern to anyone unless they were actually \*developing\* on an embedded device, rather than just \*deploying\* to one. (I just bought a 1TB disk for $114.)

- [Jonathan Amsterdam](http://www.blogger.com/profile/00724441893784184431)(December 14, 2009 12:49 PM) @ Rivorus:

I agree that in theory destructors are better than defer, but I don't think in practice that you see many abstractable patterns that generically acquire and release a resource, and don't know that it is a resource. For one thing, the constructors of such resource-acquiring classes are particular to each resource (files, db connections, etc.) and are never (well, never say never, but basically never) the no-arg constructor. So I think that in the real world, very little generic code would be unsure about whether to use defer. In theory your way is clearly superior, but is it worth the added complexity in practice? That is what the Go folks are thinking, I believe.

About threading, I think we're misunderstanding each other. I never claimed RAII wasn't useful. Maybe some code would help:

class Counter {

private:

int c;

mutex m;

public:

Counter(): c(0) {}

void inc() { lock(m); c++; }

};

Here there are two objects, the mutex and the lock wrapper, named lock. You need both. My point is that it takes the same amount of brain to write "lock(m)" as it does to write "using (m) {..}" or "m.lock(); defer m.unlock()", so RAII doesn't buy you anything over the explicit way.

- [Tarun Elankath](http://www.blogger.com/profile/16569074120665954195)(September 17, 2010 5:25 PM) I randomly fall across this old post a year later and Rivorus's wonderful arguments and insightful comments have inspired me to brush the dust off my C++ templates book and look for Stepanov's book as well... :)

- Anonymous (January 4, 2011 10:23 PM) Most people that earn money being an affiliate sign up with several Affiliate Programs. In fact, you have to experiment with several before you find those that will make you as much as possible. One of the important aspects to consider elect to promote products as an affiliate is to choose worthwhile products. If you wouldn't buy it or have any use for it chances are your customers won't either. Remember, even though you're selling over the web and not in person, irrespective of whether you truly believe in the products you are promoting will show through in your marketing efforts. Choose products that you truly believe in if you intend to persuade others to buy them.

Regards,

\[url=George (The IT Guy)\[/url\]

- [Carsten Milkau](http://www.blogger.com/profile/10773876428711731299)(May 27, 2011 5:35 AM) You forgot

4\. (The python way) Make types first-class objects. That means, type factories are full-fledged ordinary functions instead of lexical patterns.

It's certainly not fair to compare an interpreted, weakly-typed language to a compiled, strongly-typed one. Still, Go has already gone quite a bit into this direction with the Reflect package.

To put it straight, generics are actually a specialized means for computation on types. There are many cases where they are easy and straightforward, but Python shows that first-class types can do it just as well, and the rapid evolution of the boost libraries shows that there is actually a great demand in general type computation (and also that generics are inferior for this purpose). Compilation is a computation, and it only makes sense to use the same language for compile-time computation as for runtime computation doesn't it?

I don't say this would be easy, not at all. Yet I think following this path would make Go even more useful, productive and original.

- Anonymous (June 6, 2011 7:30 PM) *"x = NewX(); defer x.Destroy()" is an idiom - the defer immediately following the instantiation. Anyone who's read some example code would be surprised to see no defer. It would be a "loud silence".*

Oh, so it's an "idiom" that every object is destroyed when the function that allocated it returns? Do people who write such things actually program for a living? Shudder. I sure hope I'm not using any of their code. The same goes for Jonathan Amsterdam, who commits the even worse sin of disposing of an object allocated in the caller -- and manually, yet; how quickly he forgets that the whole point of defer is to *guarantee* that the object is destroyed when it will no longer be used. Of course, these examples together make the point -- the only way to get it right in all cases is with dtors (RAII).

Also, Jonathan's code example highlights a fatal weakness of Go's interfaces -- they aren't composable. Suppose I want to dispose of r after passing it to a function that expects a Foobler -- how do I declare r to be both

Disposable and a Foobler? I can't ... and I never will be able to, because composability would contradict Go's simple(-minded) design of no type hierarchies, and the type syntax is designed in a rigid way that makes it awkward to add such features.

*Rivorus, I admire your passion but your optimism is misplaced.*

Ditto.

*It is extremely unlikely that the Go designers are unaware of your arguments.*

The evidence strongly suggests the opposite. This very thread, by one of the designers, is counterevidence.

*Is your suggestion that we sit on our hands until then?*

Nice strawman that turns reality on its head. ncm made no such suggestion, but rather drew a conclusion from an observation about Go -- "unfortunate early design choices". It's not just choices seen in the language, but choices *about* the language -- basic design philosophy. It is now mid-2011 with no glimmer of a generic facility. One can predict, from that design philosophy and from seeing Go's incredibly weak version of exceptions, that if it has generics by 2015 they too will be incredibly weak. Getting generics right takes a lot of careful work, by people who actually *believe* in them and are aware of their power and the wealth of accumulated experience in programming language design. Take a look, for instance, at D, Scala, and Jade -- a spinoff of Emerald, which as you know has interfaces much like Go's.

- [dral](http://www.blogger.com/profile/03666405309296128681)(August 22, 2011 9:36 AM) Hi Russ,

Almost two years have passed since this post was published and go still don't have support for generic programming.

What is the status of generics in go today? Is this still something you are thinking about?

It'd be interesting to know what is your conclusion after all this discussion on generics.

Thanks.

- Anonymous (October 25, 2011 5:03 AM) http://www.bluebytesoftware.com/blog/2011/10/23/OnGenericsAndSomeOfTheAssociatedOverheads.aspx

- Anonymous (December 8, 2011 11:09 AM) I don't see any advantage in having generics in Go. It'll make the language more complex without an obvious benefit.
