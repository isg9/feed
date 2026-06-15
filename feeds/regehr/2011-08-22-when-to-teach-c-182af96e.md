---
title: When to Teach C++?
url: https://blog.regehr.org/archives/579
published: "2011-08-22T05:17:26Z"
feed: regehr
guid: http://blog.regehr.org/?p=579
---

# When to Teach C++?

Some friends and I were recently debating when CS undergrads should be taught C++. People have various opinions, but it’s clear that C++ no longer enjoys the somewhat dominant position that it did a number of years ago. My own opinions are rooted in an experience I had around 1995 as a TA for a course that taught C++ to first-year CS students. I didn’t think it worked very well.

We don’t need to teach students every industrially relevant language because any competent computer scientist can rapidly pick up a new language. Rather, a language that is used for coursework should have some pedagogical value. For example, Java and C# are reasonable languages for teaching OO design. Using C is a good way to teach systems-level concepts. Using Scheme / Perl / Python / etc. is a good way to teach algorithms, program design, and the scripting style of development.

Does C++ have pedagogical value? Certainly it can be used to teach OO and systems, but should it be? My general opinion is that it should not be — the overhead is too high. First, the language is just very large. This can be somewhat finessed by teaching a subset, but even so there are difficult feature interactions that can trip up students. Second, the near complete lack of language-level runtime error checking makes C++ a pedagogical disaster (of course the same is true for C, but the language is far more manageable). Finally, in my experience the quality of compile-time diagnostics is not that great. A programmer I know has about a 10-page printout on his cubicle wall that contains a single compiler error message from a heavily-templated program. Presumably modern implementations like Clang++ are better, but still — ugh.

Anyway, my answer to the original question is that C++ should be taught late, for example in a junior or senior level projects course, after the students have learned a bit about disciplined programming and about the design of programming languages. Furthermore, it should be optional. I’d be interested to hear others’ thoughts.
