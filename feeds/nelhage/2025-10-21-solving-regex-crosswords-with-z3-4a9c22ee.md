---
title: Solving regex crosswords with Z3
url: https://blog.nelhage.com/post/regex-crosswords-z3/
published: "2025-10-21T14:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/regex-crosswords-z3/
---

# Solving regex crosswords with Z3

For a while now, I’ve been fascinated by Z3 and by SMT solving more broadly. While on pat leave recently, I was reminded of the existence of regular-expression crossword puzzles, and allowed myself to get nerdsniped by writing a Z3-backed solver. I expected to spend perhaps an afternoon cranking out a quick solver; I ended up getting sucked into understanding and debugging Z3 performance, and learning far more about Z3 and about SMT than I expected.
