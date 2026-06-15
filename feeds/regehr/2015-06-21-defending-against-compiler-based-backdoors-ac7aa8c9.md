---
title: Defending Against Compiler-Based Backdoors
url: https://blog.regehr.org/archives/1241
published: "2015-06-21T23:09:39Z"
feed: regehr
guid: http://blog.regehr.org/?p=1241
---

# Defending Against Compiler-Based Backdoors

Scotty Bauer (a Utah grad student), Pascal Cuoq, and I have an article in [the latest PoC\|\|GTFO](https://www.alchemistowl.org/pocorgtfo/pocorgtfo08.pdf) about introducing a backdoor into sudo using a compiler bug. In other words, the C code implementing sudo does not contain a backdoor, but a backdoor appears when sudo is built using a particular compiler version (clang 3.3, here). The advantages of this kind of backdoor include subtlety, deniability, and target-specificity.

Of course I have no idea who, if anyone, is trying to use compiler bugs to create backdoors. However if I worked for the offensive wing of a well-funded security service, this would be one of the tools in my toolbox. But that’s neither here nor there. What follows are some ideas about how various groups of people can help defend against these backdoors.

### Compiler Developers

- Fix known miscompilation bugs as rapidly as possible

- Consider back-porting fixes to miscompilation bugs into maintenance releases

- Go looking for trouble using fuzz tools

### Maintainers of Open Source Packages

- Be suspicious of baroque patch submissions

- Consider rewriting patches

### OS Packagers

- Assess compilers for reliability before selecting a system compiler

- Aggressively test compiled code for all platforms before deployment

- Run a [trusting trust test](http://www.dwheeler.com/trusting-trust/) on the system compiler (our attack in the PoC\|\|GTFO article isn’t a trusting trust attack– but this won’t hurt)

### End User

- Recompile the system from source, using a different compiler or compiler version

- Run your own acceptance tests on precompiled applications

### Researchers

- Create practical proved-correct compilers — which won’t contain the kind of bug that we exploited

- Create practical translation validation schemes — which would detect the miscompilation as it occurs

- Create practical N-version programming systems — which would detect the backdoor as it executes

Overall, this kind of attack is not easy to defend against, and my guess is that most instances of it (if any exist) will never be detected.
