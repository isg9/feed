---
title: 'research!rsc: Mel was Real'
url: https://research.swtch.com/mel
published: "2008-04-30T00:00:00Z"
feed: russcox
guid: https://research.swtch.com/mel
---

# research!rsc: Mel was Real

### [research!rsc](/)

#### Thoughts and links about programming, by [Russ Cox](https://swtch.com/~rsc/)

[RSS](/feed.atom)

# Mel was Real Russ Cox April 30, 2008 *research.swtch.com/mel* Posted on Wednesday, April 30, 2008.

Things have been pretty crazy around here, so no new posts recently, but this caught my attention.

James Grimmelmann [reports](http://laboratorium.net/archive/2008/04/28/a_few_facts_about_the_lgp30) a few interesting facts about the Librascope/Royal McBee LGP-30. Specifically, [Mel](http://www.pbm.com/~lindahl/mel.html) was real, and the hexadecimal digits were 0 1 2 3 4 5 6 7 8 9 *F G J K Q W*!

I can't resist telling a story. James is now a professor at the New York Law School, but he and I were fellow computer science students in college. He was a TA for the undergraduate introductory theory course when I took it, and his duties included, among other more enjoyable tasks, grading the first problem set. The “challenge” problem asked for a proof that in any group of six people, there is either a group of three that all know each other (a clique) or a group of three that all don't know each other (an anti-clique). I didn't see how to prove it\*, so I just wrote a simple C program to iterate over all 215 possibilities and verify that the condition holds. I wrote something about using brute force and attached the C program to the problem set. This problem set was just a warm-up in proof techniques, but the course is all about what can and can't be computed by various systems. The pinnacle result, of course, is Turing's proof that some problems are uncomputable and Cook's definition of NP-completeness. In that light, my brute force proof ran against everything the course was trying to teach.

James was the grader for that question. Next to the comment about brute force, he wrote “Wait until the NP-completeness unit, Mr. Brute Force” At the end, he added, “Very, very clever. Such tricks will not avail you much where this course is headed.” I didn't appreciate it then, but in hindsight I find it very funny.

\\* Truth be told, I still don't see how to prove it. You're supposed to use the pigeonhole principle, and the details have been explained to me in the past, but I can never reconstruct them on demand. The exhaustive search still feels easier, though perhaps less enlightening.

(Comments originally posted via Blogger.)

- [steve](http://www.blogger.com/profile/13405395955183065842)(April 30, 2008 9:13 AM) Theory graders seem to have a thing about brute force. In some UCB grad theory class I was marked down for a correct proof because I enumerated the whole set of cases, instead of being fancy and avoiding brute force. There were, IIRC, \*4\* cases.

- [Russ Cox](http://swtch.com/~rsc/)(April 30, 2008 10:09 AM) Just to be clear, James wasn't at all negative about it, and I did get full credit for the answer. I just thought it was a funny, insightful comment. Unfortunately, any student at that point in the course wouldn't appreciate it. I only noticed it looking back through my problem sets a couple years later.

As a former grader myself in many contexts, I understand the "but that's not what we wanted you to do" knee-jerk reaction, but it's the problem-writer's job to make sure that can't happen. If the gradee finds a better/different solution, more power to them.

- [Chick](http://www.blogger.com/profile/13784541025357015160)(May 1, 2008 8:38 PM) Hmm I think you could enumerate the cases for that problem: consider a graph with 6 ppl being vertexes, an edge represent two ppl knowing each other. Counting the edges:

\- 0, 1 edge => an anti-clique exists

\- 2,3 edges: if no common vertex => an anti-clique exist, otherwise a clique exist

\- 4 or more edges => there has two be a common vertex (pigeon-hole principle here) => a clique exist

- [mn](http://www.blogger.com/profile/14714143877057723159)(May 2, 2008 3:28 PM) Actually, you only proved that you have at least three connected/disconnected people, a much easier condition to ask for. I'm a little scared that a correct prove immediately sprang to my mind. (From memory, though. Little originality here on my part.) But since a wrong one's already there, I might as well post it:

Choose one person. That person either knows three other persons or dosn't know three others (pigeonhole principle.) For symmetry reasons we can assume he knows three others. If two among those know each other, too, we have found a clique, else an anti-clique.

After all those years I'm still amazed by the simplicity and elegance of the proof. (And I could resist to use graph theory nomenclature.) But that “challenge” is also a curious one: Basically it's the theory of Ramsey-numbers backwards.

- [ABcdeFU](http://www.blogger.com/profile/15949923138486620207)(May 2, 2008 6:34 PM) Funny story.

Just wondering... Do you feel like a dumbass after a brute force solution to the problem cos I do. I am taking an algorithms class in college and my grader (like yours) is a nice guy. He probably would give me full credit for a brute force solution, too. But I feel shitty after writing brute force solutions.

- [Stephan202](http://www.blogger.com/profile/03746679755202178161)(May 3, 2008 6:24 AM) mn: that is a very nice proof indeed!

- [panefsky](http://www.blogger.com/profile/00875709636921240567)(May 3, 2008 6:12 PM) Since every open problem should be solved there you go:

*mn* proves that if someone knows 3 others then either a clique or an anti-clique is created, which is correct.

So the **corollary** is that everybody knows **at most** 2 others. If the whole group is denoted by **{a,b,c,d,e,f}** and someone, e.g. **a**, knows 2 others, say **b** and **c**, then there is someone in the **{d,e,f}** that **b** and **c** do not know, e.g. **d**. If **{a,b,c}** is not a clique then **{b,c,d}** is an anti-clique.

If anyone knows exactly 1 person, then a anti-clique is created.QED!

Excuse me for my geek-ness, but I like such things. Nice blog **rsc**. I'll be around

- [mn](http://www.blogger.com/profile/14714143877057723159)(May 4, 2008 10:17 AM)This post has been removed by the author.

- [mn](http://www.blogger.com/profile/14714143877057723159)(May 4, 2008 10:39 AM) I'm afraid my attempt to make the proof accessible backfired badly. An equivalent formulation of the problem would be: Every complete graph with two edge colours has a unicoloured three-cycle. (In our case, draw a blue edge between two persons knowing each other and a red one between two not knowing each other.) Then the proof is as follows: Pick one vertex. Then there are three others connected to the first with edges of the same colour. If between two of those is an edge in the same colour these two form a three-cycle with the first one. Otherwise the three new ones form a three-cycle.

Corollary: Knowing each other is no different from not knowing each other.

- [Jones](http://www.blogger.com/profile/13193072767938982221)(July 7, 2010 9:01 AM) sorry for necroposting, but I would like to see If this 'solution' of mine would be a solution too. I thought of it in half an hour and I'm currently 26 hours without sleep, so I wouldn't wonder if not, but I want to know what goes through for a proof and what doesn't.

I'll just start with the 6 vertices. Right now, there is only anti-cliques. I will now, one after another, add edges such that no clique is formed. Once a clique is formed, I have lost anyway.

My first 5 edges are able to eliminate 4 anti-cliques each.

6 and 7 each eliminate three further anti-cliques.

In this state, there is still at least one anti-clique.

Adding any other edge will now create a clique and hence loose.

Since no further anti-cliques can be eliminated without creating a clique, it is not possible to win the game.

I hope someone reads this and gives me a comment = )

Best wishes, David

- [Robert Bloomquist](http://www.blogger.com/profile/14995249833622345826)(August 2, 2011 8:08 PM) From Turan's Theorem, a triangle-free graph (meaning no 3-member cliques) with the maximum number of edges is a complete bipartite graph in which the number of vertices on each side are as equal as possible.

In this case, we have 6 people, so we have a bipartite graph with 3 people on each side. This means that even with the maximum number of edges in the graph without a clique of three people, we still have two anti-cliques of three people.
