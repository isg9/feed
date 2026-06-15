---
title: Is the Browser the New OS?
url: https://blog.regehr.org/archives/857
published: "2012-12-20T19:03:32Z"
feed: regehr
guid: http://blog.regehr.org/?p=857
---

# Is the Browser the New OS?

Yes, this is an old question. I still think it’s interesting. Disclaimer: I haven’t tried out a Chromebook yet.

**First, let’s look at the situation as of late 2012.** The applications I use generally fall into three categories:

1. Web-based.
2. Native, but easily available on Windows, Mac, and Linux. These include a file browser, a shell, Emacs, LaTeX.
3. Native and less portable: Photoshop, development tools such as compilers and debuggers, high-performance games.

A quick look at the Chrome store indicates that most or all of the applications in category #2 are already available in browser versions (though I currently use the native versions out of inertia). Category #3 is trickier; it’s not clear that it really makes sense to port, for example, Photoshop into the browser. On the other hand, category #3 is basically power tools and many people (including me, much of the time) can get by using weaker web-based versions. Even embedded software development tools, which might at first seem to be the antithesis of web applications, have [web-based versions](http://mbed.org/). In summary, to a first approximation, “the browser is the new OS” has mostly happened already, though we do see an interesting reverse trend in the mobile space (though quite a few of those native apps are thin wrappers around a browser).

**Second, how did we get to this point?** The first application I ever saw that used a browser as its primary interface was [SATAN](http://en.wikipedia.org/wiki/Security_Administrator_Tool_for_Analyzing_Networks) in 1995. I remember thinking this was an elegant and striking design decision; it permitted the implementors to focus on their application logic instead of wasting time on what would likely have ended up being the kind of crappy X windows GUI that was standard at the time. Not long after 1995 we saw the rise of Java-enabled browsers making everyone (for a short time, at least) get interested in the potential for platform-independent applications delivered over the web. But this potential remained unrealized. Around 2008 it became clear that the following three efforts, taken together, constituted a serious attempt to create a new platform. First, Google Chrome, with a fast JavaScript engine and significantly increased robustness (and perhaps also security) due to the process-per-tab model. Second, Google Gears, which facilitated disconnected operation for web applications. And third, Google Docs, which implements at least 80% of the useful stuff in MS Office. Of course Gears is gone but similar functionality is available elsewhere. More recently, Chrome OS and the Chromebook make it clear what Google wants to see. I’m painting a sort of Google-centric picture here and in fact I use their stuff a lot. However, most of the time I could get by using Firefox, Bing, and other alternative technologies.

**Finally, what is the prognosis for browser-is-the-OS, looking forward?** What will it take for people to really not care about which OS they’re running? First, we want to minimize the number of applications in category #3, but realistically I think most casual users don’t have that many apps in this category and us power users are willing to compromise. For example, the special-purpose tools I use for research are probably always going to run outside the browser and that is fine—they’ll run on workstation-class machines at the office and in the cloud. Photoshop and high-quality video processing is not likely to be moving into the cloud real soon, but on the other hand again these are special-purpose, workstation-grade applications. Most people already do their photo editing online.

The second thing that I require is that any web application that looks remotely like an editor (whether for programs, documents, presentations, web pages, or whatever) has to support rock-solid disconnected operation and have a sensible conflict resolution mechanism. Google Docs’ disconnected operation seems pretty good since last summer, but I worry that lots of applications are going to need special-purpose logic to handle this nicely, and it’s going to be a long slog since many developers won’t consider disconnected to be a priority.

Third, we need near-native performance in the browser. Plenty of JavaScript work has been done and it’s getting pretty fast. For more compute-intensive workloads Google Native Client seems like a good option.

In summary, we seem to be headed towards a segregated world where most people don’t need a PC, but rather an appliance that runs a browser. On the other hand, people who use computers more seriously will keep using workstation-style machines that run native apps when considerations such as performance, dealing with large amounts of data, security, reliability, and local control are paramount. I’ll go ahead and predict that in 2022, the number of PCs sold will be 25% of what they were in 2012. By PC I mean something like “non-Android, non-iOS machine primarily intended to be used by a single person to run native applications”—a category that includes all laptops except Chromebooks.
