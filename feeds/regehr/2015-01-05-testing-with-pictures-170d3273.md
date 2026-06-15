---
title: Testing with Pictures
url: https://blog.regehr.org/archives/1208
published: "2015-01-05T17:14:48Z"
feed: regehr
guid: http://blog.regehr.org/?p=1208
---

# Testing with Pictures

Testing code is fun and hard and looking at the problem in different ways is always good. Here’s a picture representing the behavior of a saturating subtraction operation, where the horizontal axes represent the inputs and the output is vertical:

![](https://blog.regehr.org/extra_files/saturating_ops_2014/2.png)

And here are some of the functions handed in by my students in the fall:

![](https://blog.regehr.org/extra_files/saturating_ops_2014/1.png)

![](https://blog.regehr.org/extra_files/saturating_ops_2014/10.png)

![](https://blog.regehr.org/extra_files/saturating_ops_2014/3.png)

![](https://blog.regehr.org/extra_files/saturating_ops_2014/4.png)

![](https://blog.regehr.org/extra_files/saturating_ops_2014/5.png)

![](https://blog.regehr.org/extra_files/saturating_ops_2014/6.png)

![](https://blog.regehr.org/extra_files/saturating_ops_2014/7.png)

![](https://blog.regehr.org/extra_files/saturating_ops_2014/8.png)

![](https://blog.regehr.org/extra_files/saturating_ops_2014/9.png)

![](https://blog.regehr.org/extra_files/saturating_ops_2014/0.png)

The last one represents one of the most common failure modes: failing to account for the asymmetry where INT\_MIN has no additive inverse. The thing that I like about these images is how glaringly obvious the bugs are.

Hey, who knew Gnuplot could do animated gifs now? I guess old dogs can learn new tricks.

![](https://blog.regehr.org/extra_files/saturating_ops_2014/add.gif)

(Note: I [posted some of these a few years ago](https://blog.regehr.org/archives/392), but I like the pictures so much I wanted to do it again.)
