---
title: convert a bazaar repository to a set of patches — wingolog
url: https://wingolog.org/archives/2007/12/17/convert-a-bazaar-repository-to-a-set-of-patches
published: "2007-12-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/12/17/convert-a-bazaar-repository-to-a-set-of-patches
---

# convert a bazaar repository to a set of patches — wingolog

## [convert a bazaar repository to a set of patches](/archives/2007/12/17/convert-a-bazaar-repository-to-a-set-of-patches)

17 December 2007 8:54 PM

- [version control](/tags/version%20control)
- [bazaar](/tags/bazaar)

A couple of weeks ago I was hacking on a GStreamer element from [gst-plugins-bad](http://gstreamer.freedesktop.org/modules/gst-plugins-bad.html) and realized that I needed a way to record my changes in pieces, but not commit to the repository. GStreamer uses CVS, which does not support this workflow.

[Bazaar](http://bazaar-vcs.org/) to the rescue, then. I created a new local repository, in the plugin directory. It's as simple as typing `bzr init`, adding the files you want to track, and then committing as you make changes. I ended up committing a dozen patches or so.

Of course, when I was finished I wanted to push the changes upstream. Thus the point of this writing product, a script to turn a bzr repository into a series of patch files on disk:

```
#!/bin/bash
set -e
patchbase=$(basename `pwd`)
outputdir=$1
if test -z "$outputdir"; then outputdir=.; fi

revno=`bzr version-info | grep revno | cut -d: -f2`
echo "exporting $(($revno)) patches..."
for ((i=0; $i<$revno; i=$i+1)); do
    file="$patchbase.diff.$(($i+1))"
    echo "exporting $i..$(($i+1)) to $file"
    # dunno why bzr always returns $? != 0
    bzr log -r$(($i+1))..$(($i+1)) > $file || true
    bzr diff -r$i..$(($i+1)) >> $file || true
done

```

I then reverted my tree, applying and committing the patches one-by-one with e.g. `patch -u -p0 < foo.diff.4`. I'm sure there's some kind of more integrated plugin to do this, but a 10-minute shell script was easiest to hack out.

As an aside, `bzr uncommit` is quite useful for producing a readable history -- I often go back and modify the committed patches so that they are more understandable on their own.

Happy hacking!

## related articles

- [the phoneless](/archives/2007/05/04/the-phoneless)
- [git-brunch(1)](/archives/2011/03/28/git-brunch1)
- [git and bzr](/archives/2009/01/06/git-and-bzr)
- [using newfangled version control systems from emacs](/archives/2008/03/11/using-newfangled-version-control-systems-from-emacs)
- [curiosity and the catechism](/archives/2006/11/05/curiosity-and-the-catechism)
- [group w bench](/archives/2006/09/01/group-w-bench)

### 6 responses

1. [Wouter Bolsterlee](http://uwstopia.nl/) says:[17 December 2007 9:09 PM](#3352)

   You might want to look into the "shelf" functionality offered by the (almost standard) bzrtools plugin set. It allows for stacked patches and seems to do the job you described perfectly well.

2. [Ali Sabil](http://asabil.wordpress.com) says:[17 December 2007 11:53 PM](#3353)

   Hello,

   Maybe you definitely want to give the "record" plugin of bzr a try, it is exactly what you are looking for, and on top of that it records the patches in a patches/ subfolder manageable by quilt.

   cheers

3. [wingo](http://wingolog.org/) says:[18 December 2007 3:16 AM](#3355)

   Ali: The link on http://bazaar-vcs.org/BzrPlugins to the record plugin seems to be in the "out of date" section.

4. [James Henstridge](http://blogs.gnome.org/jamesh/) says:[18 December 2007 9:41 AM](#3360)

   Perhaps Tailor can do what you want. I've never used it to convert from Bazaar to CVS before though, so it is the kind of thing you'd want to try on a copy of the repository first.

   http://progetti.arstecnica.it/tailor/

5. Steven says:[18 December 2007 11:16 PM](#3365)

   Git can definitely be used to import a CVS repository, but I'm not sure if it can directly convert git commits back into CVS commits. In either event, its as easy as:

   git format-patch HEAD~N

   to convert the last N commits into patches. For subversion its even easier, as git-svn allows a bidirectional workflow. You fetch the subversion repo with git, make you changes in as many chunks as you like with git, and then you can have git-svn send each of your local changes back to svn as a commit.

6. [wingo](http://wingolog.org/) says:[19 December 2007 1:35 AM](#3367)

   Steven: Neat trick!

Comments are closed.
