---
title: hear, memory — wingolog
url: https://wingolog.org/archives/2008/04/08/hear-memory
published: "2008-04-08T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/04/08/hear-memory
---

# hear, memory — wingolog

## [hear, memory](/archives/2008/04/08/hear-memory)

8 April 2008 11:50 PM

- [esmtp](/tags/esmtp)
- [sendmail](/tags/sendmail)
- [nargery](/tags/nargery)
- [dovecot](/tags/dovecot)
- [courier](/tags/courier)
- [gnus](/tags/gnus)
- [nail clippings](/tags/nail%20clippings)
- [nabokov](/tags/nabokov)

A few months ago I wrote about [my mail setup](//wingolog.org/archives/2007/08/03/dealing-with-mail). In summary, on the server side, the MTA delivers to a Maildir, which is then exposed to me via IMAP, which I mirror locally to a Maildir via [offlineimap](http://software.complete.org/offlineimap), and expose locally via a local IMAP server. It's a complicated solution, but on the other hand it is fairly robust, and provides me with distributed backups.

Since writing some minor things changed, and some remained undocumented, so now I write, as [Quim](http://desdeamericaconamor.org/) says, "for your eyes and my memory".

**reading mail**

I switched from Courier IMAP to [Dovecot](http://www.dovecot.org/), basically because Fedora doesn't have courier packages. (I built some, but found about about dovecot before getting to actually configure them.) I've found Dovecot to be much easier to configure. A simple `yum install`, and edits to the well-commented /etc/dovecot.conf, and I had imap working. Here are the three variables that I set:

```
protocols = imap
listen = 127.0.0.1
mail_location = maildir:~/.mail

```

The rest, I left as it came. I suppose I could have figured out some way to run the daemon as my own user, but that doesn't matter much. Dovecot's namespacing is different from Courier's, though -- it doesn't put all folders as subfolders of "INBOX" -- so I did have to tweak my mail filters.

Speaking of which, I now use [Gnus](http://www.gnus.org/) for mail. Gnus is a mail reader that runs on Emacs, started using M-x gnus. The thing is, Emacs has really grown on me, like an infectious disease or something. (Fingernail clippings in a bowl of oatmeal, according to one [L. Wall](http://www.linuxjournal.com/article/2070).) Gnus is really hard to configure, though, and I don't yet use it to its maximum efficacy. Perhaps some bits from my .gnus.el will make it to these (web) pages at some point.

**sending mail**

So that's how I read mail. I send mail using [esmtp](http://esmtp.sourceforge.net/), a simple MTA that can relay to a remote host via secure SMTP, manage a queue, and run as a normal user. It doesn't do local delivery, which makes it quite simple.

Reportedly, esmtp works with gmail as well. I did edit the esmtp-wrapper script to not run the queue when invoked as `sendmail`, so that it would always return 0 instead of failing if network wasn't present (and thus confusing gnus). Messages are queued, of course.

"Hear, memory"

## related articles

- [so you want to build a compositor](/archives/2008/07/26/so-you-want-to-build-a-compositor)
- [allocate, memory (part one of n)](/archives/2008/04/10/allocate-memory-part-one-of-n)
- [fedora is the new ubuntu](/archives/2008/04/07/fedora-is-the-new-ubuntu)
- [dealing with mail](/archives/2007/08/03/dealing-with-mail)

### One response

1. Matthew W. S. Bell says:[9 April 2008 12:55 PM](#1c69caeda90326be832b88e826dfe9e424dcee29)

   Courier is an abomination - no good will come of it.

Comments are closed.
