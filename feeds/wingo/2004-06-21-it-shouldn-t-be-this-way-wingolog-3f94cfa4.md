---
title: it shouldn't be this way — wingolog
url: https://wingolog.org/archives/2004/06/21/it-shouldnt-be-this-way
published: "2004-06-21T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/06/21/it-shouldnt-be-this-way
---

# it shouldn't be this way — wingolog

## [it shouldn't be this way](/archives/2004/06/21/it-shouldnt-be-this-way)

21 June 2004 2:50 PM

- [namibia](/tags/namibia)
- [computers](/tags/computers)

**printing hell**

Today I worked to make a printer that a local school bought work with linux, specifically SuSE 7.3 (yeah, we're a bit behind). It was ridiculous, and it's totally not the fault of the printer manufacturer.

The printer is a Samsung ML-1210, a 600 dpi laser printer that's specifically advertised to work with linux. Although you'll get a 404 if you try to find the drivers on the manufacturer's web site, Samsung did release it under the GPL, so it's also archived and patched elsewhere. But [look at the page I was pointed to!](http://linuxprinting.org/download/printing/samsung-gdi/)

After a while, I realized that **I had to recompile ghostscript**. I couldn't believe it. I can't even get a source RPM for 7.3 these days. You have to **manually patch makefiles**, and with the GNU ghostscript that I downloaded, that's not straightforward process (the directions were wrong).

But let's step back a bit: **why the fuck should I have to recompile something to install a printer?** Truly boggles the mind, that one.

Anyway, I finally get it working, manually setting the `--prefix` so it overwrites the previous installation. Then it turns out SuSE uses some arcane setup for their printing that mandates YaST usage. Let's get it straight: that thing sucks, [no matter what Nat Friedman says](http://www.ofb.biz/modules.php?name=News&file=article&sid=307). But even after fixing Ghostscript, it wouldn't show up in the list of Ghostscript printers!

To cut a long story short, I had to manually edit a strangely-formatted YaST printer database. Why they maintain a list of Ghostscript printers outside of Ghostscript itself, I don't know. But it works now. They're happy. And I'm happy too -- I wasn't expecting to be paid, but I couldn't really refuse at the end, this being on my own time and all.

**google**

I see no reason why I shouldn't be the number one wingo on the web. I'm going to start working on making this site relevant, starting with [the software page](/wingo/software/). In the meantime I just need to regain lost ground. You can help by linking to the main page, `http://ambient.2y.net/wingo/`. Thanks.

## related articles

- [computerisation](/archives/2005/05/26/computerization)
- [vaquero](/archives/2005/04/20/vaquero)
- [channeling satan](/archives/2004/05/08/8-may-2004)
- [pulling knuthes](/archives/2004/04/29/29-apr-2004)
- [africasource postmortem](/archives/2004/03/27/advogato-35)
- [africasource](/archives/2004/03/17/advogato-34)

Comments are closed.
