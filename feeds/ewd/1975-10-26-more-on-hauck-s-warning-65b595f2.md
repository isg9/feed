---
title: "More on Hauck's warning (EWD528)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD05xx/EWD528.html
published: "1975-10-26T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd05xx/EWD528.PDF
---

# More on Hauck's warning

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Copyright Notice

The following manuscript

EWD 528 More on Hauck�s warning :

is held in copyright by Springer-Verlag New York.

The manuscript was published as pages172�173 of

Edsger W. Dijkstra, Selected Writings on Computing: A Personal Perspective,

Springer-Verlag, 1982. ISBN 0�387�90652�5.

Reproduced with permission from Springer-Verlag New York.

Any further reproduction is strictly prohibited.

More on Hauck’s warning.
In EWD525 “On a warning from E.A.Hauck” I mentioned without proof that
with n=2m bit there exist 2n-m-1 different messages —I called them
“codes”, but that is an unusual terminology for which I apologize— , such
that any two different messages differ in at least four bit positions, thus
allowing correction of one-bit errors and detection of two-bit errors. Since
then I have been shown a proof of that theorem; I report that proof because
it is so nice, and because it gives some further insights.
For the sake of brevity I shall demonstrate the theorem for 16:24 bits
(in a way which is readily generalized for other values of m ). We consider
16 bits numbered from 0 through 15 , writing their index in binary:

d0000, d0001, d0010, d0011, ..., d1111

With “xxx1” we denote the set of odd indices, with “xx1x” the set {0010, 0011,
0110, 0111, 1010, 1011, 1110, 1111}, in general the set obtained by all
possible substitutions of a 0 or a 1 at a place marked “x”, and define

h0 = parity(dxxx1 ) , h1 = parity(dxx1x ) , h2 = parity(dx1xx ) , h3 = parity(d1xxx )

where the function “parity” is = 0 if among the (8) bits with an index from
the indicated set, the number of 1’s is even, and = 1 if it is odd. Further
we introduce h = parity(dxxxx) , which is just the sum of all the 16 bits
modulo 2.
The 211 correct messages are then characterized by the equations

h0 = h1 = h2 = h3 = h = 0 .

Note. The above equations have indeed 211 different solutions: the 11 bits
d3, d5, d6, d7, d9, d10, d11, d12, d13, d14, and d15 can be chosen freely,
we then solve h0 for d1, h1 for d2, h2 for d4, and h3 for d8, and finally
h for d0 .
We now denote by “a” the binary number formed by “h3 h2 h1 h0” and
observe:

0)     for each correct message we have

h = 0, a = 0

1)     for a one-bit error at bit position i we have

h = 1, a = 1

2)     for a two-bit error at bit positions i and j

h = 0, a = the bit-wise sum of i and j

(because i ≠ j , we conclude that a ≠ 0, thereby distinguishing this case
from a correct message)

3)     for a three-bit error at positions i , j , and k

h = 1, a = the bit-wise sum of i , j , and k.

4)     for a four-bit error at positions i , j , k , and l

h = 0, a = the bit-wise sum of i , j , k , and l .

etc.

Under the assumption that one- and two-bit errors are the only errors
that can occur, the rules are

h = 0 and a = 0:accept the bit sequence

h = 1 :invert bit da

h = 0 and a ≠ 0:alarm, as two-bit error has been detected.

From the above, however, we see that all errors in 3, 5, 7, ... bits
will then erroneously be interpreted as one-bit errors, i.e. in those cases
our error correction indeed increases the probability of a wrong result being
produced as if it were a correct one. The above gives a clear demonstration
of the possible “harmfulness” of error correction alluded to in EWD525’s
last paragraph. Hence this note.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

4565 - NUENENBurroughs Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2014-11-27

.
