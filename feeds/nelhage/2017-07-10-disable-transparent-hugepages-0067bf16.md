---
title: Disable Transparent Hugepages
url: https://blog.nelhage.com/post/transparent-hugepages/
published: "2017-07-10T21:15:00Z"
feed: nelhage
guid: https://blog.nelhage.com/post/transparent-hugepages/
---

# Disable Transparent Hugepages

tl;dr “Transparent Hugepages” is a Linux kernel feature intended to improve performance by making more efficient use of your processor’s memory-mapping hardware. It is enabled ("enabled=always") by default in most Linux distributions. Transparent Hugepages gives some applications a small performance improvement (~ 10% at best, 0-3% more typically), but can cause significant performance problems, or even apparent memory leaks at worst. To avoid these problems, you should set enabled=madvise on your servers by running
