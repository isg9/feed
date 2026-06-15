---
title: Middleman Parallelization
url: https://nullprogram.com/blog/2010/09/08/
published: "2010-09-08T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2010/09/08/
---

# Middleman Parallelization

## [Middleman Parallelization](/blog/2010/09/08/)

September 08, 2010

nullprogram.com/blog/2010/09/08/

I recently discovered a very clever tool called [Middleman](http://mdm.berlios.de/usage.html). It's a quick way to set up and manage multiple-process workload queue. The process output and display is done inside of a screen session, so if it's going to take awhile you can just detach and check on it again later. In the past I used `make`'s `-j` option to do this, but that's always a pain to set up.

It is composed of three programs: `mdm.screen`, `mdm-run`, and `mdm-sync`. The first is the top level supervisor that you use to launch the enhanced shell script. The second prefixes every command to be run in parallel. The third is prefixes the final command that depends on all of the individual processes.

The linked Middleman page has a good example, but I'll share my own anyway. I used it over the weekend to download a long series of videos with [youtube-dl](http://rg3.github.com/youtube-dl/). Because the transfer rate for a single video is throttled I wanted to grab several at a time, but I also didn't want to grab them *all* at the same time. Here's the dumb version of the script, `download.sh`, that does them all at once.

```
#!/bin.sh
youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX0 &
youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX1 &
youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX2 &
youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX3 &
youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX4 &
youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX5 &
youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX6 &
youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX7 &
youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX8 &

```

With Middleman all I had to do was this,

```
#!/bin/sh
mdm-run youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX0
mdm-run youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX1
mdm-run youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX2
mdm-run youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX3
mdm-run youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX4
mdm-run youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX5
mdm-run youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX6
mdm-run youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX7
mdm-run youtube-dl -t http://www.youtube.com/watch?v=XXXXXXXXXX8

```

Then just launch the script with `mdm.screen`. It defaults to 6 processes at a time, but you can adjust it to whatever you want with the `-n` switch. I used 4.

```
$ mdm.screen -n 4 ./download.sh

```

There is a screen window that lists the process queue and highlights the currently active jobs. I could switch between screen windows to see the output from individual processes and see how they were doing.

From the perspective of the shell script, the first four commands finish instantly but fifth command will block. As soon as Middleman sees one of the first four processes complete the fifth one will begin work, returning control to the shell script, and the sixth command will block, since the queue is full again.

I'm sure I'll be using this more in the future, especially for tasks like batch audio and video encoding. I bet this could be useful on a [cluster](/blog/2007/09/17/).

- [trick](/tags/trick/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Middleman%20Parallelization) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Middleman+Parallelization).

« [Java is Death By A Thousand Paper Cuts](/blog/2010/08/13/)

» [Throw Up a Quick HTTP Server](/blog/2010/09/21/)
