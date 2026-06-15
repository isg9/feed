---
title: Regular expressions in lexing and parsing
url: https://commandcenter.blogspot.com/2011/08/regular-expressions-in-lexing-and.html
published: "2011-08-22T19:36:00-07:00"
feed: robpike
guid: tag:blogger.com,1999:blog-6983287.post-1982234857091660188
---

# Regular expressions in lexing and parsing

Comments extracted from a code review. I've been asked to disseminate them more widely.

I should say something about regular expressions in lexing andparsing. Regular expressions are hard to write, hard to write well,and can be expensive relative to other technologies. (Even when theyare implemented correctly in N\*M time, they have significantoverheads, especially if they must capture the output.)Lexers, on the other hand, are fairly easy to write correctly (if notas compactly), and very easy to test. Consider finding alphanumericidentifiers. It's not too hard to write the regexp (something like"\[a-ZA-Z\_\]\[a-ZA-Z\_0-9\]\*"), but really not much harder to write as asimple loop. The performance of the loop, though, will be much higherand will involve much less code under the covers. A regular expressionlibrary is a big thing. Using one to parse identifiers is like using aMack truck to go to the store for milk. And when we want to adjust our lexer to admit other character types,such as Unicode identifiers, and handle normalization, and so on, thehand-written loop can cope easily but the regexp approach will breakdown.

A similar argument applies to parsing. Using regular expressions toexplore the parse state to find the way forward is expensive,overkill, and error-prone. Standard lexing and parsing techniques areso easy to write, so general, and so adaptable there's no reason touse regular expressions. They also result in much faster, safer, andcompact implementations.

Another way to look at it is that lexers and parsing are matchingstatically-defined patterns, but regular expressions' strength is thatthey provide a way to express patterns dynamically. They're great intext editors and search tools, but when you know at compile time whatall the things are you're looking for, regular expressions bring farmore generality and flexibility than you need.

Finally, on the point about writing well. Regular expressions are, inmy experience, widely misunderstood and abused. When I do code reviewsinvolving regular expressions, I fix up a far higher fraction of theregular expressions in the code than I do regular statements. This isa sign of misuse: most programmers (no finger pointing here, justobserving a generality) simply don't know what they are or how to usethem correctly.Encouraging regular expressions as a panacea for all text processingproblems is not only lazy and poor engineering, it also reinforcestheir use by people who shouldn't be using them at all.

So don't write lexers and parsers with regular expressions as thestarting point. Your code will be faster, cleaner, and much easier tounderstand and to maintain.
