---
title: "A machine for image construction in tomography (EWD704)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD704.html
published: "1979-02-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD704.PDF
---

# A machine for image construction in tomography

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

A machine for image construction in tomography.
Each measurement corresponds to a ray that is characterized by its
direction and its distance from the centre. It gives in each point of the
image an additive contribution to the absorption density that is

1)constant along a line parallel to the ray

2)proportional to the value measured.

For the contributions of measurements corresponding to the rays in one
direction we cover the picture by a set of stripes —not necessarily of the
same width— and in that same direction. The contribution of each ray is
approximated by a function that is constant in each stripe.
For each stripe that function value can be computed as the product of
the measured value and a constant that is uniquely determined by the stripe
number and the distance from the ray to the centre. Because that distance is
a function of the detector, we need —with a coverage by M stripes— to
store M constants per detector; obviously, the M multiplications of a
measured value can be performed simultaneously.
The area in which the image is to be constructed is divided into fields
of a size small enough for the required resolution. In each field the density
is treated as constant: the subdivision of the area into fields represents the
spatial discretization of the image. Each stripe contributes additively to
a field its function value, multiplied by the fraction of the field covered
by the stripe. If fields are so small that they are covered by at most two
stripes, this takes at most one multiplication per field. For each
measurement all these multiplications can be performed simultaneously. For each field
the identity of the covering stripe(s) and the value of the fraction follows
uniquely from the position of the field.
For the next part of the design

1)     it is a benefit that the rays in another direction have the same set
of distances from the centre, so that the same M constants per detector can

be used in all directions

2)     it is essential that the angle between two directions is a multiple
of 2π/ N for some large integer N .
Our discretization of the image consists of concentric rings —not
necessarily all of the same width— each of which is equally divided into
N/k equal fields, with k a positive integer. Because k = 1 would lead
to a very expensive machine, it is desirable that N is not prime; it is
even desirable that N has a suitable number of small factors. (N = 2400
looks realistic, still nicer would be N = 2520 , the smallest number
divisible by the numbers up to 10 .)
At any moment each field is taken care of by one cell. The N/k
cells taking care of the N/k fields of a ring are placed in the corresponding
cyclic order. Each cell takes care of its field in k “successive” positions,
where “successive” is meant in the sense that the transition from one position
to the next amounts to an image rotation of 2π / N . Each cell has a counter
c , satisfying 0 ≤ c < k , at each moment the c-values of the cells of a
ring have the same value. A (clockwise) rotation over 2π / N amounts in
each ring to c:= (c+1)mod k , and in those rings in which c has thereby
returned to zero, to a transmission by each cell to its clockwise neighbour
of its field value, accumulated thus far.
If field sizes —i.e. k-values— are chosen such that the area covered
by the k successive positions taken care of by a cell, is covered by at most
two stripes, each cell needs at most two function values per measurement.
Between 0 and 1 the “fraction” can, to all intents and purposes, be taken as
a linear function of c .
Each measurement can be processed by first rotating the image in the
appropriate direction.

24th February 1979prof.dr.Edsger W.Dijkstra

Plataanstraat 5Burroughs Research Fellow

The Netherlands

Transcribed by Martin P.M. van der Burgt

Last revision

2015-02-12

.
