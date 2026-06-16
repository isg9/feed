---
title: "Monotonic demonstranda and dummy introduction (EWD1117)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1117.html
published: "1991-12-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1117.PDF
---

# Monotonic demonstranda and dummy introduction

When we have to prove

[f.exp]

for some exp and some monotonic f, it can help to know the following theorem

Theorem For monotonic f

(0)     [f.exp]  ≡  〈∀z : [exp ⇒ z]: [f.z] 〉     and

(1)     [f.exp]  ≡  〈∃z : [z ⇒ exp]: [f.z] 〉

A reason to use (0) is that [exp ⇒ z] is the form of expression in which we can manipulate exp  . An example is given in EWD1118.

A reason to use (1) is that [z ⇒ exp] is the form of conclusion we can draw about exp; if it exists, the strongest z satisfying [f.z] is a good candidate for a witness. An example is given in EWD1116.

This theorem is very simple, very general and probably equally applicable and useful. Why did it take me a lifetime to formulate it?

Austin, 1 December 1991

prof.dr E.W.Dijkstra, CS Dept., UT, Austin TX 78712 - 1188

Transcribed by Guy Haworth.

Last modified Tue, 1 Jan 2008.
