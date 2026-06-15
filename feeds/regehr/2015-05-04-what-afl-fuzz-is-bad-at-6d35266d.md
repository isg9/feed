---
title: What afl-fuzz Is Bad At
url: https://blog.regehr.org/archives/1238
published: "2015-05-04T03:07:29Z"
feed: regehr
guid: http://blog.regehr.org/?p=1238
---

# What afl-fuzz Is Bad At

[American fuzzy lop](http://lcamtuf.coredump.cx/afl/) is a polished and effective fuzzing tool. It has found tons of bugs and there are any number of blog posts talking about that. Here we’re going to take a quick look at what it isn’t good at. For example, here’s a program that’s trivial to crash by hand, that afl-fuzz isn’t likely to crash in an amount of time you’re prepared to wait:

```
#include <stdlib.h>
#include <stdio.h>

int main(void) {
  char input[32];
  if (fgets(input, 32, stdin)) {
    long n = strtol(input, 0, 10);
    printf("%ld\n", 3 / (n + 1000000));
  }
  return 0;
}

```

The problem, of course, is that we’ve asked afl-fuzz to find a needle in a haystack and its built-in feedback mechanism does not help guide its search towards the needle. Actually there are two parts to the problem. First, something needs to recognize that divide-by-zero is a crash behavior that should be targeted. Second, the search must be guided towards inputs that result in a zero denominator. A finer-grained feedback mechanism, such as how far the denominator is from zero, would probably do the job here. Alternatively, we could switch to a different generation technology such as concolic testing where a solver is used to generate test inputs.

The real question is how to get the benefits of these techniques without making afl-fuzz into something that it isn’t — making it worse at things that it is already good at. To see why concolic testing might make things worse, consider that it spends a lot of time making solver calls instead of actually running tests: the base testing throughput is a small fraction of what you get from plain old fuzzing. A reasonable solution would be to divide up the available CPU time among strategies; for example, devote one core to concolic testing and seven to regular old afl-fuzz. Alternatively, we could wait for afl-fuzz to become stuck — not hitting any new coverage targets within the last hour, perhaps — and then switch to an alternate technique for a little while before returning to afl-fuzz. Obviously the various search strategies should share a pool of testcases so they can interact (afl-fuzz already has good support for communication among instances). Hopefully some concolic testing people will spend a bit of time bolting their tools onto afl-fuzz so we can play with these ideas.

A variant of the needle-in-a-haystack problem occurs when we have low-entropy input such as the C++ programs we would use to test a C++ compiler. Very few ascii strings are C++ programs and concolic testing is of very little help in generating interesting C++. The solution is to build more awareness of the structure of C++ into the testcase generator. Ideally, this would be done without requiring the somewhat elaborate effort we put into Csmith. Something like [LangFuzz](https://www.usenix.org/system/files/conference/usenixsecurity12/sec12-final73.pdf) might be a good compromise. (Of course I am aware that afl-fuzz has been used against clang and has found plenty of ways to crash it! This is great but the bugs being found are not the kind of semantic bugs that Csmith was designed to find. Different testcase generators solve different problems.)

So we have this great fuzzing tool that’s easy to use, but it also commonly runs up against testing problems that it can’t solve. A reasonable way forward will be for afl-fuzz to provide the basic framework and then we can all build extensions that help it reach into domains that it couldn’t before. Perhaps Valgrind is a good analogy: it provides not only an awesome baseline tool but also a rich infrastructure for building new tools.

\[Pascal Cuoq gave me some feedback on this post including a bunch of ideas about afl-fuzz-related things that he’ll hopefully blog about in the near future.\]
