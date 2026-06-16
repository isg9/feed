---
title: "Tripreport E.W.Dijkstra, Newcastle-upon-Tyne, 5–10 Sept. 1977 (EWD635)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD635.html
published: "1977-03-25T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD635.PDF
---

# Tripreport E.W.Dijkstra, Newcastle-upon-Tyne, 5–10 Sept. 1977

Copyright Notice

The following manuscript

EWD 635: Tripreport E.W.Dijkstra, Newcastle-upon-Tyne, 5–10 Sept. 1977

is held in copyright by Springer-Verlag New York.

The manuscript was published as pages 319–323 of

Edsger W. Dijkstra, Selected Writings on Computing: A Personal Perspective, Springer-Verlag, 1982. ISBN 0–387–90652–5.

Reproduced with permission from Springer-Verlag New York.
Any further reproduction is strictly prohibited.

EWD 635

Tripreport E.W.Dijkstra, Newcastle-upon-Tyne, 5 - 10 Sept. 1977.

"Gawdamighty, wot a tongue! I wonder 'er own spit don't poison 'er. I wouldn't 'ang a dog on 'er evidence."
Frank Crutchley on Mrs Ruddle [1]

At Schiphol Airport I met the colleagues van der Sluis, Blaauw and van der Poel. I heard the absence of Verrijn Stuart explained (justified?) by an admiring reference to his mountaineering exploits in the Himalaya. I did not quote Miss Twitterton's comment when told that Frank Crutchley had taken good care of the cacti [1] because I wasn't quite sure of the quotation.

The strike of the British assistant air traffic controllers delayed my arrival in Newcastle by fifteen minutes, and my return in Nuenen by twelve hours. But flights with British Caledonian do have the advantage that the planes take off and land without music.

Upon arrival my Dutch colleagues and Goos from Germany wanted to go to Henderson Hall; for me there was someone from the University with a car to take me to the Computing Laboratory. His car —a 2CV— was much too small to take all five of us, and they had to take a taxi; it was an unintended case of one-upmanship, for which I hope I won't be blamed.

The purpose of the visit was attending the yearly "Joint International Seminar on the Teaching of Computing Science", sponsored by IBM and organized by the University of Newcastle. This year's topic was "Digital Systems Design", speakers were Professor D.Aspinall (UK), Professor I.M.Barron (UK), Professor Dr.G.A.Blaauw (The Netherlands), Dr.T.C.Chen (IBM, USA), Dr.E.L.Glazer (SDC, USA), Professor F.G.Heath (UK), Professor W.M.McKeeman (USA), Professor Z.G.Vranesic (Canada), Mr.J.G.Givens (Univ. of Newcastle) and Professor C.A.R.Hoare (UK) as a stand-in for Professor Dr.Ing.R.Piloty (Germany) who was prevented from attending.

As usual the audience consisted mainly of professors of computing science; this time the speakers were mainly specialists in logic design: for many in the audience the exposure was a shock. At the level of component technology the change over the last fifteen years has been drastic: what used to be expressed in milliseconds is expressed in microseconds now, what used to be expressed in kilobucks is now expressed in dimes and quarters. This change has been so drastic that it is well-known. Much less known is that at the next levels, viz. of circuit design and logic design, the attention of the designers has been so fully usurped by the obligation to adapt to the ever changing technology, that at those levels design methodology has had no chance to mature from craft to scientific discipline. This is in sharp contrast to the developments in programming methodology, where during that period of fifteen years a fairly stable "base" could be enjoyed. Having witnessed that development in programming methodology at close quarters, I was overcome by the feeling of being exposed to the result of fifteen years of intellectual stagnation, and it was during Blaauw's lecture on the first afternoon that I asked my right-hand neighbour "Close your eyes, forget how you came here and guess in which year you are living."; without hesitation he came up with exactly the same year I had in mind: 1962.

In the corridors I later checked that that feeling of "they have failed to evolve" was much more general. It became even more justified when E.L.Glazer in his lectures and W.L.van der Poel in the discussion referred to logical design as an "art" or a "craft". In the last discussion session, on Friday afternoon, when the seminar was tied in with the general theme of the teaching of computing science, I raised the question whether the topic deserved the academic effort of trying to raise it from a craft to a scientific discipline or not. (With a few exceptions the talks had not been at an academic level, and under the assumption that the speakers had done justice to their subject, one could not avoid concluding that in its current state the topic is rather shallow; my question was essentially "Is more depth possible?".) The ensuing discussion —whom am I quoting?— "generated more heat than light". My clearest memory is that some violently objected to the idea, noteworthy I.M.Barron, who thought it appropriate to use the word "academic" in the pejorative sense it is so often used in by the vulgar. (Later that evening, while waiting for the plane to depart, I read of the effort to prevent a further debasement of the word "academic" by defining it as "a term of opprobrium applied by those who do not know their business to those who do" [2].) I also concluded that H.A.Simon had been correct [3] when he observed that today's designers —he wrote this in 1968, but it could have been written now— are perfectly willing to use the results from other scientific disciplines, but are not ready to contemplate a "science of design" nor to approach their own problems in a scientific manner.

Another overwhelming impression was the confusion between "economical" and "economic". Everybody agrees that considerations of economy play a predominant role in many aspects of computing science: it is to a large extent a science concerned with how not to waste resources. But several speakers could only deal with the (subtle) questions of economy after having translated them into the (crude) questions of economics, that is, after having equated "efficient" with "cheap". E.L.Glazer clearly demonstrated the confusion introduced by doing so. In all his lectures he mentioned the "cost equation" as his main guiding principle; at the same time he complained that its coefficients, even if known, were changing all the time. He concluded that, as a result, design was now very difficult; the only justified conclusion is that those changing values aggravate the already severe problems of doing business. The fact that in our field science and business often need each other seems in many minds to have blurred the distinction between the two, and the result is a confusing kind of unisex thinking. I.M.Barron went even further. He spoke entirely as an amateur economist, and argued that expected chip production capacity was so large, that research had to find new applications very quickly, lest the chip manufacturing firms would collapse and their large investments would be lost! (Thirty-six hours after my return I heard a proposal for automatically tuning radio sets, each equipped with a microprocessor for the decoding of the digital information to be supplied by the stations.)

T.C.Chen did in principle a good job, and his contributions were generally appreciated. With a number of very different and well-chosen examples he illustrated what novel problems may become relevant as the result of new technologies becoming available. But I found his method of presentation exasperating: he lectured as if addressing idiots. His first lecture I attended until the end, the next day I could not envisage going through that torture again and I played truant. The third day, when he gave his last lecture, I decided to be a very good boy again and to attend, but I am afraid that at the n-th insipid visual I exploded. Also Aspinall showed how easily a lecture can suffer from prepared viewgraphs. (The things being prepared in advance, one can come away with cumbersome notations; furthermore the temptation to show irrelevancies seems hard to resist.)

The most informative talks were given by J.G.Givens, W.M.McKeeman and C.A.R.Hoare. Givens described "The Work of the Digital Systems Laboratory at Newcastle-upon-Tyne" and did so very clearly. This I appreciated, independently of the fact that, if I had my way —which they are wise not to give me— I would presumably close the laboratory. When I heard the pride with which Givens told how at the end students were taught how to incorporate "more complex components", I was reminded of Donovan's article in the Comm.ACM [4] and shuddered. McKeeman's third talk "A Simple Computer" described an introductory course on computer architecture, given at Santa Cruz. His talk was informative and the course seemed indeed a broad and unbiased introduction to the problems; the associated laboratory work, however, made the course very time-consuming for the students. Hoare gave a very nice one-hour introduction to his "Communicating Sequential Processes"; he is still miles away from my ideal of defining semantics independently of any underlying computational model, but he has at least reached the stage that no one can make out whether he is talking about hardware or software. (Afterwards he wondered how many in the audience had noticed that in this respect he had not committed himself.) The reactions he evoked gave a surprising insight into some people's ignorance or small-mindedness.

During one discussion something very amazing surfaced. E.L.Glazer had described his problems in getting code for microprocessors right, and how they had been somewhat alleviated by additional hardware in which traditional debugging techniques —inspection and injection of individual register contents— could be used again. His problems had not been encountered by Fraser Duncan —who had found the good coding discipline of the late fifties again quite applicable— nor by Harry Whitfield who also had found these problems quite avoidable. So-called "cross-compilers" were mentioned as an obvious solution. Then Glazer told that he could get no one to write a cross-compiler, because computing scientists who knew how to write a compiler did not want to have anything to do with microprocessors, for fear of status and for fear that, after having been contaminated with microprocessors, "they could never return to real computing again". Nowhere else Glazer had given us reason to doubt his words, so we believed him. But then there must be something very, very wrong. Here you have jobs, challenging enough for professors in computing science in Groningen and Bristol to spend a few days, a few weeks or a few months on in order to show that the job is perfectly doable, and in Silicon Valley the professionals who should be able to do it, for some obscure (social?) reason look down on it, and the job isn't done decently. The story supported the definition of the problems of the real world as those that you are left with when you refuse to apply their effective solution. It left me very disturbed and I was reminded of a conversation with my wife, one evening a few months ago. We were talking about love of perfection, and I mentioned that R.M.Rilke always wrote flawless letters. When he made an error, he started the page afresh: as simple as that! I told her that with the suggestion that perhaps Rilke had carried a good principle too far. But my wife remarked immediately "I guess that Rilke learned very quickly how to avoid mistakes." That conversation seemed so relevant that I have told it in Newcastle to several people and I now include it in my tripreport.

I did not join the excursion (boat tour this time) on Wednesday afternoon, but went with Fraser Duncan (now at the University of Bristol) to Brian Randell's house (where I was staying), where Fraser could say hello to Brian's wife. After a cup of coffee I left them because I wanted to do some writing. Later that afternoon Brian Randell, Gerhard Seegm�ller, and Tony Hoare returned (Fraser had gone to visit the Cathedral of Durham). At the moment I was trying to comment on a paper that a friend (not one of its authors) had mailed to me, and that for eighty percent is an ugly political pamphlet disguised as a scientific paper. An editor had sent it to Tony, asking him for a rebuttal, but having other things to do Tony had declined to do so. It was an amazing coincidence, and I welcomed the opportunity to discuss that paper.

That afternoon was the only moment of peace and quiet that week. On Monday evening the participants were the guests of the Randells. On Tuesday evening the University offered a sherry party, and afterwards I had dinner in Ewan Page's new house. On Wednesday evening the participants were offered a "Mediaeval Banquet" —to be eaten with knife and fingers: appropriately called "a digital dinner"— on Thursday evening we were offered the closing dinner in the new Town Hall (furnished with an unbelievable luxury) of Newcastle-upon-Tyne. The dinner was excellent; the only shortcoming was that the dining hall was very close to the kitchen where an oven produced a loud and high-pitched tone that became very painful.

*         *         *

Recalling the sarcasms from our survival kit I can only conclude that most talks have been pretty disappointing indeed. Of one speaker I remarked that his talk had been much better than I had feared, of another speaker it has been said that his talk had enhanced the quality of the others.... "Reputations shredded while you wait." was Brian's apt comment. Brian always accuses me of a lack of tolerance, and he is, of course, right that my naive idealism should not turn me into the complete misanthrope. But what is the alternative? Am I expected to cheer when Ewan Page defends their Digital Systems Laboratory by remarking that in other departments of the University much worse things happen? Am I expected to cheer when van der Poel explains to me that there is little point in trying to educate good designers because IBM had discovered that with poor designs more money is earned? Has the seminar made me a wiser man? I hope so. And also a sadder one? I sincerely hope not.

[1]
Sayers, Dorothy L., Busman's Honeymoon, Gollancz 1937, Penguin Books 1962

[2]
Gowers, Sir Ernest, The Complete Plain Words, Pelican Books 1977

[3]
Simon, Herbert A., The Sciences of the Artificial, MIT Press, 1969

[4]
Donovan, John J., Tools and Philosophy for Software Education, Comm. ACM 19, 8 (Aug. 1976), 430 - 436.

Plataanstraat 5

5671 AL NUENEN

The Netherlands

prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Corrado Cantelmi

revised
15-Oct-2011
