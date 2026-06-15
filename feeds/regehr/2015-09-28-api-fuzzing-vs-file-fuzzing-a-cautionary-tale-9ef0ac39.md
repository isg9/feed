---
title: 'API Fuzzing vs. File Fuzzing: A Cautionary Tale'
url: https://blog.regehr.org/archives/1269
published: "2015-09-28T13:49:22Z"
feed: regehr
guid: http://blog.regehr.org/?p=1269
---

# API Fuzzing vs. File Fuzzing: A Cautionary Tale

Libraries that provide APIs should be rock solid, and so should file parsers. Although we can use fuzzing to ensure the solidity of both kinds of software, there are some big differences in how we do that.

A file parser should be fully robust: it isn’t allowed to crash even if presented with a corrupted file or the wrong kind of file.Although a library that implements an API usually does some validation of arguments, this is generally a debugging aid rather than an attempt to withstand hostility.A file usually originates outside of a trust boundary.There generally is not a trust boundary between a library and the program that uses the library.A file parser can perform arbitrary checks over its inputs.A C/C++ library providing an API cannot check some properties of its input. For example, it cannot check that a pointer refers to valid storage. Moreover, APIs are often so much in the critical path that full error checking would not be desirable even if it were possible.

For fuzzing campaigns the point is this:

The sole concern of a file fuzzer is to generate interesting files that expose bugs in the file parser or in subsequent logic.An API fuzzer must balance two goals. Like the file fuzzer, it needs to expose bugs by using the API in interesting ways. However, at the same time, it must stay within the usage prescribed by the API — careful reading of the documentation is required, as well as a bit of finesse and good taste.

As a concrete example, let’s look at [libpng](http://www.libpng.org/pub/png/libpng.html), which provides both a file parser and an API. To fuzz the file parsing side, we generate evil pngish files using something like afl-fuzz and we wait for the library to do something wrong. But what about fuzzing the API? We’ll need to look at the documentation first, where we find text like this:

**```** **To write a PNG file using the simplified API:**

**1) Declare a 'png_image' structure on the stack and memset()** **it to all zero.**

**2) Initialize the members of the structure that describe the** **image, setting the 'format' member to the format of the** **image in memory.**

**3) Call the appropriate png_image_write... function with a** **pointer to the image to write the PNG data.**

**```**

Hopefully it is immediately obvious that calling random API functions and passing random crap to them is not going to work. This will, of course, cause libpng to crash, but the crashes will not be interesting because they will not come from valid usage of the library.

You might ask why anyone would care to fuzz an API. Aren’t trust boundaries the things that matter? The answer is easy: If you are looking to find or prevent exploitable vulnerabilities, you should always focus on fuzzing at trust boundaries. On the other hand, if you are more generally interested in reliable software, you should fuzz APIs as well. We might use API fuzzing to ensure that `printf` prints floats correctly and that the red-black tree you coded up late last night doesn’t spuriously drop elements.

Now let’s look at a [blog post by GDS from last week about fuzzing mbed TLS](http://blog.gdssecurity.com/labs/2015/9/21/fuzzing-the-mbed-tls-library.html). There’s something kind of interesting going on here: they are doing API fuzzing (of the mbed TLS library) but they are doing it using afl-fuzz: a file fuzzer. This works well because afl-fuzz provides the data that is “sent” between the client and server (sent is in quotes because this fuzzing effort gains speed and determinism by putting the client and server in the same process).

At the bottom of the post we read this:

> Our fuzzing scans discovered client-side NULL pointer dereference vulnerabilities in mbed TLS which affect all maintained versions of mbed TLS. ARM fixed the issues in versions 2.1.1, 1.3.13 and PolarSSL version 1.2.16. See the [release notes](https://tls.mbed.org/tech-updates/releases/mbedtls-2.1.1-and-1.3.13-and-polarssl-1.2.16-released) for details.

However, the corresponding text in the release notes is this:

> Fabian Foerg of Gotham Digital Science [found a possible client-side NULL pointer dereference, using the AFL Fuzzer](http://blog.gdssecurity.com/labs/2015/9/21/fuzzing-the-mbed-tls-library.html). This dereference can only occur when misusing the API, although a fix has still been implemented.

In other words, the blog post author disagrees with the mbed TLS maintainers about whether a bug has been discovered or not. It is a matter of debate whether or not this kind of crash represents a bug to be fixed. I would tend to agree with the mbed TLS maintainers’ decision to err on the side of defensive programming here. But was there a vulnerability? That seems like a stretch.

In summary, API fuzzing is different from file fuzzing. Good API fuzzing requires exploration of every legal corner of the API and no illegal corners. Ambiguous situations will come up, necessitating judgement calls and perhaps even discussions with the providers of the API.
