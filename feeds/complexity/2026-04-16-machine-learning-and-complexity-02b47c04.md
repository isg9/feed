---
title: Machine Learning and Complexity
url: https://blog.computationalcomplexity.org/2026/04/machine-learning-and-complexity.html
published: "2026-04-16T12:45:00Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-5310274630571764023
---

# Machine Learning and Complexity

At Oxford I focused my research and discussions on how we can use the tools of computational complexity to help us understand the power and limitations of machine learning. Last week I posted my paper [How Does Machine Learning Manage Complexity?](https://arxiv.org/abs/2604.07233), a first step in this direction. Let me give a broad overview of the paper. Please refer to the paper for more technical details.

Instead of focusing on the machine learning concepts like neural nets, transformers, etc., I wanted to abstract out a model defined in terms of complexity, as time-efficient non-uniform computable distributions with minimum probabilities. Let's unpack this abstraction.

Neural nets as next token predictors don't always give the same next token, rather they give a distribution of tokens, which leads to a distribution on text of length up to the size of the context window. The way probabilities are computed via a [softmax function](https://en.wikipedia.org/wiki/Softmax_function) guarantee that every output can occur, with at least an exponentially small probability, the "minimum probability" in the abstraction.

In computational complexity, we study two main kinds of distributions. A distribution is *sampleable* if there is an algorithm that takes in uniformly random bits and outputs text according to that distribution. A distribution is *computable* if we can compute not only the probability that a piece of text occurs, but the [cumulative distribution function](https://en.wikipedia.org/wiki/Cumulative_distribution_function), the sum of the probabilities of all outputs lexicographically less than that text. Every efficiently computable distribution is efficiently sampleable but not likely the other way around.

Neural nets as next token predictors turn out to be equivalent to computable distributions. We need these kinds of distributions both on how neural nets are trained but also on how we use them. Computable distributions allow for conditional sampling which lets us use a large-language model to answer questions or have a conversation. You can't get conditional sampling from a sampleable distribution.

Neural nets have a limited number of layers, typically about a hundred layers deep which prevents them from handling full time-efficient (polynomial-time) computations. They can get around this restriction either with reasoning models that allow a model to talk to itself, or by directly writing and executing code which run efficient algorithms of any kind.

Most of the algorithms we use, think [Dijkstra](https://en.wikipedia.org/wiki/Dijkstra%27s_algorithm), or matching, are uniform, that is the same algorithm on graphs of twenty nodes as the algorithm on twenty million. But neural nets change their weights based on the distributions they train on. These weights encode a compressed view of that data and the extra computation needed to process that data. So we have different algorithms as our technology and data sources improve. I wrote more about this [connection between AI and nonuniformity](https://blog.computationalcomplexity.org/2025/10/ai-and-power-of-nonuniform-circuits.html) last fall.

What does this abstraction buy us?

Restricting to computable distributions forces machine learning algorithms to treat complex behavior as random behavior, much like we treat a coin flip as a random event because it is too complicated to work out all the physical interactions that would determine which side it lands on.

We illustrate this point by our main result. Let D be the distribution of outputs from a [cryptographic pseudorandom generator](https://en.wikipedia.org/wiki/Cryptographically_secure_pseudorandom_number_generator) and U be the uniform distribution of words of the same length. If \\(\\mu\\) is a time-efficient non-uniform computable distribution with minimum probabilities and \\(\\mu\\) does as good or a better job than U of modeling D then \\(\\mu\\) and U are essentially the same distribution. Machine learning models the complex PRG as a uniform distribution, simplifying what we can't directly compute by using randomness.

More in the [paper](https://arxiv.org/abs/2604.07233) and future blog posts.
