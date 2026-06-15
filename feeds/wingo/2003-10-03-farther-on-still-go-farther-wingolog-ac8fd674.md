---
title: farther on, still go farther — wingolog
url: https://wingolog.org/archives/2003/10/03/advogato-19
published: "2003-10-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2003/10/03/advogato-19
---

# farther on, still go farther — wingolog

## [farther on, still go farther](/archives/2003/10/03/advogato-19)

3 October 2003 7:12 AM

- [namibia](/tags/namibia)
- [scheme](/tags/scheme)

**hackin**

Still working on guile-gobject, off and on. It's now a [GNU project](http://www.gnu.org/software/guile-gtk/), which makes development a bit interesting. We don't have any releases out yet as GNU software, but that's coming soon, hopefully.

There are two interesting developments on that front. First, [Guile 1.8 will support multiple pthreads within guile](http://mail.gnu.org/archive/html/guile-devel/2003-09/msg00087.html). Of course there's locking during GC, but that development pleases me: you can handle callbacks from other threads without having to worry about a global interpreter lock. This bodes well for [gst-guile](http://gstreamer.net/bindings/guile/) especially.

Second, I just hacked out some libglade bindings, complete with glade-xml-signal-connect and glade-xml-signal-autoconnect. It only took three or four hours to do, too. I dig this project!

[soundscrape](http://ambient.2y.net/soundscrape/) progresses, although I need to commit some things. The music model is proving very interesting to work on, although I need to finish an enveloptor (to cause sounds to have a physical end) and a spawner (to spawn new events at timed intervals) to make sounds.

Also, I think me and Iain will be sharing (read: I'm stealing ;) some of the backend code from his [Marlin](http://marlin.sf.net) project. I'm super-excited about that: the user can spawn a dynamic synthesis network and view/edit the output in a Marlin window. Rawk!

**teachin**

The learners are learning, and the teacher is teaching. I'm also excited on this front -- it's true what they say, teaching is all about relationships. Although one boy was using the magnifying glass to burn people today and a couple of his victims have blisters (!). These kids!

**life**

Hard to believe, at the end of the month I'll have been in Africa for a year. It's a nice place. I think I'll stay another year at least.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [poverty, riches](/archives/2009/05/14/poverty-riches)
- [culture and code](/archives/2004/12/06/culture-and-code)
- [cromulation](/archives/2004/11/24/cromulation)
- [photobloggin](/archives/2004/10/17/photobloggin)
- [Antelopes](/archives/2004/09/19/the-countdown-begins)

Comments are closed.
