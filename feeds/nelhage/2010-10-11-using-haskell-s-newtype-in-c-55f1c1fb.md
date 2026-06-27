---
title: Using Haskell's 'newtype' in C
url: https://blog.nelhage.com/2010/10/using-haskells-newtype-in-c/
published: "2010-10-11T13:11:25Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/10/using-haskells-newtype-in-c/
---

# Using Haskell's 'newtype' in C

A common problem in software engineering is avoiding confusion and errors when dealing with multiple types of data that share the same representation. Classic examples include differentiating between measurements stored in different units, distinguishing between a string of HTML and a string of plain text (one of these needs to be encoded before it can safely be included in a web page!), or keeping track of pointers to physical memory or virtual memory when writing the lower layers of an operating system’s memory management.
