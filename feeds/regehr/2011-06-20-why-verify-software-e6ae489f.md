---
title: Why Verify Software?
url: https://blog.regehr.org/archives/543
published: "2011-06-20T18:58:14Z"
feed: regehr
guid: http://blog.regehr.org/?p=543
---

# Why Verify Software?

![](https://blog.regehr.org/wp-content/uploads/2011/06/ariane5-e1308596371538.jpg) People like me who work on software verification (I’m using the term broadly to encompass static analysis, model checking, and traditional formal verification, among others) like to give talks where we show pictures of exploding rockets, stalled vehicles, inoperable robots, and crashed medical devices. We imply that our work is helping, or at least could help, prevent very serious software-related problems. It’s not clear that this sort of claim stands up to a close examination.

What would it take to show a causal link between verification efforts and software safety? A direct demonstration using a controlled experiment would be expensive. An indirect argument would need several parts. First, we’d have to show that flaws revealed by verification efforts are of the kind that could compromise safety. Second, we’d need to show that these flaws would not have been found prior to deployment by traditional V&V — those bugs are merely expensive, not harmful. Third, we’d need to argue that a given dollar spent on verification adds more safety than that same dollar spent on other ways of improving software. Finally, we would need to argue that a successful verification effort won’t have unintended consequences such as emboldening management to substantially accelerate the schedule.

Of course, none of this criticizes software verification research, which I work on and very much believe in. We simply need to be clear about its purpose, which is to reduce overall cost. A top-level goal for software verification that fails to mention cost (for example “minimize damage caused by bugs in software intensive systems”) is untenable because obviously the best way to minimize such damage is to radically simplify, or even eliminate, the software. Of course, in practice we do not wish to radically simplify or eliminate the software because it brings so many benefits.

A more reasonable high-level goal for software verification might be “increase, to the largest possible extent given the methods available, total system utility.” “Total system utility” has both positive and negative components, and verification is mainly about mitigating some of the negative components, or costs, including not just development and manufacturing costs, but also maintenance and accident-related costs. In the next couple of days I’ll post a more specific example where the cost-based analysis of verification is much more useful than the feel-good analysis promoted by exploding rocket pictures.
