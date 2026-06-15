---
title: graphical repl, one year moAfrica — wingolog
url: https://wingolog.org/archives/2003/10/25/advogato-21
published: "2003-10-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2003/10/25/advogato-21
---

# graphical repl, one year moAfrica — wingolog

## [graphical repl, one year moAfrica](/archives/2003/10/25/advogato-21)

25 October 2003 6:28 AM

- [namibia](/tags/namibia)
- [scheme](/tags/scheme)

**guile-gobject**

Cleaned up the class hierarchy of the GOOPS objects. I think it might be possible to make

```
(define-class  ())

```

actually subclass the in the GType system. You can subclass now, of course, but the syntax isn't really schemey.

So now interfaces are in the class hierarchy, which is good. Some more fixes and custom wrappings later, and I have a working graphical REPL! I'm pleased. The GtkEntry turns into an input port, the GtkTextBuffer turns into an output port, and the repl just reads from and writes to those ports. Rather elegant. I like Scheme.

There's an interesting summary of the issues involved in an [old email by Neil Jerram](http://mail.gnu.org/archive/html/guile-user/2001-04/msg00033.html). Have a read, it's thought-provoking.

**soundscrape**

The REPL is a big part of the (planned) graphical interface to soundscrape. The other big part is a source code editor. I think I'll patch GtkSourceView to know about scheme, and then wrap it.

Man, I really need to release. I'll finish the envelope-generator and event spawner and release, as soon as guile-gobject can make a release as a GNU project.

**teaching**

I'm alternately shocked at how clever grade 9 is and how... slow grade 8 is. Check out this question: "Mr. Wingo. If baking soda reacts with acids to make carbon dioxide, how come when I eat a cake my stomach doesn't get big?" Wow. Those kids (grade 9) understand. (Answer: When carbonates are heated they release the carbon dioxide, which is why we put Na2HCO3 in cake dough.) But then grade 8 doesn't know the words "depart" and "arrive" and can't read timetables. How does grade 8 change into grade 9? I'm baffled.

**life**

One year in Africa on Monday. Life is nice. We had our first rains of the season on Thursday. High winds, etc -- I still haven't cleaned the house, it's all coated in dust. Unfortunately the wireless tower in town was somehow damaged, so internet is out at the school. Ah well.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [poverty, riches](/archives/2009/05/14/poverty-riches)
- [culture and code](/archives/2004/12/06/culture-and-code)
- [cromulation](/archives/2004/11/24/cromulation)
- [photobloggin](/archives/2004/10/17/photobloggin)
- [Antelopes](/archives/2004/09/19/the-countdown-begins)

Comments are closed.
