---
title: "The marriage agency (EWD1214)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1214.html
published: "1995-09-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1214.PDF
---

# The marriage agency

A marriage agency has in its files 25 men and 25 women and aims at acquainting the men with the women by organising dinner evenings, each one involving 12 tables, seating 2 men and 2 women. (Consequently, at each dinner evening, 1 man and 1 woman have a day off.) The question is how many of such dinner evenings the marriage agency can schedule without some man and woman sharing a table twice.

*                *

*

Frans van der Sommen raised this problem at the end of the ETAC session of 8 August 1995. Rutger M. Dijkstra, who happened to attend that session, immediately showed how to schedule 12 dinner evenings: partition the men into a singleton and 12 pairs numbered [0...12), do the same with the women, and let on the nth evening the men of pair i share a table with the women of pair (i+n) mod 12.

The demonstration that 12 is indeed the maximum took a litle bit longer. Consider a sequence of 13 dinner evenings. Then at most 13 men have had an evening off, and therefore at least 12 (= 25 - 13) men dined all 13 evenings. Such a man enjoyed 26 (= 2 × 13) times the company of a woman; the number of women being 25, there is —pigeon-hole principle— at least one woman whose company he enjoyed at least twice. [As said, this took a little bit longer. We first tried too crude a counting argument in terms of man-woman combinations. Their total number is 625 (= 25 × 25), each evening 48 (= 4 × 12) combinations are getting acquainted. Since 625/48 = 13+1/48, this rules out a schedule of 14 or more evenings, but the feasibility of a schedule of 13 dinner evenings it leaves open.]

This is actually a sequel to EWD1212: feasibility is shown by means of a witness, infeasibility by a counting argument.

Austin, 5 September 1995

prof.dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712 - 1188 USA

Transcribed by Guy Haworth

Last revised Sun, 30 Dec 2007.
