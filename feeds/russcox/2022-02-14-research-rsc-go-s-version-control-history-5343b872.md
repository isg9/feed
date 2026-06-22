---
title: "research!rsc: Go’s Version Control History"
url: https://research.swtch.com/govcs
published: "2022-02-14T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/govcs
---

# research!rsc: Go’s Version Control History

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Go’s Version Control History Russ Cox February 14, 2022 _research.swtch.com/govcs_ Posted on Monday, February 14, 2022.

Every once in a while someone notices the first commit in the Go
repo is dated 1972:

```
% git log --reverse --stat
commit 7d7c6a97f815e9279d08cfaea7d5efb5e90695a8
Author:     Brian Kernighan
AuthorDate: Tue Jul 18 19:05:45 1972 -0500
Commit:     Brian Kernighan
CommitDate: Tue Jul 18 19:05:45 1972 -0500

    hello, world

    R=ken
    DELTA=7  (7 added, 0 deleted, 0 changed)

 src/pkg/debug/macho/testdata/hello.b | 7 +++++++
 1 file changed, 7 insertions(+)

...

```

Obviously something silly is going on, and people usually stop
there. But Go’s actual version control history is richer and more
interesting. For example, there are a few more fake commits and
then the fifth commit is the first real one:

```
commit 18c5b488a3b2e218c0e0cf2a7d4820d9da93a554
Author:     Robert Griesemer
AuthorDate: Sun Mar 2 20:47:34 2008 -0800
Commit:     Robert Griesemer
CommitDate: Sun Mar 2 20:47:34 2008 -0800

    Go spec starting point.

    SVN=111041

 doc/go_spec | 1197 +++++++++++++++++++++++++++++++++++++++++++++++++++++++++++
 1 file changed, 1197 insertions(+)

```

Why does that commit have a different trailer than the three fake
commits?

## [Subversion](#subversion)

Go started out using Subversion, as part of a small experiment to
evaluate Subversion for wider use inside Google. The experiment
did not result in wider Subversion use, but the `SVN=111041` tag
in the
[first real commit above](https://go.googlesource.com/go/+/18c5b488a3b2e218c0e0cf2a7d4820d9da93a554)
records that on the original Subversion server, that Go commit was
revision 111,041. (Subversion assigns revision numbers in
increasing order, and the server was a small monorepo being used
for a few other projects besides Go. There were not 111,040 other
Go commits that didn’t make it out.)

## [Perforce](#perforce)

The SVN tags continue in the logs
[until July 2008](https://go.googlesource.com/go/+/05caa7f82030327ccc9ae63a2b0121a029286501),
where we see one last SVN commit and then a new form:

```
commit 777ee7163bba96f2c9b3dfe135d8ad4ab837c062
Author:     Rob Pike
AuthorDate: Mon Jul 21 16:18:04 2008 -0700
Commit:     Rob Pike
CommitDate: Mon Jul 21 16:18:04 2008 -0700

    map delete

    SVN=128258

 doc/go_lang.txt | 6 ++++++
 1 file changed, 6 insertions(+)

commit 05caa7f82030327ccc9ae63a2b0121a029286501
Author:     Rob Pike
AuthorDate: Mon Jul 21 17:10:49 2008 -0700
Commit:     Rob Pike
CommitDate: Mon Jul 21 17:10:49 2008 -0700

    help management of empty pkg and lib directories in perforce

    R=gri
    DELTA=4  (4 added, 0 deleted, 0 changed)
    OCL=13328
    CL=13328

 lib/place-holder      | 2 ++
 pkg/place-holder      | 2 ++
 src/cmd/gc/mksys.bash | 0
 3 files changed, 4 insertions(+)

```

This was the first commit after Go moved from a lightly-used
Subversion server to a lightly-used Perforce server. Google was a
[very heavy Perforce user](https://static.googleusercontent.com/media/research.google.com/en//pubs/archive/39983.pdf)
due to its use of a
[single giant monorepo for most code](https://dl.acm.org/doi/pdf/10.1145/2854146).
Go was not part of that monorepo world, and at the time, projects
that didn’t have to be on the heavily-loaded main server were
hosted instead on secondary ones.

At the transition to Perforce, you can see the introduction of
telltale `DELTA=`, `OCL=`, and `CL=` tags. Perforce imposes a
linear ordering on change lists, like Subversion revisions, but
each change list ends up with two sequence numbers: it is assigned
one when it is created and uses that number while it is a local,
pending change list, including in our code review systems. Then,
when it is submitted and becomes part of the official history, it
is assigned a new number, to keep submitted changes in order. The
`OCL=` is the original change list number, while the `CL=` is the
final one. The `R=` line means that `gri` (Robert Griesemer)
reviewed the change before it was submitted, using our internal
code review system, then called Mondrian. Because the server was
so lightly used (and presumably the review so quick), no new
change lists had been created or submitted while this one was
pending, and its final submission reused its original number
instead of needing to create a new one.

Many other changes have the same `OCL=` and `CL=` because they
were created and submitted in a single Perforce command, without
review, like
[the next one](https://go.googlesource.com/go/+/c1f5eda7a2465dae196d1fa10baf6bfa9253808a):

```
commit c1f5eda7a2465dae196d1fa10baf6bfa9253808a
Author:     Rob Pike
AuthorDate: Mon Jul 21 18:06:39 2008 -0700
Commit:     Rob Pike
CommitDate: Mon Jul 21 18:06:39 2008 -0700

    change date

    OCL=13331
    CL=13331

 doc/go_lang.txt | 2 +-
 1 file changed, 1 insertion(+), 1 deletion(-)

```

You can also see from this that the `DELTA=` line is added by the
code review system, not Perforce: this unreviewed change doesn’t
have it. (The last two lines are being provided by the
`git log --stat` command as we explore the Git version of these
changes.)

The bulk of the pre-open-source development of Go was done on that
Perforce server.

## [Mercurial](#mercurial)

The `OCL=` and `CL=` lines continue until we get to
[October 2009](https://go.googlesource.com/go/+/b74fd8ecb17c1959bbf2dbba6ccb8bae6bfabeb8),
when they switch to a new form:

```
commit 942d6590d9005f89e971ed5af0374439a264a20e
Author:     Kai Backman
AuthorDate: Fri Oct 23 11:03:16 2009 -0700
Commit:     Kai Backman
CommitDate: Fri Oct 23 11:03:16 2009 -0700

    one more argsize fix. we were copying with the correct
    alignment but not enough (duh).

    R=rsc
    APPROVED=rsc
    DELTA=16  (13 added, 0 deleted, 3 changed)
    OCL=36020
    CL=36024

 src/cmd/5g/ggen.c |  2 +-
 test/arm-pass.txt | 17 +++++++++++++++--
 2 files changed, 16 insertions(+), 3 deletions(-)

commit b74fd8ecb17c1959bbf2dbba6ccb8bae6bfabeb8
Author:     Kai Backman
AuthorDate: Fri Oct 23 12:43:01 2009 -0700
Commit:     Kai Backman
CommitDate: Fri Oct 23 12:43:01 2009 -0700

    fix build issue cause by transition to hg

    R=rsc
    http://go/go-review/1013012

 src/make-arm.bash | 4 ++--
 1 file changed, 2 insertions(+), 2 deletions(-)

```

That commit introduces yet another kind of trailer line, which
says `http://go/go-review/1013012`. At one point that link
pointed, inside Google, to an internal version of the Rietveld App
Engine app, which we used for code review on the way to submission
to a private Mercurial repo on Google Code Project Hosting. We
were doing that conversion, in October 2009, as part of the
preparation to open-source Go in November.

It was at this point, during the conversion to Mercurial, that I
introduced the “hello, world” commits. The Subversion and Perforce
repos had been Google-internal servers, and we had stored
essentially all the Go code in the world alongside the Go
implementation. We each had “user directories” like `/usr/rsc` in
the repo. Those directories contained various code that wasn’t
going out in the release, whether because it was targeting
Google-internal technologies or because it was simply not worth
publishing. (You can still see references to `/usr` in a few
commit messages that did make it out.)

For the conversion, I ended up with a directory full of patches,
one per commit, and a script to run the patches to create a new
Mercurial repository. Then I edited the patches (with automated
help) to remove the files that weren’t being released, including
the entire `/usr` tree, and to add the new open-source copyright
notices to every file.

This took me about a week, and it was annoyingly difficult. If I
removed a file from some patches but then it got renamed in
others, Mercurial would complain about the rename using a file
that hadn’t been created in the first place. And if I added a
copyright notice when the file was created, I had to be careful to
update later patches to not have merge conflicts when changing the
top of the file. And so on. I remember thinking that it was like
being a con man who constructs an elaborate false identity and
struggles to keep it all straight.

## [Hello, world](#hello_world)

A month earlier, I had created object file parsing packages like
[debug/macho](https://go.dev/pkg/debug/macho), and as test data
for each of those packages I
[checked in an object file](https://go.googlesource.com/go/+/bf69025825fd2b8e7aac01f27d5c974bd30af542)
compiled from a trivial “hello, world” program, along with the
source code for them:

```
commit bf69025825fd2b8e7aac01f27d5c974bd30af542
Author:     Russ Cox
AuthorDate: Fri Sep 18 11:49:22 2009 -0700
Commit:     Russ Cox
CommitDate: Fri Sep 18 11:49:22 2009 -0700

    Mach-O file reading

    R=r
    DELTA=784  (784 added, 0 deleted, 0 changed)
    OCL=34715
    CL=34788

 src/pkg/debug/macho/Makefile                       |  12 +
 src/pkg/debug/macho/file.go                        | 374 +++++++++++++++++++++
 src/pkg/debug/macho/file_test.go                   | 159 +++++++++
 src/pkg/debug/macho/macho.go                       | 230 +++++++++++++
 src/pkg/debug/macho/testdata/gcc-386-darwin-exec   | Bin 0 -> 12588 bytes
 src/pkg/debug/macho/testdata/gcc-amd64-darwin-exec | Bin 0 -> 8512 bytes
 .../macho/testdata/gcc-amd64-darwin-exec-debug     | Bin 0 -> 4540 bytes
 7 files changed, 775 insertions(+)

```

This was the original commit that introduced
`src/pkg/debug/macho/testdata/hello.c`, of course. As I added
copyright notices to files, it seemed wrong to add a copyright
notice to that `hello.c` file. Instead, since I had the repo split
into this patch-file-per-commit form, it was easy to create a few
fake commits that showed at least part of the real history of that
program, as an Easter egg for people who looked that closely:

```
commit 7d7c6a97f815e9279d08cfaea7d5efb5e90695a8
Author:     Brian Kernighan
AuthorDate: Tue Jul 18 19:05:45 1972 -0500
Commit:     Brian Kernighan
CommitDate: Tue Jul 18 19:05:45 1972 -0500

    hello, world

    R=ken
    DELTA=7  (7 added, 0 deleted, 0 changed)

 src/pkg/debug/macho/testdata/hello.b | 7 +++++++
 1 file changed, 7 insertions(+)

commit 0bb0b61d6a85b2a1a33dcbc418089656f2754d32
Author:     Brian Kernighan
AuthorDate: Sun Jan 20 01:02:03 1974 -0400
Commit:     Brian Kernighan
CommitDate: Sun Jan 20 01:02:03 1974 -0400

    convert to C

    R=dmr
    DELTA=6  (0 added, 3 deleted, 3 changed)

 src/pkg/debug/macho/testdata/hello.b | 7 -------
 src/pkg/debug/macho/testdata/hello.c | 3 +++
 2 files changed, 3 insertions(+), 7 deletions(-)

commit 0744ac969119db8a0ad3253951d375eb77cfce9e
Author:     Brian Kernighan
AuthorDate: Fri Apr 1 02:02:04 1988 -0500
Commit:     Brian Kernighan
CommitDate: Fri Apr 1 02:02:04 1988 -0500

    convert to Draft-Proposed ANSI C

    R=dmr
    DELTA=5  (2 added, 0 deleted, 3 changed)

 src/pkg/debug/macho/testdata/hello.c | 7 +++++--
 1 file changed, 5 insertions(+), 2 deletions(-)

commit d82b11e4a46307f1f1415024f33263e819c222b8
Author:     Brian Kernighan
AuthorDate: Fri Apr 1 02:03:04 1988 -0500
Commit:     Brian Kernighan
CommitDate: Fri Apr 1 02:03:04 1988 -0500

    last-minute fix: convert to ANSI C

    R=dmr
    DELTA=3  (2 added, 0 deleted, 1 changed)

 src/pkg/debug/macho/testdata/hello.c | 4 +++-
 1 file changed, 3 insertions(+), 1 deletion(-)

```

The
[July 1972 commit](https://go.googlesource.com/go/+/7d7c6a97f815e9279d08cfaea7d5efb5e90695a8)
shows the very first “hello, world” program, copied from Brian
Kernighan’s “
[A Tutorial Introduction to the Language B](https://www.bell-labs.com/usr/dmr/www/btut.html)”:

```
main( ) {
 extrn a, b, c;
 putchar(a); putchar(b); putchar(c); putchar(’!*n’);
}

a ’hell’;
b ’o, w’;
c ’orld’;

```

Various online sources refer to this as a “book”, but it
definitely was not. It was a printed, 17-page document that was
later included as half of January 1973’s “
[Bell Laboratories Computing Science Technical Report #8: The Programming Language B](https://www.bell-labs.com/usr/dmr/www/bintro.html)”
(still not even close to a book!). The “hello, world” program
appears in section 7, preceded by the less catchy “hi!” program in
section 6. As I write this blog post in 2022, I cannot find any
online reference for the B tutorial being originally dated July
1972, but I believe I must have had a good reason.

The
[January 1974 commit](https://go.googlesource.com/go/+/0bb0b61d6a85b2a1a33dcbc418089656f2754d32)
converts the program to C, as found in Kernighan’s “
[Programming in C — A Tutorial](https://www.bell-labs.com/usr/dmr/www/ctut.pdf)”.
The linked PDF is a retyped copy from Dennis Ritchie, without a
date, but Ritchie’s “
[C Reference Manual](https://www.bell-labs.com/usr/dmr/www/cman74.pdf)”
technical memo dated January 15, 1974 cites Kernighan’s tutorial
as “Unpublished internal memorandum, Bell Laboratories, 1974”,
implying that the tutorial must have been written in January as
well. That C program was shorter than what we are used to today,
but much nicer than the B program:

```
main( ) {
    printf("hello, world");
}

```

I skipped over the presentation in the first edition of _The C
Programming Language_, which looks like:

```
main() {
    printf("hello, world\n");
}

```

The
[April 1988 commit](https://go.googlesource.com/go/+/0744ac969119db8a0ad3253951d375eb77cfce9e)
shows the program from the “Draft-Proposed ANSI C” version of the
second edition of _The C Programming Language_:

```
#include

main()
{
    printf("hello, world\n");
}

```

The
[second April 1988 commit](https://go.googlesource.com/go/+/d82b11e4a46307f1f1415024f33263e819c222b8)
shows the final full ANSI C version we know today:

```
#include

int
main(void)
{
    printf("hello, world\n");
    return 0;
}

```

Wikipedia says the second edition was published in April 1988. I
didn’t have a day, but April 1 seemed appropriate. I didn’t have a
time for either one of these, so I used 02:03:04 for the second
commit and therefore 02:02:04 for the first.

The email addresses in the commits are also period-appropriate,
although Mondrian’s `R=` and `DELTA=` tags clearly are not: there
was no code review happening back then!

For what it’s worth, I’ve since heard from many people about how
the 1972 dates have broken various presentations and analyses
they’ve done on the Go repository as compared to other projects.
Oops! My apologies. (I also heard from someone who was analyzing
Mercurial repos for “branchiness” and decided to use the Go repo
as a large test case. Their program said it had branchiness 0 and
they spent a while debugging their program before realizing that
the repo really was completely linear, thanks to our “rebase and
merge” policy.)

## [Public Mercurial](#public_mercurial)

We published the Google Code Mercurial repo on November 10, 2009,
and to mark the occasion we added one more Easter egg. A footnote
in Brian Kernighan and Rob Pike’s 1984 book _The Unix Programming
Environment_ says:

> Ken Thompson was once asked what he would do differently if he
> were redesigning the UNIX system. His reply: “I’d spell `creat`
> with an `e`.”

This refers to the
[creat(2) system call](https://man7.org/linux/man-pages/man2/open.2.html)
and the `O_CREAT` file open flag.

Immediately once the release was public,
[Ken mailed me a change](https://go.googlesource.com/go/+/c90d392ce3d3203e0c32b3f98d1e68c4c2b4c49b)
fixing this mistake:

```
commit c90d392ce3d3203e0c32b3f98d1e68c4c2b4c49b
Author:     Ken Thompson
AuthorDate: Tue Nov 10 15:05:15 2009 -0800
Commit:     Ken Thompson
CommitDate: Tue Nov 10 15:05:15 2009 -0800

    spell it with an "e"

    R=rsc
    http://go/go-review/1025037

 src/pkg/os/file.go | 1 +
 1 file changed, 1 insertion(+)

```

Sadly, we didn’t execute the joke perfectly, in that the code
review went to our internal Rietveld server instead of the public
one. So the
[very next commit](https://go.googlesource.com/go/+/44fb865a484b8e12adfa0a1413eacc807cec085b)
updated the code review server in our configuration.

But here, for posterity, is the review, preserved in my email:

![](creat.png)

## [Git](#git)

So things stood from November 2009 until late 2014, when we knew
that Google Code Project Hosting was going to shut down and we
needed a new home. After investigating a few options, we ended up
using [Gerrit Code Review](https://www.gerritcodereview.com/),
which has been a fantastic choice. Many people think of Go as
being hosted on GitHub, but GitHub is only primary for our issue
tracker: the official primary copy of the sources is on
`go.googlesource.com`.

You can see the transition from Mercurial to Git in the logs when
the `R=` lines end, followed by a few completely unreviewed
commits, and then `Reviewed-by:` lines begin:

```
commit 94151eb2799809ece7e44ce3212aa3cbb9520849
Author:     Russ Cox
AuthorDate: Fri Dec 5 21:33:07 2014 -0500
Commit:     Russ Cox
CommitDate: Fri Dec 5 21:33:07 2014 -0500

    encoding/xml: remove SyntaxError.Byte

    It is unused. It was introduced in the CL that added InputOffset.
    I suspect it was an editing mistake.

    LGTM=bradfitz
    R=bradfitz
    CC=golang-codereviews
    https://golang.org/cl/182580043

 src/encoding/xml/xml.go | 1 -
 1 file changed, 1 deletion(-)

commit 258f53dee33b9055ea168cb186f8c076edee5905
Author:     David Symonds
AuthorDate: Mon Dec 8 13:50:49 2014 +1100
Commit:     David Symonds
CommitDate: Mon Dec 8 13:50:49 2014 +1100

    remove .hgtags.

 .hgtags | 140 ----------------------------------------------------------------
 1 file changed, 140 deletions(-)

commit 369873c6e5d00314ae30276363f58e5af11b149c
Author:     David Symonds
AuthorDate: Mon Dec 8 13:50:49 2014 +1100
Commit:     David Symonds
CommitDate: Mon Dec 8 13:50:49 2014 +1100

    convert .hgignore to .gitignore.

 .hgignore => .gitignore | 9 +--------
 1 file changed, 1 insertion(+), 8 deletions(-)

commit f33fc0eb95be84f0a688a62e25361a117e5b995b
Author:     David Symonds
AuthorDate: Mon Dec 8 13:53:11 2014 +1100
Commit:     David Symonds
CommitDate: Mon Dec 8 13:53:11 2014 +1100

    cmd/dist: convert dist from Hg to Git.

 src/cmd/dist/build.c | 100 ++++++++++++++++++++++++++++++---------------------
 1 file changed, 59 insertions(+), 41 deletions(-)

commit 26399948e3402d3512cb14fe5901afaef54482fa
Author:     David Symonds
AuthorDate: Mon Dec 8 11:39:11 2014 +1100
Commit:     David Symonds
CommitDate: Mon Dec 8 04:42:22 2014 +0000

    add bin/ to .gitignore.

    Change-Id: I5c788d324e56ca88366fb54b67240cebf5dced2c
    Reviewed-on: https://go-review.googlesource.com/1171
    Reviewed-by: Andrew Gerrand

 .gitignore | 1 +
 1 file changed, 1 insertion(+)

```

As part of the conversion from Mercurial to Git, we did not add
visible lines to the commit bodies showing the old Mercurial
hashes, but we did record them in the underlying Git commit
objects. For example:

```
% git cat-file -p 7d7c6a97f815e9279d08cfaea7d5efb5e90695a8
tree e06bd601885e16ad3d72c2a8c9b411889b2e478e
author Brian Kernighan  80352345 -0500
committer Brian Kernighan  80352345 -0500
golang-hg f6182e5abf5eb0c762dddbb18f8854b7e350eaeb

hello, world

R=ken
DELTA=7  (7 added, 0 deleted, 0 changed)
%

```

The `golang-hg` line records the original Mercurial commit hash.

And that’s the end of the story, until we move to a fifth version
control system at some point in the future.
