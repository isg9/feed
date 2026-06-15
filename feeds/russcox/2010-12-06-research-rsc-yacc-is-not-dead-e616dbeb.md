---
title: 'research!rsc: Yacc is Not Dead'
url: https://research.swtch.com/yaccalive
published: "2010-12-06T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/yaccalive
---

# research!rsc: Yacc is Not Dead

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Yacc is Not Dead Russ Cox December 6, 2010 *research.swtch.com/yaccalive* Posted on Monday, December 6, 2010.

The internet tubes have been filled with chatter recently about a paper posted on arXiv by Matthew Might and David Darais titled “ [Yacc is dead.](http://arxiv.org/abs/1010.5023)” Unfortunately, much of the discussion seems to have centered on whether people think yacc is or should be dead and not on the technical merits of the paper.

Reports of yacc's death, at least in this case, are greatly exaggerated. The paper starts by arguing against the “cargo cult parsing” of putting together parsers by copying and pasting regular expressions from other sources until they work. Unfortunately, the paper ends up being an example of a much worse kind of cargo cult: claiming a theoretical grounding without having any real theoretical basis. That's a pretty strong criticism, but I think it's an important one, because the paper is repeating the same mistakes that led to [widespread exponential time regular
expression matching.](http://swtch.com/~rsc/regexp/regexp1.html) I'm going to spend the rest of this post explaining what I mean. But to do so, we must first review some history.

Grep and yacc are the two canonical examples from Unix of good theory—regular expressions and LR parsing—producing useful software tools. The good theory part is important, and it's the reason that yacc isn't dead.

### Regular Expressions

The first seminal paper about processing regular expressions was Janus Brzozowski's 1964 CACM article “ [Derivatives of regular expressions](http://portal.acm.org/citation.cfm?id=321249).” That formulation was the foundation behind Ken Thompson's clever on-the-fly regular expression compiler, which he described in the 1968 CACM article “ [Regular expression search algorithm](http://portal.acm.org/citation.cfm?id=363387).” Thompson learned regular expressions from Brzozowski himself while both were at Berkeley, and he credits the method in the 1968 paper. The now-standard framing of Thompson's paper as an NFA construction and the addition of caching states to produce a DFA were both introduced later, [by Aho and Ullman in their various textbooks](http://swtch.com/~rsc/regexp/regexp2.html#attrib). Like many primary sources, the original is different from the textbook presentations of it. (I myself am [also guilty](http://swtch.com/~rsc/regexp/regexp1.html) of this particular misrepresentation.)

The 1964 theory had more to it than the 1968 paper used, though. In particular, because each state computed by Thompson's code was ephemeral—it only lasted during the processing of a single byte—there was no need to apply the more aggressive transformations that Brzozowski's paper included, like rewriting `x|x` into `x`.

A much more recent paper by Owens, Reppy, and Turon, titled “ [Regular-expression derivatives reexamined,](http://www.ccs.neu.edu/home/turon/re-deriv.pdf)” returned to the original Brzozowski paper and considered the effect of using Brzozowski's simplification rules in a regular expression engine that cached states (what textbooks call a DFA). There, the simplification has the important benefit that it reduces the number of distinct states, so that a lexer generator like lex or a regular expression matcher like grep can reduce the necessary memory footprint for a given pattern. Because the implementation manipulates regular expressions symbolically instead of the usual sets of graph nodes, it has a very natural expression in most functional programming languages. So it's more efficient and easier to implement: win-win.

### Yacc is dead?

That seems a bit of a digression, but it is relevant to the “Yacc is dead” paper. That paper, claiming inspiration from the “Regular-expression derivatives reexamined” paper, shows that since you can define derivatives of context free grammars, you can frame context free parsing as taking successive derivatives. That's definitely true, but not news. Just as each input transition of an NFA can be described as taking the derivative (with respect to the input character) of the regular expression corresponding to the originating NFA state, each shift transition in an LR(0) parsing table can be described as taking the derivative (with respect to the shift symbol) of the grammar corresponding to the originating LR(0) state. A key difference, though, is that NFA states typically carry no state with them and can have duplicates removed, while a parser associates with each LR(0) parse states a parse stack, making duplicate removal more involved.

### Linear vs Exponential Runtime

The linear time guarantee of regular expression matching is due to the fact that, because you can eliminate duplicate states while processing, there is a finite bound on the number of NFA states that must be considered at any point in the input. If you can't eliminate duplicate LR(0) states there is no such bound and no such linear time guarantee.

Yacc and other parser generators guarantee linear time by requiring that the parsing state automata not have even a hint of ambiguity: there must never be a (state, input symbol) combination where multiple possible next states must be explored. In yacc, these possible ambiguities are called shift/reduce or reduce/reduce conflicts. The art of writing yacc grammars without these conflicts is undeniably arcane, and yacc in particular does a bad job of explaining what went wrong, but that is not inherent to the approach. When you do manage to write down a grammar that is free of these conflicts, yacc rewards you with two important guarantees. First, the language you have defined is unambiguous. There are no inputs that can be interpreted in multiple ways. Second, the generated parser will parse your input files in time linear in the length of the input, even if they are not syntactically valid. If you are, say, defining a programming language, these are both very important properties.

The approach outlined in the “Yacc is dead” paper gives up both of these properties. The grammar you give the parser can be ambiguous. If so, it can have exponentially many possible parses to consider: they might all succeed, they might all fail, or maybe all but the very last one will fail. This implies a worst-case running time exponential in the length of the input. The paper makes some hand-wavy claims about returning parses lazily, so that if a program only cares about the first one or two, it does not pay the cost of computing all the others. That's fine, but if there are no successful parses, the laziness doesn't help.

It's easy construct plausible examples that take exponential time to reject an input. Consider this grammar for unary sums:

```
S → T
T → T + T | N
N → 1

```

This grammar is ambiguous: it doesn't say whether 1+1+1 should be parsed as (1+1)+1 or 1+(1+1). Worse, as the input gets longer there are exponentially many possible parses, [approximately O(3n) of them](http://oeis.org/A000081). If the particular choice doesn't matter, then for a well-formed input you could take the first parse, whatever it is, and stop, not having wasted any time on the others. But suppose the input has a syntax error, like 1+1+1+...+1+1+1++1. The backtracking parser will try all the possible parses for the long initial prefix just in case one of them can be followed by the erroneous ++1. So when the user makes a typo, your program grinds to a halt.

For an example that doesn't involve invalid input, we can use the fact that any regular expression can be translated directly into a context free grammar. If you translated [a test case
that makes Perl's backtracking take exponential time before finding a match](http://swtch.com/~rsc/regexp/regexp1.html) into a grammar, the translated case would make the backtracking parsers in this paper take exponential time to find a match too.

Exponential run time is a [great name for a
computer science lab's softball team](http://galleries.csail.mit.edu/main.php?g2_itemId=7333) but in other contexts is best avoided.

### Approaching Ambiguity

The “Yacc is dead” paper observes correctly that ambiguity is a fact of life if you want to handle arbitrary context free grammars. I mentioned above the benefit that if yacc accepts your grammar, then (because yacc rejects all the ambiguous context free grammars, and then some), you know your grammar is unambiguous. I find this invaluable as a way to vet possible language syntaxes, but it's not necessary for all contexts. Sometimes you don't care, or you know the language is ambiguous and want all the possibilities. Either way, you can do better than exponential time parsing. Context free parsing algorithms like the [CYK algorithm](http://en.wikipedia.org/wiki/CYK_algorithm) or the [Generalized LR parsing](http://en.wikipedia.org/wiki/GLR_parser) can build a tree representing all possible parses in only O(n3) time. That's not linear, but it's a far cry from exponential.

Another interesting approach to handling ambiguity is to give up on context free grammars entirely. Parsing expression grammars, which can be parsed efficiently using [packrat parsing](http://pdos.csail.mit.edu/~baford/packrat/), replace the alternation of context free grammars with an ordered (preference) choice, removing any ambiguity from the language and achieving linear time parsing. This is in some ways the same approach yacc takes, but some people find it easier to write parsing expression grammars than yacc grammars.

These are basically the only two approaches to ambiguity in context free grammars: either handle it (possible in polynomial time) or restrict the possible input grammars to ensure linear time parsing and as a consequence eliminate ambiguity. The obvious third approach—accept all context free grammars but identify and reject just the ambiguous ones—is an undecidable problem, equivalent to [solving the halting problem](http://www.ling.ed.ac.uk/~gpullum/loopsnoop.html).

### Cargo cults

The “Yacc is dead” paper begins with a scathing critique of the practice of parsing context free languages with (not-really-)regular expressions in languages like Perl, which Larry Wall apparently once referred to as “cargo cult parsing” due to the heavy incidence of cut and paste imitation and copying “magic” expressions. The paper says that people abuse regular expressions instead of turning to tools like yacc because “regular expressions are \`WYSIWYG'—the language described is the language that gets matched—whereas parser-generators are WYSIWYGIYULR(k)—\`what you see is what you get if you understand LR(k).' ”

The paper goes on:

> To end cargo-cult parsing, we need a new approach to parsing that:
> - handles arbitrary context-free grammars;
>
> - parses efficiently on average; and
>
> - can be implemented as a library with little effort.

Then the paper starts talking about the “Regular-expression derivatives reexamined” paper, which really got me excited. I hoped that just like returning to derivatives led to an easy-to-implement and oh-by-the-way more efficient approach to regular expression matching, I had high hopes that the same would happen for context free parsing. Instead, the paper takes one of the classic blunders in regular expression matching— [search via
exponential backtracking because the code is easier to write](http://swtch.com/~rsc/regexp/regexp1.html)—and applies it to context free parsing.

Concretely, the paper fails at goal #2: “parses efficiently on average.” Most arbitrary context-free grammars are ambiguous, and most inputs are invalid, so most parses will take exponential time exploring all the ambiguities before giving up and declaring the input unparseable. (Worse, the parser couldn't even parse an unambiguous S-expression grammar efficiently without adding a special case.)

Instead of ending the supposed problem of cargo cult parsing (a problem that I suspect is endemic mainly to Perl), the paper ends up being a prime example of what Richard Feynman called “ [cargo cult science](http://en.wikipedia.org/wiki/Cargo_cult_science)” (Larry Wall was riffing on Feynman's term), in which a successful line of research is imitated but without some key aspect that made the original succeed (in this case, the theoretical foundation that gave the efficiency results), leading to a failed result.

That criticism aside, I must note that the Haskell Scala code in the paper really does deliver on goals #1 and #3. It handles arbitrary context-free grammars, it takes little effort, and on top of that it's very elegant code. Like that Haskell Scala code, the C regular expression matcher in [this article](http://www.ddj.com/architect/184410904) (also Chapter 9 of [*The Practice of Programming*](http://www.amazon.com/dp/020161586X) and Chapter 1 of [*Beautiful Code*](http://www.amazon.com/dp/0596510047)) is clean and elegant. Unfortunately, the use of backtracking in both and the associated exponential run time makes the code much nicer to study than to use.

### Is yacc dead?

[Yacc](http://dinosaur.compilertools.net/) is an old tool, and it's showing its age, but as the Plan 9 manual says of [*lex*(1)](https://9p.io/magic/man2html/1/lex), “The asteroid to kill this dinosaur is still in orbit.” Even if you think yacc itself is unused (which is far from the truth), the newer tools that provide compelling alternatives still embody its spirit. [GNU Bison](http://www.gnu.org/software/bison/) can optionally generate a GLR parser instead of an LALR(1) parsers, though Bison still retains yacc's infuriating lack of detail in error messages. (I use an awk script to parse the bison.output file and tell me what really went wrong.) [ANTLR](http://www.antlr.org/) is a more full featured though less widespread take on the approach. These tools and many others all have the guarantee that if they tell you the grammar is unambiguous, they'll give you a linear-time parser, and if not, they'll give you at worst a cubic-time parser. Computer science theory doesn't know a better way. But any of these is better than an exponential time parser.

So no, yacc is not dead, even if it sometimes smells that way, and even if many people wish it were.

(Comments originally posted via Blogger.)

- [shaunxcode](http://www.blogger.com/profile/11443533397239445174)(December 6, 2010 8:48 AM) I believe the parser code in the paper is actually written in Scala and not Haskell.

- [gregturn](http://gregturn.myopenid.com/)(December 6, 2010 9:08 AM) If you need something newer (and slicker) check out PLY (Python Lex Yacc). I reverse engineered a grammar using this in order to create a converter.

http://www.dabeaz.com/ply/

- [Russ Cox](http://swtch.com/~rsc/)(December 6, 2010 10:04 AM) @shaunxcode: Yes, Scala not Haskell. Confused by the laziness. The Haskell code is elsewhere. Fixed, thanks.

- [deeelwy](http://www.blogger.com/profile/11968251961041194013)(December 6, 2010 1:13 PM) What do you think of Marpa by Jeffrey Kegler? It's available at http://search.cpan.org/perldoc?Marpa , and ironically implemented in Perl. It's based on Earley's parser, which seems like a really cool design. See his blog at http://blogs.perl.org/users/jeffrey\_kegler/archives.html for more details. Also, the acknowledgments (http://search.cpan.org/~jkegl/Marpa-0.200000/lib/Marpa/Doc/Marpa.pod#ACKNOWLEDGMENTS) section of the documentation includes a link to Earley's paper Marpa's algorithm is based on.

- (December 6, 2010 1:26 PM) I'm a big Yacc fan, and I wouldn't call it dead, but I'd say it's

basically unused for parsing.

By "parsing" I mean, very broadly, the problem of extracting structure

from a steam of bytes---text processing. This kind of parser is

something that programmers are writing all the time, and, in the vast

majority of cases, they do not use parser generators like Yacc to do

so. Instead they are using regexp libraries and other tools.

By "unused," I mean that the ratio of the number of Yacc/ANTLR/Peg

grammars to the number of Awk/Perl/Python text processing scripts is

approximately zero, and this ratio is decreasing over time. Yacc's

usage is vanishingly small.

It's interesting to consider why that is, given the purported

advantages of the Yacc approach. My own intuition is that the

languages that people want to parse simply cannot be expressed as

unambiguous context-free grammars. To the extent that Yacc is used,

it is because Yacc, the tool, in fact does something subtly different

than simply parse according to a grammar.

For example, many programming languages have if-then-else syntax, and

the usual grammar for if-then-else is ambiguous, leading to the

classic shift-reduce conflict. There is a way to write an unambiguous

grammar for if-then-else, but in practice people don't bother with

this. Instead, they just use the ambiguous grammar and rely on Yacc

to resolve the ambiguity by shifting instead of reducing.

I think if you examine the Yacc grammars that people have written,

you'll find that most of them have shift-reduce or reduce-reduce

conflicts: they aren't LR(1) and are often ambiguous. Conflicts get

resolved automatically by Yacc or by semi-magical annotations added by

the programmer. These grammars are still useful, of course, but I for

one don't claim to understand them well.

In theoretical terms, an LR(1) parser generator can take any

context-free grammar G, regardless of whether it is LR(1) or

ambiguous, and produce a parser P. However, when there are conflicts

in G (the usual case), the language L(P) of the parser may NOT be the

same as the language L(G) of the grammar. (I'm not even sure that

L(P) is context-free in the general case.)

Now if-then-else doesn't prove my intuition because it is still

context-free. However, most programming languages have

context-sensitive features. This includes C/C++ (because of

typedefs), Python and Haskell (significant indentation), Ruby

(here-documents), Perl (obvious), Javascript (optional line-ending

semicolons), Standard ML (user-defined infix operators). There are

ways to push these through Yacc---the so-called "lexer hacks"---but

these are hardly theoretically satisfying.

If "The good theory part is important," then we should be working to

improve on Yacc, and, in particular, context-free languages may not be

the best starting point.

\[Posting for and with permission of Trevor Jim. -rsc\]

- Anonymous (December 6, 2010 8:32 PM) >These are basically the only two approaches to ambiguity in context free grammars

The Lua parser is different. The Lua grammer is ambiguous (wierd edge cases involving function calls with newlines) but it complains when the parser encounters an ambiguity. Not so much the 'accept grammers' part, as it is a custom parser.

Dunno if this approach could be generalised to parser generators, but it is intresting, as it means the designer can effectively ignore edge cases by making them illegal.

- Anonymous (December 6, 2010 8:33 PM) I love your analysis. BTW, I notice at http://bit.ly/eSNpw1 that Spiewalk says he has posted a comment on your analysis, but the comment was rejected. In the interest of science, would you please publish his comment? Of course, I will also be interested in your response. FYI, Spiewalk's comment is lodged at http://bit.ly/gpVqbL.

- (December 6, 2010 8:39 PM) So, I promised myself that I wasn't going to comment on this, but it turns out that I can't resist...

There are a few errors in this post which I feel the need to address. Specifically dealing with derivative parsing, the article makes the claim (and uses that claim to support several arguments) that the parsing algorithm outlined in the "Yacc is dead" paper is exponential due to backtracking. This is not the case. Granted, it's not so easy to see this due to a lack of formal complexity analysis in the paper, but the algorithm doesn't do \*any\* backtracking whatsoever. Once it consumes a token, it moves on and never returns.

However, the algorithm \*is\* exponential. More specifically, the algorithm presented in the paper is O(k^n) where k is the number of distinct terminals used in the input text. Actually, the bound is a bit less than that, but it's simpler just to say O(k^n). The reason for this exponentiality has nothing to do with backtracking, it has to do with right recursion. Consider the following grammar:

X ::= X a

\| b X

\| c

When we derive X with respect to b, the result will include \*all\* of X (underived), as well as \*all\* of X derived with respect to b. If we continue deriving with alternations of a and b, the resulting derivative grammars grow exponentially with a factor of 2^n until all possible ascending substrings of the input stream have been generated and memoized. However, by that point, the grammar is absolutely enormous. And, as the derivation is linearly proportional to the size of the grammar, each successive derivative requires exponentially more time to complete. Multiply that by n derivatives, and you have the upper-bound.

Note that this has nothing to do with "exploring all alternatives" for a particular prefix, nor does it say anything about valid vs invalid input. This grammar will be exponential on \*any\* input, valid or otherwise.

Of course, whether or not the algorithm is backtracking has very little bearing on the main points of your analysis. After all, it's still exponential, and exponentiality is bad. So, it's really just splitting hairs, but I thought it was important to clarify.

In other news, you make some hand-wavy claims about an "exponential number of parses" directly implying "exponential runtime". This is not strictly true. Generalized parsing algorithms (which, ironically, you bring up) have been solving this problem for years by memoizing shared prefixes and suffixes of alternate parse trails (e.g. with Tomita's graph-structured stack). It turns out that if you don't require all possible \*results\*, you can get a "yes/no" recognition of any given input string in O(n^3). If you actually need every result though, then clearly it's going to be exponential (since there may be an exponential number of results, and you have to enumerate them all). In practice, practical tools like SGLR solve this problem of exponentiality in the number of \*results\* by generating a shared packed parse forest, which is basically just a lazy set of parse results. This, incidentally, touches on another slightly-incorrect statement in the article, which claims that GLR can "build a tree representing all possible parses in O(n^3) time". Technically, it's not building the whole tree. If it were, then it would be exponential. It's just building \*one\* tree and saving just enough information that it can build the rest if need-be.

Critically though, for a proper O(n^3) generalized parsing algorithm like CYK, RINGLR (GLR isn't actually generalized), GLL and similar, both valid \*and\* invalid strings can be recognized (or rejected) in O(n^3) time.

Anyway, the algorithm in the "Yacc is dead" paper \*is\* exponential, and most of the statements in the article are quite valid. However, the analysis of generalized parsing in general (no pun intended) is somewhat misleading and casts undue aspersions on the field as a whole.

- [Russ Cox](http://swtch.com/~rsc/)(December 6, 2010 8:40 PM) \> So, I promised myself that I wasn't going to comment on this, but it turns

\> out that I can't resist...



\> There are a few errors in this post which I feel the need to address.

\> Specifically dealing with derivative parsing, the article makes the claim

\> (and uses that claim to support several arguments) that the parsing

\> algorithm outlined in the "Yacc is dead" paper is exponential due to

\> backtracking. This is not the case. Granted, it's not so easy to see this

\> due to a lack of formal complexity analysis in the paper, but the algorithm

\> doesn't do \*any\* backtracking whatsoever. Once it consumes a token, it

\> moves on and never returns.

\> ...

I looked at the paper again and I might agree, but it's hard to say.

The code doesn't actually show what is lazy and what is not.

I interpreted the sentence about laziness and recursive grammars

on the second page of Section 9 as saying that even the rest of

the derivative wasn't computed until the first part had been used up,

so that the lazy computation would be like a frozen backtracking

waiting to be unthreaded. Your interpretation - that the derivative

is somewhat less lazy than I thought - seems equally plausible.

In that case the parser would be even less efficient than claimed,

since it would be doing more work than necessary to find the first

parse result.

\> In other news, you make some hand-wavy claims about an "exponential number

\> of parses" directly implying "exponential runtime". This is not strictly

\> true. Generalized parsing algorithms (which, ironically, you bring up) have

\> been solving this problem for years by memoizing shared prefixes and

\> suffixes of alternate parse trails (e.g. with Tomita's graph-structured

\> stack). It turns out that if you don't require all possible \*results\*, you

\> can get a "yes/no" recognition of any given input string in O(n^3). If you

\> actually need every result though, then clearly it's going to be exponential

\> (since there may be an exponential number of results, and you have to

\> enumerate them all). In practice, practical tools like SGLR solve this

\> problem of exponentiality in the number of \*results\* by generating a shared

\> packed parse forest, which is basically just a lazy set of parse results.



\> This, incidentally, touches on another slightly-incorrect statement in the

\> article, which claims that GLR can "build a tree representing all possible

\> parses in O(n^3) time". Technically, it's not building the whole tree. If

\> it were, then it would be exponential. It's just building \*one\* tree and

\> saving just enough information that it can build the rest if need-be.

I think we agree. I have used GLR parsers in the past that included

merge nodes to mark ambiguities in the resulting parse tree.

That is what I meant by "a tree representing all possible parses

in O(n^3) time." It can build \*one\* tree that contains \*all\* the information.

On the other hand, if the API is to return a lazy stream of parse

trees that enumerates all the possible parse trees (as I believe

the API in the paper does) then any use of it to collect an ambiguous

parse is necessarily exponential just to get to the end of the stream.

It was that API that I intended when talking about exponential numbers

of parses implying exponential time.

It was definitely not my intent to cast undue aspersions on generalized

parsing as a whole. In fact quite the opposite: I was trying to say that

GLR is still a far better alternative than exponential time parsing.

- (December 6, 2010 8:42 PM) \> In that case the parser would be even less efficient than claimed,

\> since it would be doing more work than necessary to find

\> the first parse result.

I'm not sure I follow on that score. It's really just an eagerly-evaluated, breadth-first traversal of a top-down parse trail. This indicates that it should require potentially less work to find the first recognition point than a standard depth-first top-down traversal (e.g. recursive descent).

- [Russ Cox](http://swtch.com/~rsc/)(December 6, 2010 8:43 PM) \[I have, with Daniel Spiewak's permission, pasted some emails we exchanged into the comment feed. I apologize for any trouble people are having with Blogger. I believe I have set up the blog to allow anonymous posting, though there is a 4096 character limit. If you post a big comment and get a Request-URI too large error, your comment made it through, believe it or not.\]

- [Russ Cox](http://swtch.com/~rsc/)(December 6, 2010 9:00 PM) \> I'm not sure I follow on that score. It's really just

\> an eagerly-evaluated, breadth-first traversal of

\> a top-down parse trail. This indicates that it

\> should require potentially less work to find the

\> first recognition point than a standard

\> depth-first top-down traversal (e.g. recursive descent).

Dropping down to regular expressions for a bit...

In regular expression search, the backtracking has no memory and might end up tracing a particular subpath through the square many times before exhausting its possibilities. Breadth-first traversal wins (in the worst case) not because of the order in which it visits things but because it guarantees not to duplicate effort. The breadth-first traversal pays extra up front cost (maintaining all the secondary possibilities that often don't end up coming into play) to avoid the worst case possibility. It can't keep up with backtracking's best case, because the best case is that backtracking cuts corners *and gets away with it*, not ever needing the work that it skipped.

That is, the breadth-first traversal itself is typically a loss compared to the depth-first one (it does more work than strictly necessary) but it makes up for that by never doing the same work twice.

An even faster search is to use the depth-first traversal but augment it with a bitmap of visited \[string position, NFA state\] pairs to avoid doing the same work twice. RE2 does this for small searches (when the bitmap is not too expensive) and it beats the pants off of the breadth-first traversal.

Moving back up to parsing...

Everything I said about depth-first beating breadth-first when ignoring duplication of work still applies here. But the really bad part is that, unlike in the regular expression case, the breadth first traversal has no compensating factor to push it ahead: it cannot collapse duplicate states, because the associated parse values may be different, so the breadth-first state grows without bound, to no useful purpose if the first choice ends up succeeding.

A GLR parser \*can\* collapse the states and record a "merge" node or some such in the parse value, and in that case breadth-first traversal comes out ahead again. But the Yacc is Dead parsing has no such merge nodes, so if it's doing an eager breadth-first traversal instead of a lazy depth-first one, it's computing all the possible parse trees in parallel before returning any of them, which would be what I called "more work than necessary to find the first parse result".

- [scw](http://www.blogger.com/profile/15048903730060869199)(December 7, 2010 10:56 PM) It may also be of use to mention [LEMON](http://www.hwaci.com/sw/lemon/) in the context of parser error checking; it seems to do a nice job.

- (December 22, 2010 8:58 PM) Jim Trevor reportedly wrote,

*If "The good theory part is important," then we should be working to*

*improve on Yacc, and, in particular, context-free languages may not be*

*the best starting point.*

after musing on why so much text-processing is done with RE's instead of yacc/bison/etc.

The answer may be in part that (1) RE's are baked into popular scripting languages with shallow learning curves and (2) no one has made the problem of using yacc easier than understanding LR.

But! you say, you cannot get linear time for free: the ability of yacc to detect ambiguity is the means and understanding LR is the price. And LL systems like ANTLR simply invert the pros/cons. That's great for flamewars but not real progress.

This is precisely what the PEG folks are challenging. Do we really need machines to understand CFG's? Is that even a good idea? We can debate that. (And we do.) But if the answer to either question **might** be "no" for any useful case, then we need to look at PEGs and other systems which reject the premise of the necessity of CFG. And now that PEG's can support mutual left-recursion (e.g. OMeta) and even higher-order left-recursion (thanks to Sérgio Queiroz de Medeiros) we really can work on making parser generators better than yacc -- at least for some applications which happen to be very useful.

- [Rajeev](http://www.blogger.com/profile/05853572318439263672)(March 19, 2011 7:12 AM) it was a good post.....http://www.hordelevelingguidewow.com/
