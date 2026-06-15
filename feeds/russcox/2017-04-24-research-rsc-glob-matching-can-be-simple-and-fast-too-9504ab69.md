---
title: 'research!rsc: Glob Matching Can Be Simple And Fast Too'
url: https://research.swtch.com/glob
published: "2017-04-24T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/glob
---

# research!rsc: Glob Matching Can Be Simple And Fast Too

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Glob Matching Can Be Simple And Fast Too Russ Cox April 24, 2017 *research.swtch.com/glob* Posted on Monday, April 24, 2017.

Here’s a straightforward benchmark. Time how long it takes to run `ls` ( `a*`)*n* `b` in a directory with a single file named `a` 100, compared to running `ls` `|` `grep` ( `a.*`)*n* `b`. Superscripts denote string repetition and parentheses are for grouping only, so that when *n* is 3, we’re running `ls` `a*a*a*b` in a directory containing the single file `aaa`... `aaa` (100 `a`’s), compared against `ls` `|` `grep` `a.*a.*a.*b` in the same directory.

01234567801234567pattern size ntime (minutes)ls010203040506070809010001234567pattern size ntime (ms)ls \| grep

For *n* = 8, `ls` takes 7.19 minutes while `ls` `|` `grep` runs in 1.56 milliseconds, making it 276,538X faster. If you’ve read my 2007 article “ [Regular Expression Matching Can Be Simple And Fast (but is slow in Java, Perl, PHP, Python, Ruby, ...)](https://swtch.com/~rsc/regexp/regexp1.html),” those graphs may look familiar. Clearly the `ls` command is using an exponential pattern-matching algorithm, while the `ls` `|` `grep` command is using a nearly linear one.

## [Shells](\#shells)

In fact it’s the shell that evaluates the [glob pattern](https://en.wikipedia.org/wiki/Glob_%28programming%29) in the first command, not `ls`, so let’s repeat the experiment with a variety of shells. All the tests were run on an HP Z440 workstation with 3.5 GHz Intel Xeon E5-1650 v3 processors running Ubuntu 14.04.

**Time to match** **( `a*`)*n* `b`** **against** **`a` 100 in shells**

0510152010000 s1000 s100 s10 s1 s100 ms10 ms1 ms100 uspattern size ntimetcsh 6.18.01ksh 93u+20120801Plan 9 rczsh 5.0.5rc 1.7.1bash 4.3dash 0.5.7csh 20110502

Basically, the same thing is going on here as in my regular expression article: backtracking was an obvious implementation strategy, so most implementations use it, causing the performance cliffs.

The exception seems to be the original Berkeley csh, which runs in linear time (more precisely, time linear in *n*). Looking at the source code, it doesn’t attempt to perform glob expansion itself. Instead it calls the C library implementation [*glob*(3)](https://linux.die.net/man/3/glob), which runs in linear time, at least on this Linux system. So maybe we should look at programming language implementations too.

## [Programming Languages](\#programming_languages)

Most programming languages provide some kind of glob expansion, like C’s `glob`. Let’s repeat the experiment in a variety of different programming languages:

**Time to match** **( `a*`)*n* `b`** **against** **`a` 100 in programming languages**

051015201000 s100 s10 s1 s100 ms10 ms1 ms100 us10 us1 uspattern size ntimeJava 8Perl 5.18.2BSD libcPython 3.4.3Tcl 8.6.1Rust 1.16.0Go 1.8Ruby 1.9.3glibc 2.19

Perhaps the most interesting fact evident in the graph is that GNU glibc, the C library used on Linux systems, has a linear-time `glob` implementation, but BSD libc, the C library used on BSD and macOS systems, has an exponential-time implementation.

PHP is not shown in the graph, because its [glob function](http://php.net/manual/en/function.glob.php) simply invokes the host C library’s *glob*(3), so that it runs in linear time on Linux and in exponential time on non-Linux systems. (I have not tested what happens on Windows.) All the languages shown in the graph, however, implement glob matching without using the host C library, so the results should not vary by host operating system.

## [FTP Servers](\#ftp_servers)

It’s not just shells and programming languages that implement glob matching. Although not mandated by the [FTP RFC](https://tools.ietf.org/html/rfc959), most FTP servers allow glob patterns as the argument to commands like LIST and STAT. Let’s repeat the experiment with a variety of FTP server implementations:

**Time to match** **( `a*`)*n* `b`** **against** **`a` 100 in FTP servers**

051015201000 s100 s10 s1 s100 ms10 ms1 ms100 us10 uspattern size ntimetnftpd (macOS 10.12.4)Pure-FTPd 1.0.36netkit ftpd 0.17Plan 9 ip/ftpdProFTPD 1.3.5vsftpd 3.0.2

On Linux, Pure-FTPd would probably have run for a hundred or more seconds for *n* = 7, but instead it died and hung up the connection after 17 seconds. All the tests were run on the same Linux system as before, except tnftpd, which was run on a mid-2015 MacBook Pro with a 2.8 GHz Intel Core i7 processor running macOS 10.12.4. On that Mac system, tnftpd (which can only be enabled using the command-line, so most people don’t run it) has no such time limit: even if a client times out and hangs up (as the command-line ftp client does after 30 seconds), the server side still consumes CPU until the full pattern match finishes.

The netkit ftpd runs quickly on Linux because it relies on the host C library’s `glob` function. If run on BSD, the netkit ftpd would take exponential time. ProFTPD ships a copy of the glibc `glob`, so it should run quickly even on BSD systems. Ironically, Pure-FTPd and tnftpd take exponential time on Linux because they ship a copy of the BSD `glob` function. Presumably they do this to avoid assuming that the host C library is bug-free, but, at least in this one case, the host C library is better than the one they ship.

Obviously not many servers have a file named `a` 100, but the pattern can be adapted to shorter, less unusual names. This gives a denial of service attack against some anonymous FTP servers. It appears that tnftpd is derived from the standard FreeBSD ftpd, so BSD systems may be affected as well.

FTP servers and C libraries have a [long history of problems with glob patterns](http://cve.mitre.org/cgi-bin/cvekey.cgi?keyword=glob), including buffer and heap overflows causing crashes or even remote code execution. But here let’s focus on CPU exhaustion issues around pattern matching.

In 2001, [CVE-2001-1501](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2001-1501) was issued for a [vulnerability in ProFTPD 1.2.1](http://marc.info/?l=bugtraq&m=98477291420305&w=2), because it could run for a very long time finding and recording the very many matches for a pattern like:

```
ls */../*/../*/../*/../*/../*/../*/../*/../*/../*

```

In response, most `glob` implementations [added a `GLOB_LIMIT` flag](https://github.com/openbsd/src/commit/98e1217625a0) that can be used to limit the number of matches returned, controlling both the CPU and the memory usage for such a pattern.

Unfortunately, a pattern like that can cause a lot of effort without finding any matches. In 2010, [CVE-2010-2632](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2010-2632) was issued for a [variant with no matches](http://cxsecurity.com/issue/WLB-2010100135):

```
ls */../*/../*/../*/../*/../*/../*/../*/../*blablahaha

```

In response, most `glob` implementations [expanded `GLOB_LIMIT`](https://github.com/openbsd/src/commit/46df4fe576b7) to count directory read and file stat operations in addition to matches.

In 2015, [CVE-2015-5917](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2015-5917) was issued for [the same vulnerability in OS X 10.10.5’s FTP server](https://cxsecurity.com/issue/WLB-2013040082) (reported in 2013), which had its own copy of the `glob` implementation and had not been patched in 2010. There have also been similar problems around FTP servers implementing brace expansion, such as [CVE-2011-0418](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2011-0418) ( [details](http://cxsecurity.com/issue/WLB-2011050004)).

Unfortunately, none of these protections address the cost of matching a single path element of a single file name. In 2005, [CVE-2005-0256](http://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2005-0256) was issued for a [DoS vulnerability in WU-FTPD 2.6.2](http://marc.info/?l=bugtraq&m=110935886414939&w=2), because it ran for a very long time finding even a single match during:

```
ftp> dir ************************************************
         ************************************************
         ************************************************
         **.*

```

The apparent [“fix” used in some implementations](http://marc.info/?l=bugtraq&m=110960890901497&w=2) is to collapse multiple adjacent stars into a single star. That solves one test case, but not the general problem. In particular it doesn’t help `a*a*a*a*a*a*a*a*b`.

The next section discusses ways to implement glob patterns efficiently, but if you have an anonymous FTP server accepting glob patterns, there are two more fundamental questions to ask: Do you really need to run an anonymous FTP server anymore? Does it really need to accept glob patterns?

At the very least, most FTP servers should probably reject glob patterns with more than, say, 3 stars. Note that glob patterns are only provided as a convenience for command-line FTP users. Graphical FTP clients typically use the [`MLST` and `MLSD` commands](https://tools.ietf.org/html/rfc3659#page-23), which have a machine-readable output format and are defined not to interpret their arguments as glob patterns.

## [Implementation Details](\#implementation_details)

How do these implementations work? What’s going on here?

The obvious implementation of glob pattern matching against a single path element is to walk the pattern and the name together, matching letters or wildcards in the pattern to letters in the name. If the walk reaches the end of the pattern at the same time as the end of the name, they match.

Here’s a basic outline of that algorithm, in Go:

```
func match(pattern, name string) bool {
	px := 0
	nx := 0
	for px < len(pattern) || nx < len(name) {
		if px < len(pattern) {
			c := pattern[px]
			switch c {
			default: // ordinary character
				if nx < len(name) && name[nx] == c {
					px++
					nx++
					continue
				}
			case '?': // single-character wildcard
				if nx < len(name) {
					px++
					nx++
					continue
				}
			case '*': // zero-or-more-character wildcard
				...
			}
		}
		// Mismatch.
		return false
	}
	// Matched all of pattern to all of name. Success.
	return true
}

```

In this code, `px` is the index into the pattern and `nx` the index into the name. The loop runs until both pattern and name are exhausted, meaning `px` `==` `len(pattern)` and `nx` `==` `len(name)`. If it does happen that both pattern and name are exhausted at the same time, then we have a match (the final `return` `true`). Inside the loop, the code must make progress (advance `px` or `nx` or both) and `continue` on each iteration. If not, the control flow ends up at the bottom of the loop body (marked `//` `Mismatch.`) and reports no match.

The only difficulty is in the implementation of the variable-length wildcard pattern ( `case` `'*'`): how much of the name should that match? In general, the code must try all possibilities from matching nothing to matching the entire remainder of the string. The obvious way to do that is with recursion:

```
func match(pattern, name string) bool {
	px := 0
	nx := 0
	for px < len(pattern) || nx < len(name) {
		if px < len(pattern) {
			c := pattern[px]
			switch c {
			default: // ordinary character
				if nx < len(name) && name[nx] == c {
					px++
					nx++
					continue
				}
			case '?': // single-character wildcard
				if nx < len(name) {
					px++
					nx++
					continue
				}
			case '*': // zero-or-more-character wildcard
				// Try to match at nx, nx+1, and so on.
				for ; nx <= len(name); nx++ {
					if match(pattern[px+1:], name[nx:]) {
						return true
					}
				}
			}
		}
		// Mismatch.
		return false
	}
	// Matched all of pattern to all of name. Success.
	return true
}

```

If there are *e* stars that can each potentially end their matches at *n* positions in the string, that gives *n* *e* possibilities to explore, leading to the exponential run times observed earlier. However, most of these possibilities are not worth exploring. Because `*` is the only variable-sized wildcard in the pattern syntax and therefore the only source of possible backtracking, there’s an even easier implementation: don’t backtrack to an earlier star.

Consider the pattern `a*bx*cy*d`. If we end the first star at the first `bx`, we have the rest of the name to find the `cy` and then the `d`. Using any later `bx` can only remove choices for `cy` and `d`; it cannot lead to a successful match that using the first `bx` missed. So we should implement `a*bx*cy*d` without any second-guessing, as “find a leading `a`, then find the earliest `bx` after that, then find the earliest `cy` after that, then find a trailing `d`, or else give up.”

We can implement this algorithm by replacing the handling of the star wildcard in our Go program:

```
func match(pattern, name string) bool {
	px := 0
	nx := 0
	nextPx := 0
	nextNx := 0
	for px < len(pattern) || nx < len(name) {
		if px < len(pattern) {
			c := pattern[px]
			switch c {
			default: // ordinary character
				if nx < len(name) && name[nx] == c {
					px++
					nx++
					continue
				}
			case '?': // single-character wildcard
				if nx < len(name) {
					px++
					nx++
					continue
				}
			case '*': // zero-or-more-character wildcard
				// Try to match at nx.
				// If that doesn't work out,
				// restart at nx+1 next.
				nextPx = px
				nextNx = nx + 1
				px++
				continue
			}
		}
		// Mismatch. Maybe restart.
		if 0 < nextNx && nextNx <= len(name) {
			px = nextPx
			nx = nextNx
			continue
		}
		return false
	}
	// Matched all of pattern to all of name. Success.
	return true
}

```

Each time the code encounters a star, it implements the repeated trials needed for that star by recording in `nextPx`, `nextNx` where to restart the search if the current match fails. Each subsequent star encountered overwrites the restart information for the previous star, in effect locking in the choice made for the previous star. If you’d like to experiment, I’ve posted [both of these programs and a test harness](glob.go).

An alternate implementation of the algorithm would be to split the pattern on stars and then consider each of the star-separated subpatterns in turn, special-casing the first subpattern and the last, which must be anchored at the start and end of the name, respectively. Go’s [src/path/filepath/match.go](https://golang.org/src/path/filepath/match.go) is an example of this implementation.

Go’s [`filepath.Match`](https://golang.org/pkg/path/filepath/#Match) (used by [`filepath.Glob`](https://golang.org/pkg/path/filepath/#Glob)), glibc’s `glob`, and vsftpd’s pattern matcher all use this algorithm.

A more straightforward approach is to translate the glob pattern to a regular expression and then invoke a linear-time regular expression match. Plan 9’s ftpd does this, as does the `ls` `|` `grep` example above. ( [Python also does this](https://github.com/python/cpython/blob/master/Lib/fnmatch.py#L39), but then it runs the regular expression with an exponential-time matching engine.)

I have not looked at the other linear-time implementations to see what they do, but I expect they all use one of these two approaches.

## [Additional Reading](\#additional_reading)

This post is an elaboration of an informal [2012 Google+ post](https://plus.google.com/u/2/+RussCox-rsc/posts/8aANoDNvhie) showing that most shells used exponential-time glob expansion. At the time, Tom Duff, the author of Plan 9’s rc shell, commented that, “I can confirm that rc gets it wrong. My excuse, feeble as it is, is that doing it that way meant that the code took 10 minutes to write, but it took 20 years for someone to notice the problem. (That’s 10 ‘programmer minutes’, i.e. less than a day.)” I agree that’s a reasonable decision for a shell. In contrast, a language library routine, not to mention a network server, today needs to be robust against worst-case inputs that might be controlled by remote attackers, but nearly all of the code in question predates that kind of concern. I didn’t realize the connection to FTP servers until I started doing additional research for this post and came across a reference to CVE-2010-2632 in [FreeBSD’s `glob` implementation](https://github.com/freebsd/freebsd/blob/5b0d2af29a95/lib/libc/gen/glob.c#L100).

Dave Presotto, the author of Plan 9’s ftpd, avoided rc’s glob implementation not because of the algorithmic complexity but to make sure that long strings were handled safely (using a Plan 9 library for handling long strings in C). He wrote this comment at the top of Plan 9’s [/sys/src/cmd/ip/glob.c](https://9p.io/sources/plan9/sys/src/cmd/ip/glob.c), which converts glob expressions into regular expressions:

```
/*
 *  I wrote this glob so that there would be no limit
 *  on element or path size.  The one in rc is probably
 *  better, certainly faster. - presotto
 */

```

As it turns out, this is untrue: the one in rc is certainly slower, at least in terms of worst-case asymptotics.

If you liked this post, you may also like to browse Nelson Elhage’s [Accidentally Quadratic](https://accidentallyquadratic.tumblr.com/post/113840433022/why-accidentally-quadratic) collection. (Of course, here the glob pattern matching is accidentally exponential.)

## [Updates, as of April 28, 2017.](\#updates)

NetBSD has an [updated glob](http://cvsweb.netbsd.org/bsdweb.cgi/src/lib/libc/gen/glob.c.diff?r1=1.36&r2=1.37&f=h) pending for the next release.

Perl has an [updated glob](https://perl5.git.perl.org/perl.git/commitdiff/33252c318625f3c6c89b816ee88481940e3e6f95?hp=57ab6c610267dba697199c8256f4258af7d391c1) pending for the next release.

Pure-FTPd 1.0.46 has added a [check that patterns must not have more than three stars](https://github.com/jedisct1/pure-ftpd/commit/63d98420e2c205c40e5a0849bde142e0aab17955).

Thanks to [js2 on Hacker News](https://news.ycombinator.com/item?id=14185822) for pointing out that Python translates globs to regular expressions.
