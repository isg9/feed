---
title: "Distances from the root in skew trees (with C.S.Scholten) (EWD801)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD801.html
published: "1981-08-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD801.PDF
---

# Distances from the root in skew trees (with C.S.Scholten)

EWD 801

Distances from the root in skew trees
by Edsger W. Dijkstra and C.S. Scholten

Renewed interest in algorithms for sorting in situ raised the question of the average distance from node to root in binary trees other than balanced ones. (Here binary trees are to be understood as rooted trees in which nodes have zero, one, or two sons.)

In particular we consider the infinite sequence of trees Ti (i ≥ 0), in which for some fixed p and q (p > q ≥ 0)

- Ti for 0 ≤ i < p are arbitrarily chosen
- Tn and Tn+q are the subtrees of Tn+p

With Hi = the number of nodes in Ti, we have

Hn+p = Hn+q + Hn + 1        ;

with Gi = the sum of the distances of te nodes of Ti from its root we have

Gn+p = Gn+q + Gn + Hn+q + Hn        .

We are interested in the asymptotic behaviour of Gi / Hi for large i, this ratio being the average distance from the root in Ti.

Without loss of generality we can confine ourselves to the case gcd(p,q) = 1, since in the case gcd(p,q) = g > 1, the sequence Ti consists of an interleaving of g mutually independent sequences.

With Ai defined by Ai = Hi + 1 and Bi defined by Bi = Gi – 2, we derive

(0)    An+p = An+q + An
(1)    Bn+p = Bn+q + Bn + An+p

Equation (1) is not homogeneous in the B's, but by solving it for the A's and substituting them in (0), we get a homogeneous recurrence relation for the B's, the characteristic polynomial of which is the product of (0)'s characteristic polynomial and the characteristic polynomial corresponding to the homogeneous part of (1). We conclude that the characteristic equation for the Ai is

(2)    xp – xq – 1 = 0

and that that for the Bi is

(3)    (xp – xq – 1)2 = 0

Under the constraints gcd(p,q) = 1 and p > q ≥ 0, (2) enjoys the property of having one positive root, r say, such that r > 1 and all other roots of (2) have a modulus smaller than r. (For a proof of this theorem, see later.)
From this and the theory of linear recurrence relations we conclude

1)
that the leading term of Ai is of the form k · ri for some constant k

2)
that the leading term of Bi is of the form ( l + L·i ) · ri for some constants l and L.

Substituting these leading terms in (1) we get

( l + L·(n+p) ) · rn+p =

( l + L·(n+q) ) · rn+q + ( l + L·n ) · rn + k · rn+p

Since r is a root of (2) this can be reduced to

(4)    L / k = rp / ( p · rp – q · rq )

We define the skewness of a binary tree as the ratio (≥ 1) of the numbers of nodes in its two subtrees. For the trees Ti it follows from the leading term of Ai that the asymptotic skewness s is given by s = rq, whence q = rlog s. Remembering that r satisfies (2), we find rp = s+1, whence p = rlog (s+1). Hence (4) can be rewritten as

(5)    L / k =

s+1

(s+1) · rlog (s+1) – s·rlog s

Because r > 1 —so that the asymptotic value of Gi / Hi equals that of Bi / Ai—, we conclude that, expressed in r and s, the average distance from the root in Ti is for large i

(s+1) · i

(s+1) · rlog (s+1) – s · rlog s

Consequently, the average distance from the root in a tree from the sequence Ti with N nodes is

(s+1) · log N

(s+1) · log (s+1) – s · log s

i.e. an expression in s and N only.

*      *      *

We are left with the obligation to prove that f x = 0 with f x = xp – xq – 1 has one positive root r > 1 dominating the others. Since f 0 = –1 and f(+∞) = +∞, f x = 0 has an odd number of positive roots. Because f' x = xq–1 · ( p·xp–q – q ) we conclude that f' x = 0 has at most 1 positive root; hence f x = 0 has 1 positive root r and, because f 1 = –1, we conclude r > 1. In other words

(6)    for x ≥ 0    sgn(f x) = sgn(x–r)

In order to prove dominance of r, we consider a root m·ei·φ of (2), with m > 0. Consequently

mp · ei·p·φ = mq · ei·q·φ + 1 ,

from which we derive —by taking absolute values—

mp < mq + 1 ∨ (p·φ) mod 2π = (q·φ) mod 2π = 0 .

Since gcd(p,q) = 1, this can be rewritten as mp – mq – 1 < 0 ∨ φ = 0 or, in view of (6): m<r ∨ φ = 0. q.e.d.

*      *      *

Finally we remark that thanks to

p

q
=
log (s+1)

log s

any value of s can be approximated arbitrarily closely by suitable choice of p and q, a fact that enhances the significance of the expression in s and N for the average distance from the root.

28 August 1981

drs. C. S. Scholten

Philips Research Laboratories

5600 MD  EINDHOVEN

The Netherlands

prof. dr. E. W. Dijkstra

Burroughs Research Fellow

Plataanstraat 5

5671 AL  NUENEN

The Netherlands

transcribed by Martijn van der Veen

revised Thu, 11 Nov 2004
