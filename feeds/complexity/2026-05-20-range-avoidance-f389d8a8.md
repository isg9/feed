---
title: Range Avoidance
url: https://blog.computationalcomplexity.org/2026/05/range-avoidance.html
published: "2026-05-20T17:49:48Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-8474824991091188818
---

# Range Avoidance

Let \\(f\\) be a function mapping binary strings of length \\(m\\) to strings of length \\(n\\) with \\(n>m\\). Since there are more strings of length \\(n\\) than \\(m\\), \\(f\\) is not onto. Can you find a string not in the range? This is known as the range avoidance problems, or AVOID for short

Let's do an example. Consider \\(f\\) that outputs an undirected graph on \\(n\\) nodes and takes as input:

- \\(n\\)
- A set \\(S\\) of \\(k\\) vertices \\(v\_1,\\ldots,v\_k\\)
- For each \\(i\\) and \\(j\\), \\(i<j\\), where at least one of \\(i\\) and \\(j\\) are not in \\(S\\), whether or not there is an edge between \\(i\\) and \\(j\\).
- A bit \\(b\\)

If \\(b=1\\) output the graph \\(G\\) which consists of the given edges plus a clique on \\(S\\). If \\(b=0\\) the same but an independent set on \\(S\\).

For an appropriate choice of \\(k\\), about \\(2 \\log n\\), the output of \\(f\\) is longer than the input. Any graph not in the range is a Ramsey graph, i.e., no clique or independent set of size \\(k\\).

You can use AVOID to capture other probabilistic method constructions, high time-bounded Kolmogorov strings, Boolean functions with high circuit complexity, two-source extractors, rigid matrices, error-correcting codes and more.

AVOID started as the problem 1-Empty in [a paper](https://doi.org/10.4230/LIPIcs.ITCS.2021.44) by Robert Kleinberg, Oliver Korten, Daniel Mitropolsky and Christos Papadimitriou as an example of a total function problem higher up the polynomial-time hierarchy as verifying that a string not in the range is a co-NP question. Korten [made the connection](https://doi.org/10.1109/FOCS52979.2021.00051) to probabilistic constructions. Hanlin Ren, Rahul Santhanam and Zhikun Wang [popularized the problem](https://doi.org/10.1109/FOCS54457.2022.00067) and gave AVOID its current name. I learned about AVOID talking about the problem with Rahul and his students during my time at Oxford.

You can solve AVOID randomly using an NP oracle, at least half the strings are not in the range, and you can check that a string is not in the range in co-NP. Whether you can solve AVOID deterministically with an NP oracle remains open. Korten [showed that](https://doi.org/10.1109/FOCS52979.2021.00051) that problem is equivalent to circuit lower bounds for ENP.

Surendra Ghentiyala, Zeyong Li and Noah Stephens-Davidowitz [show that](https://eccc.weizmann.ac.il/report/2025/210/) any decision problem that reduces to AVOID sits in AM ∩ co-AM which strongly suggests AVOID is not NP-hard.

See Korten's [BEATCS survey](https://bulletin.eatcs.org/index.php/beatcs/article/view/825/876) for much more on the AVOID problem.
