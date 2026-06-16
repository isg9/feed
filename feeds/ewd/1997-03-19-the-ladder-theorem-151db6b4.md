---
title: "The ladder theorem (EWD1258a)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD12xx/EWD1258a.html
published: "1997-03-19T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd12xx/EWD1258a.PDF
---

# The ladder theorem

Let p and q be two column vectors of the same (finite) length. Let [p ≥ q] denote that in each row, the p-element is at least the q-element , i.e.

[p ≥ q] ≡ 〈∀i :: p.i ≥ q.i 〉

Let sort.w denote the result of sorting column vector w in ascending order. Then the ladder theorem states

(0)      [p ≥ q] ⇒ [sort.p ≥ sort.q]

Proof   We observe for any x and positive n

the nth element of sort.p is ≤ x

≡    { sort.p is in ascending order }

sort.p contains at least n elements ≤ x

≡    { p is a permutation of sort.p }

p contains at least n elements ≤ x

⇒    { [p ≥ q], the antecedent of (0) }

q contains at least n elements ≤ x

≡    { q is a permutation of sort.q }

sort.q contains at least n elements ≤ x

≡    { sort.q is in ascending order }

the nth element of sort.q is ≤ x ,

and since the above calculation derives that, as a consequence of [p ≥ q], its first line implies its last line for any x , it captures by "indirect order" the elementwise demonstration of [sort.p ≥ sort.q] .

(End of Proof.)

The ladder theorem is well-known. It tells us that if, in a matrix with sorted rows, we sort all the columns, the rows remain sorted. The above proof of the ladder theorem has been recorded (i) because there are such messy proofs of it —I saw one the other month— , (ii) because I don't succeed in viewing it as "intuitively obvious", and (iii) because the above proof is so nicely disentangled: Sorting is permuting and making ascending, and please note the separate appeals to these two properties.

My thanks to the ETAC for turning my first version into a proof by indirect order.

Austin, 19 March 1997

prof. dr Edsger W.Dijkstra

Department of Computer Sciences

The University of Texas at Austin

Austin, TX 78712–1188

USA

transcribed by Diethard Michaelis

revised Mon, 10 Dec 2007
