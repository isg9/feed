---
title: XAML Schmaml — wingolog
url: https://wingolog.org/archives/2004/08/29/xaml-schmaml
published: "2004-08-29T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/08/29/xaml-schmaml
---

# XAML Schmaml — wingolog

## [XAML Schmaml](/archives/2004/08/29/xaml-schmaml)

29 August 2004 8:07 PM

- [scheme](/tags/scheme)
- [gnome](/tags/gnome)

### Meditations on XAML

I've been thinking a bit about XAML recently. Microsoft [says that](http://longhorn.msdn.microsoft.com/lhsdk/core/overviews/about%20xaml.aspx) XAML brings graphical programming to the next level by expressing it in a declarative language, XML.

Consider that many more people have made HTML forms than have made GUI apps with native libraries. Part of the attraction of HTML is that it's "just text". If you want something special, like a button, you just put it in. However, few graphical applications are like that.

In "applications", you normally put text explicitly within labels, and those labels within different alignment containers, etc. Text does not stand on its own. You construct a mental model of the structure of the GUI, and then fill it with things.

This leads to the second problem: in XAML, you need to specify every part of the GUI. This is akin to writing libglade files by hand; in theory it's possible, but noone does it. But if you treat XAML files as the compiled output from some graphical language (as most GNOME developers do, with Glade), the advancement isn't near so much. It's not glorified HTML forms anymore, it's just a language-independent GUI specification.

The need to specify each GUI element also poses interesting abstraction problems. Let me give some examples. When you write Push Me, with a suitable default namespace, an instance of the Button class is constructed to correspond with this element in the DOM tree. These classes are very generic, so that many apps can use them.

However, consider the somewhat-contrived example of a list of documentable modules. I might feel that the GUI pattern of documenting a module (a bold label like in a

, with text describing the module, then a link to the source code) could be encapsulated by a QuuxModule element. There would be two ways to do this. First, you might think of using something like XSLT to transform the higher-level element into its primitives, but then to do anything based on those primitives you'd have to break that abstraction barrier, for instance to access the link element.

Otherwise, you would define QuuxModule from within your own code, probably in C#. Then I think you'd have to give a different namespace, such that would look to your own code rather than to Microsoft's Avalon classes.

I guess this second method is OK. Obviously XSLT is out of place, though. And anyone who hopes of automatic transformation between XUL and XAML via XSLT is a hopeless moron. The connection between the imperative, controlling code (e.g. C#) and the declarative "GUI document" is very tight, and therefore brittle. Change the document, fine, but you can't change the code. Not in fixnum days!

Clearly, XAML has something. But it seems very much a research project: usable, but not polished. When I make GUIs with Gtk+, I feel like I'm doing something the compiler should be doing: g\_object\_new, gtk\_container\_add, gtk\_box\_pack\_start, etc. Like any good lisper, I don't believe in design patterns. If I'm doing something repetitive, it means I'm programming in too low-level a language, and there's an abstraction waiting to be written.

Declaring these parentage relationships via a structured declarative language is attractive. However, integration between declarative and imperative programming is difficult. You end up constructing a DOM so that you can change the document. There is still a language disconnect between what talks to the DOM (C#, JavaScript, ..) and what makes the DOM (XML).

Making a declarative, symbolic GUI language is easy in Lisp. The problem is that the language is not the GUI -- you need another parallel representation to change the GUI after it was made. (This is analogous to GladeXML v/s GObject.)

I'm thinking out loud a bit, pondering what a Schmaml would look like -- XAML in Scheme, for local GUI construction. Stay tuned.

### Rich client languages

Local application construction is not the primary target of XAML, though.

Regardless of the technological merits and flaws of XAML, the time has come for a rich client library. People use the web as they would an application. The time of JavaScript rollover/preload hacks has stretched on way too long.

The only thing XAML has going for it is its potential install base, along with the CLR. Microsoft got it right when they chose to put a general-purpose VM into clients. Technologically, though, XAML/.NET is just Java without the applet boundary -- a language for programmers to write and people to run on their machines. It's not an authoring language.

XUL is equivalent, as far as I can tell. However, it sorely lacks a general-purpose VM. This could be fixed by making CLR or Java bindings.

The time for innovation is past, I think. When the rich client library comes along, it will be based on one of the major VMs: .NET or Java. The question is only what graphical language is supported: Avalon, XUL, Swing/AWT, or something similar to Avalon but not made yet. The problems we have are no longer technical; they are organizational.

### Prospects for incremental change

That the web would standardize on a new runtime is scary for free software, because we don't control either one of the frontrunners. On the other hand, the current situation does have its advantages: HTML forms a platform barrier, between whatever it is on the server and whatever it is on the client. Such is the nature of a purely declarative language, and such is the danger of .NET, that the declarative would be intermingled with interfaces to Microsoft libraries. The platform barrier could break down, demanding that both sides run Microsoft.

That's a long shot, to be sure. But it could happen in specialized domains. We could avoid the whole situation if we could [fix whatever it is that's wrong with HTML](http://whatwg.org/).

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)
- [brought to you by torres viña sol](/archives/2007/11/10/brought-to-you-by-torres-vina-sol)
- [happy birthday gnome!](/archives/2007/08/14/happy-birthday-gnome)
- [documenting language bindings](/archives/2007/07/18/documenting-language-bindings)
- [foreign interfaces](/archives/2007/05/24/foreign-interfaces)

Comments are closed.
