---
title: Fuzzer Puzzle
url: https://blog.regehr.org/archives/1042
published: "2013-09-24T17:44:30Z"
feed: regehr
guid: http://blog.regehr.org/?p=1042
---

# Fuzzer Puzzle

A [recent post](http://www.cert.org/blogs/certcc/2013/09/one_weird_trick_for_finding_mo.html) on the CERT/CC blog says:

> … BFF 2.7 and FOE 2.1 do have an optional feature that in our testing drastically increases the number of uniquely-crashing test cases: crasher recycling. When enabled, the frameworks will add \[a\] uniquely-crashing test case back into the pool of seed files for further fuzzing. When crashing test cases are fuzzed more, this can uncover both new vulnerabilities as well as new ways in which an already-known vulnerability can cause a crash.

It seems pretty clear why adding a crasher back into the seed pool will find new ways to trigger the bug that the crasher already triggers: fuzzing this test case will generate test cases that are near the crasher (in some abstract test case space that we won’t try to define too closely). These nearby test cases are presumably more likely to find different ways to trigger the same bug.

But why is it the case that this technique “drastically increases the number of uniquely-crashing test cases”? I believe the authors intend “uniquely-crashing test case” to mean a test case that triggers a new bug, one that is distinct from any bugs already found during fuzzing. I’ll avoid speculating for now so I can see what you folks think.

**UPDATE:** Allen Householder from CERT has [written some more](https://www.cert.org/blogs/certcc/2013/09/putting_the_rocket_on_the_chai.html) about this.
