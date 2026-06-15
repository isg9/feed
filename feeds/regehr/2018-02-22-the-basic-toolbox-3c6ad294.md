---
title: The Basic Toolbox
url: https://blog.regehr.org/archives/1578
published: "2018-02-22T18:56:55Z"
feed: regehr
guid: http://blog.regehr.org/?p=1578
---

# The Basic Toolbox

This post is aimed at computer science students.

In the software engineering course I’m teaching this spring, I often find myself saying things like “you need to know a scripting language” or “everyone should be able to run a code coverage tool.” Finally, the other day, a student stopped me and asked for the whole list. In other words, what — in my opinion — is the collection of tools that someone graduating with a CS degree should know how to use. Of course I couldn’t answer this on the spot but I’ve been thinking about it since then. The basic idea is that for most any common situation, you should have a decent tool at hand and be able to start solving problems with it without too much fumbling around. (Keep in mind that this is a wish list for self-study: I doubt that any CS program teaches all of these. Also, I didn’t have all of these tool skills when I got my undergraduate CS degree, though I did by the time I got a PhD.)

**A version control system:** Git is the obvious choice; the main thing you should have is a basic Github-centric workflow including pull requests, remotes, dealing with merge conflicts, etc.

**A text editor:** We all end up using different editors from time to time, but we should each have a solid default choice that does a good job with most editing tasks. It should highlight and indent any common programming language, integrate with a spellchecker, easily load gigantic files, have nice regex-based search and replace, etc. There are plenty of choices, many CS people migrate to vim or emacs.

**A graphing program:** I routinely use gnuplot, graphviz, and Powerpoint to make figures. Lots of people like matplotlib.

**A presentation tool:** Powerpoint, Keynote, Google Slides, something LaTeX based, etc.

**An interactive debugger for native executables:** LLDB, GDB, something IDE-based.

**A generic build system:** Make, CMake, etc.

**A scripting language:** This is for low-grade automation, quick and dirty data analysis tasks, etc. Python and JavaScript would seem like natural choices. Around 20 years ago I was an intern at a networking company and my supervisor popped out of a meeting with some data concerning switch errors, and asked me to do some analysis to locate the underlying pattern. I wasn’t sure how; he handed me a Perl book and I was able to get the job done before the meeting ended.

**A shell language:** This is probably bash or PowerShell, but there are plenty of other choices. There’s some overlap with scripting languages but I think there are two distinct niches here: a shell language is for scripting a smallish number of commands, doing a bit of error checking, and perhaps looping or interacting with the user slightly This sort of job is a bit too cumbersome in Python, Perl, or JavaScript.

**A systems language:** This is for creating servers, daemons, and other code that wants to go fast, use little memory, have few dependencies, and interact tightly with the OS. C or C++ would be the obvious choices, but Rust and Go may be fine too.

**A workhorse language:** This is your default programming language for most tasks, it should have a huge collection of high-quality libraries, be pretty fast, run on all common platforms, have a great tool ecosystem, etc. Racket, Java, Scala, OCaml, C#, Swift, or Haskell would be great — even C++ would work.

**A pocket calculator:** This is your go-to REPL for basic arithmetic and conversions between number representations, it should be near-instantaneous to get answers. For reasons I no longer remember, I use gdb for this — typically multiple times in any work day. Old standbys like bc and dc also seem like bad choices. I’m curious what other people do here.

## Tools for Programming Languages

There’s no reason these days to use a language that doesn’t have a good tool ecosystem. For any given language you should know how to use its interactive debugger, static and dynamic bug-finding tools, a profiler, a code coverage tool, a build system, a package manager, and perhaps a random test-case generator.

## Secondary Tools

There are a lot of other tools that could have gone into my basic toolbox, such as a data analysis tool, a browser language, a cloud-based testing service, a statistics language, a typesetting system, a spreadsheet, a database, and a GUI builder/toolkit. I don’t consider these as fundamental; of course, your mileage may vary.
