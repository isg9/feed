---
title: call/cc and ecmascript — wingolog
url: https://wingolog.org/archives/2009/02/25/callcc-and-ecmascript
published: "2009-02-25T00:00:00Z"
feed: wingo
guid: https://wingolog.org/archives/2009/02/25/callcc-and-ecmascript
---

# call/cc and ecmascript — wingolog

## [call/cc and ecmascript](/archives/2009/02/25/callcc-and-ecmascript)

25 February 2009 9:29 AM

- [guile](/tags/guile)
- [ecmascript](/tags/ecmascript)
- [continuations](/tags/continuations)

I suppose it should have been obvious, but I was still delightfully surprised when Clinton Ebadi came up with these lines on IRC this morning:

> ```
> ecmascript@(guile-user)> var callcc = this['call/cc'];
> ecmascript@(guile-user)> var k = false;
> ecmascript@(guile-user)> 1 + callcc (function (kont) { k = kont; return 0; });
> 1
> ecmascript@(guile-user)> k (2);
> 3
>
> ```

42 good times indeed!

## related articles

- [partitioning ambiguous edges in guile](/archives/2025/04/25/partitioning-ambiguous-edges-in-guile)
- [scheme quiz time!](/archives/2013/11/02/scheme-quiz-time)
- [generators in v8](/archives/2013/05/08/generators-in-v8)
- [on generators](/archives/2013/02/25/on-generators)
- [eval, that spectral hound](/archives/2012/02/01/eval-that-spectral-hound)
- [JavaScriptCore, the WebKit JS implementation](/archives/2011/10/28/javascriptcore-the-webkit-js-implementation)

### 5 responses

1. [wingo](http://wingolog.org/) says:[25 February 2009 9:39 AM](#8ccab2b44d0371f0fb9dc522b03ef7b04edbd82a)

   Hum, that pre-formatted text looks pretty terrible, overflowing through the navigation box thing. I suppose I should fix the stylesheet somehow. Does someone have a clever idea about how to do that?

2. Dan says:[25 February 2009 10:36 AM](#34123230c162a6163b9a714941fd99519316b3a1)

   I don't think that there is a good way to avoid floats, as far a pre is conserned, the two goals are mutually exclusive. I would at least make the pre overflow: auto so that it does not spill out of the parent.

   Preformatted text is of course not the only way to format code, it is just the easiest. Using a different method (eg. a div for each line with non-collapsing yet still breaking spaces of some description) would allow wrapping to occur, athough I still have not invented a nice way to automatically have continuation symbols show up.

   I remain certain I'll find a clean CSS way eventually.

3. zorg says:[25 February 2009 10:38 PM](#52b31759cb5f27dfdee2716c9cef13a8340c8233)

   pre { margin-right : 10em ; }

4. notzorg says:[26 February 2009 2:32 AM](#a1a7296eab6e0513d4b00ccb29e57372701a7112)

   pre { width:24em; overflow:auto; }

5. [Arne Babenhauserheide](http://draketo.de) says:[13 August 2015 8:54 AM](#26098ce8176b0973bd10d2d16ccbef2e4f705f95)

   You could make this look good by simply leaving out most of the prompt: ecmascript@(guile-user)> → >

Comments are closed.
