---
title: "It is all distributivity (EWD1142)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD11xx/EWD1142.html
published: "1992-11-02T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd11xx/EWD1142.PDF
---

# It is all distributivity

Let • be a binary infix operator. Depending on how we Curry, we can form two classes of unary operators from , the general elements of which are often denoted by (x•) and (•y), defined

(x•) = 〈λy: x•y〉      (•y) = 〈λx: x•y〉

We call them in this note "the derived unary operators".

To begin with we observe —with apologies for the two different usages of parentheses—

(x•z)•y = x•(z•y)

=        {definition of derived operators}

(•y).((x•).z) = (x•).((•y).z)

=        {definition of functional composition}

((•y)◦(x•)).z = ((x•)◦(•y)).z  ,

hence, that a binary operator is "associative" means that its derived unary operators commute.

To say that • "distributes over" some binary operator is really a statement about its derived unary operators: "• distributes from the left over □" means that for all x, y, z:

(x•).(y □ z) = (x•).y □ (x•).z   ;

distribution from the right means similarly

(•x).(y □ z) = (•x).y □ (•x).z   .

Both formulae are of the form

f.(y □ z) = f.y □ f.z  ,

which captures "f distributes over □". But what comes of this formula if □ turns out to be unary? Replacing in the last formula p□q by g.p, we get

f.(g.y) = g.(f.y) or f◦g = g◦f .

Hence, that two unary operators "commute" means that they distribute over each other. Hence, that a binary operator is associative just means that its derived unary operators distribute over each other.

Austin, 2 November 1992

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas as Austin

Austin, TX 78712-1188 USA

Transcribed by Guy Haworth.

Last revised Mon, 31 Dec 2007.
