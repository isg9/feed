---
title: 'research!rsc: I Could Tell You, But ...'
url: https://research.swtch.com/secret
published: "2008-01-21T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/secret
---

# research!rsc: I Could Tell You, But ...

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# I Could Tell You, But ... Russ Cox January 21, 2008 *research.swtch.com/secret* Posted on Monday, January 21, 2008.

For two centuries, Benjamin Franklin had the final word on sharing secrets: “ [Three may keep a Secret, if two of them are dead.](http://en.wikiquote.org/wiki/Benjamin_Franklin)”

Shamir's 1979 paper “ **[How to share a secret](http://www.caip.rutgers.edu/%7Evirajb/readinglist/shamirturing.pdf)**” is a significant advance over Franklin's algorithm. It shows how to split a secret into *n* pieces such that any *k* pieces can be used to reconstruct the secret but *k*–1 cannot.

Best of all, the idea is dead simple. Assume the secret *s* is a number smaller than some prime *p*. Then pick random integer coefficients to create a degree- *k*–1 polynomial *f*( *x*) computed mod *p*. Set the *x* 0 coefficient to the secret message *s*, so that *f*(0) = *s*. Then hand out the pairs (1, *f*(1)), (2, *f*(2)), ..., ( *n*, *f*( *n*)) as the secret shares. Since any polynomial of degree *k*–1 is uniquely determined by *k* points, any *k* shares can be used to reconstruct the original polynomial and then compute *f*(0). But each set of *k*–1 shares is consistent with *p* different possible polynomials, each with a different *f*(0) value, and thus leak no information at all.

Shamir's idea is not quite as simple as Franklin's, but I'm confident [Franklin would have understood it](http://press.princeton.edu/titles/8478.html).

Martin Tompa and Heather Woll explain how to detect participants that lie about their shares in “ [How to Share a Secret with Cheaters](http://dsns.csie.nctu.edu.tw/research/crypto/HTML/PDF/C86/261.PDF).”

(Comments originally posted via Blogger.)

- [Ralph Corderoy](http://www.blogger.com/profile/13140975971019765573)(July 26, 2010 7:43 AM) Wikipedia has a little [example](http://en.wikipedia.org/wiki/Shamir%27s_Secret_Sharing#Example), and package [ssss](http://point-at-infinity.org/ssss/) provides one implementation for Debian and Ubuntu.
