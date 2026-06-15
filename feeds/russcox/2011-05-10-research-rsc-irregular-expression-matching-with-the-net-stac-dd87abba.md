---
title: 'research!rsc: Irregular expression matching with the .NET stack'
url: https://research.swtch.com/irregexp
published: "2011-05-10T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/irregexp
---

# research!rsc: Irregular expression matching with the .NET stack

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Irregular expression matching with the .NET stack Russ Cox May 10, 2011 *research.swtch.com/irregexp* Posted on Tuesday, May 10, 2011.

Python introduced a named capturing form in its regular expressions: `(?P...)` is like `(...)` but you refer to it as `\k'x'` instead of `\1`. Perl and .NET picked it up without the Pythonic P. They also let you say `(?'x'...)` instead of `(?...)`. (Perl: there's more than one way to do it!)

You can use these during the match just like any backreference; the value of `\k'x'` is the most recent text matched by `(?...)`. So `(?[0-9])+\k'd'` matches any digit string in which the last two digits are the same.

.NET introduces a variant on the named capture, `(?<-x>...)`, which, during the match, deletes the last captured substring for x, exposing the one that was there before. This implies that the named captures are stored on an internal per-name stack and it lets you do nested matching: `(?[0-9])+(\k'd'(?<-d>))+` matches the longest prefix of an even-length palindrome of digits, so `12345543` in `1234554399`.

There's a Perl addition that appears to have snuck into .NET undocumented, which is that you can say `(?(cond)then|else)` or just `(?(cond)then)` as a conditional expression: if `cond` is true, then it tries to match `then`, else `else` (or the empty string). The condition `1` means that there was a match for `\1`. In Perl the condition `` means that there was a match for the named back-reference `\k'name'`. In .NET it appears that the syntax is just `name`, by closer analogy with the numbered backreferences.

If you want full palindromes, you can insert a condition that there must be no matches left for `d` on the stack, by using `d` as the condition and `(?!)` \- which cannot match anything - as the true branch. The final thunderclap, then, is:

`(?[0-9])+(\k'd'(?<-d>))+(?(d)(?!))`

I haven't tried this, since I don't have easy access to .NET.

[This post](http://porg.es/blog/so-it-turns-out-that-dot-nets-regex-are-more-powerful-than-i-originally-thought) expands on the technique to match well-formed XML.

(Comments originally posted via Blogger.)

- [Wolfgang Kluge](http://gehirnwindung.de/)(November 15, 2011 2:25 PM) You can always use the index or the name in the .NET implementation. It's even valid to define a group with an index >0.

(?<4>.)

(?<5-4>a)

\\k<5>

(?(4)b\|c)

I don't like it, but it's possible...
