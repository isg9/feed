---
title: nine months with arch — wingolog
url: https://wingolog.org/archives/2004/10/01/nine-months-with-arch
published: "2004-10-01T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/10/01/nine-months-with-arch
---

# nine months with arch — wingolog

## [nine months with arch](/archives/2004/10/01/nine-months-with-arch)

1 October 2004 1:39 PM

- [computers](/tags/computers)

### in the beginning, there was autofoo

[Advogato](http://advogato.org/recentlog.html?thresh=3) people, think back to when you first learned how to hack free software. Getting all the auto\* confused, figuring out how those macros worked (don't put a space before the paren), making dlopen-able libraries, navigating \`info', etc. Bizarre. The extent of the strangeness is evident if we consider the "new project": Who, upon creating a new project, would write the auto\* files from scratch? Deep voodoo is best done once and for all.

Above all, though, was CVS. Import, checkin, add, remove, weird options to get new directories into your copy, CVS\_RSH, CVSROOT, CVSEDITOR, ssh-agent, etc. More than that was the idea of CVS, that you might record, document, and preserve your changes. Although the first-time CVS user has creating documents for quite some time, the idea that one could version them was completely new. It was a revolution in the way I worked.

Over time, I grew to know CVS like second nature. I still couldn't tag or branch without looking at the redbean book, but with everything else I was a champ. It's like commuting on a bicycle -- after a while you don't even notice it, you just think about where you're going.

I survived like that for five years. Last year, though, I got the itch to hack gstreamer from within guile. I found the old guile-gobject project, took it over as my own, and starting making releases. It was a bit strange because it was split over a CVS archive on gnome.org and one on savannah, but there was nothing I could do about that.

That's about when Andreas Rottmann came in. He took over the [FFI generation sub-project](http://www.nongnu.org/g-wrap/) and started to hack guile-gnome, adding a couple of wrappers. His changes were on a branch of the FFI package, and the needed changes in guile-gnome on another branch. Then the GNOME platform bindings process began, which demands the whole platform in one tarball. Andreas worked out a way to split guile-gnome into mutiple packages, while retaining the ability to release many from within the same tarball.

The only catch was, to do this I needed to use GNU arch for source management. I didn't want to change. I was happy and productive with CVS, and I didn't even have arch installed on my machine.

I feel like I've been living with Tom Lord for the last nine months. It's like he's over my shoulder, telling me what to name my files, how to build my software, and how to deal with revision control. With apologies to the man himself, whom I've never met, this post chronicles our life together.

(Ha! I tripped your "this is wierd" meter!)

### the inventory

Arch wants to know about everything that is contained in your "project tree". It divides files into five types, which is akin to a manifest type system where regexps are the type predicates. Some regexps are predefined, like files beginning with `=` or `,`. Files that don't match one of the predefined types by default are treated as source.

However, if a file looks like source, but has not been added to the inventory (akin to `cvs add`), arch will barf. It's as if CVS would stop if you forgot to add a .o file to .cvsignore.

This isn't a huge problem for Tom, because he wants you to build in a separate directory. I won't do that for various reasons. The whole system makes me think of the puritan Protestant approach to economics -- *"You've had it good for a long time, we need a contraction to cleanse out the inflation demons. Then we will be reborn and free from sin. This is going to hurt a bit, but we've had it coming."* Your project tree should be picked clean as a bone, or else.

Fortunately, one can programmatically construct .arch-inventory files (like .cvsignore) in each directory. I have a script to do that [here](http://ambient.2y.net/wingo/tmp/tla-update-inventories). In fact, if there is an arch operation that you are lacking, it's usually possible to implement it in sh -- either built on more primitive tla commands, but as a last-ditch effort you can read the archive or control files directly. [tla-tools](http://wiki.gnuarch.org/moin.cgi/tla_2dtools) implements a number of them, fix-changelog-conflicts and commit-merge being two of the more useful ones.

### decentralization: method

I live in the bush of Africa. I carry every byte on my back, walking to my internet connection. I need an offline solution.

Arch provides this in two ways. First, each project tree contains a "pristine" copy of the upstream revision being hacked. That means you can diff and patch your tree without accessing the archive.

Secondly, arch has extensive mirroring capabilities. If you mirror a remote archive locally, you can do any read-only operation on your copy that you could on the remote copy. It is also possible to publish your private archive by mirroring it to a remote location. People can get revisions and make branches from my archive even as I sit here on the homestead.

Without the ability to branch between archives, all of this would be of little value. Arch's ability to branch and merge takes some getting used to, but it's pretty good. Each tree knows the set of patches that have been applied to it, which makes it easy to determine the patches needed from other branches. The star-merge operator automates this process.

I've been pretty happy with all of this. I can do experimental hacking in my branches, only merging them back upstream when the code is in a good state.

### decentralization: exegesis

I hate being a loser, but I like Scheme too much. My illusions that the rest of the world will one day see the light are dim. I'm almost resigned to hacking in a marginal language, where the pool of available developers is small. The success of projects in such an environment is dependent on the dedication and interest of solitary programmers, because there really isn't anyone else to do it.

That's my rationalization, anyway. How else would one explain [guile-gnome](http://home.gna.org/guile-gnome/)? [Martin Baulig](http://primates.ximian.com/~martin/blog/index.php) laid some firm groundwork, making lots of progress with GObject and ORBit wrappers, but the hacking stopped when he did in 2001.

It took two years for development to really start again. In this environment, we need to lower the barriers to development, so that people can take over and hack, even releasing their own versions, all with the full benefits of revision control right from the start. It allows a lone hacker to easily pick up a fallow project without knowing its old maintainer, who may have even dropped off the net.

### assessment

I'm comfortable with arch now. I wouldn't recommend it to a happy CVS user, but I don't think I will ever go back.

As a final note, I should mention that I can't compare arch to subversion, because I've never used SVN. So don't take this as a polemic. Thanks.

### meta

A web log, like this one is, is somewhere between the advogato diary entry and the advogato article. I hope advogato folks don't mind my verbosity.

### notes

**The count:** 75 days until airplane.

Namblish (Namibian broken english) uses the gerund a lot: I am having, I was feeling, etc. I am infected. You are affected.

My learners have found IRC. It's hilarious. I worry about pedophiles a bit -- everyone wants to exchange photos, but I wonder why 30 year old men would want to talk to ninth grade boys. There's not much they could do, though, and I do check up on the kids. Strange to be in this kind of supervisory role at 24.

[luis](http://tieguy.org/blog/index.cgi/194.html): Yikes, I feel naked! I'll have to get on the administrator's case, who by the way is a KDE contributor. Small world.

## related articles

- [Roundup](/archives/2005/10/27/roundup)
- [ppc ruminations](/archives/2005/06/09/ppc-ruminations)
- [computerisation](/archives/2005/05/26/computerization)
- [pretending we're bunny rabbits](/archives/2005/04/25/pretending-we)
- [vaquero](/archives/2005/04/20/vaquero)
- [Rollin the dice](/archives/2005/04/09/101)

Comments are closed.
