---
title: Improve type generic&nbsp;programming
url: https://gustedt.wordpress.com/2021/01/12/improve-type-generic-programming/
published: "2021-01-12T13:00:28Z"
feed: gustedt
guid: https://gustedt.wordpress.com/?p=3766
---

# Improve type generic&nbsp;programming

This is a new paper that appeared today on the site of the C committee:

[http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2638.pdf](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2638.pdf)

C already has a variaty of interfaces for type-generic programming, but lacks a systematic approach that provides type safety, strong ecapsulation and general usability. This paper is a summary paper for a series that provides improvements through

- [N2632](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2632.pdf). type inference for variable definitions (auto feature) and function return
- [N2633](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2633.pdf). function literals and value closures
- [N2634](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2634.pdf). type-generic lambdas (with auto parameters)
- [N2635](http://www.open-std.org/jtc1/sc22/wg14/www/docs/n2632.pdf). lvalue closures (pseudo-references for captures)

The aim is to have a complete set of features that allows to easily specify and reuse type-generic code that can equally be used by applications or by library implementors. All this by remaining faithful to C’s efficient approach of static types and automatic (stack) allocation of local variables, by avoiding superfluous indirections and object aliasing, and by forcing no changes to existing ABI.
