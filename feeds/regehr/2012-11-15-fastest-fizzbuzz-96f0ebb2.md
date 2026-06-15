---
title: Fastest FizzBuzz
url: https://blog.regehr.org/archives/839
published: "2012-11-15T20:34:55Z"
feed: regehr
guid: http://blog.regehr.org/?p=839
---

# Fastest FizzBuzz

The always-entertaining [FizzBuzz problem](http://c2.com/cgi/wiki?FizzBuzzTest) came up again on Hacker News today, and for no other reason than I just got out from under a nasty deadline, I looked around on the net for interesting solutions, for which this [Rosetta Code](http://rosettacode.org/wiki/FizzBuzz) page is a gold mine. The [Windows batch file](http://rosettacode.org/wiki/FizzBuzz#Batch_File) and [BSD make](http://rosettacode.org/wiki/FizzBuzz#make) versions are hilarious, though I was a bit disappointed that Piet was not represented. Anyhow, here it is: ![](https://blog.regehr.org/extra_files/fizzbuzz.png) ( [reference](http://www.dangermouse.net/esoteric/piet/samples.html))

Another wonderful FizzBuzz [solution uses template metaprogramming](http://rosettacode.org/wiki/FizzBuzz#C.2B.2B) to evaluate the solution at compile time. I was disappointed, however, to look at the code emitted by Clang++, G++, and Intel’s C++ compiler, which all generate a large amount of crap instead of boiling the program down to its essence: emitting a single string.

I also noticed that various sites that discuss the efficiency of FizzBuzz implementations, such as [this one](http://www.zoharbabin.com/which-fizzbuzz-solution-is-the-most-efficient), failed to provide what I would consider the most efficient code:

```
#include <stdio.h>

int main (void)
{
  puts("1\n2\nFizz\n4\nBuzz\nFizz\n7\n8\nFizz\nBuzz\n11\nFizz\n13\n14\nFizzBuzz\n16\n17\nFizz\n19\nBuzz\nFizz\n22\n23\nFizz\nBuzz\n26\nFizz\n28\n29\nFizzBuzz\n31\n32\nFizz\n34\nBuzz\nFizz\n37\n38\nFizz\nBuzz\n41\nFizz\n43\n44\nFizzBuzz\n46\n47\nFizz\n49\nBuzz\nFizz\n52\n53\nFizz\nBuzz\n56\nFizz\n58\n59\nFizzBuzz\n61\n62\nFizz\n64\nBuzz\nFizz\n67\n68\nFizz\nBuzz\n71\nFizz\n73\n74\nFizzBuzz\n76\n77\nFizz\n79\nBuzz\nFizz\n82\n83\nFizz\nBuzz\n86\nFizz\n88\n89\nFizzBuzz\n91\n92\nFizz\n94\nBuzz\nFizz\n97\n98\nFizz\nBuzz");
  return 0;
}

```

It would perhaps be better style to use printf() instead of puts(), counting on the compiler to optimize the former to the latter, but the version of GCC that I use does not actually do that.

Using [my timing code](https://blog.regehr.org/archives/330) on a random Nehalem 64-bit Linux machine I get:

g++ -O3: 125,000 cycles

Intel C++ -fast: 103,000 cycles

Clang++ -O3: 103,000 cycles

gcc -O3: 29,000 cycles

gcc -O3 -static: 21,000 cycles

Here’s the [C code](https://blog.regehr.org/extra_files/fizz.c) and [C++ code](https://blog.regehr.org/extra_files/fizz.cpp). For benchmarking purposes I piped their STDOUT to /dev/null.
