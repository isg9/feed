---
title: "Fast image construction in computerized axial tomography (CAT) (with A.J.M. van Gasteren) (EWD810a)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD810a.html
published: "1982-02-11T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD810a.PDF
---

# Fast image construction in computerized axial tomography (CAT) (with A.J.M. van Gasteren)

Fast image construction in computerized axial tomography (CAT)

The purpose of computerized axial tomography is to approximate the distribution of the absorption density in a cross-section of a specimen probed by many X-rays in the plane of the cross-section. This approximation, which is a non-negative function on a circular domain, is called the “image” because it is usually displayed on a screen, brightness representing the function value.

For the sake of discretization, the circular domain is subdivided into so-called “pixels”, i.e. little picture elements in each of which the approximating function is constant. The required resolution imposes an upper bound on the size of the pixels and hence a lower bound on their number. Each pixel requires the computation of a scalar value and, these computations being elaborate, there is a premium on not introducing many pixels more than needed for the resolution. (For each pixel the value to be computed is, in fact, a linear compositum of all the measurements.)

This note describes the structure of a special-purpose machine that could compute the image on the fly, i.e. during the few seconds the X-ray scanning of the cross-section takes. In medical applications in particular, such immediate availability of the image is deemed important.

The specimen is probed with rays in N homogeneously distributed directions. (Practical values of N would be 2400 or 2520 , the least common multiple of the integers up to 10 : we shall show later why it is desirable for N to have many small factors.) The rays in one direction are not necessarily equidistant; the same pattern of parallel rays is used for all N directions by rotating it in the plane of the cross-section around the centre of the circular domain. (Hence the name “axial” tomography.)

With M rays in each direction, the complete probing of the specimen yields M*N scalar measurements, viz, one for each ray traversing the specimen. Each measurement yields an additive contribution to each point of the image and this contribution —for a justification we refer to the literature— is

(i) proportional to the measurement

(ii) constant along lines in the direction of the ray with which the measurement was taken.

The contribution is, in fact, positive along the ray and negative everywhere else (but monotonically approaching 0 as the distance from the ray increases); beyond (i) and (ii) the precise shape of the contribution is of no importance in what follows. The number of pixels is large, say 50,000; the number of measurements is a few times larger and, with each of the measurements contributing to each pixel value, we conclude that image construction is a very sizeable computation indeed. The ensuing severity of the real-time requirement calls for special purpose equipment with a high degree of concurrency.

We shall first describe the processing of a single measurement, i.e. how its additive contribution to the image is approximated. To that end the circular domain is covered by a pattern of narrow stripes in the direction of the ray from which the measurement was derived; for all measurements from rays in the same direction we shall use the same pattern of stripes. Contributions are approximated by functions that are constant in each stripe and the widths of the stripes must be small enough to make this approximation adequate. Thus we have taken (ii) into account. For each measurement/stripe pair the contribution value requires one multiplication: the value measured has to be multiplied by a constant whose value is determined by the positions of the stripe and of the ray of the measurement. (With a multiplier per stripe, the multiplications for a single measurement can be performed simultaneously.) Thus we have taken (i) into account.

In the second phase each stripe contributes to the pixels (partially) covered by it: the pixel value is increased by the product of the stripe value and the fraction of the pixel that is covered by the stripe. This requires a few constants per pixel. (If no pixel is covered by more than two stripes, in fact one fraction of coverage and one multiplication suffice.)

Note. Since the coverage fractions only depend on the pixels and the stripes, the measurements in the same direction can, if so desired, share the second phase: for each stripe the contributions from the individual measurements are added and this sum is multiplied by the coverage fractions. This reduces the amount of computation per pixel by a factor M (= the number of rays in each direction). Depending on the type of scanner, collecting the measurements in the same direction may require a considerable amount of high-speed storage. (End of Note.)

Our next purpose is to choose the pixels so as to exploit the rotational symmetry of the measuring device. To this end the pixels are arranged in concentric rings with N/k equal pixels in each ring. In the outermost rings the required resolution implies a small value for k, 3 say. The smaller the ring, the larger the value for k that the resolution allows; note, however, that k is restricted to the divisors of N.

Finally we decide to use the same stripe pattern for all directions. All through the image construction, the stripe pattern has N different positions relative to the image, and hence relative to each ring. A ring with N/k (equal!) pixels and the same ring after a rotation of k steps—i.e. a rotation over 2π k/N— are, however, congruent. In a ring with N/k pixels, the same coverage fractions therefore reappear after a rotation of k steps, but shifted over one pixel. Hence the total number of converage fractions pertaining to the pixels of a single ring is of order N.

A ring of N/k pixels is taken care of by a cyclic arrangement of N/k computational elements, called “nodes”, with a node for each pixel. In the ring, the node/pixel correspondence is, however, not constant. After a rotation of k steps, the pixel values are cyclically transferred to the next node in the cycle. (The direction in which pixel values are transferred is opposite to the direction of the rotation of the rays through the specimen.) In other words, with respect to the pixels the cycle of nodes acts as a —possibly two-way— cyclic shift register.

The arrangement has two important consequences. To each node belong precisely k (sets of) coverage fractions, corresponding to k successive directions, and to each node pertain only a few fixed stripes. The ring has —at least conceptually— a phase counter q ; at each rotation over 2π/N, q is stepped up or down mod K by 1 . At its transition between 0 and k–1 the pixel values are shifted (in the appropriate direction) to the next node. In actual fact, the value q is duplicated so that under control of it each node can select the appropriate set of coverage fractions. Thus we have achieved the distribution of the main computation over a sparsely connected network of nodes with modest traffic between them.

Intentionally we have not called our nodes “microprocessors”, since a single microprocessor might take care of a number of nodes corresponding to a number of adjacent pixels. The pixels may also be taken from adjacent rings; if these rings have the same number of pixels —i.e. the same value for k — the cyclic traffic between microprocessors can be smoothed by choosing a homo- geneous distribution of their phases. It should be stressed that neither the decision what phase to assign to a ring nor the decision which nodes to combine in the same microprocessor affects in any way the freedom of choosing the pixels for the discretization of the circular domain. The only two constraints for that discretization are, firstly, that in each ring the number of pixels is a divisor of N and, secondly, that all pixels in the same ring are congruent.

(I invented the above design while explaining the problem at a privatissimum —known as “The Tuesday Afternoon Club”— ; the above description is a joint effort of drs.A.J.M. van Gasteren and me.)

11 February 1982
prof.dr.Edsger W.Dijkstra,

Burroughs Research Fellow/

Buitengewoon Hoogleraar THE,

Plataanstraat 5

5671 AL NUENEN

Transcribed by Hamilton Richards

Revised
06-May-2016
