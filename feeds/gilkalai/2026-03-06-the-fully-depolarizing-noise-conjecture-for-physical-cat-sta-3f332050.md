---
title: The Fully Depolarizing Noise Conjecture for Physical Cat States is Twenty Years Old!
url: https://gilkalai.wordpress.com/2026/03/06/the-fully-depolarizing-noise-conjecture-for-physical-cat-states-is-twenty-years-old/
published: "2026-03-06T08:46:21Z"
feed: gilkalai
guid: http://gilkalai.wordpress.com/?p=31285
---

# The Fully Depolarizing Noise Conjecture for Physical Cat States is Twenty Years Old!

*Amidst the war, I am holding onto hope for a future of peace and tolerance. At the end of the post I include a few pictures from the safe room (closet). Writing this also made me reflect on related posts* *[from 2017](https://gilkalai.wordpress.com/2017/09/19/friendship-and-sesame-maryam-and-marina-israel-and-iran/) and [2024](https://gilkalai.wordpress.com/2024/10/05/time-for-peace-song/).*

## The Fully Depolarizing Noise Conjecture (2006)

> ***Conjecture (Fully Depolarizing Noise for Bell States).***
>
> *For every physical implementation of a quantum computer, whenever a Bell state on a pair of physical qubits is created, there exists a small probability ![t>0](https://s0.wp.com/latex.php?latex=t%3E0&bg=ffffff&fg=333333&s=0&c=20201002) that both qubits are replaced by the maximally mixed state.*

Equivalently: the preparation of a Bell state (i.e., a two-qubit cat state) on two physical qubits necessarily carries a non-negligible probability that *both* qubits undergo fully depolarizing noise.

Twenty years ago I proposed that this phenomenon cannot be avoided by any method of preparing a Bell state on a pair of physical qubits. In particular, the conjecture applies to any pair of transmons in a Bell state in superconducting architectures. As far as I know, the conjecture remains open.

### What is a Bell state?

A Bell state (also called a two-qubit cat state) is a maximally entangled state of two qubits. A canonical example is

![\frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)](https://s0.wp.com/latex.php?latex=%5Cfrac%7B1%7D%7B%5Csqrt%7B2%7D%7D%28%7C00%5Crangle+%2B+%7C11%5Crangle%29&bg=ffffff&fg=333333&s=0&c=20201002)

The other Bell states are obtained from this one by local unitary transformations (for example, by applying Pauli operations to one of the qubits).

Bell states represent the simplest and most fundamental form of quantum entanglement. They are the basic resource behind quantum teleportation, entanglement swapping, and many constructions in quantum error correction.

**Remarks.**

1. In its original formulation, the conjecture was stated more broadly: it applied to every entangled pair of physical qubits, not necessarily maximally entangled ones. Moreover, the weight of the joint fully depolarizing component was conjectured to grow at least linearly with the amount of entanglement between the qubits. The discussion below implicitly relies on this more general formulation.

2. *(Added March 17.)* Bell states possess a special symmetry: two-qubit depolarization acting jointly on the pair is observationally indistinguishable from single-qubit depolarization acting on either qubit.

   **However, the conjecture concerns the underlying noise channel, not merely what can be inferred from Bell-state tomography.**

   This observation nevertheless highlights the importance of formulating and experimentally testing the conjecture for general entangled states, and not only for Bell pairs.

### Direct versus indirect creation of entanglement

For directly gated qubits, this behavior is already built into standard stochastic noise models. The novelty of the conjecture is the claim that this remains true with a similar error level even when the cat state is created indirectly. Quantitatively  I conjecture that the value of ![t](https://s0.wp.com/latex.php?latex=t&bg=ffffff&fg=333333&s=0&c=20201002) in the conjecture is in the ballpark of 2-gate fully depolarizing error.

For example, one may create a cat state on qubits 1 and 3 by:

- applying a CNOT on (1,2),
- then a CNOT on (2,3),
- and measuring qubit 2 (or uncomputing it).

In standard gate-level noise models, if each two-qubit gate independently depolarizes its participating pair with probability ![t](https://s0.wp.com/latex.php?latex=t&bg=ffffff&fg=333333&s=0&c=20201002), then:

- qubit 1 is replaced with probability ![t](https://s0.wp.com/latex.php?latex=t&bg=ffffff&fg=333333&s=0&c=20201002),
- qubit 3 is replaced with probability ![t](https://s0.wp.com/latex.php?latex=t&bg=ffffff&fg=333333&s=0&c=20201002),
- **both** are replaced only with probability ![t^2](https://s0.wp.com/latex.php?latex=t%5E2&bg=ffffff&fg=333333&s=0&c=20201002).

Thus, in such models, the probability that **both** entangled qubits are hit drops from order ![t](https://s0.wp.com/latex.php?latex=t&bg=ffffff&fg=333333&s=0&c=20201002) (direct gate) to order ![t^2](https://s0.wp.com/latex.php?latex=t%5E2&bg=ffffff&fg=333333&s=0&c=20201002) (indirect construction).

My conjecture asserts that nature does **not** permit such quadratic suppression. Even when entanglement is generated indirectly—through mediating qubits, measurement, feedforward, or clever circuit identities—the resulting Bell state on the two physical qubits still carries an intrinsic order-![t](https://s0.wp.com/latex.php?latex=t&bg=ffffff&fg=333333&s=0&c=20201002) probability that both qubits are replaced by the maximally mixed state.

Formulating the conjecture for state-of-the-art superconducting devices makes it more concrete. For superconducting quantum computers, the best rate of 2-gate errors is around 0.005 and we can assume that a large fraction of the noise is depolarizing channel hitting both qubits.  If the conjecture is correct for indirect entangled qubits, we will be able to identify fully depolarizing errors for pairs of entangled qubits of similar rate rather than two orders of magnitude smaller as suggested by the computation above.

### Physical intuition

The conjecture expresses, in a mathematically sharp way, a common physical intuition:

> Entanglement between two physical systems requires a genuine physical interaction, and such interaction inevitably exposes both systems to correlated noise.

For example, there is no indirect method of performing an entangling gate between two transmons (say) that is much more reliable than just directly interacting them. Standard gate-level noise modeling does **not** enforce this. Indeed, in simple circuit models (as we computed above) indirect constructions reduce fully depolarization errors from order ![t](https://s0.wp.com/latex.php?latex=t&bg=ffffff&fg=333333&s=0&c=20201002) to order ![t^2](https://s0.wp.com/latex.php?latex=t%5E2&bg=ffffff&fg=333333&s=0&c=20201002). Thus the conjecture postulates an additional, more subtle “nasty” property of physical noise—one that goes beyond standard local stochastic models. (I was myself recently [confused](https://scottaaronson.blog/?p=9425#comment-2021989) about whether this stronger behavior follows from standard simulations; [it does not](https://scottaaronson.blog/?p=9425#comment-2025917).)

### Localizable entanglement

Given a quantum state, the **localizable entanglement** ( [Verstraete–Popp–Cirac, 2004](https://arxiv.org/abs/quant-ph/0307009)) of a pair of qubits is defined as the maximum amount of entanglement that can be obtained between them when single-qubit measurements are performed on all remaining qubits and the outcomes are conditioned upon.

The conjecture extends to such **localizable entangled pairs**: whenever a pair of qubits exhibits non-zero localizable entanglement, there exists a small probability ![t>0](https://s0.wp.com/latex.php?latex=t%3E0&bg=ffffff&fg=333333&s=0&c=20201002) (depending linearly on the value of the localizable entanglement) that **both qubits are replaced by the maximally mixed (maximum-entropy) state.**

### Implications for larger entangled states

For more complicated entangled states—surface-code states, GHZ states, random-circuit sampling states, and cluster states—the extended conjecture applies to every pair of physical qubits that can exhibit localized entanglement. In several canonical families (such as GHZ states, cluster states, and surface-code states), this includes essentially every pair of qubits.

If true, this would have severe consequences for quantum error correction: correlated depolarizing noise on pairs of qubits is far more damaging than the quasi-independent noise assumed in threshold theorems. The reason is that a noise channel in which the events “qubit ![i](https://s0.wp.com/latex.php?latex=i&bg=ffffff&fg=333333&s=0&c=20201002) is depolarized” and “qubit ![j](https://s0.wp.com/latex.php?latex=j&bg=ffffff&fg=333333&s=0&c=20201002) is depolarized” are positively correlated for all pairs ![(i,j)](https://s0.wp.com/latex.php?latex=%28i%2Cj%29&bg=ffffff&fg=333333&s=0&c=20201002) necessarily leads to large-scale error synchronization.

## The state of the conjecture

As far as I know, the conjecture remains open.

Current NISQ-scale devices could in principle test it experimentally—even at noise levels above the fault-tolerance threshold. The challenge is not the scale, but the identification of the specific correlated fully-depolarizing component in the noise.

Recent demonstrations of distance-3, distance-5, and even distance-7 surface codes appear, at least superficially, to be in tension with the conjecture. Whether this tension is genuine or only apparent deserves careful examination. It will also be interesting to examine the situation for experimentally realized “large” GHZ states.

I look forward to the conjecture being carefully tested—and, of course, possibly refuted.

### Motivation

The original motivation for the conjecture was to “reverse engineer” natural structural conditions on noise that would cause quantum fault tolerance to fail.

For this reason, I [did not view](https://rjlipton.com/2012/01/30/perpetual-motion-of-the-21st-century/#comment-18029) the conjecture itself as a reason for people to revise their a priori beliefs about quantum computers. Rather, I regarded it as a concrete and testable benchmark for quantum devices — one that is meaningful both for small systems with only a handful of qubits and for larger systems. The conjecture is relevant even to systems operating at noise levels above the threshold required for fault tolerance.

### Sources

In March 2006 I [raised](https://dabacon.org/pontiff/2006/03/08/what-are-the-assumptions-of-classical-fault-tolerance/#comment-1872) an early version of the conjecture on Dave Bacon’s blog and discussed it with Dave Bacon, John Sidles, and others. The conjecture later appeared as **Postulate 1** in my 2006 paper *[How quantum computers can fail](https://arxiv.org/abs/quant-ph/0607021)* and in several subsequent works. The related “noise synchronization” consequence for highly entangled states appeared as **Postulate 2** and it connects back to my [2005 paper](https://arxiv.org/abs/quant-ph/0508095).

These postulates became Conjecture 3 and Conjecture 4 in my [2012 debate with Aram Harrow](http://rjlipton.wordpress.com/2012/01/30/perpetual-motion-of-the-21st-century/), where they played a central role. Conjectures 3 and 4 from the debate and a further consequence for the error rate (in terms of qubit errors) are Predictions 1,2, and 3 in my 2016 paper “ [The quantum computer Puzzle](https://www.ams.org/notices/201605/rnoti-p508.pdf)“.

**Remark:** In my papers I struggled to formulate the conjecture precisely and to identify the most appropriate notion of entanglement. Formulating the conjecture for localized entanglement (which I referred to as *emergent entanglement*) was proposed in my 2009 paper *“ [Quantum computers: Noise propagation and adversarial noise models.](https://arxiv.org/abs/0904.3265)“*

I also considered a weaker notion where the maximum amount of entanglement over single-qubit measurements is replaced by the average entanglement obtained from such measurements.

In some early papers I also formulated related conjectures for correlated *classical* noisy systems,  asserting that correlation for the “signal” implies correlation for the noise. In this generality classical computation provides simple counterexamples. Nevertheless, suitably adjusted versions of the conjecture appear to apply to several natural physical systems.

## Some counter arguments (and responses)

### 1) “This may be true for physical qubits, but it is false for logical qubits.”

Yes — the conjecture refers to **physical qubits**.

If the conjecture holds, it lowers the prospects for achieving reliable logical qubits. In fact, good-quality logical qubits would violate the Fully Depolarizing Noise Conjecture at the physical level. Thus the conjecture asserts that sufficiently good logical qubits are out of reach because the necessary physical behavior is unavailable.

### 2) “The conjecture does not describe a legitimate quantum channel (namely a channel supported by quantum mechanics).”

The conjecture does **not** specify a complete noise model. It imposes **constraints** on whatever the correct physical noise model is.

Even if universally true, the physical mechanisms leading to the conjectured behavior may differ between implementations. The conjecture is structural, not mechanistic.

### 3) “The conjecture violates  linearity of QM. It is possible that it will apply to one initial state of your quantum computer but not to another one.”

This is incorrect (while interesting). As I wrote above, the conjecture proposes constraints on the noise model rather than a complete description. If for the same circuit, with one initial condition the conjecture applies and for another initial condition the conjecture does not apply,  this does not violate linearity. Instead, it implies that if preparing these initial conditions makes no difference for the noise, then correlated depolarizing errors will appear in *both* cases.

### 4) The noise (or Nature) cannot “know” if the state is entangled or not. Entanglement cannot cause correlations for the noise.

The conjecture does not assume that noise “detects” entanglement or that entanglement directly “causes” correlation. It asserts that the physical processes required to generate entanglement inevitably produce correlated noise.

### 5) “The conjectured noise resembles nonphysical random-unitary models.”

Even if certain effective noise models implied by the conjecture appear unphysical, that does not refute the conjecture. It may instead indicate that certain large-scale entangled states are themselves physically unrealizable.

If creating a distance-15 surface code necessarily produces nonphysical types of noise, that would suggest that building such a code is itself nonphysical. The figure in my 2009 paper *When Noise Accumulates* was meant to illustrate precisely this point

[![](https://gilkalai.wordpress.com/wp-content/uploads/2026/02/2009-paper-fig2.png?w=640)](https://gilkalai.wordpress.com/wp-content/uploads/2026/02/2009-paper-fig2.png)

### 6) “The threshold theorem extends to general long-range noise.”

Yes — but these extensions still violate the conjecture. Mathematically speaking, many small long-range interaction terms are not the same as imposing substantial correlated depolarization.

### 7) “Entanglement is basis dependent.”

The conjecture refers to correlations in a fixed computational basis. There is no ambiguity here.

### 8) “Empirical independence in random circuit sampling shows that there are no unexpected nasty aspects of quantum noise.

No.

Fidelity measures (including XEB) primarily estimate the probability that **no error occurred**. They are not sensitive to correlated structure among the error events that *do* occur.

The conjecture concerns correlations between qubits conditioned on noise events, not the global fidelity of outputs.

### 9) If the error rate is ![p](https://s0.wp.com/latex.php?latex=p&bg=ffffff&fg=333333&s=0&c=20201002), the conjecture  implies ![\Omega(n^2 p)](https://s0.wp.com/latex.php?latex=%5COmega%28n%5E2+p%29&bg=ffffff&fg=333333&s=0&c=20201002) errors per round. (We expect ![\Omega(np)](https://s0.wp.com/latex.php?latex=%5COmega%28np%29&bg=ffffff&fg=333333&s=0&c=20201002) errors per round.)”

This observation captures an important feature of the conjecture.

In trace-distance terms, the total noise per round may scale linearly.

But in terms of **qubit-error counts**, error synchronization can produce quadratic scaling ![\Omega(n^2 p)](https://s0.wp.com/latex.php?latex=%5COmega%28n%5E2+p%29&bg=ffffff&fg=333333&s=0&c=20201002) in the number of correlated qubit hits.

This quadratic synchronization is precisely the phenomenon the conjecture predicts.

### 10) The conjecture is too vague; it does not explicitly describe the noise channel. It also does not describe the physical source of the noise and its exact modeling.

This is partially true. The conjecture is structural and not mechanistic (see further explanation below).

Testing the conjecture experimentally would require identifying in experimental data specific correlated fully-depolarizing components. Supporting it theoretically would require fine-grained modeling of concrete physical systems.

### 11) As long as *t* \> 0 is unspecified, then the conjecture might always stay open.

I conjecture that *t* is in the ballpark of the rate of 2-gate fully depolarizing error.

In contrast, (to the best of my memory) for fault tolerance quantum computers with ![n](https://s0.wp.com/latex.php?latex=n&bg=ffffff&fg=333333&s=0&c=20201002) physical qubits, for most pairs of qubits ![i,j](https://s0.wp.com/latex.php?latex=i%2Cj&bg=ffffff&fg=333333&s=0&c=20201002), the correlation between the events “qubit ![i](https://s0.wp.com/latex.php?latex=i&bg=ffffff&fg=333333&s=0&c=20201002) is hit by a depolarizing  error” and “qubit ![j](https://s0.wp.com/latex.php?latex=j&bg=ffffff&fg=333333&s=0&c=20201002) is hit by a depolarizing  error” tends to zero as ![n](https://s0.wp.com/latex.php?latex=n&bg=ffffff&fg=333333&s=0&c=20201002) tends to infinity.

### 12) The correlation conjecture (and my earlier line of research from 2005) has no direct bearing on topological quantum computing.

Right.

### 13) The conjecture represents the “physical-noise-is-conspiring position”.  However, Nature is not malicious.

We will see about that ![🙂](https://s0.wp.com/wp-content/mu-plugins/wpcom-smileys/twemoji/2/72x72/1f642.png) .

Remarks:

(to items 3,4) The possibility for systematic relation between the noiseless state and the noise was raised and discussed in my 2005 paper and through the years has led to interesting heated discussions.

(to item 5) noise models which resemble the behavior of random unitary operator were offered by early skeptical views. They do not violate the postulates of quantum mechanics but are considered unphysical: they violate how quantum physics is believed to be mapped into quantum mechanics.

(to item 6) Preskill’s 2012 [paper](https://arxiv.org/abs/1207.6131) (partially [triggered](https://rjlipton.com/2012/01/30/perpetual-motion-of-the-21st-century/#comment-17984) by our 2011 Caltech discussion) presented general conditions for the threshold theorem to hold and cites earlier papers in this direction. Robert Alicki opined that the conditions proposed by Preskill are violated for open quantum systems, and I [explained in this comment](https://rjlipton.com/2012/09/16/quantum-repetition/#comment-27003) a specific feature of Preskill’s very general noise models that I find physically questionable.

### Gated qubits and “purifying” gate errors

The assertion of the conjecture for gated qubits is (a rather vanilla) part of the standard model of noise for noisy quantum circuits and it is also clearly manifested by experimental data.  The negation of the conjecture would mean the ability to create entangled pairs of transmons (or other physical qubits) without any fully depolarizing errors. In effect, this would amount to “purifying” the two-qubit gate used to generate entanglement: starting with two-qubit gates that include fully depolarizing noise at rate ![t](https://s0.wp.com/latex.php?latex=t&bg=ffffff&fg=333333&s=0&c=20201002), one could nevertheless produce physical entangled pairs with no fully depolarizing component on the pair itself. The conjecture asserts that such purification is not physically possible.

### The conjecture is structural and not mechanistic

Some of the objections listed above (especially, 2,3,4,10) treat the conjecture as if it were proposing a specific mechanistic claim of the form: “Here is the physical source of the noise — e.g., microscopic Hamiltonian couplings, thermal photons, leakage, cross-talk —

and here is the dynamical derivation showing why fully depolarizing correlations appear. Such a claim would specify, the environment, the interaction model, the time evolution, and the exact channel arising from tracing out the bath. My conjecture is a structural claim of the form

> *Whenever two physical qubits can be prepared (or projected) into a Bell state,* *there is an intrinsic order-![t](https://s0.wp.com/latex.php?latex=t&bg=ffffff&fg=333333&s=0&c=20201002) fully depolarizing component acting jointly on them.*

This is a statement about the **form** of the effective channel, not about the physical process generating it. Of course, mechanistic explanations (that may differ for different implementations) that support the conjecture could be valuable.

### A Simple Probabilistic Model: Tail Bounds from Pairwise Marginals

To illustrate how strong pairwise correlations can force even large-scale error synchronization, consider the following simple probabilistic model. There is a probability distribution ![W](https://s0.wp.com/latex.php?latex=W&bg=ffffff&fg=333333&s=0&c=20201002) on ![\{0,1\}^n](https://s0.wp.com/latex.php?latex=%5C%7B0%2C1%5C%7D%5En&bg=ffffff&fg=333333&s=0&c=20201002). We generate a random bitstring ![w](https://s0.wp.com/latex.php?latex=w&bg=ffffff&fg=333333&s=0&c=20201002) according to the distribution ![W](https://s0.wp.com/latex.php?latex=W&bg=ffffff&fg=333333&s=0&c=20201002). If ![w_k=1](https://s0.wp.com/latex.php?latex=w_k%3D1&bg=ffffff&fg=333333&s=0&c=20201002) we fully depolarize qubit ![k](https://s0.wp.com/latex.php?latex=k&bg=ffffff&fg=333333&s=0&c=20201002).

Let ![X=(x_1,\dots,x_n)\in\{0,1\}^n](https://s0.wp.com/latex.php?latex=X%3D%28x_1%2C%5Cdots%2Cx_n%29%5Cin%5C%7B0%2C1%5C%7D%5En&bg=ffffff&fg=333333&s=0&c=20201002) be a random 0–1 vector of length ![n](https://s0.wp.com/latex.php?latex=n&bg=ffffff&fg=333333&s=0&c=20201002), and write ![S=\sum_{i=1}^n x_i](https://s0.wp.com/latex.php?latex=S%3D%5Csum_%7Bi%3D1%7D%5En+x_i&bg=ffffff&fg=333333&s=0&c=20201002). We assume that the joint distribution of every pair ![(x_i,x_j)](https://s0.wp.com/latex.php?latex=%28x_i%2Cx_j%29&bg=ffffff&fg=333333&s=0&c=20201002) (for ![i\neq j](https://s0.wp.com/latex.php?latex=i%5Cneq+j&bg=ffffff&fg=333333&s=0&c=20201002)) is fixed. The goal is to minimize the upper tail probability ![\Pr(S\ge \lceil sn\rceil)](https://s0.wp.com/latex.php?latex=%5CPr%28S%5Cge+%5Clceil+sn%5Crceil%29&bg=ffffff&fg=333333&s=0&c=20201002).

---

### Theorem (symmetric “one-parameter” pair law): Assume that for every ![i\neq j](https://s0.wp.com/latex.php?latex=i%5Cneq+j&bg=ffffff&fg=333333&s=0&c=20201002),

![\displaystyle \Pr(x_i=0,x_j=0)=1-t,\qquad \Pr(x_i=0,x_j=1)=\Pr(x_i=1,x_j=0)=\Pr(x_i=1,x_j=1)=\frac{t}{3}, ](https://s0.wp.com/latex.php?latex=%5Cdisplaystyle+%5CPr%28x_i%3D0%2Cx_j%3D0%29%3D1-t%2C%5Cqquad+%5CPr%28x_i%3D0%2Cx_j%3D1%29%3D%5CPr%28x_i%3D1%2Cx_j%3D0%29%3D%5CPr%28x_i%3D1%2Cx_j%3D1%29%3D%5Cfrac%7Bt%7D%7B3%7D%2C+&bg=ffffff&fg=333333&s=0&c=20201002)

where ![t\in[0,1]](https://s0.wp.com/latex.php?latex=t%5Cin%5B0%2C1%5D&bg=ffffff&fg=333333&s=0&c=20201002). Let ![S=\sum_{i=1}^n x_i](https://s0.wp.com/latex.php?latex=S%3D%5Csum_%7Bi%3D1%7D%5En+x_i&bg=ffffff&fg=333333&s=0&c=20201002) and ![K=\lceil sn\rceil](https://s0.wp.com/latex.php?latex=K%3D%5Clceil+sn%5Crceil&bg=ffffff&fg=333333&s=0&c=20201002). Then the minimum possible value of ![\Pr(S\ge K)](https://s0.wp.com/latex.php?latex=%5CPr%28S%5Cge+K%29&bg=ffffff&fg=333333&s=0&c=20201002) over all such distributions equals

![\displaystyle \min \Pr(S\ge K)= \begin{cases} \frac{4t}{3}\cdot\frac{n}{n+1}, & \text{if } K \le \frac{n+1}{2},\\[6pt] 0, & \text{if } K > \frac{n+1}{2}. \end{cases} ](https://s0.wp.com/latex.php?latex=%5Cdisplaystyle+%5Cmin+%5CPr%28S%5Cge+K%29%3D+%5Cbegin%7Bcases%7D+%5Cfrac%7B4t%7D%7B3%7D%5Ccdot%5Cfrac%7Bn%7D%7Bn%2B1%7D%2C+%26+%5Ctext%7Bif+%7D+K+%5Cle+%5Cfrac%7Bn%2B1%7D%7B2%7D%2C%5C%5C%5B6pt%5D+0%2C+%26+%5Ctext%7Bif+%7D+K+%3E+%5Cfrac%7Bn%2B1%7D%7B2%7D.+%5Cend%7Bcases%7D+&bg=ffffff&fg=333333&s=0&c=20201002)

(In particular, for large ![n](https://s0.wp.com/latex.php?latex=n&bg=ffffff&fg=333333&s=0&c=20201002) this is asymptotically ![\frac{4t}{3}](https://s0.wp.com/latex.php?latex=%5Cfrac%7B4t%7D%7B3%7D&bg=ffffff&fg=333333&s=0&c=20201002) for ![s<\tfrac12](https://s0.wp.com/latex.php?latex=s%3C%5Ctfrac12&bg=ffffff&fg=333333&s=0&c=20201002) and ![0](https://s0.wp.com/latex.php?latex=0&bg=ffffff&fg=333333&s=0&c=20201002) for ![s>\tfrac12](https://s0.wp.com/latex.php?latex=s%3E%5Ctfrac12&bg=ffffff&fg=333333&s=0&c=20201002).)

A proof and discussion of this implication will be given separately.

### Time-parametrization and smooth Lindblad evolutions

My conjecture asserts that for noisy quantum states and evolutions — when the noise level is low — there are systematic relations between the noise and the corresponding “ideal” state. (Such systematic relations were already explored in my 2005 paper.) For example, the noise in a quantum computation may reflect symmetries and statistical features of the underlying signal.

Over the years, I made several attempts to place these ideas into a broader mathematical framework, extending beyond the specific (and hypothetical) setting of quantum computers. One direction I proposed was the study of certain subclasses of Lindblad evolutions, which I referred to as **smoothed Lindblad evolutions**. Another direction involved introducing an intrinsic time-parametrization for noisy quantum evolutions. These ideas appear as Predictions 6 and 7 in my 2016 Notices of the AMS paper.

Both directions aim at understanding noise not as arbitrary perturbation, but as a structured phenomenon constrained by the evolving quantum state.

### My paper and discussion with Greg Kuperberg.

One offshoot of these conjectures — particularly the notion of smoothed Lindblad evolutions — and long email discussions with Greg Kuperberg led to a joint paper: [Contagious error sources would need time travel to prevent quantum computation.](https://arxiv.org/abs/1412.1907) Following ideas of Emanuel Knill, we proved that fault tolerance is possible even under very general forms of “disease-like” spreading noise.

Greg viewed this paper as a successful response to my skepticism. I did not quite see it that way. I saw our paper as clarifying an important point: if time-smoothing in my proposed class of Lindblad evolutions applies only forward in time, then fault tolerance can still succeed. To obstruct fault tolerance, the smoothing would need to extend to both past and future — effectively requiring a kind of “time-travel” correlation structure.

Recently, Greg jokingly proposed a joint mathematical collaboration (not necessarily on quantum topics) involving the two of us together with Gal Kronenberg and Guy Kindler. I think this is a wonderful idea.

### My argument against QC (2013-2020), and other related directions.

Between 2013 and 2020, I pursued a different skeptical direction regarding quantum computation. Together with Guy Kindler (initially in the context of Boson Sampling), I developed an argument based on computational complexity and the notion of **noise sensitivity**. This line of work suggests that NISQ devices cannot achieve “quantum supremacy.”

Unlike the Fully Depolarizing Noise Conjecture — which posits a structurally “nasty” type of correlated noise beyond standard modeling — this later argument relies entirely on standard noise assumptions. It explains why the extremely low error rates required for quantum supremacy and scalable fault tolerance may be beyond reach.

Naturally, the quantum-supremacy claims of the past six years directly challenge this position. The central issue is careful evaluation of experimental details. As far as I know, those claims have no direct bearing on the Fully Depolarizing Noise Conjecture.

Both skeptical directions generate near-term experimental predictions. Both are discussed in my 2016 paper “ [The quantum computer Puzzle](https://www.ams.org/notices/201605/rnoti-p508.pdf)“. While experimental validation is the primary testing ground, another possible avenue is to derive broader physical consequences beyond the specific framework of quantum computing.

Let me also note that the question of whether “NISQ computers” could deliver “quantum supremacy” arose in my debate with Aram Harrow — and in other discussions — before the terms “NISQ” and “quantum supremacy” were coined.

### Statistical testing

Since 2019, I have also been working (with Yosi Rinott, Tomer Shoham, and Carsten Voelkmann) on developing statistical tools for analyzing samples produced by NISQ experiments. Engaging directly with experimental data and statistical methodology may prove useful for testing the Fully Depolarizing Noise Conjecture as well.

One possible approach is to begin with a standard noise model, augment it with the hypothetical two-qubit depolarizing component (DEPOLARIZE2) predicted by the conjecture at some rate ![p](https://s0.wp.com/latex.php?latex=p&bg=ffffff&fg=333333&s=0&c=20201002), and then determine the value of ![p](https://s0.wp.com/latex.php?latex=p&bg=ffffff&fg=333333&s=0&c=20201002) that best matches empirical data. Here DEPOLARIZE2 refers to a two-qubit depolarizing channel acting jointly on the pair.

I recently had useful discussions with Craig Gidney about this type of simulation. Craig is optimistic that the skeptical position will be decisively refuted in the coming years. Statistical fitting of experimental samples to classes of ![W](https://s0.wp.com/latex.php?latex=W&bg=ffffff&fg=333333&s=0&c=20201002)-noise models (discussed earlier) may also provide empirical tests of the conjecture.

## Conclusion

The Fully Depolarizing Noise Conjecture was proposed twenty years ago as a structural stress test for quantum computing — a condition that scalable quantum devices must overcome. It does not attempt to describe a specific microscopic mechanism. Rather, it imposes a constraint on the effective noise channel: whenever physical qubits can generate entanglement — or localizable entanglement — correlated fully depolarizing noise must appear at linear order.

Whether this structural constraint reflects a fundamental limitation of quantum devices, or whether it will ultimately be refuted by experiment, remains an open question. The answer lies mainly in precise experimental scrutiny together with careful theoretical modeling and analysis.

[![](https://gilkalai.wordpress.com/wp-content/uploads/2026/03/safe-room2.jpeg?w=640)](https://gilkalai.wordpress.com/wp-content/uploads/2026/03/safe-room2.jpeg)[![](https://gilkalai.wordpress.com/wp-content/uploads/2026/03/safe-room1.jpeg?w=640)](https://gilkalai.wordpress.com/wp-content/uploads/2026/03/safe-room1.jpeg)

[![](https://gilkalai.wordpress.com/wp-content/uploads/2026/03/safe-room3.jpeg?w=640)](https://gilkalai.wordpress.com/wp-content/uploads/2026/03/safe-room3.jpeg)

A few pictures from the safe room (closet).

**Late remark:** There is a [lovely interview of Scott Aaronson with Yuval Boger](https://podcast.yboger.com/2026/03/02/scott-aaronson-professor-of-computer-science-ut-austin/) of QuEra. Yuval asked: *“I’ve heard you refer many times in your talks to an argument with Gil Kalai about whether large-scale quantum computing is even possible. Do you think he still has a path to being vindicated, or is it over?”* Scott gave a detailed and nice answer presenting his view of my position. (See also [this SO post](https://scottaaronson.blog/?p=9606).) The discussion nicely illustrates how this debate continues to evolve twenty years after the Fully Depolarizing Noise Conjecture was first proposed.
