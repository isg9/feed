---
title: 'research!rsc: B Languages'
url: https://research.swtch.com/b-lang
published: "2008-02-08T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/b-lang
---

# research!rsc: B Languages

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# B Languages Russ Cox February 8, 2008 *research.swtch.com/b-lang* Posted on Friday, February 8, 2008.

I enjoy reading about programming languages of the past. (I would probably enjoy reading about programming languages of the future too, but those are harder to find.) Today's links are a glimpse back into the programming world before types.

The predecessor of the C programming language was, of course, Ken Thompson's B, which was in turn influenced by Martin Richards's BCPL.\*

Thompson's 1972 technical memorandum “ [**Users' Reference to B**](http://cm.bell-labs.com/cm/cs/who/dmr/kbman.pdf)” (PDF, 1MB; also [HTML](http://cm.bell-labs.com/cm/cs/who/dmr/kbman.html)) describes B in detail. The most striking part about B is how similar it is to C. The fundamental difference is the lack of types, or rather the single type: the machine word. As Thompson puts it, “There is no check to insure that there are no type mismatches. Similarly, there are no wanted, or unwanted, type conversions.” To me, the lack of types gives B a flavor distinctly different from C. Having grown up programming with types, being able to dereference any value at all seems like not wearing a seat belt.

The [BCPL manual](http://cm.bell-labs.com/cm/cs/who/dmr/bcpl.html) is also online, courtesy of Dennis Ritchie. It is easy to see B as BCPL filtered through Thompson's minimalist sensibilities. In a [1989 interview](http://www.princeton.edu/~hos/mike/transcripts/thompson.htm), Thompson recounted, “\[B\] was very small, very clean. It was the same language as BCPL, it looked completely different, syntactically it was, you know, a redo. The semantics was exactly the same as BCPL. And in fact the syntax of it was, if you looked at, you didn’t look too close, you would say it was C. Because in fact it was C, without types.”

The B memo also contains other interesting trivia. Thompson defines the newfangled `while` loop in terms of `goto`, an indicator of just how uncommon ‘structured’ programming was at the time. The memo also contains what is probably the first published implementation of `printf`.

In a language whose only type is the machine word, character processing is particularly cumbersome. B included primitives `char` and `lchar` to fetch and store characters from character strings (vectors). On his BCPL history page linked above, Ritchie says that “C was invented in part to provide a plausible way of dealing with characters when one begins with a word-oriented language.”

B had the full suite of the `=` *op* operators (what became *op* `=` in modern C), not just for arithmetical operators like `=+` and `=-` but also for comparison operators: `===`, `=!=`, `=<`, `=<=`, `=>`, and `=>=`. Unfortunately, the end of the manual notes that none of these comparison operators were actually implemented, nor were `=/` and `=&`.

Brian Kernighan's 1973 [B tutorial](http://cm.bell-labs.com/cm/cs/who/dmr/btut.pdf) (PDF; also [HTML](http://cm.bell-labs.com/cm/cs/who/dmr/btut.html)) contains what is probably the very first “hello, world” program. (See page 4 of the PDF.)

\\* There is an old story that at Bell Labs there were arguments over whether the successor to C should be called D (following the alphabet) or P (following the letters in BCPL) and that Bjarne Stroustroup ended the argument by choosing the name C++ instead. In Larry Wall's 1999 talk “ [Perl, the first postmodern computer language](http://www.perl.com/pub/a/1999/03/pm.html),” he said that Perl was the true successor to C, having taken both of the remaining letters from BCPL as its conventional script extension.
