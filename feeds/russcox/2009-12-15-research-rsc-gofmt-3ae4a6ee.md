---
title: 'research!rsc: Gofmt'
url: https://research.swtch.com/gofmt
published: "2009-12-15T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/gofmt
---

# research!rsc: Gofmt

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Gofmt Russ Cox December 15, 2009 *research.swtch.com/gofmt* Posted on Tuesday, December 15, 2009.

One of the more controversial aspects of the Go project is that Go has a canonical program format, implemented by a program ( `gofmt`) that will pick up any Go program and reformat it in the canonical format. Every program in the Go source tree has been formatted with gofmt. The code review and revision control tools check that any new code is gofmt-formatted too.

(I am avoiding the word “style,” because gofmt doesn't enforce style, in the sense of *The Elements of Programming Style*. Running code through gofmt changes the layout but not whether it's a well-written, elegant program. Referring to these mundane formatting rules as style elevates them far beyond their importance. The only style that gofmt imposes is consistency.)

The most obvious benefit of using gofmt is that when you open an unfamiliar Go program, your brain doesn't get distracted, even subconsciously, about why that brace is in the wrong place; you can focus on the code, not the formatting. But there are many more interesting uses for gofmt. Gofmt can take any file in the Go source tree, parse it into an internal representation, and then write exactly the same bytes back out to the file. There are two reasons this works: the primary reason is that a lot of effort went into gofmt, but an important secondary reason is that gofmt only has to worry about one formatting convention, and we've agreed to accept that as the official one.

Once you have a tool that can parse and print a program losslessly, it's easy to insert mechanical processing in the middle, to transform the program between parsing and printing. Want to change something about the language? Change it in gofmt and reformat the source tree. This has already happened multiple times since the public launch.

The December 9 release of Go added a new built-in function `copy(dst, src)` that copies data from one slice to another. (It's a built-in both because it's such a common operation and because Go's lack of pointer arithmetic makes it hard to write the overlap check yourself.) The new `copy` replaced the library routine `bytes.Copy`, which was similar but only worked on byte slices.

Gofmt has an option, `-r`, to specify a rewrite rule in the form `pattern -> replacememt`. In the pattern and replacement, single-letter lower case names serve as wildcards matching arbitrary expressions. Want to convert from `bytes.Copy` to `copy`?

```
gofmt -w -r 'bytes.Copy(d, s) -> copy(d, s)' *.go

```

(The -w flag writes the new version of the program back to the input file.)

The December 9 release also added support for writing `x[i:]` instead of `x[i:len(x)]`. Want to convert programs to use the new syntax when possible?

```
gofmt -w -r 'x[i:len(x)] -> x[i:]' *.go

```

These two minor changes were upstaged by discussion of a larger syntactic change, the elimination of line-ending semicolons. Again, gofmt is crucial to making the switch. The syntax being considered is mostly, but not completely, backwards compatible with the current version of Go. Normally, that would mean having to make the compilers support both syntaxes for some time, during which the compiler might warn about programs depending on the old syntax, expecting users to fix their programs. Instead, having one tool whose job it is to rewrite programs means that all the other tools can focus on other things. Gofmt can fix programs mechanically, and the compilers and other tools can drop the old syntax instead of having to support both.

I'd like to see more tools built using gofmt and the libraries it uses ( `go/parser` and `go/printer`). For example, there's no reason an IDE plugin couldn't sit on the same libraries to build interesting refactoring tools. Gofmt's `-r` is just the beginning. It's going to be exciting.

One final note. Being able to pick up a program and write it back out, preserving comments, is not an easy task in any language. Robert Griesemer deserves all the credit for the huge effort to make gofmt handle real programs so well.

(Comments originally posted via Blogger.)

- [andrew](http://andrew.ducker.org.uk/)(December 15, 2009 9:57 AM) I had to do something similar with SQL - read in a large number of COBOL files, find all the SQL in them, read it in, make a small change and write them back out. Parsing the SQL, changing the expression tree, and then writing it back out turned out to be the simplest way of doing this reliably - because at least then you knew when you'd hit some SQL you didn't understand.

- [ncm](http://www.blogger.com/profile/00831355954619691739)(December 15, 2009 4:10 PM) This is really interesting.

For one thing, it means it's not too late to fix the single biggest syntax design mistake I've noticed so far: prefix dereference operator "\*". Postfix dereference "^" may be the only detail Pascal got right.

- [rog peppe](http://www.blogger.com/profile/00839344798030831980)(December 16, 2009 9:37 AM) gofmt is great, and i love the rewriting rule... but it's not quite a catch-all. i'm not sure, for instance, that there's a way i can transform "range (a)" into "range a"?

- [Russ Cox](http://swtch.com/~rsc/)(December 16, 2009 9:43 AM) You can't yet do the range rewrite because there's no syntax for a statement block wildcard, which you'd need for -r 'range (a) {x} -> range a {x}'. But maybe {x} would be good enough as a wildcard.

You can get rid of all unnecessary parens with -r '(a) -> a'.

- [Benno](http://benno.id.au)(January 8, 2011 6:59 PM) rewrite is very cool. Is there anyway to change a field from unexported to exported?

E.g:

type Foo struct { bar int } -> type Foo struct { Bar int }

Simply replacing bar -> Bar doesn't necessarily work perfectly (if there are other things named 'bar' like variable name or fields in other structs.
