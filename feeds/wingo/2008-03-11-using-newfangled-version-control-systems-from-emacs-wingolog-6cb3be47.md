---
title: using newfangled version control systems from emacs — wingolog
url: https://wingolog.org/archives/2008/03/11/using-newfangled-version-control-systems-from-emacs
published: "2008-03-11T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/03/11/using-newfangled-version-control-systems-from-emacs
---

# using newfangled version control systems from emacs — wingolog

## [using newfangled version control systems from emacs](/archives/2008/03/11/using-newfangled-version-control-systems-from-emacs)

11 March 2008 2:23 PM

- [version control](/tags/version%20control)
- [emacs](/tags/emacs)
- [git](/tags/git)
- [bzr](/tags/bzr)

So I'm a big fan of these new distributed version control systems. For most of my personal projects I use [bzr](http://bazaar-vcs.org/), which works fine and doesn't tax the brain too much. At [work](http://oblong.net/) (awesome web site) we use [git](http://git.or.cz/), which I have grown quite fond of.

These systems, while distinct, do share obvious similarities. They're all distributed and more or less fast, they all eschew changelogs, they all do tree-wide atomic commits. So the obvious question I know that you have on your mind is, "Isn't there a rocking emacs mode that can support all of these things?"

**yes**

OK if not, you should probably stop reading. Otherwise, [DVC](http://www.xsteve.at/prg/emacs_dvc/dvc.html) is your answer. It's super. Here's how to get it:

```
cd ~/src
bzr get http://bzr.xsteve.at/dvc/
cd dvc
autoreconf -vif
mkdir ++build/
cd ++build/
../configure
make

```

Then add this to your `.emacs`:

```
(add-to-list 'load-path "~/src/dvc/++build/lisp")
(require 'dvc-autoloads)

```

Then restart emacs (or `C-x C-e` after each of the closing parentheses), and open some file that's in git (or bzr, or monotone, ...). Edit something, save, and type `M-x dvc-diff`. You get a list of files that changed, followed by diffs. You can use `j` to jump back and forth between the file list and the diffs, `n` and `p` to navigate between diffs, or `C-c C-c` to go to the file in question.

Most excellently, pressing `t` will open a new buffer for the commit log, *with a nicely-formatted log entry* (similar to `C-x 4 a`), which you can edit, save, kill, whatever -- the next time you press `t` in the dvc-diff buffer, you'll have that same log message to edit. Until the point at which you want to commit, at which time you might want to press `g` in the dvc-diff buffer to refresh the diff, then `C-c C-c` in the log buffer to actually commit the change.

The nice thing is that all of this works regardless of the VC system that you're using. Each system also has some extra methods you might be interested in (e.g. `xgit-apply-mbox`), but more or less you can treat them the same.

**oppression**

As an aside, I think people that like git do so out of a kind of software [Stockholm syndrome](http://en.wikipedia.org/wiki/Stockholm_syndrome): you have to learn so much about esoterics like refs, the object database, the index, etc. that you end up feeling empathy for git's idiosyncracies. Because objectively, git's working tree index should not be a concept that occupies space in my mind.

## related articles

- [git and bzr](/archives/2009/01/06/git-and-bzr)
- [git-brunch(1)](/archives/2011/03/28/git-brunch1)
- [recent developments in guile](/archives/2010/04/02/recent-developments-in-guile)
- [wikipedia & guile](/archives/2009/10/01/wikipedia-guile)
- [a brief history of guile](/archives/2009/01/07/a-brief-history-of-guile)
- [merging in unrelated git branches](/archives/2008/10/14/merging-in-unrelated-git-branches)

### 7 responses

1. [Brit Butler](http://www.redlinernotes.com/blog/) says:[11 March 2008 2:40 PM](#cc93ec3f23bbcec7e5169d61cd182bbb7f13b2a9)

   Damn Wingo. You read my mind. I'm getting my git server set up and emacs configured right now. I'll probably also do a write up later.

   PS: I think Tekuti was an awesome idea. I'd help out but I don't really know enough Scheme yet. I'm still getting through SICP section 1.3!

2. Anonymous says:[11 March 2008 2:54 PM](#32918ef31662c99aded7fde992ac8e2dccbf2719)

   What does oblong.net do, exactly, other than have an oblong logo? :)

3. Stephane Raimbault says:[11 March 2008 4:33 PM](#12caf7bf72d8fe958dc7888c7bc05bc973344928)

   An autoconf call is missing in your installation guide.

   cd ~/src

   bzr get http://bzr.xsteve.at/dvc/

   cd dvc

   autoconf

   mkdir ++build/

   cd ++build/

   ../configure

   make

4. q says:[11 March 2008 5:15 PM](#55c7dbf985b3631472200dbf36c455bf26e6f6c1)

   Alternatively you can use emacs from CVS and things should work out of the box.

   On top of that you also get support for anti-aliased fonts and the ability to use the same emacs instance with X11 and tty frames.

5. [wingo](http://wingolog.org/) says:[11 March 2008 5:30 PM](#f1b8d04ae7ea67133ca6018862ddfc78abb7538a)

   @anon: Check the [archives](http://wingolog.org/tags/oblong).

   @Stephane: Yeeps, thanks for the correction.

   @q: I do run CVS emacs, at least at work; but it doesn't have DVC, which is the hotness.

6. q says:[11 March 2008 5:41 PM](#1bbcf2838c6231d9eed2f7bba6d464c9dcb46e19)

   CVS emacs does not have DVC, but you might find that VC there is quite good.

7. Jakub Narebski says:[11 March 2008 10:21 PM](#5199b41e6e139a07fb7eb2872583d779cd979232)

   About git index:

   Usually you don't have to worry about it. You just have to remember to use "git commit -a"; I guess that DVC does that for you (does DVC works with GNU Emacs 21?).

   But when it is needed, for example with more complicated merge conflict, or need for partial commit (commit of some, but not all, changes in your working area), or generating commit part by part examining what have changed since last part, or un-adding a file, or recovering added but not yet comitted file which was removed by mistake... then you would be grateful that it is there available for you.

Comments are closed.
