---
title: "The arithmetic and geometric means once more (EWD1231)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1231.html
published: "1996-02-10T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1231.PDF
---

# The arithmetic and geometric means once more

EWD 1231

The arithmetic and geometric means once more

In the following,  x, y, c  are positive.

In EWD1140, I used

(0)         (x+y)� = (x−y)� + 4 � x � y

to argue that

(1)         x,y = c, x+y−c       ,

which does not change x+y, increases x � y provided c lies between the initial values of x and y.

In EWD1171, I used (0) to argue that

(2)         x,y := c, x � y/c       ,

which does not change x � y, decreases x+y provided c lies between the initial values of x and y.

In both cases the use of (0) came a little bit as a rabbit and the link between the condition on c and the decrease of the distance between x and y remained informal. Last Thursday, when I asked for an expression that contained both x+y and x � y, my [student] An Thai Nguyen suggested that we look at (c−x)�(c−y), and this expression indeed plays a central role in the derivations from which all rabbits have been removed.

*         *         *

We want to change x,y such that (i) their sum is not changed, and (ii) their product is increased. Any assignment satisfying (i) can be written like (1); in order to satisfy (ii) we now observe for any c

“(1) increases x � y”

=         {program semantics, (1)}

x � y < c �(x+y−c)

=         {algebra}

(c−x)�(c−y) < 0      .

We now consider the change of x,y such that (iii) their product is not changed, and (iv) their sum is decreased. Any assignment satisfying (iii) can be written like (2); in order to satisfy (iv) as well, we observe for any positive c

“(2) increases x+y”

=         {program semantics, (2)}

c + x � y/c < x+y

=         { c > 0 }

c� + x � y < c �(x+y)

=         { algebra }

(c−x)�(c−y) < 0      .

So, in both cases, the completely forced calculations lead in exactly the same form to the conclusion that c should lie between the initial values of x and y. The secret is that Nguyen’s expression can be rewritten as

x � y − c �(x+y−c)

and as

(c� + x � y) − (c � x + c � y)       ,

i.e. the difference of two products with equal sums of their factors, and the difference of two sums with equal products of their addenda. I was surprised.

Austin, 10 February 1996

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188, USA

transcribed by Corrado Cantelmi

revised
26-Dec-2011
