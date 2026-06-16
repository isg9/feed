---
title: "To an anonymous referee of AvG45/EWD901 (EWD938)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD09xx/EWD938.html
published: "1985-09-22T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd09xx/EWD938.PDF
---

# To an anonymous referee of AvG45/EWD901

I find it instructive to submit now and then something for publication, as it gives me some insight in the standards other referees apply to their refereeing work. To my regret I must inform you that I did not find your report up to par. Let me explain why I hold that opinion.

Let E be defined as the strongest solution of some equation. The strongest theorem about E is that for some P we have [P ≡ E] . It is appropriate to prove this by proving separately

(i) [P ⇛ E]      and

(ii) [E ⇛ P]      ,

because E is defined by

(iii) E is a solution of the equation

(iv) E implies any solution of the equation,

and therefore

— on account of (iii) alone, (i) can be proved by showing [P ⇛ X] for any solution X, and

— on account of (iv) alone, (ii) can be proved by showing [X ⇛ P] for some solution X (e.g. by showing that P itself is a solution).

And, indeed, many nice proofs I know of [P ≡ E] prove – not surprisingly – (i) and (ii) separately – the only refinement being that for an equation of the form [f.X ≡ X] with monotonic f, one can take in (iv) thanks to Knaster-Tarski the more tolerant equation [f.X ⇛ X] .

Hence I see very little place for the proof – alluded to in your first paragraph – “which [...] required one to find the strongest fixed point”, as the moral of the above is to avoid such proofs. So what am I to do with your remark “It is therefore, I think, not too surprising, etc.”? I have only two options: either you are not surprised because you knew of a proof like ours – but failed to publish it like everybody else – or you did not think too well. So much for your first paragraph.

In the second paragraph you suggest that our “id est” is in error because “the equation is set up so that all solutions [...] correspond to termination.” Quod non. If, for instance, S is guaranteed to terminate – i.e. [ B ⋀ wp(S,X) ≡ B ⋀ wlp(S,X) ] – then wp(DO.X) and wlp(DO.X) are both solutions of the equation in X

(0)      [ X ≡ (B ⋀ wp(S,X)) ⋁ (¬B ⋀ P) ]

(viz. its strongest solution and its weakest solution respectively); hence, “partial correctness” is governed by the same equation. So here the error is yours, not ours. So much for your second paragraph.

So [sic!], in your third paragraph, you suspect that this be a note rather than a paper. Well, some journals carry that distinction, thereby creating for someone the obligation to decide whether something should be published as “note” or as “paper”. As this journal does not know “notes” versus “papers”, your suggestion is not applicable, unless it is meant to imply that you recommend rejection.

Finally, you “don’t think it’s a deep mathematical result”. For scientific journals you seem to be in favor of the latter, but then you are on dangerous grounds, for: what is meant by “deep”? Webster’s New Collegiate Dictionary (1972) gives

“difficult to penetrate or comprehend [...]

< ~ mathematical problems >” ,

thus confirming what I found years ago, when I held an inquiry on “depth” among a large number of mathematical colleagues: they all coupled the depth of a result to the complexity or conceptual sophistication of the argument needed to establish it. The danger of your ground is that you might find yourself to accept a proof based on a transfinite formalism but inclined to reject an elementary proof of the same result as being insufficiently “deep”. I once offered a short article on a problem to which lengthy papers (with errors) had been devoted; one of the referees explicitly recommended rejection of the article because the (correct!) solution it presented was “too simple to be of academic interest”. You did not go quite as far as he went, but you were dangerously close because your third paragraph does confirm the image of the mathematicians as forming a perverse guild that clings for dear life on its pompous complexities. So much for your third paragraph.

And what is the rôle of the last paragraph? (By the way, the paper has not been written for “computer people” that one can shock by indexing a bibliography from zero, but for computing scientists.) Everybody who knows the literature knows that rigorous proofs are rarily published, the usual excuse being that they are long, tedious, and boring, and that, therefore, mathematicians are not interested in them and prefer to waive their hands. (See, for instance, “The Development of Mathematics” by E. T. Bell.) The calculational proof method we employed, however, combined rigour with brevity. What is wrong with expressing the hope that this will be noticed? Or do you wish to warn the editor that many readers might dislike their usual excuse for their hand-waving being invalidated? Let us distinguish the salesman from the scientist: the task of the latter is not the please his clientele but to enlighten his fellowmen (whether they like that or not). So much for your last paragraph.

So you have confined yourself to being vaguely negative on flimsy or erroneous grounds. You have not given a single suggestion for improvements, you have abstained from all comments on the proof itself (have not even confirmed to the editor that it is correct and very complete) or on the adequacy of the introduction. The overall impression is that you couldn’t be bothered.

Austin, 22 September 1985

prof. dr. Edsger W. Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

Appendix

Report on “A simple fix-point argument without the restriction to continuity” by Edsger W. Dijkstra and A.J.M. van Gasteren

This paper gives a proof for the main theorem about Dijkstra’s repetitive construct which, “though not relying on or-continuity, does not require transfinite formalism”. It does so by defining the appropriate weakest precondition as the strongest solution of a particular equation (relying on Tarski’s well-known theorem to guarantee the existence of such a solution), and showing that the required property holds of any solution of that equation. That is to say, the proof does not depend on the solution’s being the strongest, or any other particular solution. It is therefore, I think, not too surprising that transfinite induction is not required – it would be more remarkable on a proof which, in the absence of continuity, required one actually to find the strongest fixed point.

The proof itself uses an induction over a general well-founded set (the set of possible values for the repetitive construct’s variant), and the authors claim that the argument is “the first one to connect well-foundedness in its full generality to a non-operational notion of termination, i.e. to the strongest solution of a fit-point equation”. I think the “id est” is in error there – the equation is set up so that all the solutions are satisfactory (that is, correspond to termination), so that the strongest is not distinguished. I suppose it’s a manifestation of the fact that once you’ve confined yourself to problems for which you can give a variant, proofs of termination are easy (I would agree with EWD that those are the problems to which programmers ought to be confining themselves).

So I suspect that this ought to be a note, rather than a paper. It’s useful to point out to computer people that this theorem does not depend on continuity as the proof in A Discipline of Programming perhaps suggests. But I don’t think it’s a deep mathematical result.

The authors also hope that the reader will “regard the effectiveness and austere rigour of [their] argument as a plea for the calculational proof method employed”. Well, that’s a matter of taste, on which people will make up their own minds. But I suspect that many computer people are put off by the very austerity (even the bibliography is indexed from zero), while mathematicians seem to expect that the mountains should not usually be required to labour in this very formalistic kind of way.

transcribed by Jonathan David Arndt

revised
01-Mar-2022
