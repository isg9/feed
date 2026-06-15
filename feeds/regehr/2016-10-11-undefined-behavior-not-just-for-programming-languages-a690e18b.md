---
title: 'Undefined Behavior: Not Just for Programming Languages'
url: https://blog.regehr.org/archives/1438
published: "2016-10-11T14:38:27Z"
feed: regehr
guid: http://blog.regehr.org/?p=1438
---

# Undefined Behavior: Not Just for Programming Languages

This is an oldie but goodie. Start with this premise:

a = b

Multiply both sides by a:

a2 = ab

Subtract b2 from both sides:

a2 – b2 = ab – b2

Factor the left side:

(a + b)(a – b) = ab – b2

Factor the right side:

(a + b)(a – b) = b(a – b)

Divide both sides by (a – b) and cancel:

a + b = b

Substitute b for a:

b + b = b

Finally, let b = 1 and simplify:

2 = 1

I ran into this derivation when I was nine or ten years old and it made me deeply uneasy. The explanation, that you’re not allowed to divide by (a – b) because this term is equal to zero, seemed to raise more questions than it answered. How are we supposed to keep track of which terms are equal to zero? What if something is equal to zero but we don’t know it yet? What other little traps are lying out there, waiting to invalidate a derivation? This was one of many times where I noticed that in school they seemed willing to teach the easy version, and that the real world was never so nice, even in a subject like math where — you would think — everything is clean and precise.

Anyway, the point is that undefined behavior has been confusing people [for well over a thousand years](https://en.wikipedia.org/wiki/Division_by_zero#Early_attempts) — we shouldn’t feel too bad that we haven’t gotten it right in programming languages yet.
