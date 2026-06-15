---
title: 'research!rsc: The Hideous Name'
url: https://research.swtch.com/name
published: "2008-02-18T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/name
---

# research!rsc: The Hideous Name

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# The Hideous Name Russ Cox February 18, 2008 *research.swtch.com/name* Posted on Monday, February 18, 2008.

In 1985, email addresses looked a lot different than they do today. They were full of now-foreign symbols like : ! and %, because each mailer had its own syntax and there was no translation between syntaxes at borders, resulting in addresses like

```
research!ucbvax!@cmu-cs-pt.arpa:@CMU-ITC-LINUS:dave%CMU-ITC-LINUS@CMU-CS-PT.

```

Rob Pike and Peter Weinberger used that address as the opening example in their paper “ [**The Hideous Name**](http://pdos.csail.mit.edu/~rsc/pike85hideous.pdf),” which examines good and bad examples of naming in both file systems and mail addresses. Name spaces, they argue, are most powerful when new semantics can be added without adding new syntax.

The mail examples in “The Hideous Name” are not particularly relevant anymore. Everyone has standardized on the two-level ARPANET `@` syntax, and the other syntaxes are used only by people looking for open mail relays. But the file system examples are more relevant today than they were 20 years ago.

Unix file names are the canonical example: a program that parses file names like `/etc/passwd` will have no problem parsing names of ‘special’ files like `/dev/stdin`. The latter is special only in semantics, not in syntax: if you run `cp /dev/stdin /tmp`, *cp* will have no trouble deciding the name of the target file ( `/tmp/stdin`). Contrast this with the convention of using `–` to mean standard input: what target should `cp – /tmp` use? In fact (on Linux, at least) it looks for a file named `–` rather than using standard input, illustrating another problem. Special syntax is often implemented by individual programs rather than the operating system, resulting in non-uniform handling of the names. It's hard to justify why (again, on Linux) `cat – >/tmp/a` means something different from `cp – /tmp/a`.

Providing uniform interpretation of names means having some central arbiter, tradtionally the kernel. Then all programs use the same name space, and one can apply old tools to new resources. For example, if other machines' file systems were mounted on `/n/` *system* `/`,

```
sed 's!:.*!!' /n/*/etc/passwd | sort -u

```

produces a list of all the user of all the machines. And if [backups are mounted](http://swtch.com/plan9port/man/man8/vbackup.html) on `/dump`,

```
ls -l /dump/am/2007/02*/etc/passwd

```

shows all the password files from last February. Special semantics, but no special syntax.

Better than changing the kernel for each file system is having a general mechanism by which the kernel can defer interpretation of particular file trees to other programs. [Plan 9](https://9p.io/sys/doc/9.html) was the first system to make extensive use of such user-level file servers. [FUSE](http://fuse.sourceforge.net/) now makes it possible to do the same on modern Unixes. For example, Amit Singh has built some [interesting](http://osxbook.com/book/bonus/chapter11/procfs/) [new](http://code.google.com/p/macfuse/wiki/MACFUSE_FS_SPOTLIGHTFS) [file](http://code.google.com/p/macfuse/wiki/MACFUSE_FS_SSHFS) [systems](http://code.google.com/p/macfuse/wiki/HELLOWORLDFS) for OS X using MacFUSE.

Pike and Weinberger end their paper with an observation from Doug McIlroy:

> bad notations can stifle progress. Roman numerals hobbled mathematics for a millennium but were propagated by custom and by natural deference to authority. Today we no longer meekly accept individual authority. Instead, we have “standards,” impersonal imprimaturs on convention. Some standards are sound and indispensable; some simply celebrate bureaucratic littleness of mind. A harvest of gimmicks to save appearances within the standard has grown up, then gimmicks to save the appearances within the appearances. You know how each one got there: an overnight hack to paste another tumor onto a wild cancerous growth. The concern was with method, regardless of results. The result is extravagantly worse than Roman numerals: you can't read the notation right to left or left to right. As an amalgam of languages, it can't be deciphered by a native speaker of any one of them, much as if we were to switch at random places in a number between Roman and Arabic signs and between big-endian and little-endian order. But now that it all “works” — at least for the strong of stomach — the tumors themselves are being standardized.

What bad notations are stifling your progress?
