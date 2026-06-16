---
title: "Finding the correctness proof of a concurrent program (EWD640a)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD640a.html
published: "1975-05-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD640a.PDF
---

# Finding the correctness proof of a concurrent program

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing
transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its
accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Finding the correctness proof of a concurrent program.
Introduction.
In this paper we want to do more than just giving another —be it
unusual— example of the utility of the first-order predicate calculus in
proving the correctness of programs. In addition we want to show how,
thanks to a systematic use of the first-order predicate calculus, fairly
general —almost “syntactic”— considerations about the formal manipulations
involved can provide valuable guidance for the smooth discovery of an
otherwise surprising argument.
For proofs of program correctness two fairly different styles have
been developed, “operational” proofs and “assertional” proofs. Operational
correctness proofs are based on a model of computation, and the corresponding
computational histories are the subject matter of the considerations. In
assertional correctness proofs the possibility of interpreting the program
text as executable code is ignored and the program text itself is the subject
matter of the formal considerations.
Operational proofs —although older and, depending on one’s education,
perhaps more “natural” than_assertional proofs— have proved to be tricky to
design. For more complicated programs the required classification of the
possible computational histories tends to lead to an exploding case analysis
in which it becomes very clumsy to verify that no possible sequence of events
has been overlooked, and it was in response to the disappointing experiences
with operational proofs that the assertional style has been developed.
The design of an assertional proof —as we shall see below— may
present problems, but, on the whole, experience seems to indicate that assertional
proofs are much more effective than operational ones in reducing the gnawing
uncertainty whether nothing has been overlooked. This experience, already
gained while dealing with sequential programs, was strongly confirmed while
dealing with concurrent programs: the circumstance that the ratios of the
speeds with which the sequential components proceed is left undefined greatly
increases the class of computational histories that an operational argument
would have to cover!

In the following we shall present the development of an assertional
correctness proof of a program of N-fold concurrency. The program has been
taken from the middle of a whole sequence of concurrent programs of
increasing complexity —the greater complexity at the one end being the consequence
of finer grains of interleaving— . For brevity‘s sake we have selected here
from this sequence the simplest item for which the assertional correctness
proof displays the characteristic we wanted to show. (It is got the purpose
of this paper to provide supporting material in favour of the assertional
style: in fact, our example is so simple that an operational proof for it
is still perfectly feasible.)
*              *
*

In the following y denotes a vector of N components y[i] for
0 ≤ i < N . With the identifier f we shall denote a vector-valued function
of a vector-valued argument, and the algorithm concerned solves the equation

y = f(y)(1)

or, introducing f0, f1, f2,... for the components of f

y[i] = fi(y) for 0 ≤ i < N .(2)

It is assumed that the initial value of y and the function f are
such that repeated assignments of the form

<y[i]== fi(y) >(3)

will lead in a finite number of steps to y being a solution of (1). In
(3) we have used Lamport’s notation of the angle brackets: they enclose
“atomic actions” which can be implemented by ensuring between their
executions mutual exclusion in time. For the sake of termination we assume that
the sequence of i-values for which the assignments (3) are carried out is
(the proper begin of) a sequence in which each i-value occurs infinitely
often. (We deem this property guaranteed by the usual assumption of “finite
speed ratios”; he who refuses to make that assumption can read the following
as a proof of partial correctness.)
For the purpose of this paper it suffices to know that functions f
exist such that with a proper initial value of y equation (1) will be solved

by a finite number of assignments (3). How for a given function f and
initial value y this property can be established is ggt the subject of
this paper. (He who refuses to assume that the function f and the initial
value of y have this property is free to do so: he can, again, read the
following as a proof of partial correctness that states that when our
concurrent program has terminated, (1) is satisfied.)
Besides the vector y there is —for the purpose of controlling
termination— a vector h , with boolean elements h[i] for 0 ≤ i < N , all
of which are true to start with. We now consider the following program of
N-fold concurrency, in which each atomic action assigns a value to at most
one of the array elements mentioned. We give the program first and shall
explain the notation afterwards.
The concurrent program we are considering consists of the following
N components (0 ≤ i < N):

cpnti:

L0:
do < (E j: h[j]) > →

L1:
< if y[i] = fi(y) → h[i]:= false >

▯ y[i] ≠ fi(y) → y[i]:= fi(y) > ;

L2j:
(A j: < h[j]:= true > )

fi

od

In line L0 , “(E j: h[j])” is an abbreviation for

(E j: 0 ≤ j < N: h[j]) ;

for the sake of brevity we shall use this abbreviation throughout this paper.
By writing < (E j: h[j]) > in the guard we have indicated that the inspection
whether a true h[j] can be found is an atomic action.
The opening angle bracket “ < ” in L1 has two corresponding closing
brackets, corresponding to the two “atomic alternatives”; it means that in
the same atomic actions the guards are evaluated and either “h[i]:= false”
or “y[i]:= fi(y)” is executed. In the latter case, N separate atomic
actions follow, each setting an h[j] to true: in line L2j we have used

the abbreviation ”(A j: < h[j]:= true > )“ for the program that performs
the N atomic actions < h[0]:= true > through < h[N-1]:= true > in some
order which we don’t specify any further.
In our target state y is a solution of (1), or, more explicitly

(A j: y[j] = fj(y)) .(4)

holds. We first observe that (4) is an invariant of the repeatable statements,
i.e. once true it remains true. In the alternative constructs always the
first atomic alternative will then be selected, and this leaves y , and
hence (4) unaffected. We can even conclude a stronger invariant

non (E j: h[j]) and (A j: y[j] = fj(y))(5)

or, equivalently

(A j: non h[j]) and (A j: y[j] = fj(y))(5’)

for, when (5) holds, no assignment h[i]:= false can destroy the truth of
(A j: non h[j]) . when (4) holds, the assumption of finite speed ratios
implies that within a finite number of steps (5) will hold. But then the
guards of the repetitive constructs are false, and all components will terminate
nicely with (4) holding. The critical point is: can we guarantee that none
of the components terminates too soon?
We shall give an assertional proof, following the technique which has
been pioneered by Gries and Owicki [1]. We call an assertion ”universally
true“ if and only if it holds between any two atomic actions —i.e. ”always“
with respect to the computation, ”everywhere“ with respect to the text— .
More precisely: proving the universal truth of an assertion amounts to showing

1)     that it holds at initialization

2)     that its truth is an invariant of each atomic action.
In order to prove that none of the components terminates too soon, i.e.
that termination implies that (4) holds, we have to prove the universal truth of

(E j: h[j]) or (A j: y[j] = fj(y)) .(6)

Relation (6) certainly holds when the N components are started because
initially all h[j] are true. We are only left with the obligation to
prove the invariance of (6); the remaining part of this paper is devoted
to that proof, and to how it can be discovered.

We get a hint of the difficulties we may expect when trying to prove
the invariance of (6) with respect to the first atomic alternative of L1:

< y[i] = fi(y) → h[i]:= false >

as soon as we realize that the first term of (6) is a compact notation for

h[0] or h[1] or h[2] or ... or h[N-1]

which only changes from true to false when, as a result of “h[i]:= false”
the last true h[j] disappears. That is ugly!
We often prove mathematical theorems by proving a stronger —but,
somehow, more manageable— theorem instead. In direct analogy: instead
of trying to prove the invariant truth of (6) directly, we shall try to
prove the invariant truth of a stronger assertion that we get by replacing
the conditions y[j] = fj(y) by stronger ones. Because non R is stronger
than Q provided (Q or R) holds, we can strengthen (6) into

(E j= h[j]) or (A j: non Rj)(7)

provided

(A j: y[j] = fj(y) or Rj)(8)

holds. (Someone who sees these heuristics presented in this manner for the
first time may experience this as juggling, but I am afraid that it is quite
standard and that we had better get used to it.)
What have we gained by the introduction of the N predicates Rj ?
Well, the freedom to choose them! More precisely: the freedom to define
them in such a way that we can prove the universal truth of (8) —which is
structurally quite pleasant— in the usual fashion, while the universal truth
of (7) —which is structurally equally “ugly” as (6)— follows more or less
directly from the definition of the Rj’s : that is the way in which we
may hope that (7) is more “manageable” than the original (6).
In order to find a proper definition of the Rj’s, we analyse our
obligation to prove the invariance of (8).
If we only looked at the invariance of (8), we might think that a
definition of the Rj’s in terms of y :

Rj = (y[j] ≠ fj(y))

would be a sensible choice. A moment’s reflection tells us that that
definition does not help: it would make (8) universally true by definition,
and the right-hand terms of (6) and (7) would be identical, whereas under the
truth of (8), (7) was intended to be stronger than (6).
For two reasons we are looking for a definition of the Rj’s in which
the y does not occur: firstly, it is then that we can expect the proof of
the universal truth of (8) to amount to something —and, thereby, to contribute
to the argument— , secondly, we would like to conclude the universal truth
of (7) —which does not mention y at all!— from the definition of the
Rj’s . In other words, we propose a definition of the Rj’s which does not
refer to y at all: only with such a definition does the replacement of
(6) by (7) and (8) localize our dealing with y completely to the proof
of the universal truth of (8).
Because we want to define the Rj’s independently of y , because
initially we cannot assume that for some j-value y[j] = fj(y) holds, and
because (8) must hold initially, we must guarantee that initially

(A j: Rj)(9)

holds. Because, initially, all the h[j] are true, the initial truth of
(9) is guaranteed if the Rj’s are defined in such a way that we have

(E j: non h[j]) or (A j: Rj) .(10)

We observe, that (10) is again of the recognized ugly form we are trying to
get rid of. We have some slack —that is what the Rj’s are being
introduced for— and this is the moment to decide to try to come away with a
stronger —but what we have called: “structurally more pleasant”— relation
for the definition of the Rj’s , from which (10) immediately follows. The
only candidate I can think of is

(A j: non h[j] or Rj)(11)

and we can already divulge that, indeed, (11) will be one of the defining
equations for the Rj’s .
From (11) it follows that the algorithm will now start with all the

Rj’s true. From (8) it follows that the truth of Rj can be appreciated
as “the equation y[j] = fj(y) need not be satisfied”, and from (7) it follows
that in our final state we must have all the Rj’s equal to false.
Let us now look at the alternative construct
L1:
<if y[i] = fi(y) → h[i]:= false >

▯ y[i] ≠ fi(y) → y[i]:= f(y) >;

L2j:
(A j: < h[j]:= true >)

fi

We observe that the first alternative sets h[i] false, and that the second
one, as a whole, sets all h[j] true. As far as the universal truth of (11)
is concerned, we therefore conclude that in the first alternative Ri is
allowed to, and hence may become false, but that in the second alternative as a
whole, all Rj's must become true.
Let us now confront the two atomic alternatives with (8). Because,
when the first atomic alternative is selected, only y[i] = fi(y) has been
observed, the universal truth of (8) is guaranteed to be an invariant of the
first atomic alternative, provided it enjoys the following property (12):
In the execution of the first atomic alternative

< y[i] = fi(y) → h[i]:= false >

no Rj for j ≠ i changes from true to false.(12)

Confronting the second atomic alternative

< y[i] ≠ fi(y) → y[i]:= fi(y) >

with (8), and observing that upon its completion none of the relations
y[j] = fj(y) needs to hold, we conclude that the second atomic alternative
itself must already cause a final state in which all the Rj’s are true,
in spite of the fact that the subsequent assignments h[j]:= true —which
would each force an Rj to true on account of (11)— have not been executed
yet. In short: in our definition for the Rj’s we must include besides
(11) another reason why an Rj should be defined to be true.
As it stands, the second atomic alternative only modifies y but we
had decided that the definition of the Rj’s would not be expressed in terms

of y ! The only way in which we can formulate the additional reason for an
Rj to be true is in terms of an auxiliary variable (to be introduced in a
moment), whose value is changed in conjunction with the assignment to y[i] .
The value of that auxiliary variable has to force each R, to true until the
subsequent assignment < h[j]:= true > does so via (11). Because the second,
atomic alternative is followed by N subsequent, separate atomic actions
< h[j]:= true > —one for each value of j — , it stands to reason that we
introduce for the i-th component cpnti an auxiliary local boolean array
si with elements si[j] for 0 ≤ j < N . Their initial (and “neutral”)
value is true. The second atomic alternative of L1 sets them all to false,
the atomic statements L2j will reset them to true one at a time.
In contrast to the variables y and h , which are accessible to
all components —which is expressed by calling them “global variables”— ,
each variable si is only accessible to its corresponding component cpnti
—which is expressed by calling the variable si “local” to component
as is usual in annotating or verifying sequential programs. If a local
assertion contains only local variables, it can be justified on account of
the text of the corresponding component only.
Local variables give rise to so-called "local assertions". Local
assertions are most conveniently written in the program text of the
individual components at the place corresponding to their truth: they state
a truth between preceding and succeeding statements in exactly the same way
as is usual in annotating or verifying sequential programs. If a local
assertion contains only local variables, it can be justified on account of
the text of the corresponding component only.
In the following annotated version of cpnti we have inserted local
assertions between braces. In order to understand the local assertions about
si it suffices to remember that si is local to cpnti . The local
assertion {Ri} in the second atomic alternative of Ll is justified by
the guard y[i] ≠ fi(y). in conjunction with (8). We have further
incorporated in our annotation the consequence of (12) and the fact that the
execution of a second alternative will never cause an Rj to become false:
a true Rj can only become false by virtue of the execution of the first
alternative of Ll by cpnti itself! Hence, Ri is true all through the
execution of the second alternative of cpnti .

cpnti:
L0:
do < (E j: h[j]) >→ {(A j: si[j])}

L1:
<if y[i] = fi(y) → h[i]:= false > {(A j: si[j])}

▯ y[i] ≠ fi(y) →

{Ri} y[i]:= fi(y);

(A j: si[j]:= false) > {Ri and (A j: non si[j])};

(A j:{Ri and non si[j]} < h[j]:= true; si[j]:= true ≠ )

L2j:
fi {(A j: si[j])} .

od

On account of (11) Rj will be true upon completion of L2j . But
the second atomic alternative of Ll should already have made Rj true,
and it should remain so until L2j is executed. The precondition of L2j,
as given in the annotation, hence tells us the “other reason besides

(A j: non h[j] or Rj)(11)

why an Rj should be defined to be true”:

(A i, j: non Ri or si[j] or Rj)�(13)

Because it is our aim to get eventually all the Rj's false, we define
the Rj's as the minimal solution of (11) and (13), minimal in the sense
of: as few Rj's true as possible.
The existence of a unique minimal solution of (11) and (13) follows
from the following construction. Start with all Rj's false —all equations
of (13) are then satisfied on account of the term “non Ri” — . If all
equations of (11) are satisfied as well, we are ready —no true Rj's at
all— ; otherwise (11) is satisfied by setting Rj to true for all j-values
for which h[j] holds. Now all equations of (11) are satisfied, but some
of the equations of (13) need no longer be satisfied: as long as an (i, j)-
pair can be found for which the equation of (13) is not satisfied, satisfy it
by setting that Rj to true: as this cannot cause violation of (11) we
end up with the Rj's being a solution of (11) and (13). But it is also
the minimal solution, because any Rj true in this solution must be true
in any solution.
For a value of i , for which

(A j: si[j])(14)

holds, the above construction tells us that the truth of Ri forces no
further true Rj’s via (13); consequently, when such an Ri becomes false,
no other Rj-values are then affected. This, and the fact that the first
atomic alternative of L1 is executed under the truth of (14) tells us,
that with our definition of the Rj’s as the minimal solution of (11) and (13),
requirement (12) is, indeed, met.
We have proved the universal truth of (8) by defining the Rj’s as
the minimal solution of (11) and (13). The universal truth of (7), however,
is now obvious. If the left-hand term of (7) is false, we have

(A j:non h[j]),

and (11) and (13) have as minimal solution all Rj’s false, i.e.

(A j: non Rj),

which is the second term of (7). From the universal truth of (7) and (8),
the universal truth of (6) follows, and our proof is completed.
Concluding remarks.
This note has been written with many purposes in mind:

1)To give a wider publicity to an unusual problem and the mathematics
involved in its solution.

2)To present a counterexample contradicting the much-propagated and hence
commonly held belief that correctness proofs for programs are only laboriously
belabouring the obvious.

3)To present a counterexample to the much-propagated and hence commonly
held belief that there is an antagonism between rigour and formality on the
one hand and “understandability” on the other.

4)To present an example of a correctness proof in which the first-order
predicate calculus is used as what seems an indispensable tool.

5)To present an example of a correctness proof in which the first-order
predicate calculus is a fully adequate tool.

6)To show how fairly general —a1most “syntactic”—_considerations about
the formal manipulations involved can provide valuable guidance for the
discovery of a surprising and surprisingly effective argument, thus showing how
a formal discipline can assist “creativity” instead of —as is sometimes
suggested— hampering it.

7)To show how also in such formal considerations the principle of
separation of concerns can be recognized as a very helpful one.

I leave it to my readers to form their opinion whether with the above
I have served these purposes well.
Acknowledgements. I would like to express my gratitude to both IFIP WE2.3 and
“The Tuesday Afternoon Club”, where I had the opportunity to discuss this
problem. Those familiar with the long history that led to this note, however,
know that in this case I am indebted to C.S.Scholten more than to anyone else.
Comments from S.T.M.Ackermans, David Gries, and W.M.Turski on an earlier version
of this paper are greatfully acknowledged.
[1] Owicki, Susan and Gries, David, “Verifying Properties of Parallel
, Programs: An Axiomatic Approach”. Comm.ACM 19, 5 (May 1975), pp.279-285.

Plataanstraat 5prof.dr.Edsger W.Dijkstra

5671 AL NUENENBurroughs Research Fellow

The Netherlands.

Transcribed by Martin P.M. van der Burgt

Last revision

2015-04-10

.
