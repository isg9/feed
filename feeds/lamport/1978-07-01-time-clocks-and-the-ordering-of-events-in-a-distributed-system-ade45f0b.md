---
title: "Time, Clocks, and the Ordering of Events in a Distributed System"
url: https://lamport.azurewebsites.net/pubs/time-clocks.pdf
published: "1978-07-01T00:00:00Z"
feed: lamport
guid: https://lamport.azurewebsites.net/pubs/time-clocks.pdf
---

# Time, Clocks, and the Ordering of Events in a Distributed System

Time, Clocks, and the Ordering of Events in a Distributed System Leslie Lamport Massachusetts Computer Associates, Inc.

A distributed system consists of a collection of distinct processes which are spatially separated, and which communicate with one another by exchanging messages. A network of interconnected computers, such as the ARPA net, is a distributed system. A single computer can also be viewed as a distributed system in which the central control unit, the memory units, and the input-output channels are separate processes. A system is distributed if the message transmission delay is not negligible compared to the time between events in a single process. We will concern ourselves primarily with systems of spatially separated computers. However, many of our remarks will apply more generally. In particular, a multiprocessing system on a single computer involves problems similar to those of a distributed system because of the unpredictable order in which certain events can occur.

The concept of one event happening before another in a distributed system is examined, and is shown to define a partial ordering of the events. A distributed algorithm is given for synchronizing a system of logical clocks which can be used to totally order the events. The use of the total ordering is illustrated with a method for solving synchronization problems. The algorithm is then specialized for synchronizing physical clocks, and a bound is derived on how far out of synchrony the clocks can become. Key Words and Phrases: distributed systems, computer networks, clock synchronization, multiprocess systems CR Categories: 4.32, 5.29 Introduction The concept of time is fundamental to our way of thinking. It is derived from the more basic concept of the order in which events occur. We say that something happened at 3:15 if it occurred after our clock read 3:15 and before it read 3:16. The concept of the temporal ordering of events pervades our thinking about systems. For example, in an airline reservation system we specify that a request for a reservation should be granted if it is made before the flight is filled. However, we will see that this concept must be carefully reexamined when considering events in a distributed system. General permission to make fair use in teaching or research of all or part of this material is granted to individual readers and to nonprofit libraries acting for them provided that ACM's copyright notice is given and that reference is made to the publication, to its date of issue, and to the fact that reprinting privileges were granted by permission o f the table, other substantial excerpt, or the entire work requires specific permission as does republication, or systematic or multiple reproduction. This work was supported by the Advanced Research Projects Agency of the Department of Defense and Rome Air Development Center. It was monitored by Rome Air Development Center under contract number F 30602-76-C-0094. Author's address: Computer Science Laboratory, SRI International, 333 Ravenswood Ave., Menlo Park CA 94025.

In a distributed system, it is sometimes impossible to say that one of two events occurred first. The relation "happened before" is therefore only a partial ordering of the events in the system. We have found that problems often arise because people are not fully aware of this fact and its implications. In this paper, we discuss the partial ordering defined by the "happened before" relation, and give a distributed algorithm for extending it to a consistent total ordering of all the events. This algorithm can provide a useful mechanism for implementing a distributed system. We illustrate its use with a simple method for solving synchronization problems. Unexpected, anomalous behavior can occur if the ordering obtained by this algorithm differs from that perceived by the user. This can be avoided by introducing real, physical clocks. We describe a simple method for synchronizing these clocks, and derive an upper bound on how far out of synchrony they can drift.

The Partial Ordering Most people would probably say that an event a happened before an event b if a happened at an earlier time than b. They might justify this definition in terms of physical theories of time. However, if a system is to meet a specification correctly, then that specification must be given in terms of events observable within the system. If the specification is in terms of physical time, then the system must contain real clocks. Even if it does contain real clocks, there is still the problem that such clocks are not perfectly accurate and do not keep precise physical time. We will therefore define the "happened before" relation without using physical clocks. We begin by defining our system more precisely. We assume that the system is composed of a collection of processes. Each process consists of a sequence of events. Depending upon the application, the execution of a subprogram on a computer could be one event, or the execution of a single machine instruction could be one Communications of the ACM

July 1978 Number 7

Fig. 1.

Fig. 2.

a,

CY

,Y

(9

(9

~o

P4'

(9 O

q7

q5

r3

ql

~) U

-2

- - -

P3' ~

~~

q6 ;--# . i ~~ _ ~ ~ - ~ Y _

r3

r1

event. We are assuming that the events of a process form a sequence, where a occurs before b in this sequence if a happens before b. In other words, a single process is defined to be a set of events with an a priori total ordering. This seems to be what is generally meant by a process.~ It would be trivial to extend our definition to allow a process to split into distinct subprocesses, but we will not bother to do so. We assume that sending or receiving a message is an event in a process. We can then define the "happened before" relation, denoted by "---~", as follows. Definition. The relation "---->"on the set of events of a system is the smallest relation satisfying the following three conditions: (1) I f a and b are events in the same process, and a comes before b, then a ~ b. (2) I f a is the sending of a message by one process and b is the receipt o f the same message by another process, then a ~ b. (3) I f a ~ b and b ~ c then a ---* c. Two distinct events a and b are said to be concurrent if a ~ b and b -/-* a. We assume that a ~ a for any event a. (Systems in which an event can happen before itself do not seem to be physically meaningful.) This implies that ~ is an irreflexive partial ordering on the set of all events in the system. It is helpful to view this definition in terms of a "space-time diagram" such as Figure 1. The horizontal direction represents space, and the vertical direction represents t i m e - - l a t e r times being higher than earlier ones. The dots denote events, the vertical lines denote processes, and the wavy lines denote messagesfl It is easy to see that a ~ b means that one can go from a to b in ' The choice of what constitutes an event affects the ordering of events in a process. For example, the receipt of a message might denote the setting of an interrupt bit in a computer, or the execution of a subprogram to handle that interrupt. Since interrupts need not be handled in the order that they occur, this choice will affect the ordering of a process' message-receiving events. 2 Observe that messages may be received out of order. We allow the sending of several messages to be a single event, but for convenience we will assume that the receipt of a single message does not coincide with the sending or receipt of any other message.

(9 O

r2

P2'

Pl ~

c~

r4

q6

P3

cy

the diagram by moving forward in time along process and message lines. For example, we have p, --~ r4 in Figure 1. Another way of viewing the definition is to say that a --) b means that it is possible for event a to causally affect event b. Two events are concurrent if neither can causally affect the other. For example, events pa and q:~ of Figure 1 are concurrent. Even though we have drawn the diagram to imply that q3 occurs at an earlier physical time than 1)3, process P cannot know what process Q did at qa until it receives the message at p , (Before event p4, P could at most know what Q was planning to do at q:~.) This definition will appear quite natural to the reader familiar with the invariant space-time formulation of special relativity, as described for example in [1] or the first chapter of [2]. In relativity, the ordering of events is defined in terms of messages that could be sent. However, we have taken the more pragmatic approach of only considering messages that actually are sent. We should be able to determine if a system performed correctly by knowing only those events which did occur, without knowing which events could have occurred.

Logical Clocks We now introduce clocks into the system. We begin with an abstract point of view in which a clock is just a way of assigning a number to an event, where the n u m b e r is thought of as the time at which the event occurred. More precisely, we define a clock Ci for each process Pi to be a function which assigns a number Ci(a) to any event a in that process. The entire system o f c l b c k s is represented by the function C which assigns to any event b the number C ( b ) , where C ( b ) = C/(b) i f b is an event in process Pj. For now, we make no assumption about the relation of the numbers Ci(a) to physical time, so we can think of the clocks Ci as logical rather than physical clocks. They m a y be implemented by counters with no actual timing mechanism. Communications of the ACM

July 1978 Number 7

Fig. 3. CY

n¢

c~!

~ ~iLql

.r 4

We now consider what it means for such a system of clocks to be correct. We cannot base our definition of correctness on physical time, since that would require introducing clocks which keep physical time. Our definition must be based on the order in which events occur. The strongest reasonable condition is that if an event a occurs before another event b, then a should happen at an earlier time than b. We state this condition more formally as follows.

Clock Condition. For any events a, b: if a---> b then C ( a ) < C(b). Note that we cannot expect the converse condition to hold as well, since that would imply that any two concurrent events must occur at the same time. In Figure 1, p2 and p.~ are both concurrent with q3, so this would mean that they both must occur at the same time as q.~, which would contradict the Clock Condition because p2 -----> /93.

It is easy to see from our definition of the relation "---~" that the Clock Condition is satisfied if the following two conditions hold. C 1. I f a and b are events in process P~, and a comes before b, then Ci(a) < Ci(b). C2. If a is the sending of a message by process Pi and b is the receipt of that message by process Pi, then Ci(a) < Ci(b). Let us consider the clocks in terms of a space-time diagram. We imagine that a process' clock "ticks" through every number, with the ticks occurring between the process' events. For example, if a and b are consecutive events in process Pi with Ci(a) = 4 and Ci(b) = 7, then clock ticks 5, 6, and 7 occur between the two events. We draw a dashed "tick line" through all the likenumbered ticks of the different processes. The spacetime diagram of Figure 1 might then yield the picture in Figure 2. Condition C 1 means that there must be a tick line between any two events on a process line, and

condition C2 means that every message line must cross a tick line. F r o m the pictorial meaning of--->, it is easy to see why these two conditions imply the Clock Condition. We can consider the tick lines to be the time coordinate lines of some Cartesian coordinate system on spacetime. We can redraw Figure 2 to straighten these coordinate lines, thus obtaining Figure 3. Figure 3 is a valid alternate way of representing the same system of events as Figure 2. Without introducing the concept of physical time into the system (which requires introducing physical clocks), there is no way to decide which of these pictures is a better representation. The reader may find it helpful to visualize a twodimensional spatial network of processes, which yields a three-dimensional space-time diagram. Processes and messages are still represented by lines, but tick lines become two-dimensional surfaces. Let us now assume that the processes are algorithms, and the events represent certain actions during their execution. We will show how to introduce clocks into the processes which satisfy the Clock Condition. Process Pi's clock is represented by a register Ci, so that C~(a) is the value contained by C~ during the event a. The value of C~ will change between events, so changing Ci does not itself constitute an event. To guarantee that the system of clocks satisfies the Clock Condition, we will insure that it satisfies conditions C 1 and C2. Condition C 1 is simple; the processes need only obey the following implementation rule: IR1. Each process P~ increments Ci between any two successive events. To meet condition C2, we require that each message m contain a timestamp Tm which equals the time at which the message was sent. U p o n receiving a message timestamped Tin, a process must advance its clock to be later than Tin. More precisely, we have the following rule. IR2. (a) I f event a is the sending of a message m by process P~, then the message m contains a timestamp T m = Ci(a). (b) U p o n receiving a message m, process Pi sets Ci greater than or equal to its present value and greater than Tin. In IR2(b) we consider the event which represents the receipt of the message m to occur after the setting of Ci. (This is just a notational nuisance, and is irrelevant in any actual implementation.) Obviously, IR2 insures that C2 is satisfied. Hence, the simple implementation rules I R l and IR2 imply that the Clock Condition is satisfied, so they guarantee a correct system of logical clocks.

Ordering the Events Totally We can use a system of clocks satisfying the Clock Condition to place a total ordering on the set of all system events. We simply order the events by the times Communications of the ACM

July 1978 Number 7

at which they occur. To break ties, we use any arbitrary total ordering < of the processes. More precisely, we define a relation ~ as follows: if a is an event in process Pi and b is an event in process Pj, then a ~ b if and only if either (i) Ci{a) < Cj(b) or (ii) El(a) Cj(b) and Pi < Py. It is easy to see that this defines a total ordering, and that the Clock Condition implies that if a ----> b then a ~ b. In other words, the relation ~ is a way of completing the "happened before" partial ordering to a total ordering, a The ordering ~ depends upon the system of clocks Cz, and is not unique. Different choices of clocks which satisfy the Clock Condition yield different relations ~ . Given any total ordering relation ~ which extends --->, there is a system of clocks satisfying the Clock Condition which yields that relation. It is only the partial ordering which is uniquely determined by the system of events. Being able to totally order the events can be very useful in implementing a distributed system. In fact, the reason for implementing a correct system of logical clocks is to obtain such a total ordering. We will illustrate the use of this total ordering of events by solving the following version of the mutual exclusion problem. Consider a system composed of a fixed collection of processes which share a single resource. Only one process can use the resource at a time, so the processes must synchronize themselves to avoid conflict. We wish to find an algorithm for granting the resource to a process which satisfies the following three conditions: (I) A process which has been granted the resource must release it before it can be granted to another process. (II) Different requests for the resource must be granted in the order in which they are made. (III) If every process which is granted the resource eventually releases it, then every request is eventually granted. We assume that the resource is initially granted to exactly one process. These are perfectly natural requirements. They precisely specify what it means for a solution to be correct/ Observe how the conditions involve the ordering of events. Condition II says nothing about which of two concurrently issued requests should be granted first. It is important to realize that this is a nontrivial problem. Using a central scheduling process which grants requests in the order they are received will not work, unless additional assumptions are made. To see this, let P0 be the scheduling process. Suppose P1 sends a request to Po and then sends a message to P2. U p o n receiving the latter message, Pe sends a request to Po. It is possible for P2's request to reach P0 before Pl's request does. Condition II is then violated if P2's request is granted first. To solve the problem, we implement a system of ----"

;~The ordering < establishes a priority among the processes. If a "fairer" method is desired, then < can be made a function of the clock value. For example, if Ci(a) = C/b) andj < L then we can let a ~ b ifj < C~(a) mod N --< i, and b ~ a otherwise; where N is the total number of processes. 4 The term "eventually" should be made precise, but that would require too long a diversion from our main topic.

clocks with'rules IR 1 and IR2, and use them to define a total ordering ~ of all events. This provides a total ordering of all request and release operations. With this ordering, finding a solution becomes a straightforward exercise. It just involves making sure that each process learns about all other processes' operations. To simplify the problem, we make some assumptions. They are not essential, but they are introduced to avoid distracting implementation details. We assume first o f all that for any two processes P / a n d Pj, the messages sent from Pi to Pi are received in the same order as they are sent. Moreover, we assume that every message is eventually received. (These assumptions can be avoided by introducing message numbers and message acknowledgment protocols.) We also assume that a process can send messages directly to every other process. Each process maintains its own request queue which is never seen by any other process. We assume that the request queues initially contain the single message To:Po requests resource, where Po is the process initially granted the resource and To is less than the initial value of any clock. The algorithm is then defined by the following five rules. For convenience, the actions defined by each rule are assumed to form a single event. 1. To request the resource, process Pi sends the message TIn:P/requests resource to every other process, and puts that message on its request queue, where T,~ is the timestamp of the message. 2. When process Pj receives the message T,~:P~ requests resource, it places it on its request queue and sends a (timestamped) acknowledgment message to P~.'~ 3. To release the resource, process P~ removes any Tm:Pi requests resource message from its request queue and sends a (timestamped) Pi releases resource message to every other process. 4. When process Pj receives a Pi releases resource message, it removes any Tm:P~ requests resource message from its request queue. 5. Process P/is granted the resource when the following two conditions are satisfied: (i) There is a Tm:Pi requests resource message in its request queue which is ordered before any other request in its queue by the relation ~ . (To define the relation " ~ " for messages, we identify a message with the event of sending it.) (ii) P~ has received a message from every other process timestamped later than Tin.~ Note that conditions (i) and (ii) of rule 5 are tested locally by P~. It is easy to verify that the algorithm defined by these rules satisfies conditions I-III. First of all, observe that condition (ii) of rule 5, together with the assumption that messages are received in order, guarantees that P~ has learned about all requests which preceded its current '~This acknowledgment messageneed not be sent if Pj has already sent a messageto Pi timestamped later than T.... " If P, -< Pi, then Pi need only have receiveda messagetimestamped _>T,,, from P/. Communications of the ACM

July 1978 Number 7

request. Since rules 3 and 4 are the only ones which delete messages from the request queue, it is then easy to see that condition I holds. Condition II follows from the fact that the total ordering ~ extends the partial ordering ---~. Rule 2 guarantees that after Pi requests the resource, condition (ii) of rule 5 will eventually hold. Rules 3 and 4 imply that if each process which is granted the resource eventually releases it, then condition (i) of rule 5 will eventually hold, thus proving condition III. This is a distributed algorithm. Each process independently follows these rules, and there is no central synchronizing process or central storage. This approach can be generalized to implement any desired synchronization for such a distributed multiprocess system. The synchronization is specified in terms of a State Machine, consisting of a set C of possible commands, a set S of possible states, and a function e: C × S--~ S. The relation e(C, S) -- S' means that executing the command C with the machine in state S causes the machine state to change to S'. In our example, the set C consists of all the commands Pi requests resource and P~ releases resource, and the state consists of a queue of waiting request commands, where the request at the head of the queue is the currently granted one. Executing a request command adds the request to the tail of the queue, and executing a release command removes a command from 'he queue. 7 Each process independently simulates the execution of the State Machine, using the commands issued by all the processes. Synchronization is achieved because all processes order the commands according to their timestamps (using the relation ~ ) , so each process uses the same sequence of commands. A process can execute a command timestamped T when it has learned of all commands issued by all other processes with timestamps less than or equal to T. The precise algorithm is straightforward, and we will not bother to describe it. This method allows one t o implement any desired form of multiprocess synchronization in a distributed system. However, the resulting algorithm requires the active participation of all the processes. A process must know all the commands issued by other processes, so that the failure of a single process will make it impossible for any other process to execute State Machine commands, thereby halting the system. The problem of failure is a difficult one, and it is beyond the scope of this paper to discuss it in any detail. We will just observe that the entire concept of failure is only meaningful in the context of physical time. Without physical time, there is no way to distinguish a failed process from one which is just pausing between events. A user can tell that a system has "crashed" only because he has been waiting too long for a response. A method which works despite the failure of individual processes or communication lines is described in [3]. 7 If each process does not strictly alternate request and release commands, then executing a release command could delete zero, one, or more than one request from the queue.

Anomalous Behavior Our resource scheduling algorithm ordered the requests according to the total ordering =*. This permits the following type of "anomalous behavior." Consider a nationwide system of interconnected computers. Suppose a person issues a request A on a computer A, and then telephones a friend in another city to have him issue a request B on a different computer B. It is quite possible for request B to receive a lower timestamp and be ordered before request A. This can happen because the system has no way of knowing that A actually preceded B, since that precedence informatiori is based on messages external to the system. Let us examine the source of the problem more closely. Let O° be the set of all system events. Let us introduce a set of events which contains the events in b° together with all other relevant external events, such as the phone calls in our example. Let ~ denote the "happened before" relation for ~. In our example, we had A B, but A-~ B. It is obvious that no algorithm based entirely upon events in 0 °, and which does not relate those events in any way with the other events i n ~ , can guarantee that request A is ordered before request B. There are two possible ways to avoid such anomalous behavior. The first way is to explicitly introduce into the system the necessary information about the ordering --~. In our example, the person issuing request A could receive the timestamp TA of that request from the system. When issuing request B, his friend could specify that B be given a timestamp later than TA. This gives the user the responsibility for avoiding anomalous behavior. The second approach is to construct a system of clocks which satisfies the following condition.

Strong Clock Condition. For any events a, b in O°: i f a --~ b then C(a} < C(b). This is stronger than the ordinary Clock Condition because ~ is a stronger relation than ---~. It is not in general satisfied by our logical clocks. Let us identify ~ with some set of "real" events in physical space-time, and let ~ be the partial ordering of events defined by special relativity. One of the mysteries of the universe is that it is possible to construct a system of physical clocks which, running quite independently of one another, will satisfy the Strong Clock Condition. We can therefore use physical clocks to eliminate anomalous behavior. We now turn our attention to such clocks.

Physical Clocks Let us introduce a physical time coordinate into our space-time picture, and let Ci(t) denote the reading of the clock Ci at physical time t.8 For mathematical conWe will assume a Newtonian space-time. If the relative motion of the clocks or gravitational effects are not negligible, then CM) must be deduced from the actual clock reading by transforming from proper time to the arbitrarily chosen time coordinate. Communications of the ACM

July 1978 Number 7

venience, we assume that the clocks run continuously rather than in discrete "ticks." (A discrete clock can be thought of as a continuous one in which there is an error of up to ½ "tick" in reading it.) More precisely, we assume that Ci(t) is a continuous, differentiable function of t except for isolated j u m p discontinuities where the clock is reset. Then dCg(t)/dt represents the rate at which the clock is running at time t. In order for the clock Cg to be a true physical clock, it must run at approximately the correct rate. That is, we must have d C i ( t ) / d t -~ 1 for all t. More precisely, we will assume that the following condition is satisfied: P C I . There exists a constant x << 1 such that for all i: [dCg(t)/dt - 1 [ < x. For typical crystal controlled clocks, x _< 10-(~. It is not enough for the clocks individually to run at approximately the correct rate. They must be synchronized so that Cg(t) = C/(t) for all i,j, and t. More precisely, there must be a sufficiently small constant e so that the following condition holds: PC2. For all i, j: [ C i ( t )

Cy(t)[ < •.

If we consider vertical distance in Figure 2 to represent physical time, then PC2 states that the variation in height of a single tick line is less than E. Since two different clocks will never run at exactly the same rate, they will tend to drift further and further apart. We must therefore devise an algorithm to insure that PC2 always holds. First, however, let us examine how small x and • must be to prevent anomalous behavior. We must insure that the system 5e of relevant physical events satisfies the Strong Clock Condition. We assume that our clocks satisfy the ordinary Clock Condition, so we need only require that the Strong Clock Condition holds when a and b are events in 0 ° with a 4-> b. Hence, we need only consider events occurring in different processes. Let # be a number such that if event a occurs at physical time t and event b in another process satisfies a ~ b, then b occurs later than physical time t + bt. In other words,/~ is less than the shortest transmission time for interprocess messages. We can always choose # equal to the shortest distance between processes divided by the speed of light. However, depending upon how messages in ~ are transmitted, # could be significantly larger. To avoid anomalous behavior, we must make sure that for any i, j, and t: Ci(t + #) - CAt) > 0. Combining this with PC I and 2 allows us to relate the required smallness of x and ~ to the value of # as follows. We assume that when a clock is reset, it is always set forward and never back. (Setting it back could cause C I to be violated.) PCI then implies that Cg(t + #) - Cg(t) > (1 - x)#. Using PC2, it is then easy to deduce that Cg(t + #) - C/(t) > 0 if the following inequality holds: E/(I

~) _< ~.

This inequality together with PC 1 and PC2 implies that anomalous behavior is impossible.

We now describe our algorithm for insuring that PC2 holds. Let m be a message which is sent at physical time t and received at time t'. We define l,m ~ - t t - - I to be the total delay of the message m. This delay will, of course, not be known to the process which receives m. However, we assume that the receiving process knows some minim u m delay tzm >_ 0 such that ~£m ~ Pro. We call ~,, = I,m -- #m the unpredictable delay of the message. We now specialize rules I R I and 2 for our physical clocks as follows: IR 1'. For each i, if Pi does not receive a message at physical time t, then C/is differentiable at t and dCg(t)/dt >0. IR2'. (a) If Pg sends a message m at physical time t, then m contains a timestamp T m = C/(t). (b) Upon receiving a message m at time t', process P/ sets C/(t') equal to m a x i m u m (Cj(t' - 0), Tm + /Zm).9 Although the rules are formally specified in terms of the physical time parameter, a process only needs to know its own clock reading and the timestamps of messages it receives. For mathematical convenience, we are assuming that each event occurs at a precise instant of physical time, and different events in the same process occur at different times. These rules are then specializations of rules IR1 and IR2, so our system of clocks satisfies the Clock Condition. The fact that real events have a finite duration causes no difficulty in implementing the algorithm. The only real concern in the implementation is making sure that the discrete clock ticks are frequent enough so C 1 is maintained. We now show that this clock synchronizing algorithm can be used to satisfy condition PC2. We assume that the system of processes is described by a directed graph in which an arc from process Pi to process P/represents a communication line over which messages are sent directly from Pi to P/. We say that a message is sent over this arc every T seconds if for any t, Pi sends at least one message to P / b e t w e e n physical times t and t + -r. The diameter of the directed graph is the smallest n u m b e r d such that for any pair of distinct processes P/, Pk, there is a path from P / t o P~ having at most d arcs. In addition to establishing PC2, the following theorem bounds the length of time it can take the clocks to become synchronized when the system is first started. THEOREM. Assume a strongly connected graph of processes with diameter d which always obeys rules IR 1' and IR2'. Assume that for any message m, #m --< # for some constant g, and that for all t > to: (a) PC 1 holds. (b) There are constants ~"and ~ such that every ~-seconds a message with an unpredictable delay less than ~ is sent over every arc. Then PC2 is satisfied with • = d(2x~- + ~) for all t > to + Td, where the approximations assume # + ~<< z. The proof of this theorem is surprisingly difficult, and is given in the Appendix. There has been a great deal of work done on the problem of synchronizing physical clocks. We refer the reader to [4] for an intro:) C/(t' - 0) = lim C,(t' Communications of the A C M

181). July 1978 Number 7

duction to the subject. T h e methods described in the literature are useful for estimating the message delays ktm and for adjusting the clock frequencies d C i / d t (for clocks which permit such an adjustment). However, the requirement that clocks are never set backwards seems to distinguish our situation from ones previously studied, and we believe this t h e o r e m to be a new result.

ti+l, to <-- t~, and that at time t[ process Pi sends a message to process Pi+l which is received at time ti+l with an

unpredictable delay less than 4. T h e n repeated application o f the inequality (3) yields the following result for t >_ t n + l .

Ct~t(t) --> Cl(tl') + (1 - ~)(t - tl') - n~.

(4)

F r o m PC1, I R I ' and 2' we deduce that C l ( / l ' ) >" C l ( t l ) + (1 -- K)(tl' -- /1).

Conclusion

C o m b i n i n g this with (4) and using (2), we get W e have seen that the concept o f " h a p p e n i n g before" defines an invariant partial ordering of the events in a distributed multiprocess system. W e described an algorithm for extending that partial ordering to a s o m e w h a t arbitrary total ordering, and showed how this total ordering can be used to solve a simple synchronization problem. A future p a p e r will show how this a p p r o a c h can be extended to solve a n y synchronization problem. T h e total ordering defined by the algorithm is somewhat arbitrary. It can produce a n o m a l o u s b e h a v i o r if it disagrees with the ordering perceived by the system's users. This can be prevented by the use of properly synchronized physical clocks. O u r t h e o r e m showed h o w closely the clocks can be synchronized. In a distributed system, it is i m p o r t a n t to realize that the order in which events occur is only a partial ordering. W e believe that this idea is useful in understanding any multiprocess system. It should help one to u n d e r s t a n d the basic p r o b l e m s o f multiprocessing independently of the m e c h a n i s m s used to solve them. Appendix Proof of the Theorem

(1)

[dCz(t)/dtldt

Ci(t') >_ Cit(t ') for all t' >__ t.

(2)

Suppose process P~ at time tl sends a message to process Pz which is received at time t2 with an unpredictable delay _< ~, where to <- ta _< t2. T h e n for all t ___ t2 we have: C~(t) >_ C~(t2) + (1 - x)(t - t2) > Cfftl) +/~m + (1 -- x)(t -- t2) Cl(tl) + (1 - x)(t

tl)

[by (1) and P C I ] [by I R 2 ' (b)]

[(t2 - tO - ~m] + x(t2 -

t,)

>-- Cl(tl) + (1 - x ) ( t - tl) - 4. Hence, with these assumptions, for all t >_ t2 we have: C~(t) _> Cl(tl) + (1 - x)(t - / 1 )

4"

(3)

NOW suppose that for i = 1, ..., n we have t, _< t ~, <

for t > t n + l . F o r a n y two processes P and P', we can find a sequence of processes P -- Po, P~ . . . . . Pn+~ = P', n _< d, with c o m m u n i c a t i o n arcs f r o m each Pi to Pi+~. By hypothesis (b) we can find times ti, t[ with t[ - ti <- T and ti+l - t" <_ v, where v = # + 4. Hence, an inequality o f the form (5) holds with n <_ d w h e n e v e r t >_ t~ + d('r + v). F o r a n y i, j and any t, tl with tl > to and t ___ t~ + d(z + v) we therefore have: Ci(t) _> Cj(ta) + (1 - x)(t - tx) - d~.

(6)

N o w let m be any message t i m e s t a m p e d Tin, a n d suppose it is sent at time t and received at time t'. W e pretend that m has a clock Cm which runs at a constant rate such that C,~(t) = tm and Cm(t') = tm +/~m. T h e n #0, ___t' - t implies that d C m / d t _< 1. Rule I R 2 ' (b) simply sets Cj(t') to m a x i m u m (Cj(t' - 0), Cm(t')). Hence, clocks are reset only by setting t h e m equal to other clocks. F o r a n y time tx >-- to + / ~ / ( 1 - ~), let Cx be the clock having the largest value at time t~. Since all clocks run at a rate less t h a n 1 + x, we have for all i and all t >_ tx: (7)

W e n o w consider the following two cases: (i) Cx is the clock Cq o f process Pq. (ii) Cx is the clock Cm o f a message sent at time ta by process Pq. In case (i), (7) simply becomes El(t) -< Cq(tx) + (1 + x)(t - tx).

(8i)

In case (ii), since Cm(tx) = Cq(tl) and d C m / d t have

for all t' >_ t. Note that

(5)

Ci(t) _< Cx(tx) + (1 + x)(t - tx).

F o r a n y i and t, let us define C~t to be a clock which is set equal to C~ at time t and runs at the same rate as Ci, but is never reset. In other words, C i t ( t ') = Ci(t) +

Cn+a(t) > C~(tl) + (1 - x)(t - t~) - n~

_ 1, we

Cx(tx) <_ Cq(tl) + (tx - tO.

Hence, (7) yields (8ii)

Ci(t) <-~ Cq(/1) + (1 + K)(t -- /1).

Since tx >-- to + ~t/(1 -- X), we get Cq(tx --

~/(1 -- K)) <_ Cq(tx) - ~ ___ Cm(tx) - / z -< Cm(tx) - (tx - t l ) # m / V m =Tm = Cq(tl)

Hence, Cq(tx - / L / ( 1 ~) and thus ll ~ to. Communications of the ACM

[by P C I ] [by choice o f m] [tXm <-- r.t, tx - tl <_ v,,]

[by definition o f Cm] [by IR2'(a)].

- x)) ___ Cq(tl), so tx - tl < - / x / ( l

July 1978 Number 7

Letting tl = tx in case (i)~ we c a n c o m b i n e (8i) a n d (8ii) to d e d u c e that for a n y t, tx with t _> tx _> to + # / (1 - x) there is a process Pq a n d a time tl with tx - # / (1 - i¢) ~ tl "< tx such t h a t for all i: Ci(t) _< Cq(t~) + (1 + x)(t - h).

(9)

C h o o s i n g t a n d tx with t _ tx + d ( r + ~,), we c a n c o m b i n e (6) a n d (9) to c o n c l u d e that there exists a tl a n d a process Pq such that for all i: Cq(t 0 + (1 - x)(t - tl) - d~ -< Ci(t)

(10)

<: Cq(tl) + (1 + tc)(t - t])

Programming Languages

J.J. Homing

Shallow Binding in Lisp 1.5 Henry G. Baker, Jr. Massachusetts Institute of Technology

Letting t = t~ + d ( r + u), we get d(r+p)__.t-tl_<d(r+v)+#/(1-x). C o m b i n i n g this with (10), we get Cq(tl) + (t - tl) - x d ( r + p) - d~ -< Ci(t) _< Cq(tl)

(11)

+ (t - tl) + x[d(r + u) + / z / ( 1 - x)] U s i n g the h y p o t h e s e s that x << 1 a n d / t _< ~, << r, we can rewrite (11) as the following a p p r o x i m a t e inequality. Cq(tl) + (t - tl) - d(tg'r + ~) ~.< Ci(t)

(12)

~' Cq(/1) + (l -- tl) -I- dKr.

Since this h o l d s for all i, we get

[Ci(t) - CAt)[ ~<d(2xr + O, a n d this h o l d s for all t > to + dr. [] N o t e that r e l a t i o n (11) o f the p r o o f yields an exact u p p e r b o u n d for [Ci(t) - Cj(t) l in case the a s s u m p t i o n /~ + ~ << r is invalid. A n e x a m i n a t i o n o f the p r o o f suggests a simple m e t h o d for r a p i d l y initializing the clocks, o r r e s y n c h r o n i z i n g t h e m i f t h e y s h o u l d go out o f s y n c h r o n y for a n y reason. E a c h process sends a message w h i c h is r e l a y e d to every o t h e r process. T h e p r o c e d u r e c a n be i n i t i a t e d b y a n y process, a n d requires less t h a n 2 d ~ + ~) seconds to effect the s y n c h r o n i z a t i o n , a s s u m i n g e a c h o f the messages has a n u n p r e d i c t a b l e d e l a y less t h a n ~.

Shallow binding is a scheme which allows the value of a variable to be accessed in a bounded amount of computation. An elegant model for shallow binding in Lisp 1.5 is presented in which context-switching is an environment tree transformation called rerooting. Rerooting is completely general and reversible, and is optional in the sense that a Lisp 1.5 interpreter will operate correctly whether or not rerooting is invoked on every context change. Since rerooting leaves assoc [ v, a] invariant, for all variables v and all environments a, the programmer can have access to a rerooting primitive, shallow[], which gives him dynamic control over whether accesses are shallow or deep, and which affects only the speed of execution of a program, not its semantics. In addition, multiple processes can be active in the same environment structure, so long as rerooting is an indivisible operation. Finally, the concept of rerooting is shown to combine the concept of shallow binding in Lisp with Dijkstra's display for Algol and hence is a general model for shallow binding. Key Words and Phrases: Lisp 1.5, environment trees, FUNARG'S, shallow binding, deep binding, multiprogramming, Algol display CR Categories: 4.13, 4.22, 4.32

A c k n o w l e d g m e n t . T h e use o f t i m e s t a m p s to o r d e r operations, a n d the concept o f a n o m a l o u s b e h a v i o r are d u e to P a u l J o h n s o n a n d R o b e r t T h o m a s . Received March 1976; revised October 1977

References 1. Schwartz, J.T. Relativity in lllustrations. New York U. Press, New York, 1962. 2. Taylor, E.F., and Wheeler, J.A. Space-Time Physics, W.H. Freeman, San Francisco, 1966. 3. Lamport, L. The implementation of reliable distributed multiprocess systems. To appear in Computer Networks. 4. Ellingson, C, and Kulpinski, R.J. Dissemination of system-time. 1EEE Trans. Comm. Com-23, 5 (May 1973), 605-624.

General permission to make fair use in teaching or research of all or part of this material is granted to individual readers and to nonprofit libraries acting for them provided that ACM's copyright notice is given and that reference is made to the publication, to its date of issue, and to the fact that reprinting privileges were granted by permission of the table, other substantial excerpt, or the entire work requires specific permission as does republication, or systematic or multiple reproduction. This research was supported by the Advanced Research Projects Agency of the Department of Defense and was monitored by the Office of Naval Research under contract number N00014-75-C-0522. Author's present address: Computer Science Department, University of Rochester, Rochester, NY 14627. Communications of the ACM

July 1978 Number 7
