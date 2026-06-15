---
title: A Month of Invalid GCC Bug Reports, and How to Eliminate Some of Them
url: https://blog.regehr.org/archives/1412
published: "2016-08-15T16:57:28Z"
feed: regehr
guid: http://blog.regehr.org/?p=1412
---

# A Month of Invalid GCC Bug Reports, and How to Eliminate Some of Them

During July 2016 the GCC developers marked 38 bug reports as INVALID. Here’s the [full list](https://gcc.gnu.org/bugzilla/buglist.cgi?bug_status=RESOLVED&bug_status=CLOSED&cf_known_to_fail_type=allwords&cf_known_to_work_type=allwords&f1=short_desc&f2=creation_ts&f3=creation_ts&n1=1&o1=equals&o2=greaterthaneq&o3=lessthan&query_format=advanced&resolution=INVALID&v1=spam&v2=2016-07-01&v3=2016-08-01). They fall into these (subjective) categories:

- 8 bug reports stemmed from undefined behavior in the test case ( [71753](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71753), [71780](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71780), [71803](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71803), [71813](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71813), [71885](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71885), [71955](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71955), [71957](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71957), [71746](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71746))

- 1 bug report was complaining about UB exploitation in general ( [71892](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71892))

- 15 bug reports came from a misunderstanding (or disagreement) about the non-UB semantics of a programming language, usually C++ but also C and Fortran ( [71786](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71786), [71788](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71788), [71794](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71794), [71804](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71804), [71809](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71809), [71890](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71890), [71914](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71914), [71939](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71939), [71963](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71963), [71967](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71967), [72580](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=72580), [72750](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=72750), [72761](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=72761), [71844](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71844), [71772](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71772))

- 4 bug reports stemmed from a misunderstanding of something besides the language semantics, such as command line flags ( [72736](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=72736), [71729](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71729), [71995](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71995), [71777](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71777))

- 5 bug reports were caused by an unrelated problem on the reporter’s system such as an out-of-memory error, a borked Cygwin installation, out-of-date files in a build tree, etc ( [71735](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71735), [71770](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71770), [71903](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71903), [71918](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71918), [71978](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71978))

- 1 bug report was about a bug that the devs didn’t want to fix since it was in an inactive branch and had been fixed in all active branches ( [72051](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=72051))

  4 bug reports didn’t end up demonstrating any reproducible problem ( [71940](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71940), [71944](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71944), [71986](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=71986), [72076](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=72076))

I’ve often thought that it would be nice for a compiler bug reporting system to be active instead of passively serving up files and discussions. An active bug reporting system would be able to run:

- a wide variety of compiler versions,

- the compiler’s output, and

- tools for finding undefined behaviors.

Of course not all bug reports would be able to make use of these features. However, one can imagine that there is a significant subset of compiler bug reports where the reporter, cooperating with the system, would be able to conclusively demonstrate that the compiler crashes or generates wrong code. In cases where this cannot be demonstrated, the process of interacting with an active bug reporting system will help the reporter understand what the actual issue is without wasting a compiler developer’s time.

An active bug reporting system can run lots of experiments to determine how many compiler versions, how many target platforms, and how many optimization levels are affected by the bug. It can also determine which revision introduced the problem and who committed the breaking change — suggesting an initial owner for the bug. It can run a testcase reducer to make the program triggering the bug smaller. All of these things will help compiler developers prioritize among reported bugs. The system should also be able to automatically flag duplicate bug reports. When a bug stops reproducing, the bug reporting system will notice this and flag the PR as being ready to close, and could also help out by packaging up an addition to the regression test suite.

**Update:**

A few additional details. During July a total of 328 bugs were reported, ignoring those marked as spam. 143 of these were resolved: 22 as duplicates, 81 as fixed, 38 as invalid, 1 as wontfix, and 1 as worksforme. Out of the remaining 185 unresolved bugs, 15 are assigned, 86 are new, 1 is reopened, 74 are unconfirmed, and 9 are waiting.

I believe that an active bug reporting system will make many of these 290 non-invalid bug reports easier to deal with, as opposed to only helping with the invalid ones!

Note: In the initial version of this post I mentioned 36 invalid bugs, not 38, because I was only searching for bugs that were marked as resolved. Also searching for closed bugs brings the total to 38.
