---
title: 'Check Plus: An EDSL for writing unit tests in C'
url: https://blog.nelhage.com/2010/06/check-plus-an-edsl-for-writing-unit-tests-in-c/
published: "2010-06-26T15:54:53Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/06/check-plus-an-edsl-for-writing-unit-tests-in-c/
---

# Check Plus: An EDSL for writing unit tests in C

Check is an excellent unit-testing framework for C code, used by a number of relatively well-known projects. It includes features such as running all tests in separate address spaces (using fork(2)), which means that the test suite can properly report segfaults or similar crashes without the test runner crashes. My main complaint about Check is that (unsurprisingly for a framework written in C), it’s not very declarative. After you define all your tests as separate functions, you need to write code to manually collect them into “test cases”, which you then collect into “test suites”, which you can then run.
