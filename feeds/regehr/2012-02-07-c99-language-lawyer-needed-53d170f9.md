---
title: C99 Language Lawyer Needed
url: https://blog.regehr.org/archives/681
published: "2012-02-07T03:59:53Z"
feed: regehr
guid: http://blog.regehr.org/?p=681
---

# C99 Language Lawyer Needed

The program below came up during some tests. The question is, is it well-defined by the C99 language? It appears to clearly be undefined behavior by C11 and C++11.

```
struct {
  int f0:4;
  int f1:4;
} s;

int main (void) {
  (s.f0 = 0) + (s.f1 = 0);
  return 0;
}

```
