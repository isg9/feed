---
title: 'research!rsc: Face the Nation'
url: https://research.swtch.com/face
published: "2008-02-27T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/face
---

# research!rsc: Face the Nation

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Face the Nation Russ Cox February 27, 2008 *research.swtch.com/face* Posted on Wednesday, February 27, 2008.

“ [The Hideous Name](http://research.swtch.com/2008/02/hideous-name.html)” makes mention of a face server, perhaps the first user-level file system. The face server's main purpose was to provide faces for *vismon*, which showed a face for each message in a user's (electronic) mail box. Vismon is described in Rob Pike and Dave Presotto's 1985 Usenix paper “ [**Face the Nation**](http://mailglance.com/facethenation.html).”

[![](vismon1.png)](vismon.png)

(The appearance of grey scale is due only to the image being shrunk. Click on the image for the full-size version.) This example shows the time, system load, a graph of cpu time, faces for the five most recent emails, and the names of the two most recent senders.

The [AT&T 5620 Retrocomputing](http://www.brouhaha.com/~eric/retrocomputing/att/5620/) page contains a file tree used in the commercially distributed system, including the face set:

![](faces.png)

(The mailbox with the tail, labeled *md*, is actually *mailer-daemon*, but that label didn't fit.)

The faces include Brian Kernighan (bwk), Dennis Ritchie (dmr), Ken Thompson (ken), and Rob Pike (rob). Curiously, Peter Weinberger's face is missing. It was featured prominently in the paper and in [many other contexts](http://www.spinroot.com/pico/pjw.html), including water towers. It is also used as the default “unknown” icon in *vismon*. Here it is in its two *vismon* instantiations:

![](http://2.bp.blogspot.com/_9P3HFEQk8wo/R7u_tQ4bCgI/AAAAAAAAAEM/EMDSTbAGNd4/s400/pjw.png)

The spirit of vismon lives on in Plan 9's *[faces](https://9p.io/plan9/screenshot.html)* (née *seemail*), Unix's *[faces](http://faces.sourceforge.net/)*, and OS X's [MailGlance](http://mailglance.com/index.html), among [others](http://www.cs.indiana.edu/ftp/faces/index.html).

Vismon also inspired in 1987 the creation of the Usenix [FaceSaver](http://www.metron.com/FaceSaver/) project. Digging around in the archives you can find pictures of luminaries such as [Kirk McKusick](http://www.toad.com/facesaver/loukatz/gif/mckusick@berkeley.edu.gif), [Dennis Ritchie](http://www.toad.com/facesaver/loukatz/gif/dmr@research.att.com.gif), [Henry Spencer](http://www.toad.com/facesaver/loukatz/gif/henry@zoo.toronto.edu.gif), and [Larry Wall](http://www.toad.com/facesaver/loukatz/gif/lwall@jpl-devvax.jpl.nasa.gov.gif).

The [Picons Archive](http://www.cs.indiana.edu/picons/ftp/index.html) has even [more pictures](http://www.cs.indiana.edu/picons/ftp/demo/index.html).

(Comments originally posted via Blogger.)

- [Rklz](http://wikitravel.org/en/User:Rklz)(February 28, 2008 5:37 PM) This is most intriguing. Fox 11 seems to have used the **noface** icon as "Anonymous" in their documentary on 4chan.

And **owl1** and **owl2** are now famous around the Net as the O RLY and YA RLY owls, respectively; "originating" at Something Awful forums. Little did I know...
