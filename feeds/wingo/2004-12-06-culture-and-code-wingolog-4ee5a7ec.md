---
title: culture and code — wingolog
url: https://wingolog.org/archives/2004/12/06/culture-and-code
published: "2004-12-06T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2004/12/06/culture-and-code
---

# culture and code — wingolog

## [culture and code](/archives/2004/12/06/culture-and-code)

6 December 2004 12:03 PM

- [namibia](/tags/namibia)
- [scheme](/tags/scheme)

A bit of culture first, then about code.

**evening**

I didn't do anything I intended to do this evening, which was nice. I was going to the bars in the village to fraternize with folk after quaffing a bit (to lubricate the tongue), but got roped into going to the pastor's house to help his wife edit a job application. It was pleasant, actually, and I got *oshikundu*, a fine beverage made from millet, which they don't seem to make at my house.

Then I headed to a certain bar, but got waylaid before that -- ended up somewhere else just gabbing with various characters. After a few beverages, tried to go home, but ended up at another bar called *Don't Destoried My Friends' Please For Sure !!* Long chats at sunset, which get more beautiful by the day. Today was one of those low cloud cover days, cool and breezy, but with a space for the sun to light up the sky from underneath. Then, with the fading light I did head home, savannah silhouette on the horizon, black palm trees, black ground, yellow sky, the vastness of it all fading into stars.

**dancing**

After eating porridge and meat with the family, we sat around watching the one channel that comes in on TV. Some program about traditional dances of the various peoples here in Namibia. It's wild -- these people are programmed, imbibed, steeped in rhythm and dance. It's not just a stereotype. There are a few families I know back home that are like this, mostly from the Appalachains, but few they are. I envy them.

The Hereros are funny I think. The women adopted their dresses from the German colonial wives in the 1880s and the men even adopted the old soldier uniforms. Culture lives though, in the headpieces worn by the women, which resemble the horns of a cow, an almost sacred animal to them. The dance I saw involved them imitating the bobbing and swishing of cattle heads, ending in an impaling of some invisible creature. Near holiness does not, however, impede Hereros from enjoying beef.

The Owambos, the people where I am, have the strangest dances I'd say. Often the beat is odd, literally: the one on TV tonight was in 7/4. Two halfs and three quarters, maybe at 90 bpm or so. When I see them dance, I see the shape of old knarled trees, whose roots belie their motion: moving with wind, but also changing shape over time. So their flailing arms and legs appear to me. I was out in the deep bush a couple of weekends ago and played one of my Oshiwambo songs for some folks at a bar. They broke out into a clapping circle, 65-year-old women singing and dancing, limbs flailing in the full moonlight.

That is one flavor of Owambo dance, unique to them. The other is more common to the various groups, and is overtly, almost terribly sexual. People here can move their hips in all directions. To say hips will limit your thoughts though. "Pelvis" might be more apt. These dances are performed only by the young, from about 5 up to 25. Sometimes you can catch older people doing it, but only if small people are doing it to, and the moment has to be right. It's deemed improper.

(Before I continue, I should note that yes, I do like it.)

I'm not sure, but get the idea that pre-puberty kids don't think anything sexual about the dances. Probably for that reason, it's more acceptable for them to do it. For instance, at my school, which is grades 8-10, you won't find but a couple people willing to dance like that in public. But imagine, the Kwangali people had "R.S.S." tatooed on the back of their skirts: "Rundu Senior Secondary". The announcer noted that "like a lot of African dance styles, this one mostly involves shaking of the hips. Some of you might get some ideas, but remember, this is strictly for entertainment!"

There's some kind of contradiction here. My American mind sees something wrong with allowing sexuality to kids, but the Owambo mind thinks pelvic gyrations are only for the young. But I envy them too, with their inborn rhythm, and I loath the missionaries that shoved listless Lutheran church tunes into mouths that once called and responded in 7/4. Yet, the incidence of rape is phenomenal in this country, presumably the burn of the gift of sexuality to children. It is acceptable for old people to look sexually at grade-school girls shaking polaroids on TV. Here it is the wo-man chained by the gift, not Prometheus the giver.

Coda: There's a girl on my homestead these days that is astonishingly beautiful. She scares me though, because I don't want her to die. I know that when I come back in however many years, many people I know will be sleeping underground. *AIDS: 20-30% of sexually active adults served.*

**code**

Worked on some of the lower-level wrappers in [guile-gnome](http://home.gna.org/guile-gnome/). Today, the focus was on custom wrappings for boxed structures, like GdkRectangle and GdkColor. Types like these have more useful Scheme representations than opaque objects, for instance as vectors or as strings (in the case of GdkColor).

However, these representations were only being used for wrapping arguments and return values, not general GValue marshalling (as in GObject properties and signal closures). After writing the low-level code to get this to work, it took surprisingly long to get the bindings generator to do what I needed. The final interface looks like this (wordpress is butchering the code tag though): `(wrap-custom-boxed! "GdkRectangle" "GDK_TYPE_RECTANGLE" (list "if (" c-var ")" scm-var " = scm_gdk_rectangle_to_scm (" c-var ");" "else " scm-var " = SCM_BOOL_F;") (list c-var " = scm_scm_to_gdk_rectangle (" scm-var ");"))`

The first and second args are the type name and the GType ID. The third and fourth are code fragments to translate between C and Scheme values. The fragments are made of strings and lists of strings that should be concatenated. The interesting thing is that the `c-var` and `scm-var` bindings aren't bound by the proc, which indicates that `wrap-custom-boxed!` is a macro, but also that they can't be known when this form is evaluated -- only when the bindings are actually written out. (This code might be expanded inline into a complicated function, like one with a `GList*-of-const-gchar*` argument, for instance.)

So, the macro wraps the forms in closures, which will later be called with `scm-var` and `c-var` as arguments. Then when the closures are executed, the proper names go in. The technique of lambda-wrapping is reminiscent of Guy Steele's hygienic macro implementation in his [Rabbit scheme compiler](ftp://publications.ai.mit.edu/ai-publications/pdf/AITR-474.pdf). (I wonder what problem the theoreticians had with his implementation -- it was really elegant.)

The list notation for code fragments has been in [g-wrap](http://home.gna.org/g-wrap/) for a long time. It's ugly to work in two languages, though; it would be nice if we could implement a straightforward mapping between pseudo-scheme and C, like: `'(if (% c-var) (set! (% scm-var) (scm_gdk_rectangle_to_scm (% c-var))) (set! (% scm-var) #f))`

In this case a list with % in the first arg would indicate a substitutable parameter. A bit trickier would be: `'(set! (% scm-var) (if (% c-var) (scm_gdk_rectangle_to_scm (% c-var)) #f))`

In the latter case the `if` maps to the terniary operator, whereas in the first it maps to C's `if`. And there's a lot of C that Scheme doesn't have -- `.` and ->, bit shifts, infix arithmetic, casts, types, prototypes, etc. But a lot of it maps nicely, and with a way to escape down into text output we should be able to limp along with an incomplete implementation.

**press**

[λgtk](http://common-lisp.net/project/lambda-gtk/) got a disturbing amount of blog-press, especially considering that it is really immature right now. (Uses the C closure interface, no subclassing, probably no object properties, etc.) I am jealous, yes! Someone should get a planet scheme going. I have a low probability of doing so, but given enough time I don't need 10,000 monkeys, I'll get around to it.

10,000 monkeys should have been Ximian's motto.

**village**

**The count:** 2 days to leaving the village, 10 to landing in san francisco. I'm packing up the computer today -- no more hacking for a while.

## related articles

- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [poverty, riches](/archives/2009/05/14/poverty-riches)
- [cromulation](/archives/2004/11/24/cromulation)
- [photobloggin](/archives/2004/10/17/photobloggin)
- [Antelopes](/archives/2004/09/19/the-countdown-begins)
- [Precipitation](/archives/2004/08/31/precipitation)

Comments are closed.
