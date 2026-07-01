---
title: Count the number of Safari tabs
url: https://simonwillison.net/2026/Jun/29/safari-tab-count/#atom-everything
published: "2026-06-29T18:36:18Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/29/safari-tab-count/#atom-everything
---

# Count the number of Safari tabs

Tiniest TIL, using AppleScript to count the number of open browser tabs in Safari:

```
osascript -e 'tell application "Safari" to count tabs of every window'

```

![I ran it in a terminal window and got back 370.](https://static.simonwillison.net/static/2026/tab-shame.jpg)

Tags: [safari](https://simonwillison.net/tags/safari), [til](https://simonwillison.net/tags/til), [applescript](https://simonwillison.net/tags/applescript)
