---
title: Notes on Paxos
url: https://matklad.github.io/2020/11/01/notes-on-paxos.html
published: "2020-11-01T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2020/11/01/notes-on-paxos.html
---

# Notes on Paxos

Nov 1, 2020

These are my notes after learning the [Paxos](https://en.wikipedia.org/wiki/Paxos_%28computer_science%29) algorithm. The primary goal here is to sharpen my own understanding of the algorithm, but maybe someone will find this explanation of Paxos useful! This post assumes fluency with mathematical notation.

I must confess it took me a long time to understand distributed consensus. I’ve read a whole bunch of papers ( [Part Time Parliament](https://lamport.azurewebsites.net/pubs/pubs.html#lamport-paxos), [Paxos Made Simple](https://lamport.azurewebsites.net/pubs/pubs.html#paxos-simple), [Practical BFT](http://pmg.csail.mit.edu/pubs/castro99practical-abstract.html), [In Search of an Understandable Consensus Algorithm](https://raft.github.io/), [CASPaxos: Replicated State Machines without logs](https://arxiv.org/abs/1802.07000)), but they didn’t make sense. Or rather, nothing specific was unclear, but, at the same time, I was unable to answer the core question:

What breaks if this particular condition is removed?

That means that I didn’t actually understand the algorithm.

What finally made the whole thing click are

- [The TLA+ Video Course](https://lamport.azurewebsites.net/video/videos.html).

- [The Paxos Algorithm or How to Win a Turing Award](https://lamport.azurewebsites.net/tla/paxos-algorithm.html) lecture (I can’t believe I actually was in St. Petersburg at that time and missed this!)

I now think that the thing is actually much simpler than it is made to believe :-)

Buckle in, we are starting!

## [What is Paxos?](\#What-is-Paxos)

Paxos is an algorithm for implementing distributed consensus. Suppose you have `N` machines which communicate over a faulty network. The network may delay, reorder, and lose messages (it can not corrupt them though). Some machines might die, and might return later. Due to network delays, “machine is dead” and “machine is temporary unreachable” are indistinguishable. What we want to do is to make machines agree on some value. “Agree” here means that if some machine says “value is X”, and another machine says “value is Y”, then X necessary is equal to Y. It is OK for machine to answer “I don’t know yet”.

The problem with this formulation is that Paxos is an elementary, but subtle algorithm. To understand it (at least for me), a precise, mathematical formulation is needed. So, let’s try again.

What is Paxos? Paxos is a theorem about sets! This is definitely mathematical, and is true (as long as you base math on set theory), but is not that helpful. So, let’s try again.

What is Paxos? Paxos is a theorem about nondeterministic state machines!

A system is characterized by a state. The system evolves in discrete steps: each step takes system from `state` to `state'`. Transitions are non-deterministic: from a single current `s1`, you may get to different next states `s2` and `s3`. (non-determinism models a flaky network). An infinite sequence of system’s states is called a behavior:

```
state_0 → state_1 → ... → state_n → ...
```

Due to non-determinism, there’s a potentially infinite number of possible behaviors. Nonetheless, depending on the transition function, we might be able to prove that some condition is true for any state in any behavior.

Let’s start with a simple example, and also introduce some notation. I won’t use TLA+, as I don’t enjoy its concrete syntax. Instead, math will be set in monospaced unicode.

The example models an integer counter. Each step the counter decrements or increments (non-deterministically), but never gets too big or too small

Counter

```
Sets:
  ℕ -- Natural numbers with zero

Vars:
  counter ∈ ℕ

Init ≡
  counter = 0

Next ≡
    (counter < 9 ∧ counter' = counter + 1)
  ∨ (counter > 0 ∧ counter' = counter - 1)

Theorem:
  ∀ i: 0 ≤ counter_i ≤ 9

-- Notation
-- ≡: equals by definition
-- ∧: "and", conjunction
-- ∨: "or",  disjunction
```

The sate of the system is a single variable — `counter`. It holds a natural number. In general, we will represent a state of any system by a fixed set of variables. Even if the system logically consists of several components, we model it using a single unified state.

The `Init` formula specifies the initial state, the `counter` is zero. Note that `=` is a mathematical equality, and not an assignment. `Init` is a *predicate* on states.

`Init` is true for `{counter: 0}`.

`Init` is false for `{counter: 92}`.

`Next` defines a non-deterministic transition function. It is a predicate on pairs of states, `s1` and `s2`. `counter` is a variable in the `s1` state, `counter'` is the corresponding variable in the `s2` state. In plain English, transition from `s1` to `s2` is valid if one of these is true:

- Value of `counter` in `s1` is less than `9` and value of `counter` in `s2` is greater by 1.

- Value of `counter` in `s1` is greater than `0`, and value of `counter` in `s2` is smaller by 1.

`Next` is true for `({counter: 5}, {counter: 6})`.

`Next` is false for `({counter: 5}, {counter: 5})`.

Here are some behaviors of this system:

- `0 → 1 → 2 → 3 → 4 → 5 → 6 → 7 → 8 → 9`
- `0 → 1 → 0 → 1 → 0 → 1`
- `0 → 1 → 2 → 3 → 2 → 1 → 0`

Here are some **non** behaviors of this system:

- `1 → 2 → 3 → 4 → 5`: `Init` does not hold for initial state

- `0 → 2`: `Next` does not hold for `(0, 2)` pair

- `0 → 1 → 0 → -1`: `Next` does not hold for `(0, -1)` pair

“behavior” means that the initial state satisfies `Init`, and each transition satisfies `Next`.

We can state and prove a theorem about this system: for every state in every behavior, the value of counter is between 0 and 9. Proof is by induction:

- The condition is true in the initial state.

- If the condition is true for state `s1`, and `Next` holds for `(s1, s2)`, then the condition is true for `s2`.

- QED.

As usual with induction, sometimes we would want to prove a *stronger* property, because it gives us more powerful base for an induction step.

To sum up, we define a non-deterministic state machine using two predicates `Init` and `Next`. `Init` is a predicate on states which restricts possible initial states. `Next` is a predicate on *pairs* of states, which defines a non-deterministic transition function. `Vars` section describes the state as a fixed set of typed variables. `Sets` defines auxiliary fixed sets, elements of which are values of variables. `Theorem` section specifies a predicate on behaviors: *sequences* of steps evolving according to `Init` and `Next`.

The theorem does not automatically follow from `Init` and `Next`, it needs to be proven. Alternatively, we can simulate a range of possible behaviors on a computer and check the theorem for the specific cases. If the set of reachable states is small enough (finite would be a good start), we can enumerate *all* behaviors and produce a brute force proof. If there are too many reachable states, we can’t prove the theorem this way, but we often can prove it to be wrong, by finding a counter example. This is the idea behind model checking in general and TLA+ specifically.

## [What is Consensus?](\#What-is-Consensus)

Having mastered the basic vocabulary, let’s start slowly building towards Paxos. We begin with defining what consensus is. As this is math, we’ll do it using sets.

```
Sets:
  𝕍 -- Arbitrary set of values

Vars:
  chosen ∈ 2^𝕍 -- Subset of values

Theorem:
    ∀ i: |chosen_i| ≤ 1
  ∧ ∀ i, j: i ≤ j ∧ chosen_i ≠ {} ⇒ chosen_i = chosen_j

-- Notation
-- {}:  empty set
-- 2^X: set of all subsets of X, powerset
-- |X|: cardinality (size) of the set
```

The state of the system is a set of chosen values. For this set to constitute consensus (over time) we need two conditions to hold:

- at most one value is chosen

- if we choose a value at one point in time, we stick to it (math friendly: any two chosen values are equal to each other)

Here’s the simplest possible implementation of consensus:

Consensus

```
Sets:
  𝕍 -- Arbitrary set of values

Vars:
  chosen ∈ 2^𝕍 -- Subset of values

Init ≡
  chosen = {}

Next ≡
  chosen = {} ∧ ∃ v ∈ 𝕍: chosen' = {v}

Theorem:
    ∀ i: |chosen_i| ≤ 1
  ∧ ∀ i, j: i ≤ j ∧ (chosen_i ≠ {} ⇒ chosen_i = chosen_j)
```

In the initial state, the set of chosen values is empty. We can make a step if the current set of chosen values is empty, in which case we select an arbitrary value.

This technically breaks our behavior theory: we require behaviors to be infinite, but, for this spec, we can only make a single step. The fix is to allow empty steps: a step which does not change the state at all is always valid. We call such steps “stuttering steps”.

The proof of the first condition of the consensus theorem is a trivial induction. The proof of the second part is actually non-trivial, here’s a sketch. Assume that `i` and `j` are indices, which violate the condition. They might be far from each other in state-space, so we can’t immediately apply `Next`. So let’s choose the *smallest* `j1 ∈ [i+1;j]` such that the condition is violated. Let `i1 = j1 - 1`. The condition is still violated for `(i1, j1)` pair, but this time they are subsequent steps, and we can show that `Next` does not hold for them, concluding the proof.

Yay! We have a distributed consensus algorithm which works for 1 (one) machine:

Distributed Consensus For One Machine

```
Pick arbitrary value.
```

## [Simple Voting](\#Simple-Voting)

Let’s try to extend this to a truly distributed case, where we have `N` machines (“acceptors”). We start with formalizing the naive consensus algorithm: let acceptors vote for values, and select the value which gets a majority of votes.

Majority Vote

```
Sets:
  𝕍 -- Arbitrary set of values
  𝔸 -- Finite set of acceptors

Vars:
  votes ∈ 2^(𝔸×𝕍) -- Set of (acceptor, value) pairs

Init ≡
  votes = {}

Next ≡
  ∃ a ∈ 𝔸:
      ∃ v ∈ V: votes' = votes ∪ {(a, v)}
    ∧ ∀ v ∈ V: (a, v) ∉ votes

chosen ≡
  {v ∈ V: |{a ∈ 𝔸: (a, v) ∈ votes}| > |𝔸| / 2}
```

The state of the system is the set of all votes cast by all acceptors. We represent a vote as a pair of an acceptor and the value it voted for. Initially, the set of votes is empty. On each step, some acceptor casts a vote for some value (adds `(a, v)` pair to the set of votes), but only if it hasn’t voted yet. Remember that `Next` is a predicate on pairs of states, so we check `votes` for existing vote, but add a new one to `votes'`. The value is chosen if the set of acceptors which voted for the value ( `{a ∈ 𝔸: (a, v) ∈ votes}`) is at least half as large as the set of all acceptors. In other words, if a majority of acceptors has voted for the value.

Quiz

What would be the difference between

```
∃ a ∈ 𝔸:
    ∃ v ∈ V: votes' = votes ∪ {(a, v)}
  ∧ ∀ v ∈ V: (a, v) ∉ votes
```

and

```
∃ a ∈ 𝔸, v ∈ V:
    votes' = votes ∪ {(a, v)}
  ∧ ∀ (a1, v1) ∈ votes: a1 = a ⇒ v1 = v
```

?

Spoiler

Trick question!

They are equivalent. The first formula allows `a` to vote for `v` only if `a` hasn’t voted before. The second formula allows `a` to vote for `v` only if all previous votes of `a` were cast for `v`. That is, if `a` hasn’t voted yet, or if it has already voted for `v` (in which case this would be a stuttering step).

Let’s prove consensus theorem for Majority Vote protocol. TYPE ERROR, DOES NOT COMPUTE. The consensus theorem is a predicate on behaviors of states consisting of `chosen` variable. Here, `chosen` isn’t a variable, `votes` is! `chosen` is a function which maps current state to some boolean.

While it is intuitively clear what “consensus theorem” would look like for this case, let’s make this precise. Let’s *map* states with `votes` variable to states with `chosen` variable using the majority rule, `f`. This mapping naturally extends to a mapping between corresponding behaviors (sequences of steps):

```
  f(votes_0   →   votes_1  → ...)
= f(votes_0)  → f(votes_1) → ...
=  chosen_0   →  chosen_1  → ...
```

Now we can precisely state that for every behavior `B` of majority voting spec, the theorem holds for `f(B)`. This yields a better way to prove this! Instead of proving the theorem directly (which would again require i1, j1 trick), we prove that our mapping `f` is a homomorphism. That is, we prove that if `votes_0 → votes_1 → ...` is a behavior of the majority voting spec, then `f(votes_0) → f(votes_1) → ...` is a behavior of the consensus spec. This lets us to re-use existing proof.

The poof for initial step is trivial, but let’s spell it out just to appreciate the amount of details a human mind can glance through

```
  f({votes: {}})
= {chosen: {v ∈ V: |{a ∈ 𝔸: (a, v) ∈ {}}| > |𝔸| / 2}}
= {chosen: {v ∈ V: |{}| > |𝔸| / 2}}
= {chosen: {v ∈ V: 0 > |𝔸| / 2}}
= {chosen: {v ∈ V: FALSE}}
= {chosen: {}}
```

Let’s show that if Majority Vote’s `Next_m` holds for `(votes, votes')`, then Consensus’s `Next_c` holds for `(f(votes), f(votes'))`. There’s one obstacle on our way: this claim is false! Consider a case with three acceptors and two values: `𝔸 = {a1, a2, a3}`, `𝕍 = {v1, v2}`. Consider these values of `votes` and `votes'`:

```
votes  = {(a1, v1), (a2, v1), (a1, v2)}
votes' = {(a1, v1), (a2, v1), (a1, v2), (a3, v2)}
```

If you just mechanically check `Next`, you see that it works! `a3` hasn’t cast its vote, so it can do this now. The problem is that `chosen(votes) = {v1}` and `chosen(votes') = {v1, v2}`.

We are trying to prove too much! `f` works correctly only for states reachable from `Init`, and the bad value of `votes` where `a1` votes twice is not reachable.

So, we first should prove a lemma: each acceptor votes at most once. After that, we can prove `Next_m(votes, votes') = Next_c(f(votes), f(votes'))` under the assumption of at most once voting. Specifically, if `|f(votes')|` turns out to be larger than `1`, then we can pick two majorities which voted for different values, which allows to pin down a single acceptor which voted twice, which is a contradiction. The rest is left as an exercise for the reader :)

So, majority vote indeed implements consensus. Let’s look closer at the “majority” condition. It is clearly important. If we define `chosen` as

```
chosen ≡
  {v ∈ V: |{a ∈ 𝔸: (a, v) ∈ votes}| > 0}
```

then its easy to construct a behavior with several chosen values. The property of majority we use is that any two majorities have at least one acceptor in common. But any other condition with this property would work as well as majority. For example, we can assign an integer weight to each acceptor, and require the sum of weights to be more than half. As a more specific example, consider a set of for acceptors `{a, b, c, d}`.

Its majorities are:

```
{a, b, c, d}
{a, b, c}
{a, b, d}
{a, c, d}
{b, c, d}
```

But the following set of sets would also satisfy non-empty intersection condition:

```
{a, b, c, d}
{a, b, c}
{a, b, d}
{a, c}
{b, c}
```

Operationally, it is strictly better, as fewer are acceptors needed to reach a decision.

So let’s refine the protocol to a more general form.

Quorum Vote

```
Sets:
  𝕍       -- Arbitrary set of values
  𝔸       -- Finite set of acceptors
  ℚ ∈ 2^𝔸 -- Set of quorums

Assume:
  ∀ q1, q2 ∈ ℚ: q1 ∩ q2 ≠ {}

Vars:
  votes ∈ 2^(𝔸×𝕍) -- Set of (acceptor, value) pairs

Init ≡
  votes = {}

Next ≡
  ∃ a ∈ 𝔸:
      ∃ v ∈ V: votes' = votes ∪ {(a, v)}
    ∧ ∀ v ∈ V: (a, v) ∉ votes

chosen ≡
  {v ∈ V: ∃ q ∈ ℚ: AllVotedFor(q, v)}

AllVotedFor(q, v) ≡
  ∀ a ∈ q: (a, v) ∈ votes
```

We require to specify a set of quorums — set a of subsets of acceptors such that every two quorums have at least one acceptor in common. The value is chosen if there exists a quorum such that its every member voted for the value.

There’s one curious thing worth noting here. Consensus is a property of the whole system, there’s no single “place” where we can point to and say “hey, this is it, this is consensus”. Imagine 3 acceptors, sitting on Earth, Venus, and Mars, and choosing between values `v1` and `v2`. They can execute Quorum Vote algorithm without communicating with each other at all. They will necessary reach consensus without knowing which specific value they agreed on! An external observer can then travel to the three planets, collect the votes and discover the chosen value, but this feature isn’t built into the algorithm itself.

OK, so we’ve just described an algorithm for finding consensus among `N` machines, proved the consensus theorem for it, and noted that it has staggering communication efficiency: *zero* messages. Should we collect our Turing Award?

Well, no, there’s a big problem with Quorum Vote — it can get stuck. Specifically, if there are three values, and the votes are evenly split between them, then no value is chosen, and only stuttering steps are possible. If you can vote for different values, it might happen that neither value receives a majority of votes. Voting satisfies the safety property, but not the liveness property — the algorithm can get stuck even if all machines are on-line and communication is perfect.

There is a simple fix to the problem, with a rich historical tradition among many “democratic” governments. Let’s have a vote, and let’s pick the value chosen by the majority, but let’s allow to vote only for a single candidate value:

Rigged Quorum Vote

```
Sets:
  𝕍       -- Arbitrary set of values
  𝔸       -- Finite set of acceptors
  ℚ ∈ 2^𝔸 -- Set of quorums

Assume:
  ∀ q1, q2 ∈ ℚ: q1 ∩ q2 ≠ {}

Vars:
  votes ∈ 2^(𝔸×𝕍) -- Set of (acceptor, value) pairs

Init ≡
  votes = {}

Next ≡
  ∃ a ∈ 𝔸, v ∈ V:
      ∀ (a1, v1) ∈ votes: v1 = v
    ∧ votes' = votes ∪ {(a, v)}

chosen ≡
  {v ∈ V: ∃ q ∈ ℚ: AllVotedFor(q, v)}

AllVotedFor(q, v) ≡
  ∀ a ∈ q: (a, v) ∈ votes
```

The new condition says that an acceptor is only allowed to cast a vote if all other votes are for the same value. As a special case, if the set of votes is empty, the acceptor can vote for any value (but all other acceptors would have to vote for this value afterwards).

From a mathematical point of view, this algorithm is perfect. From a practical stand point, not so much: an acceptor to cast the first vote somehow needs to make sure that it is indeed the first one. The obvious fix to this problem is to assign a unique integer number to each acceptor, call the highest-numbered acceptor “leader”, and allow only the leader to cast the first decisive vote.

So acceptors first communicate with each other to figure out who the leader is, then the leader casts the vote, and the followers follow. But this also violates liveness: if the leader dies, then the followers would wait indefinitely. A fix for this problem is to let the second highest acceptor to take over the leadership if the leader perishes. But under our assumptions, it’s impossible to distinguish between a situation when the leader is dead from a situation when it just has a *really* bad internet connection. So naively picking successor would lead to a split vote and a standstill again (power transitions are known to be problematic for authoritarian regimes in real life too!). If only there were some kind of … distributed consensus algorithm for picking the leader!

## [Ballot Voting](\#Ballot-Voting)

This is the place were we start discussing real Paxos :-) It starts with a “ballot voting” algorithm. This algorithm, just like the ones we’ve already seen, does not define any messages. Rather, message passing is an implementation detail, so we’ll get to it later.

Recall that rigged voting requires all acceptors to vote for a single values. It is immune to split voting, but is susceptible to getting stuck when the leader goes offline. The idea behind ballot voting is to have many voting rounds, ballots. In each ballot, acceptors can vote only for a single value, so each ballot individually can get stuck. However, as we are running many ballots, some ballots will make progress. The value is chosen in a ballot if it is chosen by some quorum of acceptors. The value is chosen in an overall algorithm if it is chosen in some ballot.

The Turing award question is: how do we make sure that no two ballots choose different values? Note that it is OK if two ballots choose the same value.

Let’s just brute force this question, really. First, assume that the ballots are ordered (for example, by numbering them with natural numbers). And let’s say we want to pick some value `v` to vote for in ballot `b`. When `v` is safe? Well, when no other value `v1` can be chosen by any other ballot. Let’s tighten this up a bit.

Value `v` is safe at ballot `b` if any smaller ballot `b1` ( `b1 < b`) did not choose and will not choose any value other than `v`.

So yeah, easy-peasy, we *just* need to predict which values will be chosen in the future, and we are done! We’ll deal with it in a moment, but let’s first convince ourselves that, if we only select safe values for voting, we won’t violate consensus spec.

So, when we select a safe value `v` to vote for in a particular ballot, it might get chosen in this ballot. We need to check that it won’t conflict with any other value. For smaller ballots that’s easy — it’s the definition of safety condition. What if we conflict with some value `v1` chosen in a future ballot? Well, that value is also safe, so whoever chose `v1`, was sure that it won’t conflict with `v`.

How do we tackle the precognition problem? We’ll ask acceptors to commit to *not* voting in certain ballots. For example, if you are looking for a safe value for ballot `b` and know that there’s a quorum `q` such that each quorum member never voted in smaller ballots, and promised to never vote in smaller ballots, you can be sure that any value is safe. Indeed, any quorum in smaller ballots will have at least one member which would refuse to vote for any value.

Ok, but what if there’s some quorum member which has already voted for some `v1` in some ballot `b1 < b`? (Take a deep breath, the next sentence is the kernel of the core idea of Paxos). Well, that means that `v1` was safe at `b1`, so, if there will be no votes between `b1` and `b`, `v1` is also safe at `b`! (Exhale). In other words, to pick a safe value at `b` we:

1. Take some quorum `q`.

2. Make everyone in `q` promise to never vote in ballots earlier than `b`.

3. Among all of the votes already cast by the quorum members we pick the one with the highest ballot number.

4. If such vote exists, its value is a safe value.

5. Otherwise, any value is safe.

To implement the “never vote” promise, each acceptor will maintain `maxBal` value. It will never vote in ballots smaller or equal to `maxBal`.

Let’s stop hand-waving and put this algorithm in math. Again, we are not thinking about messages yet, and just assume that each acceptor can observe the state of the whole system.

Ballot Vote

```
Sets:
  𝔹       -- Numbered set of ballots (for example, ℕ)
  𝕍       -- Arbitrary set of values
  𝔸       -- Finite set of acceptors
  ℚ ∈ 2^𝔸 -- Set of quorums

Assume:
  ∀ q1, q2 ∈ ℚ: q1 ∩ q2 ≠ {}

Vars:
  -- Set of (acceptor, ballot, value) triples
  votes ∈ 2^(𝔸×𝔹×𝕍)

  -- Function that maps acceptors to ballot numbers or -1.
  -- maxBal :: 𝔸 -> 𝔹 ∪ {-1}
  maxBal ∈ (𝔹 ∪ {-1})^𝔸

Voted(a, b) ≡
  ∃ v ∈ 𝕍: (a, b, v) ∈ votes

Safe(v, b) ≡
  ∃ q ∈ ℚ:
      ∀ a ∈ q: maxBal(a) ≥ b - 1
    ∧ ∃ b1 ∈ 𝔹 ∪ {-1}:
          ∀ b2 ∈ [b1+1; b-1], a ∈ q: ¬Voted(a, b2)
        ∧ b1 = -1 ∨ ∃ a ∈ q: (a, b1, v) ∈ votes

AdvanceMaxBal(a, b) ≡
    maxBal(a) < b
  ∧ votes' = votes
  ∧ maxBal' = λ a1 ∈ 𝔸: if a1 = a then b else maxBal(a1)

Vote(a, b, v) ≡
    maxBal(a) < b
  ∧ ∀ (a1, b1, v1) ∈ votes: b = b1 ⇒ v = v1
  ∧ Safe(v, b)
  ∧ votes' = votes ∪ (a, b, v)
  ∧ maxBal' = λ a1 ∈ 𝔸: if a1 = a then b else maxBal(a1)

Init ≡
    votes = {}
  ∧ maxBal = λ a ∈ 𝔸: -1

Next ≡
  ∃ a ∈ 𝔸, b ∈ 𝔹:
      AdvanceMaxBal(a, b)
    ∨ ∃ v ∈ 𝕍: Vote(a, b, v)

chosen ≡
  {v ∈ V: ∃ q ∈ ℚ, b ∈ 𝔹: AllVotedFor(q, b, v)}

AllVotedFor(q, b, v) ≡
  ∀ a ∈ q: (a, b, v) ∈ votes

-- Notation
-- [b1;b2]: inclusive interval of ballots
--- Y^X: set of function from X to Y (f: X -> Y)
-- λ x ∈ X: y: function that maps x to y
-- ¬: "not", negation
--
-- f' = λ x1 ∈ X: if x1 = x then y else f(x1):
-- A tedious way to write that f' is the same function as f,
-- except on x, where it returns y instead.
--
-- I am sorry! In my defense, TLA+ notation for this
--- is also horrible :-)
```

Let’s unwrap this top-down. First, the `chosen` condition says that it is enough for some quorum to cast votes in some ballot for a value to be accepted. It’s trivial to see that, if we fix the ballot, then any two quorums would vote for the same value — quorums intersect. Showing that quorums vote for the same value in different ballots is the tricky bit.

The `Init` condition is simple — no votes, any acceptor can vote in any ballot (= any ballot with number larger than -1).

The `Next` consists of two cases. On each step of the protocol, some acceptor either votes for some value in some ballot `∃ v ∈ 𝕍: Vote(a, b, v)`, or declares that it won’t cast additional vote in small ballots `AdvanceMaxBal(a, b)`. Advancing ballot just sets `maxBal` for this acceptor (but takes care not to rewind older decisions). Casting a vote is more complicated and is predicated on three conditions:

- We haven’t forfeited our right to vote in this ballot.

- If there’s some vote in this ballot already, we are voting for the same value.

- If there are no votes, then the value should be safe.

Note that the last two checks overlap a bit: if the set of votes cast in a ballot is not empty, we immediately know that the value is safe: somebody has proven this before. But it doesn’t harm to check for safety again: a safe value can not become unsafe.

Finally, the safety check. It is done in relation to some quorum — if `q` proves that `v` is safe, than members of this quorum would prevent any other value to be accepted in early ballots. To be able to do this, we first need to make sure that `q` indeed finalized their votes for ballots less than `b` ( `maxBall` is at least `b - 1`). Then, we need to find the latest vote of `q`. There are two cases

- No one in `q` ever voted ( `b1 = -1`). In this case, there are no additional conditions on `v`, any value would work.

- Someone in `q` voted, and `b1` is the last ballot when someone voted. Then `v` must be the value voted for in `b1`. This implies `Safe(v, b1)`.

If all of these conditions are fulfilled, we cast our vote and advance `maxBall`.

This is the hardest part of the article. Take time to fully understand Ballot Vote.

Quiz

What breaks if we don’t advance `maxBall` in `Vote`? Ie, if we replace

```
maxBal' = λ a1 ∈ 𝔸: if a1 = a then b else maxBal(a1)
```

with just `maxBal' = maxBal`?

Spoiler

Trick question!

I believe that nothing really changes. Safety condition guarantees that no different value will be chosen in any *previous* ballot. However, by casting our own vote, we fix the outcome for the current ballot as well! If we don’t set `maxBal`, we can re-enter `Vote` and vote the second time, but we’ll necessary vote for the same value!

Voting the second time for the same value is wasteful, and upping `maxBall` here reduces the state space, but it doesn’t affect safety.

Rigorously proving that Ballot Voting satisfies Consensus would be tedious — the specification is large, and the proof would necessary use every single piece of the spec! But let’s add some hand-waving. Again, we want to provide homomorphism from Ballot Voting to Consensus. Cases where the image of a step is a stuttering step (the set of chosen values is the same) are obvious. It’s also obvious that the set of chosen values never decreases (we never remove votes, so a value can not become unchosen). It also increases by at most one value with each step.

The complex case is to prove that, if currently only `v1` is chosen, no other `v2` can be chosen as a result of the current step. Suppose the contrary, let `v2` be the newly chosen value, and `v1` be a different value chosen some time ago. `v1` and `v2` can’t belong to the same ballot, because every ballot contains votes only for a single value (this needs proof!). Lets say they belong to `b1` and `b2`, and that `b1 < b2`. Note that `v2` might belong to `b1` — nothing prevents smaller ballot from finishing later. When we chose `v2` for `b2`, it was safe. This means that some quorum either promised not to vote in `b1` (but then `v1` couldn’t have been chosen in `b1`), or someone from the quorum voted for `v2` in `b1` (but then `v1 = v2` (proving this might require repeated application of safety condition)).

Ok, but is this better than Majority Voting? Can Ballot Voting get stuck? No — if at least one quorum of machines is online, they can bump their `maxBall` to a ballot bigger than any existing one. After they do this, there necessary will be a safe value relative to this quorum, which they can then vote on.

However, Ballot Voting is prone to a live lock — if acceptors continue to bump `maxBal` instead of voting, they’ll never select any value. In fact, in the current formulation one needs to be pretty lucky to not get stuck. To finish voting, there needs to be a quorum which can vote in ballot `b`, but not in any smaller ballot, and in the above spec this can only happen by luck.

It is impossible to completely eliminate live locks without assumptions about real time. However, when we implement Ballot Voting with real message passing, we try to reduce the probability of a live lock.

## [Paxos for Real](\#Paxos-for-Real)

One final push left! Given the specification of Ballot Voting, how do we implement it using message passing? Specifically, how do we implement the logic for selecting the first (safe) value for the ballot?

The idea is to have a designated leader for each ballot. As there are many ballots, we don’t need a leader selection algorithm, and can just statically assign ballot leaders. For example, if there are N acceptors, acceptor 0 can lead ballots `0, N, 2N, …`, acceptor 1 can lead `1, N + 1, 2N + 1, …` etc.

To select a value for ballot `b`, the ballot’s leader broadcasts a message to initiate the ballot. Upon receiving this message, each acceptor advances its `maxBall` to `b - 1`, and sends the leader its latest vote, unless the acceptor has already made a promise to not vote in `b`. If the leader receives replies from some quorum, it can be sure that this quorum won’t vote in smaller ballots. Besides, the leader knows quorum’s votes, so it can pick a safe value.

In other words, the practical trick for picking a safe value is to ask some quorum to abstain from voting in small ballots and to pick a value consistent with votes already cast. This is the first phase of Paxos, consisting of two message types, 1a and 1b.

The second phase is to ask the quorum to cast the votes. The leader picks a safe value and broadcasts it for the quorum. Quorum members vote for the value, unless in the meantime they happened to promise to a leader of the bigger ballot to not vote. After a member voted, it broadcasts its vote. When a quorum of votes is observed, the value is chosen and the consensus is reached. This is the second phase of Paxos with messages 2a and 2b.

Let’s write this in math! To model message passing, we will use `msgs` variable: a set of messages which have ever been send. Sending a message is adding it to this set. Receiving a message is asserting that it is contained in the set. By not removing messages, we model reorderings and duplications.

The messages themselves will be represented by records. For example, phase 1a message which initiates voting in ballot `b` will look like this:

```
{type: "1a", bal: b}
```

Another bit of state we’ll need is `lastVote` — for each acceptor, what was the last ballot the acceptor voted in, together with the corresponding vote. It will be `null` if the acceptor hasn’t voted.

Without further ado,

Paxos

```
Sets:
  𝔹       -- Numbered set of ballots (for example, ℕ)
  𝕍       -- Arbitrary set of values
  𝔸       -- Finite set of acceptors
  ℚ ∈ 2^𝔸 -- Set of quorums

  -- Sets of messages for each of the four subphases
  Msgs1a ≡ {type: {"1a"}, bal: 𝔹}

  Msgs1b ≡ {type: {"1b"}, bal: 𝔹, acc: 𝔸,
            vote: {bal: 𝔹, val: 𝕍} ∪ {null}}

  Msgs2a ≡ {type: {"2a"}, bal: 𝔹, val: 𝕍}

  Msgs2b ≡ {type: {"2b"}, bal: 𝔹, val: 𝕍, acc: 𝔸}

Assume:
  ∀ q1, q2 ∈ ℚ: q1 ∩ q2 ≠ {}

Vars:
  -- Set of all messages sent so far
  msgs ∈ 2^(Msgs1a ∪ Msgs1b ∪ Msgs2a ∪ Msgs2b)

  -- Function that maps acceptors to ballot numbers or -1
  -- maxBal :: 𝔸 -> 𝔹 ∪ {-1}
  maxBal ∈ (𝔹 ∪ {-1})^𝔸

  -- Function that maps acceptors to their last vote
  -- lastVote :: 𝔸 -> {bal: 𝔹, val: 𝕍} ∪ {null}
  lastVote ∈ ({bal: 𝔹, val: 𝕍} ∪ {null})^𝔸

Send(m) ≡ msgs' = msgs ∪ {m}

Phase1a(b) ≡
    Send({type: "1a", bal: b})
  ∧ maxBal' = maxBal
  ∧ lastVote' = lastVote

Phase1b(a) ≡
  ∃ m ∈ msgs:
      m.type = "1a" ∧ maxBal(a) < m.bal
    ∧ maxBal' = λ a1 ∈ 𝔸: if a = a1
                            then m.bal - 1
                            else maxBal(a1)
    ∧ lastVote' = lastVote
    ∧ Send({type: "1b", bal: m.bal, acc: a, vote: lastVote(a)})

Phase2a(b, v) ≡
   ¬∃ m ∈ msgs: m.type = "2a" ∧ m.bal = b
  ∧ ∃ q ∈ ℚ:
    let
      qmsgs  ≡ {m ∈ msgs: m.type = "1b" ∧ m.bal = b ∧ m.acc ∈ q}
      qvotes ≡ {m ∈ qmsgs: m.vote ≠ null}
    in
        ∀ a ∈ q: ∃ m ∈ qmsgs: m.acc = a
      ∧ (  qvotes = {}
         ∨ ∃ m ∈ qvotes:
               m.vote.val = v
             ∧ ∀ m1 ∈ qvotes: m1.vote.bal <= m.vote.bal)
      ∧ Send({type: "2a", bal: b, val: v})
      ∧ maxBal' = maxBal
      ∧ lastVote' = lastVote

Phase2b(a) ≡
  ∃ m ∈ msgs:
      m.type = "2a" ∧ maxBal(a) < m.bal
    ∧ maxBal' = λ a1 ∈ 𝔸: if a = a1 then m.bal else maxBal(a1)
    ∧ lastVote' = λ a1 ∈ 𝔸: if a = a1
                              then {bal: m.bal, val: m.val}
                              else lastVote(a1)
    ∧ Send({type: "2b", bal: m.bal, val: m.val, acc: a})

Init ≡
    msgs = {}
  ∧ maxBal   = λ a ∈ 𝔸: -1
  ∧ lastVote = λ a ∈ 𝔸: null

Next ≡
    ∃ b ∈ 𝔹:
        Phase1a(b) ∨ ∃ v ∈ 𝕍: Phase2a(b, v)
  ∨ ∃ a ∈ 𝔸:
        Phase1b(a) ∨ Phase2b(a)

chosen ≡
  {v ∈ V: ∃ q ∈ ℚ, b ∈ 𝔹: AllVotedFor(q, b, v)}

AllVotedFor(q, b, v) ≡
  ∀ a ∈ q: (a, b, v) ∈ votes

votes ≡
  let
    msgs2b ≡ {m ∈ msgs: m.type = "2b"}
  in
    {(m.acc, m.bal, m.val): m ∈ msgs2b}

-- Notation
-- {f1: value1, f2: value}  -- a record with .f1 and .f2 fields
-- {f1: Set1, f2: Set2}     -- set of records
--- let name ≡ def in expr   -- local definition of name
```

Let’s go through each of the phases.

`Phase1a` initiates ballot `b`. It is executed by the ballot’s leader, but there’s no need to model who exactly the leader is, as long as it is unique. This stage simply broadcasts 1a message.

`Phase1b` is executed by an acceptor `a`. If `a` receives `1a` message for ballot `b` and it can vote in `b`, then it replies with its `lastVote`. If it can’t vote (it has already started some larger ballot), it simply doesn’t respond. If enough acceptors don’t respond, the ballot will get stuck, but some other ballot might succeed.

`Phase2a` is the tricky bit, it checks if the value `v` is save for ballot `b`.

First, we need to make sure that we haven’t already initiated `Phase2a` for this ballot. Otherwise, we might initiate `Phase2a` for different values. Here is the bit where it is important that the ballot’s leader is stable. The leader needs to remember if it has already picked a safe value.

Then, we collect 1b messages from some quorum (we need to make sure that every quorum member has send 1b message for this ballot). Value `v` is safe if the whole quorum didn’t vote ( `vote` is null), or if it is the value of the latest vote of some quorum member. We know that quorum members won’t vote in earlier ballots, because they had increased `maxBal` before sending 1b messages.

If the value indeed turns out to be safe, we broadcast 2a message for this ballot and value.

Finally, in `Phase2b` an acceptor `a` votes for this value, if its `maxBall` is still good. The bookkeeping is updating `maxBal`, `lastVote`, and sending the 2b message.

The set of 2b messages corresponds to the `votes` variable of the Ballot Voting specification.

## [Notes on Notes](\#Notes-on-Notes)

There’s a famous result called FLP impossibility: [Impossibility of Distributed Consensus with One Faulty Process](https://groups.csail.mit.edu/tds/papers/Lynch/jacm85.pdf). But we’ve just presented Paxos algorithm, which works as long as more than half of the processes are alive. What gives? FLP theorem states that there’s no consensus algorithm *with finite* *behaviors*. Stated in a positive way, any asynchronous distributed consensus algorithm is prone to live-lock. This is indeed the case for Paxos.

Liveness can be improved under partial synchronity assumptions. Ie, if we give each process a good enough clock, such that we can say things like “if no process fails, Paxos completes in `t` seconds”. If this is the case, we can fix live locking (ballots conflicting each other) by using naive leader selection algorithm to select the single acceptor which can initiate ballots. If we don’t reach consensus after `t` seconds, we can infer that someone has failed and re-run naive leader selection. If we are unlucky, naive leader selection will produce two leaders, but this won’t be a problem for safety.

Paxos requires atomicity and durability to function correctly. For example, once the has leader picked safe value and has broadcasted a 2a message, it should persist the selected value. Otherwise, if it goes down and then resurrects, it might choose a different value. How to make a choice of value atomic and durable? Write it to a local database! How to make local transaction atomic and durable? Write it first into the write ahead log? How to write something to WAL? Using the `write` syscall/DMA. What happens if the power goes down exactly in the middle of the write operation? Well, we can write a chunk of bytes with a checksum! Even if the write itself is not atomic, a checksummed write is! If we read the record from disk and checksum matches, then the record is valid.

I use slightly different definition of `maxBal` (less by one) than the one in the linked lecture, don’t get confused about this!

See [https://github.com/matklad/paxosnotes](https://github.com/matklad/paxosnotes) for TLA specs.
