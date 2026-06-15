---
title: Classic Bug Reports
url: https://blog.regehr.org/archives/1270
published: "2015-09-30T20:48:45Z"
feed: regehr
guid: http://blog.regehr.org/?p=1270
---

# Classic Bug Reports

A bug report is sometimes entertaining either because of the personalities involved or because of the bug itself. Here are a collection of links into public bug trackers; I learned about most of these in a [recent Twitter thread](https://twitter.com/johnregehr/status/648912004657319936).

- GCC’s magnum opus, its War and Peace, is [Bug 323: optimized code gives strange floating point result.](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=323) With a 15 year history and 90 duplicates, this bug shows how badly wrong things can go when the compiler gets caught between a dubious standard and the weird x87.
- Another classic from GCC is [Bug 30475: assert(int+100 > int) optimized away](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=30475)
- [Bug 56888: memcpy implementation optimized as a call to memcpy](https://gcc.gnu.org/bugzilla/show_bug.cgi?id=56888) is about the compiler doing what modern compilers do best: straddling the line between clever and idiotic

- In Ubuntu [Bug 255161](https://bugs.launchpad.net/ubuntu/+source/cupsys/+bug/255161) our intrepid heroes solve the mysterious inability to print on a Tuesday
- Ulrich “this is no place to get a free education” Drepper was the source of much consternation during the time that he maintained glibc. [Bug 12701: scanf accepts non-matching input](https://sourceware.org/bugzilla/show_bug.cgi?id=12701) shows him in fine form.
- RedHat [Bug 1202858: restarting testing build of squid results in deleting all files in hard-drive](https://bugzilla.redhat.com/show_bug.cgi?id=1202858)
- Similarly, [steam-for-linux issue 3671](https://github.com/valvesoftware/steam-for-linux/issues/3671) illustrates the importance of sanitizing arguments passed to `rm -rf`.
- Jesse Ruderman provides [this glimpse into the lighter side of Mozilla’s bug tracker](https://bugzilla.mozilla.org/buglist.cgi?quicksearch=ALL%20cc%3Afoxymonkies&order=bugs.bug_id%20desc) including:

  - [Bug 362178: ‘amature’ not in dictionary](https://bugzilla.mozilla.org/show_bug.cgi?id=362178)
  - [Bug 648158: Mozilla office is on fire](https://bugzilla.mozilla.org/show_bug.cgi?id=648158)
  - [Bug 38437: Read this you craps](https://bugzilla.mozilla.org/show_bug.cgi?id=38437)
- Ubuntu [Bug 1310292: installing \`ruby2.0\` results in ruby 1.9.3-p484 as default version](https://bugs.launchpad.net/ubuntu/+source/ruby2.0/+bug/1310292) makes you wonder what people are thinking

I hope you enjoy these as much as I do. Thanks to everyone who contributed links.

Updates from comments and Reddit:

- Angular [Bug 5017: Unit tests fail when run in Australia](https://github.com/angular/angular.js/issues/5017)
- [Bumblebee 1.4.31 had a small problem](https://github.com/MrMEEE/bumblebee-Old-and-abbandoned/commit/a047be85247755cdbe0acce6f1dafc8beb84f2ac)
- MySQL [Bug 20786](https://bugs.mysql.com/bug.php?id=20786) eventually got a birthday party

- Ubuntu [Bug 1](https://bugs.launchpad.net/ubuntu/+bug/1) is still unfixed fixed due to Android

- Chromium [Issue 31482: Huge amount of goats teleported](https://code.google.com/p/chromium/issues/detail?id=31482) and [Issue 125981: The fundamental error in the icon of cheeseburger](https://code.google.com/p/chromium/issues/detail?id=125981) — as far as I know the locale-aware sandwich icon tragically remains unimplemented
