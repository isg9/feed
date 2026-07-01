---
title: Have your agent record video demos of its work with shot-scraper video
url: https://simonwillison.net/2026/Jun/30/shot-scraper-video/#atom-everything
published: "2026-06-30T16:54:26Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/30/shot-scraper-video/#atom-everything
---

# Have your agent record video demos of its work with shot-scraper video

[shot-scraper video](https://shot-scraper.datasette.io/en/stable/video.html) is a new command introduced in today's [shot-scraper 1.10](https://github.com/simonw/shot-scraper/releases/tag/1.10) release which accepts a `storyboard.yml` file defining a routine to run against a web application and uses Playwright to record a video of that routine. I've written before about the importance of [having coding agents produce demos](https://simonwillison.net/2026/Feb/10/showboat-and-rodney/#proving-code-actually-works) of their work; this is my latest attempt at enabling them to do that.

Here's an example video created using `shot-scraper video`, exercising a [still in development](https://github.com/simonw/datasette/pull/2813) feature adding the ability to create new tables in Datasette from pasted CSV, TSV or JSON data:

That video was created by running this command:

```
shot-scraper video datasette-bulk-insert-storyboard.yml \
  --auth datasette-demo-auth.json --mp4
```

(That `--auth` JSON file [contains a cookie](https://gist.github.com/simonw/287b26aff53fcb72942b19f5b69d7e5c), as [described here](https://shot-scraper.datasette.io/en/stable/authentication.html) in the documentation.)

Here's the `datasette-bulk-insert-storyboard.yml` file:

```
output: /tmp/datasette-bulk-insert-demo.webm
server:
  - uv
  - --directory
  - /Users/simon/Dropbox/dev/datasette
  - run
  - datasette
  - -p
  - 6419
  - --root
  - --secret
  - "1"
  - /tmp/demo.db
url: http://127.0.0.1:6419/demo/tasks
viewport:
  width: 1280
  height: 720
cursor: true
wait_for: 'button[data-table-action="insert-row"]'
javascript: |
  (() => {
    let clipboardText = "";
    Object.defineProperty(navigator, "clipboard", {
      configurable: true,
      get: () => ({
        writeText: async (text) => {
          clipboardText = String(text);
        },
        readText: async () => clipboardText,
      }),
    });
  })();
scenes:
  - name: Bulk insert existing table rows
    do:
      - pause: 0.8
      - click: 'button[data-table-action="insert-row"]'
      - wait_for: "#row-edit-dialog[open]"
      - pause: 0.5
      - click: ".row-edit-bulk-insert"
      - wait_for: ".row-edit-bulk-textarea"
      - pause: 0.5
      - click: ".row-edit-copy-template"
      - wait_for: "text=Copied"
      - pause: 0.8
      - fill:
          into: ".row-edit-bulk-textarea"
          text: |
            title,owner,status,priority,notes
            Prepare release video,Ana,doing,1,Recorded with shot-scraper
            Check pasted CSV import,Ben,review,3,Previewed before inserting
            Share the branch demo,Chen,queued,2,Bulk insert creates three rows
      - pause: 0.8
      - click: ".row-edit-save"
      - wait_for: "text=Previewing 3 rows."
      - pause: 1.2
      - click: ".row-edit-save"
      - wait_for: "text=3 rows inserted."
      - pause: 1.0
      - click: ".row-edit-cancel"
      - wait_for: "text=Prepare release video"
      - pause: 1.0
  - name: Create a table from pasted CSV
    open: http://127.0.0.1:6419/demo
    wait_for: 'details.actions-menu-links summary'
    do:
      - pause: 0.8
      - click: 'details.actions-menu-links summary'
      - click: 'button[data-database-action="create-table"]'
      - wait_for: "#table-create-dialog[open]"
      - pause: 0.5
      - fill:
          into: ".table-create-table-name"
          text: "launch_metrics"
      - click: ".table-create-from-data"
      - wait_for: ".table-create-data-textarea"
      - pause: 0.5
      - fill:
          into: ".table-create-data-textarea"
          text: |
            metric_id,name,score,recorded_on
            m001,Activation rate,87.5,2026-06-29
            m002,Retention check,72.25,2026-06-30
            m003,CSV import health,95,2026-07-01
      - pause: 0.8
      - click: ".table-create-save"
      - wait_for: "text=Previewing 3 rows."
      - pause: 1.2
      - click: ".table-create-save"
      - wait_for_url: "**/demo/launch_metrics"
      - wait_for: "text=Activation rate"
      - pause: 1.2
```

The [video command documentation](https://shot-scraper.datasette.io/en/stable/video.html) includes simpler examples, but for the purpose of this post I thought I'd go with something more comprehensive.

That demo YAML storyboard was constructed entirely by GPT-5.5 xhigh running in Codex Desktop, using the following prompt run inside my `~/dev/datasette` checkout of [this branch](https://github.com/simonw/datasette/commits/b759ea548606bc9bf9a4bf0e33e2d57ead7e0ab8/):

> `Review the changes on this branch.`
>
> `cd to ~/dev/shot-scraper and run the command "uv run shot-scraper video --help"`
>
> `Now use that new video command to record a video demo of the new features from this branch, including running a "uv run datasette -p 6419 --root --secret 1 /tmp/demo.db" development server so you can record the video against a demo DB that you first create.`

Now that I've released the feature the prompt could say " `run uvx shot-scraper video --help`" instead and it should achieve the same result.

I really like this pattern where the `--help` output for a command provides enough detail that a coding agent can use it - it works kind of like bundling a `SKILL.md` file directly inside the tool. I used the same pattern for [showboat and rodney](https://simonwillison.net/2026/Feb/10/showboat-and-rodney/).

#### How I built this

`shot-scraper video` started as an experimental prototype. `shot-scraper` is built on top of [Playwright](https://playwright.dev/), and the key feature it needed was for Playwright to be able to record video of browser sessions with enough control to create the desired demo.

I first tried this a few years ago and found that the Playwright-produced videos included additional chrome that was useful for debugging a test failure but unwanted for a product demo.

They fixed that a while ago, but there were still some minor blockers. In particular I was getting [a few white frames at the start of the videos](https://github.com/simonw/shot-scraper/pull/194/changes/c2f3b3a52ba84f2adcf3ad6da4d39c2570328584#issuecomment-4724459369), since the recording mechanism kicked in before the first URL was loaded by the browser.

Playwright 1.59 added a new [screencast mechanism](https://playwright.dev/python/docs/api/class-screencast) providing much more finely grained control over video recording. This was very nearly what I needed, but the resulting videos were fixed at 800px wide.

I found a [landed PR fixing that](https://github.com/microsoft/playwright/pull/41183) but it wasn't yet in a release. Then yesterday they shipped it in [playwright-python 1.61.0](https://github.com/microsoft/playwright-python/releases/tag/v1.61.0) and I was finally unblocked to finish implementing the feature!

The code itself was all written by GPT-5.5 xhigh in Codex Desktop. I had it write the documentation as well which gave me a very useful frame for reviewing the design - much of the iteration on the feature came from reviewing that documentation, spotting things that were redundant, inconsistent or confusing, and requesting (or dictating) a better design.

The YAML format itself was mostly defined by the coding agent. I had it [use Pydantic](https://github.com/simonw/shot-scraper/blob/1.10/shot_scraper/video.py#L24) to both define and validate the format, partly to make the design easier to review.

This is a great example of the kind of feature that I almost certainly wouldn't have taken on without coding agent support. I filed the [original issue](https://github.com/simonw/shot-scraper/issues/142) in February 2024, and had difficulty finding the necessary time to solve this in amongst all of my other projects.

Tags: [projects](https://simonwillison.net/tags/projects), [python](https://simonwillison.net/tags/python), [yaml](https://simonwillison.net/tags/yaml), [ai](https://simonwillison.net/tags/ai), [datasette](https://simonwillison.net/tags/datasette), [playwright](https://simonwillison.net/tags/playwright), [shot-scraper](https://simonwillison.net/tags/shot-scraper), [generative-ai](https://simonwillison.net/tags/generative-ai), [llms](https://simonwillison.net/tags/llms), [pydantic](https://simonwillison.net/tags/pydantic), [coding-agents](https://simonwillison.net/tags/coding-agents), [agentic-engineering](https://simonwillison.net/tags/agentic-engineering)
