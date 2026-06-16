---
title: "A non algebraic example of a constructive correctness proof (EWD375)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD03xx/EWD375.html
published: "1973-05-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd03xx/EWD375.PDF
---

# A non algebraic example of a constructive correctness proof

We consider a text built from five types of characters, viz. digits,
operators, “(”, “)” and “;”. The text can be read from left to right by
means of the primitive move which assigns to the variable named “x” the next
character of the text. Initially the value of x is undefined, after the
first execution of move, x equals the left-most character of the text. The
text has infinite length and it is required to write a recognizer called “sent” that has to establish whether the text starts with a sent > ,
given by the following syntax.

sent > ::= exp >;

exp > ::= term > {operator > term >}

term > ::= digit > {digit >} | (exp >)

(1)

Here { ..... } should be read as “zero or more times the enclosed”. An
alternative syntax would have been

< sent > ::= < exp >;

< exp > ::= {< term >< operator >} < term >

< term > ::= {< digit >} < digit > | (< exp >)

(2)

We regard (1) and (2) as trivially equivalent.

In order to carry out this investigation, the routine has at its
disposal five boolean functions, defined when x has a value, viz. digit(x),
operator(x), open(x), close(x) and semi(x), such that always exactly one of
the five has the value true.

The result of the investigation has to be recorded in the global
boolean named “c” (short for: “correct”). In order to specify more rigorously
the net effect to be established by a call on sent, we introduce the following
notations.

Let S be the non-empty string of characters read by sent; by definition
x then equals the right-most character of S and by S − x we denote the
string of characters of S up to and excluding this right-most character.

We now want to express two things: if a sent > is found, S should
equal that sentence; if, however, sent has established that the text does
not start with a sent >, no more characters than are needed for this
conclusion should have been read. In view of our later needs, we introduce
two new syntactic categories, viz. pbo sent > and bo sent >. Here
pbo sent > stands for “proper begin of a sentence but not a complete
sentence”, i.e:. a string of characters not forming a sentence all by itself,
but that still can be extended to form a correct sentence; bo sent > ,
being short for “begin of sentence”, is either a correct sentence all by
itself, or allowing extension to a correct sentence. In other words

bo sent > ::= pbo sent > | sent >

(3)

Similarly we introduce:

bo exp > ::= pbo exp > | exp >

(4)

bo term ::= pbo term > | term >

(5)

Note: A pbo Y > has not been defined as a string that by a non-empty
extension can be transformed into a Y > : it has also been required not
to be a Y > all by itself.

In terms of S and the notion pbo sent > we now specify the desired
net effect of sent by requiring that upon completion

S – x = < pbo sent > and

S ≠ < pbo sent > and

c = [ S = < sent > ]
(6)

The first of these terms is required to ensure that not too much has
been read, the second guarantees that enough has been read and the final
term defines the value of c. Note that specification (5) has been given
without taking syntax (1) into account! All proper initial strings of S are
of the syntactical category pbo sent > : the empty string is, and as soon
as reading has progressed such that the first two terms are true, reading
stops and c will get its proper value.

In order to proceed with (6) in which that syntactic category
pbo sent > occurs, we derive from the first line of (1)

< pbo sent > ::= < bo exp>

(7)

where bo exp > comprises all expressions and sequences that can be
extended to an expression.

(Note: According to (4) it is not excluded that a proper initial string of
an expression by itself is an expression!)

Relation (7), together with the first line of the syntax (1) enables us
to rewrite the desired net effect of sent as follows: upon termination of
sent we require

S – x = < bo exp > and

S ≠ < bo exp > and

c = [ S – x = < exp
> and semi(x)]

(8)

The second line shows that the concept bo exp > is related to the
complete string S. As a result, when we wish to express sent in terms of
a primitive exp, whose analysis has to embody the syntactical rules of an
exp > and that shall read a string E, then E and S should be identical.

In terms of a primitive exp, reading a string E, such that upon
termination

E – x = < bo exp > and

E ≠ < bo exp > and

c = [ E – x = < exp
> ]

(9)

the body

proc sent: exp; c := c and semi(x) corp

(10)

satisfies requirement (8). A formal proof of the correctness of (10) relies
on the axiom of assignment only: with post-condition (8) we derive on account
of (1) as the post-condition for its call of exp

S – x = < bo exp > and

S ≠ < bo exp > and

c and semi(x) = [ S –

x = < exp > and semi(x)]

which is obviously satisfied by E = S and specification (9). The equality
E = S is guaranteed when, prior to its call on exp, sent will not command
any move. It does not: it opens with exp.

Now we are left with the duty of constructing a recognizer exp, reading
a string E given by the first two lines of (9) and assigning to c the value
given by the last line of (9). Because (9) mentions the syntactic category
bo exp >, we had better derive its syntactic definition, now from the
second line of the syntax. Syntax (2) is the most convenient and we find
—because operator > is always a single character!—

< bo exp > ::= {< term >< operator >} < bo term >

(11)

The braces —“zero or more times”— suggest a while loop and therefore
we must try to find a convenient invariant, implied by (9). It is certainly
necessary to maintain
E – x = bo exp >
as this takes care of the requirement that not too much will be read. On
account of (11) we can then write

E = {< term >< operator >} T′

with              T′ – x = < bo term >

(12)

Equation (11) tells us that the necessary and sufficient condition for
E = bo exp > is in terms of T′, as given by (12),
T′ = bo term > or T′ = term >operator > .
The necessary and sufficient condition for E ≠ bo exp > is therefore
T′ ≠ bo term > and T′ ≠ term >operator >
or
T′ ≠ bo term > and non [ T′ – x = term > and operator(x)]
Furthermore it follows from (12) and the syntax for exp > that
E – x = exp > (from the third line of (9)) is equivalent to
T′ – x = term >. Therefore we can rewrite (9) as

Q and non [ c and operator(x)]

with Q:                E – x = < bo exp > and

T′ – x = < bo term > and

T′ ≠ < bo term > and

c = [ T′ – x =< term > ]

(13)

where we have combined in Q all assertions about the total text read by exp.
Now the invariance theorem for loops suggests a body for exp of the form

establish Q:

while c and operator(x) do something without violating Q od

Analogous to (9) we try a primitive term, such that upon completion
the string T read by it and c satisfy

T – x = < bo term > and

T ≠ < bo exp > and

c = [ T – x = < term

>]

(14)

Relations (12), (13) and (14) suggest that we try to prove that

E = {< term >< operator >}

(15)

is a sufficient precondition for term to ensure the postcondition Q. As
far as the effect on E of the primitive term is concerned, an execution
of term is equivalent to

E := E + T
(where we have used the "+" for concatenation) or
E – x := E + T – x
The post-condition E – x = bo exp > reduces thanks to the axiom of assignment
and (11) to the pre-condition
E + T – x = {term >operator >} bo term > .
Because T – x = bo term > , (15) is a sufficient precondition, allowing
the identification T′ = T, guaranteeing on account of (14) the last three
lines of Q as well. Because E = is a specific instance of (15),
the initialization of the loop —i.e. establishing Q to start with— can be
done by a single call on term. As repeatable statement term would be
acceptable, provided that we can show that
Q and c and operator(x) ⇒ relation (15)
The left-hand implies
E – x = { term >operator>} T′ – x and
T′ – x = term > and
operator(x)
and therefore (15) holds, but (15) was a sufficient pre-condition for term
not to violate Q and we have found the program
proc exp:    term; while c and operator(x) do term od corp .

We are left with the duty to construct a primitive term such that the
string T, read by it, and c will satisfy (14) upon completion. In order to
construct it, we derive from the third line of syntax (2) the syntactic rule

< bo term > ::= {< digit >} | (< bo exp > | (< exp >)

(16)

and the primary duty of term is to establish a string T such that

T – x = bo term > and

T ≠ bo term >    .
According to the syntax (16) we have three cases
A)      T – x = {digit >}

In this case x ≠ digit > on account of the requirement T ≠ bo term > ,
and c is to get the value true when T – x ≠ .
B)            T – x = (bo exp >

or, with        T = ( E
(17)

we then know that

E – x = < bo exp > and E ≠ < bo exp > .

(18)

But we know also

non [ E – x = < exp > and close(x)]

(19)

because otherwise T = bo term > (3rd alternative) would have been true. In
this case c must get the value false, because from T – x = term > would
follow E – x = exp ) and this is incompatible with E – x = bo exp >
because via mathematical induction it is easily proved that in an exp >
the number of opening brackets equals the number of closing brackets while
in a bo exp > the number of closing brackets does not exceed the number
of opening brackets.
C)      T – x = (exp >)

In this case, regardless the value of x, T ≠ bo term > holds and c must
get the value true. With
T – x = ( E

—in contrast to (17), covering case B— we have again relation (18) but also

E – x = exp > and close(x).

But this is exactly the negation of relation (19)! Thus we are led to the
following program (with comments inserted between braces)

proc term:
move;

if open(x)

then
{case B and case C} exp;

if c and close(x)

then {case C} move

else {case B} c := false

fi

else
{case A} c := digit(x);

while digit(x) do move od

fi

corp

*              *

*
Conclusion. The primary purpose of the above exercise was to demonstrate that
the applicability of the technique of developing correctness proof and program
hand in hand is not confined to simple programs dealing with algebraic relations
between integers but can also be applied to non-arithmetic data processing.
Some readers might argue that this example rather weakens my point, because
the essay is rather long compared to the length of the program. My answer to them
is the following:
1) A program need not be long in order to be difficult to conceive and also,
unless one has produced absolutely insipid code, a final program is always a
a very compact product of one's intellectual labour.
2) We have not made strong assumptions about the reader’s familiarity with
the theory of recursively defined syntaxes and therefore have derived explicitly
all we needed.
3) Anyone who thinks that he can make reliable programs without a sufficient
“theory” about the subject matter of the computations, fools himself.
Finally, although the alert reader will have noticed it, it seems worthwhile
to draw attention to the fact, that in all our reasoning in the above
exercise we have not given a single example of a sentence, nor of a non-sentence.

Eindhoven, 18th May 1973
Edsger W.Dijkstra

Transcribed by Martin P.M. van der Burgt

Last revision

2014-11-05

.
