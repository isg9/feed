---
title: Property-Based Testing Is Fuzzing
url: https://blog.nelhage.com/post/property-testing-is-fuzzing/
published: "2017-10-03T16:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/property-testing-is-fuzzing/
---

# Property-Based Testing Is Fuzzing

“Property-based testing” refers to the idea of writing statements that should be true of your code (“properties”), and then using automated tooling to generate test inputs (typically, randomly-generated inputs of an appropriate type), and observe whether the properties hold for that input. If an input violates a property, you’ve demonstrated a bug, as well as a convenient example that demonstrates it. A classic example of property-based testing is testing a sort function:
