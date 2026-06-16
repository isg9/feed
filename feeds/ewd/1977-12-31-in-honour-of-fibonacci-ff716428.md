---
title: "In honour of Fibonacci (EWD654)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD654.html
published: "1977-12-31T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD654.PDF
---

# In honour of Fibonacci

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

In honour of Fibonacci.
Studying an artificial intelligence approach to programming the other
day —I read the most weird documents!— I was reminded of the Fibonacci
sequence. given by

Fi = 0 , F2 = 1 ,

Fn = Fn-1 + Fn-2 (-inf

For N ≥ 2 the relation

R:x = FN

is trivially established by the program

y, x, i := 0, 1, 2 {Y = Fi-1 and x = Fi and 2 ≤ i ≤ N};(1)

do i ≠ N → y, x, i := x, x + y, i + l .od {R}

a program with a time-complexity proportional to N; I remembered —although
I did not know the formulae— that R can also be established in a number of
operations proportional to log(N) and wondered —as a matter of fact: I still
wonder— how proponents of “program transformations” propose to transform the
linear algorithm (1) into the logarithmic one.
Yesterday evening I was wondering whether I could reconstruct the
logarithmic scheme for the Fibonacci sequence, and whether similar schemes
existed for higher order recurrence relations (for a k ≥ 2 ):

F1 = F2 = ... = Fk-1 = 0 , Fk = 1

Fn = Fn-1 + ... + Fn-k (-inf < n < +inf)(2)

Eventually I found a way of deriving these schemes. For k = 2 ,
the normal Fibonacci numbers, the method leads to the well-known formulae

F2j = fj2 + Fj+12

F2j+1 = (2Fj + Fj+1) * Fj+1 or F2j-1 = (2fj+1 - Fj) * Fj .

This note is written, because I liked my general derivation. I shall describe
it for k = 3 .

Because for k = 3 we have F1 = F2 = 0 and F3 = l we may write

Fn = F3 * Fn + (F2 + F1)* Fn-1 + F2 * Fn-2(3)

From (3) we deduce the truth of

Fn = Fi+3* Fn-i + (Fi+2 + Fi+1)* Fn-i-1+ Fi+2* Fn-i-2(4)

for i = 0 . The truth of (4) for all positive values of i is derived by
mathematical induction; the induction step consists of

1)     substituting Fn-i-1 + Fn-i-2 + Fn-i-3 for Fn-i

2)     combining after rearrangement Fi+3 + Fi+2 + Fi+1 for Fi+4 .

(The proof for negative values of i is done by performing the induction
step the other way round.)
Substituting in (4) n = 2j and i = j-l we get

F2j = Fj2 * Fj+1 + (Fj+1 + Fj)* Fj + Fj+1* Fj-1

and, by substituting Fj+2 - Fj+1 - Fj for Fj-1 , and subsequent rearranging

F2j = Fj2 + (2Fj+2 - Fj+1)* Fj+1(5)

Substituting in (4) n = 2j+l and i = j-l we get

F2j+l = Fj+2 + (Fj+1 + Fj)* Fj+1 + Fj+1 * Fj

= (2Fj + Fj+1) * Fj+1 + Fj+22(6)

Formulae (5) and (6) were the ones I was after.
Note. For k = 4 the analogue to (4) is
Fn = Fi+4 * Fn-i + (Fi+3 + Fi+2 + Fi+1)* Fn-i-1 +
(Fi+3 + Fi+2)* Fn-i-2 + Fi+3 * Fn-i-3 �
(End of note.)
.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

5671 AL NuenenBurroughs Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2015-01-29

.
