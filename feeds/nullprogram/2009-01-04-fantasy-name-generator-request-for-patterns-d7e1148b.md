---
title: 'Fantasy Name Generator: Request for Patterns'
url: https://nullprogram.com/blog/2009/01/04/
published: "2009-01-04T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2009/01/04/
---

# Fantasy Name Generator: Request for Patterns

## [Fantasy Name Generator: Request for Patterns](/blog/2009/01/04/)

January 04, 2009

nullprogram.com/blog/2009/01/04/

![](/img/misc/name-generation.jpg)

Whether choosing a name for my character in a fantasy game or populating a world which I pretend to myself that I will one day DM, I have always gone to the [RinkWorks Fantasy Name Generator](http://www.rinkworks.com/namegen/). The author of this tool, Samuel Stoddard, gives [some
history](http://www.rinkworks.com/namegen/history.shtml) on how he came to design and develop it.

It works by using pattern to select sets of letters to put together into a name. There is a thorough, [long
description](http://www.rinkworks.com/namegen/instr.shtml) on the website. Unfortunately, he didn't share his source code and so we can see how he did it.

Therefore, I used his description to duplicate his generator.

You can grab a copy here with [git](http://git.or.cz/),

```
git clone git://github.com/skeeto/fantasyname.git

```

It includes a command line interface as well as a web interface, which I am running and linked to at the beginning of this post for you to use. The code is available under the same license as Perl itself.

I used Perl and the [Parse::RecDescent](http://search.cpan.org/dist/Parse-RecDescent/lib/Parse/RecDescent.pm) parser generator. Thanks to this module, it essentially comes down to about 40 lines of code. The name pattern is executed, just like a computer program, to generate a name. Here is the BNF grammar I came up with,

```
LITERAL ::= /[^|()<>]+/

TEMPLATE ::= /[-svVcBCimMDd']/

literal_set ::= LITERAL | group

template_set ::= TEMPLATE | group

literal_exp ::= literal_set literal_exp | literal_set

template_exp ::= template_set template_exp | template_set

literal_list ::= literal_exp "|" literal_list | literal_exp "|" | literal_exp

template_list ::= template_exp "|" template_list | template_exp "|" | template_exp

group ::= "<" template_list ">" | "(" literal_list ")"

name ::= template_list | group

```

The program is just that, decorated with some bits of Perl. Since I came up with it, I have found that it is slightly different than Mr. Stoddard's generator, in that his allows empty sets anywhere. Mine only allows them at the end of lists. For example, this is valid for his generator,

```
<|B|C>(ikk)

```

But to work in mine, the empty item must be moved to the end,

```
(ikk)

```

This can be adjusted my making the proper changes to the grammar, which I haven't figured out yet.

Another problem with mine is that Parse::RecDescent is *slooooowwwww*. Ridiculously slow. Maybe I designed the grammar poorly? This is probably the biggest problem. Even simple patterns can take several seconds to generate names, specifically with deeply nested patterns. For example, this can take minutes,

```
<<<<<<>>>>>>

```

Before you go thinking you are going to tank my server, I have written the web interface so that it limits the running time of the generator. If you want to do something fancy, use your own hardware. ;-)

There is also a problem that it will silently drop invalid pattern characters at the end of the pattern. This has to do with me not quite understanding how to apply Parse::RecDescent yet.

And this is where I need your help. I have had some trouble coming up with good patterns. I don't even have a good default, generic fantasy name pattern. Here are some of mine,

```
 # default
D # idiot
 # short

```

None of which I am very satisfied.

You can design patterns for Nordic names, Gallic names, Tolkienesque Middle Earth names, orc names, idiot names, dragon names, dwarf names, elf names, Wheel of Time names, and so on. There is so much potential available with this tool.

To suggest one to me, e-mail me some patterns, or even better, clone my git repository and add one to it yourself (then ask me to pull from you). This way your credit will stay directly attached to it with a commit.

Good luck!

- [game](/tags/game/)
- [perl](/tags/perl/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Fantasy%20Name%20Generator:%20Request%20for%20Patterns) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Fantasy+Name+Generator%3A+Request+for+Patterns).

« [Don't Write Your Own E-mail Validator](/blog/2008/12/24/)

» [Git is Better](/blog/2009/02/12/)
