---
title: "The Weak Byzantine Generals Problem"
url: https://lamport.azurewebsites.net/pubs/weak-byz.pdf
published: "1983-07-01T00:00:00Z"
feed: lamport
guid: https://lamport.azurewebsites.net/pubs/weak-byz.pdf
---

# The Weak Byzantine Generals Problem

The Weak Byzantine Generals Problem L. LAMPORT SRI International, Menlo Park, Califorma

Abstract The Byzantine Generals Problem requires processes to reach agreement upon a value even though some of them may fad. It is weakened by allowing them to agree upon an "incorrect" value if a failure occurs. The transaction eormmt problem for a distributed database Js a special case of the weaker problem. It is shown that, like the original Byzantine Generals Problem, the weak version can be solved only ff fewer than one-third of the processes may fad. Unlike the onginal problem, an approximate solution exists that can tolerate arbaranly many failures. Categories and Subject Descnptors: D.4.5 [Operating Systems]: Reliability, F.2.m [Analysis of Algorithms and Problem Complexity]: Mzscellaneous

General Terms: Reliabthty Addmonal Key Words and Phrases: Agreement, interactive consistency, dlstnbuted systems

1. Introduction

T h e Byzantine Generals P r o b l e m involves obtaining agreement a m o n g a collection o f processes, some o f which m a y be faulty. It can be stated precisely as follows. Byzantine Generals Problem: G i v e n a collection o f processes n u m b e r e d f r o m 0 to n - 1 which c o m m u n i c a t e by sending messages to one another, to f m d an algorithm by which Process 0 can transmit a value v to all the processes such that:

(1) I f Process 0 is nonfaulty, then a n y n o n f a u l t y Process i obtains the value v. (2) I f Processes i and j are nonfaulty, then they b o t h obtain the same value. N o t e that condition 2 follows f r o m condition 1 if Process 0 is nonfaulty. N o n f a u l t y processes are assumed to correctly follow their algorithm, but faulty processes m a y do anything. W e assume that the absence o f a message is detectable, which is equivalent to assuming that a faulty process sends every message that it is supposed t o - - a l t h o u g h it need not send the correct message. T h e difficulty o f the p r o b l e m lies in the fact that a faulty process m a y send conflicting information to two different processes. This p r o b l e m was described in [1] in terms o f a Byzantine general m e t a p h o r - hence its name. Essentially the same p r o b l e m appeared in [2], where it was called the Interactive Consistency Problem. T h e p r o b l e m was s h o w n there to be solvable if a n d only if fewer t h a n one-third o f the processes are f a u l t y - - u n l e s s unforgeable, signed Thts work was supported in part by the NaOonal Science Foundation under Grant MCS 81-04459. Author's address: Computer Science Laboratory, SRI Internauonal, 333 Ravenswood Avenue, Menlo Park, CA 94025 or distributed for direct commercml advantage, the ACM copyright notice and the title of the publication and its date appear, and notice is given that copying is by permisston of the AssoclaUon for Computing Machmery To copy otherwise, or to repubhsh, requires a fee and/or specific permtssion Journal of the Assoclatton for Computfftg Machinery, Vol 30, No 3, July 1983, pp 668-676

Weak Byzantine Generals Problem

messages are assumed. In particular, no solution works for three processes in the presence of a single fault. In this paper, we consider a weaker version of the problem, in which condition 1 is replaced by the following: (1) If all processes are nonfaulty, then every Process i obtains the value v. The transaction commit problem for a distributed database is an instance of this weaker problem, in which Process 0 represents a transaction coordinator, and the other processes represent the database sites affected by the transaction [3]. The commit coordinator's "value" is its decision of whether to commit or abort the transaction. All sites must agree on whether the transaction is committed or aborted (condition 2), but the failure of any site is allowed to abort the transaction--hence the weaker version of condition 1. Any solution to the original Byzantine Generals Problem is obviously a solution to the Weak Byzantine Generals (WBG) Problem, so the WBG Problem is solvable if fewer than one-third of the processes may be faulty. In Section 2, we prove the converse: no solution exists if one-third or more of the processes are faulty. Hence, the WBG Problem is solvable in precisely those situations in which the original Byzantine Generals Problem is. However, in Section 3 we give a "solution" that works with any number of faulty processes, but requires the processes to send an infmite number of messages before choosing their values. Of course, this "solution" is of no practical interest, since it cannot be implemented. Its interest lies in the fact that the original Byzantine Generals Problem does not possess such a "solution". (The impossibility proof of [2] did not assume a t'mite number of messages.) Hence, the WBG Problem is in some sense strictly weaker than the Byzantine Generals Problem. In Section 3, we also show that if condition 2 of the WBG Problem is replaced by a weaker condition requiring only approximate equality, then the problem is solvable with any number of faulty processes. More precisely, if the set of possible values is a bounded set of numbers, then for any ~ > 0 there is an algorithm which guarantees that the values chosen by any two nonfaulty processes differ by less than ~. It was shown in [1] that no such approximate solution exists for the original Byzantine Generals Problem. The Byzantine Generals Problem arises in practice when trying to get the nonfaulty processes to agree upon the value of some input quantity. As discussed in [1], it is central to the implementation of fault-tolerant computer systems. The WBG Problem arises when trying to get the nonfaulty processes simply to agree, regardless of what they agree upon. To eliminate the trivial possibility of having them agree upon a prearranged value, we can assume that each process chooses a private value, and that these private values are used in reaching agreement upon a single public value. The general problem of reaching agreement can then be formulated as follows:

Weak Interactive Consistency Problem: Each Process i chooses a private value w,. The processes must then communicate among themselves to allow each process to compute a public value, such that: (1) If all processes are nonfaulty and all the w, have the same value, then every process computes this value as its public value. (2) Any two nonfaulty processes compute the same public value. It is easy to show that this is equivalent to the WBG Problem. First of all, it is easy to see that if Process 0 transmits the value Woto all processes using a solution to the

L. LAMPORT

WBG problem, and all nonfaulty processes choose the value they obtain as their public value, then the above two conditions hold, so this is a solution to the Weak Interactive Consistency Problem. Conversely, given a solution to the Weak Interactive Consistency Problem, a solution to the WBG problem is obtained by having Process 0 send its value v to all the processes, and then letting each Process i use the value it received as its private value in the Weak Interactive Consistency solution.

2. Impossibility Result In this section, we prove that no solution to the WBG problem exists if one-third or more of the processes are faulty. This requires a precise statement of what constitutes a W B G solution. We begin with some notation. (A glossary of all our notation appears at the end of the paper.) Let P denote the set {0. . . . . n - 1}, and P* the set of all finite sequences of elements of P (including the null sequence). Let H denote the set of all finite sequences of the form 0, ~r with ~r ~ P*--i.e., all elements of P* whose first element is 0. We think of 0, pl . . . . . pk as the path of length k traveled by a message that starts at Process 0 and is relayed via Processes pl . . . . . pk-~ to Process pk. (This is different from the notation used in [2].) Let II, denote the subset of II consisting of all sequences ending in/--i.e., all message paths leading from Process 0 to Process i. A scenario ~ is a mapping from II into a set of values V. If we think of an element ~r of H as a message path, then ¢(~r) is the contents of the message received at its final destination. We say that Process i is nonfaulty in a scenario ¢ if for every message path ~r, i E I I and every j E P: O(,r, i, j ) ' = ¢(~r, i). In other words, i is nonfaulty in • if Process i correctly relays all messages. If all processes are nonfaulty in ~, then ~(~r) = ~(0) for all 7r E H. We define an i-scenario to be a mapping from l-I, to V. An/-scenario thus describes the contents of the messages received by Process i. For any scenario ~, we let O, be the/-scenario that is the restriction of • to IL, so ~, is the part of * "seen" by Process i. A solution to the WBG problem consists of an algorithm by which the processes send messages to one another based upon the contents of messages already received. Initially, the only information is the value v, which is known only to Process 0. Therefore, all information travels along paths in I13 To send the maximum amount of information to one another, Process 0 would send the value v to all processes, and then processes would send one another the contents of every message they receive. Thus, if ¢(0) equals v, then a scenario • describes the maximum amount of information that the processes could send to one another. A nonfaulty process can always ignore information that it receives, and a faulty process can do anything-including guess any information that was withheld from it. Hence, any algorithm for choosing values based upon information sent among the processes can be described in terms of an algorithm based upon the entire scenario ¢. We therefore make the following definition.

Definition. An m-fault W B G Algorithm B consists of a set of mappings B, from /-scenarios into V, for all i E P, such that for any scenario • in which at least n - m We could have considered paths startmg from other processes than Process 0 as well, and the unposslbdlty proof would remam essentially the same. However, for simplicity we have restncted ourselves to message paths m H

Weak Byzantine Generals Problem

processes are nonfaulty: (1) If all processes in P are nonfaulty in 4, then for all i ~ P: B,(¢9i) --- ¢~(0). (2) For any i,j E P: i f / a n d j are nonfaulty in 4, then B,(t~,) ffi Bj(d~j). We will show that no m-fault WBG algorithm exists if 3 _ n _< 3m. (The problem becomes trivial if n -- 2.) If the value of B,(~,) depended upon the entire infinite/-scenario ~,, then the algorithm B would require an infinite amount of message passing and would not be a real solution to the WBG Problem. We thus make the following definition, where rI ck) is defined to be the set of message paths in II of length at most k, and I'llk~ -- H <k) n H,.

Definition. A WBG algorithm B is said to befinite if for every scenario • there is an integer k such that for any scenario xI, and all i ~ P: if the restrictions of 4, and ~ to II <k) are equal, then B,(~) = B,('~). A finite WBG algorithm is one in which for every scenario, there is a k such that each process can choose its value after k rounds of message passing. This is a natural definition, since it insures that every process is eventually able to choose a value. However, it does not immediately rule out the possibility that the required number of rounds k can become arbitrarily large. We now show that this is not the case, and that a single value of k can be chosen for all scenarios. LV.MMA 1. For any finite WBG algorithm B there is a nonnegative integer k such that for any scenarios dp and xp and all i E P: if the restrictions of tb, and x~ to II~k) are equal, then B,(t~,) = B,('~). PROOF. Define an r-level fmite scenario to be a mapping from rl <r~to v. For any luted i, we define a tree structure on the set of all such finite scenarios by letting an r-level scenario ~ be an ancestor of an r'-level scenario 4 ' if r < r' and 4, is the restriction of ~ to 1-IIr>. Consider the subtree consisting of r-level scenarios 4, for all r, such that there exist (infinite) scenarios xI, and f~ whose restrictions to IIl ~> equal ~ , and for which B,(~,) does not equal B,(~,). If this subtree were infinite, then by Konig's lemma it would have an infinite path. Such an infinite path defines an infinite scenario d~ which contradicts the definition of finiteness. Hence, this subtree must be finite, which implies the existence of a ks such that for any scenarios and ~: if the restrictions o f ~ , and xI', to IIl h') are equal, then Bi(¢~,) ffi B,(~,). To complete the proof, we let k equal sup{k,: i E P}. [] To prove the nonexistence of an m-fault algorithm when n ___ 3m, we first prove the nonexistence of a l-fault algorithm for n = 3. Therefore, until further notice, we assume that P ffi {0, 1, 2}. We define the signed distance function 8 on P by: 8(0, 1) = 8(1, 2) = 8(2, 0) -- 1, 8(i, j ) = - 8 ( j , i). For any path Ir = 0, p~. . . . . pk we define o0r ) to equal k

Y. 8(p,_~,p,). If we think of the processes 0, 1 and 2 being arranged clockwise in a circle, then 8(i, j ) is the clockwise angular distance from i t o j (where a distance of 3 represents a full circle), and o(~r) is the signed angular distance traveled by the path ~-.

LEMMA 2.

L.

LAMPORT

For any path 0,pl . . . . . pk E H: o(O, pl, . . . ,pk) m o d 3 = p~.

PROOF. This is a simple consequence of the observation that 6(r, s) + 6(s, t) - 6(r, t) m o d 3.

[]

For any integer r, we let b"denote r m o d 3, which equals 0, 1, or 2. We now choose two particular elements o f V, which we denote T and F. The following lemma asserts the existence of a sequence o f scenarios O ~') for integral values o f r (including negative integers) which will form the basis for a proof by contradiction. Only two values, denoted T and F, appear in O <r). In this scenario, Processes r + 1 and r + 2 are nonfaulty, so they relay values correctly. The faulty Process F acts correctly except when relaying messages or for which o(~) = F, in which case it sends the value T to Process r + 1 and the value F to Process r - 1 =r+2. LEMMA 3. For any values T and F in V, and any integer r there is a scenario 0 (r) such that f o r i = 1, 2: (1) Process r + i is nonfaulty in • ~r). (2) For any ~r E H~-~: O~,)(~r)={F

if if

o(~r) >_ r + i, o(Ir)<r+i.

PROOF. By Lemma 2, condition 2 defines ~,+~¢b~r--'~for i = 1, 2. Since there are no ~(r) requirements on qJF, and Process F is allowed to be faulty, we need only show that Condition 2 is achievable when Processes r + 1 and r + 2 correctly relay messages to one another. However, this follows easily from the observation that if ~ E H;~,, then o(~', r + i + 1) = o(~r) + 1. [ ] Note that the two conditions o f Lemma 3 define the values of all messages in the scenario O t~) except for the ones that Process r sends to itself. The following result is a simple corollary o f L e m m a 3. LEMMA 4.

For any integer r: if O ~ is as in L e m m a 3, then O~f>+~ffi 0 ~ 1).

We can now prove the impossibility o f a 1-fault W B G algorithm with three processes. LEMMA 5. I f there are at least two distinct elements in V, then there does not exist a 1-fault W B G algorithm f o r n = 3. PROOF. Let B be such an algorithm~ and let T and F be distinct elements of V. Let • r and • F be the scenarios defined by oT(~r) ----T and OF(~r) ----F for all ~r ~ H. It follows from condition 1 o f the definition of a W B G algorithm that B,(OT) = T and B,(O~) = F for all i. For each integer r, let • t~) be the scenario whose existence was proved in L e m m a 3. Let k be the nonnegative integer whose existence is guaranteed by Lemma 1, with O T substituted for O. Since o(~r) is less than or equal to the length o f ~r, for any ~r in n~k_k> "'k+l, we have o(~r) <_ k < k + 1, so • tk)(or) = T. Hence, the restrictions o f the T (k) (k) (k) scenarios O~-i and 0~-,-~to H~-i are equal, so we must have B r ~ (Ok-~-~) = T. Similarly, choosing such a nonnegative integer k' for • F, since -o(~r) is less than or equal to (k) the length o f ~ , for any 7r inH'_~, we have o(~r) _ - k r ffi ( - k f - 1) + 1, so (-k'-l) • • F (-k'-l) (k') O_-~ (~r) = F. Hence, the restnctmns o f O_--~ and O_--~ to 1"[_~ are equal, so

B_~---~,( ~(--k '--1) )ffi F.

W e a k Byzantine Generals Problem

t~--i~((b~l)~ ~-~ ~. Since r + 1 and It follows from L e m m a 4 that for any r: B;~(~;;~) <r) = ~ r + 2 are nonfaulty in (I)(r), it follows from condition 2 o f the definition o f a W B G • (r) (r) (r) (r+l) algorithm that B ~ ( ~ - i ) -- B~+--~((I);-~). Hence, for each r: B ~ ( ( I ) ~ ) - - - B r - - ~ ( ( b ~ ). A simple induction argument then shows that B~((I)~+)I) = B--~((I)~-1)). However, we saw above that B~-~ ((I)(k-~) ---- T and B--~(~(-~k~,-a)) = F. Since T and F are distinct elements, this provides the required contradiction. [] We can now prove our main result. THEOREM I. I f n > 2 and V contains at least two distinct elements, then there exists an m-fault W B G algorithm if and only if n > 3m. PROOF. The " i f " part follows from the existence of algorithms to solve the original Byzantine Generals Problem, demonstrated in [1] and [2]. T o prove the "only if" part, we assume the existence of such an algorithm and derive a contradiction. Assume B is an m-fault W B G algorithm, with 3 < n < 3m. We will use it to construct a l-fault W B G algorithm for three processes, thereby contradicting Lemma 5. We first partition the (n-element) set P into three nonempty, disjoint sets Po, P1, P2 each containing at most m elements• (We can do this because 3 _< n __. 3m.) Let 0 be an element o f P0. We define the mapping ~: P ~ {0', 1', 2'} by letting X(p) = i' if and only i f p E P,. We extend X to a mapping from P * into {0', 1', 2')* in the obvious way by letting X(p0 . . . . . pk) = X(p0), . . . , ~(PD. We also let 0", 1", 2" be elements in P such that 0" = 0, 1" ~ Pa and 2" ~ / 2 . Hence, ~(i") = i'. We construct a l-fault W B G algorithm B' for the set P ' = {0', 1', 2'} as follows. For any scenario ~ ' on P', we define the scenario A[~'] on P by A[~'](~r) = ~P'(h(~r)). The W B G algorithm B' is defined by B',,~,,~ ta), ~ = B , . ( [A( I h -' ] ) . Observe that if i' is nonfaulty in (I)', then every process in P, (including i") is nonfaulty in A[(I)']. To show that B' is a l-fault W B G algorithm, we must verify the following two conditions. , , = ~'(0'). (1) IfaU processes in P' are nonfaulty in ~P,t then for all i' E pl :B,,(@,,) , (2) For any l•, j ' ~ P': if i' and f are nonfaulty in (I),, then //, ~,,~(¢I~op-~-- , Bj,((I)~,).

To prove these conditions, we use our observation that if Process i' is nonfaulty in (I)', then every process in P, is nonfaulty in A[(I)']. Hence, if all processes in P ' are nonfaulty in ~', then all processes in P are nonfaulty in A[~']. Using condition 1 for the m-fault W B G algorithm B, we see that

,',.:,'z

B' (~' ~ =

B,.,(A[~'],.)

= A[~,'](0") =

~'(0'),

which proves condition 1 for B'. Next, assume that the i' and j ' are nonfaulty in q~'. Since i" and j " are nonfaulty in A[~'], condition 2 for B yields # t B,,((I),,) = B,.(A[~'],.)

= Bf(A[~P']f) #

= B,,(~':,).

This proves condition 2 for B'. We have thus constructed a l-fault W B G algorithm for the three processes 0', 1', 2', contradicting Lemma 5. []

L. L A M P O R T

3. Approximate and Infinite Solutions We now describe an approximate solution to the W B G problem that works in the presence of any number o f faulty processes. By taking the limit o f a sequence o f such solutions, we obtain an exact solution using an infinite number of messages. In order to make the concept o f an approximate solution meaningful, we assume that the set V o f possible values is a set o f real numbers. For each integer k > 0, we define an algorithm A G tk~ that requires k rounds of message passing. Rather than describing it in terms of formal scenarios, we will simply talk about processes sending messages to one another. Nonfaulty processes are constrained to follow the algorithm, while faulty ones may do anything. We assume that a faulty process sends every message that it is supposed to, although possibly with an incorrect value. However, every value it sends is assumed to be some element o f V. It should be obvious how this description can be translated into a definition o f mappings on i-scenarios.

Algorithm AGtk): The following k rounds of message passing are executed to compute the values v~,~>for i E P and 1 _< r < k. --Round 1: • Process 0 sends the value v to every Process i. • Each Process i sets vl1) equal to the value it receives from Process 0. --Round r: (1 < r --< k) • Each Processj sends the value v~r-~ to every Process i. • Each Process i sets vlr) equal to the maximum of the n values it receives. Each Process i then sets v, equal to the average of the k values v~r~. We now prove the following result about this algorithm, which shows that it is an approximate solution to the W B G problem. THEOREM II. I f Iv[ < D for all v E V, then the algorithm AG tk) satisfies the following properties. (1) I f all processes are nonfauhy, then v, = v for every i. (2) I f Processes i a n d j are nonfaulty, then Iv, - vj[ < 2D/k. Note that no limit is placed upon the number of faulty processes. The proof of this theorem uses the following lemma. LEMMA 6.

Assume that Ivl < D for all v E V. I f Sr, tr are elements of Vsuch that: Sr <: t r + l ,

tr ~ S r + l

for all r with 1 <_ r < k, then

sr-- ~ tr < 2 D .

r~l

PROOF.

r--1

It follows from the first inequality o f the hypothesis that k

E Sr~Sk+ r.=l

E tr. r--2

F r o m this, we deduce that k

Y, st-- ~, t r < - - s ~ - t l < 2 D .

r--1

r--1

Weak Byzantine Generals Problem

The symmetric argument, interchanging s and t, yields k

Z t~- 2 sr<2D,

r~l

I',=1

and combining the two inequalities proves the lemma.

[]

PROOF OF THEOREM. To prove the first property, we simply observe that if all processes are nonfaulty, then they correctly relay values, so all the v~~) equal v. To prove the second property, we note that if Processes i and j are nonfaulty, then they correctly relay the values of v,(~) a n d -vj(~) to one another in round r + 1. It therefore follows that for each r ___ 1:

v~r) < v~~+1),

v (f) <_ v~r+~).

The second property then follows immediately from the above lemma, substituting (r) (r) v, f o r s r a n d v j fort~. [] To construct an infinite-message solution to the W B G problem, we let each Process i take as its value of v, the limit as k goes to infinity o f the values obtained by the algorithms A G (k). If the set Vis unbounded, then this limit could be infinite, in which case some arbitrary preassigned value is used. This gives us the following.

Algorithm AG(®):Compute the values v(,r) as described m Algorithm AG (k), for all i E P and r (r) r _~ I. For each i, define v, to equal sup (L : r ~ l }, where oo is interpreted to be some arbitrary fixed element of V. We now show that A G (~) is a "solution" to the W B G problem that can tolerate any number o f faults. O f course, since it requires choosing a value based upon an infinite sequence of messages, it cannot be regarded as a solution in any practical sense.

THEOREM III. I f V is a bounded set of numbers, then AG t°°) is an infinite m-fault WBG algorithm, for any m. PROOF. The proof is quite simple, and rests upon the observation that if i and j are nonfaulty, then v~(r) --< ]))r+l),

I)j(r) .~ vt(r+l)

for all r > 0, which in turn implies that sup(vl r)} = sup(v(f)). The details are left to the reader. []

Glossary of Notation General Notation: - - n : the number of processes. - - P : the set (0 . . . . . n - 1} o f processes. - - P * : the set o f finite sequences of processes. - - I I : the set of message paths from ()--sequences in P* beginning with 0. - - I I , : the set of message paths from 0 to/--sequences in II ending in L --IItk): set of message paths of length _< k in II. --II~k): set of message paths of length __ k in IL. V: the set of all possible values v.

L. LAMPORT

--scenario: a mapping ¢: H ~ V--specifying the value o f the contents of every message. --/-scenario: a mapping ~,: 17, ~ V--the part o f a scenario "seen" by Process i. - - W B G algorithm B: a collection o f mappings Bi from/-scenarios to V. Notation for n = 3: --8(i, j): the signed, clockwise distance from i to j. --o(~'): the signed angular distance traveled by the path ~r. --F: r m o d 3. --l-I('): a scenario in which a faulty Process ~'relays each message ~r correctly unless o(~r) -- r, in which case it relays the value F t o r - l and the value T t o r + 1. Notation used in proof of Theorem I: - - P ' : the set of processes {0', 1', 2'}. --A: a mapping assigning to each process in P a process in P', which assigns at most m processes to each process in P'. Also, its extension to a mapping from message paths in P to message paths in P'. - - i " : an element in P that is assigned by ~, to i', where i = 0, 1 or 2. - - A : a mapping that takes scenarios on P ' into scenarios on P, defmed by letting the value o f A[d#'] on the message path ~r equal the value of ~ ' on the path A(~r).

ACKNOWLEDGMENT. We wish to thank Michael Fischer for discovering an error in an earlier version o f this paper, and for his help in improving the presentation. REFERENCES I. LAMPORT, L., SHOSTAK, R., AND PEASE, M.

The Byzantine Generals Problem Trans Prog. Lang.

Syst. 4, 3 (July 1982),382-401 2 PEASE,M., SHOSTAK,R, AND LAMPORT, L

Reachingagreement in the presence of faults. J. ACM

27, 2 (Apr. 1980), 228-234. 3 GRAY, J.

Notes on database operating systems. In Operating Systems, an Advanced Course, Lecture

Notes m Computer Science60, R Beyer, R M. Graham, and G. Seegmuller, Eds., Springer-Verlag, New York, 1978,pp. 393-481. RECEIVED JANUARY 1981; REVISED MARCH 1982; ACCEPTED JUNE 1982

Journal of the Associationfor ComputingMachinery,Vol 30, No 3, July 1983
