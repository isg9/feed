---
title: "The Mathematical Divide (EWD1268)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1268.html
published: "1997-12-10T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1268.PDF
---

# The Mathematical Divide

EWD 1268

The Mathematical Divide

This note uses the simplest example I could think of to illustrate an old
issue —a bone of contention, one might say— that tends to divide the mathematical world. I shall present two proofs of the well-known equality.

(0)        (a+b)2 = a2 + 2∙a∙b + b2

We give the intuitive proof first:

<∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙ a + b ∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙>

<∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙ a ∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙∙>

a2

<∙∙∙ b ∙∙∙>

a∙b

a∙b

b2

The lefthand side of (0) represents the area of the undivided big square; the righthand side represents the sum of the areas of the four parts as in the above picture. In short: equality (0) stares you in the face!

Now we turn to the calculational proof. As usual, we shall exploit the associativity of the addition —i.e. the equality

(p + q) + r = p + (q + r )        for all p, q, r   —

implicitly by omitting such semantically redundant parenthesis. Here we go! For any a, b,

(a + b)2

=        { definition of exponentiation }

(a + b) ∙ (a + b)

=        { distribution p∙(q + r ) = p∙q + p∙r }

(a + b) ∙ a + (a + b) ∙ b

=        { distribution (p + q) ∙ r = p∙r + q∙r, twice }

a∙a + b∙a + a∙b + b∙b

=        { symmetry of ∙ : p∙q = q∙p }

a∙a + a∙b + a∙b + b∙b

=         { definition of exponentiation twice; property of 2: p + p = 2∙p }

a2 + 2∙a∙b + b2

(In the very last line —as in (0)— we have implicitly used that also multiplication is associative.) Compared with the calculational proof, our former intuitive proof is seductively simple, but, though a great lover of simplicity, I think that the temptation should be resisted.

The traditional objection to the intuitive, pictorial arguments is that they raise more questions than they answer because pictures are always over specific. In our example, for instance, we have in the picture the case 0<b<a. Does (0) also hold when 0<a<b? When b<0<a? Or, more precisely, when 0 < a+b < a and when a+b < 0 < a? The intuitive argument does not answer these questions. Because its silent assumptions obscure what is purportedly demonstrated, we can question whether the intuitive argument should be considered a proof at all.

With respect to the above questions, the calculational proof does much better: the transformations used are value-preserving, whatever the signs involved, and, hence, the calculational argument demonstrates at one fell swoop that (0) is valid for all possible sign combinations.

There is one further advantage of this calculational proof. And that concerns the question whether (0) also holds if a, b are matrices. (Not an unreasonable question, for matrices can be added, squared, and multiplied, just like real numbers.) To the answer of this question, the intuitive proof cannot contribute at all. The calculational proof immediately tells us that in general, (0) does not hold for matrices since matrix multiplication is not symmetric (i.e. in general, p∙q ≠ q∙p for matrices p, q). In the intuitive argument, (0)'s dependence on symmetry of multiplication is swept under the rug, and the intuitive argument is error-prone, because its apparent simplicity has been obtained by committing the sin of omission.

So much for the nature of the Mathematical Divide.

*                *

*

Each side of this Mathematical Divide has its fervent advocates. Pictorial arguments are praised for being natural and intuitive, for being short and simple, for providing insight, and for aiding and guiding in the discovery of new mathematics; finally, pictures are deemed less error-prone than calculations. Calculational arguments are praised for their rigour, for their completeness, for their brevity, and lately for their amenability to mechanical verification or even generation; finally, calculations are deemed less error-prone than pictorial arguments.

To the discussion of the pros and the cons I would like to add two remarks, because I do not notice them being made while I do think they are relevant.

The first remark is that, no matter how surprising, tickling, or charming "intuitive proofs" may be, we do not know how to teach their design, this in contrast to calculation, whose design to a very large extent can be taught.

When this alien resident of the USA sees a publisher's catalogue of mathematical textbooks, he cannot fail to notice that "intuitive" occurs as an almost universal recommendation. But these blurbs make him very doubtful, for he knows his Concise Oxford Dictionary, which describes "intuition" as

"immediate apprehension by the mind, without reasoning"

which almost forces one to conclude that either the thing apprehended is totally trivial or the apprehension does not amount to very much.

Another thing this alien resident of the USA cannot fail to notice [is] how the American mathematical community is obsessed with mathematical pedagogy and educational reform. I am not making this up. In the Fall, 1997, Newsletter of the Texas Section of the Mathematical Association of America, the Department of Mathematics and Statistics of Texas Tech University, Lubbock, TX 79409, invites applications for two tenure track assistant professor positions. So far, so good, but the 1st (!) requirement starts with

"The applicant must show strong evidence of research potential in collegiate level mathematics education."

(Anyone interested?) This pedagogical obsession is not just a modern product of Political Correctness ("How to keep girls and other minorities in the mathematical fold", etc.), it already showed its ugly face a hundred years ago, when academic mathematics was hardly established in the USA. I quote

"By 1897, the University [of Chicago] had promoted Jacob Young to an assistant professorship specifically in mathematical pedagogy."

(from Karen Hunger Parshall and David E. Rowe, "The Emergence of the American Mathematical Research Community, 1876–1900: J.J. Sylvester, Felix Klein, and E. H. Moore"). This was shortly after the period that American mathematics had been heavily influenced by Felix Klein, an ardent advocate of intuition and visualization ("Anschauung"). Could it be that America's attraction to the unteachable form of doing mathematics and its persistent but ineffective obsession with mathematical pedagogy are related?

My second remark concerns the extent to which the problems of computing, programming, and digital systems design in general are amenable to mathematical treatment. The answer is "Hardly" when we identify the doing of mathematics with its intuitive, pictorial style, because the latter does not scale up. To return to our earlier example, a pictorial proof of

(a + b)3 = a3 + 3∙a2∙b + 3*a∙b2 + b3

has lost most of the attraction of simplicity, while a pictorial proof of the expansion of (a + b)4 baffles the imagination. Also in a nonpictorial setting, "immediate apprehension by the mind, without reasoning" does not scale up.

But computing's successful designs are highly sophisticated logical constructs; they are unavoidably large, and a major design concern is always to structure them in such a way that the amount of reasoning needed for their justification remains doable. The use of techniques that do scale up is an essential aspect, which, for instance, has been addressed explicitly under the heading "compositionality". We are getting more and more successful at learning how to keep sophisticated, ambitious systems under control, but invariably (and not surprisingly) this requires the kind of formal calculations Felix Klein disliked and attention to detail he despised.

You see, another thing this alien resident of the USA could not fail to notice was the strong scepticism evoked over here by his —I thought eloquent and convincing— pleas for a more mathematical approach to computing science and programming. An implication of that story is that, in the USA, the term "mathematics" has a meaning that differs from the one I grew up with. I am getting a feeling for how that divergence could occur; the role of education, which differs from country to country, has a lot to do with it.

*                    *

*

The Mathematical Divide also manifests itself in what is considered the role of proofs. At the intuitive side, proofs have to be convincing; at the calculational side, proofs have to be correct.

At the intuitive side there is a whole "philosophy" of mathematics —I guess it has a name, but I don't know it— that pushes mathematics as a social activity: a proof is acceptable if enough people accept it. (Names like Imre Lakatos, Philip J. Davis and Reuben Hersch come to mind.) It is an attitude very similar to the one, not unpopular in software engineering circles, for which "user satisfaction" is the primary quality criterion for a piece of software. The problem with the goal of "user satisfaction", however, is that it provides no technical guidance to the program designer.

In programming, the answer has been the introduction of the functional specification as the embodiment of a separation of concerns: the functional specification acts as a logical firewall between what are now known as the pleasantness problem and the correctness problem. The pleasantness problem deals with the question how satisfactory a system meeting a given functional specification would be; the correctness problem deals with the question whether a given system meets that functional specification. The correctness problem being an entirely technical one, the functional specification can provide strong heuristic guidance for the system designer. (This separation of concerns has provoked strong objections; correctness concerns have been blamed for stifling the programmer's creativity .....).

In the last 120 years or so, an analogous development has taken place in mathematics, where the wooly quality criterion of "convincing enough of your fellow mathematicians" has been replaced by the requirement that the result be derived, according to the rules of an explicitly stated logic, from explicitly stated postulates. Logic and postulates act as the firewall, shielding the proof-designing mathematician from the psychological preferences of his colleagues. And again the formalization provides strong heuristic guidance.

The moral of this story is that the introduction of the functional specification in programming is strongly analogous to, and has undoubtedly been inspired by the transition from the intuitive to the calculational side of the Mathematical Divide. That example is one more debt we owe the mathematical culture, but I should add that not all computer scientists share such feelings of gratitude. The title of the infamous 1977-paper by DeMillo, Lipton and Perlis "Social Processes and Proofs of Theorems and Programs" is revealing. Staying firmly at the intuitive side of the Mathematical Divide, they argue the futility of trying to treat programs as mathematical objects and boldly postulate: "... the transition between specification and program must be left unformalized".

I cannot resist to quote E. T. Bell (The Development of Mathematics, 1940, 1945) on the topic. He leaves no doubt as to which side of the Mathematical Divide has his preference.

"Mathematicians and scientists of the conservative persuasion may feel that a science constrained by an explicitly formulated set of assumptions has lost some of its freedom and is almost dead. Experience shows that the only loss is denial of the privilege of making avoidable mistakes in reasoning. As is perhaps but humanly natural, each new encroachment of the postulational method is vigorously resisted by some as an invasion of hallowed tradition. Objection to the method is neither more nor less than objection to mathematics."

Amen.
Austin, 10 December 1997

prof. dr. Edsger W. Dijkstra
Department of Computer Sciences
The University of Texas at Austin
Austin, TX 78712-1188
USA

transcribed by Kevin Beckwith Bush

revised Mon, 10 Dec 2007
