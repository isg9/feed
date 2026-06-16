---
title: "A somewhat open letter to D.A.Turner (EWD759)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD759.html
published: "1980-11-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD759.PDF
---

# A somewhat open letter to D.A.Turner

A somewhat open letter to D.A. Turner.
Dear David,

I read with considerable interest your "Program Proving and Applicative Languages", but when I tried to apply your techniques to the first slightly more ambitious example my assistant W.H.J. Feijen came up with, it turned out to be a very humbling experience: I clearly lacked the knack of dealing with SASL programs.

Consider the following short SASL definitions (my syntax! I trust you will forgive me that)

def k x y (p : q) =

if p ≤ y → k (x + 1) y q

⌷ p > y → x : k x (y + 1) (p : q)

fi ;

def g = k 0 0 f

Here f is supposed to be ascending, i.e. to satisfy (P0 f ) with

P0 (a : b : c)   =   a ≤ b ∧ P0 (b : c)

and unbounded, i.e. to satisfy (Ay : y ≥ 0: P1 y f ) with
P1 y (p : q)   =   p > y ∨ (P1 y q).

When f is ascending and unbounded, it is clearly an infinite list. (I try to use the more positive term "continued concatenation" for that, but that is another story.)

You would do me a great service by showing me a proof that when f is ascending and unbounded, g is ascending and unbounded.

The precise formal relation between g and f is that for all y ≥ 0

sub y g = (N x : x ≥ 0 : y ≥ (sub x f ))

(0)

where sub is defined by

def sub n (p : q) =

if n = 0 → p

⌷ n > 0 → sub (n−1) q

fi

and the (N ... ) should be read as: the number of distinct values x in the range x ≥ 0 such that y ≥ (sub x f). You would do me a great service by showing me as well, how you would prove (0). I am looking forward to your reply.
With my greetings and best wishes,

Yours ever

Edsger

5 November 1980

transcribed by Guy Haworth
revised Thu, 8 Nov 2007
