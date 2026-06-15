---
title: presentations and svg — wingolog
url: https://wingolog.org/archives/2007/02/19/presentations-and-svg
published: "2007-02-19T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/02/19/presentations-and-svg
---

# presentations and svg — wingolog

## [presentations and svg](/archives/2007/02/19/presentations-and-svg)

19 February 2007 7:16 PM

- [scheme](/tags/scheme)
- [guile](/tags/guile)
- [svg](/tags/svg)
- [presentations](/tags/presentations)
- [inkscape](/tags/inkscape)

**50 ways to leave your presentation software**

I've recently had occasion to make a couple of presentations, and have grown intolerant of OpenOffice. Presentations have two immediate purposes: to convey information and to affect people's emotions. OpenOffice gets in the way of both.

OpenOffice facilitates bullet-point-style presentations, in which all slides look the same. I don't think this is so good for memory retention, because the brain quickly gets the pattern ("next slide: more of the same"), and starts to think about other things. At that point, the listener feels that she's finished listening to your presentation, having already drawn her conclusions.

To keep the listener engaged, the presenter needs to be able to affect emotions, something that a bullet-point firing squad won't do. But every time I try to do something mildly different with OpenOffice, it throws up a panoply of dialog boxes, screen-tall context menus, and obscure editing modes to block my way. I just don't understand OO.o's fundamental idea, its structure. I'm left floundering and frustrated, falling back on a bullet-point form I know isn't going to work well.

So off I went on a search for something, anything different. My goal was something that let me control the what my presentation would look like, and have a result that was different from everybody else's work. Maybe that way I could hold an audience's interest for half an hour, that and my charm. Realizing that my charm is not universally acknowledged, I redoubled my search efforts.

**inkscape, yes yes**

While [Inkscape](http://inkscape.org/) is an artist's tool, I've used it in the past for technical drawings, preferring it over the Dias and XFigs of the free software landscape. Inkscape is very understandable and powerful, and also produces quality output. I wanted to use it for presentations.

While I was investigating Inkscape's possibilities, [Jan](http://www.noraisin.net/~jan/diary/) told me about [Rusty Russell](http://ozlabs.org/~rusty/)'s presentation at LCA 2007, done with SVG in firefox, with javascript to move through slides via toggling layers' visibility. Pretty interesting. I hear it wasn't perfect in any sense, though, and I don't feel like re-learning javascript. A PDF of the slides would be better.

After experimenting with various options, I accidentally happened upon Inkscape's XML editor, which showed me that the svg structure is actually rather simple. It's easy to open up an SVG file in your text editor and e.g. remove a layer from it. I decided to write a tool to split a layered SVG into many files, one per layer.

My result is [svg-split](//wingolog.org/pub/svg-split), a script written in Guile, using [guile-lib](http://home.gna.org/guile-lib) (Debian/Ubuntu package: guile-library). The solution was complicated by the presence of XML namespaces; specific comments are in the source.

I was left with the smaller problem of turning a set of SVG files into one PDF. [rsvg-convert](http://librsvg.sourceforge.net/) will do this, but exhibits some rendering bugs regarding text spacing. Because of some problems in rsvg, you have to make inkscape strip XML namespacing from the individual pages via `inkscape -l`. My script to turn a layered SVG to a PDF is [here](//wingolog.org/pub/layeredsvg2pdf).

I also tried Inkscape's PostScript output, combined with `psmerge` from `psutils`, but `psmerge` shrunk the bounding box for each page to the drawing size instead of the page size, which was worse than the rsvg rendering bugs.

For doing the actual presentation, it turns out that Inkscape has a slideshow mode, which allows me to avoid the rendering bugs. You just pass all of the svg's to `inkscape -s` and it runs through them all without any interface. My script to do a presentation with a layered SVG is [here](//wingolog.org/pub/layeredsvgpresent). I got the window to go fullscreen via [devilspie](http://live.gnome.org/DevilsPie), writing the following into `~/.devilspie/inkscape-slideshow.ds`:

```
(if (is (window_name) "Inkscape slideshow")
    (fullscreen))

```

Inkscape's slideshow mode is true ghetto. It responds to all key presses, even alt when you are trying to alt-tab away from it. However it does render correctly.

[Christian](http://blogs.gnome.org/view/uraeus) tells me that the W3C [SVG print](http://www.w3.org/TR/SVGPrint/) specification might solve the svg-to-multipage-pdf problem, but that the free software toolchain does not yet support it.

**conclusions and further work**

My current presentation-making process involves lots of work with pen and paper, ending up with an outline of what all of my slides should do. At that point I type everything into emacs, and start the work of making all of the slides in Inkscape, one layer per slide. I put some alignment lines and hints visible on the bottom layer so that I keep some consistency within the presentation. At the end I run my script on the SVG, which gives me a PDF for posterity; I run `inkscape -s` when actually giving the presentation.

One problem with this process is that Inkscape is not a publishing program. It doesn't understand paragraphs, for example. This is less of a problem than one might think -- presentations shouldn't have too many words -- but it is a minor irritation.

The slides-making process is still too painful. I think that the initial import procedure (from emacs to svg) should be easier; given that the basic SVG I'm generating is simple, I can make the first pass automatically generated from some source. I suspect a custom XML/SXML language would be most appropriate for the kinds of solutions I am interested in. It would be nice if someone with Emacs-fu could hack something to translate outline-mode's format to an s-expression structure.

That idea generalizes as "transforming structured text to SVG", which is valid for many forms of structured text. One interesting possibility would be Tomboy, the GNOME note-taking application. It stores its notes as XML, which can be translated to SVG via some kind of formatting transformer (am currently working up some ideas). You could make your whole presentation in Tomboy, and have a script walk the links in your notes, creating a presentation structure.

I am satisfied enough with the layered-svg solution that I'll continue to poke at it in the future when making presentations. Those of you that want to laugh at me as my strategy crashes and burns should drop by the GNOME room at [FOSDEM](http://fosdem.org/) on Saturday, where you can catch it in person. Comments welcome, and see you at FOSDEM!

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [fold, xml, presentations](/archives/2007/07/11/fold-xml-presentations)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)
- [hoot's wasm toolkit](/archives/2024/05/24/hoots-wasm-toolkit)
- [growing a bootie](/archives/2024/05/22/growing-a-bootie)
- [on hoot, on boot](/archives/2024/05/16/on-hoot-on-boot)

### 17 responses

1. [Özgür Caner](http://www.ocaner.de) says:[19 February 2007 7:45 PM](#286)

   You can use [Batik](http://xmlgraphics.apache.org/batik/) Java SVG Library and [Apache FOP](http://xmlgraphics.apache.org/fop/) for converting SVG Files into PDF. But I don't know if Batik handles multipage SVG Files correctly! Both Projects are under the Apache License.

2. Jeroen says:[19 February 2007 8:21 PM](#287)

   You might want to have a look at Eric meier's 'S5' as well: http://meyerweb.com/eric/tools/s5/

   It's a pure HTML and JS based presentation environment.

3. Jani says:[19 February 2007 8:32 PM](#288)

   I second Jeroen, S5 is very nice, simple to edit and works fine with firefox and the full-screen plugin

4. Patrick Pletscher says:[19 February 2007 8:46 PM](#289)

   If you want to check out an alternative system to prepare your slides:

   http://latex-beamer.sourceforge.net/

   I can really whole-heartily recommend it. Although it's designed for academic presentations, it might be useful for tech talks, too.

5. jorge says:[19 February 2007 8:58 PM](#290)

   A second for beamer. It was easier for me to learn latex and deal with beamer than put up with Impress.

6. hello says:[19 February 2007 10:50 PM](#291)

   +1 bullets must die

   +1 for s5

   \\* emacs outline mode -> basic multilayer svg

   \\* hack on svg in Inkscape

   \\* Save as -> s5 (use svg2xhtml as external filter)

   Exercise for reader: implement svg2xhtml (use xslt?)

   World domination via audience attention!

7. Christian says:[20 February 2007 1:54 AM](#292)

   I like your approach. I would recommend using Emacs org-mode (which is basically outline-mode on speed). Org-mode will let you export your outline as XOXO (see microformats.org). XOXO is XML, so you can transform it to svg if you like.

   Your post made me want to come to FOSDEM :-), but alas can't make it.

8. tobias says:[20 February 2007 3:17 AM](#293)

   Have a look at this:

   http://kaioa.com/k/xsvgslide/

9. Frantisek says:[20 February 2007 4:25 AM](#294)

   Two links worth checking out:

   http://www.sethgodin.com/freeprize/reallybad-1.pdf

   http://www.presentationzen.com/

   They're not about presentation software, but about presentation themselves. I can't recommend them hard enough :)

10. Jos Hirth says:[20 February 2007 4:45 AM](#295)

    Thanks for the linkage, Tobias.

    The demo above was a hand-edited proof of concept thingy (the fifth iteration, I think).

    There is also another demo, which was created via some python extension:

    http://kaioa.com/k/xsvgslide/new.xmlz

    (Note: some of the font stuff doesn't look as intended, because I didn't convert em to paths)

    You can also link to specific slides like:

    http://kaioa.com/k/xsvgslide/new.xmlz#6

    Well, it works, but there are lots of issues with that extension. The biggest one is thats some ugly hard to read SAX hack. And there were a bunch of other silly decisions... oh well.

    I'm working on some proper version, which I initially wanted to get into the 0.45 release, but I didn't make it in time.

    The extension basically works like this:

    -each layer = 1 slide

    -layer title = slide title (eg the title for the linked slide example is "\[6/6\] YOU!")

    -images are embedded automatically

    -clones across layers etc work (something which was broken with the initial extension... ooops)

    And that's it. Can't get any easier than that, I guess.

11. Munchkinguy says:[20 February 2007 8:00 AM](#296)

    Also, after you have made a PDF file for presentation, you can use KeyJnote (http://keyjnote.sourceforge.net/) to present it with flair (ie alpha-blended transitions, 3d-page turns, etc...)

12. Tiffany says:[21 February 2007 3:39 AM](#297)

    nobody here speaks english!

    aikido was good tonight. are you going tomorrow? i'm planning to go every day this week. actually, i'm planning to have it in the back of my head that aikido is on the schedule every day. yay!

    also i quit drinkin. sort of. weekends don't count. yay!

    you need my new harper's. but i need it back when you're done.

13. [Andy Fitzsimon](http://andy.brisgeek.com) says:[21 February 2007 8:50 AM](#298)

    Just thought id quickly mention what shouted to rusty: inkview has been included with inkscape for a long time now.

14. [fosdem, presentations, foo -- andy wingo](http://wingolog.org/archives/2007/03/07/fosdem-presentations-foo) says:[7 March 2007 3:17 AM](#311)

    \[...\] I got loads of wonderful comments on my last writing product, concerned with the mechanics of presentations. Especially useful was the tip that inkscape comes with a slide-show-like wrapper, inkview. Thanks Andy! \[...\]

15. [fold, xml, presentations -- andy wingo](http://wingolog.org/archives/2007/07/11/fold-xml-presentations) says:[11 July 2007 4:56 PM](#874)

    \[...\] Of course, the next time I gave a presentation, I would have to figure out how to use the algorithms that I had worked on, otherwise it would be wasted effort. The occasion was the Jornadas del Programari Lliure last week in Girona. Following a recommendation by a kind commenter, I decided to write the presentation in Emacs’ Org Mode, then transform its XML export into the presentation language that I defined, then use the algorithms that I presented to transform that into SVG. After the presentation, I cleaned up the scripts I had, and packaged them into a library that I just released, Guile-Present. \[...\]

16. Frederik says:[1 December 2007 3:47 PM](#3225)

    There is another great tool for making SVG slideshows that makes use of SVG's animation capabilities to add nice effects to the slideshow:

    http://www.hoylen.com/products/jacksvg/

    The problem is that currently there are no SVG viewers for Linux that are advanced enough to display these files correctly. So you have to rely on ASV for Windows. Opera is quite close when it comes to rendering the animations, but there are still some issues with the fonts.

    It would be great to see OpenSource tools to be capable of displaying these files. You might see it as a test case for what is possible with SVG once its more advanced features are implemented.

17. Robert Castañas says:[17 November 2011 7:35 PM](#3ad1c545d2058df7c33570581c7ea974797eafed)

    Yo Dude,

    You are everywhere! Just a heads up, there is now an SVG presentation editor that runs right in Inkscape called SOZI.

    http://sozi.baierouge.fr/wiki/sozi

    It is working quite well for me right now. The problem is, I want to view the presentation in Firefox and Firefox is slow as bones, so now I'm looking for ways to make the presentation better at scrolling.

Comments are closed.
