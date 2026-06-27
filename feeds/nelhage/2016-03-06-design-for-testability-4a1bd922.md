---
title: Design for Testability
url: https://blog.nelhage.com/2016/03/design-for-testability/
published: "2016-03-06T10:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/2016/03/design-for-testability/
---

# Design for Testability

When designing a new software project, one is often faced with a glut of choices about how to structure it. What should the core abstractions be? How should they interact with each other? In this post, I want to argue for a design heuristic that I’ve found to be a useful guide to answering or influencing many of these questions: Optimize your code for testability Specifically, this means that when you write new code, as you design it and design its relationships with the rest of the system, ask yourself this question: “How will I test this code?
