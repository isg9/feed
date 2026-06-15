---
title: 'research!rsc: Rotating Hashes'
url: https://research.swtch.com/hash
published: "2008-03-12T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/hash
---

# research!rsc: Rotating Hashes

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Rotating Hashes Russ Cox March 12, 2008 *research.swtch.com/hash* Posted on Wednesday, March 12, 2008.

Long ago, before hash tables (aka scatter storage files) were a fundamental data type like `int`, programmers had to implement them by hand. (Nowadays, manually-implemented hash tables are rarely seen outside of historical reenactments like undergraduate algorithms classes and C programs.) Back then, there was fierce debate about how to handle hash collisions.

One option, called direct chaining, makes each table slot a linked list head, and then all the entries that hash to that slot are kept in the linked list pointed to by the slot. (Bonus points if you reorder the lists using the [move to front heuristic](http://www.nist.gov/dads/HTML/movefront.html).)

Another option, called linear scanning or open addressing, is to try table entry *h*, and then if that is full, try *h* +1, *h* +2, and so on until an empty slot is found. This only works well if the table is sized so that it has some large fraction (say, 1/4) of unused slots.

Direct chaining and linear scanning are still the most common collision resolution methods, but in the 1960s, more were explored. Still another option is to use non-linear scanning, so that each data item produces a sequence of hash values *h* 1, *h* 2, *h* 3, and those table slots are tried in turn. Two objects with the same *h* 1 won't necessarily have the same *h* 2, unlike in linear scanning. But hashing is a relatively slow operation. Is it worth the time and code to compute all those extra hash functions?

This sets the stage for Doug McIlroy's 1963 **[letter to the Communications\** **of the ACM Pracniques page](http://doi.acm.org/10.1145/358141.358146)**, describing a trick by Vic Vyssotsky:

> A VARIANT METHOD OF FILE SEARCHING
>
> I would like to publicize a trick due to V. A. Vyssotsky for improving the efficiency of scatter storage files.
>
> Vyssotsky's idea, used in a remarkably short and elegant code for the IBM 7090, is to continue random searching after the first probe fails:
>
> (1) From a data item, compute a pseudo-random address in the file.
>
> (2) If the item is in this place, or if the place is empty, the job is done.
>
> (3) If another item is there, probe again with a new address computed by another pseudo-random function, until the stock of pseudo-random functions has been exhausted.
>
> (4) As a last resort, when all pseudo-random functions are exhausted, do a linear search of the whole file for the item or a hole, whichever comes first.
>
> In Vyssotsky's program, the later random functions are trivially obtained from the first one. He computes an *n*-bit function of which only the first *m* bits are used as an address. Later, random addresses are obtained by rotating the same *n* bits. After *n* probes he gives up and resorts to linear search.
>
> More familiar methods of scatter storage employ chaning or linear search for later probes. Chaining is the most efficient method in terms of average number of probes per item but pays a price in storage space for the chaining information. Linear search is expensive in time. Table I, comparing average number of probes per item for these methods, is based on popular uniformity and independence assumptions:
>
> TABLE I. Average Number of Probes per Item _Load Factor__Chaining \[1\]__Linear Search \[2\]__Random Search_p 1 + p/2 1 + ½p/(1-p) -p-1 log(1-p) .50 1.25 1.5 1.39 .75 1.38 2.5 1.83 .90 1.45 5.5 2.56
>
> Chaining and random search have a clear edge. If storage requirements permit, chaining is superior. Vyssotsky's random search pays an attractively low time penalty to achieve ultimate storage utilization in pressing situations.
>
> A typical 7090 application used a file of two-word items. Chaining would add an extra word or, with extra coding complexity to handle items of nonintegral word length, an extra half word per item. Table II compares lookup times for various file schemes given a certain amount of storage:
>
> TABLE II. Lookup Times for Various File Schemes *Load Factor with*
>
> *3-word items__Chaining*
>
> *3-word__Chaining*
>
> *2½-word__Linear*
>
> *2-word__Random*
>
> _2-word_0.6 1.3 1.25 1.33 1.30 0.8 1.4 1.33 1.57 1.49 1.0 1.5 1.42 2.00 1.65 1.2 overflow 1.50 3.00 2.01 1.4 overflow overflow 6.00 2.90
>
> References:
>
> 1\. Johnson, L. R. Indirect chaining method for addressing on secondary keys. *Comm. ACM 4* (1961), 218-223.
>
> 2\. Schay, G. and Spruth, W. G. Analysis of a file addressing method. *Comm. ACM 5* (1962), 459-462.
>
> M. D. McIlroy
>
> *Bell Telephone Laboratories, Inc.*
>
> *Murray Hill, N. J.*

(Posting may be intermittent for the rest of March.)

(Comments originally posted via Blogger.)

- [nasorenga](http://www.blogger.com/profile/05686871315696257402)(March 12, 2008 11:25 AM) Seems to me that with the simple bit-rotation method, all members of a collision group will have the same sequence of addresses, making it equivalent to linear scanning. So why does it perform better?

- [Russ Cox](http://swtch.com/~rsc/)(March 12, 2008 12:08 PM) @nasorenga: It performs better because you compute more bits than you need for a single hash, but not necessarily as many as you need for n different hashes. You might compute a 36-bit hash and then use different 16-bit windows as the scanning sequence.

- [nasorenga](http://www.blogger.com/profile/05686871315696257402)(March 12, 2008 12:21 PM) @rsc: I think your point about computing several hashes applies to non-linear scanning, not to linear scanning, which is what the random scanning is being compared to in the article. -- Btw, it occurs to me that an advantage that random does have over linear is that each collision group's scan sequence is unique. With linear scanning, when a clump starts to form, it affects the performance of all nearby collision groups. This may explain the performance improvement gained with random scan.

- [Michael](http://www.blogger.com/profile/01123264110675539010)(March 22, 2008 12:54 AM) Hi Ross,

Generating and trying multiple hash values has found a more recent application in Cuckoo hashing, which offers worst-case O(1) lookup. And it works very well -- I've used an implementation in production that keeps over 2 million elements with a 0.99 load factor. See http://www.it-c.dk/people/pagh/papers/cuckoo-undergrad.pdf, but to achieve these high load factors, you have to generalize the number of hash functions partitions to 5 or so.

- [Michael](http://www.blogger.com/profile/01123264110675539010)(March 22, 2008 1:01 AM) Err, I meant Russ. How embarrassing, sorry! :)
