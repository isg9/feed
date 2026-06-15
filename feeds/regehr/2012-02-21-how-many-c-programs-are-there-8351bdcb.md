---
title: How Many C Programs Are There?
url: https://blog.regehr.org/archives/689
published: "2012-02-21T04:28:18Z"
feed: regehr
guid: http://blog.regehr.org/?p=689
---

# How Many C Programs Are There?

If I choose a size S, can you tell me how many valid C programs exist that are no larger than that size? I’m actually interested in the answer — it’ll help me make a point in a paper I’m writing. Shockingly, the Internet (or at least, the part of it that I looked at based on a few searches) does not provide a good answer.

Let’s start with a few premises:

1. Since it would be exceedingly difficult to construct the exact answer, we’re looking for a respectably tight lower bound.
2. S is measured in bytes.
3. Since it seems obvious that there’s an exponential number of C programs, our answer will be of the form NS.

But how to compute N? My approach (and I’m not claiming it’s the best one) is to consider only programs of this form:

```
int main() {
int id1;
int id2;
....
}

```

The C99 standard guarantees that the first 63 characters of each identifier are significant, so (to drop into Perl regex mode for a second) each identifier is characterized as: \[a-zA-Z\_\]\[a-zA-Z0-9\_\]{62}. The number of different identifiers of this form is 53\*6362, but we spend 69 bytes in order to generate that many programs, in addition to 15 bytes of overhead to declare main(). The 69th root of the large number is between 43 and 44, so I claim that 43(N-15) is a not-incredibly-bad lower bound on the number of C programs of size S. Eventually we’ll hit various other implementation limits and be forced to declare additional functions, but I suspect this can be done without invalidating the lower bound.

Of course, the programs I’ve enumerated are not very interesting. Perhaps it would have been better to ask:

- How many non-alpha-equivalent C programs are there?

or even:

- How many functionally inequivalent C programs are there?

But these sound hard and don’t serve my narrow purpose of making a very blunt point.

Anyway, I’d be interested to hear people’s thoughts on this, and in particular on the 43S (claimed) lower bound.
