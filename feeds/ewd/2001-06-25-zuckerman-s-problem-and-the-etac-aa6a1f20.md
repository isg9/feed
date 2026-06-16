---
title: "Zuckerman’s problem and the ETAC (EWD1309a)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD13xx/EWD1309a.html
published: "2001-06-25T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd13xx/EWD1309a.PDF
---

# Zuckerman’s problem and the ETAC

Zuckerman's problem and the ETAC

Thanks to David Zuckerman, the following problem circulated, before I left for Europe, at the Department in Austin.

At a University with N professors, committees have to be formed under the following constraints.

C0: each committee consist of an odd number of professors

C1: any two distinct committees share an even number of professors.

What is the maximum number of committees that can be formed?

*                 *

*

Before leaving the USA I had thought about the problem but had not solved it. The N distinct committees each consisting of a single professor meet both constraints, so the answer is ≥ N. Exploring the possibilities for a few small values of N gave me the "moral certainty" that the answer was also ≤ N.

I quickly decided not to use set notation, which I don't like (never found it helpful) but to characterize instead each committee by a vector of N bits. It took me more time to decide how to interpret these bits: were they Booleans representing true/false, or were they integers representing 0/1.

Initially I was attracted by the Boolean interpretation because the continued equivalence expresses so nicely the evenness of the number of its false arguments and C0 requires us to pay attention to such odd/evenness. In C1, however, the notion of sharing requires us to pay attention to the identity of the professors as well, but this identity is not really reflected in the continued equivalence which is defined on unordered bags of arguments. Thus I found that in the context of this problem I could not manipulate the continued equivalence in a meaningful manner.

Turning my attention to the integer interpretation I realized that both constraints could be expressed without explicit mention of the individual professors in terms of the scalar product.

Note The scalar product • is defined on pairs of integer sequences of equal length by

[]•[] = 0

[x:xs]•[y:ys] = x∙y + [xs]•[ys]

(End of Note)

Here is the problem as I posed it to the ETAC on Tuesday 12 June 2001.

Consider a set V of 0/1 vectors of N components such that

C0:    〈∀i: vi ∈ V : odd.(vi•vi)〉

C1:    〈∀i,j: vi,vj ∈ V ∧ i ≠j: even.(vi•vj)〉

Show that V contains at most N vectors.

Remark The two constraints can elegantly be combined into

〈∀i,j: vi,vj ∈ V: i = j  ≡  odd.(vi•vj)〉

and this is very encouraging. (End of Remark)

Because in N-dimensional space there are no sets of more than N linearly independent vectors, the ETAC set out to solve the problem by showing that the vectors in V are linearly independent. Let ON denote the zero-vector of length N and let α be a sequence of as many coefficients as there are vectors in V. Linear independence of the vectors vi in V then means that for any α

(0)           〈∑i:: αi•vi 〉 = ON  ⇒  〈∀i:: αi=0 〉

In my experience with linear algebra, the components of the vectors and the coefficients αi of the linear composition used to range over the real numbers, but fortunately some of the ETAC members knew the whole theory is also applicable in the field of rational numbers. The vector components being integer, and hence rational, it suffices to prove (0) for rational α. But since left- and right-hand side of (0) are invariant for multiplication of α by a constant that differs from zero, we can confine our attention to integer α. (Because the number of vi is at most 2N, i.e. finite, we can multiply α by a common multiple of the denominators of the αi to get all coefficients integer.) From now on, α stands for a sequence of integer coefficients. We observe for any j in range

〈∑i :: αi ∙ vi 〉 = ON

⇒         { ON • vj = 0}

〈∑i :: αi∙ vi 〉• vj = 0

≡         { • distributes over ∑ }

〈∑i :: (αi∙vi ) • vj 〉 = 0

≡         { ∙ and • mutually associative }

〈∑i :: αi∙(vi • vj )〉 = 0

≡         { split range; 1-point rule }

〈∑i : i≠j : αi∙(vi • vj )〉 + αj∙(vj • vj ) = 0

⇒         { αi integer; C1, i.e. even.(vi • vj ) }

even.(αj∙(vj • vj ))

⇒         { C0, i.e. odd.(vj • vj )}}

even.αj

The above calculation implies with the aid of some elementary algebra that

(1)       int.α ∧ 〈∑i:: αi∙vi 〉 = ON

is invariant under α := α/2, which allows us to conclude that under the truth of (1), each αj is divisible by arbitrary high powers of 2 and hence = 0. And thus, (0) has been established.

Nuenen, 25 June 2001

prof. dr Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA.

transcribed by Wolfgang Houben

revised Wed, 14 Nov 2007
