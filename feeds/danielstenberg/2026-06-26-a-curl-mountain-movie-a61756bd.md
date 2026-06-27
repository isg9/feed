---
title: A curl mountain movie
url: https://daniel.haxx.se/blog/2026/06/26/a-curl-mountain-movie/
published: "2026-06-26T11:34:00Z"
feed: danielstenberg
guid: https://daniel.haxx.se/blog/?p=30202
---

# A curl mountain movie

One of my favorite visuals for known vulnerabilities in curl is *the mountain*. It shows how many currently known vulnerabilities were present in the code through-out curl’s history.

In the end of June 2026 it looks like this:

![](https://daniel.haxx.se/blog/wp-content/uploads/2026/06/Screenshot-2026-06-26-at-11-48-10-curl-Project-status-dashboard.png)

Over time we get more vulnerabilities reported. Since every flaw has a version range during which the problem existed and with more issues that have overlapping version ranges, the mountain grows. It changes shape every time we do a release or we publish a new vulnerability.

At this moment in time, [curl version 7.34.0](https://curl.se/ch/7.34.0.html) is the release that contains the most number of known vulnerabilities: [101](https://curl.se/docs/vuln-7.34.0.html). The worst one ever if you will. Out of a total of 206.

The mountain uses different colors for different severity levels of the published vulnerabilities, as the legend in the top-left of the image explains.

To illustrate the ever-changing nature of the shape and size, I wrote a script that renders *the mountain* the way it looked at specific dates in the past up until today. More specifically, the script renders one image for every month since curl started (March 1998). I then turned these 340 individual images into a little movie that shows how it grew into today’s shape. At four months/second.

The data for this come from [vuln.pm](https://github.com/curl/curl-www/blob/master/docs/vuln.pm) and the [curl git repository](https://github.com/curl/curl). The graph rendering is based on the [dashboard scripts](https://github.com/curl/stats/). All images put into a movie with ffmpeg of course.

## The 2016 drop

Several people have asked what happened in 2016 that caused the notable drop. A slope if you will.

If we zoom in on that, we can spot that [curl 7.51.0](https://curl.se/ch/7.51.0.html) has eleven fewer vulnerabilities than the version before that. This release was the first one after the 2016 [Cure53 code audit](https://daniel.haxx.se/blog/2016/11/23/curl-security-audit/), but other than that there is no clear distinct process or obvious code changes that explain this trend shift.

Lots of other graphs show just the ordinary pace and growth in various project areas. It was still fairly early days CI-wise but had been running at least a few CI jobs per commit for a few years already by then.

curl was adopted into the OSS-Fuzz project in July 2017, which since then makes us find some issues better, but the drop looks like it happened before then.

We had already been analyzing the code regularly on Coverity since a few years.

Better tooling? New compiler options? We simply don’t know.

## Future

As we keep announcing more vulnerabilities going forward, things will continue to change. Maybe I will come back and make another movie in five years?
