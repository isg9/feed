---
title: 'research!rsc: Killing Quicksort'
url: https://research.swtch.com/qsort
published: "2008-01-02T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/qsort
---

# research!rsc: Killing Quicksort

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Killing Quicksort Russ Cox January 2, 2008 *research.swtch.com/qsort* Posted on Wednesday, January 2, 2008.

[Quicksort](http://en.wikipedia.org/wiki/Quicksort) is a worst-case *O*(n2) algorithm, but if you assume some randomness in either the input or in the decisions made by the algorithm itself, the worst case becomes exceedingly unlikely and the expected runtime becomes *O*( *n* log *n*). As a result, most people treat quicksort as an *O*( *n* log *n*) algorithm. The only wrinkle in this logic is that most quicksort implementations are entirely deterministic, so whether they run in *O*( *n* log *n*) time depends entirely on whether the input data is “random enough.” So there must exist inputs that force these quicksorts into quadratic behavior.

Doug McIlroy's 1999 paper “ [A Killer Adversary for Quicksort](http://www.cs.dartmouth.edu/%7Edoug/mdmspe.pdf)” describes a simple way to find these deadly inputs. The basic idea is that since the C library qsort calls a user-supplied comparison function as the only way it inspects the input, the comparison function can decide what the input is “on the fly,” making decisions that are consistent with previous comparisons but bad for qsort. See the paper for the elegant details.

The paper graphs the 64-element input that forces Digital's qsort to go quadratic:

![](https://research.swtch.com/qsort2.png)

McIlroy provides a [simple C program](http://www.cs.dartmouth.edu/%7Edoug/aqsort.c) illustrating the technique. The GNU C library quicksort goes quadratic on exactly the same kind of input as the Digital quicksort.

(A note of caution for those playing along at home: the GNU C library qsort function is actually mergesort if the C library thinks the computer has enough free memory; to force quicksort, call the undeclared \_quicksort function and compile with gcc-static.)

(Comments originally posted via Blogger.)

- [Adam Fokken](http://www.blogger.com/profile/17269061341779276519)(January 2, 2008 6:20 PM) Does it matter how the quicksort algorithm picks its "pivot"?

- [Adam Fokken](http://www.blogger.com/profile/17269061341779276519)(January 2, 2008 6:25 PM) I see, from the source...

*2\. The pivot-choosing phase uses O(1) comparisons.*

*3\. Partitioning is a contiguous phase of n-O(1) comparisons, all*

*against the same pivot value.*

- [nikolasco](http://nikolasco.livejournal.com/)(January 2, 2008 7:29 PM) I'm shocked, really ... selecting the median is a well-studied problem. Here are bounds for common/textbook algorithms and "state of the art".

Determinisitic:

Median of medians: 10n

Schonhage et al.: 3n

-Lower bound: 2n

Randomized:

Hoare's: 4n

LazySelect: 1.5n

- [jqb](http://www.blogger.com/profile/07510836914645398165)(June 6, 2011 5:14 PM) @nikolasco The problem here is to sort, not to find the median. Surely you aren't suggesting using a 1.5n algorithm to find the median and then use the median as the pivot? That would more than double qsort's average time. Anyway, it's moot, as Musser's Introsort solves the problem.

- (July 31, 2011 7:26 AM) hello

Im finding how to generat a killer series for quicksort....

cant this series made by just array and costom the len of the series?

thanks
