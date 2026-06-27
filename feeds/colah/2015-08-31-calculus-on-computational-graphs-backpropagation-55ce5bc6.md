---
title: 'Calculus on Computational Graphs: Backpropagation'
url: http://colah.github.io/posts/2015-08-Backprop/index.html
published: "2015-08-31T00:00:00Z"
feed: colah
guid: http://colah.github.io/posts/2015-08-Backprop/index.html
---

# Calculus on Computational Graphs: Backpropagation

Backpropagation is the key algorithm that makes training deep models computationally tractable. For modern neural networks, it can make training with gradient descent as much as ten million times faster, relative to a naive implementation. That’s the difference between a model taking a week to train and taking 200,000 years.

Beyond its use in deep learning, backpropagation is a powerful computational tool in many other areas, ranging from weather forecasting to analyzing numerical stability – it just goes by different names. In fact, the algorithm has been reinvented at least dozens of times in different fields (see [Griewank (2010)](http://www.math.uiuc.edu/documenta/vol-ismp/52_griewank-andreas-b.pdf)). The general, application independent, name is “reverse-mode differentiation.”

Fundamentally, it’s a technique for calculating derivatives quickly. And it’s an essential trick to have in your bag, not only in deep learning, but in a wide variety of numerical computing situations.

[Read more.](http://colah.github.io/posts/2015-08-Backprop/index.html)
