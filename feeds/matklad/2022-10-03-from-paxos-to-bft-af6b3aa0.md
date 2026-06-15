---
title: From Paxos to BFT
url: https://matklad.github.io/2022/10/03/from-paxos-to-bft.html
published: "2022-10-03T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2022/10/03/from-paxos-to-bft.html
---

# From Paxos to BFT

Oct 3, 2022

This is a sequel to [Notes on Paxos](https://matklad.github.io/2020/11/01/notes-on-paxos.html) post. Similarly, the primarily goal here is for me to understand why the BFT consensus algorithm works in detail. This might, or might not be useful for other people! The Paxos article is a prerequisite, best to read that now, and return to this article tomorrow :)

Note also that while Paxos was more or less a direct translation of Lamport’s lecture, this post is a mish-mash oft the original BFT paper by Liskov and Castro, my own thinking, and a cursory glance as [this formalization](https://lamport.azurewebsites.net/tla/byzpaxos.html). As such, the probability that there are no mistakes here is quite low.

## [What is BFT?](\#What-is-BFT)

BFT stands for Byzantine Fault Tolerant consensus. Similarly to Paxos, we imagine a distributed system of computers communicating over a faulty network which can arbitrary reorder, delay, and drop messages. And we want computers to agree on some specific choice of value among the set of possibilities, such that any two computers pick the same value. Unlike Paxos though, we also assume that computers themselves might be faulty or malicious. So, we add a new condition to our list of bad things. Besides reordering, duplication, delaying and dropping, a fake message can be manufactured out of thin air.

Of course, if absolutely arbitrary messages can be forged, then no consensus is possible — each machine lives in its own solipsistic world which might be completely unlike the world of every other machine. So there’s one restriction — messages are cryptographically signed by the senders, and it is assumed that it is impossible for a faulty node to impersonate non-faulty one.

Can we still achieve consensus? As long as for each `f` faulty, malicious nodes, we have at least `2f + 1` honest ones.

Similarly to the Paxos post, we will capture this intuition into a precise mathematical statement about trajectories of state machines.

## [Paxos Revisited](\#Paxos-Revisited)

Our plan is to start with vanilla Paxos, and then patch it to allow byzantine behavior. Here’s what we’ve arrived at last time:

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

Safe(b, v) ≡
  ∃ q ∈ ℚ:
  let
    qmsgs  ≡ {m ∈ msgs: m.type = "1b" ∧ m.bal = b ∧ m.acc ∈ q}
    qvotes ≡ {m ∈ qmsgs: m.vote ≠ null}
  in
      ∀ a ∈ q: ∃ m ∈ qmsgs: m.acc = a
    ∧ (  qvotes = {}
       ∨ ∃ m ∈ qvotes:
             m.vote.val = v
           ∧ ∀ m1 ∈ qvotes: m1.vote.bal <= m.vote.bal)

Phase1a(b) ≡
    maxBal' = maxBal
  ∧ lastVote' = lastVote
  ∧ Send({type: "1a", bal: b})

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
  ∧ Safe(b, v)
  ∧ maxBal' = maxBal
  ∧ lastVote' = lastVote
  ∧ Send({type: "2a", bal: b, val: v})

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
```

Our general idea is to add some “evil” acceptors 𝔼 to the mix and allow them sending arbitrary messages, while at the same time making sure that the subset of “good” acceptors continues to run Paxos. What makes this complex is that we don’t know which acceptor are good and which are bad. So this is our setup

```
Sets:
  𝔹       -- Numbered set of ballots (for example, ℕ)
  𝕍       -- Arbitrary set of values
  𝔸       -- Finite set of good acceptors
  𝔼       -- Finite set of evil acceptors
  𝔸𝔼 ≡ 𝔸 ∪ 𝔼 -- All acceptors
  ℚ ∈ 2^𝔸𝔼 -- Set of quorums

  Msgs1a ≡ {type: {"1a"}, bal: 𝔹}

  Msgs1b ≡ {type: {"1b"}, bal: 𝔹, acc: 𝔸𝔼,
            vote: {bal: 𝔹, val: 𝕍} ∪ {null}}

  Msgs2a ≡ {type: {"2a"}, bal: 𝔹, val: 𝕍}

  Msgs2b ≡ {type: {"2b"}, bal: 𝔹, val: 𝕍, acc: 𝔸𝔼}

Assume:
  𝔼 ∩ 𝔸 = {}
  ∀ q1, q2 ∈ ℚ: q1 ∩ q2 ∩ 𝔸 ≠ {}
```

If previously the quorum condition was “any two quorums have an acceptor in common”, it is now “any two quorums have a good acceptor in common”. An alternative way to say that is “a byzantine quorum is a super-set of normal quorum”, which corresponds to the intuition where we are running normal Paxos, and there are just some extra evil guys whom we try to ignore. For Paxos, we allowed `f` faulty out of `2f + 1` total nodes with `f+1` quorums. For Byzantine Paxos, we’ll have `f` byzantine out `3f + 1` nodes with `2f+1` quorums. As I’ve said, if we forget about byzantine folks, we get exactly `f + 1` out of `2f + 1` picture of normal Paxos.

The next step is to determine behavior for byzantine nodes. They can send any message, as long as they are the author:

```
Byzantine(a) ≡
      ∃ b ∈ 𝔹:             Send({type: "1a", bal: b})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2a", bal: b, val: v})
    ∨ ∃ b1, b2 ∈ 𝔹, v ∈ 𝕍: Send({type: "1b", bal: b1, acc: a,
                                  vote: {bal: b2, val: v}})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2b", bal: b, val: v, acc: a})
  ∧ maxBal' = maxBal
  ∧ lastVote' = lastVote
```

That is, a byzantine acceptor can send any `1a` or `2a` message at any time, while for `1b` and `2b` the author should match.

What breaks? The most obvious thing is `Phase2b`, that is, voting. In Paxos, as soon as an acceptor receives a `2a` message, it votes for it. The correctness of Paxos hinges on the `Safe` check before we send `2a` message, but a Byzantine node can send an arbitrary `2a`.

The solution here is natural: rather than blindly trust `2a` messages, acceptors would themselves double-check the safety condition, and reject the message if it doesn’t hold:

```
Phase2b(a) ≡
  ∃ m ∈ msgs:
      m.type = "2a" ∧ maxBal(a) < m.bal
    ∧ Safe(m.bal, m.val)
    ∧ maxBal' = λ a1 ∈ 𝔸: if a = a1 then m.bal else maxBal(a1)
    ∧ lastVote' = λ a1 ∈ 𝔸: if a = a1
                              then {bal: m.bal, val: m.val}
                              else lastVote(a1)
    ∧ Send({type: "2b", bal: m.bal, val: m.val, acc: a})
```

Implementation wise, this means that, when a coordinator sends a `2a`, it also wants to include `1b` messages proving the safety of `2a`. But in the spec we can just assume that all messages are broadcasted, for simplicity. Ideally, for correct modeling you also want to model how each acceptor learns new messages, to make sure that negative reasoning about a certain message *not* being sent doesn’t creep in, but we’ll avoid that here.

However, just re-checking safety doesn’t fully solve the problem. It might be the case that several values are safe at a particular ballot (indeed, in the first ballot any value is safe), and it is exactly the job of a coordinator / `2a` message to pick one value to break the tie. And in our case a byzantine coordinator can send two `2a` for different valid values.

And here we’ll make the single non-trivial modification to the algorithm. Like the `Safe` condition is at the heart of Paxos, the `Confirmed` condition is the heart here.

So basically we expect a good coordinator to send just one `2a` message, but a bad one can send many. And we want to somehow distinguish the two cases. One way to do that is to broadcast ACKs for `2a` among acceptors. If I received a `2a` message, checked that the value therein is safe, and also know that everyone else received this same `2a` message, I can safely vote for the value.

So we introduce a new message type, `2ac`, which confirms a valid `2a` message:

```
Msgs2ac ≡ {type: {"2ac"}, bal: 𝔹, val: 𝕍, acc: 𝔸}
```

Naturally, evil acceptors can confirm whatever:

```
Byzantine(a) ≡
      ∃ b ∈ 𝔹:             Send({type: "1a", bal: b})
    ∨ ∃ b1, b2 ∈ 𝔹, v ∈ 𝕍: Send({type: "1b", bal: b1, acc: a,
                                 vote: {bal: b2, val: v}})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2a", bal: b, val: v})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2ac", bal: b, val: v, acc: a})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2b", bal: b, val: v, acc: a})
  ∧ maxBal' = maxBal
  ∧ lastVote' = lastVote
```

But, if we get a quorum of confirmations, we can be sure that no other value will be confirmed in a given ballot (each good acceptors confirms at most a single message in a ballot (and we need a bit of state for that as well))

```
Confirmed(b, v) ≡
  ∃ q ∈ ℚ: ∀ a ∈ q: {type: "2ac", bal: b, val: v, acc: a} ∈ msgs
```

Putting everything so far together, we get

Not Yet BFT Paxos

```
Sets:
  𝔹          -- Numbered set of ballots (for example, ℕ)
  𝕍          -- Arbitrary set of values
  𝔸          -- Finite set of acceptors
  𝔼          -- Finite set of evil acceptors
  𝔸𝔼 ≡ 𝔸 ∪ 𝔼 -- Set of all acceptors
  ℚ ∈ 2^𝔸𝔼   -- Set of quorums

  Msgs1a ≡ {type: {"1a"}, bal: 𝔹}

  Msgs1b  ≡ {type: {"1b"}, bal: 𝔹, acc: 𝔸,
             vote: {bal: 𝔹, val: 𝕍} ∪ {null}}

  Msgs2a  ≡ {type: {"2a"}, bal: 𝔹, val: 𝕍}
  Msgs2ac ≡ {type: {"2ac"}, bal: 𝔹, val: 𝕍, acc: 𝔸}

  Msgs2b  ≡ {type: {"2b"}, bal: 𝔹, val: 𝕍, acc: 𝔸}

Assume:
  𝔼 ∩ 𝔸 = {}
  ∀ q1, q2 ∈ ℚ: q1 ∩ q2 ∩ 𝔸 ≠ {}

Vars:
  -- Set of all messages sent so far
  msgs ∈ 2^(Msgs1a ∪ Msgs1b ∪ Msgs2a ∪ Msgs2ac ∪ Msgs2b)

  -- Function that maps acceptors to ballot numbers or -1
  -- maxBal :: 𝔸 -> 𝔹 ∪ {-1}
  maxBal ∈ (𝔹 ∪ {-1})^𝔸

  -- Function that maps acceptors to their last vote
  -- lastVote :: 𝔸 -> {bal: 𝔹, val: 𝕍} ∪ {null}
  lastVote ∈ ({bal: 𝔹, val: 𝕍} ∪ {null})^𝔸

  -- Function which maps acceptors to values they confirmed as safe
  -- confirm :: (𝔸, 𝔹) -> 𝕍 ∪ {null}
  confirm ∈ (𝕍 ∪ {null})^(𝔸 × 𝔹)

Send(m) ≡ msgs' = msgs ∪ {m}

Confirmed(b, v) ≡
  ∃ q ∈ ℚ: ∀ a ∈ q: {type: "2ac", bal: b, val: v, acc: a} ∈ msgs

Safe(b, v) ≡
  ∃ q ∈ ℚ:
  let
    qmsgs  ≡ {m ∈ msgs: m.type = "1b" ∧ m.bal = b ∧ m.acc ∈ q}
    qvotes ≡ {m ∈ qmsgs: m.vote ≠ null}
  in
      ∀ a ∈ q: ∃ m ∈ qmsgs: m.acc = a
    ∧ (  qvotes = {}
       ∨ ∃ m ∈ qvotes:
             m.vote.val = v
           ∧ ∀ m1 ∈ qvotes: m1.vote.bal <= m.vote.bal)

Byzantine(a) ≡
      ∃ b ∈ 𝔹:             Send({type: "1a", bal: b})
    ∨ ∃ b1, b2 ∈ 𝔹, v ∈ 𝕍: Send({type: "1b", bal: b1, acc: a,
                                 vote: {bal: b2, val: v}})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2a", bal: b, val: v})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2ac", bal: b, val: v, acc: a})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2b", bal: b, val: v, acc: a})
  ∧ maxBal' = maxBal
  ∧ lastVote' = lastVote
  ∧ confirm' = confirm

Phase1b(a) ≡
  ∃ m ∈ msgs:
      m.type = "1a" ∧ maxBal(a) < m.bal
    ∧ maxBal' = λ a1 ∈ 𝔸: if a = a1
                            then m.bal - 1
                            else maxBal(a1)
    ∧ lastVote' = lastVote
    ∧ confirm' = confirm
    ∧ Send({type: "1b", bal: m.bal, acc: a, vote: lastVote(a)})

Phase2ac(a) ≡
  ∃ m ∈ msgs:
      m.type = "2a"
    ∧ confirm(a, m.bal) = null
    ∧ Safe(m.bal, m.val)
    ∧ maxBal' = maxBal
    ∧ lastVote' = lastVote
    ∧ confirm' = λ a1 ∈ 𝔸, b1 \in 𝔹:
                 if a = a1 ∧ b1 = m.bal then m.val else confirm(a1, b1)
    ∧ Send({type: "2ac", bal: m.bal, val: m.val, acc: a})

Phase2b(a) ≡
  ∃ b ∈ 𝔹, v ∈ 𝕍:
      Confirmed(b, v)
    ∧ maxBal' = λ a1 ∈ 𝔸: if a = a1 then m.bal else maxBal(a1)
    ∧ lastVote' = λ a1 ∈ 𝔸: if a = a1
                              then {bal: m.bal, val: m.val}
                              else lastVote(a1)
    ∧ confirm' = confirm
    ∧ Send({type: "2b", bal: m.bal, val: m.val, acc: a})

Init ≡
    msgs = {}
  ∧ maxBal   = λ a ∈ 𝔸: -1
  ∧ lastVote = λ a ∈ 𝔸: null
  ∧ confirm = λ a ∈ 𝔸, b ∈ 𝔹: null

Next ≡
    ∃ a ∈ 𝔸:
        Phase1b(a) ∨ Phase2ac(a) ∨ Phase2b(a)
  ∨ ∃ a ∈ 𝔼:
        Byzantine(a)

chosen ≡
  {v ∈ V: ∃ q ∈ ℚ, b ∈ 𝔹: AllVotedFor(q, b, v)}

AllVotedFor(q, b, v) ≡
  ∀ a ∈ q: (a, b, v) ∈ votes

votes ≡
  let
    msgs2b ≡ {m ∈ msgs: m.type = "2b"}
  in
    {(m.acc, m.bal, m.val): m ∈ msgs2b}
```

In the above, I’ve also removed phases `1a` and `2a`, as byzantine acceptors are allowed to send arbitrary messages as well (we’ll need explicit `1a`/ `2a` for liveness, but we won’t discuss that here).

The most important conceptual addition is `Phase2ac` — if an acceptor receives a new `2a` message for some ballot with a safe value, it sends out the confirmation provided that it hadn’t done that already. In `Phase2b` then we can vote for confirmed values: confirmation by a quorum guarantees both that the value is safe at this ballot, and that this is a single value that can be voted for in this ballot (two different values can’d be confirmed in the same ballot, because quorums have an honest acceptor in common). This *almost* works, but there’s still a problem. Can you spot it?

The problem is in the `Safe` condition. Recall that the goal of the `Safe` condition is to pick a value `v` for ballot `b`, such that, if any earlier ballot `b1` concludes, the value chosen in `b1` would necessary be `v`. The way `Safe` works for ballot `b` in normal Paxos is that the coordinator asks a certain quorum to abstain from further voting in ballots earlier than `b`, collects existing votes, and uses those votes to pick a safe value. Specifically, it looks at the vote for the highest-numbered ballot in the set, and declares a value from it as safe (it *is* safe: it was safe at *that* ballot, and for all future ballots there’s a quorum which abstained from voting).

This procedure puts a lot of trust in that highest vote, which makes it vulnerable. An evil acceptor can just say that it voted in some high ballot, and force a choice of arbitrary value. So, we need some independent confirmation that the vote was cast for a safe value. And we can re-use `2ac` messages for this:

```
Safe(b, v) ≡
  ∃ q ∈ Q:
  let
    qmsgs  ≡ {m ∈ msgs: m.type = "1b" ∧ m.bal = b ∧ m.acc ∈ q}
    qvotes ≡ {m ∈ qmsgs: m.vote ≠ null}
  in
      ∀ a ∈ q: ∃ m ∈ qmsgs: m.acc = a
   ∧ (  qvotes = {}
       ∨ ∃ m ∈ qvotes:
             m.vote.val = v
           ∧ ∀ m1 ∈ qvotes: m1.vote.bal <= m.vote.bal
           ∧ Confirmed(m.vote.bal, v))
```

And … that’s it, really. Now we can sketch a proof that this thing indeed achieves BFT consensus, because it actually models normal Paxos among non-byzantine acceptors.

Phase1a messages of Paxos are modeled by Phase1a messages of BFT Paxos, as they don’t have any preconditions, the same goes for Phase1b. Phase2a message of Paxos is emitted when a value becomes confirmed in BFT Paxos. This is correct modeling, because BFT’s Safe condition models normal Paxos Safe condition (this … is a bit inexact I think, to make this exact, we want to separate “this value is safe” from “we are voting for this value” in original Paxos as well). Finally, Phase2b also displays direct correspondence.

As a final pop-quiz, I claim that the `Confirmed(m.vote.bal, v)` condition in `Safe` above can be relaxed. As stated, `Confirmed` needs a byzantine quorum of confirmations, which guarantees both that the value is safe and that it is the single confirmed value, which is a bit more than we need here. Do you see what would be enough?

The final specification contains this relaxation:

BFT Paxos

```
Sets:
  𝔹          -- Numbered set of ballots (for example, ℕ)
  𝕍          -- Arbitrary set of values
  𝔸          -- Finite set of acceptors
  𝔼          -- Finite set of evil acceptors
  𝔸𝔼 ≡ 𝔸 ∪ 𝔼 -- Set of all acceptors
  ℚ ∈ 2^𝔸𝔼   -- Set of quorums
  𝕎ℚ ∈ 2^𝔸𝔼  -- Set of weak quorums

  Msgs1a ≡ {type: {"1a"}, bal: 𝔹}

  Msgs1b  ≡ {type: {"1b"}, bal: 𝔹, acc: 𝔸𝔼,
             vote: {bal: 𝔹, val: 𝕍} ∪ {null}}

  Msgs2a  ≡ {type: {"2a"}, bal: 𝔹, val: 𝕍}
  Msgs2ac ≡ {type: {"2ac"}, bal: 𝔹, val: 𝕍, acc: 𝔸𝔸𝔼}

  Msgs2b  ≡ {type: {"2b"}, bal: 𝔹, val: 𝕍, acc: 𝔸𝔸𝔼}

Assume:
  𝔼 ∩ 𝔸 = {}
  ∀ q1, q2 ∈ ℚ: q1 ∩ q2 ∩ 𝔸 ≠ {}
  ∀ q ∈ 𝕎ℚ: q ∩ 𝔸 ≠ {}

Vars:
  -- Set of all messages sent so far
  msgs ∈ 2^(Msgs1a ∪ Msgs1b ∪ Msgs2a ∪ Msgs2ac ∪ Msgs2b)

  -- Function that maps acceptors to ballot numbers or -1
  -- maxBal :: 𝔸 -> 𝔹 ∪ {-1}
  maxBal ∈ (𝔹 ∪ {-1})^𝔸

  -- Function that maps acceptors to their last vote
  -- lastVote :: 𝔸 -> {bal: 𝔹, val: 𝕍} ∪ {null}
  lastVote ∈ ({bal: 𝔹, val: 𝕍} ∪ {null})^𝔸

  -- Function which maps acceptors to values they confirmed as safe
  -- confirm :: (𝔸, 𝔹) -> 𝕍 ∪ {null}
  confirm ∈ (𝕍 ∪ {null})^(𝔸 × 𝔹)

Send(m) ≡ msgs' = msgs ∪ {m}

Safe(b, v) ≡
  ∃ q ∈ ℚ:
  let
    qmsgs  ≡ {m ∈ msgs: m.type = "1b" ∧ m.bal = b ∧ m.acc ∈ q}
    qvotes ≡ {m ∈ qmsgs: m.vote ≠ null}
  in
      ∀ a ∈ q: ∃ m ∈ qmsgs: m.acc = a
    ∧ (  qvotes = {}
       ∨ ∃ m ∈ qvotes:
             m.vote.val = v
           ∧ ∀ m1 ∈ qvotes: m1.vote.bal <= m.vote.bal
           ∧ confirmedWeak(m.vote.val, v))

Confirmed(b, v) ≡
  ∃ q ∈ ℚ: ∀ a ∈ q: {type: "2ac", bal: b, val: v, acc: a} ∈ msgs

ConfirmedWeak(b, v) ≡
  ∃ q ∈ 𝕎ℚ: ∀ a ∈ q: {type: "2ac", bal: b, val: v, acc: a} ∈ msgs

Byzantine(a) ≡
      ∃ b ∈ 𝔹:             Send({type: "1a", bal: b})
    ∨ ∃ b1, b2 ∈ 𝔹, v ∈ 𝕍: Send({type: "1b", bal: b1, acc: a,
                                 vote: {bal: b2, val: v}})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2a", bal: b, val: v})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2ac", bal: b, val: v, acc: a})
    ∨ ∃ b ∈ 𝔹, v ∈ 𝕍:      Send({type: "2b", bal: b, val: v, acc: a})
  ∧ maxBal' = maxBal
  ∧ lastVote' = lastVote
  ∧ confirm' = confirm

Phase1b(a) ≡
  ∃ m ∈ msgs:
      m.type = "1a" ∧ maxBal(a) < m.bal
    ∧ maxBal' = λ a1 ∈ 𝔸: if a = a1
                            then m.bal - 1
                            else maxBal(a1)
    ∧ lastVote' = lastVote
    ∧ confirm' = confirm
    ∧ Send({type: "1b", bal: m.bal, acc: a, vote: lastVote(a)})

Phase2ac(a) ≡
  ∃ m ∈ msgs:
      m.type = "2a"
    ∧ confirm(a, m.bal) = null
    ∧ Safe(m.bal, m.val)
    ∧ maxBal' = maxBal
    ∧ lastVote' = lastVote
    ∧ confirm' = λ a1 ∈ 𝔸, b1 \in 𝔹:
                 if a = a1 ∧ b1 = m.bal then m.val else confirm(a1, b1)
    ∧ Send({type: "2ac", bal: m.bal, val: m.val, acc: a})

Phase2b(a) ≡
  ∃ b ∈ 𝔹, v ∈ 𝕍:
      confirmed(b, v)
    ∧ maxBal' = λ a1 ∈ 𝔸: if a = a1 then m.bal else maxBal(a1)
    ∧ lastVote' = λ a1 ∈ 𝔸: if a = a1
                              then {bal: m.bal, val: m.val}
                              else lastVote(a1)
    ∧ confirm' = confirm
    ∧ Send({type: "2b", bal: m.bal, val: m.val, acc: a})

Init ≡
    msgs = {}
  ∧ maxBal   = λ a ∈ 𝔸: -1
  ∧ lastVote = λ a ∈ 𝔸: null
  ∧ confirm = λ a ∈ 𝔸, b ∈ 𝔹: null

Next ≡
    ∃ b ∈ 𝔹:
        Phase1a(b) ∨ ∃ v ∈ 𝕍: Phase2a(b, v)
  ∨ ∃ a ∈ 𝔸:
        Phase1b(a) ∨ Phase2ac(a) ∨ Phase2b(a)
  ∨ ∃ a ∈ 𝔼:
        Byzantine(a)

chosen ≡
  {v ∈ V: ∃ q ∈ ℚ, b ∈ 𝔹: AllVotedFor(q, b, v)}

AllVotedFor(q, b, v) ≡
  ∀ a ∈ q: (a, b, v) ∈ votes

votes ≡
  let
    msgs2b ≡ {m ∈ msgs: m.type = "2b"}
  in
    {(m.acc, m.bal, m.val): m ∈ msgs2b}
```

TLA+ specs for this post are available here: [https://github.com/matklad/paxosnotes](https://github.com/matklad/paxosnotes).
