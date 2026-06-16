---
title: "The problem of the maximum length of an ascending subsequence (EWD591)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD05xx/EWD591.html
published: "1976-10-19T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd05xx/EWD591.PDF
---

# The problem of the maximum length of an ascending subsequence

We consider a sequence of   N   elements   A(1)   through   A(N)  . The order of increasing subscript value will be denoted by “the order from left to right”. From such a sequence we can take so-called “subsequences of length s” by the removal of an arbitrary collection of   N–s   elements and retaining the remaining   s   elements in the order in which they occurred in the original sequence. When, in addition, each element has an integer value, we call a subsequence "ascending" iff it contains no element with a right-hand neighbour smaller than itself.

Note.   According to this definition, all   N   subsequences of length   1   and even the empty subsequence are ascending ones. (End of note.)

We look for an algorithm that determines for any given sequence (with N > 0) the maximum length of an ascending subsequence that can be taken from it.

Note.   Although there need not be a unique longest ascending subsequence, the maximum length is unique, e.g.   3 1 1 2 4 3   gives   4   for the maximum length, realized either by   1 1 2 4   or by   1 1 2 3  . (End of note.)

If   k   represents the value we are looking for, we seek to establish the relation

R:    k =

the maximum length of an ascending subsequence taken from   A(1)   through   A(N)

Because   R   contains the parameter   N  , it is strongly suggested to take as invariant relation —or, as we shall see in a moment: as part of the invariant relation—

P1(k, n):    k =

the maximum length of an ascending subsequence taken
from   A(1)   through   A(n)  .

It has the virtues that it would do the job in the sense that   (P1(k, n) and n = N) ⇒ R   and is easily established, e.g. by   k, n := 1, 1  . These observations suggest to establish   P1(k, n)   for   n = 1   and then to increase   n   under invariance of P1(k, n)   until   n = N , more precisely: to increase   n   repeatedly by   1   and to restore each time, when destroyed, the truth of   P1(k, n)   by adjusting the value of   k . Because extension with a next element can never decrease the maximum length of an ascending subsequence and can increase it by at most   1  , the adjustment of   k  , when needed, will have the form   k:= k + 1  . More precisely: because

P1(k, n) = wp(“n:= n + 1”, P1(k, n–1))

we have to investigate after   “n:= n + 1”   under which circumstances no adjustment of   k   is needed, i.e. when

P1(k, n–1) ⇒ P1(k, n)

,

(1)

and under what circumstances adjustment of   k   is needed, i.e. when

P1(k, n–1) ⇒ P1(k+1, n)

,

(2)

Relation (2) holds iff   A(n)   can be used to extend an ascending subsequence of maximum length ( = k) taken from   A(1)   through   A(n–1) ; this is true iff

A(n)  ≥

the smallest rightmost value of an ascending subsequence of length k   taken from   A(1)   through   A(n–1)  .

This last inequality shows us, that besides   k   —as defined by   P1(k, n)—   we would also like to store the minimum rightmost value —let us call it   m   for a moment— of an ascending subsequence of maximal length. If (2) holds,  k   is obviously to be adjusted by   k:= k + 1  , and the assignment   m:= A(n)   would make   m   again equal to the minimum rightmost value of an ascending subsequence of maximum length (because, in this case, all ascending subsequences of maximal length taken from   A(1)   through   A(n)   will have A(n)   as their rightmost element.)

The introduction of   m   as the minimum value of the rightmost value of an ascending subsequence of length   k  , presents, however, a problem in case (1). In that case, the extension with   A(n)  , although not leading to an increase of k  , may require adjustment of   m   as it may lead to decrease of the minimum rightmost value of an ascending subsequence of that unchanged maximal length. This would be the case if the value   A(n)  —which now satisfies   A(n) < m — could be used to extend an ascending subsequence of length   k–1  , taken from A(1)   through   A(n–1)  . In order to decide that, we would also need the minimum rightmost value of an ascending subsequence of length   k–1  . Repeating the argument, we conclude that, instead of a scalar   m  , we need in addition to   k   an array variable   m   satisfying

P2(k, n, m):

for all   j   satisfying   1 ≤ j ≤ k

m(j) = the minimum rightmost value of an ascending subsequence of length   j   and taken from A(1)   through   A(n)  .

Our total invariant relation will be   P1 and P2  .

Again, for   n = 1, P2   is easily initialized   —with   m(1) = A(1)—;   we have to investigate, however, what updating obligations for the array variable   m   are implied by our duty to keep   P2   invariant. The crucial discovery in the analysis of our updating obligations for the array variable   m   is that the elements of   m   itself are ascending, more precisely:

(1 ≤ i < j ≤ k) ⇒ (m(i) ≤ m(j))        .

This follows from the fact that   1 ≤ i < j ≤ k and m(i) > m(j)   leads to a contradiction: by removing from an ascending sequence of length   j   and with m(j)   as its rightmost value the leftmost   j–i   elements, an ascending sequence of length   i   with   m(j)   as rightmost value remains, and   m(i) > m(j)   then contradicts   P2  .

Again we investigate the situation as reached after   n:= n + 1  , i.e. when P1(k, n–1) and P2(k, n–1, m)   holds. Relation (2) holds iff   A(n) ≥ m(k). The new element   A(n)   can be used to form a longer ascending sequence,   k   has to be increased and the sequence of values is extended with   A(n)   by

m : hiext(A(n))       ;

it is correct to leave the values   m(i)   with   1 ≤ i < k   unchanged, for the new element   A(n) ≥ m(k)   and can never give rise to a smaller rightmost value for any of the ascending sequences shorter than the new maximum length   k  . Relation (1) holds iff   A(n) < m(k)  . Remembering that after the increase n:= n + 1   the relation   P2(k, n–1, m)   holds, we have to answer the question for which value(s) of   j   is the minimum rightmost value of an ascending sequence of length   j   taken from   A(1)   through   A(n)   smaller than taken from   A(1)   through   A(n–1)?   This can only be the case if   A(n)   is its new rightmost element, which must be smaller than its old value   m(j)  . So we have

A(n) < m(j)

(3)

But   A(n)   can only be the rightmost element of an ascending sequence of length   j   if

either   j = 1   or   j > 1 and m(j–1) ≤ A(n)

(4)

Combining (3) and (4) we find
j = 1   iff   A(n) < m(1)   and otherwise   j = the only(!) solution of   m(j–1) ≤ A(n) < m(j)  . This last solution is found in the program with a binary search; the invariant relation for the inner loop is   m(i) ≤ A(n) < m(j)  .

Observing that   k = m.hib  , the current higher bound for the index, we can use for   m(k) = m(m.hib)   the usual abbrevation   m.high   and conclude that we don’t need the variable   k   after all. For reasons of symmetry we denote   m(1)   by   m.low  , as   m.lob = 1  . Omitting all declarations we get the following program.

n:= 1; m:= (1, A(1));do n ≠ N → n:= n + 1; if A(n) ≥ m.high → m:hiext(A(n)) ⌷ A(n) < m.high → if m.low > A(n) → j:= 1 ⌷ m.low ≤ A(n) → i, j := m.lob, m.hib; do i ≠ j - 1 → h:= (i + j)div 2; if m(h) ≤ A(n) → i:= h ⌷ A(n) < m(h) → j:= h fi od fi; m:(j)=A(n) fiod;print(m.hib)

Plataanstraat  5   prof. dr. Edsger W. Dijkstra

NL-4565    NUENEN   Burroughs Research Fellow

The Netherlands

transcribed by Swarup Sahoo
last revised Tue, 5 Jul 2011
