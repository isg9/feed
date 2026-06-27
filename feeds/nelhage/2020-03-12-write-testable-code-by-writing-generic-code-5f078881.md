---
title: Write testable code by writing generic code
url: https://blog.nelhage.com/post/write-testable-code-by-writing-generic-code/
published: "2020-03-12T01:30:17Z"
feed: nelhage
guid: https://blog.nelhage.com/post/write-testable-code-by-writing-generic-code/
---

# Write testable code by writing generic code

Alex Gaynor recently asked this question in an IRC channel I hang out in (a channel which contains several software engineers nearly as obsessed with software testing as I am): uhh, so I’m writing some code to handle an econnreset… how do I test this? This is a good question! Testing ECONNRESET is one of those fiddly problems that exists at the interface between systems — in his case, with S3, not even a system under his control — that can be infuriatingly tricky to reproduce and test.
