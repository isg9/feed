---
title: hackings — wingolog
url: https://wingolog.org/archives/2008/01/19/hackings
published: "2008-01-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/01/19/hackings
---

# hackings — wingolog

## [hackings](/archives/2008/01/19/hackings)

20 January 2008 4:13 AM

- [gnome](/tags/gnome)
- [guile](/tags/guile)
- [linux](/tags/linux)
- [el país](/tags/el%20pa%C3%ADs)
- [john taylor gatto](/tags/john%20taylor%20gatto)

I've been hacking a bit recently, and meaning to write about it but keep getting backlogged.

I upgraded my linux kernel to the latest git when I was in the states. I was pleased to see someone pushed in the powerbook capslock-mangling patch, although with (ahem) some needed [fixes](http://git.kernel.org/?p=linux/kernel/git/torvalds/linux-2.6.git;a=commit;h=9f31c05ea0f5690d002ae30710fc0fbe0f0c201f). I gave [nouveau](http://nouveau.freedesktop.org/) another shot, and it actually worked. Unfortunately it required me not to use `nvidiafb`, leaving me without backlight control, so I wrote a [module to control backlight on nvidia cards](http://lkml.org/lkml/2008/1/12/170), which works fine for me.

That's me tooting my own horn. Sorry about that! If there is any lesson out of this it's that hacking the kernel is for any programmer; it's just another piece of software. If I can do it, it's not special :)

Other than that, I haven't been hacking that much. I made a [guile-gnome](http://www.gnu.org/software/guile-gnome/) release last month. Did you know that emacs could automatically give you [documentation for the symbol at point](http://thread.gmane.org/gmane.lisp.guile.user/6296)? This pleases me.

**roots**

One of my flatmates works the night shift at a hotel in the center. Every morning I wake up to see that he has brought home the day's copy of [El País](http://elpais.com/) from work. So often in the bar where I go to hack I find myself reading the paper instead of pounding parentheses.

Anyway, the "Fourth Page" editorial from yesterday, [*Iquique, un siglo del drama*](http://www.elpais.com/articulo/opinion/Iquique/siglo/drama/elpepiopi/20080118elpepiopi_12/Tes) was very moving, discussing the massacre of hundreds of peacefully protesting saltpeter workers in the northern Chilean city of Iquique. A kind of southern [Haymarket](http://dwardmac.pitzer.edu/ANARCHIST_ARCHIVES/haymarket/augustspies.html).

I've been bathing myself in the radical media in recent days; realizing the extent of the inhumanity of the current system invites cynicism. I've become very convinced that the complacent obedience exhibited by most adults is actually a state that was inflicted on them by schooling. The award-winning teacher [John Taylor Gatto](http://www.johntaylorgatto.com/) reaches the same conclusion, expressed in his resignation essay, [I Quit, I Think](http://www.johntaylorgatto.com/underground/prologue2.htm). Worthy reading.

## related articles

- [to guadec!](/archives/2011/08/03/to-guadec)
- [git and bzr](/archives/2009/01/06/git-and-bzr)
- [stable guile-gnome released](/archives/2008/07/03/stable-guile-gnome-released)
- [spamming the world into submission](/archives/2008/04/25/spamming-the-world-into-submission)
- [brought to you by torres viña sol](/archives/2007/11/10/brought-to-you-by-torres-vina-sol)
- [random things](/archives/2007/11/08/random-things)

### One response

1. [Germán Poó-Caamaño](http://www.calcifer.org/) says:[20 January 2008 6:25 AM](#3792)

   Andy,

   The article is not acurate in a very special detail. In the school, there were families, not only employees. And there were are not disturbs (like any big riot), they were in the school.

   Unfortunately, nearby where I live, there is a regiment whose name is "Sylva Renard" in honor to such General (probably not because of that act in particular).

Comments are closed.
