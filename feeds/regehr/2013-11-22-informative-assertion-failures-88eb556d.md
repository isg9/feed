---
title: Informative Assertion Failures
url: https://blog.regehr.org/archives/1068
published: "2013-11-22T16:43:58Z"
feed: regehr
guid: http://blog.regehr.org/?p=1068
---

# Informative Assertion Failures

The other day a student was asking me how to make assertion failures in software more informative. I told him there are two pretty easy ways to do this. First, define a customized assert that takes two arguments: the predicate and also a description. For example, in the [Botan](http://botan.randombit.net/) crypto library we see lines like this:

```
BOTAN_ASSERT(signing_key, "Signing key was set");

```

The second technique is to use a standard single-argument assert like this:

```
assert((depth > 0) && "Invalid depth!");

```

Any other good tricks out there?
