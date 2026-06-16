---
title: Because It Doesn't Have To
url: https://blog.computationalcomplexity.org/2026/04/because-it-doesnt-have-to.html
published: "2026-04-29T13:50:04Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-9125418320439922410
---

# Because It Doesn't Have To

My favorite quote about networking came from Jim Kurose.

> The Internet works so well because it doesn't have to.

The IP and lower layers of the internet stack make no promises of delivery. Complete failure fulfills the protocol. This allows for simpler and more powerful protocols without the extra complexity needed to guarantee success. TCP aims for delivery basically by restarting the IP communication when it fails, and even TCP can report failure to the layers above.

We can say the same about modern artificial intelligence.

> Machine learning works so well because it doesn't have to.

With the softmax function that neural nets use to determine the probability of outputs, neural nets never completely rule out a possibility, always giving it at least some tiny probability. In cases where the complexity is just too difficult, neural nets give several possibilities with nontrivial probabilities, as I described in my [recent post](https://blog.computationalcomplexity.org/2026/04/machine-learning-and-complexity.html), where a machine learning model would generate a uniform distribution to capture the output of a pseudorandom generator. Instead of rigidly forcing the model to give us a specific answer, by looking at distributions we allow the models to make mistakes.

Thus a machine learning model can be correct when it makes probabilistic guesses in situations too complicated to solve directly, which allows it to achieve its best possible performance. Because we allow the models to make mistakes, they have the flexibility to solve complex problems far more frequently.
