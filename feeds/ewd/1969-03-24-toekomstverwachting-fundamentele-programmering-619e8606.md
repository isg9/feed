---
title: "Toekomstverwachting Fundamentele Programmering (EWD259)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD02xx/EWD259.html
published: "1969-03-24T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd02xx/EWD259.PDF
---

# Toekomstverwachting Fundamentele Programmering

EWD 259

Toekomstverwachting Fundamentele Programmering.

Onderhanden is een
onderzoeksproject naar de mogelijkheden om ons programmerend vermogen
een orde van grootte te verhogen. De motivering voor dit onderzoek is
gelegen in de observatie, dat de huidige State of the Art ten
duidelijkste een gezonde methodologie ontbeert, met alle nadelige
gevolgen van dien: “kleine” programma's worden met vallen
en opstaan gemaakt, “grote” programma's ontaarden maar
al te vaak in debacles. De reden dat dit
onderzoek ter hand heb genomen is omdat ik dacht in dezen mijn
steentjes te kunnen bijdragen, een hoop die op grond van aard,
bezorgdheid, werkwijze, ervaringen in het verleden bereikte
resultaten niet helemaal ongerechtvaardigd lijkt.

Ik ben nu zo' ongeveer een jaar bezig. Het eerste half jaar is gaan
zitten in een analyse van waarom programmeren zo vaak zo onmenselijk
moeilijk wordt (“een catalogus van ons onvermogen”) en
een uitzoeken van wat we nog betrouwbaar kunnen zonder ons
voorstellings- of overzichtsvermogen te zeer te overbelasten (“een
catalogus van onze relevante vermogens”). Dit alles om gevoel
te krijgen tot wat voor programmastructuren we ons in naam van de
concipieerbaarheid zouden moeten proberen te beperken.

In het tweede halfjaar heb ik allerlei programmastructuren
geprobeerd, aanvankelijk nog erg tastend in het donker. (Het grote
probleem bij deze experimenten was natuurlijk om ze zo goedkoop
mogelijk uit te voeren, om “speelmateriaal” van nét
voldoende complexiteit te vinden, zodat er iets van te leren viel).
Duidelijke aanwijzingen ter zake van een nuttige structuur hebben
zich pas aangediend toen ik serieus recht liet wedervaren aan de eis
van “aanpasbaarheid”, dwz. dat elk groot programma
(althans potentieel) in honderden aanverwante versies moet kunnen
bestaan, zonder dat de hiervoor benodigde intellectuele investering
evenredig met dit aantal stijgen mag. Sindsdien zijn mijn ervaringen
uiterst bemoedigend: de eis van bewijsbaarheid, dat het programma
correct is, evenals de eis, dat het programma aanpasbaar zij,
blijken, wanneer ter harte genomen,de taak van de programmeur
aanzienlijk te verlichten, omdat ze hem voor allerlei verwarrende
kronkels behoeden.

De meest significante uitslag van de experimenten tot op heden is
wellicht dat duidelijk aan het licht is gekomen dat programmeertalen,
zoals die op het ogenblik en vogue zijn (inclusief al hun varianten)
als programmeervehikel inadequaat zijn, wanneer we de eis stellen,
dat we het programma zo willen formuleren als we het begrijpen en
concipiëren kunnen. Er kleeft aan hen nog hetzelfde bezwaar dat
ook kleefde aan de machinetalen, nl. dat het programma in wezen
geschreven en begrepen moet worden op eenzelfde semantisch niveau.
Als ik nu een ALGOL-programma bedenk, merk ik, dat ik het op de mij
inmiddels aangeleerde “getrapte” manier concipieer en
opschrijf – met grote winst aan tijd en betrouwbaarheid; maar
wat ik dan heb opgeschreven, moet ik “met de hand” zo
goed en zo kwaad als het gaat in ALGOL vertalen. Aan den lijve heb ik
ervaren, hoe riskant dit proces is; nu weet ik, hoe “onleesbaar”
bv. ALGOL-programma's zijn en begin ik in te zien, waarom.

Wat begonnen is als het zoeken naar een geestelijke discipline “Hoe
programmeren we voor een gegeven machine, resp. in een gegeven
programmeertaal?” groeit naar de ontwikkeling van een nieuw
”programmeergereedschap”. Nu speel ik nog, maar er komt
(hoop ik!) een moment, dat dit gereedschap er om vraagt om tot een
voldoend compleet systeem voltooid te worden en tot een werkend
systeem geimplementeerd te worden. (En met “werkend”
bedoel ik: met een realistische graad van performance: “the
proof of the pudding is the eating”, maar dat werkt niet echt,
als je eens per maand een muizenhapje nemen mag!)

Op de mogelijkheden hiervoor kan ik op tweeenhalve manier aandringen.
(Naarmate het project volgroeit met meer klem en wellicht met drie).

1)
Het project streeft een uiterst practisch doel na en zij realistisch.
Zonder de realisatiemogelijkheden word ik gedoemd tot de rol van
kamergeleerde (resp kamerdromer), een bespiegelende functie, die ik –
dacht ik – nog niet verdien. (Terzijde: dit is een mieters
dubbelzinnige formulering!)

2)
Als in mijn hoofd het gereedschap tot een voldoende graad van
volledigheid ontwikkeld is, dan heb ik het nodig om studenten
metterdaad
te leren programmeren. Studenten leren om zich te behelpen met een
stuk gereedschap waarvan je de gebreken inmiddels kent is erg en
deprimerend genoeg; op het moment, dat je weet, hoe je deze gebreken
ondervangen kunt, wordt zulks de academie onwaardig.

3)
(Dit is voorhands het halve argument.) Als het gereedschap
conceptueel uit de verf is gekomen, dan is deszelfs implementering
nog een aanzienlijk software project, dat wij zelf voor onze rekening
zullen moeten nemen. De vraag is, in hoeverre via geraffineerde
“bootstrapping” het stuk gereedschap in ontwikkeling
zinnig gebruikt kan worden voor zijn eigen opbouw. Het antwoord
hierop weet ik nog niet. Als het echter bevestigend luidt, ligt hier
een ideaal werkterrein voor afstudeerders: beter kunnen ze het dan
niet hebben!

Als de belofte, die dit project lijkt in te houden, vervuld worden,
is het een project voor tenminste twee vaste medewerkers. We moeten
proberen om het zó
te organiseren, dat meer mensen (vaste medewerkers, afstudeerders of
misschien zelfs stageaires) hieraan kunnen meewerken, opdat zij ervan
leren. Uit educatieve overwegingen is dit gewenst, over het
bereikbare aantal is in dit stadium – nl. voordat een solide
stuk ontwerpwerk gedaan is – nog geen verantwoorde uitspraak te
doen. (Mijn persoonlijke neiging is om met kleine groepen te werken
en, eventueel ten koste van de doorlooptijd, de hoeveelheid werk te
minimaliseren. Weet ik enerzijds dat premature parallellisering tot
debacles leidt, anderzijds weet ik ook (van dichtbij) dat het streven
om parallelle ontwikkeling mogelijk te maken, een ontwerp zeer ten
goede komt. Ik zal mijn best doen.)

Als het project op gang komt en
wil blijven, zullen de vaste medewerkers zich hier volledig aan
moeten wijden en tegen de tijd, dat we een machine in onze
werkzaamheden kunnen inschakelen, ongelimiteerd acces tot deze
machine hebben.

transcribed by Martin van der Burgt

revised
2013-06-20

---

## English translation

### Future Prospects of Fundamental Programming

EWD 259

Future Prospects of Fundamental Programming.

Underway is a research project into the possibilities of raising our
programming capacity by an order of magnitude. The motivation for this
research lies in the observation that the present State of the Art most
clearly lacks a sound methodology, with all the detrimental consequences
that this entails: “small” programs are made by trial and error,
“large” programs all too often degenerate into debacles. The reason
that I have taken up this research is that I thought I might contribute
my modest share in this matter, a hope that, on the grounds of
temperament, concern, manner of working, and results achieved in the
past, does not seem altogether unjustified.

I have now been at it for about a year. The first half year went into an
analysis of why programming so often becomes so inhumanly difficult (“a
catalogue of our incapacity”) and into a sorting out of what we can
still do reliably without overburdening our powers of imagination or
oversight too greatly (“a catalogue of our relevant capacities”). All
this in order to get a feeling for what kinds of program structures we
ought to try to confine ourselves to in the name of conceivability.

In the second half year I tried out all manner of program structures, at
first still very much groping in the dark. (The great problem with these
experiments was, of course, to carry them out as cheaply as possible, to
find “play material” of just sufficient complexity that something could
be learned from it.) Clear indications regarding a useful structure first
presented themselves when I seriously did justice to the requirement of
“adaptability”, i.e. that every large program must (at least
potentially) be able to exist in hundreds of related versions, without
the intellectual investment required for this being allowed to rise in
proportion to that number. Since then my experiences have been extremely
encouraging: the requirement of provability, that the program is correct,
as well as the requirement that the program be adaptable, turn out, when
taken to heart, to lighten the task of the programmer considerably,
because they guard him against all manner of confusing convolutions.

The most significant outcome of the experiments to date is perhaps that
it has clearly come to light that programming languages, such as are at
the moment in vogue (including all their variants), are inadequate as a
programming vehicle when we impose the requirement that we want to
formulate the program in the same way as we can understand and conceive
it. The same objection still adheres to them that also adhered to the
machine languages, namely that the program must in essence be written and
understood at one and the same semantic level. When I now devise an ALGOL
program, I notice that I conceive and write it down in the “stepwise”
manner I have meanwhile taught myself — with great gain in time and
reliability; but what I have then written down, I must translate “by
hand” into ALGOL as best I can. I have experienced in my own person how
risky this process is; now I know how “unreadable” ALGOL programs, for
instance, are, and I am beginning to see why.

What began as the search for a mental discipline “How do we program for a
given machine, or in a given programming language?” is growing into the
development of a new ”programming tool”. For now I am still playing, but
there will come (I hope!) a moment at which this tool calls to be
completed into a sufficiently complete system and to be implemented into
a working system. (And by “working” I mean: with a realistic degree of
performance: “the proof of the pudding is the eating”, but that does not
really work if you are allowed to take only a mouse-sized bite once a
month!)

On the possibilities for this I can press in two and a half ways. (As the
project matures, with more emphasis and perhaps with three.)

1)
The project pursues an extremely practical aim and be realistic. Without
the possibilities of realization I am doomed to the role of an armchair
scholar (or armchair dreamer), a contemplative function which — I thought
— I do not yet deserve. (Aside: this is a wonderfully ambiguous
formulation!)

2)
Once the tool has been developed in my head to a sufficient degree of
completeness, then I need it in order actually to teach students
programming. To teach students to make do with a piece of tooling whose
defects you have meanwhile come to know is bad and depressing enough; at
the moment that you know how you can circumvent these defects, doing such
a thing becomes unworthy of the academy.

3)
(This is for the time being the half argument.) Once the tool has, in
conceptual terms, come into its own, then its implementation is still a
considerable software project, which we shall have to take upon ourselves.
The question is to what extent, by means of refined “bootstrapping”, the
piece of tooling under development can be sensibly used for its own
construction. The answer to this I do not yet know. If, however, it turns
out to be affirmative, here lies an ideal field of work for graduating
students: they could not have it any better!

If the promise which this project seems to hold is fulfilled, it is a
project for at least two permanent staff members. We must try to organize
it in such a way that more people (permanent staff members, graduating
students, or perhaps even trainees) can collaborate on it, so that they
learn from it. From educational considerations this is desirable; about
the attainable number, no responsible statement can be made at this stage
— namely before a solid piece of design work has been done. (My personal
inclination is to work with small groups and, if need be at the expense
of throughput time, to minimize the amount of work. While on the one hand
I know that premature parallelization leads to debacles, on the other
hand I also know (from close at hand) that the striving to make parallel
development possible greatly benefits a design. I shall do my best.)

If the project gets going and is to keep going, the permanent staff
members will have to devote themselves to it completely and, by the time
that we can bring a machine into our activities, have unlimited access to
this machine.

transcribed by Martin van der Burgt

revised
2013-06-20
