---
title: I'm telling you — wingolog
url: https://wingolog.org/archives/2008/02/29/im-telling-you
published: "2008-02-29T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2008/02/29/im-telling-you
---

# I'm telling you — wingolog

## [I'm telling you](/archives/2008/02/29/im-telling-you)

29 February 2008 5:18 PM

- [scheme](/tags/scheme)
- [meta](/tags/meta)
- [guile](/tags/guile)
- [git](/tags/git)

A little while ago I got a mail from my good friend Leif, on whose laurels or electrons I have been resting, freeloading [wingolog.org](//wingolog.org/) on his colocated server. Perhaps this was an acceptable arrangement when I was teaching in Namibia, earning five dollars a day, but now that I earn my keep programming for the man it is a less tenable position.

In any case, the mail went like this: "Wingo, find a new hosting arrangement." Or something to that effect. So I bought a [slice](http://slicehost.com/) and started to migrate my wingolog.org services there.

It is at this point in the story that I experienced a bit of revulsion at the thought of migrating my blog product, powered by [Wordpress](http://wordpress.org/), to the new server. Wordpress is (1) plagued with [security vulnerabilities](http://lwn.net/Articles/270566/), and (2) written in PHP, neither of which I was excited to have on my fresh clean slice.

Hence, the bad idea: what if I wrote my own weblog software, so that I wouldn't have to install PHP?

Then, the idea whose worth I have not yet fully evaluated: what if I wrote said weblog software in Scheme?

Then, constraints result in awesomeness: I considered how I could use a database from my usual Scheme implementation, [Guile](http://www.gnu.org/software/guile/). There are no standard database bindings for Guile, nothing like Perl's DBI or Python's DB-API, and the only binding that I knew of, for PostGres, was bitrotten. (Incidentally the PG binding has been updated within the last couple weeks.) But then I thought, why not use [git](http://git.or.cz/)'s object database as a persistent store?

**tekuti is born**

So in this way I turned a terrible idea into a great one: write weblog software, written in Scheme for fun, and using Git as the persistent storage layer. Three weeks later, if you are reading this, [Tekuti](//wingolog.org/software/tekuti/) has the basics working great.

If you visit my web site, it's probably not very impressive. Some things aren't even implemented yet, for example the `tags/` tree, and archives aren't yet exposed to the user. But under the hood there is much new excellence, and quite a bit of old excellence. By way of explaining what I mean, first a bit of background about the architecture.

Tekuti is implemented as a long-running Guile process sitting behind [Apache](http://httpd.apache.org/). Tekuti listens on a TCP port, and Apache connects to the port for every request that it receives, sending headers and post data, and receiving headers and body to send to the client. This protocol is implemented by the misnamed [mod\_lisp](http://www.fractalconcept.com/asp/NS9/sdataQ0hD$o9qWQxMDM==/sdataQuvY9x3g$ecX), which actually has nothing to do with Lisp, but is just used as a way to use Apache as your frontend while processing requests with some kind of long-running process. FastCGI is similar, but with a more complicated protocol.

Like I mentioned, Git's object database is used as a persistent store. Tekuti is configured to look at a particular directory for a "bare" git repository. The format of the store is rather simple: each post is in a tree. There is a blob for post metadata, and a blob for the raw post content. There is a comment tree under each post tree, and each comment is one blob, somewhat like an rfc 822 mail message.

Clearly, however, this format is not appropriate for all access requirements of weblog software. For example, a request to the main page would involve scanning all of the metadata blobs, parsing post timestamps out of all of them, then taking the latest N timestamps, and showing those posts. This process might work fine if you have 10 posts, but with 200 it is a bit more problematic.

Hence, Tekuti keeps an *index* of the persistent store. The index is extensible. It currently stores a pre-parsed and time-ordered copy of post metadata, and a mapping between tags and posts for making a tag cloud or for navigating by tag. In that way showing the last N posts is a simple matter of looking at the index, then pulling specific post contents out of the database with `git show` (and subsequent post-processing).

The names of the post trees are made in such a way that going to 2008/01/19/hackings directly translates into a tree name, so that lookup of individual posts is O(1).

I should mention that the entirety of Tekuti, modulo a couple of local procedures, is written in the functional style. Once allocated, data is never mutated.

Oddly enough, this immutable style of programming promotes the mutability of the program itself. If you run tekuti with the `--repl` argument, it spawns the main listening and processing loop in a thread, leaving the main thread to run a read-eval-print loop. *If you redefine a function in the REPL, it is picked up immediately by the processing thread*. My current deployment strategy is to run it within a [screen](http://www.gnu.org/software/screen/) session, so my REPL is always available to me. I just have to SSH into my server, run `screen -r`, and I can diagnose any problems or change any code that I want to, without restarting the Guile process.

I used similar facilities while developing the software, via Guile's Emacs integration code, known as [GDS](http://article.gmane.org/gmane.lisp.guile.user/6423). If you run Tekuti with `--gds`, it will spawn a thread to connect to a locally running Emacs. At that point, *you can redefine code in the running Tekuti process from within Emacs*. It's an incredible reduction in the standard edit-compile-run cycle, effectively removing the "compile" step.

**it's all functional programming**

Git actively supports the functional programming style: if you have a reference to a tree, that tree will never change out from below you. The only kind of imperative modification that Git supports, besides the interning of new objects into the database, is updating the value of "refs", which conveniently has a compare-and-swap operator, `git-update-ref`. So not only is git's object database excellent for functional programming, it supports concurrency natively. Indeed, the algorithms that one must use to update a ref are exactly those used in lock-free datastructures. Neat stuff!

So, to summarize: the core of Tekuti is a tail-recursive loop, a là erlang, in which after every iteration we do a `goto` to the new code of the function. At the beginning of the iteration, we get the value of the "master" ref, and if it's different from the one Tekuti saw last, we do a reindexing operation (on only those parts of the tree that changed). Then we read the data from the `mod_lisp` socket, dispatch the request, and then -- oh, something I didn't discuss --

All this without harming a single angle-bracket. No , no

; all of the data is represented as s-expressions (instead of x\[ml\]-expressions). Hence we have the entire power of the SXML toolkit at hand: `pre-post-order`, functional tree-fold input parsing, and you never have to think about that stray ampersand in your text, because strings and XML are distinct data types.

**the bad, the ugly**

Instructions on getting the source code, and importing your old Wordpress blog posts, are available at [http://wingolog.org/software/tekuti/](//wingolog.org/software/tekuti/). It's two thousand lines of scheme. The user-level side of things is OK, although it lacks pingback support, and the archives and tag browsing stuff isn't quite finished yet.

On the side of the admin, the one who writes the weblog posts, things are, um, spare. I committed this post directly to the git repository. There's some scaffolding for an admin interface which as yet fails to be awesome. In addition, I'm not sure how well things work when you're starting from an empty repository; probably best to import from wordpress first.

You'll want the latest Guile 1.8, ideally with threading support so you can do things like `--repl` and `--gds`. Then you'll want the latest guile-lib, preferably from [the development repository](http://home.gna.org/guile-lib/).

**things I learned**

So, as I was hacking this thing, I allowed myself the liberty to investigate the meta-level, practices and code that could help me to hack better. I now use [paredit-mode](http://www.emacswiki.org/cgi-bin/wiki/ParEdit) when hacking Scheme code in Emacs, which is a revolution. I now miss it when I hack C. I figured out how to link Emacs and a running Guile application via GDS, which is both crazy futuristic and also crazy 20 years old. (It's pretty much like SLIME for you common lispers.) I learned how to use mod\_lisp and Guile's sockets, and lots of ickiness surrounding HTTP: status codes, decoding post data, implementing basic authentication (and base64 decoding!), encoding of URLs, Last-Modified and If-Modified-Since, etc.

Guile's built-in exception handing is a bit ad-hoc, so I learned SRFI-34 and SRFI-35 to try to systematize things a bit more. Mixed results. The same conclusion probably applies to learning the intricacies of \`format', including its iteration constructs. PCL good book.

I investigated writing this all in R6RS scheme, but the best-looking compiler, ikarus, is not available for my development machine, which runs PPC. Also, R6's support for dynamic programming still isn't there. I'd like to look at it again at some point, perhaps with an eye towards porting; I think that for me, Tekuti will be the litmus test for R6's validity as a programming language. We'll see.

**what's with the name**

Ote ku ti kutya mOshiwambo, "tekuti" otashi ti "ote ku ti".

Te estoy contando que en oshiwambo, "tekuti" quiere decir "te estoy contando".

I'm telling you that in Oshiwambo, "tekuti" means "I'm telling you".

## related articles

- [it's probably spam](/archives/2017/03/06/its-probably-spam)
- [a register vm for guile](/archives/2013/11/26/a-register-vm-for-guile)
- [the merry month of ma](/archives/2012/03/12/the-merry-month-of-ma)
- [meta data](/archives/2010/12/13/meta-data)
- [biting the hand that feeds](/archives/2008/03/07/biting-the-hand-that-feeds)
- [cps in hoot](/archives/2024/05/27/cps-in-hoot)

### 5 responses

1. Andy Wingo says:[29 February 2008 7:00 PM](#0194582e5db7f9435335fa3a91d12078093b4c7f)

   Hm, seems that the post link in the atom feed is busted. If you see any other bugs, please comment. Thanks!

2. Thomas Kappler says:[1 March 2008 10:17 AM](#4da43e720599e56769f11166148925e888b09747)

   Sounds awesome, both the Scheme and the git part!

   Can you give a rough estimate about how much the code depends on Guile, i.e., how much work it would be to port it to, say, PLT?

   Giving "http://thomas11.wordpress.com" as website URL gave me a "Bad URL" error, even though it works fine.

3. Andy Wingo says:[1 March 2008 9:33 PM](#a0d246a3ccc837cd6869b60b9cb377f8dd308c0b)

   Thanks for the feedback!

   Like I say, tekuti's only about 2000 lines of scheme, so there is an upper bound to how much work it would be to port it. Given that PLT has a pretty deep library stack, I wouldn't think that it would be more than about 20 hours of work.

   That said, I think a more interesting port would be to port to some kind of standard scheme, be it r6rs or whatever enhanced r5rs proposals people are bandying about these days.

4. Jakub Narebski says:[2 March 2008 7:19 PM](#696f61cde2a73c293814f414e9de9bc1ae15d037)

   Do you use git directly, calling git commands, or do you access repository structure? I recall that there was discussion on git mailing list about storing maildirs in git, and what could be changed in repository organization to make it faster and more sane. But unfortunately I don't remember details, and don't know if it would help you or not.

   What makes me wondering is why you use Bazaar as SCM for tekuti, and not Git...

5. Jakub Narebski says:[2 March 2008 7:56 PM](#e8991eecaa6ba74ee56368487699ff0bf63212aa)

   If/when 'tekuti' is out of alpha, could you post announcement to git mailing list, git@vger.kernel.org (you don't need to subscribe to post), and perhaps add info about it to GitWiki:

   http://git.or.cz/gitwiki/InterfacesFrontendsAndTools

   TIA

Comments are closed.
