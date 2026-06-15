---
title: doing it wrong — wingolog
url: https://wingolog.org/archives/2010/12/23/doing-it-wrong
published: "2010-12-23T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2010/12/23/doing-it-wrong
---

# doing it wrong — wingolog

## [doing it wrong](/archives/2010/12/23/doing-it-wrong)

23 December 2010 0:04 AM

- [web](/tags/web)
- [parsing](/tags/parsing)
- [tekuti](/tags/tekuti)

Intrepid hacker [Aleix Conchillo Flaqué](http://hacks-galore.org/aleix/) writes in to say that, against all odds, he actually managed to install the [Tekuti](//wingolog.org/software/tekuti/) blog software on his server. [Rock on!](http://hacks-galore.org/aleix/blog/)

Of course, it was not without a couple of problems, and indeed the important one points to something more fundamentally wrong with existing internet technologies.

**in which mod\_rewrite fails the author**

Whoa, Mr. Wingo, calm down there! Blaming the internet for a bug in your software! [Hubris is a virtue](http://c2.com/cgi/wiki?LazinessImpatienceHubris) and all that, but perhaps this is taking it too far? I'm sure my readers will let me know in the comments, but first, some background.

The symptom of the problem is like this. To refer to a post in a URL, tekuti has an identifier, the post key. For example, the path to access to edit a post is `/admin/posts/key`.

Tekuti doesn't have post numbers though, so the easiest way to generate the post key is to serialize its location in the data store, like `2010/12/22/doing-it-wrong`. As you can see the post key can include any character, and indeed typically does include slashes, so the actual text representation of the URL to edit a post has to be percent-encoded, like `/admin/posts/2010%2f12%2f22%2fdoing-it-wrong`.

This scheme works fine, and indeed when accessing the Tekuti web server directly, everything works. But if you put it behind Apache using a `mod_rewrite` proxying rule, as Aleix did:

```
RewriteRule ^/blog(/?.*)$ http://localhost:8080/$1 [P]

```

Then trying to edit posts doesn't work! What's the deal?

**parsing it wrong**

The deal is, `mod_rewrite` does a textual match on *decoded paths*. So the string that `mod_rewrite` sees isn't the string given to it -- in this case, it sees `/admin/posts/2010/12/22/doing-it-wrong`. Then it does some arbitrary transformation on that decoded string, and somehow constructs the result.

*But you cannot produce the desired result from the intermediate, decoded string!*

There are various options to re-encode parts of the output string, or to not encode parts of it -- but you can't determine which slash in the intermediate string is to be re-encoded.

The problem is that `mod_rewrite` treats slashes specially, not re-encoding them. This problem is *fundamental* to any technology that processes URL paths using textual comparisons.

To parse a URL path properly, you must first split it according to the delimiters you are interested in ( `/`, in this case). Then you percent-decode the path components into a list, and match on that list.

Query parameters need to be parsed similarly -- first you split on `&`, then split on the first `=` in each component, then percent-decode the resulting keys and values into an ordered list of key-value pairs.

Quoth the RFC:

> When a URI is dereferenced, the components and subcomponents significant to the scheme-specific dereferencing process (if any) must be parsed and separated before the percent-encoded octets within those components can be safely decoded, as otherwise the data may be mistaken for component delimiters.
>
> [RFC 3986, section 2.4: When to encode or decode](http://www.apps.ietf.org/rfc/rfc3986.html#sec-2.4)

This problem isn't specific to regular expressions -- it also occurs, for example, when dispatching a HTTP request to a URI handler. The dispatch should be based on [path components](http://gitorious.org/tekuti/tekuti/blobs/master/tekuti/web.scm#line36), and not the against the path as a string.

**it's a programming language problem**

So why does this problem occur, even in technologies as venerable as `mod_rewrite`? Because it's one that can only be solved by programming languages. You need some sort of sequence data type to parse paths. You need some sort of map to parse query strings, conventionally at least. And then to reconstitute a URI, you need to do so from those data types (lists, maps, &c).

Most contexts in which you do dispatch, like in your `.htaccess`, aren't equipped with the expressivity to do it right. Regular expressions are only generally applicable on URI subcomponents like path components -- they can only be used in limited situations on full paths.

**workarounds**

In Aleix's case, the workaround is to use `mod_proxy` directly:

```
ProxyPass /blog http://localhost:8080/

```

It's unfortunate, as you can't use `mod_proxy` from an `.htaccess` file -- only from the main configuration, and requiring a restart of the server. Oh well, though.

You could still use mod\_rewrite, but with special rules for the specific URI paths that might include escaped slashes, like this:

```
RewriteRule ^/blog/admin/posts/(.+)$ http://localhost:8080/admin/posts/$1 [P,B]

```

But such a special solution is just that -- special, i.e., not general.

**en brevis**

If you're hacking something for that web that is intended for general use, and if you parse or generate URIs, do your users a favor and give them an appropriate programming language. Your users will thank you, or more likely, continue in blissful ignorance, which is just as well.

## related articles

- [meta data](/archives/2010/12/13/meta-data)
- [service update](/archives/2023/12/14/service-update)
- [colophonwards](/archives/2023/12/05/colophonwards)
- [guile's reader, in guile](/archives/2021/04/11/guiles-reader-in-guile)
- [it's probably spam](/archives/2017/03/06/its-probably-spam)
- [the merry month of ma](/archives/2012/03/12/the-merry-month-of-ma)

### 2 responses

1. [Peter Bex](http://sjamaan.ath.cx) says:[26 December 2010 7:55 PM](#32fed099211de0eee7e83b2f09fbe0873f577a8c)

   I feel your pain. For something so old and thoroughly debugged as Apache you'd think basic stuff like URI matching would work, but no. I also had some issues with trying to represent empty path components. Apparently there's no way to stop Apache from dropping empty paths parts.

2. James Henstridge says:[27 December 2010 11:14 AM](#e720b0ad111f0f503460d3abcd8b659219cbd218)

   We ran into a similar problem putting a Apache mod\_rewrite frontend in front of a CouchDB server. In this case, there were encoded slashes in the path that mod\_rewrite was decoding. It turned out that CouchDB didn't mind if the slashes were decoded, but doing so broke the OAuth signature on the request which caused the problem to show up as an authentication failure.

Comments are closed.
