---
title: Authorship in the AI Age
url: https://blog.computationalcomplexity.org/2026/05/authorship-in-ai-age.html
published: "2026-05-27T14:39:12Z"
feed: complexity
guid: tag:blogger.com,1999:blog-3722233.post-7044748621172288438
---

# Authorship in the AI Age

The [technical paper](https://cdn.openai.com/pdf/74c24085-19b0-4534-9c90-465b8e29ad73/unit-distance-proof.pdf) for the Erdős Unit Distance Problem lists only "OpenAI" as an author. When Bill [posted on Sunday](https://blog.computationalcomplexity.org/2026/05/two-erdos-problems-on-points-in-plane.html) about the Erdős distance problems, he mentioned the names of OpenAI researchers who prompted and checked the proof. Sebastien Bubeck of OpenAI reached out directly and [later as a comment](https://blog.computationalcomplexity.org/2026/05/two-erdos-problems-on-points-in-plane.html?showComment=1779727243132#c6137786223904113449) saying this was really an OpenAI-wide achievement, and we shouldn't just single out people at the end of the funnel. I agree with Bubeck for this paper, but it brings up some challenging authorship issues for the future.

Most publishers follow the [position of the Committee on Publication Ethics](https://publicationethics.org/guidance/cope-position/authorship-and-ai-tools) that AI tools cannot be listed as an author of a paper.

> AI tools cannot meet the requirements for authorship as they cannot take responsibility for the submitted work. As non-legal entities, they cannot assert the presence or absence of conflicts of interest nor manage copyright and license agreements.

The paper doesn't list the model as an author, rather the organization (OpenAI) that produced it. The closest analog might be the [paper that verified the Higgs boson](https://doi.org/10.1016/j.physletb.2012.08.020), which has only "ATLAS Collaboration" on the author line and lists the nearly three thousand authors at the end of the paper. If OpenAI were to publish their paper, they should list everyone who worked on the model with at least a corresponding author to handle the legal issues mentioned in the COPE position.

There is also all the mathematicians whose work was fed into the training data, but I wouldn't list them even if we could. We don't add as authors those whose work we rely on, rather we cite them, and the OpenAI paper does have a healthy references section.

Shortly after the OpenAI announcement, Princeton Professor Will Sawin [improved and gave an explicit exponent for the lower bound](https://arxiv.org/abs/2605.20579) in a currently singularly authored paper. Often, but not always, when a paper improves a bound of a recently announced result, the papers get merged. I don't expect that to happen here, but if it did I would have no idea how to handle the authorship of the combined paper beyond just listing Sawin and OpenAI as the authors.

OpenAI used an internal model for this work, but one would hope they will eventually release this model to the public. If someone outside of OpenAI uses a released model to answer another open math question, even with a single prompt, do they need to add OpenAI as a co-author? I think not, as the model now becomes a tool when released. The researcher should disclose the role the model played, but the model's creators no longer need to be authors.

How about a thought experiment. Suppose you are working with a student by email on a paper and you put in equal effort so you will both be co-authors. Then you discover the student was mostly just feeding your emails to ChatGPT and sending you back the responses. Now who should be on the author list? The student should come off but it would feel strange writing this a solely authored paper.

What if someone builds an AI mathematician that doesn't take any prompts? It just reads papers and occasionally writes its own. Who is the author? That "someone". What if they just used a regular AI agent and gave it the prompt "You are a research mathematician. Go read papers, prove theorems and write up your results." Maybe we'll have AI math journals filled with papers written, edited and refereed by AI models. I wonder how we'll COPE with that.
