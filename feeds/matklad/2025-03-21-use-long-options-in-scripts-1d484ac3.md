---
title: Use Long Options in Scripts
url: https://matklad.github.io/2025/03/21/use-long-options-in-scripts.html
published: "2025-03-21T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2025/03/21/use-long-options-in-scripts.html
---

# Use Long Options in Scripts

Mar 21, 2025

Many command line utilities support short form options ( `-f`) and long form options ( `--force`). Short form is for interactive usage. In scripts, use the long form.

That is, in your terminal, type `$ git switch -c my-new-branch`

In your release infrastructure script, write

```
try shell.exec("git fetch origin --quiet", .{});
try shell.exec(
    "git switch --create release-{today} origin/main",
    .{ .today = stdx.DateUTC.now() },
);
```

Long form options are much more self-explanatory for the reader.
