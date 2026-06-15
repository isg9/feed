---
title: 'Another Perl One-liner: Byte Order'
url: https://nullprogram.com/blog/2009/05/20/
published: "2009-05-20T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/05/20/
---

# Another Perl One-liner: Byte Order

## [Another Perl One-liner: Byte Order](/blog/2009/05/20/)

May 20, 2009

nullprogram.com/blog/2009/05/20/

At work right now I am using two different machines. One is a 32-bit SPARC and the other is a very powerful SMP x86-64. Sometimes data are generated on one machine and used in a simulation on the other. There is a problem of byte-order, though. The SPARC is big-endian and the other is little-endian, and the programs on both sides don't pay attention to that small detail.

Luckily, the data are all 4-byte aligned. That's perfect for a Perl one-liner byte order conversion,

```
perl -e 'print scalar reverse while read STDIN, $_, 4' < in.le > out.be

```

Perl is really great for concise hacks. I really like how this one-liner almost reads like a natural language sentence. Is there any other language that can do powerful one-liners like Perl?

- [perl](/tags/perl/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Another%20Perl%20One-liner:%20Byte%20Order) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Another+Perl+One-liner%3A+Byte+Order).

« [Emacs Web Server](/blog/2009/05/17/)

» [Getting Lisp](/blog/2009/05/23/)
