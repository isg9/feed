---
title: "A generalization of the Sheffer Stroke for n-valued logic (by C.S.Scholten) (EWD429)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD04xx/EWD429.html
published: "1974-06-18T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd04xx/EWD429.PDF
---

# A generalization of the Sheffer Stroke for n-valued logic (by C.S.Scholten)

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

A generalization of the Shaffer Stroke for n-valued logic.

by C.S.Scholten

For binary variables —i.e. ranging over {0, 1}— x and y the
“Shaffer Stroke” —a1ias “alternative denial” or “nand”— is the operator
—or binary function— denoted by

x | y

and (usually) defined as

1 - x.y .

As 1 - x.y = (x.y + 1) mod 2

and x.y = min(x, y)

an alternative definition is:

x | y = (min(x, y) + 1) mod 2 .

It is well-known [1] that any binary function of k binary arguments can
be expressed using the Shaffer Stroke as the only primitive operator.
In the following we regard n-ary variables —i.e. ranging over
{0, 1, ..., n-1}— and shall demonstrate that any n-ary function of k
n-ary arguments can be expressed, using as the only operator the generalization

x | y = (min(x, y) + 1) mod n .

In the following we shall use the notation

suc(x) = (x + 1) mod n .

in terms of which we can rewrite:

x | y = suc(min(x, y)) .

Theorem 1. The function suc —as defined above— is expressible.

Proof. suc(x) = x | x

Theorem 2. The function pred —defined by pred(x) = (x - 1) mod n— is
expressible.

Proof. pred(x) = sucn-1(x)

Theorem 3. The function min is expressible.

Proof. min(x, y) = pred(x | y)

Note. Because

min(x, y, z) = min(min(x, y), z) etc.

also the minimum of more than two arguments is expressible. (End of note.)
Theorem 4. For any n-ary constant c the discriminator function dc(x),

given by dc(x) = n - 1 for x = c

= 0 for x ≠ c

is expressible.

Proof. Because

sucn-c(x) = 0 for x = c

= 0 for x ≠ c

is expressible.

we have min(1, sucn-c(x)) = 0 for x = c

= 1 for x ≠ c ,

so that dc(x) = pred(min(1, sucn-c(x))) .
Theorem 5. For any n-ary constant c the weighten discriminator function
wc(x, y), given by

wc(x, y) = y for x = c

= n - 1 for x f c

is expressible.

Proof. min(dc(x), suc(y)) = suc(y) for x = c

= 0 for x ≠ c

and because pred(suc(y)) = y , we find

wc(x, y) = pred(min(dc(x), suc(y))) .

Theorem 6. The selector function, defined by

sel(x, y0, y1. ... , yn-1) = yi for x = i

is expressible.

Proof. sel(x, y0, y1, ... , yn-1) =

min(w0(x, y0). w1(x, y1). . wn-1(x, yn_1)) .

The final induction from 2 to k arguments is straightforward.
[1] Shaffer. H.M., A set of five independent postulates for Boolean algebras,
with applications to logical constants. Trans.Amer.Math.Soc. vol 14, pp,481-488

Transcribed by Martin P.M. van der Burgt

Last revision

2014-11-15

.
