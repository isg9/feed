---
title: "A somewhat open letter to Uri Leron (EWD1049)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1049.html
published: "1989-04-12T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1049.PDF
---

# A somewhat open letter to Uri Leron

EWD 1049

A somewhat open letter to Uri Leron

Dear Colleague,

you end your letter of March 30, 1989, with:

“I finish with a puzzle, that somehow seems to me related to your Sylvester's theorem (EWD1016). Consider two finite, equinumerous sets of points in the plane. Assume there are no colinear triples. Show that there exists a pairing (bijective correspondence) between the two sets, so that no two line segments connecting corresponding points intersect. (This theorem has a very short, elegant, “existential” proof that mathematicians love and, I suspect, you won’t.)”

The relation to Sylvester’s conjecture is quite close indeed: rephrased as a programming problem —“design an algorithm that constructs a pairing without intersecting segments”— it is solved by standard programming methodology. It is much simpler than Sylvester’s problem in the sense that, in refining the repeatable statement, it will turn out that there is no nondeterminism to be restricted; nor does it require the introduction of a variable (like E in EWD1016) to be incorporated in the invariant. In this case we have a variable q and the invariant P is

P:   q represents a pairing of the two sets

and the simplest program structure to be tried is

“initialise q so as to establish P” {P}

;  do there exists a pair of intersecting segments

→ {?} “change q under invariance of P” {P}

od

and now we have to prove termination of the algorithm. At the place marked with {?}, the only thing that can be asserted besides P are two intersecting segments, say A0-B0 and A1-B1

and no three points (of these four) are colinear. This information allows only one way of changing q under invariance of P: “flip” the pairs as indicated by the dotted segments

In order to prove termination we have to find a function on q that is decreased by such a flip, which replaces in q’s set of pairs/segments two of those by two others. If replacement of a few elements of an otherwise unknown set has to decrease the value of a function on the set, the latter had better be a monotonic function —such as the sum— of a function on the individual elements. (This rules out the number of segment intersections as choice for decreasing function, since that one is defined on the pairs of elements.) What is the simplest function of a pair/segment? The length of the segment, or any function thereof, but the identity function will do: the triangular inequality tells us that the sum of the lengths of the dotted segments is less than the sum of the lengths of the solid segments (the absence of collinear triples guaranteeing that the triangles involved are nondegenerate). The sum of the lengths of the segments is thus a function on q that meets all the requirements for our proof of termination.

*         *         *

My guess is that your “very short, elegant, “existential” proof that mathematicians love” begins like “Consider a pairing with minimal total length of the corresponding segments and assume now that two line segments intersect.” Etc. Such a presentation starts by pulling a gigantic rabbit out of a hat and, indeed, I don’t like that. The above program derivation shows that the choice of decreasing function (which embodies the proof’s invention) is all but forced.

*         *         *

In defence of vague notions such as “intuition” and “understanding” you wrote

“Formal, rigorous terminology are not nearly powerful enough to capture the richness and complexity of [human thought] processes.”.

I don’t and won’t deny you the right to be intrigued by what you perceive as the richness and complexity of human thought processes; it just so happens that I am more interested in increasing the power and simplicity of mathematics. The extent to which we live in different worlds is amply illustrated by the fact that you introduced your problem as “a puzzle”, something I would never have done. For the noun, the Concise Oxford Dictionary gives “problem or toy designed to test knowledge, ingenuity, or patience”, for the verb, Webster’s New Collegiate Dictionary gives “1: to be uncertain as to action or choice   2: to attempt a solution of a puzzle by guesswork or experiment”, for me reasons enough to avoid the term in this context. I definitely prefer a much more conscious approach to problem solving, avoiding trial and error as much as possible.

With my greetings and best wishes, and in gratitude for drawing my attention to your problem,

yours ever,

Edsger W.Dijkstra

Austin, 12 April 1989

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

United States of America

transcribed by Corrado Cantelmi

revised
27-Nov-2011
