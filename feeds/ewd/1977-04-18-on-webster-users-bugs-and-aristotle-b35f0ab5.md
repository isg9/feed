---
title: "On Webster, users, bugs and Aristotle (EWD618)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD618.html
published: "1977-04-18T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD618.PDF
---

# On Webster, users, bugs and Aristotle

Copyright Notice

The following manuscript

EWD 618: On Webster, users, bugs and Aristotle

is held in copyright by Springer-Verlag New York.

The manuscript was published as pages 288–291 of

Edsger W. Dijkstra, Selected Writings on Computing: A Personal Perspective, Springer-Verlag, 1982. ISBN 0–387–90652–5.

Reproduced with permission from Springer-Verlag New York.
Any further reproduction is strictly prohibited.

EWD 618

On Webster, Users, Bugs and Aristotle.

Thinking is our most intimate activity, and a lot of it is revealed by the way in which we use (and misuse) our Language. As a matter of fact, so much is revealed by it that one cannot be a careful listener without the guilty feeling of committing the indiscreet sin of voyeurism. It is exactly this sin that I propose to commit with respect to the computing community: in this case committing the sin is too illuminating to remain virtuous.

*         *         *

Linguistical analyses tend to start with dictionaries. There are two types of dictionaries. There is the writer’s dictionary, giving hints as to how a language should be used; the Concise Oxford Dictionary is a perfect example of a writer’s dictionary. At the other end of the spectrum we have the dictionaries for the reader; they faithfully record how a language happens to be used. Webster’s New Collegiate Dictionary is a good example of a dictionary of the latter type. You can hardly use Webster’s New Collegiate Dictionary as a guide when writing —it contains terrible verbs like “to disfurnish” and “to disambiguate”— , but its authors are no fools: if an existing word is consistently used in a way that really stretches its original meaning too much, the new meaning is faithfully recorded in the next edition. A beautiful example is given in Webster’s New Collegiate Dictionary under the heading “intelligent”, where in more recent editions a third meaning has been added:

able to perform some of the functions of a computer <an ~ computer terminal>.

It is very amusing —and enlightening!— to draw the attention of members of the American computing community to this addition to Good New Webster. They are always startled by it, the Artificial Intelligentsia react to it with indignation, the others chuckle with delight, but show not seldomly signs of disbelief or amazement at Webster’s “courage”. Often they get hold of the nearest Webster to check my statement. Having verified it, they give a sigh of relief: the story is clearly too good not to be true. For many a computing scientist this additional meaning of “intelligent” in Webster acts as an authorization of his doubts about the Artificial Intelligentsia —doubts that are shared by almost all computing scientists, but that give many in the USA (where AI is officially regarded as more or less respectable) guilty feelings— . Two cheers for Webster’s ruthless accuracy!

*         *         *

The meaning of the “user” —
in extreme cases the “casual user”—
has also been extended. My Webster (1973) does not record it yet —
“one that uses” is the only definition given— , but in this case I have other linguistical indications.

It is already a change of long standing, for it was at least a decade ago when, consulting a Dutch computer manufacturer, I was amazed by and annoyed at the frequent appeal to the untranslated, just copied, “user” in the middle of a Dutch sentence defending some design decision. The noun “user”, is, of course, perfectly translatable into Dutch, but those guys did not do it! At the time I did not pay too much attention to this linguistic anomaly, I think that I classified it as the same silly mannerism as displayed by the all-English texts that are printed on Dutch cigarette packages.

But I got definitely suspicious when I learned that also the French —in spite of all their Anglophobia— embed the untranslated English word “user” in the middle of their French sentences! Since then I was alert, and I can now tell you that the word “user” is not only good Russian, but also perfect Japanese.

Now, this is very telling. One of the requirements of the final examination at the end of my training at secondary school was the translation of texts from various foreign languages “into good Dutch”. Translating a foreign text, we were taught, is a two-stage process: first the exact meaning has to be extracted from the foreign text, and then that meaning has to be rendered exactly in good Dutch. The fact that the “user” of Anglo-Saxon computing community is copied instead of translated is, therefore, for me a proof that the “user” has lost its original meaning. Subconsciously the foreign term is imported as a neologism, as a new word for a new concept.

The computer “user” isn’t a real person of flesh and blood, with passions and brains. No, he is a mythical figure, and not a very pleasant one either. A kind of mongrel with money but without taste, an ugly caricature that is very uninspiring to work for. He is, as a matter of fact, such an uninspiring idiot that his stupidity alone is a sufficient explanation for the ugliness of most computer systems. And oh! Is he uneducated! That is perhaps his most depressing characteristic. He is equally education-resistant as another equally mythical bore, “the average programmer”, whose solid stupidity is the greatest barrier to progress in programming. It is a sad thought that large sections of computing science are effectively paralyzed by the narrow-mindedness and other grotesque limitations with which a poor literature has endowed these influential mythical figures. (Computing science is not unique in inventing such paralyzing caricatures: universities all over the world are threatened by the invention of “the average student”, scientific publishing is severely hampered by the invention of “the innocent reader” and even “the poor reader”!)

*         *         *

In passing I draw attention to another English expression which often occurs in Dutch texts: “the real world”. In Dutch —and I am afraid not in Dutch alone— its usage is almost always a symptom of a violent anti-intellectualism.

*         *         *

With the publication of the Communications of the ACM, in the late fifties, began my regular exposure to American computing literature. I still vividly remember how shocked I was at first by the heavy use of anthropomorphic terminology. (Later I learned that we owe this habit to John von Neumann.) In the meantime we know that the implied metaphor is more misleading than illuminating. (For instance, in 1964 Fraser G. Duncan eloquently drew attention to all the confusion generated by calling programming languages “languages”.) Because the anthropomorphic terminology invites us to identify with programs in execution, and because “existing” is our most essential “activity”, the prevalence of these metaphors presents a severe psychological barrier to freeing our minds from the grip that operational semantics still have on them. I therefore regard the introduction of anthropomorphic terminology into computing as one of the worst services rendered to mankind by John von Neumann. (Lecturing I recently learned that among the Artificial Intelligentsia even the suggestion that anthropomorphic terminology might be unwholesome is already sheer heresy: the mere suggestion is enough to make them raving mad at you! It was very amusing and very revealing.)

It is, however, probably more than just an unhappy consequence of one of John von Neumann’s personal tastes. Recently I read Arthur Koestler’s account (in The Sleepwalkers) of how by the work of Copernicus, Kepler, Galileo, and Newton the separation between astronomy and astrology began to take place. Slowly mankind was parting from the Aristotelean animism that had ruled thought for so many centuries. In this light, the prevalence of anthropomorphic terminology in computing can also be viewed as a characteristic of its pre-scientific stage, and a consequence would be that computing scientists don’t deserve that name before they have the courage to call a “bug” an “error”.

Post Scriptum.   The day after the above was typed I had to deliver the last lecture at the ACM/ECI International Computing Symposium 1977 in Liege, Belgium. When I arrived I heard that the day before B. Meltzer —one of the Edinburgh Artificial Intelligentsia— had extensively challenged my statement:

The superstition that underlies so much of Artificial Intelligence activity is that everything difficult is so boring that it had better be done mechanically.

He had done so so emphatically that clearly some sort of rebuttal from my side was expected. Meltzer, however, had already left, and I restricted myself to writing on the blackboard the quotation from Webster that I have given above. I gave the full name of the dictionary and even mentioned the 1973 Edition.

After the closing ceremony and before starting on the journey back home I had a cup of coffee with the ICS Symposium Chairman, David Hirschberg from IBM, who asked me “Is it really true that Webster gives that third definition of “intelligent” or did you just make it up?”.

People won’t believe it! By now I am wondering what percentage of the readers of this note have already consulted their Webster... (End of post scriptum.)

Please note my new Postal Code!

Plataanstraat 5

5671 AL NUENEN

The Netherlands

prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Corrado Cantelmi

transcription edited by H. Richards into consistency with the published version

revised ...
22-Dec-2011
