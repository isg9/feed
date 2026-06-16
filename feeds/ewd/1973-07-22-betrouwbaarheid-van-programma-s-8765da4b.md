---
title: "Betrouwbaarheid van programma's (EWD384)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD03xx/EWD384.html
published: "1973-07-22T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd03xx/EWD384.PDF
---

# Betrouwbaarheid van programma's

NOTE: This transcription was contributed by Martin P.M. van der Burgt, who has devised a process for producing transcripts automatically. Although its markup is incomplete, we believe it serves a useful purpose by virtue of its searchability and its accessibility to text-reading software. It will be replaced by a fully marked-up version when time permits. —HR

Betrouwbaarheid van programma’s.
Mathematische uitspraken plegen zich door drie eigenschappen te
onderscheiden:

a)     ze zijn algemeen in de zin, dat ze op een groot aantal gevallen van
toepassing zijn; (men formuleert een eigenschap van “alle niet ontaarde
driehoeken”)

b)     ze zijn heel precies; (dit in tegenstelling tot alle uitspraken, die
hun algemeenheid aan vaagheid ontlenen)

c)     er bestaat een bewijstraditie, waardoor de kans op onjuistheid van de
uitspraak drastisch gereduceerd kan worden; (hierbij tekenen we aan, dat er
voor het bereiken van een dergelijk betrouwbaarheidsniveau geen alternatieven
bekend zijn).
Zodra wij rekenautomaten aan het werk zetten met de bedoeling aan de
uitkomsten consequenties te verbinden, dan zijn wij daartoe slechts gemachtigd
voor zover wij ons geloof kunnen rechtvaardigen, dat de geproduceerde uitkomsten
inderdaad het antwoord op het door ons gestelde probleem zijn. Een eerste
vereiste daartoe in, dat wij ons ervan kunnen overtuigen, dat het programma
correct in. Hoe kunnen wij deze overtuiging bereiken?
In de begintijd heeft men dit door “programma testen” geprobeerd te
bereiken: men onderwerpt het programma aan een aantal bekende testgevallen,
proefberekeningen, waarvan het antwoord bekend is: als in deze testgevallen
het juiste resultaat bereikt werd, nam men aan, dat het programma in alle
gevallen het correcte resultaat zou produceren. De bittere ervaring heeft
geleerd, dat deze laatste aanname ongerechtvaardigd is.
Het punt is, dat de klasse mogelijk bedoelde rekenprocessen, dat onder
controle van een niet helemaal triviaal programma zo enorm groot is, dat men
met de testgevallen slechts een absoluut verwaarloosbare fractie metterdaad
proberen kan en hele subklassen van in een of andere zin “critische gevallen”
kan en zal missen. Sterker: de klasse van de mogelijke berekeningen is een
klasse van zo sterk gestructureerde objecten, dat als van de 1000 testgevallen
er 1 is misgegaan, de verwachting dat ook in de toekomst van de 1000
testgevallen er gemiddeld 1 zal misgaan, helemaal ongerechtvaardigd is: de berekeningen
zijn onvoldoende gelijksoortig om hier zinvol statistiek op te kunnen bedrijven.
(Dit is iets, wat allelei mensen maar heel moeilijk kunnen geloven.) Om het
vereiste betrouwbaarheidsniveau te halen, rest ons slechts een ding: nl.
bewijzen, dat het programma correct is.
Om dit te kunnen doen, moet aan drie voorwaarden voldaan zijn:

a)     de vereiste eigenschappen moeten voldoende scherp en hanteerbaar
geformuleerd zijn (anders hoeven we over “correctheid” helemaal niet te praten)

b)     er moet een bewijstraditie zijn (inclusief stellingen, axioma’s en
beproefde redeneerpatronen)

c)     de voor een correctheidsbewijs benodigde hoeveelheid “manipulatie” moet
binnen de grenzen van het mogelijke blijven.
De eerste pogingen om correctheidsbewijzen te leveren, waren niet
bemoedigend. In retrospectie is dat ook heel begrijpelijk.
Om van een programma de correctheid te bewijzen, zal men een formele
definitie van de semantiek van de programmeertaal, waarin het programma is
uitgedrukt, moeten hebben en de eerste pogingen, zulke formele definities te
geven, hebben, hoewel tot ondubbelzinnige definities -en dat was in die tijd
al heel wat!- niet geleid tot axiomastelsels, die zich soepel leenden tot
basis van correctheidsbewijzen. In dit opzicht lijken de tijden inmiddels
veranderd.
Een tweede oorzaak van de aanvankelijk ontmoedigende ervaringen was
gelegen in het feit, dat men zich in zijn optimisme stortte in het leveren
van correctheidsbewijzen van programma’s geschreven in toen gangbare
programmeertalen, die meestal niet vrij waren van anomalieen, die een zekere
barokheid niet ontzegd kon worden en die zeker niet ontworpen waren in een
tijd, dat men zich over bewijsbaarheid van correctheid al veel zorgen maakte.
De grootste doorbraak is echter gekomen door de ontdekking, dat de
hoeveelheid vereiste manipulatie sterk kon afhangen van de structuur van het
programma en dat een van de dingen, die men met structurering van een
programma na kon streven juist vermindering van de bewijslast was. Een onmiddellijk
gevolg van deze ontdekking is geweest, dat men het correctheidsprobleem
constructiever ging benaderen. In plaats van dat men eerst een programma ging
schrijven en dan ging proberen om te bewijzen, dat het goed was, gingen mensen
correctheidsbewijs en programma hand in hand ontwikkelen: op het moment, dat
het correctheidsbewijs vorm kreeg, schreef men een programma, waarop dit
correctheidsbewijs van toepassing was. Een extra “benefit” van deze
benaderingswijze was, dat op dat moment de correctheids overwegingen zich ontpopten tot
een vruchtbaar heuristisch hulpmiddel.
Een van de machtigste hulpmiddelen is de invariantiestelling voor loops
van C.A.R.Hoare:
Indien voor de statement S de voorwaarde P and B een voldoende
preconditie is om te garanderen dat, mits de uitvoering van S eindigt, aan de
post-conditie P dan voldaan zal zijn, dan is voor de statement

while B do S od

de pre-conditie P voldoende om in geval van beeindiging de post-conditie
P and non B te garanderen.
Het belang van deze stelling is daarin gelegen, dat hij zich zeer wel
blijkt te lenen voor de constructie van programma’s. Om een bepaalde relatie
R te bewerkstelligen, kiest men een P en een B, zodat P and non B de
eindrelatie R impliceert. Het programma krijgt dan de algemene vorm

vestig de geldigheid van P;

while B do onder invariantie van P een “stap”

in de richting van “non B” od .

De stelling is in bijzonder machtig, omdat we ons bij de bewijsvoering
er niet over hoeven uit te laten, hoe effectief de stap in de richting “non B”
nou wel precies is: zolang we er ons van kunnen overtuigen dat de toestand
“non B” in een eindig aantal stappen bereikt zal worden, is dit stuk van het
correctheidsbewijs compleet. Doordat de stelling van Hoare zich er niet over
uitspreekt, hoevaak de herhaalbare statement herhaald wordt, hebben we hier
een stuk gereedschap dat ons in staat stelt programma’s met een deterministisch
netto effect te construeren onder gebruikmaking van het semi-deterministische
primitivum “een stap in de goede richting”. Het is dit gebruik van
semi-deterministische primitiva, dat onder de naam “strategische abstractie” bezig is
burgerrecht te verkrijgen.
Bijna alle programma’s, die ik ken, om bij een gegeven puntenwolk in
het platte vlak het convex omhulsel te vinden, zijn via strategische abstractie
bijvoorbeeld afbeeldbaar op hetzelfde abstracte programma van de volgende
vorm, waarin de mogelijke waarden van de variabele “convex omhulsel” het
convex omhulsel van een niet lege deelverzameling van de gegeven punten zijn.

initialiseer het convex omhulsel, zodat het een of meer

punten omvat;

while er bestaat een punt buiten het convex omhulsel do

kies een punt buiten het convex omhulsel en noem dat q;

pas het convex omhulsel aan, zodat ook punt q omvat is

od

In hun uiteindelijke uitwerking kunnen de hierop geente programma’s
nog enorm verschillen, bv. doordat ze zich in de keuze van het punt q altijd
zullen beperken tot een punt dat op het uiteindelijke contour zal blijken
te liggen of zich deze beperking niet opleggen.
Een tweede techniek begint burgerrecht te verkrijgen onder de naam
“representationele abstractie”. Hier scheidt men de feitelijk gekozen
techniek om in het geheugen verschillende waarden van een (abstracte)
variabele te representeren van de voor de bewijsvoering meest conveniente
wijze om deze verschillende waarden te karakteriseren, daarmee het
correctheidsbewijs in dit opzicht factoriserend. Aan de hand van enige simpele
voorbeelden zal een en ander geillustreerd worden.
2 juli 1973                                 prof.dr.Edsger W.Dijkstra

---

## English translation

### Reliability of programs

Reliability of programs.
Mathematical statements tend to be distinguished by three properties:

a)     they are general in the sense that they apply to a large number of cases; (one formulates a property of “all non-degenerate triangles”)

b)     they are very precise; (this in contrast to all statements that derive their generality from vagueness)

c)     there exists a tradition of proof, by means of which the probability of incorrectness of the statement can be drastically reduced; (here we note that, for attaining such a level of reliability, no alternatives are known).
As soon as we set computers to work with the intention of attaching consequences to the outcomes, we are authorized to do so only insofar as we can justify our belief that the produced outcomes are indeed the answer to the problem we have posed. A first requirement to that end is that we be able to convince ourselves that the program is correct. How can we attain this conviction?
In the early days one tried to attain this by means of “program testing”: one subjects the program to a number of known test cases, trial computations whose answer is known: if in these test cases the correct result was attained, one assumed that the program would produce the correct result in all cases. Bitter experience has taught that this latter assumption is unjustified.
The point is that the class of possibly intended computational processes that is under the control of a not entirely trivial program is so enormously large, that with the test cases one can in fact try only an absolutely negligible fraction, and whole subclasses of, in one sense or another, “critical cases” can and will be missed. Even stronger: the class of the possible computations is a class of such strongly structured objects, that if of the 1000 test cases 1 has gone wrong, the expectation that in the future too, of the 1000 test cases on average 1 will go wrong, is entirely unjustified: the computations are insufficiently alike to be able to conduct meaningful statistics on them. (This is something that all sorts of people can only believe with great difficulty.) To attain the required level of reliability, only one thing remains to us: namely, to prove that the program is correct.
In order to be able to do this, three conditions must be satisfied:

a)     the required properties must be formulated sufficiently sharply and manageably (otherwise we need not speak of “correctness” at all)

b)     there must be a tradition of proof (including theorems, axioms and tried-and-tested patterns of reasoning)

c)     the amount of “manipulation” needed for a correctness proof must remain within the bounds of the possible.
The first attempts to deliver correctness proofs were not encouraging. In retrospect that is also quite understandable.
In order to prove the correctness of a program, one will have to have a formal definition of the semantics of the programming language in which the program is expressed, and the first attempts to give such formal definitions, although they led to unambiguous definitions —and in those days that was already quite something!— did not lead to axiom systems that lent themselves smoothly as a basis for correctness proofs. In this respect the times appear to have changed in the meantime.
A second cause of the initially discouraging experiences lay in the fact that, in one’s optimism, one plunged into delivering correctness proofs of programs written in the then-current programming languages, which were usually not free of anomalies, which could not be denied a certain baroqueness, and which had certainly not been designed at a time when one was already much concerned about provability of correctness.
The greatest breakthrough, however, came through the discovery that the amount of required manipulation could depend strongly on the structure of the program, and that one of the things one could strive for through structuring of a program was precisely a reduction of the burden of proof. An immediate consequence of this discovery has been that one began to approach the correctness problem more constructively. Instead of first writing a program and then trying to prove that it was good, people began to develop correctness proof and program hand in hand: at the moment that the correctness proof took shape, one wrote a program to which this correctness proof applied. An additional “benefit” of this approach was that at that moment the correctness considerations turned out to be a fruitful heuristic aid.
One of the most powerful aids is C.A.R. Hoare’s invariance theorem for loops:
If for the statement S the condition P and B is a sufficient precondition to guarantee that, provided the execution of S terminates, the post-condition P will then be satisfied, then for the statement

while B do S od

the pre-condition P is sufficient to guarantee, in the case of termination, the post-condition P and non B.
The importance of this theorem lies in the fact that it turns out to lend itself very well to the construction of programs. In order to bring about a certain relation R, one chooses a P and a B such that P and non B implies the final relation R. The program then takes the general form

establish the validity of P;

while B do under invariance of P a “step”

in the direction of “non B” od .

The theorem is particularly powerful, because in the proof we need not commit ourselves as to how effective the step in the direction of “non B” precisely is: as long as we can convince ourselves that the state “non B” will be reached in a finite number of steps, this part of the correctness proof is complete. Because Hoare’s theorem makes no pronouncement about how often the repeatable statement is repeated, we have here a piece of tooling that enables us to construct programs with a deterministic net effect by making use of the semi-deterministic primitive “a step in the right direction”. It is this use of semi-deterministic primitives that, under the name “strategic abstraction”, is in the process of gaining its civic rights.
Almost all programs that I know for finding the convex hull of a given cloud of points in the plane are, via strategic abstraction, for example mappable onto the same abstract program of the following form, in which the possible values of the variable “convex hull” are the convex hull of a non-empty subset of the given points.

initialize the convex hull so that it comprises one or more

points;

while there exists a point outside the convex hull do

choose a point outside the convex hull and call it q;

adjust the convex hull so that point q too is comprised

od

In their eventual elaboration the programs grafted onto this can still differ enormously, e.g. in that in the choice of the point q they will always restrict themselves to a point that will turn out to lie on the eventual contour, or do not impose this restriction upon themselves.
A second technique is beginning to gain its civic rights under the name “representational abstraction”. Here one separates the technique actually chosen for representing different values of an (abstract) variable in store from the most convenient way, for the purpose of the proof, of characterizing these different values, thereby factorizing the correctness proof in this respect. By means of a few simple examples one thing and another will be illustrated.
2 July 1973                                 prof.dr. Edsger W. Dijkstra
