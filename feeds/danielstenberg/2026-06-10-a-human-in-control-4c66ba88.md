---
title: A human in control
url: https://daniel.haxx.se/blog/2026/06/10/a-human-in-control/
published: "2026-06-10T07:08:26Z"
feed: danielstenberg
guid: https://daniel.haxx.se/blog/?p=30056
---

# A human in control

There seems to be a fair amount of people in either extremes in
the current AI landscape. At one side we see the “vibe coders” who
use agents and allow them to merge code without any person even
looking at the source, while on the other side of the field there
are people who are against everything and anything even remotely
associated with AI.

My personal stance is somewhere in between, as I suppose shouldn’t
be too surprising to readers of this blog.

## A work of love and pride

The core team behind curl, and that is more people than just me,
consists of individuals to whom code quality and source code
excellence is important. We do software development because it is
a craft we love and we are proud of what we have accomplished this
far. We do not hand over our responsibilities to any machines. _We
stand for every bit of code we merge – as humans._

## AIs do mistakes

Blindly accepting code written by AI means that you merge a
certain amount of errors, but this is certainly true for human
written code as well, so this is not in itself special. Some data
suggests that AI generated code might even contain more mistakes
than the human versions.

We invented test cases and code review a long time ago as a means
to help us combat and reduce mistakes to get merged. The
particular way code was written does not take away the benefits
from code review and getting additional checks and eyes on pending
changes. A good code review helps spotting mistakes, omissions or
slip-ups. It also helps reinforce the architecture and established
design choices. This is true however the code was created.

This far, code reviews done by automatic AI bots and the likes
have not yet managed to replace the humans. They are simply not
good enough.

Human reviews are much better. They catch other things and they
help make sure proposed changes stay on track.

Not to mention how I want to know how curl works, even if I don’t
keep 100% intimate knowledge of every single angle and corner, I
know most of it. I think it helps me make better decisions, debug
better, help users better and keep the architecture sound.

Getting the initial code written is not the big deal. For curl,
maintaining and polishing the landed code _through decades_ is the
real task.

_Everything we merge in curl is determined fine and fitting by
humans._

## Humans do mistakes

In all living software projects we get bugs reported and we fix
them. We do new releases and continue to iterate. We have done
this since software was invented and we still do, as humans are
quite fallible and easily make mistakes.

We try to reduce the error density and frequency by adding tests
and by adding more human eyes on the code before we green-light
it. It helps, but is not perfect.

To help us do better code we invent, introduce and enforce a wide
variety of different tools. With tools that look at code and
identify problems in the early stages, they help avoid landing bad
code in the first place. They make us do better code. They reduce
the bug frequency.

Some of the best tools for detecting coding mistakes today use AI.
These tools might work on existing source code in a git repository
or they might look at proposed changes in pull-requests.

Above I mentioned that human code reviews are better; but the
opposite is also true. In a somewhat complicated change request,
it is now common that after the humans can’t spot any more
problems, the AI PR review bots can still find an issue or two to
remark on. Sure, sometimes they are wrong and then the comment is
easily dismissed, but more often than not the findings they point
out are actually something worth addressing before merge.

_curl is developed and driven by humans, assisted by tools._

## Communication is for humans

Open Source is about sharing code and is a development model where
we do things in the open. The _communication_ part of this model
is key. Share your ideas, your visions, your problems or maybe
just your ideas for what to do this afternoon.

Express what you want or what the problem is, and the team can
respond and we can work together on fixing and improving whatever
needs to be done.

Effective communication, a condition for good Open Source, implies
_human-to-human_ interaction. Inserting a large AI generated
tone-deaf large wall-of-text into such a flow _can_ still work,
but only in the same way humans can learn to work with difficult
individuals as well. It is not ideal and it is not a smooth way of
working. It introduces sand in the machine. Don’t do that. It is
rude.

_Effective Open Source work means we communicate as humans, even
if parts of the work and the code is made with the help of AI._

## The combination

Humans and machines excel at different things. We can complement
each other in software development.

Everyone is free to act to their own will, but in the curl project
we don’t hand over responsibility to machines. We stand for our
product. We make it as good as we possibly can; using all the
tools that are available to us. I claim that in order to do this,
humans need to remain in control.
