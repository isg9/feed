---
title: "A very simple exercise in SASL (EWD827)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD827.html
published: "1982-06-06T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD827.PDF
---

# A very simple exercise in SASL

The idea is to prove the equality of two recursively defined infinite sequences by showing that the one satisfies the defining equation of the other. Here we shall try this out on the simplest example I can think of.

Let us consider the equations

(0) from n = n : from (n+1)

(1a) inc(a:b) = a+1 : inc b

(1b) x = 0 : inc x .

We have to prove x = from 0 . We observe

(2)      (incn x = n : incn+1 x )

⇒    { inc is a function }

(inc(incn x ) = inc (n : incn+1 x ))

=    { 1a }

(inc(incn x) = n +1 : inc(incn+1 x ))

=    { def. of iterated functional composition }

(3)      (incn+1 x = n+1 : incn+2 x )

With (2)⇒(3) providing the induction step and —since    x = inc0 x    by the definition of iterated functional definition— (1b) providing the base, (2) has been proved for all n. Since (2) is the same equation in    incn x    as (0) is in from   n   ,   incn x = from n   has been established for all n, in particular for n=0  , which completes the proof.
*                *
*

I tried it the other way round , viz. I tried to show that from 0 was a solution of (1b), but this became a mess.
It is too warm and too humid to write much more now.

Burroughs 6 June 1982

12201 Technology Blvd. prof.dr.Edsger W.Dijkstra

AUSTIN, Texas, 78759 Burroughs Research Fellow

USA

transcribed by Germán González-Morris
revised Sun, 26 Dec 2010
