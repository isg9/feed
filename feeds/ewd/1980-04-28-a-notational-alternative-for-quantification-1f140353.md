---
title: "A notational alternative for quantification (EWD737)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD737.html
published: "1980-04-28T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD737.PDF
---

# A notational alternative for quantification

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

A notational alternative for quantification.
In the following I restrict myself as usual to (beginnings of) countable
domains; booleans and (enough) integers will do.
The so-called universal quantification is given by:

(A i: 0 ≤ i < 0: B(i)) = true

(A i: 0 ≤ i < n+1: B(i)) = (A i: 0 ≤ i < n: B(i)) and B(n) .

The so-called existential quantification is given by:

(E i: 0 ≤ i < 0: B(i)) = false

(E i: 0 ≤ i < n+1: B(i)) = (E i: 0 ≤ i < n: B(i)) or B(n) .

Augustus de Morgan's Law

non (E i: 0 ≤ i < n: B(i)) = (A i: 0 ≤ i < n: non B(i))

is easily proved by mathematical induction.
*              *
*

Quite a few years ago the above notations inspired another one, which I
found under circumstances indispensable. It denotes a natural number, viz.
the number of different values of i in a range such that B(i) holds. Its
recursive definition is

(N i: 0 ≤ i < 0: B(i)) = 0

(N i: 0 ≤ i < n+1: B(i)) = if non B(n) → (N i: 0 ≤ i < n: B(i))

▯ B(n) → (N i: 0 ≤ i < n: B(i)) + 1

fi .

Let us call it "the quantified count". Once the quantified count has been
introduced, we don't need the universal and existential quantifiers anymore.
They can be defined in terms of the quantified count:

(A i: 0 ≤ i < n: B(i)) = ((N i: 0 ≤ i < n: non B(i)) = 0)

(E i: 0 ≤ i < n: B(i)) = ((N i: 0 ≤ i < n: B(i)) ≥ 1) .

We leave the proof that our new definitions are equivalent to our earlier
ones as an exercise for the interested reader.

One advantage of the new notation is that Augustus de Morgan's Law
becomes extremely simple. Omitting for shortness sake the ranges —and
indicating this omission by a succession of two colons— it becomes

(non (N i:: B(i)) ≥ 1) = ((N i:: B(i)) =0) ,

a relation that follows directly from the fact that a quantified count is
a natural number.
All this was discovered quite a few years ago; I remember that it raised
a mild suspicion that the introduction of two quantifiers where one quantified
count would have done had perhaps been a notational mistake. In the meantime
I encountered some more food for that suspicion.
*              *
*

The introduction of the universal and existential quantifiers reflects
in two ways a doubtful practice. The minor complaint is that it is wasteful
to introduce two special characters —in print they appear as inverted, bold,
sans-serif A and E , respectively— when, in addition, we then need
Augustus de Morgan's Law (1) to remind us that the one is only the negation
of the other.
(In this connection it might be worthwhile to point out that in the
boolean domain the "operation" of negation may be regarded as trivial in
the sense that you cannot determine a boolean value x without determining
the value of non x as well. In the integer domain you cannot determine a
value x by determining a value y such that x ≠ y ; in the boolean
domain you can!)
I called this a minor complaint: for the sake of brevity some redundancy
in special characters may be defended. (This admission is not to be
construed as a carte blanche from my side for the notational immodesty that
some of our less wise mathematicians indulge in.) The major complaint is
the introduction of a special notation for what seems at first sight an
arbitrary dichotomy. Is for natural numbers the dichotomy in "equal to zero"
versus "at least one" really the only meaningful one?

Of course it isn't. In the mathematical literature one finds the
existential quantifier sometimes adorned with an exclamation mark to indicate
the existence of exactly one value of i such that B(i), i.e.

(E! i:: B(i)) = ((N i:: B(i) = 1) .

I do not know who introduced that exclamation mark —no doubt his name can
be found in Florian Cajori's "A History of Mathematical Notations"— ; the
introduction of such specific notational gadgetry is of course a road sign
saying "Blind Alley".
Of this I made a mental note when I decided to let the matter rest
until I had encountered the need for more other dichotomies. Yesterday's
experience prompted the writing of this note. During my lectures, when
applying the Gries-Uwicki Theory to a solution to the mutual exclusion
problem between two processes, I derived the invariant

non luck(0) or non luck(1)
(x)

or, in the notation used above

(E i: 0 ≤ i < 2: non 1uck(i)) .
(y)

When my lectures were over and I walked back to my room I realized that the
analogous relation after generalization from 2 to n processes would not
be

(E i: 0 ≤ i < n: non luck(i))

but

(N i: 0 ≤ i < n: luck(i)) ≤ 1 ,

and that is a formula of a very different nature! In retrospect, formula (x),
which suggests formula (y) so strongly, should be regarded as a notational
trap (into which I duly fell).
Finally I ask my readers to savour the expression —in which the ranges
have again been omitted—

(N x:: (N i:: P[i] = x) ≠ (N j:: Q[j] = x)) = 0 ,

expressing that the finite sequence represented by array P is a permutation
of the one similarly represented by array Q . Rendered in English it would
be something like "No value occurs different numbers of times in P and Q ,
respectively.".

Without thus paraphrasing it I gave the above formula to a few mathematicians
to stare at; they all knew the notation of the quantified count
—I had used it before— but were by an accident of history more familiar
with the universal and existential quantifiers. One of them needed a surprisingly
long time to come to grips with it: one of his problems seemed
to be to realize that a value occurring in neither of the two sequences
occurs in both an equal number of times, viz. zero. The explanation may
be that he had his original training in the Department of Mathematics in
Nijmegen, and there —I recently learned— the natural numbers still start
at 1 !
Note. There are two reasons why it is stupid to define the natural numbers
as {1, 2, 3, 4, ....} instead of as {0, 1, 2, 3, ....}. Firstly, we
already have a perfect name for the set {1, 2, 3, 4, ....}, viz. the set
of positive integers. Secondly, by excluding zero from the natural numbers
it becomes "unnatural", and the fact that for many the number "zero" is
still a second-grade citizen is a standard stumbling block in mathematical
thinking. In this case the situation is aggravated because, the University
of Nijmegen being a Roman Catholic one, the term "unnatural" is there loaded
with more pejorative connotations than elsewhere. (End of Note.)
The writing of the above was a pleasant —and I think fully appropriate—
way of spending the day of the Abdication of Queen Juliana and the Coronation
of Queen Beatrix. The weather is much nicer than predicted and I trust to
become as grateful for her reign to Queen Beatrix as I am to who is now
Princess Juliana again. The writing and subsequent typing of the above took
me five hours; I consider them well-spent.

Plataanstraat 530 April 1980

5671 AL NUENENprof.dr.Edsger W.Dijkstra

The NetherlandsBurroughs Research Fellow

Transcribed by Martin P.M. van der Burgt

Last revision

2015-03-11

.
