---
title: "A somewhat open letter to the Editor-in-Chief of Acta Informatica (EWD739)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD739.html
published: "1980-05-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD739.PDF
---

# A somewhat open letter to the Editor-in-Chief of Acta Informatica

EWD 739

A somewhat open letter to the Editor-in-chief of Acta Informatica.

Dear Professor Paul,

please do not interpret this letter as a request to revoke the rejection of our paper "Termination detection for diffusing computations": editors are free to decide. We are just sending you some of our comments on the reports from the referees.

Referee A seems to complain about the inclusion of a proof of the correctness of our solution. This is, in general, a very odd complaint, since the absence of a correctness proof would have been a more valid ground for objection. In this particular case the complaint is still more amazing, since the structure of the proof is the central theme of our paper. By considering this topic inappropriate for a scientific journal such as Acta Informatica he almost insults its readership.

The technical "criticism" of referee B boils down to the remark that inadequate implementations of our scheme might fail to display the desired behaviour. He blames us for the absence of implementation details, which are, however, irrelevant to the quintessence of our scheme. We have, for instance, intentionally left unspecified whether messages and signals are transmitted instantaneously or may "crawl", since —in contrast to other designs— our scheme may be implemented under either circumstance. To provide the additional synchronization that would be required if buffers are bounded and to derive the minimum buffer sizes then required for the absence of deadlock is a problem we have intentionally left out: firstly, it is not the subject of this paper, and secondly, it is trivial because its solution is absolutely standard. We believe to have used the well-known device of abstraction to good advantage.

*               *

*

We do not pretend that our paper is without shortcomings. On the contrary, after the 15 months (!) that the refereeing of our paper (of less than 2000 words) has taken, we ourselves would like to rephrase at least one sentence. The paper has been tersely written —perhaps even too much so— and must be hard to read for those that erroneously assume half of the sentences to be noise. One thing is certain: the suggestion, made by both referees, that this paper could be improved by severe condensation is ridiculous.

Please transmit our comments to the referees; two additional copies of this letter have been included for your convenience.

With our greetings and best wishes,

yours sincerely

prof.dr.Edsger W.Dijkstra

Burroughs

Plataanstraat 5

5671 AL NUENEN

The Netherlands
drs.C.S.Scholten

Philips Research Laboratories

5656 AA EINDHOVEN

The Netherlands

transcribed by Tristram Brelstaff

revised Thu, 14 Jul 2005
