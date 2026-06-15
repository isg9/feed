---
title: Writing Solid Code Week 2
url: https://blog.regehr.org/archives/1083
published: "2014-01-21T06:05:07Z"
feed: regehr
guid: http://blog.regehr.org/?p=1083
---

# Writing Solid Code Week 2

First, I wanted to thank everyone for the great discussion on the [post about week 1](https://blog.regehr.org/archives/1080). My single favorite thing about blogging is the discussion and involvement that these posts sometimes generate.

During week 2 we worked on issues relating to the triangle classifier. As several commenters predicted, getting correct results using floating point code was not easy. In fact, as of last Thurs, the only FP implementation that passed all test cases was [one supplied by reader David Eisenstat](https://github.com/regehr/solid_code_class/blob/master/triangle/eisenstat_fp/triangle.c). On Tuesday we went through the code of a randomly selected student; [here it is](https://github.com/regehr/solid_code_class/blob/master/triangle/joshkunz/triangle.c). Josh’s code is — we believe — correct. On Thursday we went through [David Eisenstat’s integer-based code](https://github.com/regehr/solid_code_class/blob/master/triangle/eisenstat_int/triangle.c), which was useful for several reasons. First, sometimes it’s easier to review code written by someone who isn’t actually present. Second, David’s code uses a couple of little C tricks — stringification and use of “static” in an array parameter — that I wanted to show the class. Third, except for a couple of little warts (I’d have preferred library calls for sorting an array and for converting a string to an int) his code is strong. We also went through a couple of people’s test scripts. Most students in the class used Python for test automation; a minority used Bash, one person used Ruby, and one student embedded his triangle code in a cool iOS app that classifies triangles interactively. [My voting-based tester](https://github.com/regehr/solid_code_class/blob/master/triangle/test_triangle.pl) is the only Perl code anyone has committed to the repo. Perl seems totally dead as far as younger developers are concerned, which makes me a tiny bit sad. I asked the students to wrap up their triangle classifiers so we can move on.

The project that I want to assign for week 3 is only about testing. I plan to give each student a random UNIX utility (grep, sed, ed, bc, etc.) based on the premise that it is a product that their company is evaluating for production use. Each student is to perform some testing and then either argue that the tool is good enough for serious use or that it is too buggy to be adopted. It doesn’t matter which conclusion they arrive at as long as the conclusion is supported by evidence. A negative argument could be supported by bugs or deviations from the spec; a positive argument could be supported by a solid testing strategy, coverage data, etc. Of course I pointed the class to the [classic Bart Miller papers](http://pages.cs.wisc.edu/~bart/fuzz/) evaluating the robustness of UNIX tools.

So far this class has been lots of fun and at least some of the students are enjoying it. I’ve gotten kind of obsessed with it and have been spending lots of time dreaming up projects that we probably won’t have time for.
