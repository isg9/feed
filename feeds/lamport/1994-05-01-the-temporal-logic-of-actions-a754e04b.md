---
title: "The Temporal Logic of Actions"
url: https://lamport.azurewebsites.net/pubs/lamport-actions.pdf
published: "1994-05-01T00:00:00Z"
feed: lamport
guid: https://lamport.azurewebsites.net/pubs/lamport-actions.pdf
---

# The Temporal Logic of Actions

The Temporal Logic of Actions LESLIE LAMPORT Digital Equipment Corporation

The temporal logic of actions (TLA) is a logic for specifying and reasoning about concurrent systems. Systems and their properties are represented in the same logic, so the assertion that a system meets its specification and the assertion that one system implements another are both expressed by logical implication. TLA is very simple; its syntax and complete formal semantics are summarized in about a page. Yet, TLA is not just a logician's toy; it is extremely powerful, both in principle and in practice. This report introduces TLA and describes how it is used to specify and verify concurrent algorithms. The use of TLA to specify and reason about open systems will be described elsewhere. Categories and Subject Descriptors: D.2.4 [Software Engineering]: Program Verification— correctness proofs; F.3.1 [Logics and Meanings of Programs]: Specifying and Verifying and Reasoning about Programs—Specification techniques General terms: Theory, Verification Additional Key Words and Phrases: Concurrent programming, liveness properties, safety properties

1. LOGIC VERSUS PROGRAMMING A concurrent algorithm is usually specified with a program. Correctness of the algorithm means that the program satisfies a desired property. We propose a simpler approach in which both the algorithm and the property are specified by formulas in a single logic. Correctness of the algorithm means that the formula specifying the algorithm implies the formula specifying the property, where implies is ordinary logical implication. We are motivated not by an abstract ideal of elegance, but by the practical problem of reasoning about real algorithms. Rigorous reasoning is the only way to avoid subtle errors in concurrent algorithms, and we want to make reasoning as simple as possible by making the underlying formalism simple. How can we abandon conventional programming languages in favor of logic if the algorithm must be coded as a program to be executed? The answer is that we almost always reason about an abstract algorithm, not about a concurrent program that is actually executed. A typical example is the distributed spanning-tree algorithm used in the Autonet local area network [Schroeder et al. 1990]. The algorithm can be described in about one page of pseudo-code, but its implementation required Author's address: Systems Research Center, Digital Equipment Corporation, 130 Lytton Avenue, Palo Alto, CA 94301. not made or distributed for direct commercial advantage, the ACM copyright notice and the title of the publication and its date appear, and notice is given that copying is by permission of the specific permission. c 1993 ACM 0000-0000/93/0000-0000 $00.00 

Leslie Lamport

about 5000 lines of C code and 500 lines of assembly code.1 Reasoning about 5000 lines of C would be a herculean task, but we can reason about a one-page abstract algorithm. By starting from a correct algorithm, we can avoid the timingdependent synchronization errors that are the bane of concurrent programming. If the algorithms we reason about are not real, compilable programs, then they do not have to be written in a programming language. But, why replace a programming language by logic? Aren't programs simpler than logical formulas? The answer is no. Logic is the formalization of everyday mathematics, and everyday mathematics is simpler than programs. Consider the Pascal statement y := x + 1. Using the convention that y  denotes the new value of y, we can rewrite this statement as the mathematical formula y  = x + 1. Many readers will think that the Pascal statement and the formula are equally simple. They are wrong. The formula is much simpler than the Pascal statement. Equality is a simple concept that five-year-old children understand. Assignment (:=) is a complicated concept that university students find difficult. Equality obeys simple algebraic laws; assignment doesn't. If we assume that all variables are integervalued, we can subtract y  from both sides of the formula to obtain the equivalent formula 0 = x + 1 − y  . Trying this with the Pascal statement yields the absurdity 0 := x + 1 − y. A programming language may use mathematical terms like function, but the constructs they represent are not as simple as the corresponding mathematical concepts. Mathematical functions are simple; children in the United States learn about them at the age of twelve. Pascal functions are complicated, involving concepts like call by reference, call by value, and aliasing; it is unlikely that many university students understand them well. Advocates of so-called functional programming languages often claim that they just use ordinary mathematical functions, but try explaining to a twelve-year-old how evaluating a mathematical function can display a character on her computer screen. Since real languages like Pascal are so complicated, methods for reasoning about algorithms are usually based on toy languages. Although simpler than real programming languages, toy languages are more complicated than simple logic. Moreover, their resemblance to real languages can be dangerously misleading. In toy languages, the Hoare triple {x = 0} y := x + 1 {y = x + 1} is valid, which means that executing y := x + 1 in a state in which x equals 0 produces a state in which y equals x + 1. However, in Pascal, the program fragment x := 0; y := x + 1; write(y, x + 1) can print two different values when used in certain contexts, even if x and y are variables of type integer. The programmer who tries using toy-language rules to reason about real Pascal programs is in for a rude surprise. We do not mean to belittle programming languages. They are complicated because they have a difficult job to do. Mathematics can be based on simple concepts like functions. Programming languages cannot, because they must allow reasonably simple compilers to translate programs into reasonably efficient code for complex computers. Real languages must embrace difficult concepts like the distinction between values and locations, which leads to call-by-reference arguments and 1 Assembly code was needed because C has no primitives for sending messages across wires.

The Temporal Logic of Actions

aliasing—complications that have no counterpart in simple mathematics. Programming languages are necessary for writing real programs; but mathematics offers a simpler alternative for reasoning about concurrent algorithms. To offer a practical alternative to programming languages, a logic must be both simple and expressive. There is no point trading a programming language for a logic that is just as complicated and hard to understand. Furthermore, a logic that achieves simplicity at the expense of needed expressiveness will be impractical because the formulas describing real algorithms will be too long and complicated to understand. The logic that we propose for reasoning about concurrent algorithms is the temporal logic of actions, abbreviated as TLA. All TLA formulas can be expressed in terms of familiar mathematical operators (such as ∧) plus three new ones:  (prime), ✷, and ∃ . TLA is simple enough that its syntax and complete formal semantics can be written in about a page. Almost all that is needed to specify and reason about algorithms in TLA—its syntax, formal semantics, derived notation, and axioms and proof rules—appears in Figures 4 and 5 of Section 5.6 and Figure 9 of Section 8.2. (Missing from those figures are the rules for adding auxiliary variables, mentioned in Section 8.3.2.) Logic is a tool. Its true test comes with use. Although TLA and its proof rules can be described formally in a couple of pages, such a description would tell you nothing about how TLA is used. In this article, we develop TLA as a method of describing and reasoning about concurrent algorithms. We limit ourselves to simple examples, so we can only hint at how TLA works with real algorithms. TLA combines two logics: a logic of actions, described in Section 2, and a standard temporal logic, described in Section 3. TLA is easiest to explain in terms of a logic called RTLA, which is defined in Section 4. We describe TLA itself and illustrate its use in Sections 5–8. Section 9 mentions further applications and discusses what TLA can and cannot do, and Section 10 relates TLA to other formalisms. 2. THE LOGIC OF ACTIONS 2.1 Values, Variables, and States Algorithms manipulate data. We assume a collection Val of values, where a value is a data item. The collection Val includes numbers such as 1, 7, and −14, strings like "abc", and sets like the set Nat of natural numbers. We don't bother to define Val precisely, but simply assume that it contains all the values needed for our examples (Note2 1). We also assume the booleans true and false, which for technical reasons are not considered to be values. We think of algorithms as assigning values to variables. We assume an infinite set Var of variable names. We won't describe a precise syntax for generating variable names, but will simply use names like x and sem. A logic consists of a set of rules for manipulating formulas. To understand what the formulas and their manipulation mean, we need a semantics. A semantics is given by assigning a semantic meaning [[F ]] to each syntactic object F in the logic. The semantics of our logic is defined in terms of states. A state is an assignment of values to variables—that is, a mapping from the set Var of variable names to 2 Notes appear at the end of the article.

Leslie Lamport

the collection Val of values. Thus a state s assigns a value s(x) to a variable x. The collection of all possible states is denoted St. We write s[[x]] to denote s(x). Thus, we consider the meaning [[x]] of the variable x to be a mapping from states to values, using a postfix notation for function application. States and values are purely semantic concepts; they do not appear explicitly in formulas. 2.2 State Functions and Predicates A state function is a nonboolean expression built from variables and constant symbols—for example, x2 + y − 3 (Note 2). The meaning [[f ]] of a state function f is a mapping from the collection St of states to the collection Val of values. For example, [[x2 + y − 3]] is the mapping that assigns to a state s the value (s[[x]])2 + s[[y]] − 3, where 2 and 3 are constant symbols, and 2 and 3 are the values that they represent. We will not bother distinguishing between constant symbols and their values. We use a postfix functional notation, letting s[[f ]] denote the value that [[f ]] assigns to state s. The semantic definition is ∆

s[[f ]] = f (∀ 'v ' : s[[v]]/v)

(1)

where f (∀ 'v ' : s[[v]]/v) denotes the value obtained from f by substituting s[[v]] for ∆ v, for all variables v. (The symbol = means equals by definition.) A variable x is a state function—the state function that assigns the value s[[x]] to the state s. The definition of [[f ]] for a state function f therefore extends the definition of [[x]] for a variable x. A state predicate, called a predicate for short, is a boolean expression built from variables and constant symbols—for example, x2 = y −3 and x ∈ Nat. The meaning [[P ]] of a predicate P is a mapping from states to booleans, so s[[P ]] equals true or false for every state s. We say that a state s satisfies a predicate P iff (if and only if) s[[P ]] equals true. State functions correspond both to expressions in ordinary programming languages and to subexpressions of the assertions used in ordinary program verification. Predicates correspond both to boolean-valued expressions in programming languages and to assertions. 2.3 Actions An action is a boolean-valued expression formed from variables, primed variables, / z  are actions, where and constant symbols—for example, x + 1 = y and x − 1 ∈ x, y, and z are variables. An action represents a relation between old states and new states, where the unprimed variables refer to the old state and the primed variables refer to the new state. Thus, y = x + 1 is the relation asserting that the value of y in the old state is one greater than the value of x in the new state. An atomic operation of a concurrent program will be represented in TLA by an action. Formally, the meaning [[A]] of an action A is a relation between states—a function that assigns a boolean s[[A]]t to a pair of states s, t. We define s[[A]]t by considering s to be the "old state" and t the "new state", so s[[A]]t is obtained from A by replacing each unprimed variable v by s[[v]] and each primed variable v  by t[[v]]: s[[A]]t = A(∀ 'v ' : s[[v]]/v, t[[v]]/v  ) ∆

(2)

The Temporal Logic of Actions

Thus, s[[y = x + 1]]t equals the boolean s[[y]] = t[[x]] + 1. The pair of states s, t is called an "A step" iff s[[A]]t equals true. If action A represents an atomic operation of a program, then s, t is an A step iff executing the operation in state s can produce state t. 2.4 Predicates as Actions We have defined a predicate P to be a boolean-valued expression built from variables and constant symbols, so s[[P ]] is a boolean, for any state s. We can also view P as an action that contains no primed variables. Thus, s[[P ]]t is a boolean, which equals s[[P ]], for any states s and t. A pair of states s, t is a P step iff s satisfies P . Both state functions and predicates are expressions built from variables and constant symbols. For any state function or predicate F , we define F  to be the expression obtained by replacing each variable v in F by the primed variable v  : F  = F (∀ 'v ' : v  /v) ∆

(3)

If P is a predicate, then P  is an action, and s[[P  ]]t equals t[[P ]] for any states s and t. 2.5 Validity and Provability An action A is said to be valid, written |= A, iff every step is an A step. Formally, |= A = ∀s, t ∈ St : s[[A]]t ∆

As a special case of this definition, if P is a predicate, then |= P = ∀s ∈ St : s[[P ]] ∆

A valid action is one that is true regardless of what values one substitutes for the primed and unprimed variables. For example, the action (x + y ∈ Nat) ⇒ (2(x + y) ≥ x + y)

(4)

is valid. The validity of an action thus expresses a theorem about values. A logic contains rules for proving formulas. It is customary to write F to denote that formula F is provable by the rules of the logic. Soundness of the logic means that every provable formula is valid—in other words, that F implies |= F . The validity of an action such as (4) is proved by ordinary mathematical reasoning (Note 3). How this reasoning is formalized does not concern us here, so we will not bother to define a logic for proving the validity of actions. But, this omission does not mean such reasoning is unimportant. When verifying the validity of TLA formulas, most of the work goes into proving the validity of actions (and of predicates, a special class of actions). The practical success of any TLA verification will depend primarily on how good the verifier is at ordinary mathematical reasoning. 2.6 Rigid Variables and Quantifiers Consider a program that is described in terms of a parameter n—for example, an n-process mutual exclusion algorithm. An action representing an atomic operation of that program may contain the symbol n. This symbol does not represent a known value like 1 or "abc". But unlike the variables we have considered so far, the value of n does not change; it must be the same in the old and new state.

Leslie Lamport

The symbol n denotes some fixed but unknown value. A programmer would call it a constant because its value doesn't change during execution of the program, while a mathematician would call it a variable because it is an "unknown". We call such a symbol n a rigid variable. The variables introduced above will be called flexible variables, or simply variables. An expression like n+ 1, built from rigid variables and constant symbols, is called a constant expression. We generalize state functions and actions to allow arbitrary constant expressions instead of just constant symbols, and to allow quantification over rigid variables. For example, if x is a (flexible) variable and m and n are rigid variables, then ∃ m ∈ Nat : mx = n + x is the action asserting that there exists some natural number m such that m times the value of x in the new state equals n plus its value in the old state (Note 4): s [[∃ m ∈ Nat : mx = n + x]] t = ∃ m ∈ Nat : m(t[[x]]) = n + s[[x]] ∆

Thus, the semantics of state functions and actions is no longer given in terms only of values, but of first-order formulas containing free rigid variables and values. However, a state is still an assignment of values to flexible variables. An action A is valid iff s[[A]]t equals true for all states s and t and all possible values of its free rigid variables—for example: |= (x + y + m ∈ Nat) ⇒ ∀n ∈ Nat : n(x + y + m) ≥ (x + y + m) We do not permit quantification over flexible variables in state functions and actions. 2.7 The Enabled Predicate For any action A, we define Enabled A to be the predicate that is true for a state iff it is possible to take an A step starting in that state. Semantically, Enabled A is defined by s[[Enabled A]] = ∃ t ∈ St : s[[A]]t ∆

(5)

for any state s. The predicate Enabled A can be defined syntactically as follows. If v1 , . . . , vn are all the (flexible) variables that occur in A, then Enabled A = ∃ c1 , . . . , cn : A(c1 /v1 , . . . , cn /vn ) ∆

where A(c1 /v1 , . . . , cn /vn ) denotes the formula obtained by substituting new rigid variables ci for all occurrences of the vi in A. For example, Enabled (y = (x )2 + n) ≡ ∃ c : y = c2 + n If action A represents an atomic operation of a program, then Enabled A is true for those states in which it is possible to perform the operation. 3. SIMPLE TEMPORAL LOGIC An execution of an algorithm is often thought of as a sequence of steps, each producing a new state by changing the values of one or more variables. We will consider an execution to be the resulting sequence of states, and will take the semantic meaning of an algorithm to be the collection of all its possible executions. Reasoning about algorithms will therefore require reasoning about sequences of states. Such reasoning is the province of temporal logic.

The Temporal Logic of Actions

3.1 Temporal Formulas A temporal formula is built from elementary formulas using boolean operators and the unary operator ✷ (read always). For example, if E1 and E2 are elementary formulas, then ¬E1 ∧ ✷(¬E2 ) and ✷(E1 ⇒ ✷(E1 ∨ E2 )) are temporal formulas. We define simple temporal logic for an arbitrary class of elementary formulas. TLA will be defined later as a special case of simple temporal logic by specifying its elementary formulas. The semantics of temporal logic is based on behaviors, where a behavior is an infinite sequence of states. Think of a behavior as the sequence of states that a computing device might go through when executing an algorithm. (It might seem that a terminating execution would be represented by a finite sequence of states, but we will see in Section 5.5 why infinite sequences are enough.) We will define the meaning of a temporal formula in terms of the meanings of the elementary formulas it contains. Since an arbitrary temporal formula is built up from elementary formulas using boolean operators and the ✷ operator, and all the boolean operators can be defined in terms of ∧ and ¬, it suffices to define [[F ∧ G]], [[¬F ]], and [[✷F ]] in terms of [[F ]] and [[G]]. We interpret a temporal formula as an assertion about behaviors. Formally, the meaning [[F ]] of a formula F is a boolean-valued function on behaviors. We let σ[[F ]] denote the boolean value that formula F assigns to behavior σ, and we say that σ satisfies F iff σ[[F ]] equals true. The definitions of [[F ∧ G]] and [[¬F ]] are simple: σ[[F ∧ G]] = σ[[F ]] ∧ σ[[G]] ∆

σ[[¬F ]] = ¬σ[[F ]] ∆

In other words, a behavior satisfies F ∧ G iff it satisfies both F and G; and a behavior satisfies ¬F iff it does not satisfy F . One can derive similar formulas for the other boolean operators. For example, since F ⇒ G equals ¬(F ∧ ¬G), a straightforward calculation proves that σ[[F ⇒ G]] equals σ[[F ]] ⇒ σ[[G]]. We now define [[✷F ]] in terms of [[F ]]. Let s0 , s1 , s2 , . . . denote the behavior whose first state is s0 , second state is s1 , and so on. Then s0 , s1 , s2 , . . . [[✷F ]] = ∀n ∈ Nat : sn , sn+1 , sn+2 , . . . [[F ]] ∆

(6)

Think of the behavior s0 , . . .  as representing the evolution of the universe, where sn is the state of the universe at "time" n. The formula s0 , . . . [[F ]] asserts that F is true at time 0 of this behavior, and sn , . . . [[F ]] asserts that it is true at time n. Thus, s0 , . . . [[✷F ]] asserts that F is true at all times during the behavior s0 , . . . . In other words, ✷F asserts that F is always true. 3.2 Some Useful Temporal Formulas 3.2.1 Eventually. For any temporal formula F , let ✸F be defined by ✸F = ¬✷¬F ∆

(7)

This formula asserts that it is not the case that F is always false. In other words, ✸F asserts that F is eventually true. Since ¬∀¬ is the same as ∃, we have s0 , s1 , s2 , . . . [[✸F ]] ≡ ∃n ∈ Nat : sn , sn+1 , sn+2 , . . . [[F ]]

Leslie Lamport

for any behavior s0 , s1 , . . . . Therefore, a behavior satisfies ✸F iff F is true at some time during the behavior. 3.2.2 Infinitely Often and Eventually Always. The formula ✷✸F is true for a behavior iff ✸F is true at all times n during that behavior, and ✸F is true at time n iff F is true at some time m greater than or equal to n. Formally, s0 , s1 , . . . [[✷✸F ]] ≡ ∀n ∈ Nat : ∃m ∈ Nat : sn+m , sn+m+1 , . . . [[F ]] A formula of the form ∀n : ∃m : g(n + m) asserts that g(i) is true for infinitely many values of i. Thus, a behavior satisfies ✷✸F iff F is true at infinitely many times during the behavior. In other words, ✷✸F asserts that F is true infinitely often. The formula ✸✷F asserts that eventually F is always true. Thus, a behavior satisfies ✸✷F iff there is some time such that F is true from that time on. 3.2.3 Leads To. For any temporal formulas F and G, we define F ❀ G to equal ✷(F ⇒ ✸G). This formula asserts that any time F is true, G is true then or at some later time. The operator ❀ (read leads to) is transitive, meaning that any behavior satisfying F ❀ G and G ❀ H also satisfies F ❀ H. We suggest that readers convince themselves both that ❀ is transitive, and that it would not be had F ❀ G been defined to equal F ⇒ ✸G. 3.3 Validity and Provability A temporal formula F is said to be valid, written |= F , iff it is satisfied by all possible behaviors. More precisely, |= F = ∀σ ∈ St∞ : σ[[F ]] ∆

(8)

where St denotes the collection of all behaviors (infinite sequences of elements of St). We will represent both algorithms and properties as temporal formulas. An algorithm is represented by a temporal formula F such that σ[[F ]] equals true iff σ represents a possible execution of the algorithm. If G is a temporal formula, then F ⇒ G is valid iff σ[[F ⇒ G]] equals true for every behavior σ. Since σ[[F ⇒ G]] equals σ[[F ]] ⇒ σ[[G]], validity of F ⇒ G means that every behavior representing a possible execution of the algorithm satisfies G. In other words, |= F ⇒ G asserts that the algorithm represented by F satisfies property G. In Section 5.6, we give rules for proving temporal formulas. As usual, soundness of the rules means that every provable formula is valid—that is, F implies |= F for any temporal formula F . 4. THE RAW LOGIC 4.1 Actions as Temporal Formulas The Raw Temporal Logic of Actions, or RTLA, is obtained by letting the elementary temporal formulas be actions. To define the semantics of RTLA formulas, we must define what it means for an action to be true on a behavior. In Section 2.3, we defined the meaning [[A]] of an action A to be a boolean-valued function that assigns the value s[[A]]t to the pair of states s, t. We defined s, t to be

The Temporal Logic of Actions

an A step iff s[[A]]t equals true. We now define [[A]] to be true for a behavior iff the first pair of states in the behavior is an A step (Note 5). Formally, s0 , s1 , s2 , . . . [[A]] = s0 [[A]]s1 ∆

(9)

RTLA formulas are built up from actions using logical operators and the temporal operator ✷. Thus, if A is an action, then ✷A is an RTLA formula. Its meaning is computed as follows. s0 , s1 , s2 , . . . [[✷A]] ≡ ∀n ∈ Nat : sn , sn+1 , sn+2 , . . . [[A]] ≡ ∀n ∈ Nat : sn [[A]]sn+1

by (6) by (9)

In other words, a behavior satisfies ✷A iff every step of the behavior is an A step. In Section 2.4, we observed that if P is a predicate, then s[[P ]]t equals s[[P ]]. Therefore, s0 , s1 , . . . [[P ]] ≡ s0 [[P ]] s0 , s1 , . . . [[✷P ]] ≡ ∀n ∈ Nat : sn [[P ]] In other words, a behavior satisfies a predicate P iff the first state of the behavior satisfies P . A behavior satisfies ✷P iff all states in the behavior satisfy P . We will see that the raw logic RTLA is too powerful; it allows one to make assertions about behaviors that should not be assertable. We will define the formulas of TLA to be a subset of RTLA formulas. 4.2 Describing Programs with RTLA Formulas We have defined the syntax and semantics of RTLA formulas, but have given no idea what RTLA is good for. We illustrate how RTLA can be used, by describing the simple Program 1 of Figure 1 as an RTLA formula. This program is written in a conventional language, using Dijkstra's do construct [Dijkstra 1976], with angle brackets enclosing operations that are assumed to be atomic. An execution of this program begins with x and y both zero, and repeatedly increments either x or y (in a single operation), choosing nondeterministically between them. We now define an RTLA formula Φ that represents this program, meaning that σ[[Φ]] equals true iff the behavior σ represents a possible execution of Program 1. The formula Φ is defined in Figure 2. The predicate Init Φ asserts the initial condition, that x and y are both zero. The semantic meaning of action M1 is a relation between states asserting that the value of x in the new state is one greater than its value in the old state, and the value of y is the same in the old and new states. Thus, an M1 step represents an execution of the program's atomic operation of incrementing x. Similarly, an M2 step represents an execution of the program's other atomic operation, which increments y. The action M is defined to be the disjunction of M1 and M2 , so an M step represents an execution of one program operation. Formula Φ is true of a behavior iff Init Φ is true of the first state and every step is an M step. In other words, Φ asserts that the initial condition is true var natural x, y = 0 ; do  true → x := x + 1   true → y := y + 1 

Fig. 1. Program 1—a simple program, written in a conventional language. od

Leslie Lamport ∆

Init Φ = (x = 0) ∧ (y = 0) ∆

M1 = (x = x + 1) ∧ (y  = y)

M2 = (y  = y + 1) ∧ (x = x)

M = M1 ∨ M2 ∆

Φ = Init Φ ∧ ✷M Fig. 2.

An RTLA formula Φ describing Program 1.

initially, and that every step of the behavior represents the execution of an atomic operation of the program. Clearly, a behavior satisfies Φ iff it represents a possible execution of Program 1 (Note 6). There is nothing special about our choice of names, or in the particular way of writing Φ. There are many ways of writing equivalent logical formulas. Here are a couple of formulas that are equivalent to Φ. (x = 0) ∧ ✷(M1 ∨ M2 ) ∧ (y = 0) Init Φ ∧ ✷((x = x + 1) ∨ (y  = y + 1)) ∧ ✷((x = x) ∨ (y  = y)) The particular way of defining Φ in Figure 2 was chosen to make the correspondence with Figure 1 obvious. 5. TLA 5.1 Adding Stuttering Steps Formula Φ of Figure 2 is very simple. Unfortunately, it is too simple. In addition to steps in which x or y is incremented, a formula describing Program 1 should allow "stuttering" steps that leave both x and y unchanged. To understand why stuttering steps are needed, consider a clock that displays hours and minutes. It is specified by a formula Π with two variables: h representing the hours display and m representing the minutes display. Now consider a clock that displays hours, minutes, and seconds; it is represented by a formula Ψ with three variables: h, m, and another variable s representing the seconds display. A clock that displays hours, minutes, and seconds should satisfy the specification Π of a clock that displays hours and minutes. (If we don't want the seconds display, we can always cover it up.) Hence, any behavior satisfying Ψ should satisfy Π. Behaviors satisfying Ψ contain sequences of 59 consecutive steps in which h and m do not change, so Π must allow such steps. From the point of view of a clock displaying only hours and minutes, steps in which h and m do not change are stuttering steps. In general, a specification Π should be invariant under stuttering, meaning that adding or removing stuttering steps from a behavior does not affect whether the behavior satisfies Π. It is easy to modify formula Φ of Figure 2 so it asserts that every step is either an M step or a step that leaves x and y unchanged; the new definition is Φ = Init Φ ∧ ✷(M ∨ ((x = x) ∧ (y  = y))) ∆

(10)

We now introduce notation that makes it easy to ensure that a formula allows stuttering steps. Two ordered pairs are equal iff their components are equal, so the conjunction (x = x) ∧ (y  = y) is equivalent to the single equality x , y   = x, y. The definition of priming a state function (formula (3)) allows us to write x , y  

The Temporal Logic of Actions

as x, y (Note 7). For any action A and state function f , we let [A]f = A ∨ (f  = f ) ∆

(11)

(The action [A]f is read square A sub f .) Then [M]x, y ≡ M ∨ (x, y = x, y) ≡ M ∨ ((x = x) ∧ (y  = y)) and we can rewrite (10) as Φ = Init Φ ∧ ✷[M]x, y ∆

(12)

We define TLA to be the temporal logic whose elementary formulas are predicates and formulas of the form ✷[A]f , where A is an action and f a state function. Since these formulas are RTLA formulas, we have already defined their semantic meanings. 5.2 Adding Liveness The formula Φ defined by (12) allows behaviors that start with Init Φ true (x and y both zero) and never change x or y. Such behaviors do not represent acceptable executions of Program 1, so we must strengthen Φ to disallow them. Formula Φ of (12) asserts that a behavior may not start in any state other than one satisfying Init Φ and may never take any step other than a [M]x, y step. An assertion that something may never happen is called a safety property. An assertion that something eventually does happen is called a liveness property. (Safety and liveness have been defined formally by Alpern and Schneider [1985].) The formula Init Φ ∧ ✷[M]x, y is a safety property. To complete the description of Program 1, we need an additional liveness property asserting that the program keeps going. By Dijkstra's semantics for his do construct, the liveness property for Program 1 should assert only that the program never terminates. In other words, Dijkstra would require that a behavior must contain infinitely many steps that increment x or y. This property is expressed by the RTLA formula ✷✸M, which asserts that there are infinitely many M steps. Dijkstra would have us define Φ by Φ = Init Φ ∧ ✷[M]x, y ∧ ✷✸M ∆

(13)

However, the example becomes more interesting if we add the fairness requirement that both x and y must be incremented infinitely often. (Dijkstra's definition would allow an execution in which one variable is incremented infinitely often while the other is incremented only a finite number of times.) Since we are not fettered by the dictates of conventional programming languages, we will adopt this stronger liveness requirement. The formula Φ representing the program with this fairness requirement is Φ = Init Φ ∧ ✷[M]x, y ∧ ✷✸M1 ∧ ✷✸M2 ∆

(14)

Formulas (13) and (14) are RTLA formulas, but not TLA formulas. An action A can appear in a TLA formula only in the form ✷[A]f (unless A is a predicate), so ✷✸M1 and ✷✸M2 are not TLA formulas. We now rewrite them as TLA formulas. Let A be any action and f any state function. Then ¬A is also an action, so ¬✷[¬A]f is a TLA formula. Applying our definitions gives

Leslie Lamport

¬✷[¬A]f ≡ ✸¬[¬A]f ≡ ✸¬(¬A ∨ (f  = f )) ≡ ✸(A ∧ (f  = f ))

by (7), which implies ¬✷ . . . ≡ ✸¬ . . . by (11) by simple logic

We define the action Af (read angle A sub f ) by Af = A ∧ (f  = f ) ∆

(15)

The calculation above shows that ✸Af equals ¬✷[¬A]f , so it is a TLA formula (Note 8). Since incrementing a variable changes its value, both M1 and M2 imply x, y = x, y (Note 9). Hence, M1 is equivalent to M1 x, y , and M2 is equivalent to M2 x, y . We can therefore rewrite Φ as a TLA formula as follows. Φ = Init Φ ∧ ✷[M]x, y ∧ ✷✸M1 x, y ∧ ✷✸M2 x, y ∆

(16)

5.3 Fairness Using arbitrary liveness properties like ✷✸M1 x, y to express fairness requirements is dangerous because it can add unexpected safety properties. For example, conjoining the liveness property ✷✸(x = 0), which asserts that x infinitely often equals 0, to Init Φ ∧ ✷[M]x, y implies the additional safety property that the value of x never changes. Accidentally adding safety properties in this way is a common source of errors in temporal logic specifications. We avoid such errors by expressing liveness in terms of fairness. Fairness means that if a certain operation is possible, then the program must eventually execute it. The fairness requirements for concurrent algorithms can be expressed in terms of weak fairness and strong fairness conditions. We first define weak and strong fairness informally, then translate the informal definitions into TLA formulas. Weak fairness asserts that an operation must be executed if it remains possible to do so for a long enough time. "Long enough" means until the operation is executed, so weak fairness asserts that eventually the operation must either be executed or become impossible to execute—perhaps only briefly. A naive temporal logic translation is weak fairness:

(✸ executed) ∨ (✸ impossible)

Strong fairness asserts that the operation must be executed if it is often enough possible to do so. Interpreting "often enough" to mean infinitely often, strong fairness asserts that either the operation is eventually executed, or its execution is not infinitely often possible. Not infinitely often possible is the same as eventually always impossible (because (7) implies ¬✷✸ . . . ≡ ✸✷¬ . . .), so we get strong fairness:

(✸ executed) ∨ (✸✷ impossible)

These two temporal formulas assert fairness at "time zero", but we want fairness to hold at all times. The correct formulas are therefore weak fairness: strong fairness:

✷((✸ executed) ∨ (✸ impossible)) ✷((✸ executed) ∨ (✸✷ impossible))

The Temporal Logic of Actions

Temporal logic reasoning, using either the axioms in Section 5.6 or the semantic definitions of ✷ and ✸, shows that these conditions are equivalent to weak fairness: strong fairness:

(✷✸ executed) ∨ (✷✸ impossible) (✷✸ executed) ∨ (✸✷ impossible)

To formalize these definitions, we must define "executed" and "impossible". In Program 1, execution of the operation x := x + 1 corresponds to an M1 step in the behavior. To obtain a TLA formula, the "✸ executed" for this operation must be expressed as ✸M1 x, y . In general, "✸ executed" will be expressed as ✸Af , where A is the action that corresponds to an execution of the operation, and f is an n-tuple of relevant variables. Recall that an Af step is an A step that changes the value of f . Steps that do not change the values of any relevant variables might as well not have occurred, so there is no need to consider them as representing operation executions. We now define "impossible". Executing an operation means taking an Af step for some action A and state function f . It is possible to take such a step iff Enabled Af is true. Thus, Enabled Af asserts that it is possible to execute the operation represented by the action Af , so "impossible" is ¬Enabled Af . Weak fairness and strong fairness are therefore expressed by the two formulas WFf (A) = (✷✸Af ) ∨ (✷✸¬Enabled Af )

(17)

SFf (A) = (✷✸Af ) ∨ (✸✷¬Enabled Af )

(18)

∆ ∆

Since ✸✷F implies ✷✸F for any F , the strong fairness condition SFf (A) implies the weak fairness condition WFf (A). The pair of formulas Init ∧ ✷[N ]v , F is said to be machine closed if conjoining F to Init ∧ ✷[N ]v introduces no additional safety properties. (In this case, we often say that Init ∧ ✷[N ]v ∧ F is machine closed.) We avoid accidentally adding safety properties by writing machine-closed specifications. It can be shown that if F is the conjunction of fairness conditions of the form WFf (A) and/or SFf (A), where each Af implies N , then Init ∧ ✷[N ]v ∧ F is machine closed [Abadi and Lamport 1992]. 5.4 Rewriting the Fairness Requirement We now rewrite the property ✷✸M1 x, y ∧ ✷✸M2 x, y in terms of fairness conditions. An M1 x, y step is one that increments x by one, leaves y unchanged, and changes the value of x, y. It is always possible to take a step that adds one to x and leaves y unchanged, and adding one to a number changes it. Hence, Enabled M1 x, y equals true throughout any execution of Program 1 (Note 10). Since ✷¬true equals false, both WFx, y (M1 ) and SFx, y (M1 ) equal ✷✸M1 x, y . Similarly, WFx, y (M2 ) and SFx, y (M2 ) both equal ✷✸M2 x, y . We can therefore rewrite the definition (16) of Φ as shown in Figure 3. Suppose we wanted the weaker liveness condition that execution never terminates, so the program is described by the RTLA formula (13). The same argument as for M1 and M2 shows that ✷✸Mx, y equals WFx, y (M). Therefore, Program 1 with this weaker liveness condition is described by the TLA formula Init Φ ∧ ✷[M]x, y ∧ WFx, y (M). The actions M1 x, y , M2 x, y , and Mx, y all imply M. Hence neither of the liveness conditions WFx, y (M1 ) ∧ WFx, y (M2 ) and WFx, y (M1 ) add safety

Leslie Lamport

properties to Init Φ ∧ ✷[M]x, y . 5.5 Examining Formula Φ TLA formulas that represent programs can always be written in the same form as Φ of Figure 3—that is, as a conjunction Init ∧ ✷[N ]f ∧ F , where Init is a predicate specifying the initial values of variables. N is the program's next-state relation, the action whose steps represent executions of individual atomic operations. f is the n-tuple of all flexible variables. F is the conjunction of formulas of the form WFf (A) and/or SFf (A), where A is an action representing some subset of the program's atomic operations. We now examine the behaviors that satisfy formula Φ. Let ((x = 7, y = −10, z = "abc", . . . )) ∆

denote a state s such that s[[x]] = 7, s[[y]] = −10, and s[[z]] = "abc". (The ". . ." indicates that the value of s[[v]] is left unspecified for all other variables v.) A behavior that satisfies Φ begins in a state satisfying Init Φ , and consists of a sequence of [M]x, y steps—ones that are either M steps or else leave x and y unchanged. One such behavior is ∆

(( x = 0, y = 0, z = "abc" . . . )) ∆ ∆ ∆ (( x = 1, y = 0, z = 14 . . . )) ∆ ∆ ∆ (( x = 2, y = 0, z = Nat . . . )) ∆ ∆ ∆ (( x = 2, y = 0, z = −20 . . . )) √ ∆ ∆ ∆ (( x = 2, y = 1, z = 2 . . . )) .. . Observe that Φ constrains only the values of x and y; it allows all other variables such as z to assume completely arbitrary values. Suppose Ψ is a formula describing a program that has no variables in common with Φ. Then a behavior satisfies Φ∧Ψ iff it represents an execution of both programs—that is, iff it describes a universe in which both Φ and Ψ are executed concurrently. Thus, Φ ∧ Ψ is the TLA formula representing the parallel composition of the two programs. In general, parallel composition is represented in TLA by conjunction. For example, let Φ1 = (x = 0) ∧ ✷[M1 ]x ∧ WFx (M1 ) ∆

Φ2 = (y = 0) ∧ ✷[M2 ]y ∧ WFy (M2 ) ∆

Init Φ = (x = 0) ∧ (y = 0) ∆

M1 = (x = x + 1) ∧ (y  = y)

M2 = (y  = y + 1) ∧ (x = x)

M = M1 ∨ M2 ∆

Φ = Init Φ ∧ ✷[M]x, y ∧ WFx, y (M1 ) ∧ WFx, y (M2 ) Fig. 3.

The TLA formula Φ describing Program 1.

The Temporal Logic of Actions

A straightforward calculation shows that [M1 ]x ∧ [M2 ]y is equivalent to [M]x, y , and temporal logic reasoning (using the axioms of Section 5.6) then shows that Φ is equivalent to Φ1 ∧Φ2 . Formulas Φ1 and Φ2 are the specifications of the two processes forming Program 1. This example illustrates a general method for decomposing the specification of a multiprocess program as the conjunction of the specifications of its processes [Abadi and Lamport 1993]. The observation that a single behavior can represent an execution of two or more noninteracting programs explains why we represent terminating as well as nonterminating executions by infinite behaviors. Termination of a program means that it has stopped; it does not mean that the entire universe has come to a halt. A terminating execution is represented by a behavior in which eventually all of the program's variables stop changing. It is unusual in computer science for the semantics of a formula describing a program with variables x and y to involve other variables such as z that appear nowhere in the program. One of the keys to TLA's simplicity is that its semantics rests on a single, infinite set of variables—not on a different set of variables for each program. Thus, in TLA as in elementary logic, we can take the conjunction F ∧ G of any formulas F and G—not just of formulas with properly matching variable declarations. 5.6 Simple TLA We now complete the definition of Simple TLA by adding one more bit of notation. (The full logic, containing quantification, is introduced in Section 8.) It is convenient to define the action Unchanged f , for f a state function, by Unchanged f = f  = f ∆

Thus, an Unchanged f step is one in which the value of f does not change. The syntax and semantics of Simple TLA, along with the additional notation we use to write TLA formulas, are all summarized in Figure 4. This figure explains all you need to know to understand TLA formulas such as formula Φ of Figure 3. A logic contains not only syntax and semantics, but also rules for proving theorems. Figure 5 lists all the axioms and proof rules we need for proving simple TLA formulas.3 The rules of simple temporal logic are used to derive temporal tautologies— formulas that are true regardless of the meanings of their elementary formulas. Rule STL1 encompasses the rules of ordinary logic, such as modus ponens (Note 11). The Lattice Rule assumes a (possibly infinite) set S and a mapping that assigns a TLA formula Hc to each element c of S. A partial order  on S is well-founded iff there exists no infinite descending chain c1  c2  . . . with all the ci in S. This rule permits the formalization of counting-down arguments, such as the ones traditionally used to prove termination of sequential programs. Rules STL1–STL6, the Lattice Rule, and the basic rules TLA1 and TLA2 form a relatively complete proof system for reasoning about algorithms in TLA. Roughly speaking, this means that every valid TLA formula that we must prove to verify properties of algorithms would be provable from these rules if we could prove all 3 A proof rule F, G asserts that  F and  G imply  H. We use the term "rule" for both axioms H

and proof rules, since an axiom may be viewed as a proof rule with no hypotheses.

Leslie Lamport

Syntax ∆

formula

= predicate  | ✷[action]state function  | ¬formula  | formula  ∧ formula  | ✷ formula

action

= boolean-valued expression containing constant symbols, variables, and primed variables

predicate 

= action with no primed variables | Enabled action 

∆ ∆

state function = nonboolean expression containing constant symbols and variables Semantics s[[f ]]

= f (∀ 'v ' : s[[v]]/v)

σ[[F ∧ G]] = σ[[F ]] ∧ σ[[G]]

s[[A]]t = A(∀ 'v ' : s[[v]]/v, t[[v]]/v ) |= A

σ[[¬F ]]

= ¬σ[[F ]]

|= F

= ∀σ ∈ St∞ : σ[[F ]]

= ∀s, t ∈ St : s[[A]]t s[[Enabled A]]

∆ ∆

= ∃t ∈ St : s[[A]]t ∆

s0 , s1 , . . . [[✷F ]] = ∀n ∈ Nat : sn , sn+1 , . . . [[F ]] s0 , s1 , . . . [[A]]

= s0 [[A]]s1

Additional notation p

= p(∀ 'v ' : v /v) ∆ ∆

[A]f =

A ∨ (f  = f )

✸F

= ¬✷¬F

F ❀G

= ✷(F ⇒ ✸G)

Af = A ∧ (f  = f )

WFf (A) = ✷✸Af ∨ ✷✸¬Enabled Af

Unchanged f = f  = f

SFf (A)

∆ ∆

= ✷✸Af ∨ ✸✷¬Enabled Af

where f is a state function A is an action F and G are formulas p is a state function or predicate  Fig. 4.

s, s0 , s1 , . . . are states σ is a behavior (∀ 'v ' : . . . /v, . . . /v ) denotes substitution for all variables v Simple TLA.

valid action formulas. (This is analogous to the traditional relative completeness results for program verification, which assume provability of all valid predicates [Apt 1981].) A more precise statement of this result is given in Section 8.3.2 below (Note 12). A complete proof system is not necessarily a convenient one. For practical reasoning, STL1–STL6 should be augmented with some useful temporal tautologies like (✷F ) ∧ (✸G) ⇒ ✸(F ∧ G) With practice, such simple tautologies become as obvious as the ordinary laws of propositional logic. They are usually taken for granted in hand proofs, and practical decision procedures exist for checking them mechanically [Burch et al. 1992]. The temporal operators ✷, ✸, and ❀ are standard [Manna and Pnueli 1991], so we will not discuss the rules of simple temporal logic. Assuming simple temporal reasoning, we have found that TLA2 and the "additional rules" INV1–SF2 of Figure 5 provide a convenient system for all the proofs that arise in reasoning about programs with TLA. The overbars in rules WF2 and SF2 are explained in Section 8.3.3; for now, the reader can pretend that they are not there, obtaining special cases of the rules.

The Temporal Logic of Actions

The Rules of Simple Temporal Logic STL1. F provable by STL4. F ⇒ G propositional logic ✷F ⇒ ✷G ✷F STL2.  ✷F ⇒ F STL5.  ✷(F ∧ G) ≡ (✷F ) ∧ (✷G) STL3.  ✷✷F ≡ ✷F STL6.  (✸✷F ) ∧ (✸✷G) ≡ ✸✷(F ∧ G) LATTICE.  a well-founded partial order on a set S F ∧ (c ∈ S) ⇒ (Hc ❀ (G ∨ ∃d ∈ S : (c  d) ∧ Hd )) F ⇒ ((∃c ∈ S : Hc ) ❀ G) The Basic Rules of TLA P ∧ (f  = f ) ⇒ P 

TLA1.

TLA2.

✷P ≡ P ∧ ✷[P ⇒ P  ]f Additional Rules INV1. I ∧ [N ]f ⇒ I 

P ∧ [A]f ⇒ Q ∧ [B]g ✷P ∧ ✷[A]f ⇒ ✷Q ∧ ✷[B]g

INV2.  ✷I ⇒ (✷[N ]f ≡ ✷[N ∧ I ∧ I  ]f )

I ∧ ✷[N ]f ⇒ ✷I WF1.

P ∧ [N ]f ⇒ (P  ∨ Q ) P ∧ N ∧ Af ⇒ Q P ⇒ Enabled Af

✷[N ]f ∧ WFf (A) ⇒ (P ❀ Q)

WF2. N ∧ Bf ⇒ Mg P ∧ P  ∧ N ∧ Af ∧ Enabled Mg ⇒ B P ∧ Enabled Mg ⇒ Enabled Af ✷[N ∧ ¬B]f ∧ WFf (A) ∧ ✷F ∧ ✸✷Enabled Mg ⇒ ✸✷P ✷[N ]f ∧ WFf (A) ∧ ✷F ⇒ WFg (M)

SF1. P ∧ [N ]f ⇒ (P  ∨ Q ) P ∧ N ∧ Af ⇒ Q ✷P ∧ ✷[N ]f ∧ ✷F ⇒ ✸Enabled Af ✷[N ]f ∧ SFf (A) ∧ ✷F ⇒ (P ❀ Q)

SF2. N ∧ Bf ⇒ Mg P ∧ P  ∧ N ∧ Af ⇒ B P ∧ Enabled Mg ⇒ Enabled Af ✷[N ∧ ¬B]f ∧ SFf (A) ∧ ✷F ∧ ✷✸Enabled Mg ⇒ ✸✷P ✷[N ]f ∧ SFf (A) ∧ ✷F ⇒ SFg (M)

where

F , G, Hc are TLA formulas A, B, N , M are actions Fig. 5.

P , Q, I are predicates f , g are state functions

The axioms and proof rules of Simple TLA.

Leslie Lamport

The validity of these rules can be proved rigorously with the raw logic RTLA. Rules STL1–STL6 are valid when F and G are arbitrary RTLA formulas, not just TLA formulas. The validity of TLA1–SF2 can be proved using STL1–STL6 and the RTLA rule ✷P ≡ P ∧ ✷(P ⇒ P  ). The validity of this rule follows easily from the semantic definitions of ✷ and  (prime). We leave the rigorous proofs as an exercise for the reader; instead, we give informal justifications. Sections 6 and 7 illustrate how the rules are used. Rule TLA1 provides an induction principle for proving the formula ✷P . It asserts the obvious fact that a predicate P is always true iff P holds initially and every step starting with P true leaves P true. Rule TLA2 follows immediately from the validity of STL4 and STL5 for RTLA formulas. (We have to use RTLA because ✷(P ∧ [A]f ) is not a TLA formula.) Rule INV1 is used to prove that a program satisfies an invariance property ✷I. The hypothesis asserts that a [N ]f step cannot falsify I. The conclusion asserts the obvious consequence that if I is true initially and every step is a [N ]f step, then I is always true. Rule WF1 is used to deduce a leads-to property P ❀ Q from a weak fairness condition WFf (A). It can be applied when an A step that starts with P true makes Q true. To prove the validity of the conclusion, we assume that every step is a [N ]f step and that WFf (A) holds, and we prove P ❀ Q. It suffices to derive a contradiction by assuming that P is true at some time n and Q is false then and at all later times. Since every step is a [N ]f step and Q is false from time n on, the first hypothesis implies that P is true from time n on. The third hypothesis then implies that Enabled Af is true from time n on. Hence, WFf (A) implies that infinitely many A steps occur. Any such step occurring after time n starts with P true, and the second hypothesis implies that the step makes Q true. This contradicts the assumption that Q remains false after time n, proving the validity of the rule. Rule WF2 is used to deduce one weak fairness condition from another. We deduce WFg (M) from WFf (A) by finding an action B such that every B step is an M step and, if M remains forever enabled, then eventually every A step is a B step. (We ignore the overbars.) To prove the validity of WF2, we first observe that since WFg (M) equals ✷✸¬Enabled Mg ∨ ✷✸Mg and ✸ equals ¬✷¬, we can rewrite the conclusion as follows. ✷[N ]f ∧ WFf (A) ∧ ✷F ∧ ✸✷Enabled Mg ⇒ ✷✸Mg It therefore suffices to obtain a contradiction by assuming that ✷[N ]f ∧ WFf (A) ∧ ✷F ∧✸✷Enabled Mg holds and only finitely many Mg steps occur. Since [N ]f ∧ Bf equals N ∧ Bf , it follows from the first hypothesis that only finitely many Bf steps can occur. Hence, there must eventually be a time after which no more Bf steps occur, so every further step is a [N ∧¬B]f step. By the fourth hypothesis, there must then be a time at which P becomes true forever. The third hypothesis then implies that Enabled Af eventually becomes true forever, so WFf (A) implies that there are infinitely many Af steps. The second hypothesis then implies that there are infinitely many Bf steps, which is the required contradiction. Rules SF1 and SF2 are the analogs of WF1 and WF2 for strong fairness. We omit their justifications, which are similar to those of WF1 and WF2.

The Temporal Logic of Actions

6. PROVING SIMPLE PROPERTIES OF PROGRAMS Having expressed Program 1 of Figure 1 as the TLA formula Φ of Figure 3, we now consider how to express and prove properties of such a program. A property is expressed by a TLA formula F . The assertion "program Φ has property F " is expressed in TLA by the validity of the formula Φ ⇒ F , which asserts that every behavior satisfying Φ satisfies F . We consider two popular classes of properties, invariance and eventuality. 6.1 Invariance Properties 6.1.1 Definition. An invariance property is expressed by a TLA formula ✷P , where P is a predicate. Examples of invariance properties include partial correctness. P asserts that if the program has terminated, then the answer is correct. deadlock freedom. P asserts that the program is not deadlocked. mutual exclusion. P asserts that at most one process is in its critical section. Invariance properties are proved with rule INV1 of Figure 5. 6.1.2 An Example: Type Correctness. One part of the program in Figure 1 does not correspond to anything in Figure 3—the type declaration of the variables x and y. Such a declaration is not needed because type-correctness is an invariance property of the program, asserting that x and y are always natural numbers. We illustrate invariance proofs by proving type correctness of Program 1. Type correctness is expressed formally as Φ ⇒ ✷T , where T = (x ∈ Nat) ∧ (y ∈ Nat) ∆

(19)

Rule INV1 tells us that we must prove Init Φ ⇒ T

(20)

T ∧ [M]x, y ⇒ T

(21)

from which we deduce Φ ⇒ ✷T as follows Φ ⇒ Init Φ ∧ ✷[M]x, y ⇒ T ∧ ✷[M]x, y ⇒ ✷T

by definition of Φ (Figure 3) by (20) by (21) and INV1

The proof of (20) is trivial. The proof of (21) is quite simple, but we will sketch it to show how the structure of the formulas leads to a natural decomposition of the proof. First, we expand the definition of [M]x, y . [M]x, y ≡ M ∨ (x, y = x, y) ≡ M1 ∨ M2 ∨ (x, y = x, y)

by (11) by definition of M

Since [M]x, y is the disjunction of three actions, the proof of (21) decomposes into the proof of three simpler formulas: T ∧ M1 ⇒ T  

T ∧ M2 ⇒ T T ∧ (x, y = x, y) ⇒ T 

(22) (23) (24)

Leslie Lamport

We consider the proof of (22); the others are equally simple. First, we expand the definition of T  . T  ≡ ((x ∈ Nat) ∧ (y ∈ Nat)) ≡ (x ∈ Nat) ∧ (y  ∈ Nat)

by (19) by (3)

The structure of T  as the conjunction of two actions leads to the decomposition of the proof of (22) into the proof of the two simpler formulas T ∧ M1 ⇒ x ∈ Nat T ∧ M1 ⇒ y  ∈ Nat

(25) (26)

The proof of (25) is T ∧ M1 ⇒ (x ∈ Nat) ∧ (x = x + 1) ⇒ x ∈ Nat

by definition of T and M1 by properties of natural numbers

and the proof of (26) is equally trivial. The purpose of this exercise in simple mathematics is to illustrate how "mechanical" the proof of this invariance property is. Rule INV1 tells us we must prove (20) and (21), and the structure of [M]x, y and T leads to the decomposition of those proofs into the verification of simple facts about natural numbers, such as (x ∈ Nat) ⇒ (x + 1 ∈ Nat). 6.1.3 General Invariance Proofs. The proof of Φ ⇒ ✷T was simple because T is an invariant of the action [M]x, y , meaning that T ∧ [M]x, y implies T  . Therefore, Φ ⇒ ✷T could be proved by simply substituting T for I in rule INV1. For invariance properties ✷P other than simple type correctness, P is usually not an invariant. In general, one proves that ✷P is an invariance property of the program represented by the TLA formula Init ∧ ✷[N ]f ∧ F by finding a predicate I (the invariant) satisfying the three conditions Init ⇒ I I⇒P

(27) (28)

I ∧ [N ]f ⇒ I 

(29)

Rule INV1 and some simple temporal reasoning shows that (27)–(29) imply Init ∧ ✷[N ]f ⇒ ✷P . Creative thought is needed to find the invariant I. Once I is found, verifying (27)– (29) is a matter of mechanically applying the definitions and using the structure of the formulas to decompose the proofs, just as in the proof of Φ ⇒ ✷T above. The formulas Init , I, and N will usually be much more complicated than in the example, but the principle is the same. Formulas (27)–(29) are assertions about predicates and actions; they are not temporal formulas. All the work in proving an invariance property is done in the realm of predicates and actions—expressions involving variables and primed variables that can be manipulated by ordinary mathematics. Temporal reasoning is used only to deduce Init ∧ ✷[N ]f ⇒ ✷P from (27)–(29). TLA is practical because it minimizes temporal reasoning, relying on ordinary, nontemporal reasoning whenever possible. 6.1.4 More About Invariance Proofs. Over the years, many methods have been proposed for proving invariance properties of programs, including Floyd's method

The Temporal Logic of Actions

[Floyd 1967], Hoare logic [Hoare 1969], and the Owicki-Gries method [Owicki 1975]. All of these methods are essentially the same—when applied to the same program, they involve the same proof steps, though perhaps in different orders and with different notation. These methods can be described formally in TLA as applications of rule INV1. The advantage of TLA is that the proof method arises directly from the logic, without the need for proof rules based on a particular programming language. We illustrate the advantage of working in a simple logic by considering the use of one invariance property to prove another. We have just proved Φ ⇒ ✷T , the assertion that the program satisfies the invariance property ✷T . How can we use this fact when proving that the program satisfies a second invariance property ✷P ? Some methods have a special rule saying that if a program satisfies ✷T , then one can pretend that T is true when reasoning about the program. (The "substitution axiom" of Unity [Chandy and Misra 1988] is such a rule.) In TLA, we use Rule INV2 of Figure 5. This rule implies that having proved Φ ⇒ ✷T , we can rewrite the definition of Φ in Figure 3 as Φ = Init Φ ∧ ✷[M ∧ T ∧ T  ]x, y ∧ WFx, y (M1 ) ∧ WFx, y (M2 ) ∆

It follows that in proving Φ implies ✷P , instead of assuming every step to be a [M]x, y step, we can make the stronger assumption that every step is a [M ∧ T ∧ T  ]x, y step. More precisely, we can substitute M ∧ T ∧ T  instead of M for N in rule INV1, giving a stronger proof rule. This stronger rule is tantamount to "pretending T is true". The validity of this pretense follows directly from the logic; it is not an ad hoc rule. 6.1.5 More About Types. In TLA, variables have no types. Any variable can assume any value. Type-correctness of a program is a provable property, not a syntactic requirement as in strongly-typed programming languages. This has some subtle consequences. Consider the action x = x + 1. Its meaning is a booleanvalued function on pairs of states. Suppose s and t are states that assign the values "abc" and 17 to x, respectively—that is, so s[[x]] equals "abc" and t[[x]] equals 17. Then s[[x = x + 1]]t equals 17 = "abc" + 1. But what is "abc" + 1? Does it equal 17? We don't know the answers to these questions, and we don't care. All we know is that "abc" + 1 is some value. Since that value is either equal to or unequal to 17, the expression 17 = "abc" + 1 is equal to either true or false. More precisely, we assume that m + n is a value, for any values m and n. However, we have no rules for deducing anything about the value of m + n except when m and n are numbers. In general, we assume that all operators such as + are total —they are defined on all possible values. What we usually think of as the domain of an operator is just the set of values for which we know how to evaluate the operator. We know how to evaluate m + n only when m and n are numbers, but it is defined (in the mathematical sense of being a meaningful expression) for all values m and n. Since we can't deduce anything about the value "abc" + 1, whatever we prove about an algorithm is true regardless of what that value is. If we can prove that the program is correct, then either it will never add 1 to "abc" (as in the case of Program 1), or else correctness does not depend on the result of that addition.

Leslie Lamport

gramming languages, but it captures the way mathematicians have reasoned for thousands of years. For example, mathematicians would say that the formula (n ∈ Nat) ⇒ (n + 1 > n)

(30)

is true for all n. Substituting "abc" for n yields ("abc" ∈ Nat) ⇒ ("abc" + 1 > "abc") This formula is true regardless of what "abc" + 1 equals, and whether or not that value is greater than "abc", because "abc" ∈ Nat is false (Note 13). The formula is not meaningless or "type-incorrect" just because we don't know the value of "abc" + 1. There is one subtle pitfall raised by the absence of types in TLA. It is tempting to think that we can replace x = x + 1 by x = x − 1 in the TLA formula Φ describing Program 1. However, these two expressions need not be equivalent unless x and x are both numbers. For example, if x equals 16, then x = x + 1 is true only if x equals 17. However, we don't know what "abc" − 1 equals, so it might equal 16. Hence, the formula 16 = x − 1 might be true when x equals "abc" as well as when x equals 17. We could not prove the property Φ ⇒ ✷(x ∈ Nat) if we replaced x = x + 1 by x = x − 1 in the definition of Φ. We can avoid this pitfall by writing actions as conjunctions and disjunctions of formulas of the form v  = e and v  ∈ e, where v is a variable and e a state function. The absence of types then produces no surprises. Moreover, actions written in this form have the advantage of being easier to understand, since they express the new values of variables directly in terms of their old values. We could define a typed version of TLA. Semantically, we just restrict the collection of states to ones that assign to each variable a value of the proper type. However, types add a great deal of complexity to a logic. For example, what is the type of the division operator? If it is Real × NonzeroReal → Real, then type correctness becomes undecidable, so we lose automatic type checking, arguably the major benefit of types. If the type is Real × Real → Real, then what is the value of 1/0? If it is a real number, then we will be able to prove the correctness of algorithms that we would usually consider to be incorrect, such as an iterative algorithm that takes 1/0 as an initial approximation. Letting 1/0 be a special "undefined" value ⊥ leads to a complicated logic with truth values true, false, and ⊥. The type of an operator like division is just one of many problems introduced by types. These problems are easily hidden in informal presentation such as ours and the ones in most articles and books—for example, [Manna and Pnueli 1991] and [Chandy and Misra 1988]. The problems cannot be avoided in a formal treatment, such as is necessary for true mechanical verification. (They can be hidden when mechanically checking hand-translated verification conditions). Although one can formalize a typed version of TLA, the result is not nearly so simple as the untyped version. Types may be good for programming languages, but we believe that the difficulties they add far outweigh their advantages in a logic for reasoning about algorithms (Note 14). 6.2 Eventuality Properties The second class of properties we consider are eventuality properties—ones asserting that something eventually happens. Here are some traditional eventuality

The Temporal Logic of Actions

properties and their expressions in temporal logic: termination. The program eventually terminates: ✸ terminated. service. If a process has requested service, then it eventually is served: requested ❀ served . message delivery. If a message is sent often enough, then it is eventually delivered: (✷✸ sent) ⇒ ✸ delivered . Although eventuality properties are expressed by a variety of temporal formulas, their proofs can always be reduced to the proof of leads-to properties—formulas of the form P ❀ Q. For example, suppose we want to prove that Program 1 increases the value of x without bound. The TLA formula to be proved is Φ ∧ (n ∈ Nat) ⇒ ✸(x > n)

(31)

The Lattice Rule of Figure 5 together with some simple temporal reasoning shows that (31) follows from Φ ⇒ ((n ∈ Nat ∧ x = n) ❀ (x = n + 1))

(32)

To illustrate the use of TLA in proving leads-to properties, we now sketch the proof of (32). Since safety properties don't imply that anything ever happens, leads-to properties must be derived from the program's fairness condition. Examining Figure 5 leads us to try rule WF1, with the following substitutions: P ← n ∈ Nat ∧ x = n Q←x=n+1

N ←M A ← M1

f ← x, y

The rule's hypotheses become (n ∈ Nat ∧ x = n) ∧ [M]x, y ⇒ ((n ∈ Nat ∧ x = n) ∨ (x = n + 1)) (n ∈ Nat ∧ x = n) ∧ M1 x, y ⇒ (x = n + 1) (n ∈ Nat ∧ x = n) ⇒ Enabled M1 x, y which follow easily from the definitions of M1 and M in Figure 3. The rule's conclusion becomes ✷[M]x, y ∧ WFx, y (M1 ) ⇒ ((n ∈ Nat ∧ x = n) ❀ (x = n + 1)) which, by definition of Φ, implies (32). 6.3 Other Properties We have seen how invariance properties and eventuality properties are expressed as TLA formulas and proved. But, what about more complicated properties? How would one state the following property as a TLA formula? A behavior begins with x and y both zero, and repeatedly increments either x or y (in a single operation), choosing nondeterministically between them, but choosing each infinitely many times. The answer, of course, it that we already have expressed this property in TLA. It is formula Φ of Figure 3. In TLA, there is no distinction between a program and a property. Instead of viewing Φ as a description of a program, we can just as well consider it to be a

Leslie Lamport var integer x, y =0; semaphore sem = 1 ; cobegin loop α1 :  P (sem)  ; β1 :  x := x + 1  ; endloop γ1 :  V (sem)  loop α2 :  P (sem)  ; β2 :  y := y + 1  ; endloop γ2 :  V (sem)  coend Fig. 6.

Program 2—our second example program.

property that we want a program to satisfy. The formula Φ, like the program of Figure 1 that it represents, is so simple that we can regard it as a specification of how we want a program to behave. As our next example, we consider a program that implements property Φ. That is, we give a program represented by a TLA formula Ψ that implies Φ. 7. ANOTHER EXAMPLE 7.1 Program 2 Our next example is Program 2 of Figure 6, written in a language invented for this program. (Since its only purpose is to help us write the TLA formula, the programming-language description of the program can be written with any convenient notation.) The program consists of two processes, each repeatedly executing a loop that contains three atomic operations. The variable sem is an integer semaphore, and P and V are the standard semaphore operations [Dijkstra 1968]. Since Figure 6 is an informal description, it doesn't matter whether or not you understand it. The real definition of Program 2 is the TLA formula Ψ defined below. Describing the execution of Program 2 as a sequence of states requires each state to specify not only the values of the variables x, y, and sem, but also the control state of each process. Control in process 1 can be at one of the three "control points" α1 , β1 , or γ1 . We introduce the variable pc 1 that will assume the values "a", "b", and "g", denoting that control is at α1 , β1 , and γ1 , respectively. A similar variable pc 2 denotes the control state of process 2. The definition of the TLA formula Ψ that represents Program 2 is given in Figure 7 .4 A vertically aligned list of formulas preceded by "∧"s or "∨"s denotes the conjunction or disjunction of those formulas, and we use indentation to eliminate parentheses. (These notational conventions make large formulas much easier to read.) Thus, Figure 7 defines the predicate Init Ψ to be the conjunction of three formulas, the second of which is (x = 0) ∧ (y = 0). As we explained in Section 5.5, a program is represented by a formula Init ∧ ✷[N ]f ∧F . In this example, Init and f are fairly obvious: Init is the predicate Init Ψ that specifies the initial values of the variables, and f is the 5-tuple w consisting of all the program's variables. The next-state relation N and the fairness requirement F are less obvious and merit some discussion. 4 Section 9.2 discusses why Figure 7 is longer and seems more complex than Figure 6.

The Temporal Logic of Actions

Init Ψ = ∧ (pc 1 = "a") ∧ (pc 2 = "a") ∧ (x = 0) ∧ (y = 0) ∧ sem = 1 ∆

α1 = ∧ (pc 1 = "a") ∧ (0 < sem) ∧ pc 1 = "b" ∧ sem  = sem − 1 ∧ Unchanged x, y, pc 2 

α2 = ∧ (pc 2 = "a") ∧ (0 < sem) ∧ pc 2 = "b" ∧ sem  = sem − 1 ∧ Unchanged x, y, pc 1 

β1 = ∧ pc 1 = "b" ∧ pc 1 = "g" ∧ x = x + 1 ∧ Unchanged y, sem, pc 2 

β2 = ∧ pc 2 = "b" ∧ pc 2 = "g" ∧ y = y + 1 ∧ Unchanged x, sem, pc 1 

γ1 = ∧ pc 1 = "g" ∧ pc 1 = "a" ∧ sem  = sem + 1 ∧ Unchanged x, y, pc 2 

γ2 = ∧ pc 2 = "a" ∧ pc 2 = "g" ∧ sem  = sem + 1 ∧ Unchanged x, y, pc 1 

N1 = α1 ∨ β1 ∨ γ1

N2 = α2 ∨ β2 ∨ γ2

= N1 ∨ N2 ∆

w = x, y, sem, pc 1 , pc 2  ∆

Ψ = Init Ψ ∧ ✷[N ]w ∧ SFw (N1 ) ∧ SFw (N2 ) Fig. 7.

The formula Ψ describing Program 2.

7.1.1 The Next-State Relation. Corresponding to the six atomic operations in Figure 6 are the six actions α1 , . . . , γ2 defined in Figure 7. The four conjuncts in the definition of α1 assert that an α1 step: (1) Starts in a state with pc 1 = "a" (control in the first process is at control point α1 ) and 0 < sem (the semaphore is positive). (2) Ends in a state with pc 1 = "b" (control in the first process is at control point β1 ). (3) Decrements sem. (4) Does not change the values of x, y, and pc 2 Thus, an α1 step represents an execution of statement α1 of Figure 6. Similarly, the other actions represent the other operations of the program in Figure 6. An N1 step is either an α1 step, a β1 step, or a γ1 step, so it represents an execution of an atomic operation by the first process. Similarly, an N2 step represents an execution of an atomic operation by the second process. An N step represents a step of either process, so every program step is an N step—in other words, N is the program's next-state relation. Thus, ✷[N ]w is true for a behavior iff every step of the behavior is either a program step or else leaves the variables x, y, sem, pc 1 , and pc 2 unchanged. 7.1.2 The Fairness Requirement. We want program Ψ to implement program Φ. Hence, Ψ must guarantee that both x and y are incremented infinitely often. To guarantee that x is incremented infinitely often, we need some fairness requirement to ensure that infinitely many N1 steps occur. This requirement must rule out the following behavior, in which process 1 is never executed. ∆

((x = 0, y = 0, sem = 1, pc 1 = "a", pc 2 = "a", . . . )) ∆ ∆ ∆ ∆ ∆ ((x = 0, y = 0, sem = 0, pc 1 = "a", pc 2 = "b", . . . ))

Leslie Lamport ∆

((x = 0, y = 1, sem = 0, pc 1 = "a", pc 2 = "g", . . . )) ∆ ∆ ∆ ∆ ∆ ((x = 0, y = 1, sem = 1, pc 1 = "a", pc 2 = "a", . . . )) ∆ ∆ ∆ ∆ ∆ ((x = 0, y = 1, sem = 0, pc 1 = "a", pc 2 = "b", . . . )) ∆ ∆ ∆ ∆ ∆ ((x = 0, y = 2, sem = 0, pc 1 = "a", pc 2 = "g", . . . )) ∆ ∆ ∆ ∆ ∆ ((x = 0, y = 2, sem = 1, pc 1 = "a", pc 2 = "a", . . . )) .. . Observe that an α1 step is possible iff pc 1 equals "a" and sem is positive, so Enabled α1 equals (pc 1 = "a") ∧ (0 < sem). In this behavior, Enabled α1 is true whenever pc 2 equals "a", and false otherwise—both situations occurring infinitely often. An α1 step is also an N1 step. Moreover, every α1 step changes pc 1 and sem, so it changes w. Hence, any α1 step is an N1 w step, so N1 w is enabled and disabled infinitely often in this behavior. The weak fairness condition WFw (N1 ) asserts that N1 w is disabled infinitely often or infinitely many N1 w steps occur. Since N1 w is disabled infinitely often, WFw (N1 ) does not rule out this behavior. The strong fairness condition SFw (N1 ) asserts that either N1 w is eventually forever disabled or else infinitely many N1 w steps occur. Neither assertion is true for this behavior, so the behavior does not satisfy SFw (N1 ). This example indicates why we need the fairness condition SFw (N1 ) to guarantee that x is incremented infinitely often. There are other ways of writing this fairness condition. An equivalent definition of Ψ is obtained by replacing SFw (N1 ) with SFw (α1 ) ∧ SFw (β1 ) ∧ SFw (γ1 ) or with SFw (α1 ) ∧ WFw (β1 ) ∧ WFw (γ1 ). Equivalence of these definitions follows from the formulas Init Ψ ∧ ✷[N ]w ⇒ (SFw (N1 ) ≡ SFw (α1 ) ∧ SFw (β1 ) ∧ SFw (γ1 ))

(33)

Init Ψ ∧ ✷[N ]w ⇒ (SFw (β1 ) ≡ WFw (β1 )) Init Ψ ∧ ✷[N ]w ⇒ (SFw (γ1 ) ≡ WFw (γ1 ))

(34) (35)

Intuitively, (33) holds because once control reaches α1 , β1 , or γ1 , it remains there until the corresponding action is executed; (34) holds because once control reaches β1 , action β1 is enabled until it is executed; and (35) is similar to (34). Corresponding reasoning about y and N2 leads to the fairness condition SFw (N2 ) for the second process. 7.2 Proving Program 2 Implements Program 1 To show that Program 2 implements Program 1, we must prove the TLA formula Ψ ⇒ Φ, where Ψ is defined in Figure 7 and Φ is defined in Figure 3. By these definitions, Ψ ⇒ Φ follows from the following three formulas. Init Ψ ⇒ Init Φ

(36)

✷[N ]w ⇒ ✷[M]x, y Ψ ⇒ WFx, y (M1 ) ∧ WFx, y (M2 )

(37) (38)

Formula (36) asserts that the initial condition of Ψ implies the initial condition of Φ. It follows easily from the definitions of Init Ψ and Init Φ . Roughly speaking, formula (37) asserts that every N step simulates an M step, and (38) asserts that Program 2 implements Program 1's fairness conditions. We

The Temporal Logic of Actions

now sketch the proofs of these two formulas. 7.2.1 Proof of Step-Simulation. Applying rule TLA2 of Figure 5 with true substituted for P and Q shows that (37) follows from [N ]w ⇒ [M]x, y

(39)

By definition, [N ]w equals α1 ∨ . . . ∨ γ2 ∨ (w = w) and [M]x, y equals M1 ∨ M2 ∨ (x, y = x, y). Formula (37) therefore follows from α2 ⇒ x, y = x, y α1 ⇒ x, y = x, y β2 ⇒ M 2 β1 ⇒ M 1 γ1 ⇒ x, y = x, y γ2 ⇒ x, y = x, y (w = w) ⇒ x, y = x, y

(40)

These implications are all trivial consequences of the definitions. 7.2.2 Proof of Fairness. For the fairness requirement (38), we sketch the proof that Ψ implies WFx, y (M1 ). The proof that it implies WFx, y (M2 ) is similar. Strong fairness of Program 2 is necessary to insure that x is incremented infinitely often, so Figure 5 suggests applying SF2 (without the overbars). At first glance, SF2 doesn't seem to work because its conclusion implies a strong fairness condition, and we want to prove Ψ ⇒ WFx, y (M1 ). However, if x is a natural number, so x = x + 1, then Enabled M1 x, y equals true. A simple invariance argument proves Ψ ⇒ ✷(x ∈ Nat), so Ψ ⇒ ✷(Enabled M1 x, y ). Hence, Ψ implies that SFx, y (M1 ) and WFx, y (M1 ) are equivalent—both being equal to ✷✸M1 x, y . We can thus expect to prove Ψ ⇒ SFx, y (M1 ), which implies Ψ ⇒ WFx, y (M1 ). Comparing the conclusion of rule SF2 with the formula we are trying to prove apparently leads to the following substitutions in the rule. N ←N

M ← M1

f ←w

g ← x, y

However, it turns out that we need to strengthen N by the use of an invariant. We must find a predicate I (an invariant) that satisfies Init Ψ ∧ ✷[N ]w ⇒ ✷I

(41)

By rule INV2, we can then rewrite Ψ as Init Ψ ∧ ✷[N ∧ I ∧ I  ]w ∧ SFw (N1 ) ∧ SFw (N2 ) and substitute N ∧ I ∧ I  for N . We will discover the invariant I in the course of the proof. The first hypothesis of the rule and (40) suggest substituting β1 for B. The conclusion and the second hypothesis leads to the substitution of N1 for A and SFw (N2 ) for ✷F , using the temporal tautology SFw (N2 ) ≡ ✷SFw (N2 ). The second and fourth hypotheses lead to the substitution of pc 1 = "b" for P . With these substitutions, the proof rule becomes N ∧ I ∧ I  ∧ β1 w ⇒ M1 x, y (pc 1 = "b") ∧ (pc 1 = "b") ∧ N ∧ I ∧ I  ∧ N1 w ⇒ β1 (pc 1 = "b") ∧ Enabled M1 x, y ⇒ Enabled N1 w ✷[N ∧ I ∧ I  ∧ ¬β1 ]w ∧ SFw (N1 ) ∧ SFw (N2 ) ∧ ✷✸Enabled M1 x, y ⇒ ✸✷(pc 1 = "b") ✷[N ∧ I ∧ I  ]w ∧ SFw (N1 ) ∧ SFw (N2 ) ⇒ SFx, y (M1 )

Leslie Lamport

The first three hypotheses are simple action formulas. The second and third follow easily from the definitions of N1 , β1 and M1 . To prove the first hypothesis, we must show that N ∧ I ∧ I  ∧ β1 ∧ (w = w) implies M1 ∧ (x, y = x, y). As we observed in (40), β1 implies M1 . Since β1 also implies x = x + 1, which implies x = x if x is a natural number, the first hypothesis holds if the invariant I implies x ∈ Nat. The fourth hypothesis is a temporal formula, which we now examine. To simplify the intuitive reasoning, let us ignore steps that don't change w. The fourth hypothesis then asserts that if every step is an N ∧ I ∧ I  step that is not a β1 step, and the fairness conditions hold, then eventually control reaches β1 and remains there forever. From the informal description of the program in Figure 6, this seems valid. No matter where control starts in process 1, fairness implies that eventually it must reach β1 , where it must remain forever if no β1 step is performed. Unfortunately, this intuitive reasoning is wrong. The fourth hypothesis is not a valid TLA formula. For example, consider a behavior that starts in a state with pc 1 = pc 2 = "a" and sem = 0, and that remains in this state forever. In such a behavior, the left-side of the implication in the fourth hypothesis is true, but pc 1 never becomes equal to "b". Thus, the hypothesis is not satisfied by these behaviors. The fourth hypothesis is invalid for behaviors starting in "bad" states—ones that are not reachable by executing the program from an initial state satisfying Init Ψ . Such states have to be ruled out by the invariant I. We must substitute SFw (N2 ) ∧ ✷I for ✷F (using the tautology SFw (N2 ) ∧ ✷I ≡ ✷(SFw (N2 ) ∧ I)) and (pc 1 = "b") ∧ I for P in Rule SF2, obtaining the following as the fourth hypothesis. G ⇒ ✸✷((pc 1 = "b") ∧ I)

(42)

where G = ✷[N ∧ I ∧ I ∧ ¬β1 ]w ∧ SFw (N1 ) ∧ SFw (N2 ) ∧ ✷I ∧ ✷✸Enabled M1 x, y ∆

Remembering that I must imply x ∈ Nat, the reader with experience reasoning about concurrent programs will discover that the appropriate invariant is I = ∧ x ∈ Nat ∧ ∨ (sem = 1) ∧ (pc 1 = pc 2 = "a") ∨ (sem = 0) ∧ ∨ (pc 1 = "a") ∧ (pc 2 ∈ {"b", "g"}) ∨ (pc 2 = "a") ∧ (pc 1 ∈ {"b", "g"}) ∆

With this definition of I, the invariance property (41) follows easily from Rule INV1. Having deduced that we need to prove (42), we must understand why it is true. A little thought reveals that (42) holds because control in process 1 must eventually reach β1 , and ✷[N . . . ∧ ¬β1 ]w , which asserts that a β1 action is never executed, implies that control must then remain there forever. This reasoning is formalized by applying simple temporal reasoning based on the Lattice Rule to derive (42) from: G ⇒ ( (pc 1 = "g") ∧ I ❀ (pc 1 = "a") ∧ I )

(43)

G ⇒ ( (pc 1 = "a") ∧ I ❀ (pc 1 = "b") ∧ I ) G ⇒ ( (pc 1 = "b") ∧ I ⇒ ✷((pc 1 = "b") ∧ I) )

(44) (45)

The Temporal Logic of Actions

To see how to prove these formulas, we once again use simple pattern matching against the proof rules of Figure 5. We find that (43) and (44) should be proved by Rule SF1 with N1 substituted for A and SFw (N2 ) substituted for ✷F , and that (45) should be proved by rule INV1 with (pc 1 = "b") ∧ I substituted for I. The proofs of (43) and (45) are simple. The proof of (44) is not so easy, the hard part being the proof of the third hypothesis: (46) H ⇒ ✸Enabled N1 w ∆ where H = ✷((pc 1 = "a") ∧ I) ∧ ✷[N ∧ I ∧ I  ∧ ¬β1 ]w ∧ SFw (N2 ) Once again, we have reached a point where blind application of rules fails; we must understand why (46) is true. If pc 1 equals "a", then action N1 is enabled when control in process 2 is at α2 , and strong fairness for N2 implies that control must eventually reach α2 . This intuitive reasoning leads us to deduce (46) by temporal reasoning from (pc 1 = "a") ∧ I ⇒ (Enabled N1 w ≡ (pc 2 = "a")) H ⇒ ((pc 2 = "b") ❀ (pc 2 = "g")) H ⇒ ((pc 2 = "g") ❀ (pc 2 = "a")) The first formula follows from the observation that Enabled N1 w ≡ ((pc 1 = "a") ∧ (0 < sem)) ∨ (pc 1 = "b") ∨ (pc 1 = "g") Pattern matching against the proof rules leads to simple proofs of the remaining two formulas by substituting N2 for A and true for F in SF1. 7.3 Comments on the Proof This example illustrates the general method of proving that a lower-level program Ψ implements a higher-level program Φ. There are three things to prove: (i) the initial predicate of Ψ implies the initial predicate of Φ, (ii) a step of Ψ simulates a step of Φ, and (iii) Ψ implies the fairness requirement of Φ. As in the example, proving the initial condition is generally straightforward. Of course, in more realistic examples there will be more details to check. Because our example was so simple, the proof of step-simulation was atypical. Usually, a step of the lower-level program starting in a completely arbitrary state does not simulate a step of the higher-level program. We must first find the proper invariant, and then apply Rule INV2 to prove step-simulation. Once the invariant is found, the proof is a straightforward exercise in showing that one action implies another. The structure of the formulas tells us how to decompose a large proof into a number of smaller ones. Our proof of fairness was quite typical in its alternation of blind application of proof rules with the need to understand why a property holds. As in this proof, an invariant is almost always required. Usually, it is the same invariant as in the proof of step-simulation. Of course, the proofs of real algorithms will be more complicated. Our proof may already have seemed rather complicated for such a simple example, but the example is a bit more subtle than it appears. The reader who attempts a rigorous informal proof will discover that each step in the TLA proof mirrors a step in the informal proof. The more rigorous the informal proof, the more it will resemble the TLA proof. Rules SF1 and SF2 conveniently encapsulate reasoning

Leslie Lamport

that occurs over and over again in informal proofs. We believe that temporal logic provides an ideal formalism for translating intuitive understanding of why a liveness property holds into a formal proof. Were we to choose the weaker fairness requirement WFx, y (M) for Program 1, then Program 2's fairness requirement could be weakened to WFw (N ). The proof of Ψ ⇒ Φ, using WF2 instead of SF2, would then be simpler. Writing out this proof is a good exercise in applying TLA. 7.4 Stuttering and Refinement Program 2 is finer grained than Program 1, in the sense that the three atomic operations of each process's loop in Program 2 correspond to a single atomic operation of Program 1. Besides the steps that increment x or y, Program 2 takes steps that modify sem and pc 1 or pc 2 , but leave x and y unchanged. Program 2 implements Program 1—that is, the formula Ψ ⇒ Φ is valid—only because Φ allows stuttering steps that do not change x and y. Program 2 can in turn be implemented by a still finer-grained program because Ψ allows steps that do not change any of its variables. Allowing stuttering steps is the key to refining the grain of atomicity. 8. HIDING VARIABLES 8.1 A Memory Specification We now consider another example: a simple processor/memory interface. The processor issues read and write operations that are executed by the memory. The interface consists of three registers, represented by the following three variables. op

Set by the processor to indicate the desired operation, and reset by the memory after executing the operation. adr Set by the processor to indicate the address of the memory location to be read or written. val Set by the processor to indicate the value to be written by a write, and set by the memory to return the result of a read. Here is a typical behavior, where "—" indicates that the value is irrelevant, and memory location 432 happens to have the initial value 777. ∆

((op = "ready", adr = — , val = — , . . . )) ∆ ∆ ∆ ((op = "read", adr = 432, val = — , . . . )) ∆ ∆ ∆ ((op = "ready", adr = — , val = 777, . . . )) ∆ ∆ ∆ ((op = "write", adr = 196, val = 0, . . . )) ∆ ∆ ∆ ((op = "ready", adr = — , val = — , . . . )) .. . It is easy to specify this interface if we introduce an additional variable memory to denote the contents of memory, so memory (n) is the current value of memory location n. The property Φ describing the desired behaviors is shown in Figure 8, where Address is the set of legal addresses, and MemVal is the set of possible memory values. Action S(m, v) represents the assignment memory(m) := v (Note 15). Actions Rproc and Wproc represent the processor's read- and write-request operations; actions Rmem and Wmem represent the memory's responses to those requests.

The Temporal Logic of Actions

Init Φ = ∧ op = "ready" ∧ ∀n ∈ Address : memory (n) ∈ MemVal ∆

S(m, v) = ∀n ∈ Address : ∧ (n = m) ⇒ (memory (n) = v) ∧ (n = m) ⇒ (memory (n) = memory(n)) ∆

Rmem

= ∧ op = "read" ∧ op  = "ready" ∧ val  = memory(adr ) ∧ memory  = memory

Wmem

= ∧ op = "write" ∧ op  = "ready" ∧ S(adr , val )

Rproc = ∧ op = "ready" ∧ op  = "read" ∧ adr  ∈ Address ∧ memory  = memory Wproc = ∧ op = "ready" ∧ op  = "write" ∧ adr  ∈ Address ∧ val  ∈ MemVal ∧ memory  = memory Nmem N

= Rmem ∨ Wmem

= Nmem ∨ Rproc ∨ Wproc ∆

w = op, adr , val , memory  ∆

Φ = Init Φ ∧ ✷[N ]w ∧ WFw (Nmem ) Fig. 8.

"Internal" specification of a processor/memory interface.

Action Nmem denotes the memory's next-state relation. The fairness condition WFw (Nmem ) asserts that the memory eventually responds to each request; there is no requirement that the processor ever issues requests. Observe that the action S(m, v) is used only to define Wmem ; it was introduced just to keep the definition of Wmem from running off the page. There is no formal significance to our choice of names such as Rproc . Our decision to define Nmem as the disjunction of two simpler actions was completely arbitrary; we could just as well have defined it all at once, or as the disjunction of more than two actions. There are countless ways of writing logically equivalent formulas Φ. The formula Φ specifies the right behavior for the interface variables op, adr , and val . However, it also specifies the value of the variable memory, which we did not want to specify. We want to specify only how the three interface variables change; we do not care how any other variables such as x , sem, or memory change. We therefore want a formula asserting that op, adr , and val behave as described by Φ, but that it doesn't matter what values memory assumes. Such a formula is sometimes described as Φ with the variable memory "hidden". This formula is written ∃ memory : Φ. The precise meaning of the formula ∃ memory : Φ is defined below. Here, we simply want to observe that the free (flexible) variables of this formula are op, adr , and val . Since x , sem, and memory do not occur free, the formula does not constrain them in any way. 8.2 Quantification over Flexible Variables We now define ∃ x : F , where x is a (flexible) variable and F a temporal formula. Intuitively, ∃ x : F asserts that it doesn't matter what the actual values of x are, but that there are some values x can assume for which F holds. For example, ∃ x : ✷[y = x ]x, y is satisfied by the behavior ∆

((x = "a", y = 0, z = "uvw",. . . ))

Leslie Lamport

((x = "b", y = 1, z = −13, . . . )) ∆ ∆ ∆ ((x = "c", y = 1, z = −13, . . . )) ∆ ∆ ∆ ((x = 77, y = 2, z = "vw", . . . )) .. . ∆

because by changing only the values of x, we get the following behavior that satisfies ✷[y = x ]x, y . ∆

((x = "a", ∆ ((x = 0, ∆ ((x = 1, ∆ ((x = 1, .. .

y = 0, z = "uvw",. . . )) ∆ ∆ y = 1, z = −13, . . . )) ∆ ∆ y = 1, z = −13, . . . )) ∆ ∆ y = 2, z = "vw", . . . ))

In fact, every behavior satisfies ∃ x : ✷[y = x ]x, y . To define ∃ x : F formally, we need some auxiliary definitions. For any variable x and states s and t, let s =x t mean that s and t assign the same values to all variables other than x. More precisely, s =x t = ∀ 'v ' = 'x ' : s[[v]] = t[[v]] ∆

We extend the relation =x to behaviors in the obvious way: s0 , s1 , . . .  =x t0 , t1 , . . .  = ∀ n ∈ Nat : sn =x tn ∆

The obvious next step is to define σ[[∃ x : F ]] = ∃ τ ∈ St∞ : (σ =x τ ) ∧ τ [[F ]] ∆

(47)

for any behavior σ. (Recall that St is the collection of all behaviors.) However, this definition is not quite right, because the formula it defines is not necessarily invariant under stuttering. For example, suppose F is satisfied only by behaviors in which x changes before y does, including the behavior ∆

((x = 1, y = "a", z = 7, . . .)) ∆ ∆ ∆ ((x = 2, y = "a", z = 7, . . .)) ∆ ∆ ∆ ((x = 2, y = "b", z = 14, . . .)) .. . Then definition (47) implies that the behavior ∆

((x = 999, y = "a", z = 7, . . .)) ∆ ∆ ∆ ((x = 999, y = "a", z = 7, . . .)) ∆ ∆ ∆ ((x = 999, y = "b", z = 14, . . .)) .. . satisfies ∃ x : F (because we can produce a behavior satisfying F by changing only the values of x). However, the behavior ∆

((x = 999, y = "a", z = 7, . . .)) ∆ ∆ ∆ ((x = 999, y = "b", z = 14, . . .)) .. .

The Temporal Logic of Actions

does not satisfy ∃ x : F (because of the assumption that F requires x to change before y does). With appropriate values for all other variables, these two behaviors differ only by stuttering steps. Hence, with definition (47), ∃ x : F is not necessarily invariant under stuttering even though F is. To obtain invariance under stuttering, we must define ∃ x : F to be satisfied by a behavior σ iff we can obtain a behavior that satisfies F by first adding stuttering and then changing the values of x. We define #σ to be the behavior obtained from the behavior σ by removing all stuttering steps—except that if σ ends with infinite stuttering, then those final stuttering steps are kept. The precise definition is: #s0 , s1 , s2 , . . .  = if ∀n ∈ Nat : sn = s0 then s0 , s0 , s0 , . . .  else if s1 = s0 then #s1 , s2 , s3 , . . .  else s0  ◦ #s1 , s2 , . . .  ∆

(48)

where ◦ denotes concatenation of sequences. We then define ∃ by σ[[∃ x : F ]] = ∃ ρ, τ ∈ St∞ : (#σ = #ρ) ∧ (ρ =x τ ) ∧ τ [[F ]] ∆

(49)

The operator ∃ x differs from ordinary existential quantification because it asserts the existence not of a single value to be substituted for x, but of an infinite sequence of values. However, it really is existential quantification because it obeys the ordinary laws of existential quantification. In particular, the usual rules E1 and E2 of Figure 9 are sound. From these rules, one can deduce the expected properties of existential quantification, such as (∃ x : F ∨ G) ≡ (∃ x : F ) ∨ (∃ x : G) We can extend TLA to allow quantification over rigid as well as flexible variables. Since the value of a rigid variable is constant throughout a behavior, quantification over rigid variables is much simpler than quantification over flexible variables (Note 16). However, it is of less use. The semantics of quantification over rigid variables is defined in Figure 9. General TLA formulas consist of all formulas obtained from simple TLA formulas by logical operators and quantification over program and rigid variables. The syntax and semantics of quantification are summarized in Figure 9, which together with Figure 4 gives the complete definition of TLA. It is easy to check that TLA formulas are invariant under stuttering, which means formally that #σ = #τ implies σ[[F ]] = τ [[F ]] for all TLA formulas F and behaviors σ and τ . 8.3 Refinement Mappings 8.3.1 Implementing The Memory Specification. We now give a simple implementation of the processor/memory interface specified by the formula ∃ memory : Φ, where Φ is defined in Figure 8. The implementation uses a main memory and a cache, represented by variables main and cache. The value of cache(m) represents the cache's value for memory location m, the special value ⊥ (assumed not to be in MemVal) denoting that this memory location is not in the cache. The processor's read and write requests are serviced from the cache, and separate internal actions (not visible from the interface) move values between the cache and main memory. When the processor reads a value not in the cache, the value is first moved into the cache and then put in val .

Leslie Lamport

Syntax ∆

general formula = formula | ∃ variable : general formula | ∃ rigid variable : general formula | general formula ∧ general formula | ¬general formula ∆

formula

= a simple TLA formula (see Figure 4)

Semantics ∆

s0 , s1 , . . .  =x t0 , t1 , . . .  = ∀ n ∈ Nat : ∀ 'v ' = 'x ' : sn [[v]] = tn [[v]] ∆

s0 , s1 , s2 , . . .  = if ∀n ∈ Nat : sn = s0 then s0 , s0 , s0 , . . .  else if s1 = s0 then s1 , s2 , s3 , . . .  else s0  ◦ s1 , s2 , . . .  ∆ σ[[∃ x : F ]] = ∃ ρ, τ ∈ St∞ : (σ = ρ) ∧ (ρ =x τ ) ∧ τ [[F ]] ∆

σ[[∃ c : F ]] = ∃c ∈ Val : σ[[F ]] Proof Rules E1.  F (f /x) ⇒ ∃ x : F

F 1.  F (e/c) ⇒ ∃ c : F

E2. F ⇒ G x does not occur free in G (∃ x : F ) ⇒ G F 2. F ⇒ G c does not occur free in G (∃ c : F ) ⇒ G

where x is a variable f is a state function c is a rigid variable e is a constant expression

F , G are general formulas s, s0 , t0 , s1 , t1 , . . . are states σ is a behavior ◦ denotes concatenation of sequences

Fig. 9.

Quantification in TLA.

The "internal" description, in which main and cache are free variables, is the formula Ψ of Figure 10. The actions defined in the figure have the following interpretations. T (a, m, v ). Represents the assignment a(m) := v. This action is introduced only to simplify the definitions of other actions. Rpro , Wpro . The processor's read- and write-request operations. Rcch , Wcch . The memory's responses to processor requests, the value being read from or written to the cache. An Rcch action can be executed only if the value to be read is in the cache. Cget (m), Cfl (m). The internal actions of moving a value from main memory to the cache, and of flushing a value from the cache to main memory. The second conjunct of Cfl (m) prevents a value from being flushed while it is being read. This is the only constraint on when values can be moved into or out of the cache; no particular cache maintenance policy is specified. P. The next-state relation, which is the disjunction of all possible actions of the processor and the memory. F . The disjunction of all the memory actions that must be performed to respond to a processor request. The third disjunct represents the action of moving the value

The Temporal Logic of Actions

Init Ψ = ∧ op = "ready" ∧ ∀n ∈ Address : (main(n) ∈ MemVal) ∧ (cache(n) = ⊥) T (a, m, v ) = ∀n ∈ Address : ∧ (n = m) ⇒ (a  (n) = v ) ∧ (n = m) ⇒ (a  (n) = a(n)) ∆

Rcch = ∧ op = "read" ∧ cache(adr ) = ⊥ ∧ op  = "ready" ∧ val  = cache(adr ) ∧ Unchanged main, cache

Wcch = ∧ op = "write" ∧ op  = "ready" ∧ T (cache, adr , val ) ∧ Unchanged main

Rpro = ∧ op = "ready" ∧ op  = "read" ∧ adr  ∈ Address ∧ Unchanged main, cache Wpro = ∧ op = "ready" ∧ op  = "write" ∧ adr  ∈ Address ∧ val  ∈ MemVal ∧ Unchanged main, cache ∆

Cget (m) = ∧ cache(m) = ⊥ ∧ T (cache, m, main(m)) ∧ Unchanged op, adr , val , main

Cfl (m) = ∧ cache(m) = ⊥ ∧ ∨ op = "read" ∨ m = adr ∧ T (main, m, cache(m)) ∧ T (cache, m, ⊥) ∧ Unchanged op, adr , val

P = Rpro ∨ Wpro ∨ Rcch ∨ Wcch ∨ (∃ m ∈ Address : Cget (m) ∨ Cfl (m)) ∆

F = Rcch ∨ Wcch ∨ (Cget (adr ) ∧ (op = "read")) ∆

u =

op, adr , val , main, cache

Ψ = Init Ψ ∧ ✷[P]u ∧ WFu (F) Fig. 10.

A simple cached memory.

for a read request from main memory into the cache. (It is enabled only if the value is not already in the cache.) If we consider main and cache to be internal variables, then the cached memory is described by the TLA formula5 ∃ main, cache : Ψ. The assertion that the cached memory correctly implements the processor/memory interface is expressed by the formula ∃ main, cache : Ψ) ⇒ (∃ ∃ memory : Φ) (∃

(50)

To prove (50), we define the state function memory by memory(m) = if cache(m) = ⊥ then main(m) else cache(m) ∆

and then prove Ψ ⇒ Φ, where Φ denotes the formula Φ(memory/memory) obtained by substituting memory for all free occurrences of memory in Φ. Applying rule E1 of Figure 9, substituting memory for f and memory for x, we obtain Ψ ⇒ ∃ memory : Φ. Rule E2 then yields (50). 5 As usual in logic, we write ∃ x, y : F as an abbreviation for ∃ x : ∃ y : F , which by E1 and E2 of Figure 9 is equivalent to ∃ y : ∃ x : F .

Leslie Lamport

The formula Ψ ⇒ Φ asserts that any sequence of values for the variables op, adr , and val , and for the state function memory, that is allowed by Ψ is a sequence of values that Φ allows for the variables op, adr , val , and memory. We can regard memory as the "concrete" state function with which Ψ implements the "abstract" variable memory. How do we prove that Ψ implies Φ? To find the answer, we examine the structure of Φ. For any formula F , let F denote the formula F (memory/memory) obtained by substituting memory for all free occurrences of memory in F . For example, w is the state function op, adr , val , memory. Then Φ equals Init Φ ∧ ✷[N ]w ∧ WFw (Nmem ). The formula Φ therefore looks much like an ordinary TLA formula representing a program, with initial condition Init Φ and next-state relation N . The only difference is that instead of an ordinary weak fairness condition, Φ has as a conjunct the "barred" fairness condition WFw (Nmem ). The proof of Ψ ⇒ Φ is similar to the proof in Section 7.2 that Program 2 implements Program 1. We first prove that Init Ψ implies Init Φ . We next prove that Ψ implies ✷[N ]w (step-simulation) by applying rule TLA2 of Figure 5 with the substitutions A←P

B←N

f ←u

g←w

P ← true

Q ← true

Finally, we prove that Ψ implies WFw (Nmem ) (fairness) by applying WF2 with the substitutions M ← Nmem A←F f ←u N ←P B ← Rcch ∨ Wcch g←w P ← (op = "write") ∨ (op = "read" ∧ cache(adr ) = ⊥) (Observe that Rule WF2 has the appropriate "bars" to prove the desired conclusion.) As in our previous example, the proofs consist of straightforward calculations punctuated by the occasional need for insight into why what we are trying to prove is true. This cached memory is quite abstract; it allows any policy for deciding when to move values between the cache and main memory. Given a particular caching algorithm, we would prove that it implements the simple cached memory—meaning that the TLA formula representing the algorithm implies ∃ main, cache : Ψ. By the transitivity of implication, this proves that the algorithm implements the memory/processor interface. 8.3.2 Refinement Mappings. It is clear how to generalize the example above to the problem of proving (∃ x1 , . . . , xm : Ψ) ⇒ (∃ y1 , . . . , yn : Φ)

(51)

for arbitrary Ψ and Φ. We must define state functions y1 , . . . , yn in terms of the variables that occur in Ψ and prove Ψ ⇒ Φ, where for any formula F , we let F denote the formula F (y1 /y1 , . . . , yn /yn ) obtained by substituting yi for the free occurrences of yi in F , for all i. We then infer (51) from rules E1 and E2. The collection of state functions y1 , . . . , yn is called a refinement mapping. The "barred variable" yi is the state function with which Ψ implements the variable yi of Φ. To prove (51), one must find a refinement mapping such that Ψ ⇒ Φ is valid, and use the rules of Figure 5 to prove its validity. But can the requisite refinement

The Temporal Logic of Actions

mapping always be found? Does the validity of (51) imply the existence of a refinement mapping such that Ψ ⇒ Φ is valid? The answer is no; a refinement mapping need not exist. As an example, we return to Programs 1 and 2, represented by formulas Φ of Figure 3 and Ψ of Figure 7. Program 2 permits precisely the same sequences of values for x and y as does Program 1. Therefore, the formula ∃ sem, pc 1 , pc 2 : Ψ, which describes only the sequences of values for x and y allowed by Program 2, is equivalent to Φ. Can we prove this equivalence? We already sketched the proof of Ψ ⇒ Φ, which by Rule E2 implies (∃ sem, pc 1 , pc 2 : Ψ) ⇒ Φ In this case, Φ has no internal variables, so the refinement mapping is the trivial one consisting of the empty set of barred variables. Now consider the converse, Φ ⇒ (∃ sem, pc 1 , pc 2 : Ψ)

(52)

Can we define the requisite state functions sem, pc 1 , and pc 2 in terms of x and y (the only variables that occur in Φ) so that Program 1 allows them to assume only those sequences of values that Program 2 allows the corresponding variables to assume? Clearly not. There is no way to infer from the values of x and y what the values of sem, pc 1 , and pc 2 should be. Thus, there does not exist a refinement mapping for which Φ implies Ψ. To prove (52), one must modify Φ by adding auxiliary variables. Intuitively, an auxiliary variable is one that is added to a program without affecting the program's behavior. Formally, adding an auxiliary variable d to a formula Π means finding a formula Πd such that ∃ d : Πd is equivalent to Π. (The variable d is assumed not to occur free in Π.) Formula (52) can be proved by adding two auxiliary variables h and p to Φ. That is, we can construct a formula Φhp such that ∃ h, p : Φhp is equivalent to Φ, and can then prove (∃ h, p : Φhp ) ⇒ (∃ sem, pc 1 , pc 2 : Ψ) by constructing a refinement mapping such that Φhp implies Ψ. The refinement mapping can be found because the state functions sem, pc 1 , and pc 2 are allowed to depend upon h and p as well as x and y. In general, refinement mappings can be found if we add the right auxiliary variables. The completeness theorem of Abadi and Lamport [1991] shows that, under certain reasonable assumptions about Φ and Ψ, if (51) is valid, then one can in principle add auxiliary variables to Φ to obtain the formula Φhp and find the requisite refinement mapping such that Φhp ⇒ Ψ is valid. Relative completeness of STL1–STL6, the Lattice Rule, TLA1, and TLA2 for simple TLA means that this implication is provable from those rules if Φ and Ψ have the form Init ∧ ✷[N ]v ∧ F , where F is the conjunction of weak and strong fairness formulas, and Φ is machine closed. We thus have a relative completeness result for TLA formulas of the form (51). 8.3.3 "Barring" Fairness. The "barring" operator denotes substitution of state functions yi for variables yi . Barring distributes over most of our operators; for example, ✷(F ∨ G) equals ✷(F ∨ G), for any formulas F and G. Thus, when Φ has the canonical form Init ∧ ✷[N ]f ∧ F , the formula Φ equals Init ∧ ✷[N ]f ∧ F . If F

Leslie Lamport

is the conjunction of fairness conditions of the form WFg (M) and SFg (M), then F is the conjunction of barred fairness conditions WFg (M) and SFg (M). We might expect that WFg (M) would be equivalent to WFg (M) and SFg (M) equivalent to SFg (M), but that need not be the case. It is true that WFg (M) ≡ ✷✸¬Enabled Mg ∨ ✷✸Mg SFg (M) ≡ ✷✸¬Enabled Mg ∨ ✸✷Mg

(53)

However, Enabled Mg is not necessarily equivalent to Enabled Mg . For example, let M be the action (x = x) ∧ (y  = y), let g equal x, y, and let the refinement mapping be defined by x = z and y = z. Then Enabled Mg equals ∃ c, d : (c = x) ∧ (d = y), which equals true. Hence Enabled Mg , the formula obtained by substituting x for x and y for y in Enabled Mg , equals true. But Enabled Mg ≡ Enabled (x = x) ∧ (y  = y)x, y ≡ Enabled (x  = x ) ∧ (y  = y)x ,y ≡ Enabled (z  = z) ∧ (z  = z)z,z ≡ Enabled false ≡ false

by definition of M and g by definition of . . . by definition of x and y by definition of  . . .... by definition of Enabled

Thus, Enabled Mg is not equivalent to Enabled Mg . In general, the primed variables in the action Mg are not free variables of the expression Enabled Mg , so we can't obtain Enabled Mg from Enabled Mg by blindly barring all variables. In rules WF2 and SF2, the formulas WFg (M) and SFg (M) are defined by (53). The rules are sound when M is any action, g any state function, and Enabled Mg any predicate—assuming that WFg (M) and SFg (M) are defined by (53). In practice, the barred formulas will be obtained from unbarred ones by substituting barred variables (state functions) for variables, as in our example. 9. FURTHER COMMENTS 9.1 Mechanical Verification Because it is a simple logic, TLA is ideally suited for mechanization. Urban Engberg and Peter Grønning have been working on the mechanical verification of TLA, using LP—an "off-the-shelf" verification system based on rewriting [Garland and Guttag 1989]. Although initial experiments showed that LP can be used directly, Engberg and Grønning decided to develop a system called TLP to translate TLA definitions and proofs into LP input [Engberg et al. 1992] (Note 17). In addition to allowing more readable specifications, TLP allows separate LP proofs for action formulas and temporal formulas, using simpler encodings of the formulas than would be possible with a single proof. Since most reasoning in a TLA proof is about actions, a simple encoding of action formulas is important. They also hope to use verification systems other than LP to check parts of the proof. The proof in Section 7.2, that the formula Ψ describing Program 2 implies the formula Φ describing Program 1, has been checked with TLP. Figure 11 shows the definitions of Φ and Ψ in the actual TLP input. (For simplicity, we are omitting some declarations and TLP directives.) Observe that these definitions are almost

The Temporal Logic of Actions InitPhi == == == == == Phi ==

(x = 0) /\ (y = 0) (x  = x + 1) /\ (y  = y) (y  = y + 1) /\ (x  = x) M1 \/ M2 (x * y) InitPhi /\ [][M]_v /\ WF(v,M1) /\ WF(v,M2)

InitPsi

==

alpha1

==

/\ (pc1 = a) /\ (pc2 = a) /\ (x = 0) /\ (y = 0) /\ sem = 1 /\ (pc1 = a) /\ (0 << sem) /\ pc1  = b /\ sem  = sem - 1 /\ Unchanged(x * y * pc2)

.. . gamma2

==

N1 N2 N w Psi

== == == == ==

/\ pc2 = g /\ pc2  = a /\ sem  = sem + 1 /\ Unchanged(x * y * pc1) alpha1 \/ beta1 \/ gamma1 alpha2 \/ beta2 \/ gamma2 N1 \/ N2 (x * y * pc1 * pc2 * sem) InitPsi /\ [][N]_w /\ SF(w,N1) /\ SF(w,N2)

Fig. 11: The representation of the formulas Φ and Ψ of Figures 3 and 7 for the mechanical verification of the theorem Ψ ⇒ Φ.

perfect transliterations of the ones in Figures 3 and 7. The major differences are the use of "*" to represent tuples and "<<" instead of "<"—differences introduced because comma and "<" have other meanings. Following these definitions is the statement Theorem

Psi => Phi

that asserts the validity of the temporal formula Ψ ⇒ Φ. The rest of the TLP input is a hierarchically structured proof of this theorem. TLP translates the definitions and proof into a form that can be checked by LP. The only part of the proof not checked by LP is the computation of the Enabled predicates. Although algorithmically simple, these computations are awkward to do in LP. We hope that a future version of TLP will compute Enabled predicates. The work on mechanically verifying TLA formulas is preliminary. So far, only simple examples have been completed. The verification described above was done with an early version of the preprocessor, whose implementation required about two man-months of effort. The goal of the project is to assess the feasibility of implementing a verification system that will be useful for real problems. Recent work has concentrated on developing a convenient user interface for managing proofs, which we feel is a prerequisite for a practical system.

Leslie Lamport

9.2 TLA versus Programming Languages Let us compare Figure 6, the description of Program 2 in a conventional programming language, with Figure 7, its representation as a TLA formula. At first glance, the program looks simpler than the TLA formula. However, the program seems simple only because you are already familiar with its notation. To understand what the program means, you need to understand the meaning of the var declarations, the cobegin, loop, ";", and ":=" constructs, and the P and V operations. In contrast, everything needed to understand the TLA formula appears in Figure 4. It is easy to make something seem simple by omitting the complicated definitions needed to understand it. One reason for the conventional program's apparent simplicity is that it does not specify liveness properties. Nothing in Figure 6 told us that the fairness requirement for Ψ should be strong fairness (SFw (M1 ) ∧ SFw (M2 )) rather than weak fairness (WFw (M)). To allow either fairness requirement, a programming language should provide different flavors of cobegin and semaphore operations. If the language provides only one kind of fairness, specifying a different fairness requirement needs a complicated encoding with additional variables—if it is even possible. The TLA formula can be made shorter and easier to read by introducing some simple definitions. The representation of program control can be encapsulated by defining Go(i, d, e) to mean that control in process i goes from d to e (Note 18): Go(i, d, e) = (pc i = d) ∧ (pc i = e) ∧ (pc 3−i = pc 3−i ) ∆

(54)

The semaphore operations can be expressed more compactly by defining P (sem) to equal (0 < sem) ∧ (sem  = sem − 1) and V (sem) to equal sem  = sem + 1. The definition of Ψ then appears much simpler; for example α1 is defined by α1 = P (sem) ∧ Go(1, "a", "b") ∧ Unchanged x, y ∆

The ability to use definitions to simplify formulas makes TLA practical for large specifications. There are just two basic reasons why a TLA formula is longer than the corresponding conventional program: (i) what remains unchanged is implicit in a program statement, but must be stated explicitly in an action definition; and (ii) how the control state changes is implicit in the program, but is described explicitly in the formula. We now discuss these two sources of length. The explicit Unchanged clauses add only about 10% to the length of the definition of Ψ in Figure 7. In larger examples they add less; about 1% of one 700-line TLA specification consists of Unchanged clauses. Still, why pay that price? An obvious way of simplifying the formulas is to let the omission of a variable from an action mean that the variable is left unchanged. Thus, x = x + 1 would be equivalent to (x = x+1)∧(y  = y). However, this "simplification" would in fact make TLA much more complicated. For example, it would mean that the obviously true formula y  = y  is not equivalent to true, since the formulas (x = x + 1) ∧ (y  = y  ) and x = x+1 would not be equivalent—the first would allow y to change and the second would not. Like ordinary mathematics, TLA is simple because a formula constrains only the variables that it explicitly mentions. This is what makes x = x + 1 so much simpler than x := x + 1. Writing Unchanged clauses is a small price to pay for the simplicity of ordinary mathematics.

The Temporal Logic of Actions

The control structures of ordinary programming languages provide a convenient method of specifying that operations are to be performed in a particular order. Specifying this in TLA requires the use of explicit control variables, which are a source of complexity. However, remember that we are interested in reasoning about abstract descriptions of algorithms, not C code. An abstract algorithm usually has few separate actions, so its control structure is simple; if control variables are needed, they usually add little complexity. In abstract algorithms, it is just as common to specify that actions can occur in any order as it is to specify that they occur in some particular order. Conventional languages make it awkward to allow operations to occur in any order. Dijkstra's guarded commands provide a simple mechanism for allowing nondeterminism, but they lack a convenient way to specify the grain of atomicity in the evaluation of guards. We urge the reader to code the cache example of Figure 10 in his favorite programming language. The precise liveness condition will probably be very difficult or even impossible to express within the language. Even ignoring the liveness condition, we expect that the TLA formula will be simpler than the program. Any language will be better than TLA at representing a program written especially for that language. Furthermore, a familiar notation, no matter how cumbersome, invariably seems simpler than an unfamiliar one. Our experience suggests that after one gets used to its notation, the TLA description of a "randomly chosen" algorithm is likely to seem simpler than its representation in a conventional programming language, though it may be longer. (If brevity were synonymous with simplicity, APL would be easier to read than Pascal.) 9.3 Reduction An algorithm must ultimately be translated into a computer program. One develops a program through a series of refinements, starting from a high-level algorithm and eventually reaching a low-level program. Just as we went from Program 1 to the finer-grained Program 2, and from the simple processor/memory interface to the more complicated cached memory, the entire process from specification to C code could in principle be carried out in TLA. "All" we would need is a precise semantics of C, which would allow the translation of any C program into a TLA formula. In practice, the refinement will be carried out in TLA until it becomes obvious how to hand-translate the TLA formula into a program in a real programming language—one with a compiler that produces satisfactory code. But what does it mean for the translation to be obvious? From the point of view of concurrency, the translation from the TLA formula to the program is obvious when any step of the next-state relation corresponds to an atomic operation of the program. In this sense, the translation from an action (sem  = sem + 1) ∧ Unchanged . . . to an atomic V (sem) program statement is obvious. Real programming languages usually guarantee only an extremely fine grain of atomicity. When executing the statement x := x + 1, the read and write of x might each consist of several atomic operations. It would be impractical to describe such a fine-grained program with a TLA formula. Instead, one refines the TLA formula to the point where each step of the next-state relation either corresponds to an atomic program operation like V (sem), or else can be implemented with any grain of atomicity—for example, because it occurs inside an appropriate critical section.

Leslie Lamport

When can an atomic operation be implemented with any grain of atomicity? To answer this, we must first ask: when does a fine-grained program implement a coarser-grained one? There have been a number of partial answers to this question. Some lie in folk theorems—for example, that if shared variables are accessed only in critical sections, then an entire critical section is equivalent to a single atomic operation. Other answers lie in precise results stating that certain classes of properties are satisfied by a fine-grained program if they are satisfied by a coarser-grained version [Lipton 1975]. The question of when a fine-grained program implements a coarser-grained one is answered in TLA by a "reduction" theorem. This theorem seems to include all prior answers as special cases—both the folk theorems and the precise results. The precise statement of the theorem is somewhat complicated, and will be given elsewhere. Here, we give only a rough description of what it says. The theorem's conclusion is approximately Φ ⇒ ∃ w1 , . . . , wn : Φred (w1 /v1 , . . . , wn /vn ) ∧ ✷R

(55)

where Φ is the simple TLA formula (with no hidden variables) describing the original program. Φred is the coarser-grained "reduced" version of the program. v1 , . . . , vn are all the variables that occur in Φ and Φred . R is a predicate containing the variables wi and vi . Think of the vi as "real" variables and the wi as "pretend" variables. Formula (55) asserts that there exist pretend variables such that the original program operating on the real variables implements the reduced program operating on the pretend variables, and the relation R always holds between the real and the pretend variables. In applying the reduction theorem to critical sections, the reduced formula Φred is obtained from the original formula Φ by changing the next-state relation to turn an entire execution of a critical section into a single step. The relation R asserts that the real and the pretend variables are equal when no process is in its critical section. In practice, one reasons about the reduced formula Φred and checks that (55) implies the correctness of the fine-grained formula Φ. For example, in the criticalsection application, if a property does not depend on the values assumed by variables while processes are in their critical sections, then Φ satisfies the property if Φred does. One must then verify that the formula Φ representing the actual program satisfies the hypotheses of the Reduction Theorem, without actually writing Φ. For complicated languages like C, which lack a reasonable formal semantics, this verification must be informal. Whether formal verification is practical, with either a new language or a useful subset of an existing one, is a topic for research. 9.4 What is TLA Good For? TLA, like any useful formal system, has a limited domain of applicability. A formalism that encompasses everything is good for nothing. We believe that TLA is useful for specifying and verifying safety and liveness properties of discrete systems.

The Temporal Logic of Actions

Intuitively, a safety property asserts that something bad does not happen, and a liveness property asserts that something good does eventually happen. We feel that the most significant limitation of TLA is that TLA properties are true or false for an individual behavior. Thus, one cannot express statistical properties of sets of behaviors—for example, that the program has probability greater than .99 of terminating. The only way we know of verifying such properties is to construct a formal model of the system, use TLA to verify that the system correctly implements the model, and then apply other techniques such as Markov analysis to verify that the model has the desired property. The limited expressiveness of TLA is not always a disadvantage. As we have seen, TLA allows fine-grained implementations of coarser-grained specifications because it can express only properties that are invariant under stuttering. A formalism that distinguished between doing nothing and taking a step that produces no change would seem to have a tenuous relation to reality. Another class of properties whose inexpressibility in TLA causes us no concern are possibility properties. We have never found it useful to be able to assert that it is possible for a system to produce the right answer. We want to assert that, under certain assumptions, the system must produce the right answer. TLA can be used to reason about a discrete system even if its behavior depends upon continuous physical values. A particularly important physical value is time. Best- and worst-case time bounds on algorithms can be expressed as safety properties and proved with TLA. For example, the assertion that an algorithm always terminates within 15 seconds is a safety property, where time having advanced 15 seconds without the algorithm having terminated is the "something bad" that does not happen. A real-time algorithm can be specified by conjoining timing constraints to the TLA specification of the untimed version of the algorithm. A description of how TLA is used to reason about real time appears in [Abadi and Lamport 1992]. The use of TLA for hybrid systems is described in [Lamport 1993]. 9.5 What We Have Omitted 9.5.1 Program Derivation. Derivation of a program by a rigorous procedure that guarantees its correctness is preferable to post hoc verification. Concurrent algorithms are derived by refining higher-level, coarser-grained algorithms to lower-level, finer-grained ones. Refinement is the same as implementation—Ψ refines Π means that Ψ implements Π. Any method for proving that one program implements another can be used as the basis for program derivation. Formalisms based on a programming language can usually prove only that a program satisfies certain properties, not that one program implements another. In such formalisms, refinement is either done informally or with special-purpose rules. In TLA, refinement is implication, and we have shown how one can prove that a finer-grained algorithm refines a coarser-grained one. Moreover, some traditional refinement steps can be performed in TLA by applying standard mathematical laws to rewrite formulas. We believe that TLA should be at least as good as any other formalism for deriving concurrent algorithms. Unfortunately, we know of no concurrent algorithm used in a real system that was systematically derived, not simply justified by a post hoc derivation. The derivation of concurrent algorithms is still in the realm of research.

Leslie Lamport

9.5.2 "True" Concurrency. TLA is based on an interleaving model of concurrency, in which we assume that an execution of the system consists of a sequence of atomic events. It seems paradoxical to represent concurrent systems with a formalism in which events are never concurrent. We will not attempt to justify the philosophical correctness of interleaving models for reasoning about concurrent algorithms. Instead, we have tried to demonstrate the best reason we know for using TLA: it is a practical formalism for specifying and verifying safety and liveness properties. 9.5.3 Open Systems. We have discussed only closed systems. A closed system is one that is completely self-contained—in contrast to an open system, which interacts with its environment. Any real system is open; it does not eternally contemplate its navel, oblivious to the outside world. But for most purposes, one can model the actual system together with its environment as a single closed system— as we did for the memory example of Section 8. Such an approach is generally adequate for reasoning about algorithms. However, some problems can be studied only in the context of open systems. For example, composing component systems to form one large system makes sense only for components that are open systems. TLA can be used to describe and reason about open as well as closed systems. But closed systems are simpler, and they provide a necessary foundation for the study of open systems. Here, we have developed TLA and applied it to closed systems. Open systems are discussed elsewhere [Abadi and Lamport 1993]. 9.5.4 System Specifications. Most readers would expect a two-page Pascal program to be simple and a two-page mathematical formula to be too complicated to understand. Yet, since the semantics of TLA is simpler than the semantics of Pascal, a TLA formula should be simpler than a Pascal program of the same length. The main reason mathematical formulas seem to get very complicated when they get large is that mathematicians have not developed notations for structuring large formulas. We have introduced some simple conventions that make the TLA formulas describing abstract algorithms as easy to read as the corresponding programs. TLA can be used not just to describe abstract algorithms, but also to specify complex systems. System specifications can be dozens or even hundreds of pages long. Managing the complexity of large specifications requires additional notation for modular structuring. We have added such notation to TLA to form a language called TLA+ , a purely syntactic extension to TLA with nothing new semantically. TLA+ will be described elsewhere. 10. CONCLUSIONS 10.1 Historical Note TLA is in the tradition of assertional methods for reasoning about programs. These methods go back to Floyd [1967], who first proved partial correctness and termination of sequential programs. Hoare [1969] recast partial correctness reasoning into a logical framework. The first practical assertional method for reasoning about concurrent programs was proposed by Ashcroft [1975]. Ashcroft's work was followed by a number of variations on the same theme [Flon and Suzuki 1978; Keller 1976; Lamport 1977]; but the one that became popular is the Owicki/Gries method, developed by Susan Owicki in her thesis [Owicki 1975], which was supervised by David

The Temporal Logic of Actions

Gries. All these methods, though clothed in different notations, proved safety properties by the use of an invariant; they would be described in TLA as applications of Rule INV1. Temporal logic was first used to reason about concurrency by Pnueli [1977]. It provided the first practical approach to proving more general liveness properties than simple termination. Pnueli introduced the simple temporal logic described in Section 3, with predicates as the only elementary formulas. Pnueli's logic was not expressive enough to describe all desired properties. It was followed by a plethora of proposals for more expressive logics, all obtained by introducing more powerful temporal operators. Pnueli [1979] was the first to describe a program by a temporal logic formula. He, and almost everyone else who followed him, represented programs by formulas that are not invariant under stuttering, so a finer-grained program could not implement a coarser-grained one. The observation that invariance under stuttering permits refinement first appeared in [Lamport 1983b]. The current use of primed and unprimed variables (or their equivalent) for describing "before" and "after" states of a program probably goes back to the early 1970s; we do not know where it first appeared. The idea of actually specifying a program operation by a relation between primed and unprimed variables appears to have been introduced independently by us [Lamport 1983a], Hehner [1984], and Shankar and Lam [1984]. These approaches all used the convention that variables not mentioned are not changed, so they had the inherent complexity epitomized by the observation that y  = y  is not equivalent to true. 10.2 Comparison with Related Formalisms The correctness of an algorithm does not depend on the formalism in which the algorithm and its properities are expressed. A proof of correctness should be essentially the same regardless of the formalism in which it is expressed. A sufficiently informal proof can usually be expressed quite easily in any formalism. Formalisms tend to differ in the ease with which the proof can be formalized. They also differ in how practical they are for writing large specifications. (Formal verification of large specifications is a difficult and rarely attempted task; it seems premature to draw any conclusions about its practicality.) 10.2.1 Methods Based on a Program Text. Extensions of the standard Floyd/ Hoare method to concurrent programs prove only invariance properties and termination [Apt and Olderog 1990]. Methods for proving more general temporal properties have been developed for Unity [Chandy and Misra 1988] and other toy languages [Manna and Pnueli 1991]. It is straightforward to translate a proof in any of these methods into a TLA proof. However, none of the common approaches based on proving properties of a program text can express the concept of one program implementing another. Toy programming languages are not very good for representing real algorithms, even if the toy language resembles the language in which the algorithm is implemented. The most convenient abstraction of a real program may require language features, such as atomic operations on complex data structures, that are seldom provided by toy languages. Moreover, one must reason about the complete system, including components such as file servers and communication lines that are not part of the program itself. A language designed for expressing programs may be ill suited

Leslie Lamport

for describing other components of the system. In particular, it can be awkward, or even impossible, to express the complex liveness properties needed to describe these components with the simple fairness assumptions built into a programming language. Programming languages also lack the abstraction mechanisms needed to manage the complexity of large specifications. Consider how we encapsulated the lower-level details of Program 2 by defining Go, P , and V in Section 9.2. Such definitions are a common method of structuring TLA specifications. In a programming language, Go, P , and V would correspond to procedures with side effects that are executed as part of the current atomic operation. Such procedures are missing from all programming languages, both toy and real, that we know of. 10.2.2 Automata-Based Approaches. The shortcomings of conventional programming languages inspired the use of various forms of abstract automata [Lam and Shankar 1984; Lynch and Tuttle 1987]. Liveness properties of the automata are expressed either by fairness conditions on sets of actions or by formulas in a standard temporal logic. A suitable language for describing automata should permit abstractions such as Go. However, automata-based methods lack the ability to manipulate specifications as mathematical formulas. For example, it seems impossible to write a real-time specification by expressing timing constraints as an automaton that is combined with the untimed specification. Specifying liveness properties of automata is problematic. Although more expressive than the built-in fairness assumptions of a programming language, fairness conditions on sets of actions cannot conveniently specify all desired liveness properties. Temporal logic is expressive enough, but we know of no way to check for machine closure if the fairness requirement can be an arbitrary temporal logic formula. As explained in Section 5.3, the lack of machine closure often indicates an error. Automata-based methods can prove that one specification implements another. The proofs should be essentially the same as the corresponding proof in TLA. We do not know of any completely formal proof system for an automata-based method, so we cannot say what problems may arise in formalizing the proofs. There is reason to believe that machine closure of specifications is necessary for completeness [Abadi and Lamport 1991]. The method of Lam and Shankar [1984] is known to be incomplete because it lacks rules for introducing certain kinds of auxiliary variables. 10.2.3 Temporal Logics. Many forms of temporal logic have been proposed for specifying and reasoning about concurrent algorithms. The most popular ones are probably Unity logic [Chandy and Misra 1988] and the logic of Manna and Pnueli [1991]. These logics share with TLA all the advantages that come from representing an algorithm as a formula. TLA differs from other temporal logics because it is based on the principle that temporal logic is a necessary evil that should be avoided as much as possible. Temporal formulas tend to be harder to understand than formulas of ordinary first-order logic, and temporal logic reasoning is more complicated than ordinary mathematical (nonmodal) reasoning. A typical TLA specification contains one ✷, one ∃ , and a few WF and/or SF

The Temporal Logic of Actions

formulas—even if the specification is hundreds of lines long. Most of the specification consists of definitions of predicates and actions. Temporal reasoning is reduced to a bare minimum; it is used almost exclusively for proving liveness properties. Formalizing liveness proofs is temporal logic's forte. TLA avoids the extensive use of temporal operators that characterizes other temporal logic specification methods by relying instead on internal variables—variables that are hidden with the existential quantifier ∃ . With any temporal logic, existential quantification over flexible variables is neeeded to express most nontrivial specifications. For example, it is necessary for specifying an n-element buffer, where n is a parameter (free rigid variable) of the specification (Note 19). Unity logic lacks a hiding operator, so it is inadequate as a formal specification language. Instead, Unity is used informally by adding the necessary internal variables and pretending that they are hidden [Misra 1990]. All logics that include existential quantification over flexible variables in principle have essentially the same expressive power. TLA can express all formulas invariant under stuttering that Manna and Pnueli's logic can. However, their logic can also express formulas that are not invariant under stuttering. Such formulas yield specifications that cannot be refined. Although all TLA formulas are expressible in Manna and Pnueli's logic, there is no simple translation from TLA to their logic because its quantification operator is not invariant under stuttering. Notes Note 1. We find set theory to be the most natural basis for a logic of actions, in which case Val is the collection of all sets. However, a "smaller" collection Val is adequate for many purposes. Note 2. Formally, + is an operator, and x + y is an abbreviation for +(x, y). An expression is either a variable, a constant symbol, or an expression of the form o(e1 , . . . , en ) where o is an operator and the ei are expressions. An operator o has a meaning [[o]], and the meaning of an expression is defined inductively—for example, s[[+(e1 , e2 )]] equals [[+]](s[[e1 ]], s[[e2 ]]). If we base the logic on set theory, all the operators we need, such as +, can be defined in terms of the following four primitive ones: ∧, ¬, ∈, and ε (Hilbert's "choice" operator [Leisenring 1969]). In the discussion, we do not distinguish between + and [[+]], so we write s[[x + y]] = s[[x]] + s[[y]]. Note 3. When proving the validity of an action by ordinary reasoning, x and x must be considered distinct variables. For example, let A be the action (x = y) ⇒ (x = y  ). Naive substitution of equals for equals might lead one to think that A is valid. However, s[[A]]t does not equal true if s is a state such that s[[x]] = s[[y]] and t is a state such that t[[x]] = t[[y]]. Note 4. Just as we do not bother distinguishing the constant symbol Nat from its meaning [[Nat]], the set of naturals, we do not distinguish the rigid variables m and n from their meanings, which are first-order variables of the semantics. Note 5. Formally, we should distinguish actions from temporal formulas and boolean operators on actions from boolean operators on temporal formulas. Letting

Leslie Lamport

θ(A) denote the temporal formula that we now write A, we should rewrite (9) as s0 , s1 , s2 , . . . [[θ(A)]] = s0 [[A]]s1 ∆

Letting ∨ denote disjunction of temporal formulas, we would then notice that the temporal formula we now write A ∨ B can denote either θ(A ∨ B) or θ(A)∨θ(B). However, these two formulas are equivalent, which is why we can get away with writing A instead of θ(A) and using the same symbols for boolean operators on actions and temporal formulas. Note 6. Formula Φ of Figure 2 asserts that every step of Program 1 increments either x or y, but not both. We could allow simultaneous incrementing of x and y by simply redefining M to equal M1 ∨ M2 ∨ M12 , where M12 = (x = x + 1) ∧ (y  = y + 1) ∆

However, there is no reason to complicate Φ in this way. In representing the execution of x := x + 1 by a single step, we are already modeling a complex operation as one event. Nothing would be gained by allowing the additional possibility of representing the executions of two separate statements as a single step. Note 7. To write the state function x, y, we must assume that any pair of values is a value. More generally, we assume that c1 , . . . , cn  is a value, for any values c1 , . . . , cn . Note 8. More precisely, we define ✸Af to be an abbreviation for ¬✷[¬A]f . The calculation shows that ¬✷[¬A]f is equivalent to the RTLA formula ✸Af obtained from the definitions (7) of ✸ and (15) of Af . Note 9. We have told a white lie; M1 is not equivalent to M1 x, y . For example, suppose there is a value ∞ such that ∞ + 1 equals ∞, and let s be a state in which x has the value ∞. Then the pair s, s is an M1 step, but not an M1 x, y step. However, it is true that definitions (14) and (16) are equivalent, because Init Φ ∧ ✷[M]x, y implies that the values of x and y are always natural numbers, and n + 1 = n is true for any natural number n. Note 10. Observe that Enabled M1 x, y is not equivalent to true. For example, M1 x, y is not enabled in a state in which x equals ∞ (see Note 9). However, M1 x, y is enabled in any state in which x is a natural number, so Init Φ ∧ ✷[M]x, y implies ✷Enabled M1 x, y . Hence, (16) and the definition of Φ in Figure 3 are equivalent. Note 11. The hypothesis of STL1 means that F is a propositional tautology or is derivable by the laws of propositional logic from provable formulas. Note 12. It is somewhat surprising that this completeness result holds even though rules STL1–STL6 are not enough to prove all tautologies of simple temporal logic. Note 13. It is not necessary for "abc" ∈ Nat to be false. Formula (30) is true even if "abc" should happen to equal 135. By not assuming that strings and numbers are disjoint sets, we allow implementations in which strings and numbers share a common representation—for example, as strings of bits. We do, however, assume that "abc" does not equal "xyz".

The Temporal Logic of Actions

Note 14. As an example of the problems introduced by types, consider a variable b of type "array of boolean". If e is an expression, then b[e] is a state predicate. What is the meaning of this state predicate when the value of e is not in the index set of b? There are several possible answers, none of which we find pleasant. This problem is not addressed in the books by Manna and Pnueli [1991] and Chandy and Misra [1988], which allow boolean array variables. It is this very problem that led us to make booleans distinct from values, so state predicates are distinct from state functions. Note 15. In the definition of S(m, v), the symbols m and v are parameters. The expression S(adr , val ) denotes the formula obtained by substituting adr for m and val for v in this definition. Note 16. Quantification over rigid variables can be defined in terms of quantification over flexible variables by ∃ c : F = ∃ x : F (x/c) ∧ ✷[false]x ∆

where c is a rigid variable and x is any flexible variable that does not occur in the temporal formula F . Note 17. The fact that TLA is a typeless logic and LP is typed caused no problem; just two basic LP types, corresponding to TLA's values and booleans, were used. The actual LP proofs are somewhat cumbersome because LP is designed to use types instead of sets. However, translating from a typed version of TLA into LP would be quite difficult—unless one used LP's type system for TLA. Adopting LP's type system, with its lack of subtyping and union types, would make it quite awkward to represent many simple algorithms in TLA. Note 18. To be rigorous, we would have to write pc[i] instead of pc i , replacing the two variables pc 1 and pc 2 by a single "array" variable pc. (Formally, pc is a variable whose value is a function with domain {1, 2}.) Note 19. A specification of a buffer should describe the interface operations of adding and removing elements from the buffer, but should not describe the internal state of the buffer. In all the temporal logics we know of, the only way to specify an n-element buffer is to describe how the interface operations interact with the internal state, and then hide the internal state by existential quantification. ACKNOWLEDGMENTS

Martı́n Abadi helped to develop TLA and is the source of many of the ideas presented here, including the use of an untyped logic. Frits Vaandrager helped me clarify many parts of the exposition and made a valiant but unsuccessful attempt to persuade me that types are useful in logic. Fred Schneider, Reino Kurki-Suonio, Urban Engberg, and Peter Grønning caught a number of errors in earlier versions. Bryan Olivier prevented me from augmenting STL1–STL6 with a redundant rule. A study group at Aarhus University that included Douglas Gurr, Carolyn Brown, and Henrik Andersen provided helpful comments. Jeff Line, Peter Hancock, Dominique Mery, and Jack Wiledon discovered some typographical errors. Peter Ladkin requested Note 11. Luca Cardelli and Cynthia Hibbard suggested

Leslie Lamport

ees, whose carefully considered and detailed comments went well beyond the call of duty. References Abadi, M. and Lamport, L. 1993. Conjoining specifications. Res. Rep. 118, Digital Equipment Corp., Systems Research Center, Palo Alto, Calif. Abadi, M. and Lamport, L. 1992. An old-fashioned recipe for real time. Research Report 91, Digital Equipment Corp., Systems Research Center, Palo Alto, Calif.6 Abadi, M. and Lamport, L. 1991. The existence of refinement mappings. Theor. Comp. Sci. 82, 2 (May), 253–284. Alpern, B. and Schneider, F. B. 1985. Defining liveness. Inf. Process. Lett. 21, 4 (Oct.), 181–185. Apt, K. R. 1981. Ten years of Hoare's logic: A survey—part one. ACM Trans. Programm. Lang. Syst. 3, 4 (Oct.), 431–483. Apt, K. R. and Olderog, E.-R. 1990. Verification of Sequential and Concurrent Programs. Springer-Verlag, New York. Ashcroft, E. A. 1975. Proving assertions about parallel programs. J. Comput. Syst. Sci. 10, 110–135. Burch, J. R., Clarke, E. M., McMillan, K. L., Dill, D. L., and Hwang, L. J. 1992. Symbolic model checking: 1020 states and beyond. Inf. Comput. 98, 2 (June), 142–170. Chandy, K. M. and Misra, J. 1988. Parallel Program Design. Addison-Wesley, Reading, Mass. de Bakker, J. W., Huizing, C., de Roever, W. P., and Rozenberg, G. (Eds.) 1992. Real-Time: Theory in Practice, Lecture Notes in Computer Science, vol. 600. Proceedings of a REX Real-Time Workshop, Springer-Verlag, Berlin. Dijkstra, E. W. 1968. The structure of the "THE"-multiprogramming system. Comm. ACM 11, 5 (May), 341–346. Dijkstra, E. W. 1976. A Discipline of Programming. Prentice-Hall, Englewood Cliffs, N. J. Engberg, U., Grønning, P., and Lamport, L. 1992. Mechanical verification of concurrent systems with TLA. In Computer-Aided Verification, Lecture Notes in Computer Science, Proceedings of the 4th International Conference, CAV'92. Springer-Verlag, Berlin. Flon, L. and Suzuki, N. 1978. Consistent and complete proof rules for the total correctness of parallel programs. In Proceedings of 19th Annual Symposium on Foundations of Computer Science. IEEE, New York, 184–192. Floyd, R. W. 1967. Assigning meanings to programs. In Proceedings of the Symposium on Applied Math. Vol. 19, American Mathematical Society, Providence, Rhode Island, 19–32. 6 An earlier version, without proofs, appeared in [de Bakker et al. 1992, pages 1–27].

The Temporal Logic of Actions

Garland, S. J. and Guttag, J. V. 1989. An overview of LP, the Larch Prover. In Proceedings of the 3rd International Conference on Rewriting Techniques and Applications, N. Dershowitz, Ed. Lecture Notes on Computer Science, vol. 355. Springer-Verlag, New York, 137–151. Hehner, E. C. R. 1984. Predicative programming. Comm. ACM 27, 2 (Feb.), 134–151. Hoare, C. 1969. An axiomatic basis for computer programming. Comm. ACM 12, 10 (Oct.), 576–583. Keller, R. M. 1976. Formal verification of parallel programs. Comm. ACM 19, 7 (July), 371–384. Lam, S. S. and Shankar, A. U. 1984. Protocol verification via projections. IEEE Trans. Softw. Eng. SE-10, 4 (July), 325–342. Lamport, L. 1993. Hybrid systems in TLA+ . In Hybrid Systems, R. L. Grossman, A. Nerode, A. P. Ravn, and H. Rischel, Eds. Lecture Notes in Computer Science, vol. 736. Springer-Verlag, Berlin, 77–102. Lamport, L. 1983a. Specifying concurrent program modules. ACM Trans. Program. Lang. Syst 5, 2 (April), 190–222. Lamport, L. 1983b. What good is temporal logic. In Information Processing 83: Proceedings of the IFIP 9th World Congress, R. E. A. Mason, Ed. North-Holland, Amsterdam, 657–668. Lamport, L. 1977. Proving the correctness of multiprocess programs. IEEE Trans. Softw. Eng. SE-3, 2 (Mar.), 125–143. Leisenring, A. C. 1969. Mathematical Logic and Hilbert's ε-Symbol. Gordon and Breach, New York. Lipton, R. J. 1975. Reduction: A method of proving properties of parallel programs. Comm. ACM 18, 12 (Dec.), 717–721. Lynch, N. and Tuttle, M. 1987. Hierarchical correctness proofs for distributed algorithms. In Proceedings of the 6th Symposium on the Principles of Distributed Computing, ACM, New York, 137–151. Manna, Z. and Pnueli, A. 1991. The Temporal Logic of Reactive and Concurrent Systems. Springer-Verlag, New York. Misra, J. 1990. Specifying concurrent objects as communicating processes. Sci. Comput. Program. 14, 2–3, 159–184. Owicki, S. 1975. Axiomatic proof techniques for parallel programs. Ph. D. thesis, Cornell University, Ithaca N. Y. Pnueli, A. 1979. The temporal semantics of concurrent programs. In Semantics of Concurrent Computation, G. Kahn, Ed. Lecture Notes in Computer Science, vol. 70. Springer-Verlag, New York, 1–20. Pnueli, A. 1977. The temporal logic of programs. In Proceedings of the 18th Annual Symposium on the Foundations of Computer Science, IEEE, New York, 46–57. Schroeder, M. D., Birrell, A. D., Burrows, M., Murray, H., Needham, R. M., Rodeheffer, T. L., Satterthwaite, E. H., and Thacker, C. P. 1990. Autonet: A high-speed, self-configuring local area network using point-to-point links. Res. Rep. 59, Digital Equipment Corporation, Systems Research Center, Palo Alto, Calif.

Leslie Lamport

Shankar, A. U. and Lam, S. S. 1984. Time-dependent communication protocols. In Principles of Communication and Networking Protocols, S. S. Lam, Ed. IEEE Computer Society Press, Los Alamitos, Calif., 504– 519. Received November 1992; revised November 1993; accepted December 1993.
