---
title: named globs with curl
url: https://daniel.haxx.se/blog/2026/05/16/named-globs-with-curl/
published: "2026-05-16T20:58:26Z"
feed: danielstenberg
guid: https://daniel.haxx.se/blog/?p=29654
---

# named globs with curl

One of the established power features of the curl command line tool is its support for “globbing”. It is a built-in way to specify ranges and sets in different ways and have curl iterate over them to simplify repeated transfers.

For example, you can easily download three images from the same host without having to repeat the almost same URL three times:

```
curl -O "https://img.example/{flower,house,garden}.jpg"
```

Or if you have them in a numbered range, you can get a thousand images in a single tiny command line:

```
curl -O "https://img.example/photo[1-1000].jpg"
```

And they can be combined in crazy ways:

```
curl -O "https://example.{org,com}/{apples,mugs,coins}/img[1-1000].jpg"
```

curl allows globs used in a single URL to create up to 263 permutations – which, if you can do one million transfers per second, would take 292 thousand years to complete.

(As an added bonus you can of course also add `-Z` to the command line to make curl transfer all those images in parallel rather than serially.)

## Saving them

To help users save files when using globbing, curl provides a way to reference the globbed components using `#[number]` when setting the target filename. The number then references the specific glob, where the first is 1, the second 2 etc.

Saving the one thousand images using different filenames locally than they use remotely:

```
curl "https://img.example/photo[1-1000].jpg" -o "local-#1.jpeg"
```

This allows a compact command line to also offer flexibility.

## News coming in 8.21.0

All functionality mentioned above has existed in curl for years; decades even. It just so happened that one day when working with curl I fell over a use case that I could not solve with the existing command line functionality.

I wanted to do a globbed upload to a HTTP server and then save all the separate responses into their own dedicated files, preferably with names based on the glob.

I will admit that I at first had a hard time to accept the fact that we actually could not do this already, but that was then rather quickly instead turned into: how should I add support for this in the smoothest and most convenient way? Using what syntax?

The road to fixing it for uploads took a little detour.

## Name instead of index

Starting in 8.21.0, curl can assign a name to each glob and then reference that glob by name instead of using just a glob index number. This allows command lines to get ever so slightly more readable I think.

The image range example from above, but instead using named globs:

```
curl "https://img.example/photo[<number>1-1000].jpg" -o "local-#<number>.jpeg"
```

Or a version with three separate globs where they all are used in the output file name:

```
curl "https://example.{<tld>org,com}/{<object>apples,mugs,coins}/img[<index>1-1000].jpg" -o "file-#<index>-#<tld>-#<object>"
```

Slick, right?

## Uploads

Back to the globbed upload challenge:

```
curl -T "{one,two,three}.jpg" https://ul.example/
```

… but with the responses saved in separate files instead of sent to stdout. Use named globs:

```
curl -T "{<counter>one,two,three}.jpg" https://ul.example/ -o "save-#<counter>"
```

The only way to refer to an upload glob is to set a name and refer to that name. There are no indexed references for uploads, only for URL globs.

## Mixing URL globs and uploads globs

It is in fact possible to also use a mix of upload globs and URL globs in the same command line if you want to upload multiple files to multiple destinations. They set the names in the same namespace and you refer to the names the same way, independently of source.

This feels more like a thing to show off in a blog post like this rather than something people will actually find good use for:

Upload three files to three sites, save all nine response in separate files:

```
curl -T "{<counter>one,two,three}.jpg" https://{<host>ul1,ul2,ul3}.example/ -o "save-#<counter>-#<host>"
```

## Index and names

Even when you set a name, the glob can still also be references by its number.
