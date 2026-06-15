---
title: dealing with mail — wingolog
url: https://wingolog.org/archives/2007/08/03/dealing-with-mail
published: "2007-08-03T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2007/08/03/dealing-with-mail
---

# dealing with mail — wingolog

## [dealing with mail](/archives/2007/08/03/dealing-with-mail)

3 August 2007 4:23 PM

- [mail](/tags/mail)
- [offlineimap](/tags/offlineimap)
- [imap](/tags/imap)
- [maildir](/tags/maildir)
- [courier](/tags/courier)

For most of this year there has been a part of my mind nagging me to deal with my email inbox, which now staggers above the thousand-message mark, after filtering. As it is easier to blame tools than to blame myself, I decided to switch away from my old system and start trying a new one. Here's a writeup of what I'm doing now, including a nifty patch.

**the basics**

I do not believe that there is a sane email solution that does not involve [IMAP](http://tools.ietf.org/html/rfc3501), [Maildir](http://en.wikipedia.org/wiki/Maildir), and [offlineimap](http://software.complete.org/offlineimap). IMAP, so that you can read the same mail from multiple machines; Maildir, so that there is no chance of corrupting your mail; and offlineimap, so that you can (1) access your IMAP mail without being connected to the network and (2) distribute backups of your mail across multiple machines.

I am willing to concede that GMail might work for you, although they do not yet have an offline solution. Apart from Google, on a technical level I would not trust any other organization to keep backups for me. So, if you are not willing to use GMail (as I am not), backing up mail via multiple maildirs on multiple machines with IMAP+offlineimap is the only solution. If my server dies, the copy of the mail I have on my machine is just as valid as the one that was on the server.

The same cannot be said for any other offline mail solution that I know of.

**the user agent**

I have been using [Evolution](http://www.gnome.org/projects/evolution/) to read mail. I'm in the process of a switch, but will write about that later. However I found that Evo didn't deal well with offlineimap changing the maildirs that it was trying to read. This was a while back; perhaps things have changed since then.

It turns out that most user agents (mail readers) don't deal well with multiple folders of maildir mail. Either they get the folder nesting wrong, take forever to read them, have problems keeping their indices in sync with the data, or simply don't support maildirs at all.

However, most of them do support IMAP well. The solution? Run a local IMAP server! Inspired by [a response](http://groups.google.com/group/linux.debian.user/msg/c3d85e9971baf659) to [this most excellent mail rant](http://groups.google.com/group/linux.debian.user/browse_thread/thread/cda937fea7196169/61956bef5bffd718%2361956bef5bffd718), I'm running a local [Courier IMAP](http://www.courier-mta.org/imap/) server, which I then use to access my mail. It's fast and robust.

Now it's easy to read and filter mail offline. You have to set up offlineimap to name its local maildirs like Courier expects, but there is an example of that in the default offlineimap configuration file.

The problem comes in if you want to create new folders locally. Courier will happily and correctly create the new maildirs, but offlineimap will not sync them to the server. This has bothered me since a few years ago, but today I sat down and decided to fix it. Two hours later, I have a patch that [Works For Me But Might Eat Your Mail](http://software.complete.org/offlineimap/ticket/13). Back up before trying it!

I'm still getting used to my new MUA, but when all that is done, there will be more blog ranting. Until then, enjoy the summer sun!

## related articles

- [hear, memory](/archives/2008/04/08/hear-memory)
- [mayday, mayday](/archives/2007/05/01/mayday-mayday)

### 4 responses

1. [Adam Williamson](http://www.happyassassin.net) says:[3 August 2007 9:06 PM](#994)

   The other good thing about running your own IMAP server is it then makes it easy to run your own webmail server, for the times when webmail actually would come in handy (you're using a public terminal with only a browser, or you're on a friend's Windows machine, you just want to check your mail quickly, and you don't feel like fighting with Outlook Express). I'd recommend Roundcube Mail - http://www.roundcube.net . It's an excellent AJAX-y webmail system that's pretty easy to set up and use. Their releasing foo sucks (it's usually best just to go with latest SVN), but other than that it's a great project.

2. nona says:[4 August 2007 1:49 AM](#996)

   Can you explain me why using offlineimap is better than rsync'ing or using unison to transfer the Maildir?

3. [Ricky Youngblood](http://www.macgregor26.com) says:[4 August 2007 6:01 AM](#998)

   Fortunately there is a growing cadre of women that is turned on by this sort of talk. I read wingolog to stay hip to the latest terminology. My dwindling supply of kimono ultrathins is a testament to the efficacy of this modality. It doesn't hurt that I am French.

4. [Brian Gough](http://www.briangough.ukfsn.org/) says:[7 August 2007 1:56 AM](#1009)

   I am using Emacs Wanderlust. It has good disconnected IMAP support. I chose it over offlineimap as it is pure elisp so I can run the same emacs configuration everywhere, including on my Zaurus. It also works well over GPRS, as there is selective header/mime-part downloading. The Wanderlust webpage is a bit dated, but the mailing list is still active. For anyone who is happy to read all their mail in emacs it is worth a look.

Comments are closed.
