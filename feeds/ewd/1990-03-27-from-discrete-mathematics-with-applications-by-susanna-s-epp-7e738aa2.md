---
title: "From “Discrete Mathematics with Applications” by Susanna S. Epp (EWD1076)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1076.html
published: "1990-03-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1076.PDF
---

# From “Discrete Mathematics with Applications” by Susanna S. Epp

EWD 1076

From "Discrete Mathematics with Applications" by Susanna S. Epp.

On p.439 of her recent book Discrete Mathematics with Applications, Susanna S. Epp presents the following problem:

There are 42 students who are to share 12 computers. Each student uses exactly one computer and no computer is used by more than six students. Show that at least five computers are used by three or more students.

My solution:

Let k be the number of computers serving at least three students; as each computer serves at most 6 students, these k machines serve at most 6k students. Each of the remaining 12–k computers serves at most 2 students; the remaining machines therefore serve at most 24–2k students. Adding these results, we conclude that the number of students served is at most 24+4k

Since 42 students are served, we conclude 42 ≤ 24 + 4k or 18 ≤ 4k and, since k is integer, 5 ≤ k, which completes the proof.

The author of the book, however, proceeds as follows:

Solution:

[Use an argument by contradiction.] Suppose that only four computers were used by three or more students. [A contradiction must be derived.] At most six students are allowed to share any computer, making a total of at most 24 students using these four computers. Since there are 42 students in all, that would leave at least 18 students to share the remaining eight computers with no more than two students per computer. But the generalized pigeonhole principle guarantees that if 18 students share eight computers, then at least three must share one of them. [The pigeons are the 18 students, the pigeonholes are the eight computers, and 18 > 2 ∙ 8.] This is a contradiction. thus the supposition is false, and so at least five computers are used by three or more students.

This is not the kind of proof I encourage my students to write; they could, in fact, immediately recognize it as not written by one of them:

(i)for the avoidable reductio ad absurdum;

(ii)
for the two occurrences of “must”, which they have learned to avoid —“A contradiction has to be derived” and “then at least three share one of them”—

(iii)
for the loose “This” in “This is a contradiction.”.

Upon somewhat closer inspection they should see that the argument as given by Epp does not hold water: it does not exclude that only three (or fewer) computers serve three or more students. The above “solution”contains a bug so serious that it sends the author back to the drawing board.

I am very grateful to Susanna Epp for her bug. You see, her book is no joke. As she wrote in her covering letter to me: “This book represents a serious effort to reveal the unspoken, often unconscious, conventions of mathematical thought to the uninitiated.”, a characterization that is fully confirmed by the book. As such this little incident confirms my suspicion that “the unspoken, often unconscious, conventions of mathematical thought” are not good enough.

Austin, 27 March 1990

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
27-Nov-2011
