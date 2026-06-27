---
title: Getting carried away with hack value
url: https://blog.nelhage.com/2010/05/hack-value/
published: "2010-05-23T19:53:30Z"
feed: nelhage
guid: https://blog.nelhage.com/2010/05/hack-value/
---

# Getting carried away with hack value

Recently, I’ve been working on some BarnOwl branches that move more of the core functionality of BarnOwl into perl code, instead of C (BarnOwl is written in an unholy mix of C and perl code that call each other back and forth obsessively). Moving code into perl has many advantages, but one problem is speed – perl code is obvious a lot slower than C, and BarnOwl has a lot of hot spots related to its tendency to keep tens or hundreds of thousands of messages in memory and loop over all of them in response to various commands.
