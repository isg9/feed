---
title: the merry month of ma — wingolog
url: https://wingolog.org/archives/2012/03/12/the-merry-month-of-ma
published: "2012-03-12T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2012/03/12/the-merry-month-of-ma
---

# the merry month of ma — wingolog

## [the merry month of ma](/archives/2012/03/12/the-merry-month-of-ma)

12 March 2012 8:27 PM

- [meta](/tags/meta)
- [tekuti](/tags/tekuti)
- [guile](/tags/guile)
- [scheme](/tags/scheme)
- [http](/tags/http)

**or, from the department of self-inflicted injuries**

Recently I saw a bunch of errors in my server logs. People were asking for pages on my web site, but only if they were newer than `Thu, 08 Ma 2012 22:44:59 GMT`. "Ma"? What kind of a month is that? The internets have so many crazy things.

On further investigation, it seemed this was just a case of garbage in, garbage out; [my intertube was busted](http://git.savannah.gnu.org/gitweb/?p=guile.git;a=commitdiff;h=89d45e850725e232ae685803ee476da5b046c2b0). I was the one returning a Last-Modified with that date. It was invalid, but client software sent it back with the conditional request.

Thinking more on this, though, and on the [well-known last-modified hack](http://nikcub.appspot.com/posts/persistant-and-unblockable-cookies-using-http-headers) in which that field can be used as an unblockable cookie, I think I have to share some blame with the clients again.

So, clients using at least Apple-PubSub/65.28, BottomFeeder/4.1, NetNewsWire, SimplePie, Vienna, and Windows-RSS-Platform/2.0 should ask the people that implement their RSS software to only pass a Last-Modified date if it's really a valid date. Implementors of the NetVibes and worio.com bots should also take a look at their HTTP stacks. I don't guess that there's much that you can do with an etag though, for better or for worse.

[Previously, in a related department.](//wingolog.org/archives/2012/02/21/for-love-and-)

## related articles

- [meta data](/archives/2010/12/13/meta-data)
- [it's probably spam](/archives/2017/03/06/its-probably-spam)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [for love and $](/archives/2012/02/21/for-love-and-)
- [on the new posix](/archives/2010/12/17/on-the-new-posix)
- [metablog](/archives/2008/04/23/metablog)

### 2 responses

1. ijp says:[14 March 2012 8:06 PM](#ccbe073e5dff887624bd01a83704d34a755bc9ef)

   And who could forget the monday bug :P http://git.savannah.gnu.org/gitweb/?p=guile.git;a=commit;h=a24885b27d6c26a5ad2dfeb18eaaaf674853bbfe

2. [wingo](http://wingolog.org/) says:[14 March 2012 9:58 PM](#44d354f2896d9aa307525aacbce4246aedf9013c)

   Indeed, indeed! One wonders if these things will combine at some point: the tuesday-in-november bug, or something.

Comments are closed.
