---
title: "A beautiful proof of a probably useless theorem (with W.H.J.Feijen) (EWD415)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD04xx/EWD415.html
published: "1972-08-03T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd04xx/EWD415.PDF
---

# A beautiful proof of a probably useless theorem (with W.H.J.Feijen)

A beautiful proof of a probably useless theorem

by
Edsger W.Dijkstra and W.H.J.Feijen

Let a positive continuous function be defined on the Euclidean plane, such that its integral taken over the whole plane is finite, say = 1. Then there exists a set of rectangular axes such that the four integrals of the function, taken over the four quadrants respectively, are all equal to each other.

For a fixed orientation of the cross we can first move it in a direction perpendicular to one axis, such that eventually the surface integrals taken over the half planes at both sides of that one axis are equal to each other, a state of affairs that is not destroyed by moving thereafter the cross in a direction perpendicular to the other axis until also for that other axis the surface integrals at both sides are equal to each other. Such a so-called "balanced position" always exists and is uniquely determined by the orientation of the cross because the function integrated is positive. In a balanced position the surface integrals taken over opposite quadrants are equal: if they are both ¼ + α, then the other two must each be ¼ - α.

While keeping the relation "the cross is in a balanced position" invariant, the cross is now slowly rotated. The continuously changing value of the surface integral, taken over the quadrant encompassed by two chosen half-axes, that was originally = ¼ + α, must have been at least once = ¼ before we have rotated the cross over more than a right angle, for after a rotation over a right angle the cross has returned to its original balanced position, in which the continuously observed surface integral has the value ¼ - α.

dr.Edsger W.Dijkstra
BURROUGHS
Plataanstraat 5
NUENEN - 4565
The Netherlands ir.W.H.J.Feijen
Department of Mathematics
Technological University
P.O.Box 513
Eindhoven
The Netherlands

transcribed by Alejandro Ruiz
revised Wed, 12 Nov 2008
