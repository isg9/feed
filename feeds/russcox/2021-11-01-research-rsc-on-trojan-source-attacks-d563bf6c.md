---
title: 'research!rsc: On “Trojan Source” Attacks'
url: https://research.swtch.com/trojan
published: "2021-11-01T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/trojan
---

# research!rsc: On “Trojan Source” Attacks

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# On “Trojan Source” Attacks Russ Cox November 1, 2021 *research.swtch.com/trojan* Posted on Monday, November 1, 2021.

There is a [paper making the rounds](https://www.trojansource.codes/trojan-source.pdf), with a [slick accompanying web site](https://www.trojansource.codes/), in which the authors describe a software supply chain attack they call “Trojan Source: Invisible Vulnerabilities”. In short, if you use comments containing Unicode LTR and RTL code points, which control whether text is rendered left-to-right or right-to-left, you can make code look different in a standard Unicode rendering than it does to a program ignoring the comments.

The authors claim this is “a new type of attack” that “cannot be perceived directly by human code reviewers” and “pose\[s\] an immediate threat”, and they propose that compilers should be “upgraded to block this attack.” None of this is true.

## [The attack is old](\#old)

First, the attacks are not new. Here is a [Go example from 2017](https://golang.org/issue/20209), a [Rust example from 2018](https://twitter.com/eggleroy/status/1006150385067855872), a [C++ example from 2019](https://twitter.com/zygoloid/status/1187150150835195905), and a [Ruby example from 2020](https://twitter.com/jupenur/status/1244286243518713857).

## [The change is visible](\#visible)

Second, it is technically true that the attack cannot be perceived directly by human code reviewers, but only in the sense that no program in a computer can be perceived directly by humans (except of course the [real programmers using magnetized needles and a steady hand](https://xkcd.com/378/)). Instead, we build tools that convert the electrical charges or magnetized particles that represent our programs into some more recognizable form, like a [pattern of lights](https://xkcd.com/722/). And of course we have complete control over the exact conversion those programs use and therefore exactly what they enable us to perceive, a fact I will return to later.

(Also, as [noted on Hacker News](https://news.ycombinator.com/item?id=29062322), any editor that syntax highlights comments will suffice to detect the examples in the paper!)

## [The threat is not immediate](\#threat_is_not_immediate)

Third, the attacks do not create an immediate threat, at least not a new one. The attack model in the paper assumes you take changes to your code from untrusted people. Not everyone does this! And the people who do take those changes are usually not reading the code carefully enough that this attack is necessary. In the [event-stream incident](https://blog.npmjs.org/post/180565383195/details-about-the-event-stream-incident.html), for example, many people focus on event-stream being hijacked by social engineering, while very few people focus on the fact that the malicious code was hidden by only including it in a “minimized” JavaScript file, a fact the Copay authors missed when upgrading their dependencies, if they looked at them at all. Perhaps most importantly, if you are letting untrusted people make arbitrary changes to your code, they can probably find [far more subtle](https://freedom-to-tinker.com/2013/09/20/software-transparency-debian-openssl-bug/) (and deniable!) ways to hide a malicious change than using LTR/RTL code points.

## [The compiler is the wrong place for a fix](\#compilers)

Finally, compilers should not be “upgraded” to block the attack. Fixing compilers closes only one of many open doors. What about assemblers? What about build and configuration file parsers? What about general YAML parsers? What about every other possible program that in some way executes the content of a text file that allows comments or quoted string literals?

For example, the Rust project issued 1.56.1 today, changing `rustc` to disallow LTR/RTL code points in Rust source files. But as far as I can tell there has been no update to `cargo`, meaning that a similar attack can probably still be carried out using comments and string literals in `Cargo.toml`.

Considering build configuration files alone, closing all the doors would mean updating the programs associated with `BUILD.bazel`, `CMakefile`, `Cargo.toml`, `Dockerfile`, `GNUmakefile`, `Makefile`, `go.mod`, `package.json`, `pom.xml`, `requirements.txt`, and many, many more. And again, that’s just build configuration. It would be an unbounded amount of work to require every program that interprets a text file to be changed to exclude a list of certain Unicode code points.

Even excluding “bad” Unicode code is pretty complicated. The next problem is [homograph attacks](https://github.com/golang/go/issues/20115), using characters that look the same but aren’t.

Changing compilers is also arguably a false sense of security. It’s [impossible in general](https://research.swtch.com/loopsnooper) for the compiler to tell whether code is good or bad, for almost any definition of good or bad. Even if the compiler rejects LTR/RTL code points, it can’t reject the many more subtle ways an attacker might introduce an unnoticed change.

## [The answer is proper review](\#review)

Instead of changing compilers, we should take advantage of the fact that we must use tools to render our programs for us—tools like text editors and code review web sites. We should make sure we are using those tools and also make sure their renderers make suspicious Unicode use easier to see. There are far fewer of those tools than there are programs that interpret or execute content from text files: most development efforts will have just one review tool and a few common editors.

Making LTR/RTL code points visible is particularly important for review tools, and it is good that [GitHub now warns about these characters in PRs](https://github.blog/changelog/2021-10-31-warning-about-bidirectional-unicode-text/). On the other hand, the blog post’s suggestion to use view UTF-8 files in VSCode as CP 437 to “see” Unicode sequences is a kludge. Hopefully VSCode will add a proper warning and visible indicators in UTF-8 mode soon too.

## [Summary](\#summary)

In summary: This is an old, already known bug. The source changes are hardly invisible: many programs can see them today, and no doubt more will be able to see them tomorrow. There is no new threat here: people who can modify the code you run can do a lot of damage already that not enough people watch for and that can be made arbitrarily subtle without Unicode tricks. Making the fix in every compiler is both a lot of work and not nearly good enough: it omits all the other programs that execute instructions from text files. Instead, the right answer to Unicode tricks is proper code review and dependency hygiene, which will handle a much broader class of problems that fully includes this one, especially using review tools that highlight suspicious Unicode, which isn’t limited to LTR/RTL attacks.

The authors of this paper have clearly done a good job promoting it. Kudos to them on that. But I am concerned that the attention and response this paper is getting is in general distracting from far more useful security efforts. We should redirect that attention and response at improving general code and dependency review instead.

The most egregious example of breathless coverage of this relatively minor issue is Brian Krebs’s headline: [‘Trojan Source’ Bug Threatens the Security of All Code](https://krebsonsecurity.com/2021/11/trojan-source-bug-threatens-the-security-of-all-code/). Imagine the writeup if he’d attended [Ken Thompson’s Turing Award Lecture](https://dl.acm.org/doi/pdf/10.1145/358198.358210). *That’s* an invisible vulnerability! (For the record, back in 2013, Ken told me that the backdoored compiler did in fact exist and was deployed, but it had a subtle bug that exposed the backdoor: the reproduction code added an extra `\0` to the string each time it copied itself. This made the compiler binary grow by one byte every time it rebuilt itself. Eventually someone noticed and debugged what was going on, aided by the fact that the “print assembly” compilation mode did not insert the backdoor at all, and all traces were cleaned away. Or so we believe.)
