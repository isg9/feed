---
title: 'tracepoints: gnarly but worth it — wingolog'
url: https://wingolog.org/archives/2025/02/14/tracepoints-gnarly-but-worth-it
published: "2025-02-14T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2025/02/14/tracepoints-gnarly-but-worth-it
---

# tracepoints: gnarly but worth it — wingolog

## [tracepoints: gnarly but worth it](/archives/2025/02/14/tracepoints-gnarly-but-worth-it)

14 February 2025 1:32 PM

- [whippet](/tags/whippet)
- [tracing](/tags/tracing)
- [profiling](/tags/profiling)
- [gc](/tags/gc)
- [garbage collection](/tags/garbage%20collection)
- [igalia](/tags/igalia)
- [nlnet](/tags/nlnet)
- [lttng](/tags/lttng)
- [perfetto](/tags/perfetto)

Hey all, quick post today to mention that I added tracing support to the [Whippet GC library](https://github.com/wingo/whippet). If the support library for [LTTng](https://lttng.org) is available when Whippet is compiled, Whippet embedders can visualize the GC process. Like this!

[![Screenshot of perfetto showing a generational PCC trace](https://wingolog.org/pub/nboyer-generational-pcc-trace-small.jpg)](https://wingolog.org/pub/nboyer-generational-pcc-trace.png)

Click above for a full-scale screenshot of the [Perfetto](https://ui.perfetto.dev) trace explorer processing the [`nboyer`
microbenchmark](https://github.com/wingo/whiffle/blob/main/examples/nboyer.scm) with the [parallel copying collector](https://github.com/wingo/whippet/blob/main/doc/collector-pcc.md) on a 2.5x heap. Of course no image will have all the information; the nice thing about trace visualizers like is that you can zoom in to sub-microsecond spans to see exactly what is happening, have nice mouseovers and clicky-clickies. Fun times!

### on adding tracepoints

Adding tracepoints to a library is not too hard in the end. You need to [pull in the `lttng-ust` library](https://lttng.org/docs/v2.13/#doc-c-application), which has a `pkg-config` file. You need to [declare your tracepoints in one of your header
files](https://github.com/wingo/whippet/blob/main/api/gc-lttng.h). Then you have a [minimal C file](https://github.com/wingo/whippet/blob/main/src/gc-tracepoint.c) that includes the header, to generate the code needed to emit tracepoints.

Annoyingly, this header file you write needs to be in one of the `-I` directories; it can’t be just in the the source directory, because `lttng` includes it seven times (!!) using [computed
includes](https://gcc.gnu.org/onlinedocs/cpp/Computed-Includes.html) (!!!) and because the LTTng file header that does all the computed including isn’t in your directory, GCC won’t find it. It’s pretty ugly. Ugliest part, I would say. But, grit your teeth, because it’s worth it.

Finally you pepper your source with tracepoints, which probably you [wrap
in some
macro](https://github.com/wingo/whippet/blob/main/api/gc-tracepoint.h) so that you don’t have to require LTTng, and so you can switch to other tracepoint libraries, and so on.

### using the thing

I wrote up a little [guide for Whippet users about how to actually get
traces](https://github.com/wingo/whippet/blob/main/doc/tracepoints.md). It’s not as easy as `perf record`, which I think is an error. Another ugly point. Buck up, though, you are so close to graphs!

By which I mean, so close to having to write a Python script to make graphs! Because LTTng writes its logs in so-called Common Trace Format, which as you might guess is not very common. I have a colleague who swears by it, that for him it is the lowest-overhead system, and indeed in my case it has no measurable overhead when trace data is not being collected, but his group uses custom scripts to convert the CTF data that he collects to... [GTKWave](https://gtkwave.sourceforge.net/) (?!?!?!!).

In my case I wanted to use Perfetto’s UI, so I found a [script](https://profilerpedia.markhansen.co.nz/converters/babeltrace-to-json-py/) to convert from CTF to the [JSON-based tracing format that Chrome
profiling used to
use](https://docs.google.com/document/d/1CvAClvFfyA5R-PhYUmn5OOQtYMH4h6I0nSsKchNAySU/preview?tab=t.0#heading=h.yr4qxyxotyw). But, it uses an old version of Babeltrace that wasn’t available on my system, so I had to write a [new
script](https://github.com/wingo/whippet/blob/main/ctf_to_json.py) (!!?!?!?!!), probably the most Python I have written in the last 20 years.

### is it worth it?

Yes. God I love blinkenlights. As long as it’s low-maintenance going forward, I am satisfied with the tradeoffs. Even the fact that I had to write a script to process the logs isn’t so bad, because it let me get nice nested events, which most stock tracing tools don’t allow you to do.

I fixed a small performance bug because of it – a [worker thread was
spinning waiting for a pool to terminate instead of helping
out](https://wingolog.org/pub/check-termination.png). A win, and one that never would have shown up on a sampling profiler too. I suspect that as I add more tracepoints, more bugs will be found and fixed.

### fin

I think the only thing that would be better is if tracepoints were a part of Linux system ABIs – that there would be header files to emit tracepoint metadata in all binaries, that you wouldn’t have to link to any library, and the actual tracing tools would be intermediated by that ABI in such a way that you wouldn’t depend on those tools at build-time or distribution-time. But until then, I will take what I can get. Happy tracing!

## related articles

- [ephemerons vs generations in whippet](/archives/2025/01/09/ephemerons-vs-generations-in-whippet)
- [preliminary notes on a nofl field-logging barrier](/archives/2024/10/03/preliminary-notes-on-a-nofl-field-logging-barrier)
- [whippet progress update: feature-complete!](/archives/2024/09/18/whippet-progress-update-feature-complete)
- [whippet progress update: funding, features, future](/archives/2024/07/24/whippet-progress-update-funding-features-future)
- [parallel ephemeron tracing](/archives/2023/01/24/parallel-ephemeron-tracing)
- [ahead-of-time wasm gc in wastrel](/archives/2026/02/06/ahead-of-time-wasm-gc-in-wastrel)

### One response

1. strsnd says:[15 February 2025 9:39 AM](#0056f1f3e15363f7a7b05911a806bf72c83afed4)

   I think the only thing that would be better is if tracepoints were a part of Linux system ABIs

   Not really a spec’ed Linux ABI but USDT trace points are written into ELF notes of binaries and can be traced using perf. My system already contains quite a bunch of them: `perf list sdt | grep postgresql | wc -l` =\> 57

   Good quick introduction: https://www.brendangregg.com/perf.html#USDT

Comments are closed.
