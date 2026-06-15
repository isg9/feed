---
title: merging in unrelated git branches — wingolog
url: https://wingolog.org/archives/2008/10/14/merging-in-unrelated-git-branches
published: "2008-10-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/10/14/merging-in-unrelated-git-branches
---

# merging in unrelated git branches — wingolog

## [merging in unrelated git branches](/archives/2008/10/14/merging-in-unrelated-git-branches)

14 October 2008 3:57 PM

- [git](/tags/git)
- [merge](/tags/merge)
- [hackery](/tags/hackery)
- [history](/tags/history)
- [fabrication](/tags/fabrication)

Sometimes when you develop code outside the "official" repository for some project, it turns out that after a few months of hacking, your code is actually suitable for inclusion in the mainline.

Typically when this is the case, the code is just imported directly, throwing away the historical records of how that code came to be.

This is a travesty. Not only is the future deprived of the past, you as a hacker do not get sufficient recognition of your efforts. (What good are [perfect patches](http://www.gnome.org/~federico/news-2008-08.html#git-rebase-interactive) if you do not reap their accolades?)

But suffer not: while Git does not support direct merges between unrelated branches, it can be made to comply. Script here: [`git-merge-unrelated-branch`](//wingolog.org/pub/git-merge-unrelated-branch). ( [Example commit](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=commit;h=e610dc3851da716e6ee4568f94f5f7cace84d2d9).)

Be not deprived of your just deserts!

## related articles

- [git-brunch(1)](/archives/2011/03/28/git-brunch1)
- [a brief history of guile](/archives/2009/01/07/a-brief-history-of-guile)
- [git and bzr](/archives/2009/01/06/git-and-bzr)
- [git: a transcriptional database](/archives/2008/04/12/git-a-transcriptional-database)
- [using newfangled version control systems from emacs](/archives/2008/03/11/using-newfangled-version-control-systems-from-emacs)
- [biting the hand that feeds](/archives/2008/03/07/biting-the-hand-that-feeds)

### 5 responses

1. [Maciej Piechotka](http://uzytkownik.jogger.pl/) says:[14 October 2008 5:24 PM](#642e8247389a27f939d4df64f0a9763fb52189cf)

   It seems to support:

   60ce752c in git://github.com/uzytkownik/gnome-overlay.git

   How the branches may be more unrelated (or I misunderstood something).

2. Elijah Newren says:[14 October 2008 6:14 PM](#462cca3f8cd5a36b54410d2846be040781442d39)

   I thought 'eg pull REPO BRANCH' did this just fine; I've used it a time or two to merge branches with no common history. Are there problems in less simplistic cases that need your script?

3. Elijah Newren says:[14 October 2008 6:15 PM](#0183918237857c4d5d46602ec2bd786222fd2a9d)

   Um, er, I mean 'git pull REPO BRANCH'. Same thing, though.

4. [Andy Wingo](http://wingolog.org/) says:[14 October 2008 7:28 PM](#f8b6d196803f642093236fa2ab167b65bf08b7f5)

   Well ain't I the fool! Elijah it does appear to work; although I normally use fetch + merge, that seemed to work as well.

   I swear that a few months ago when I first tried this, it did not do the right thing, erroring out when I went to merge. I tried all merge strategies, even. May my crufty script rot in obscurity, then.

5. [Sam Vilain](http://sam.vilain.net/) says:[14 October 2008 9:00 PM](#425b74e207fb11696e6058625017171ab8f2e53e)

   It can support those well actually, just set up a graft and you can fake the history being joined in the past. See Documentation/repository-layout.txt in the git distribution. You can later make it permanent with git-filter-branch.

Comments are closed.
