---
title: Tracking down a memory leak in Ruby's EventMachine
url: https://blog.nelhage.com/2013/03/tracking-an-eventmachine-leak/
published: "2013-03-07T13:13:37Z"
feed: nelhage
guid: https://blog.nelhage.com/2013/03/tracking-an-eventmachine-leak/
---

# Tracking down a memory leak in Ruby's EventMachine

At Stripe, we rely heavily on ruby and EventMachine to power various internal and external services. Over the last several months, we’ve known that one such service suffered from a gradual memory leak, that would cause its memory usage to gradually balloon from a normal ~50MB to multiple gigabytes. It was easy enough to work around the leak by adding monitoring and restarting the process whenever memory usage grew too large, but we were determined to track down the root cause.
