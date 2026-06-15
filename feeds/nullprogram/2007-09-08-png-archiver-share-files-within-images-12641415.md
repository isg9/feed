---
title: PNG Archiver - Share Files Within Images
url: https://nullprogram.com/blog/2007/09/08/
published: "2007-09-08T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2007/09/08/
---

# PNG Archiver - Share Files Within Images

## [PNG Archiver - Share Files Within Images](/blog/2007/09/08/)

September 08, 2007

nullprogram.com/blog/2007/09/08/

This is one of my projects.

- [PNG Archiver](https://github.com/skeeto/pngarch)

The original idea for this project came from Sean Howard’s [Gameplay Mechanic’s #012](http://www.squidi.net/three/entry.php?id=12). The basic idea here is that image files are the second easiest type of data to share on the Internet (the first being text). Sharing anything other than images may be difficult, so why not store files within an image as the image data? This is not [steganography](http://en.wikipedia.org/wiki/Steganography) as the data is not being hidden. In fact, the data is quite obvious because we are trying to make the data as compact as possible in the image.

My “PNG Archiver” is usable but should still be considered alpha quality software. I am adding support for different types of PNGs (currently it does 8-bit RGB only), but I have found that using the libpng library gives me headaches. The archiver can actually only store a single file (just as gzip doesn’t know what a file is). This is because I do not want to duplicate the work of real file archivers like `tar`. To store multiple files, make a “png-tarball”.

The PNG Archiver stores a checksum in the image that allows it to verify that the data was received correctly. This also allows it to automatically scan the image for data. When it reads in a piece that fulfills the checksum it assumes that it found the data you are looking for. You can decorate the image with text or a border and the archiver should still find the data as long as you didn’t disturb it. (examples of this on the project page)

- [c](/tags/c/)
- [trick](/tags/trick/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20PNG%20Archiver%20-%20Share%20Files%20Within%20Images) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=PNG+Archiver+-+Share+Files+Within+Images).

« [YouTube with Free Software](/blog/2007/09/05/)

» [BCCD Clusters](/blog/2007/09/17/)
