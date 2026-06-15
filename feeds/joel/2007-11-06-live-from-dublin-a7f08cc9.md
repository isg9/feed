---
title: Live from Dublin
url: https://www.joelonsoftware.com/2007/11/06/live-from-dublin/
published: "2007-11-06T00:11:23Z"
feed: joel
guid: https://www.joelonsoftware.com/?p=682
---

# Live from Dublin

![](https://i0.wp.com/www.joelonsoftware.com/wp-content/uploads/2007/11/06excelbug.png?w=730&ssl=1)Chris Lomont has a [detailed reverse-engineering of the 2007 Excel floating point bug](http://www.lomont.org/Math/Papers/2007/Excel2007/Excel2007Bug.pdf) \[PDF\]. “The bug seemed to be introduced when the formatting routine was updated from older 16-bit assembly code used in previous versions of Excel…”

Stefan is continuing his series on [Wasabi](http://www.fogcreek.com/FogBugz/blog/post/Wasabi-How-Do-I-Picture-Functions.aspx) with a look at picture functions. A “picture function” is a function that looks like a picture of what it generates, e.g.:

```
 Sub MakeTitle( s )
%>
  <title>    <% =encode(s) %>  </title>
<%
End Sub
```

We just got into the hotel in Dublin and checked out tomorrow morning’s meeting room, and it’s bigger than they told us, with room for 144 people, so we should be able to clear the waiting list.
