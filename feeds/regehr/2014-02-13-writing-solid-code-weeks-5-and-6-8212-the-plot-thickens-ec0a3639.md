---
title: Writing Solid Code Weeks 5 and 6 &#8212; The Plot Thickens
url: https://blog.regehr.org/archives/1102
published: "2014-02-13T21:42:36Z"
feed: regehr
guid: http://blog.regehr.org/?p=1102
---

# Writing Solid Code Weeks 5 and 6 &#8212; The Plot Thickens

Last week was pretty relaxed, I lectured on doing code reviews — pretty much the standard pitch. This week we spent Tuesday doing a code review, I think it went pretty well. I had chosen a team whose code was strong so that our first code review would be a positive one.

Today I announced that the Huffman compression that the class has been working on has been found by the client to be unsuitable, it does not give a good enough compression ratio for some files of interest: massive PBMs showing mostly text and some graphics. The client decided that its requirements would be better met by first performing run-length compression on the PBM files at the bit level, resulting in 8-bit run-length codes: one bit for the sense of the run, 7 bits for length. Since runs of white are much more common than runs of black, there’s still plenty of redundancy left in the RLE codes, so they should subsequently be Huffman encoded.

The other aspect of today’s assignment is that due to some internal shuffling at our company, the teams have been moved around. Each team must implement RLE compression on top of the Huffman implementation provided by the team whose number is one greater (modulo seven). Teams are forbidden from touching their old code, although I hope they will be willing to answer questions about it. This is an exercise that I’ve been wanting to spring on students for a long time. Hopefully it will be a character-building experience for everyone.

Also in class today we spent some time looking at defect reports on the students’ code from [Coverity Scan](https://scan.coverity.com/). This was (for me at least) pretty entertaining. First, the defect reports were of impressively high quality. Second, Coverity’s web interface is quite nice to use
