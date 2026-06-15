---
title: 'research!rsc: Generating Good Syntax Errors'
url: https://research.swtch.com/yyerror
published: "2010-01-27T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/yyerror
---

# research!rsc: Generating Good Syntax Errors

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Generating Good Syntax Errors Russ Cox January 27, 2010 *research.swtch.com/yyerror* Posted on Wednesday, January 27, 2010.

One of the great religious debates in compiler writing is whether you should use parser generators like yacc and its many descendants or write parsers by hand, usually using recursive descent. On the one hand, using parser generators means you have a precise definition of the language that you are parsing, and a program does most of the grunt work for you. On the other hand, the proponents of hand-written recursive descent parsers argue that parser generators are overkill, that parsers are easy enough to write by hand, and that the result is easier to understand, more efficient, and can give better error messages when presented with syntactically invalid programs.

Like in most religious debates, the choice of side seems to be determined by familiarity more than anything else. I knew how to use yacc before I knew how to write a parser by hand, which put me solidly on the parser generator side of the fence. Now that I do know how to apply both techniques, I'd still rather use a parser generator. In fact, I've worked on projects where I've written parser generators rather than write a parser by hand. Good notation counts for a lot, as we shall see.

In *[Coders at Work](http://books.google.com/books?id=nneBa6-mWfgC&printsec=frontcover&dq=coders+at+work&ei=RNRfS5fzMIO-zATqi-WlBw&cd=1#v=onepage&q=yacc&f=false)*, Ken Thompson talks to Peter Seibel about *yacc* and *lex*:

> **Seibel:** And are there development tools that just make you happy to program?
>
> **Thompson:** I love yacc. I just love yacc. It just does exactly what you want done. Its complement, lex, is horrible. It does nothing you want done.
>
> **Seibel:** Do you use it anyway or do you write your lexers by hand?
>
> **Thompson:** I write my lexers by hand. Much easier.

Many of the objections raised by the hand-written parser camp are similar to Thompson's objection to lex—the tool doesn't do what you want—but there's no fundamental reason a tool can't. For example, the spurious conflicts that an LALR(1) algorithm produces are definitely hard to understand, but if you replace it with LR(1) or GLR(1), you get a more useful tool. And if you do learn how to work within the LALR(1) algorithm, even yacc is very powerful.

The objection to parser generates that seems to resonate most is that generators like yacc produce inadequate error messages, little more than “syntax error.” Better error messages were one of the key benefits hoped for when [g++ converted](http://gcc.gnu.org/ml/gcc/2000-10/msg00573.html) from a yacc-based C++ parser to a hand-written one (and to be fair, C++ syntax is nearly impossible to parse with any tool; the many special cases cry out for hand-written code). Here the objection seems harder to work around: the parser internally gets compiled into an automaton—usually a big table of numbers—that moves from state to state as it processes input tokens. If at some point it can't find a next state to go to, it reports an error. How could you possibly turn that into a good message?

Clinton Jeffery showed how in a paper published in ACM TOPLAS in 2003 titled “ [Generating LR Syntax Error Messages from Examples](http://people.cs.vt.edu/~haebang//coursework/PL/summary.pdf).” As you can guess from the title, the idea is to introduce a training stage after the parser is generated but before it is used for real. In that stage, you feed examples of syntax errors into the parser and look at what state it ends up in when it detects the error, and maybe also what the next input token is. If the automaton hits an error in that state (and with that input token) during real use, you can issue the message appropriate to the example. The important part is that the parser states are basically an abstract summary of the surrounding context, so it's reasonable to treat errors in a particular state with a single message. For example, suppose you find that this Go program

```
package main

import (
 "fmt",
 "math"
)

```

causes a syntax error at the comma, in state 76. In the parser, that state means that you're in the middle of an import block and expecting to see a string constant. The state abstracts that context, so you'd end up in the same state if you were parsing this erroneous program:

```
package fmt

import ( "strings"; "testing", "xgb" )

```

Having told the parser that the appropriate error message for the first program is “unexpected , during import block,” the parser will use the same message for the second program.

It's an elegant idea, and the implementation can be kept on the side, without adding any complexity to the grammar itself. I heard about this idea through the grapevine years ago (in 2000, I think) but had never gotten a chance to try it until this week.

There are three Go parsers: Ian Lance Taylor wrote a hand-written recursive descent parser in C++ for gccgo, Robert Griesemer wrote a hand-written recursive descent parser in Go ( `import "go/parser"`) for [godoc](http://golang.org/cmd/godoc/) and [gofmt](http://golang.org/cmd/gofmt/), and Ken Thompson wrote a yacc-based parser in C for the gc compiler suite.

On Monday night, I [implemented Jeffery's idea in the gc compiler suite](http://code.google.com/p/go/source/detail?r=7427b07b504271532d96c630d3dc37aef4d06c7d). The gc compilers use bison, the GNU implementation of yacc. Bison doesn't support this kind of error management natively, but it is not hard to adapt. Changing bison would require distributing a new version of bison; instead, my implementation post-processes bison's output.

The examples are in a new file `go.errors`. Because the lexer is written by hand, it's not available in the simulation, so the examples are token name sequences rather than actual program fragments. In token lists, the program above and its corresponding error message are:

```
% loadsys package LIMPORT '(' LLITERAL import_package import_there ','
"unexpected , during import block",

```

Understanding the tokens on the first line requires knowing what the various grammar token names mean, but they basically mimic the syntax, and this tool is targeted at the people working on the grammar anyway. An [awk program](http://code.google.com/p/go/source/browse/src/cmd/gc/bisonerrors?spec=svn7427b07b504271532d96c630d3dc37aef4d06c7d&r=7427b07b504271532d96c630d3dc37aef4d06c7d) loads bison's tables from its debugging dump `y.output` and then processes the `go.errors` file, replacing each line like the above with the number of the offending state and input token. That produces a table that can be linked into the compiler and consulted when a syntax error is encountered. If the state and input token match an entry in the table, the better error is used instead of a plain `syntax error`.

That was an outsized amount of work to [close one bug](http://code.google.com/p/go/issues/detail?id=522), but now it's easy to add better messages for other common situations. For example, the fact that only the short `:=` declaration form is allowed in `for` initializers sometimes trips up new Go programmers. When presented with this program:

```
package main

import "fmt"

func main() {
 fmt.Printf("hello, world\n")
 for var i = 0; i < 10; i++ {
  fmt.Println(i)
 }
}

```

the compiler used to just say there was a syntax error on line 7, but now it explains:

```
$ 6g x.go
x.go:7: syntax error: var declaration not allowed in for initializer

```

because of this stanza in `go.errors`:

```
% loadsys package imports LFUNC LNAME '(' ')' '{' LFOR LVAR LNAME '=' LNAME
"var declaration not allowed in for initializer",

```

Gccgo and gofmt give more traditional token-based errors:

```
$ gccgo x.go
x.go:7:6: error: expected operand
...

$ gofmt x.go
x.go:7:6: expected operand, found 'var'

```

What's interesting isn't that neither has bothered to handle this as a special case yet. What's interesting is to consider the work required to do so. Changing either would require understanding the corresponding hand-written parser well enough to find the right place to put the special case. With the example-based approach, you just write an example, and the tool figures out where in the parser it needs to go.

(Comments originally posted via Blogger.)

- [Barry Kelly](http://www.blogger.com/profile/10559947643606684495)(January 27, 2010 6:36 PM) Another argument for hand-writing your parser is when extending the language in ways that create syntax ambiguities that are resolved using semantic tests. Parser generators that have semantic predicates do exist, but there comes a point when you need to have a really intimate knowledge of exactly what the compiler compiler is doing and what output it's going to generate, so that you can create the right input to generate just so. At that point, the compiler compiler isn't saving you much work.

The objection I have to compiler compilers is more along these lines. It's the ickiness of having little pieces of code being pasted into a template by a tool that has no semantic knowledge of that code, much like a dumb string processor, and having to deal with the auto-generated code. Making sure that the right pieces of text flow to the right locations starts feeling like a plumbing job, where a tool is doing the pipe layout, but you still need to be a plumber.

The best of both worlds, if it can be called that, would be something along the lines of an embedded DSL. Consider parser combinators in functional programming languages, for example. An idiom where you can both use a concise notation for the syntax, but also in the same language that's used for semantic checking and processing, etc. This would be my preferred model. It doesn't work out particularly well in C or C++, though, hacks like Boost Spirit aside.

- [AKS](http://www.blogger.com/profile/16120300424669884662)(January 28, 2010 3:47 AM) Another question I had on using parser generator is about languages that evolve over time.

for eg. version 2 of the language might introduce a new keyword and that keyword was an identifier in the original language.

Now we have two slightly different languages to parse (we , of course assume that we are not abandoning all the code written in version 1 of the language)

After several such language iterations , wouldn't the parser generator "input file" would be difficult to maintain.

or we might be forced to maintain multiple versions of the parser.

An error in one version of language might not be an error in another version of language.

- [LKRaider](http://www.blogger.com/profile/15376943686783326713)(February 3, 2010 8:53 AM) "An error in one version of language might not be an error in another version of language."

That how it should be. Or would you prefer a code full of exceptional cases to maintain, each version getting more intricate and complex? Doesn't seem like a good way forward to me.
