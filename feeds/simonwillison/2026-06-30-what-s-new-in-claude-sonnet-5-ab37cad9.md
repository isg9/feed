---
title: What's new in Claude Sonnet 5
url: https://simonwillison.net/2026/Jun/30/claude-sonnet-5/#atom-everything
published: "2026-06-30T21:23:02Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/30/claude-sonnet-5/#atom-everything
---

# What's new in Claude Sonnet 5

**[What's new in Claude Sonnet 5](https://platform.claude.com/docs/en/about-claude/models/whats-new-sonnet-5)**

Claude Sonnet 5 came out [this morning](https://www.anthropic.com/news/claude-sonnet-5). I always head straight for the "what's new" developer docs because they tend to have more actionable information than the official announcement post.

Anthropic say of Sonnet 5 that "its performance is close to that of Opus 4.8, but at lower prices". The [system card](https://www-cdn.anthropic.com/9e6a1044980d8c4ed85669faf9c2a8342e2e9f1e/Claude%20Sonnet%205%20System%20Card.pdf) helps explain how they were able to release the model without being blocked by the US government:

> Sonnet 5 is significantly less capable at cyber tasks than Mythos 5: its safeguards are thus similar to those we apply to Opus 4.7 and Opus 4.8 (models that are more capable than Sonnet 5 but much less capable than Mythos 5).

Of note from the "what's new" API changes:

- Sampling parameters `temperature`, `top_p`, `top_k` are no longer supported.
- It has a 1 million token context window and 128,000 maximum output tokens.
- It features "the same set of tools and platform features as Claude Sonnet 4.6"
- Adaptive thinking is on by default, unless you specify `"thinking": {type: "disabled"}`.
- The pricing is the same as Sonnet 4.6: $3/million input, $15/million input, with an introductory discount to $2/$10 until 31st August. But...
- The model has a new tokenizer, where "The same input text produces approximately 30% more tokens than on Claude Sonnet 4.6." - effectively a 30% price increase.

I used my [Claude Token Counter](https://tools.simonwillison.net/claude-token-counter) tool to try out the new tokenizer. Here are my results for several larger documents:

DocumentSonnet 4.6Opus 4.7Sonnet 5[Universal Declaration of Human Rights (English)](https://github.com/simonw/udhr-markdown/blob/main/declarations/eng.md)**2,356****3,347**

1.42x**3,341**

1.42x[Universal Declaration of Human Rights (Spanish)](https://github.com/simonw/udhr-markdown/blob/main/declarations/spa.md)**3,572****4,753**

1.33x**4,747**

1.33x[Universal Declaration of Human Rights (Chinese, Mandarin Simplified)](https://github.com/simonw/udhr-markdown/blob/main/declarations/cmn_hans.md)**3,334****3,366**

1.01x**3,360**

1.01x[sqlite\_utils/db.py](https://github.com/simonw/sqlite-utils/blob/79117b9d110d72f46dab5fe2cda412ff4789ab55/sqlite_utils/db.py) (4,279 lines of Python)**44,014****56,118**

1.28x**56,113**

1.27x

So the new token is roughly 1.4x times more expensive for English, 1.33x for Spanish, 1.28x for Python code and effectively the same cost for Simplified Mandarin.

Here's [the pelican](https://gist.github.com/simonw/a89e756b621a31e8ffc210e3428efa77). It's nothing to write home about. Sonnet 5 thinks it looks like a goose.

![Illustration of a white goose riding a bicycle, with one wing extended forward to grip the handlebar, set against a plain white background with a brown ground line.](https://static.simonwillison.net/static/2026/sonnet-5-pelican.png)

Via [Hacker News](https://news.ycombinator.com/item?id=48736605)

Tags: [ai](https://simonwillison.net/tags/ai), [generative-ai](https://simonwillison.net/tags/generative-ai), [llms](https://simonwillison.net/tags/llms), [anthropic](https://simonwillison.net/tags/anthropic), [claude](https://simonwillison.net/tags/claude), [llm-pricing](https://simonwillison.net/tags/llm-pricing), [pelican-riding-a-bicycle](https://simonwillison.net/tags/pelican-riding-a-bicycle), [llm-release](https://simonwillison.net/tags/llm-release)
