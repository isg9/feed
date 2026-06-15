---
title: Writing Solid Code Week 4
url: https://blog.regehr.org/archives/1094
published: "2014-02-01T04:55:09Z"
feed: regehr
guid: http://blog.regehr.org/?p=1094
---

# Writing Solid Code Week 4

This week most of the students in the class (but not everyone, since we ran out of time) presented their arguments about the quality of various UNIX utility programs. This was a bit more interesting than it might otherwise have been since some of the utilities running on our student lab machines are hilariously old versions. Students subjected the tools to a wide variety of stress tests such as fuzzing, building them with Clang’s UB sanitizer, Valgrind, [Murphy](https://github.com/liblit/Murphy) (an awesome tool that I haven’t had time to try, but plan to), and more that I’m forgetting now. Additionally they inspected the source code, looked at how many bugs have been fixed in the utilities in recent years, etc. The point of this exercise was to get people used to the idea of assessing an externally-produced body of code using whatever methods are suitable and available.

The other thing we did was begin the next project. This one will have two phases but I’ve only told the students about the first one. With me playing the program manager we developed [the specification](https://github.com/regehr/solid_code_class/blob/master/compress/spec.txt). They will be developing this program in groups of three: a tester, a compression person, and an I/O person. I’m hoping that this code lends itself to writing assertions; if not, we’ll do something like balanced binary search trees later on, you can’t avoid assertions when writing those.
