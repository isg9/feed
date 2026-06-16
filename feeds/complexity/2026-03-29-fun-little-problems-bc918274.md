---
title: Fun Little Problems
url: https://blog.computationalcomplexity.org/2026/03/fun-little-problems.html
published: "2026-03-29T20:14:00Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-5825055140457935046
---

# Fun Little Problems

Occasionally I run into what I consider fun problems in complexity, that require just a little bit of out of the box thinking. They require some background in theory, but nothing too deep. Some of these problems have been mentioned before on my blog or social media.

1. A language \\(L\\) is commutative if for all \\(u\\),\\(v\\) in \\(L\\), \\(uv=vu\\). Show that \\(L\\) is commutative if and only if \\(L\\) is a subset of \\(w^\*\\) for some string \\(w\\). The "only if" direction is surprisingly tricky.
2. Let an NP machine be sparse if for some \\(k\\), it has at most \\(n^k\\) accepting paths for every input of length \\(n\\). The class FewP is the set of languages accepted by sparse NP machines. Let \\(\\#N(x)\\) be the number of accepting paths of \\(N(x)\\). Show that if P=FewP then \\(\\#N(x)\\) is polynomial-time computable for any sparse NP machine \\(N\\).

Richard Beigel gave me this problem and told me the second thing I tried would work. He was right. 3. Let PERM(\\(L\\)) be the set of all permutations of the strings in \\(L\\). For example, PERM(\\(\\{000,010\\}\\)) is \\(\\{000,001,010,100\\}\\). Are regular languages closed under PERM? How about context-free languages? 4. Suppose you have a one-tape Turing machine where we allow the transition function to move the head left, right or stay put. Show there is an equivalent one-tape Turing machine that only moves the head left or right. Not hard, but now do it without increasing the size of the state space or tape alphabet.

5. Let E be the set of problem solvable in time \\(2^{O(n)}\\). Show unconditionally that E \\(\\ne\\) NP.
6. Show there is a computable list of Turing machines \\(M\_1,M\_2,\\ldots\\) such that \\(\\{L(M\_1),L(M\_2),\\ldots\\}\\) is exactly the set of computable languages. A computable list means there is a computable function \\(f\\) such that \\(f(i)\\) is a description of \\(M\_i\\).
