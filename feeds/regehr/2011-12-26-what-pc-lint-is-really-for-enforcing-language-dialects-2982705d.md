---
title: 'What PC-lint is Really For: Enforcing Language Dialects'
url: https://blog.regehr.org/archives/659
published: "2011-12-26T05:56:15Z"
feed: regehr
guid: http://blog.regehr.org/?p=659
---

# What PC-lint is Really For: Enforcing Language Dialects

[John Carmack’s piece](http://altdevblogaday.com/2011/12/24/static-code-analysis/) about using static analyzers is really nice, and he’s right that applying PC-lint to a significant piece of C code results in page after page of warnings. Faced with this problem on a code base I cared about, and not wanting to give up, I first created a config file for PC-lint that suppressed the vast majority of warnings and then, over a few weeks, cleaned up the remaining issues. Once the code base linted clean, I could turn on classes of warnings whenever I had the time and inclination.

As an example, PC-lint supports fine-grained application of stronger typing rules. So one day I might assume that all uses of “bool” or some other enumerated type are segregated from uses of integers. Of course, in any real code base, such an assumption is wrong and a pile of annoying warnings ensues. I’d either fix them or else decide that it wasn’t worth it until some major rearchitecting could be done.

The cool thing was that after a while I realized that I wasn’t really writing C code anymore, but rather a much more strongly typed dialect where I actually had a degree of (justifiable) confidence that certain kinds of errors were not being made. The C compiler, with its near-nonexistent error checking, was acting as a mere code generator. It wasn’t even the case that PC-lint was uncovering a lot of serious bugs in my code (though it did find a few). Rather, it gave me some peace of mind and assured me that what I thought was happening in the code was in fact happening.

Of course, we always write code in dialects, whether we realize it or not. In many cases, language dialects are formalized as coding conventions or standards so a group of people can all read or write the same dialect — this has enormous advantages. The things that I found charming about PC-lint are that (1) it supported a very flexible range of dialects and (2) it quickly and mechanically ensured that my code stayed within the dialect. When writing non-C code (especially Perl) I often wish for a similar dialect checker.
