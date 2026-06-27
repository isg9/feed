---
title: The architecture of declarative configuration management
url: https://blog.nelhage.com/post/declarative-configuration-management/
published: "2019-11-12T22:00:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/declarative-configuration-management/
---

# The architecture of declarative configuration management

With the ongoing move towards “infrastructure-as-code” and similar notions, there’s been an ongoing increase in the number and popularity of declarative configuration management tools. This post attempts to lay out my mental model of the conceptual architecture and internal layering of such tools, and some wishes I have for how they might work differently, based on this model. Background: declarative configuration management Declarative configuration management refers to the class of tools that allow operators to declare a desired state of some system (be it a physical machine, an EC2 VPC, an entire Google Cloud account, or anything else), and then allow the system to automatically compare that desired state to the present state, and then automatically update the managed system to match the declared state.
