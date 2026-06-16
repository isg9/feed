---
title: "On the design of a simple proof for Morley’s Theorem (EWD1050)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD10xx/EWD1050.html
published: "1989-04-23T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd10xx/EWD1050.PDF
---

# On the design of a simple proof for Morley’s Theorem

EWD 1050

On the design of a simple proof for Morley’s Theorem

The general opinion —with which I concur— is that Frank Morley’s Theorem about the angle trisectors of a triangle is a geometrical curiosity that is of historical interest at best. Independently of the (in)significance of the theorem proved by it, a proof may deserve our attention, for instance, by virtue of its structure, its simplicity, or its brevity. For the proof to be shown below, such a claim can be made: requiring no auxiliary lines or points, it is so simple that the theorem’s late discovery (1899) and the elapsed decade before the first proofs were published (1909) become the more striking. When I found this proof years ago, I was only too willing to ascribe that discovery to my great ingenuity and all that. The purpose of this note, however, is to show how, in the mean time, the art and science of proof design have advanced to a stage in which the design of such proofs almost become a routine exercise, requiring the usual care in arrangement and notation, but a minimum of invention.

Consider a plane figure —consisting of 6 distinct points and 12 straight line segments— of the following configuration —

Morley’s Theorem states that △XYZ is equilateral if, in this figure, the angles of △ABC are trisected.

Note that there are all sorts of nonredundant ways of defining that a triangle is equilateral, such as

(0)   its edges are of equal length

(1)   its angles are of equal size

(2)   it is isosceles, and its top angle equals 60�.

Definition (1) makes it clear that Morley’s Theorem can be formulated in terms of angles only, i.e. that in the above figure we can isolate size from shape.

Let us call the above configuration with the angles of △ABC trisected a Morley Figure. The simplest way of giving the shape of a Morley Figure independently of its size is to give the angles of △ABC. The simplest way of giving the size of a Morley Figure independently of its shape is to give the size of △XYZ.

This is a very nice disentanglement. To do justice to it, we propose to prove Morley’s Theorem by showing

For any triple (α,β,γ ) of positive angles whose sum equals 60� and for any equilateral triangle, one can erect on the three sides of the equilateral triangle triangles with top angles α, β, γ respectively so that the angles of the triangle formed by their tops are trisected.

The existence of these three triangles will be demonstrated by showing how to erect them. To this end we shall derive how to erect them. We exploit the cyclic symmetry (which we have been careful not to destroy) in order to avoid repetition, and focus our attention on side AB

We have marked with α ! and β ! the two target sizes, with α and β the two given sizes, and in view of the many alternative definitions we have left open how we are going to take into account that △XYZ is equilateral. In the above picture, we can draw our immediate conclusion from the target sizes: ∠AZB = 180�−α −β. Because the remainder of the picture has to be taken into account, we have to conclude that its full complement equals 180�+α +β. Now it stands to reason to draw XZ and YZ and to take into account that ∠XZY = 60�

From ∠XZY = 60� we thus conclude

(α +β = the sum of the angles marked 1) ≡

(120�+α +β = the sum of the angles marked 2)   .

A prescribed sum of two values gives a one-dimensional degree of freedom, for which we now have to find the most suitable parameterization. For the angles marked 1, the simplest parameterization is

,

since that translates our target into φ = 0. For reasons of symmetry it stands to reason to split 120�+α +β into 60�+α +ψ and 60�+α −ψ, but which is which? Because of α +β +γ=60�, we get the nicest fit if we allocate 60�+α +ψ in the triangle with top β. Thus we find ourselves contemplating the following parameterized figure:

Were we to complete it, we would find in clockwise direction 60�+α +ψ, 60�+β −ψ, 60�+γ +ψ, 60�+α −ψ, 60�+β +ψ, 60�+γ −ψ. Our remaining duty is to show that φ =0 follows.

Looking at our last diagram, we should realize that, so far, we have used from geometry no more than the fact that the angles of a triangle add up to 180�. We should be willing to use a little bit more so as to be able to exploit that the three triangles share two sides. Parameter φ (for which =0 has to be concluded) occurs only twice in the above picture, more precisely, in △AZB opposite to the shared sides. So we are looking for a fact from geometry that links angles of a triangle to their opposite sides. Here the Rule of Sines is supposed to come to the reader’s mind. Furthermore, of the fact that △XYZ is equilateral, so far we used only ∠XZY = 60� and thus can expect to have to use XZ = YZ.

So, let us start with the expression in φ that we can manipulate with the Rule of Sines

sin(α +φ)

sin(β −φ)

=   {Rule of Sines in △AZB}

BZ

AZ

=   {Rule of Sines in △BZX/△AZY so as to introduce XZ/YZ}

sin(60�+γ −ψ) • XZ / sin(β )

sin(60�+γ +ψ) • YZ / sin(α)

=   {XZ=YZ}

sin(60�+γ −ψ) � sin(α)

sin(60�+γ +ψ) � sin(β )          ,

In summary

(3)     sin(α +φ)  =  sin(60�+γ −ψ)  •  sin(α)

sin(β −φ)sin(60�+γ +ψ)sin(β )

Now the question has become: can we substitute in (3) such a value for ψ that φ =0 follows? Substituting into (3) φ :=0 yields

(4)     sin(60�+γ −ψ)   = 1         ,

sin(60�+β +ψ)

a solvable equation (e.g. ψ =0). Substituting (4) into (3), we get

sin(β ) • sin(α +φ) = sin(α) • sin(β −φ)

=       {addition formula for sin}

sin(β ) • (sin(α)•cos(φ) + cos(α)•sin(φ)) =

sin(α) • (sin(β )•cos(φ) − cos(β )•sin(φ))

=       {algebra}

(sin(β )•cos(α) + sin(α)•cos(β )) • sin(φ) = 0

=       {addition formula for sin}

sin(α +β ) • sin(φ) = 0

=       {0 < α +β < 60�}

sin(φ) = 0

=       { α +φ ≥ 0 ⇒ φ > −60�; β −φ ≥ 0 ⇒ 60� > φ }

φ =0   ,

which concludes the proof. (To solve the last equation with the addition formula for the sine, rather than via a monotonicity argument, was suggested to me by Ambuj Kumar Singh.)

*         *         *

For a traditional presentation of a geometric proof, see [0] —Coxeter amazingly contrasts “a trigonometrical proof” with “an elementary proof”— ; for a compact presentation of the above trigonometrical proof, see [1] (where the proof is reduced to almost a one-liner). This note is a critical comment on such proof presentations, which are correct, but rather uninstructive.

Coxeter starts his proof with “Much trouble is experienced if we try a direct approach, but the difficulties disappear if we work backwards, beginning with an equilateral triangle and building up a general triangle which is afterwards identified with the given triangle ABC.”. Coxeter at least mentions the “trouble” —be it as a fact learned from unanalysed experience gained from trying to prove Morley’s Theorem—; Dijkstra does not even mention the trouble and just announces “We start in our proof not with the arbitrary triangle, but with the equilateral one.”

They both pull the same big rabbit out of their mathematical hat. Above we showed how this rabbit can be avoided. In order not to interrupt the flow of the presentation, we mentioned the catchword “disentanglement” —see [2]—; here is the place to point out that it refers to a general principle. The idea is to write in particular the information to be used as a conjunction of as many independent terms as possible because this is a precondition for subsequently being able to indicate precisely what is appealed to where. (If a datum can be factored differently —such as “△XYZ is equilateral”— the advice is to list the options and to postpone the choice until it emerges which one is appropriate.) The decision not to fix the size of the Morley Figure by, say, the length of AB rests on the principle —see [2]— not to destroy symmetry lightly.

The second big rabbit that [0] and [1] share is the choice of very special angles that new lines make with the edges of the equilateral triangle. The above analysis, based on angles alone, shows that we are essentially left with one degree of freedom, viz. the relative orientation of the two triangles; hence ψ. The magician’s proof immediately substitutes for ψ its proper value (so that ψ need not be mentioned at all), thus suggesting that without such spark of genius one cannot find such proofs. It is good to know that a whole class of such sparks are superfluous: in case of freedom, capture that freedom by the introduction of as yet undetermined parameters, which, if not truly arbitrary, will be later constrained by the needs of the subsequent proof.

The third big rabbit in [0] and [1] is the absence of any motivation for the choice of mathematical fact to exploit. In [0], out of the blue Coxeter refers to a theorem about the incenter of a triangle that I was not aware of. In [1], Dijkstra writes equally unmotivated “Using the rule of sines three times (in triangles AXB, BXZ, and AXY) we deduce [....]”, leaving the reader in awe for the miracle how subsequently all nasty terms cancel out. If we wish to teach our students how to design proofs we should teach them how to let their logical needs guide them through their mathematical knowledge.

Finally I would like to point out that the above design has been driven by different concerns. Using that AZ is a shared edge of two triangles was a logical necessity, our choice or parameters was a matter of experience. Instead of angles α +φ and β −φ we could have introduced φ' and α +β −φ'; it would have uglified the later development.

[0]
Coxeter, H.S.M., Introduction to Geometry, John Wiley & Sons, Inc., New York, 1969, pp.23–25.

[1]
Dijkstra, Edsger W., Selected Writings on Computing: A Personal Perspective, Springer-Verlag New York Inc., 1982, pp.182–183.

[2]Gasteren, A.J.M. van, “On the shape of mathematical arguments”, Ph.D. Thesis, 20 December 1988, Technical University Eindhoven.

Austin, 23 April 1989

prof.dr. Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712-1188

USA

transcribed by Corrado Cantelmi

revised
15-Mar-2012
