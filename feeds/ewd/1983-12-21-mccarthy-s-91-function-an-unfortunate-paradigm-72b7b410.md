---
title: "McCarthy’s 91–function: an unfortunate paradigm (EWD845)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD845.html
published: "1983-12-21T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD845.PDF
---

# McCarthy’s 91–function: an unfortunate paradigm

McCarthy’s 91-function: an unfortunate paradigm

McCarthy’s 91-function —see below— is an ingenious and at first sight baffling construction, which deserved all the attention it attracted when it became known. Unfortunately it also acquired the status of “difficult testcase” for any proposed theory or set of proof rules for recursively defined functions. This is unfortunate for two reasons. Firstly, this function can be dealt with —see below— by such thoroughly elementary means that it is doubtful whether it is a very good testcase; secondly, its contortions are of such a nature —see below— that any method of dealing with it shows more of those contortions that of the method that is supposed to be illustrated.

McCarthy’s 91-function is g, given by the following recursive definition

g(x) = if x > 100 ⇒ x - 10 ⫾ x ≤ 100 ⇒ g(g(x + 11)) fi

and one is asked to demonstrate that it equals f, given by

f(x) = if x > 101 ⇒ x - 10 ⫾ x ≤ 101 ⇒ 91 fi

Here we go!

g(x)
definition of g

if x > 100 ⇒ x - 10 ⫾ x ≤ 100 ⇒ g(g(x + 11)) fi
splitting the first alternative and unfolding once in the second alternative

if x > 101 ⇒ x - 10

⫾ x = 101 ⇒ 91

⫾ x ≤ 100 ⇒ g(if x + 11 > 100 ⇒ x + 11 - 10

⫾ x + 11 ≤ 100 ⇒ g(g(x + 11 + 11)) fi)

fi
splitting of the last alternative and simplification

if x > 101 ⇒ x - 10

⫾ x = 101 ⇒ 91

⫾ 89 x ≤ 100 ⇒ g(x + 1)

⫾ x 3(x + 22)

fi

terminating tail-recursion of third alternative

if x > 101 ⇒ x - 10

⫾ x = 101 ⇒ 91

⫾ 89 x ≤ 100 ⇒ g(101)

⫾ x ≤ 89 ⇒ g3(x + 22)

fi
combination of second and third alternative and terminating tail-recursion of the last one

if x > 101 ⇒ x - 10

⫾ 89 x ≤ 101 ⇒ 91

⫾ x ≤ 89 ⇒ g2n + 3(y) for some n ≥ 0 and 89 y ≤ 111

fi
applying g once in the last alternative

if x > 101 ⇒ x - 10

⫾ 89 x ≤ 101 ⇒ 91

⫾ x ≤ 89 ⇒ g2n + 2(y) for some n ≥ 0 and 91 y fi
applying g another time in the last alternative

if x > 101 ⇒ x - 10

⫾ 89 x ≤ 101 ⇒ 91

⫾ x ≤ 89 ⇒ g2n + 1(91) for some n ≥ 0

fi
since g(91) = 91, the last two can be combined

if x > 101 ⇒ x - 10

⫾ x ≤ 101 ⇒ 91

fi
definition of f

f(x)          Q.E.D.

That is all, in a long sequence of minute transformations. It is a winding argument that, of course, can be delivered in 57 varieties, but when you have seen one of them, you have seen them all. The 91-function is, as said, an ingenious construction, but it is an unfortunate paradigm since it leads to demonstrations that are both lengthy and boring.

I hope this is the last technical note in which the 91-function is treated.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

21 December 1983

prof.dr.Edsger W. Dijkstra

Burroughs Research Fellow

transcribed by Jonathan David Arndt

revised
21-Jan-2020
