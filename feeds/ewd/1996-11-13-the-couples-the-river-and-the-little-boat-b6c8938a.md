---
title: "The couples, the river, and the little boat (EWD1250)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1250.html
published: "1996-11-13T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1250.PDF
---

# The couples, the river, and the little boat

Edsger W. Dijkstra Archive: The couples, the river, and the little boat (EWD 1250)
-

EWD1250

The couples, the river, and the little boat

Here is the problem:

"Two husbands and two wives have to cross a river in a boat which can only hold two people. How can they cross so that no woman is in the company of a man unless her husband is also present?"

(Copyright (c) 1967 by Morris Kline)

[It is understood that the absence/presence constraint refers to the separate banks as well as to the boat.]

*                   *

*

Request If you think this problem too trivial to waste your time on, please state instantaneously the number of different solutions. Thank you. ( End of Request.)

We can name the four people by considering the couples (H,W) and (h,w); we can then enumerate the disallowed combinations:

wH    wHW    Wh    Whw

(which is shorter than enumarating the allowed ones) and explore the tree of all possible scenarios, an extensive search.

There are two ways of reducing the size of the search tree, viz.

(i)  restoring the symmetry between the couples, a symmetry which has been destroyed by naming them differently, and

(ii)  strenghtening the constraints — which, if done unwisely, may reduce the number of solutions to zero.

Ignoring the distictions between the two couples, i.e. ignoring the distinction betweer upper and lower case in our enumeration of disallowed combinations, there is little else we can do but count the numbers of wives and husbands. We propose the stronger constraint:

the husbands stay together.

(i.e. in each combination, the number of husbands is even). But as soon as the husbands stay together, we don't need to distinguish between them —they could be truly identical twins— , nor between the wives: the pairing, as was represented by the couples, has disappeared from the picture, and after this abstraction there is only one solution left:

two wives go ;

one wife returns ;

two husbands go ;

one wife returns ;

two wives go ,

and since the two wives are as indistinguishable from each other other as the electrons in the helium atom, this is truly 1 solution. (Remember: the instruction "take away one from three" yields "two", and it makes no sense, when instructed to take away one from three, to ask "which one?".)

Aside Wim Feijen solved the above problem in exactly the same way "during a short nap on the coach". (End of Aside.)

Austin, 13 November 1996

prof. dr Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712–1188 USA

Transcribed by Gijs Geleijnse

Revised Mon, 10 Dec 2007.
