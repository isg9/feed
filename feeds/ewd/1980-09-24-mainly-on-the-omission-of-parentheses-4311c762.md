---
title: "Mainly on the omission of parentheses (EWD751)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD751.html
published: "1980-09-24T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD751.PDF
---

# Mainly on the omission of parentheses

EWD751 -
0

Mainly on the omission of parentheses.

(This is an interruption of EWD749. While working on the latter, I noticed that I wanted to give more attention to some notational questions than I had foreseen. Because these notational concerns are a separate issue, it seemed better to put EWD749 aside and to devote a separate note to them.)

In the following, <atom> stands for a character from a (finite or infinite) alphabet, say for a natural number. Consider the syntax

<seq> ::= <atom> | ( <seq> : <seq> )   .

Here we have borrowed from SASL the colon to denote concatenation. Considering taking the "head" and the "tail" as undoing the concatenation, we can indicate that in the (tentative) embellishment

<seq> ::= <atom> | ( { hd = <seq> } : { tl = <seq> } )   .

For instance, hd((1:2):3) = (1:2) and tl((1:2):3) = 3 ; for sequences that are atoms, the functions hd and tl have been left undefined.

In each sequence <seq> from the above syntax the number of colons obviously equals the number

EWD751 -
1

of parenthesis pairs, and that number - though unbounded over the set of sequences - is finite for each sequence. This latter constraint no longer holds for "continued concatenations" <contconc>

<contconc> ::= ( { hd = <seq> } : { tl = <contconc> } )

The infinite continued concatenation as being defined above being a bit unwieldy to manipulate, a practical system also comprises finite expressions representing continued concatenations. By "representing" we mean here that it is possible to derive its head and to derive (a finite expression "representing") its tail.

* * *

The number of colons being equal to the number of parenthesis pairs, the above syntax is redundant. This redundancy is usually exploited by introducing rules according to which certain parenthesis pairs from the above syntax may be omitted. For instance, by omission of an internal parenthesis pair, (1:2:3) can be derived from ((1:2):3) and from (1:(2:3)). In order to avoid ambiguity, rules have to state in which of the two sequences the internal parentheses pair is obligatory. If (1:2:3) is interpreted as an abbreviation of ((1:2):3) we say that concatenation is "left-associative"; if (1:2:3) is interpreted as an abbreviation of (1:(2:3)) we say that concatenation is "right-associative".

EWD751 -
2

If we denote by the bracket pair <⃒ >⃒ an instance of the corresponding syntactic category, but with its outer parenthesis pair, if present, optionally stripped off, the alternatives for <seq> are:

left-associative: <seq> ::= <atom> | ( <⃒seq>⃒ : <seq> )

right-associative: <seq> ::= <atom> | ( <seq> : <⃒seq>⃒ )

and similarly for <contconc>

left-ass: <contconc> ::= ( <⃒seq>⃒ : <contconc> )

right-ass: <contconc> ::= ( <seq> : <⃒contconc>⃒ )    .

The above exploration served to convince myself that the choice between left- and right-association is totally independent of the asymmetry of the continued concatenation. When the choice is determined by the goal to omit as many parenthesis pairs as possible and we assume that most heads are atoms - and, from that I have seen, this seems a reasonable assumption - right-association is preferable for concatenation. Finally I observe that the bracket pair <⃒ >⃒ has served its purpose; my earlier explorations without it were clumsy.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

24 September 1980

prof.dr. Edsger W. Dijkstra

Burroughs Research Fellow

Transcription by John C Gordon
Last revised on

Wed, 09 Jul 2003

.
