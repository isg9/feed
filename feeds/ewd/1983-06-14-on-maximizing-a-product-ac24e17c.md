---
title: "On maximizing a product (EWD856)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD856.html
published: "1983-06-14T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD856.PDF
---

# On maximizing a product

EWD856-0

On maximising a
product

The problem is to
construct a set of positive integers with given sum such that their
product is as large as possible.

The first observation
is that we need not include integers ≥
4. The reason is that such an integer k can be replaced, without
changing the sum, by the two integers 2 and k-2, thereby
replacing in the product a factor k by a factor 2∙k - 4 and that we don't lose if 2∙k - 4 ≥
k, i.e. if k ≥
4.

The second
observation is that 1 only needs to be included if it is the only
integer in the set, i.e. if the given sum equals 1. In all other cases
we can increase the product without changing the sum by removing the 1
(which does not contribute to the product) and increasing one of the
other integers by 1.

In the general case this leaves us with a set of 2's and 3's. The final observation is that we don't need to include 3 or more 2's: 2+2+2 = 3+3 but 2∙2∙2 < 3∙3.

*                *

*

We solved this
problem a number of years ago (and were not the first to do so), but in
my memory the argument was less clean than above. It was more or less
polluted by

EWD856-1

the circumstance that the answer is not unique if the given sum is of the form 3∙n +
4: two 2's may be replaced by one 4.

Another way of
complicating the first observation is to treat even and odd values
separately, e.g. comparing for k = 2∙x the value 2∙x
with x2, and for k = 2∙x + 1 the value 2∙x + 1 with x∙(x + 1).

California
Institute of Technology

PASADENA, CA 91125

United States of America

14 June 1983

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcriber: Kevin Hely.
Last revised on Sat, 20 Sep 2003.
