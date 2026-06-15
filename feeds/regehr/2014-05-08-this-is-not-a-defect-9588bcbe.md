---
title: This Is Not a Defect
url: https://blog.regehr.org/archives/1149
published: "2014-05-08T02:38:01Z"
feed: regehr
guid: http://blog.regehr.org/?p=1149
---

# This Is Not a Defect

In several previous blog entries I’ve mentioned that in some recent versions of C and C++, left-shifting a 1 bit into the high-order bit of a signed integer is an undefined behavior. In other words, if you have code that computes `INT_MIN` by evaluating `1<<31` (or `1<<(sizeof(int)*CHAR_BIT-1)` if you want to be that way) or code that does a byte swap using a subexpression like `c<<24`, then your program most likely has no meaning according to the standards. And in fact, Clang's integer sanitizer confirms that most non-trivial codes, [including several crypto libraries](https://blog.regehr.org/archives/1054), are undefined according to this rule.

An obvious fix is to tighten up the standard a bit and specify that shifting a 1 into the sign bit has the expected effect, which is what all compilers that I am aware of already do. [This is what the C++ standards committee is doing](http://www.open-std.org/jtc1/sc22/wg21/docs/papers/2012/n3367.html#1457), though as far as I can tell the fix doesn't officially take effect until a TC, a "technical corrigendum," is issued -- and even that doesn't finalize the thing, but it seems near enough in practice.

Anyhow, today Nevin Liber pointed out that there's a bit of news here, which is that last month the C standards committee decided that [this same issue is not a defect in C](http://www.open-std.org/jtc1/sc22/wg14/www/docs/dr_463.htm) and that they'll reconsider it later on, which I guess is fine since compiler implementers are ignoring this particular undefined behavior, but it seems like a bit of a missed opportunity to (1) make the language slightly saner and (2) bring it into line with the existing practice. Also you might consider perusing [the full set of defect reports](http://www.open-std.org/jtc1/sc22/wg14/www/docs/summary.htm) if you want to be thankful that you did something other than attend a standards meeting last month.
