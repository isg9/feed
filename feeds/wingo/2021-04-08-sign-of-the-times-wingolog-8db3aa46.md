---
title: sign of the times — wingolog
url: https://wingolog.org/archives/2021/04/08/sign-of-the-times
published: "2021-04-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2021/04/08/sign-of-the-times
---

# sign of the times — wingolog

## [sign of the times](/archives/2021/04/08/sign-of-the-times)

8 April 2021 7:09 PM

- [gnu](/tags/gnu)
- [guile](/tags/guile)
- [libtool](/tags/libtool)
- [ffi](/tags/ffi)

Hello all! There is a mounting backlog of things that landed in Guile recently and to avoid having to eat the whole plate in one bite, I'm going to try to send some shorter missives over the next weeks.

Today's is about a silly thing, [`dynamic-link`](https://www.gnu.org/software/guile/manual/html_node/Foreign-Libraries.html). This interface is [`dlopen`](https://man7.org/linux/man-pages/man3/dlopen.3.html), but "portable". See, back in the day -- like, 1998 -- there were lots of kinds of systems and how to make and load a shared library portably was hard. You'd have people with AIX and Solaris and all kinds of weird compilers and linkers filing bugs on your project if you hard-coded a GNU toolchain invocation when creating loadable extensions, or hard-coded `dlopen` or similar to use them.

[Libtool](https://www.gnu.org/software/libtool/) provided a solution to create portable loadable libraries, which involved installing `.la` files alongside the `.so` files. You could use libtool to link them to a library or an executable, or you could load them at run-time via the libtool-provided [`libltdl`](https://www.gnu.org/software/libtool/manual/html_node/Using-libltdl.html#Using-libltdl) library.

But, the `.la` files were a second source of truth, and thus a source of bugs. If a `.la` file is present, so is an `.so` file, and you could always just use the `.so` file directly. For linking against an installed shared library on modern toolchains, the `.la` files are strictly redundant. Therefore, all GNU/Linux distributions just delete installed `.la` files -- Fedora, Debian, and even Guix do so.

Fast-forward to today: there has been a winnowing of platforms, and a widening of the GNU toolchain (in which I include LLVM as well as it has a mostly-compatible interface). The only remaining toolchain flavors are GNU and Windows, from the point of view of creating loadable shared libraries. Whether you use libtool or not to create shared libraries, the result can be loaded either way. And from the user side, `dlopen` is the universally supported interface, outside of Windows; even Mac OS [fixed their implementation a few years back](https://developer.apple.com/library/archive/documentation/System/Conceptual/ManPages_iPhoneOS/man3/dlopen.3.html).

So in Guile we have been in an unstable equilibrium: creating shared libraries by including a probably-useless libtool into the toolchain, and loading them by using a probably-useless libtool-provided `libltdl`.

But the use of `libltdl` has not been without its costs. Because `libltdl` intends to abstract over different platforms, it encourages you to leave off the extension when loading a library, instead promising to try a platform-specific set such as `.so`, `.dll`, `.dylib` etc as appropriate. In practice the abstraction layer was under-maintained and we always had some problems on Mac OS, for example.

Worse, as ltdl would search through the path for candidates, it would only report the *last* error it saw from the underlying `dlopen` interface. It was almost always the case that if `A` and `B` were in the search path, and `A/foo.so` failed to load because of a missing dependency, the [error you would get as a user would instead be "file not found"](https://lists.gnu.org/archive/html/bug-libtool/2008-10/msg00017.html), because ltdl swallowed the first error and kept trucking to try to load `B/foo.so` which didn't exist.

In summary, this is a case where the benefits of an abstraction layer decline over time. For a few years now, `libltdl` hasn't been paying for itself. Libtool is dead, for all intents and purposes (last release in 2015); best to make plans to migrate away, somehow.

In the case of the `dlopen` replacement, in Guile we ended up [rewriting the functionality in Scheme](https://git.savannah.gnu.org/gitweb/?p=guile.git;a=blob;f=module/system/foreign-library.scm;h=d53e293ef2874e41aeff85f4c371e17c973fae6e;hb=HEAD). The underlying facility is now just plain `dlopen`, for which we shim a version of `dlopen` on Windows, inspired by the implementation in cygwin. There are still platform-specific library extensions, but that is handled easily on the Scheme layer.

Looking forward, I think it's probably time to replace Guile's use of libtool to create its libraries and executables. I loathe the fact that libtool puts shell scripts in the place of executables in build directories and stashes the actual executables elsewhere -- like, visceral revulsion. There is no need for that nowadays. Not sure what to replace it with, nor on what timeline.

And what about autotools? That, my friends, would be a whole nother blog post. Until then, & probably sooner, happy hacking!

## related articles

- [in which our protagonist dreams of laurels](/archives/2025/12/17/in-which-our-protagonist-dreams-of-laurels)
- [whippet in guile hacklog: evacuation](/archives/2025/06/11/whippet-in-guile-hacklog-evacuation)
- [on the impossibility of composing finalizers and ffi](/archives/2024/02/26/on-the-impossibility-of-composing-finalizers-and-ffi)
- [ephemeral success](/archives/2022/12/15/ephemeral-success)
- [i'm throwing ephemeron party & you're invited](/archives/2022/12/12/im-throwing-ephemeron-party-youre-invited)
- [we iterate so that you can recurse](/archives/2022/12/11/we-iterate-so-that-you-can-recurse)

### One response

1. Reuben Thomas says:[9 April 2021 10:04 AM](#c290bb8c91e18e8ddd7994a45006653ed105928a)

   Yeah, libtool is ugly, but (if and while you're using autotools), it gets the job done for portable code that you want to run on Windows.

   Enchant, which I maintain, which is a meta-spellchecker that uses dynamically-loaded plugins to talk to the actual spellchecking backends, manages to get rid of libltdl by using glib's GModule interface, though it still uses libtool for building those modules.

   FWIW, I think that even getting rid of libltdl (i.e. removing it from future libtool releases) would be a win. And at that point, dropping libtool just becomes part of the problem of dropping autotools.

Comments are closed.
