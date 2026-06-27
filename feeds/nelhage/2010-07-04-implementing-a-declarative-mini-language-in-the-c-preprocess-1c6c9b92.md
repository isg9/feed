---
title: Implementing a declarative mini-language in the C preprocessor
url: https://blog.nelhage.com/2010/07/implementing-an-edsl-in-cpp/
published: "2010-07-04T15:54:55Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/07/implementing-an-edsl-in-cpp/
---

# Implementing a declarative mini-language in the C preprocessor

Last time, I announced Check Plus, a declarative language for defining Check tests in C. This time, I want to talk about the tricks I used to implement a declarative minilanguage using the C preprocessor (and some GCC extensions). The Problem We want to write some toplevel declarations that look like: #define SUITE\_NAME example BEGIN\_SUITE("Example test suite"); #define TEST\_CASE core BEGIN\_TEST\_CASE("Core tests"); … and so on, and somehow translate them into code that does the equivalent of:
