---
title: Brute Force Incognito Browsing
url: https://nullprogram.com/blog/2018/09/06/
published: "2018-09-06T00:00:00Z"
feed: nullprogram
guid: https://nullprogram.com/blog/2018/09/06/
---

# Brute Force Incognito Browsing

## [Brute Force Incognito Browsing](/blog/2018/09/06/)

September 06, 2018

nullprogram.com/blog/2018/09/06/

Both Firefox and Chrome have a feature for creating temporary private browsing sessions. Firefox calls it [Private Browsing](https://support.mozilla.org/en-US/kb/private-browsing-use-firefox-without-history) and Chrome calls it [Incognito Mode](https://support.google.com/chrome/answer/95464). Both work essentially the same way. A temporary browsing session is started without carrying over most existing session state (cookies, etc.), and no state (cookies, browsing history, cached data, etc.) is preserved after ending the session. Depending on the configuration, some browser extensions will be enabled in the private session, and their own internal state may be preserved.

The most obvious use is for visiting websites that you don’t want listed in your browsing history. Another use for more savvy users is to visit websites with a fresh, empty cookie file. For example, some news websites use a cookie to track the number visits and require a subscription after a certain number of “free” articles. Manually deleting cookies is a pain (especially without a specialized extension), but opening the same article in a private session is two clicks away.

For web development there’s yet another use. A private session is a way to view your website from the perspective of a first-time visitor. You’ll be logged out and will have little or no existing state.

However, sometimes *it just doesn’t go far enough*. Some of those news websites have adapted, and in addition to counting the number of visits, they’ve figured out how to detect private sessions and block them. I haven’t looked into *how* they do this — maybe something to do with local storage, or detecting previously cached content. Sometimes I want a private session that’s *truly* fully isolated. The existing private session features just aren’t isolated enough or they behave differently, which is how they’re being detected.

Some time ago I put together a couple of scripts to brute force my own private sessions when I need them, generally for testing websites in a guaranteed fresh, fully-functioning instance. It also lets me run multiple such sessions in parallel. My scripts don’t rely on any private session feature of the browser, so the behavior is identical to a real browser, making it undetectable.

The downside is that, for better or worse, no browser extensions are carried over. In some ways this can be considered a feature, but a lot of the time I would like my ad-blocker to carry over. Your ad-blocker is probably *the* most important security software on your computer, so you should hesitate to give it up.

Another downside is that both Firefox and Chrome have some irritating first-time behaviors that can’t be disabled. The intent is to be newbie-friendly but it just gets in my way. For example, both bug me about logging into their browser platforms. Firefox starts with two tabs. Chrome creates a popup to ask me to configure a printer. Both start with a junk URL in the location bar so I can’t just middle-click paste (i.e. the X11 selection clipboard) into it. It’s definitely not designed for my use case.

### Firefox

Here’s my brute force private session script for Firefox:

```
#!/bin/sh -e
DIR="${XDG_CACHE_HOME:-$HOME/.cache}"
mkdir -p -- "$DIR"
TEMP="$(mktemp -d -- "$DIR/firefox-XXXXXX")"
trap "rm -rf -- '$TEMP'" INT TERM EXIT
firefox -profile "$TEMP" -no-remote "$@"

```

It creates a temporary directory under `$XDG_CACHE_HOME` and tells Firefox to use the profile in that directory. No such profile exists, of course, so Firefox creates a fresh profile.

In theory I could just create a *new* profile alongside the default within my existing `~/.mozilla` directory. However, I’ve never liked Firefox’s profile feature, especially with the intentionally unpredictable way it stores the profile itself: behind random path. I also don’t trust it to be fully isolated and to fully clean up when I’m done.

Before starting Firefox, I register a trap with the shell to clean up the profile directory regardless of what happens. It doesn’t matter if Firefox exits cleanly, if it crashes, or if I CTRL-C it to death.

The `-no-remote` option prevents the new Firefox instance from joining onto an existing Firefox instance, which it *really* prefers to do even though it’s technically supposed to be a different profile.

Note the `"$@"`, which passes arguments through to Firefox — most often the URL of the site I want to test.

### Chromium

I don’t actually use Chrome but rather the open source version, Chromium. I think this script will also work with Chrome.

```
#!/bin/sh -e
DIR="${XDG_CACHE_HOME:-$HOME/.cache}"
mkdir -p -- "$DIR"
TEMP="$(mktemp -d -- "$DIR/chromium-XXXXXX")"
trap "rm -rf -- '$TEMP'" INT TERM EXIT
chromium --user-data-dir="$TEMP" \
         --no-default-browser-check \
         --no-first-run \
         "$@" >/dev/null 2>&1

```

It’s exactly the same as the Firefox script and only the browser arguments have changed. I tell it not to ask about being the default browser, and `--no-first-run` disables *some* of the irritating first-time behaviors.

Chromium is *very* noisy on the command line, so I also redirect all output to `/dev/null`.

If you’re on Debian like me, its version of Chromium comes with a `--temp-profile` option that handles the throwaway profile automatically. So the script can be simplified:

```
#!/bin/sh -e
chromium --temp-profile \
         --no-default-browser-check \
         --no-first-run \
         "$@" >/dev/null 2>&1

```

In my own use case, these scripts have fully replaced the built-in private session features. In fact, since Chromium is not my primary browser, my brute force private session script is how I usually launch it. I only run it to test things, and I always want to test using a fresh profile.

- [linux](/tags/linux/)
- [debian](/tags/debian/)
- [trick](/tags/trick/)
- [web](/tags/web/)

Have a comment on this article? Start a discussion in my [public inbox](https://lists.sr.ht/~skeeto/public-inbox) by sending an email to [~skeeto/public-inbox@lists.sr.ht](mailto:~skeeto/public-inbox@lists.sr.ht?Subject=Re%3A%20Brute%20Force%20Incognito%20Browsing) \[ [mailing list etiquette](https://man.sr.ht/lists.sr.ht/etiquette.md)\] , or see [existing discussions](https://lists.sr.ht/~skeeto/public-inbox?search=Brute+Force+Incognito+Browsing).

« [Prospecting for Hash Functions](/blog/2018/07/31/)

» [From Vimperator to Tridactyl](/blog/2018/09/20/)
