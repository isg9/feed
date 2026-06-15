---
title: fosdem, presentations, foo — wingolog
url: https://wingolog.org/archives/2007/03/07/fosdem-presentations-foo
published: "2007-03-07T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/03/07/fosdem-presentations-foo
---

# fosdem, presentations, foo — wingolog

## [fosdem, presentations, foo](/archives/2007/03/07/fosdem-presentations-foo)

7 March 2007 2:09 PM

- [music](/tags/music)
- [fosdem](/tags/fosdem)
- [svg](/tags/svg)
- [presentations](/tags/presentations)
- [guile-gnome](/tags/guile-gnome)
- [yoga](/tags/yoga)
- [miriam makeba](/tags/miriam%20makeba)
- [continuations](/tags/continuations)

**post-fosdem**

Allow me to free-associate about [FOSDEM](http://fosdem.org/): packed hallways, rainy sidewalks, buttery croissants... actually about food I could go on for quite a while.

People-wise it was pretty good, although with a pervasive feeling of oddness. The free software community has its perceived social hierarchies, and for many people, these conference events are attempts to transfer those hierarchies to real life. So until someone knows what position that you occupy in their mental pecking order, they treat you with lots of distance, and depending on where they put you, potentially lots of distance afterwards.

The [GNOME-FR](http://www.gnomefr.org/) people, on the other hand, were refreshingly exuberant. Very positive people, the [Vincent](http://www.vuntz.net/) s and [Guillaume](http://cass.no-ip.com/~cassidy/blog/) s and the apparently unlinkable Christophe Fergeaus. Of course there were many other excellent folks there, and also of course, not enough time.

**presentation**

I presented a talk on writing GNOME applications in Scheme. It's available on the interwob as an [incorrectly-rendered PDF](//wingolog.org/pub/guile-gnome-fosdem-2007.pdf), or as a [tarball of SVGs to be presented with inkview](//wingolog.org/pub/guile-gnome-fosdem-2007.pages.tar.gz). I was shocked, I had the title slide up for a good 5 minutes or so, but still about 40 people stayed in the room.

Of course with just the slides, you are missing my charm. For better?

**meta-presentation**

I got loads of wonderful comments on my last [writing product](//wingolog.org/archives/2007/02/19/presentations-and-svg), concerned with the mechanics of presentations. Especially useful was the tip that [inkscape](http://inkscape.org/) comes with a slide-show-like wrapper, inkview. Thanks [Andy](http://andy.brisgeek.com/)!

Also fascinating was Jos Hirth's [XSVG](http://kaioa.com/k/xsvgslide/), rendering SVG inside the browser (probably Firefox-only). Pretty cool, that. Maybe next time.

I should mention that I considered using some LaTeX package, but decided not to because I wanted to be able to easily tweak the presentation. That notwithstanding, several people mentioned [latex-beamer](http://latex-beamer.sourceforge.net/) as being quite good.

In the end I used my ghetto scripts, although the text-to-svg process was more work than it should have been. I looked at doing it with my existing XML processing tools, but it turns out that using functional programming techniques to do text layout is an underexplored area. I'm still poking at that.

**other thangs, delimited by ⁋**

My new year's vow to be more communicative is not going so well.

Interesting paper: [Control Delimiters and Their Hierarchies](http://www.ccs.neu.edu/scheme/pubs/lasc1990-sf.pdf) by Dorai Sitaram and Matthias Felleisen. Summary: "Continuations in Scheme are cool but potentially expensive; we have a way of restricting them slightly but making them cheaper."

The library has rendered unto me an album by Miriam Makeba, which is providing much delight. (It's a collection, although I prefer her early-US stuff.)

I lit a fire on my terrace for the first time this year. Delicious global warming ensued.

My grandmother's not doing so well; I'm going home next week to visit.

I started yoga a little more than a month ago, at an [Iyengar](http://www.bksiyengar.com/) school. It's interesting, and way harder than I thought it would be.

## related articles

- [go put it in a washing machine](/archives/2007/09/10/go-put-it-in-a-washing-machine)
- [fold, xml, presentations](/archives/2007/07/11/fold-xml-presentations)
- [presentations and svg](/archives/2007/02/19/presentations-and-svg)
- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)
- [whippet at fosdem](/archives/2025/02/10/whippet-at-fosdem)
- [sir talks-a-lot](/archives/2023/12/12/sir-talks-a-lot)

### 3 responses

1. [Rodrigo Villarigosa](http://www.turismochile.cl/parques_nac2/delpaine.htm) says:[7 March 2007 10:36 PM](#318)

   My experience is that the instructor makes the class. Sleep around a bit.

2. [Tim Retout](http://blogs.warwick.ac.uk/tretout/) says:[8 March 2007 4:12 AM](#326)

   Andy, your talk at FOSDEM was amazing, you're a legend. :)

3. [random rules: ombliguismo -- andy wingo](http://wingolog.org/archives/2007/04/30/random-rules-ombliguismo) says:[30 April 2007 3:46 AM](#433)

   \[...\] Since FOSDEM my mind has been captivated by a problem similar to that of the SSAX parser, and have just now finished the first draft of a paper. After some comments come in I’ll post it on the interweb for the single digits to download. \[...\]

Comments are closed.
