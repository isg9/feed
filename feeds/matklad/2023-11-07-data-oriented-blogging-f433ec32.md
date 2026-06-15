---
title: Data Oriented Blogging
url: https://matklad.github.io/2023/11/07/dta-oriented-blogging.html
published: "2023-11-07T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2023/11/07/dta-oriented-blogging.html
---

# Data Oriented Blogging

Nov 7, 2023

Wherein I describe the setup of this blog. The main take away from the post are not specific technical tools, but the underlying principles and ideas, which I wish I had articulated earlier.

> If you don’t understand the data you don’t understand the problem.

[Sun Tzu](https://youtu.be/rX0ItVEVjHc?si=2IzGZSFKsi1xkls8&t=780)

Physically, a typical blog is a directory of `.html` and `.css` files which are available over HTTP. The simplest way to create those files is to just write them by hand. While in some cases that might be enough, often it isn’t.

If a blog has multiple pages, you usually want to have some common elements — header, footer, style, etc. It *is* possible to get common style by copy-pasting an existing page every time you need to add something. This makes changes hard — having consistent layout at a single point in time is not sufficient, one almost always wants to be able to apply consistent modifications as well.

The second issue with hand-written html is that some parts might be very annoying to hand-write. For example, code snippets with syntax highlighting require quite a few tags.

Finally, writing html by hand is not necessarily most convenient. A `*` bullet-list certainly is more pleasant to look at than an ``

!

That’s why a blog usually is some sort of a script (a program) which reads input content in some light markup language (Markdown typically) and writes HTML. As most blogs are similar, it is possible to generalize and abstract such a script, and the result would be called a “static site generator”. I don’t like this term very much, as it sounds more complicated than the underlying problem at hand — reading `.md` files and writing out `.html` s.

Static Site Generators are an example of the template method pattern, where the framework provides the overall control flow, but also includes copious extension points for customizing behaviors. Template method allows for some code re-use at the cost of obscure and indirect control flow. This pattern pays off when you have many different invocations of template method with few, if any, non-trivial customizations. Conversely, a template method with a single, highly customized call-site probably should be refactored away in favor of direct control flow.

If you maintain dozens mostly identical websites, you definitely need a static site generator. If you have only one site to maintain, you might consider writing the overall scaffolding yourself.

If you pick an SSG, pay close attention to its extensibility mechanism, you want to avoid situations where you know how to do something “by hand”, but the SSG is not flexible enough to express that. Flexible SSG typically have some way to inject user code in their processing, for free-form customization. For this reason, I am somewhat skeptical of static site generators implemented in languages without eval, such as Go and Rust. They might be excellent as long as they fulfill your needs exactly. However, should you need something more custom than what’s available out of the box, you might find yourself unable to implement that. This creates discontinuity in complexity.

Note that I am *not* saying that every site out there needs some custom plugins. Most (certainly, most well-maintained ones) work just fine with fairly vanilla configurations. Rather, it’s a statement about risks — there’s small, but non-zero probability that you’ll need something quite unusual. However, should you find yourself with a use-case which is not supported by your SSG’s available customization options, the cost of the work-around could be very high.

I only have to maintain this single blog, and I want the freedom to experiment with fairly custom things, so, in this context, writing the scaffolding script myself makes more sense.

## [The Best Tool For The Job](\#The-Best-Tool-For-The-Job)

Converting from one text format to another isn’t particularly resource intensive and is trivial to parallelize. But it requires a fair amount of work with strings, dates, file-system and such, which points towards a higher-level programming language. However, the overriding concern is stability — blogs usually don’t enjoy active daily maintenance, and nothing can distract more than the need to sort out the tooling even before you get to writing.

I wish I could recommend a stable high-level scripting language, but, to my knowledge, there isn’t any yet. Python falls apart as soon as you need to install a dependency. While I highly recommend [*Boring Dependency Management*](https://www.b-list.org/weblog/2022/may/13/boring-python-dependencies/), knowing that `pip-compile` is that one extra tool you need is one extra tool too many. While Node works for adding dependencies, it makes it hard to keep up with them (in Node.JS, dependencies manage you). I don’t know a lot about Ruby, but, in the Jekyll days of this blog, I’ve never learned how to configure `bundler` (or is it `gem`?) to use project-local dependencies by default.

For this reason, I’d say picking Go or Rust for the task makes sense. Yes, those are quite a bit more verbose than what you’d ideally need for bossing Markdown around, but their quality of implementation is great, and QoI is what matters here most.

I use [Deno](https://deno.land) for this blog. Deno is poised to become that scripting environment I wish existed: [https://matklad.github.io/2023/02/12/a-love-letter-to-deno.html](https://matklad.github.io/2023/02/12/a-love-letter-to-deno.html)

In addition to the overall QoI, it has particular affinity for web stuff. Out of the box, it has extra niceties, like file system watching and hot-reloading, or the permissions system to catch mistakes when reading or writing wrong files. The only reason to recommend Rust and Go over Deno at this point is that Deno is still pretty young, and, subjectively, needs more time to graduate into boring tech.

Having picked the language, which text format should be the input?

## [Data In](\#Data-In)

The most typical choice here is Markdown. Markdown is … fine overall, but it does have one pretty glaring deficiency — vanilla Markdown doesn’t allow for custom elements. An example of a custom element would be a shortcut, like `ctrl + c`. In stock Markdown, there’s no syntactic mechanism to designate something as “this is my custom element”. You can add syntactic extensions, but then you’ll need new syntax for each custom element. Alternatively, you can use a Markdown dialect which supports generic extensibility, like [Pandoc Markdown](https://pandoc.org/MANUAL.html#extension-fenced_divs) or [MDX](https://mdxjs.com).

An interesting choice for source data format would be HTML with custom tags. If you write some HTML by hand, that doesn’t mean you have to write all HTML manually — a script can desugar hand-written HTML into more verbose form for the browser. For example, the source for a post can contain a snippet like this:

```
<listing lang="rust">
fn main() {
    println!("Hello, World!")
}
listing>
```

The script then reads and parses this HTML, and produces appropriate `pre > code` with syntax highlighting soup.

HTML would be my recommendation if I were optimizing for stability. These days, many editors have [emmet](https://emmet.io) out of the box, which makes producing HTML not that horrible:

But wouldn’t it be great if there was an extensible light markup language, which combines concise syntax and tasty sugar of Markdown with extensibility and flexibility of HTML, [*The Elements of a Great Markup Language*](https://matklad.github.io/2022/10/28/elements-of-a-great-markup-language.html)? It actually sort-of-exists already: [https://djot.net](https://djot.net)

Djot is quite a bit like Deno in that it takes well established good ideas, and just doesn’t mess up the implementation. But it is also an emerging technology at this point, not even at 1.0, so use at your own risk.

## [Data Out](\#Data-Out)

The output clearly has to be HTML, but there are many ways to manufacture this markup. Producing HTML is not an entirely solved problem. Usually, some sort of textual templating is used, but that’s a fundamentally wrong approach: [https://www.devever.net/~hl/stringtemplates](https://www.devever.net/~hl/stringtemplates)

For this problem, the shape of data is not that of a string, rather it is a tree.

Luckily for the blogging domain, the main motivation for proper solution is protection from XSS. Blogs usually don’t include user-submitted content, so one can play fast and loose with escaping. That is to say, if your language supports [string interpolation](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Template_literals), that might be enough of a templating engine. That is what this blog does — just backticks in TypeScript.

What’s tantalizing is that the proper solutions is clearly visible, and is *just* out of reach. We have JSX now, the proper “write code to produce trees” solution. Sadly, I don’t think it’s immediately usable as of yet. [Deno docs](https://docs.deno.com/runtime/manual/advanced/jsx_dom/jsx#jsx-import-source) mention some “new” JSX API, with an initial support. I also don’t see some built-in (or obviously blessed) way to take my JSX syntax and convert it to string when writing to an `.html` file.

## [Look and Feel](\#Look-and-Feel)

It’s not enough to produce HTML, it also has to look good. I don’t find this acceptable:

[https://danluu.com/input-lag/](https://danluu.com/input-lag/)

It is completely unreadable on a 16:9 screen. Which might have good second-order effects (people clearly come for content rather than for style), but, really, just no :-)

It would be fair to say that that’s browser’s (aka backwards compatibility) fault — ideally, unstyled HTML would look good, with some reasonable max-width and default body font-size a touch larger than 16px. I’d love if there were some sort of ``
