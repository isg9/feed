---
title: 'Ornith-1.0: Self-Scaffolding LLMs for Agentic Coding'
url: https://simonwillison.net/2026/Jun/29/ornith/#atom-everything
published: "2026-06-29T16:17:59Z"
feed: simonwillison
guid: https://simonwillison.net/2026/Jun/29/ornith/#atom-everything
---

# Ornith-1.0: Self-Scaffolding LLMs for Agentic Coding

**[Ornith-1.0: Self-Scaffolding LLMs for Agentic Coding](https://deep-reinforce.com/ornith_1_0.html)**

This is an interesting new open weights (MIT licensed) model, the first model release from DeepReinforce.

> \[...\] with variants including 9B Dense, 31B Dense, 35B MoE, and 397B MoE. Built on top of pretrained Gemma 4 and Qwen 3.5, it achieves state-of-the-art performance among open-source models of comparable size on coding benchmarks.

As far as I can tell the licenses of those underlying models is compatible with being used in this way - Gemma 4 is Apache 2.0 licensed (and not bound by the janky additional [Gemma Terms of Use](https://ai.google.dev/gemma/terms) that afflicted the previous Gemma models) and Qwen 3.5 is Apache 2.0 licensed as well.

I've been running the model using LM Studio and the [ornith-1.0-35b-Q4\_K\_M.gguf](https://huggingface.co/deepreinforce-ai/Ornith-1.0-35B-GGUF) (20GB) GGUF, hooked up to [Pi](https://pi.dev/). Initial impressions are very good - it seems to be able to run the agent harness over many tool calls in a proficient way.

Here's [a terminal session](https://gisthost.github.io/?35da4d9ce7f0c27124c67655a0dc9e5d) where I asked it to "find the code that decodes the actor cookie" and then "find the code that opens the insert dialog when thebutton is clicked" against a Datasette checkout, which it handled with ease.

I also had it [draw this pelican](https://gist.github.com/simonw/1869e1bbcafe5bcad0f26351f6a978a6), which came out at 103 tokens/second:

![Cartoon illustration of a white pelican (albeit slightly mangled) with a large orange beak riding a red bicycle across green hills. The scene has a blue sky with a yellow sun and three white clouds, and small grass tufts dot the foreground.](https://static.simonwillison.net/static/2024/ornith-1-pelican.png)

It's a little bit mangled but the pelican is clearly a pelican.

I couldn't find much information about DeepReinforce themselves. The earliest paper I could find from the was [CUDA-L1: Improving CUDA Optimization via Contrastive Reinforcement Learning](https://arxiv.org/abs/2507.14111) from June 2025.

Tags: [ai](https://simonwillison.net/tags/ai), [generative-ai](https://simonwillison.net/tags/generative-ai), [local-llms](https://simonwillison.net/tags/local-llms), [llms](https://simonwillison.net/tags/llms), [qwen](https://simonwillison.net/tags/qwen), [pelican-riding-a-bicycle](https://simonwillison.net/tags/pelican-riding-a-bicycle), [gemma](https://simonwillison.net/tags/gemma), [llm-release](https://simonwillison.net/tags/llm-release), [lm-studio](https://simonwillison.net/tags/lm-studio)
