---
title: photobloggin — wingolog
url: https://wingolog.org/archives/2004/10/17/photobloggin
published: "2004-10-17T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/10/17/photobloggin
---

# photobloggin — wingolog

## [photobloggin](/archives/2004/10/17/photobloggin)

17 October 2004 8:20 PM

- [namibia](/tags/namibia)
- [scheme](/tags/scheme)
- [gnome](/tags/gnome)

### guile-gnome

More work going on here. I haven't had time to blog, but I actually wrapped gnome-vfs before people started talking about it on advogato. Of course the situation is nicer in an interpreted language: anything that would return a GnomeVFSResult will either succeed quietly or throw an exception, as is the case with GError, DBusError, CORBA\_Environment, etc. When you open or create a file, the return value is a normal scheme port, which you can access with normal scheme procedures. Less to remember.

I also wrapped gconf, which was a bit easier, but you still have to define custom type mappings so that things are dynamically typed on the scheme side. GValue has made this problem a lot easier over the whole GNOME stack, but it's not nice to deal with from C, so it's not as prevalent in APIs as it could be.

### photoblogger

![](http://ambient.2y.net/wingo/software/photoblogger/photoblogger.png)

I bought me a cheap digital camera a few months ago, but you wouldn't know that from reading my weblog. It's a pain to post photos, especially when you write offline and come in later -- you have to scp the thing over to the right dir, resize it, get the tag to come out right, etc.

To take a stab at this problem, last week I hacked up a [little app](http://ambient.2y.net/wingo/software/photoblogger/) to help in blogging photos. Admit it, we all need more African eye candy in our lives!

After stumbling across Owen Taylor's [pleasant web site](http://fishsoup.net/), I decided to give his GConf model a try: storing the whole application state in GConf, but not actively responding to changes in the "model". Works out pretty well for me.

### the making of $1

The app is also a test of guile-gnome -- custom cell data funcs, dnd, gnome-vfs, gdk-pixbuf-save-to-port, gconf, guile-gtk-repl, etc. It's the first app I wrote from scratch in guile-gnome in a while. Hacking in a high-level language is a qualitatively different experience, so I figured I'd blog about it.

I started like normal, hacking up something in Glade. From there, I wrote a skeleton script to load up the glade, show it, and drop me in a graphical REPL (or listener), effectively bootstrapping me into the app. From there, I'd hack in Emacs, pasting in the procedures as I defined and redefined them, without stopping the app. I started small, defining procedures to set up the treeview and add photos to it. It's a nice way to work with the treeview, because it's an inherently experimental process. Also, if/when I forget what the name of the procedures are, I can use tab completion in the REPL, query the properties of the classes, etc. The REPL also keeps a history of values and expressions, has paren matching, nice keybindings, catches exceptions with backtraces, etc. so it's easy to work with.

Eventually when the app works you have to refactor things into modules for your sanity's sake, but only at a point when the whole thing mostly works. There's never a point when the whole thing is just broken.

I should be clear, though: running the REPL runs the app. While it waits for your input, it runs the GTK main loop, but once you press enter the main loop drops out and your app is in control again. It's an inversion of inversion of control. That's only in debug mode of course ( `photoblogger --debug`) \-\- when the user runs it normally you can just drop into the gtk main loop. Hack the world, yo.

### Namib life

![](http://ambient.2y.net/wingo/wp-content/kwaito-boy.jpg)

We put on a merit awards ceremony at school recently. Teachers donned academic garb -- ridiculous in this climate to wear full-length black nylon, but hey, suffer for the status symbol.

What else. It's hot. Approaching the too-hot-to-sleep-at-night stage. That's about it, yo -- 59 days left. *Gonna see the folks I dig / I'll even kiss a sunset pig -- California, I'm comin' home.*

## related articles

- [Antelopes](/archives/2004/09/19/the-countdown-begins)
- [schoolnet, guile-gnome](/archives/2004/07/15/schoolnet-guile-gnome)
- [focus](/archives/2004/02/11/advogato-29)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [a schemer at jsconf.eu](/archives/2011/10/04/a-schemer-at-jsconf-eu)
- [poverty, riches](/archives/2009/05/14/poverty-riches)

Comments are closed.
