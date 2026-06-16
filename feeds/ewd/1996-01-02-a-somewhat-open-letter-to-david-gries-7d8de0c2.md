---
title: "A somewhat open letter to David Gries (EWD1227)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1227.html
published: "1996-01-02T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1227.PDF
---

# A somewhat open letter to David Gries

Dear David,

thank you for your letter d.d.15 December 1995 —postmarked in Elmira a week later— which arrived the other day here in Nuenen.  Yes, Rutger has shown me the discussion you and he have had;  I could just follow it but (for lack of competence) I shall not join it.  But I can react to the letter you wrote to me, and with your permission I shall do that here.

You wrote "Edsger, all I have been trying to do is to present the calculational system so that (0) students can learn it easily, and (1) logicians find it palatable, respectable, whatever.", a sentence as illuminating as it is honest.  It evokes two comments.

•     The Cornell students you are so used to are not an international standard.  Before my migration in 1984, I used to urge my European colleagues regularly not to confuse the intrinsic problems of Computing Science with the havoc created by the American educational system, and the same warning now seems appropriate for Mathematics.  I beg you to remember that what seems appropriate for you in your circumstances can be inappropriate for an equally devoted educator that operates in a different culture.

•     I never felt obliged to placate the logicians.  I (kindly!) assume that their customary precautions are needed and sufficient for reaching their goals.  If they can help and correct me by pointing out where my precautions have been insufficient and where my own discipline has led or will lead me astray, they are most welcome: I love to be corrected.  (Besides being a most instructive experience, being corrected shows that the other one cares about you.)  If however, they only get infuriated because I don't play my game according to their rules, I cannot resist the temptation to ignore their fury and to shrug my shoulders in the most polite manner.  (The other day I was shown —in Science of Computer Programming 23 (1994) 91-101—  a Book Review by Egon Börger; it was a nice example of that fury.)  I have come to the conclusion that there are such things as "disabling prejudices".

There is another sentence that evokes two comments, viz. "Well, I
happen to think that Fred and I have a logic with four inference rules
that explains your calculational system (without the everywhere
operator)."

•     From the last restriction I conclude that the inclusion of the everywhere operator presents problems for your "logic" or for your method of "explanation".  If so, I am forced to conclude that what Fred and you have is not adequate for the task at hand.

•     I do not understand what you mean by "explaining" my calculational system; an algebra is not "explained", it is "postulated"!  One postulates a domain with a few operators and relations with certain properties, and usually there are a few existence axioms as well, and the result can be viewed as a calculational system.  But Peano's Axioms don't need further "explanation", do they?  Nor do the axioms that introduce (Boolean) lattices, do they?  And I may be very naive —if I am, I would like to keep it that way— but I never saw that the definition of an algebra also needed "inference rules".  Tony had inference rules in his Axiomatic Basis of 1969; when, four years later, predicate transformers enabled me to eliminate Tony's inference rules by subsuming them in the predicate calculus and junctivity properties, I felt that that was a great simplification.  Somehow, you seem to have been induced to undo that simplification.

*            *

*

You seem to doubt Rutger's statement "...when it comes to being short, simple, illuminating, and convincing, algebraic proofs and logical deductions are simply not in the same league."  Since I started on this note, I visited the ETAC session of Tuesday 2 January 1996, where we started the study of Burghard v. Karger's draft "Temporal Logic via Galois Connections", in which he promises to show "that all seventeen axioms of the sound and complete proof system of temporal logic given in Manna and Pnueli's book [11] can be derived from just two postulates, namely (i) (⊕, ̃⊖) is a Galois connection, and (ii) (⊖,⊕) is a perfect Galois connection" and later —section 2.11— he remarks explicitly "Another advantage of algebra over logic is the ease with which one can define new algebras from old, for example by forming direct products and function spaces."  I quote von Karger to show that Rutger is not alone in his judgement.  There is a new, bright generation emerging, and as long as they calculate as effectively as they are doing now, you won't convince them.

*            *

*

You write that I have been turned off by the logicians.  That observation is correct, and I can give you a number of reasons why they turned me off.

•     Notational conventions of Whitehead and Russell —for many logicians a bible— are absolutely horrendous.  I do not blame them so much for the fact that they had not the foggiest notion of how to define their "dot notation", for BNF only came more than half a century later.  It is the dot notation itself I object to since it makes the operation of substitution a pain in the neck.  Here, Whitehead and Russell reveal themselves as complete amateurs; I don't blame them —on the contrary, for I have a weakness for amateurs— but I do blame the logicians that followed and thought this acceptable.

•     The majority of logical texts, including modern ones, that I have seen treat formulae as strings of symbols and accordingly treat formula manipulation as string manipulation.  I consider that a lack of separation of concerns.  The only thing that matters is the parsed formula —the "tree", if you wish— , all meaningful manipulations are always in terms of syntactical units, so are all meaningful definitions (like scope of dummies or the definition of the free variables of an expression), and are independent of the specific conventions that may be chosen for a linear coding of such trees.  Again, I don't blame the logicians that struggled their struggle before the notion of a formal syntax became available, but I do blame the logicians that came later and thought this failure to abstract from the specifics of the linear syntax acceptable.

•     I get completely confused as soon as logicians drag "models" and "interpretations" into the picture.  I thought that the whole purpose of the creation of a formal system in which one can calculate was to create something independent of any model or interpretation.  Doing mathematics is one thing, applying one's mathematics to a more or less real world out there is an extra-mathematical activity, and never, I think, should the two be confused.  Again, most of the logical literature I have seen and the criticism of logicians on what I (and others) are doing strikes me as a lack of separation of concerns.  As a matter of fact, the still often repeated requirement that axioms should be "self-evident" strikes me as a medieval relic; to the extent that they take philosophy seriously it is impossible for me to take the logicians seriously.  (Again this may be a cultural difference: it seems there are societies in which philosophers still have some intellectual standing.)

I never formulated it explicitly, but I could very well have the feeling that the average citizen of the logical community is not brilliant enough to make that community really interesting.

prof.dr.Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712 - 1188

USA
Nuenen, 1995.12.31 - Austin, 1996.01.11

Transcription by Andy Fugard.

Last revised on Mon, 10 Dec 2007.
