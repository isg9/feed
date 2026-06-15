---
title: Almost Everything in Subversion
url: https://blog.regehr.org/archives/339
published: "2011-01-07T18:25:30Z"
feed: regehr
guid: http://blog.regehr.org/?p=339
---

# Almost Everything in Subversion

Every file I use on a day to day basis — excluding only data shared with other people, email folders, and bulk media — is kept in a big subversion repository. For the five years that I’ve been doing this, I’ve averaged 3.5 commits per day. Overall it works really well. Advantages of this scheme include:

- Seamless operation across Mac, UNIX, and Windows.
- I’m always using a local copy of data, so access is fast and there’s no problem if I lose the network.
- Synchronization is fast and low-bandwidth, so it works fine using airport and coffee shop connections.
- My machines become stateless; reinstalling an OS is hassle-free and I never backup hard disks anymore.
- Explicit add of files means that large temporary objects are never dragged across the network.

On the other hand, a few aspects of the plan are less than ideal:

- Explicit push, pull, and add means I have to always remember to do these, though I seldom forget anymore.
- Every now and then I have a file that seems a bit too large to checkin to svn, but that doesn’t really fit into my bulk media backup plan.
- Subversion’s diffing and patching is geared towards text files, and works less than ideally for binary objects such as the .docx and .pptx files I sometimes need to deal with.

But overall I’m totally happy with working this way and probably won’t change for years to come.
