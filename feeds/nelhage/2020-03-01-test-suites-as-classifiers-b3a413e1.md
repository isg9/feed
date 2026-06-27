---
title: Test suites as classifiers
url: https://blog.nelhage.com/post/test-suites-as-classifiers/
published: "2020-03-01T20:34:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/test-suites-as-classifiers/
---

# Test suites as classifiers

Suppose we have some codebase we’re considering applying some patch to, and which has a robust and maintained test suite. Considering the patch, we may ask, is this patch acceptable to apply and deploy. By this we mean to ask if the patch breaks any important functionality, violates any key properties or invariants of the codebase, or would otherwise cause some unacceptable risk or harm. In principle, we can divide all patches into “acceptable” or “unacceptable” relative to some project-specific notion of what we’re willing to allow.
