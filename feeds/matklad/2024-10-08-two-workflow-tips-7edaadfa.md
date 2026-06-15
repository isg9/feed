---
title: Two Workflow Tips
url: https://matklad.github.io/2024/10/08/two-tips.html
published: "2024-10-08T00:00:00Z"
feed: matklad
guid: https://matklad.github.io/2024/10/08/two-tips.html
---

# Two Workflow Tips

Oct 8, 2024

An article about a couple of relatively recent additions to my workflow which I wish I knew about years ago.

## [Split And Go To Definition](\#Split-And-Go-To-Definition)

Go to definition is super useful (in general, navigation is much more important than code completion). But often, when I use “goto def” I don’t actually mean to permanently *go* there. Rather, I want to stay where I am, but I need a bit more context about a particular thing at point.

What I’ve found works really great in this context is to *split* the screen in two, and issue “go to def” in the split. So that you see both the original context, and the definition at the same time, and can choose just how far you would like to go. Here’s an example, where I want to understand how `apply` function works, and, to understand *that*, I want to quickly look up the definition of `FuzzOp`:

VS Code actually has a first-class UI for something like this, called “Peek Definition” but it is just not good — it opens some kind of separate pop-up, with a completely custom UX. It’s much more fruitful to compose two existing basic features — splitting the screen and going to definition.

Note that in the example above I do move focus to the split. I also tried a version that keeps focus in the original split, but focusing new one turned out to be much better. You actually don’t always know up front which split would become the “main” one, and moving the focus gives you flexibility of moving around, closing the split, or closing the other split.

I highly recommend adding a shortcut for this action. It’s a good idea to make it a “complementary” shortcut for the usual goto definition. I use `, .` for goto definition, and hence `, >` is the splitting version:

```
{ "key": ", .",       "command": "editor.action.revealDefinition" },
{ "key": ", shift+.", "command": "editor.action.revealDefinitionAside" },
```

## [, .](\#s-1)

Yes, you are reading this right. `, .`, that is a comma followed by a full stop, is my goto definition shortcut. This is not some kind of evil vim mode. I use pedestrian non-modal editing, where I copy with `ctrl + c`, move to the beginning of line with `Home` and kill a word with `ctrl + Backspace` (though keys like `Home`, `Backspace`, or arrows are on my home row thanks to [`kanata`](https://github.com/jtroo/kanata)).

And yet, I use `,` as a first keypress in a sequence for multiple shortcuts. That is, `, .` is not `, + .` pressed together, but rather a `,` followed by a separate `.`. So, when I press `,` my editor doesn’t actually type a comma, but rather waits for me to complete the shortcut. I have many of them, with just a few being:

- `, .` goes to definition,

- `, >` goes to definition in a split,

- `, r` runs a task,

- `, e s` **e** dits selection by sorting it, `, e C` converts to camel **C** ase,

- `, o g` **o** pens [magit for VS Code](https://github.com/kahole/edamagit), `, o k` opens keybindings.

- `, w` re- **w** raps selection at 80 (something I just did to format the previous bullet point), `, p` **p** retty-prints the whole file.

I’ve used many different shortcut schemes, but this is by far the most convenient one for me. How do I type an comma? I bind `, Space` and `, Enter` to insert comma and a space/newline respectively, which handles most of the cases. And there’s `, ,` which types just a lone comma.

To remember longer sequences, I pair the comma with [whichkey](https://github.com/VSpaceCode/vscode-which-key), such that, when I type `, e`, what I see is actually a menu of editing operations:

![](https://github.com/user-attachments/assets/57f50d23-8c4d-4ec3-a1f0-51fce46bf4d0)

This horrible idea was born in the mind of Susam Pal, and is officially (and aptly I should say) named [Devil Mode](https://susam.github.io/devil/).

I highly recommend trying it out! It is the perfect interface for actions that you do once in a while. Where it doesn’t work is for actions you want to repeat. For example, if you want to cycle through compilation errors, binding `, e` to the “next error” would probably be a bad idea, as typing `, e , e , e` to cycle three times is quite tiring.

This is actually a common theme, there are *many* things you might to cycle back and forward through:

- completion suggestions

- compiler errors

- textual search results

- reference search results

- merge conflicts

- working tree changes

It is mighty annoying to have to remember different shortcuts for all of them, isn’t it? If only there was some way to have a universal pair of shortcuts for the next/prev generalized motion…

The insight here is that you’d rarely need to cycle through several different categories of things at the same time. So I bind the venerable `ctrl+n` and `ctrl+p` to *repeating* the last next/prev motion. So, if the last next thing was a worktree change, then `ctrl+n` moves me to the next worktree change. But if I then query the next compilation error, the subsequent `ctrl+n` would continue cycling through compilation errors. To kick-start the cycle, I have a `, n` hydra:

- `, n e` next error

- `, n c` next change

- `, n C` next merge Conflict

- `, n r` next reference

- `, n f` next find

- `, n .` previous edit

I don’t know if there’s some existing VS Code extension to do this, I implement this in [my personal extension](https://github.com/matklad/config/blob/0f690f89c80b0e246909b54a0e97c67d5ce6ab0c/my-code/src/main.ts#L63-L96).

Hope this is useful! Now go and make a deal with the devil yourself!
