---
title: "To hell with “meaningful identifiers”! (EWD1044)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1044.html
published: "1989-02-17T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1044.PDF
---

# To hell with “meaningful identifiers”!

EWD 1044

To hell with “meaningful identifiers”!

This is about the construction of a bag of natural numbers with given sum and maximal product. In the following, a maximal bag means a bag that realizes for the sum of its content the maximum value of the product; for any given sum at least one maximal bag exists because the number of bags with given sum is finite.

The first argument begins by characterizing bags that are not maximal. All characterizations follow the same pattern: non-maximality is demonstrated by showing how to construct a bag with the same sum and a larger product.

Lemma 0 A bag containing 0 is not maximal

Proof Removing all zeros from the bag increases the product. (End of Proof.)

Lemma 1 A bag containing 1 and at least one further element is not maximal.

Proof Replacing the 1 and another element by their sum increases the product. (End of Proof.)

Corollary For a sum ≥ 2, a bag containing a 1 is not maximal.

Lemma 2 For x > 4, a bag containing x is not maximal.

Proof We analyze the condition that splitting up x into 2 and x – 2 increases the product

2∙(x – 2) > x

=

2∙x – 4 > x

=

x > 4

(End of Proof.)

For sums 0 and 1 the problem has been settled —{} and {1} respectively—; for larger sums the search is restricted by the above to bags built from 2's, 3's, and 4's. One way to proceed is to remark that, since 2 + 2 = 4 and 2 * 2 = 4, bags containing a 4 are disposable. Narrowed down to bags built from 2's and 3's, the observations 2 + 2 + 2 = 3 + 3 and 2 * 2 * 2 < 3 * 3 settle the problem.

The ruling out of bags with elements > 4, followed by the disposing of bags with a 4, makes it very tempting to combine those argument and state

(0)   “For x ≥ 4, a bag containing x is disposable”

Proof We analyze the condition that splitting x up into 2 and x – 2 does not decrease the product

2∙(x – 2) ≥ x

=

2∙x – 4 ≥ x

=

x ≥ 4

(End of Proof.)

Neat as this may seem, the argument is now —as Paul Pritchard made me see— fallacious, as we can see from the following simplified example.

Let us consider bags whose elements add up to 2∙N.

•     Bags containing even elements are disposable.

Proof   Each even element can be written as the sum of two odd values. (End of Proof.)

•     Bags containing odd elements are disposable.

Proof Since 2∙N is even, the number of odd elements is even; hence the odd elements can be eliminated by replacing pairs of them by their sum. (End of Proof.)

And now the conclusion is, of course, that we can restrict the search space to bags of natural numbers that are neither even nor odd, i.e. to the empty bag!

Back to our problem of maximizing the product. It seems instructive to analyze what went wrong when we introduced our disposable bags. To simplify the discussion I restrict myself to bags of “plurals” with a “plural” sum. (“plural” means “integer and ≥ 2”; like “integer” and “natural” it is both a noun and an adjective.)

The trouble has been invited by the fact that we did not give a precise definition of the notion “disposable.” Reformulating (0) for plurals and without the adjective “disposable” yields, for instance,

(1)   For every bag of plurals there exists a bag of plurals <4, with the same sum and at least the same product.

There are two ways of looking at this theorem: it is a theorem about all bags of plurals to be proved by mathematical induction, or it is an existence theorem that we can demonstrate by developing a terminating algorithm that constructs a witness. Both views boil down to the same thing as both require a well-founded order on finite bags.

[When we characterize each bag by its —natural— frequency function f: f.x is the number of occurrences of x in the bag, the well-founded order is the lexical order on the sequences f.x in the order of decreasing x. Splitting up x into x – 2 and 2 corresponds to

f.x, f.(x – 2), f.2    :=    f.x –1, f.(x – 2) + 1, f.2 + 1

which represents a lexical decrease of the sequence. Or simpler: sum – #elements in the bag.]

Compared to (1), the weaker Lemma 2

(2)   For every bag of plurals, there exists a bag of plurals ≤ 4 with the same sum and at least the same product.

does not require mathematical induction for its proof. The existence of maximal bags not being in doubt, it suffices to demonstrate that a bag with a plural > 4 is not maximal.

With the replacement of Lemma 2 and a separate treatment of 4's in the bag by (1), we have eliminated a case analysis at the price of a proof by mathematical induction. I consider that price rather heavy.

The clearest moral of the above is never to introduce terms (like “disposable”) on the basis of “you know I mean, don't you?” To hell with the “meaningful identifiers!” The havoc they create is not confined to programming but extends over all of mathematics.

Austin, 17 February 1989

prof. dr. Edsger W Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712–1188

USA

transcribed by Alan Arvesen

revised
10-Apr-2016
