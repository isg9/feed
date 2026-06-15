---
title: 'The Plausibly Deniable DataBase (PDDB): It&#8217;s Real Now!'
url: https://www.bunniestudios.com/blog/2022/the-plausibly-deniable-database-pddb-its-real-now/
published: "2022-07-28T09:17:10Z"
feed: bunnie
guid: https://www.bunniestudios.com/blog/?p=6449
---

# The Plausibly Deniable DataBase (PDDB): It&#8217;s Real Now!

Earlier I described the [Plausibly Deniable DataBase (PDDB)](https://www.bunniestudios.com/blog/?p=6307). It’s a filesystem (like [FAT](https://en.wikipedia.org/wiki/File_Allocation_Table) or [ext4](https://en.wikipedia.org/wiki/Ext4)), combined with plausibly deniable full disk encryption (similar to [LUKS](https://blog.linuxbrujo.net/posts/plausible-deniability-with-luks/) or [VeraCrypt](https://www.veracrypt.fr/en/Plausible%20Deniability.html)) in a “batteries included” fashion. Plausible deniability aims to make it difficult to prove “beyond a reasonable doubt” that additional secrets exist on the disk, even in the face of forensic evidence.

[![](https://betrusted.io/xous-book/images/pddb-example1.png)](https://betrusted.io/xous-book/ch09-01-basis.html#a-short-example)

Since then, I’ve [implemented](https://github.com/betrusted-io/xous-core/tree/main/services/pddb), [deployed](https://www.crowdsupply.com/sutajio-kosagi/precursor/updates/xous-release-v0-9-9-vault-authentication-app-and-more), and [documented](https://betrusted.io/xous-book/ch09-00-pddb-overview.html) the PDDB. Perhaps of most interest for the general reader is the extensive documentation now available in the [Xous Book](https://betrusted.io/xous-book/ch00-00-introduction.html). Here you can find discussions about the [core data structures](https://betrusted.io/xous-book/ch09-01-basis.html), [key derivations](https://betrusted.io/xous-book/ch09-02-rootkeys.html), [native](https://betrusted.io/xous-book/ch09-03-api-native.html) & [std](https://betrusted.io/xous-book/ch09-04-api-std.html) APIs, [testing](https://betrusted.io/xous-book/ch09-05-testing.html), [backups](https://betrusted.io/xous-book/ch09-06-backups.html), and [issues affecting security and deniability](https://betrusted.io/xous-book/ch09-07-discussion.html).
