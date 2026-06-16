---
title: Fun Little Solutions
url: https://blog.computationalcomplexity.org/2026/04/fun-little-solutions.html
published: "2026-04-05T15:32:00Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-435654699921431615
---

# Fun Little Solutions

Here are the solutions to [the problems I posted last week](https://blog.computationalcomplexity.org/2026/03/fun-little-problems.html).

### Problem 1

A language \\(L\\) is commutative if for all \\(u\\), \\(v\\) in \\(L\\), \\(uv = vu\\). Show that \\(L\\) is commutative if and only if \\(L\\) is a subset of \\(w^\*\\) for some string \\(w\\). The "only if" direction is surprisingly tricky.

**Answer**

For the "if" direction, suppose \\(L \\subseteq w^\*\\). Then every \\(u, v \\in L\\) can be written as \\(u = w^i\\) and \\(v = w^j\\), so \\(uv = w^{i+j} = vu\\).

For the "only if" direction, assume \\(L\\) is commutative. We may assume \\(L\\) contains a non-empty string. Let \\(u\\) be the shortest such string, let \\(m\\) be the greatest common divisor of the lengths of all non-empty strings in \\(L\\), and let \\(w\\) be the prefix of \\(u\\) of length \\(m\\). We use the following lemma.

**Lemma.** If \\(xy = yx\\) with \\(x, y\\) non-empty, then both are powers of their common prefix of length \\(\\gcd(\|x\|, \|y\|)\\).

**Proof of Lemma.** By strong induction on \\(\|x\| + \|y\|\\). If \\(\|x\| = \|y\|\\), comparing the first \\(\|x\|\\) characters gives \\(x = y\\), and both equal that string to the first power. If \\(\|x\| < \|y\|\\) (WLOG), comparing the first \\(\|x\|\\) characters of \\(xy\\) and \\(yx\\) shows \\(x\\) is a prefix of \\(y\\). Write \\(y = xz\\). Then \\(x(xz) = (xz)x\\) simplifies to \\(xz = zx\\). Since \\(\|x\| + \|z\| < \|x\| + \|y\|\\), the inductive hypothesis gives that \\(x\\) and \\(z\\) are powers of their common prefix of length \\(\\gcd(\|x\|, \|z\|) = \\gcd(\|x\|, \|y\|)\\). Since \\(y = xz\\), \\(y\\) is a power of this prefix too. \\(\\square\\)

Since \\(m\\) divides \\(\|u\|\\) and, for each \\(v \\in L\\), the lemma gives \\(u = r\_v^{\|u\|/\\gcd(\|u\|,\|v\|)}\\), combining these periodicities shows \\(u = w^{\|u\|/m}\\).

Now for any non-empty \\(v\\neq u \\in L\\), commutativity gives \\(uv = vu\\), so by the lemma, \\(u\\) and \\(v\\) are both powers of the prefix \\(r\\) of \\(u\\) of length \\(\\gcd(\|u\|, \|v\|)\\). Since \\(m\\) divides both \\(\|u\|\\) and \\(\|v\|\\), it divides \\(\|r\|\\), so \\(r\\) is itself just \\(w\\) repeated \\(\|r\|/m\\) times. Therefore \\(v = w^{\|v\|/m}\\). \\(\\square\\)

---

### Problem 2

Let an NP machine be sparse if for some \\(k\\), it has at most \\(n^k\\) accepting paths for every input of length \\(n\\). The class FewP is the set of languages accepted by sparse NP machines. Let \\(\\#N(x)\\) be the number of accepting paths of \\(N(x)\\). Show that if P = FewP then \\(\\#N(x)\\) is polynomial-time computable for any sparse NP machine \\(N\\).

**Answer**

The obvious approach is to create a machine \\(N'(x, k)\\) that accepts if there are at least \\(k\\) accepting paths of \\(N(x)\\). But this fails: if \\(N(x)\\) has \\(2n\\) accepting paths then \\(N'(x, n)\\) will have exponentially many accepting paths.

Instead, define \\(N'(x, w)\\) that accepts if \\(N(x)\\) has an accepting path starting with \\(w\\), and use tree search to find all the accepting paths.

---

### Problem 3

Let PERM(\\(L\\)) be the set of all permutations of the strings in \\(L\\). For example, PERM(\\(\\{000, 010\\}\\)) = \\(\\{000, 001, 010, 100\\}\\). Are regular languages closed under PERM? How about context-free languages?

**Answer**

Regular languages are *not* closed under PERM. Let \\(L = (01)^\*\\); then \\(\\mathrm{PERM}(L) \\cap 0^\*1^\* = \\{0^n 1^n\\}\\), which is not regular. Similarly, context-free languages are not closed under PERM in general: \\(\\mathrm{PERM}((012)^\*)\\) is not context-free.

However, over a binary alphabet \\(\\{a, b\\}\\), if \\(L\\) is context-free then \\(\\mathrm{PERM}(L)\\) is context-free. Over a binary alphabet, two strings are permutations of each other if and only if they have the same number of each letter, so \\(\\mathrm{PERM}(L)\\) depends only on the Parikh image \\(\\Pi(L) = \\{(\|u\|\_a, \|u\|\_b) : u \\in L\\}\\).

By [Parikh's theorem](https://en.wikipedia.org/wiki/Parikh%27s_theorem), \\(\\Pi(L)\\) is semilinear for any CFL \\(L\\), and every semilinear set is the Parikh image of some regular language \\(R\\). Thus \\(\\mathrm{PERM}(L) = \\mathrm{PERM}(R)\\), and it suffices to show \\(\\mathrm{PERM}(R)\\) is context-free for regular \\(R\\).

Given a DFA \\(A\\) for \\(R\\), construct a PDA that, on input \\(w\\), nondeterministically selects a rearrangement \\(u\\) of \\(w\\) while simulating \\(A\\) on \\(u\\). The stack tracks the running imbalance \\(\\Delta\_i = \|\\{a\\text{'s in } u\_1\\cdots u\_i\\}\| - \|\\{a\\text{'s in } w\_1 \\cdots w\_i\\}\|\\) in unary, while the finite control tracks the DFA state and sign of \\(\\Delta\_i\\). The PDA accepts iff the DFA reaches a state in \\(F\\) and the stack is empty (i.e., \\(\|u\|\_a = \|w\|\_a\\)), establishing \\(\\mathrm{PERM}(R)\\) is context-free.

---

### Problem 4

Suppose you have a one-tape Turing machine where we allow the transition function to move the head left, right, or stay put. Show there is an equivalent one-tape Turing machine that only moves the head left or right — and do it without increasing the size of the state space or tape alphabet.

**Answer**

For each pair \\((q, a)\\) with \\(\\delta(q, a) = (p, b, S)\\), precompute the *stay-closure*: keep applying \\(\\delta\\) while the move is \\(S\\). Since only the scanned cell changes, the process evolves on the finite set \\(Q \\times \\Gamma\\). Exactly one of three outcomes occurs: you reach \\((p', b', D)\\) with \\(D \\in \\{L, R\\}\\); you enter \\(q\_{\\mathrm{acc}}\\) or \\(q\_{\\mathrm{rej}}\\); or you fall into an \\(S\\)-only cycle. Define \\(\\delta'\\) by:

- if the first non-stay step is \\((p', b', D)\\), set \\(\\delta'(q, a) = (p', b', D)\\);
- if the closure halts, write the last \\(b'\\) and move (say \\(R\\)) into \\(q\_{\\mathrm{acc}}\\) or \\(q\_{\\mathrm{rej}}\\);
- if it \\(S\\)-loops, write any fixed symbol and move (say \\(R\\)) into \\(q\_{\\mathrm{rej}}\\).

Leave all non-\\(S\\) transitions unchanged. Then \\(Q\\) and \\(\\Gamma\\) are unchanged, no \\(S\\)-moves remain, and the accepted language is preserved.

---

### Problem 5

Let E be the set of problems solvable in time \\(2^{O(n)}\\). Show unconditionally that E \\(\\neq\\) NP.

**Answer**

EXP, the set of problems solvable in time \\(2^{n^{O(1)}}\\), has a complete problem that lies in E. So if E = NP then NP = EXP, which gives E = EXP, violating the time hierarchy theorem.

Note this proof does not say anything about whether or not E is contained in NP or vice versa.

---

### Problem 6

Show there is a computable list of Turing machines \\(M\_1, M\_2, \\ldots\\) such that \\(\\{L(M\_1), L(M\_2), \\ldots\\}\\) is exactly the set of computable languages.

**Answer**

This is impossible if the \\(M\_i\\) are all total Turing machines (halt on all inputs). But I never made that requirement.

Let \\(N\_1, N\_2, \\ldots\\) be the standard enumeration of all Turing machines. Define \\(M\_i(x)\\) to accept if \\(N\_i(y)\\) halts for all \\(y < x\\) and \\(N\_i(x)\\) accepts. If \\(N\_i\\) is total then \\(L(M\_i) = L(N\_i)\\). If \\(N\_i\\) is not total then \\(L(M\_i)\\) is finite and hence computable. Thus \\(\\{L(M\_1), L(M\_2), \\ldots\\}\\) contains all computable languages and no non-computable ones.
