---
title: "Verkavelde berekeningen, hun mogelijkheden en moeilijkheden (EWD705)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD07xx/EWD705.html
published: "1979-02-24T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd07xx/EWD705.PDF
---

# Verkavelde berekeningen, hun mogelijkheden en moeilijkheden

Edsger W.Dijkstra

In zijn nu klassieke structuur bestaat de rekenautomaat in hoofdzaak uit twee gedeelten, respectievelijk “rekenorgaan” en “geheugen” genaamd. Deze structuur is bekend geworden onder de naam “von Neumann machine”, wellicht mede doordat de term “Turing machine” al iets anders betekende. Bei de onderdelen hebben hun heel specifieke functie.

In het rekenorgaan vinden alle bewerkingen —optellen, aftrekken, vermenigvuldigen, vergelijken etc.— plaats. Die bewerkingen worden daarin de èèn na de ander uitgevoerd, en omdat grote berekeningen onvoorstelbaar veel bewerkingen vergen, is het vooral zaak, dat het rekenorgaan snel is. Op elk moment huist er in het rekenorgaan evenwel vrij weinig informatie, soms haast niet meer dan de operanden en het resultaat van de huidige bewerking.

In het geheugen wordt alle informatie opgeslagen die op dat moment niet actief in de berekening is betrokken: het fungeert als ijskast voor tussenresultaten, ter overbrugging van de periode vanaf het moment dat het tussenresultaat gevormd wordt tot aan het moment dat het tussenresultaat voor het laatst in het rekenorgaan nodig is.

Beide onderdelen zijn met elkaar verbonden op een wijze die intensief informatieverkeer in beide richtingen toestaat. Omdat de lichtsnelheid — slechts een voet per nanoseconde— een bovengrens voor de snelheid van dit informatieverkeer is, mag in zeer snelle machines de onderlinge afstand tussen deze twee onderdelen vooral niet te groot zijn.

De klassieke von Neumann machine heeft twee eigenschappen, die voor het onderstaande van belang zijn

1)     elke berekening bestaat uit een welgedefinieerde opeenvolging van bewerkingen

2)     alle informatie in het geheugen is gelijkelijk toegankelijk voor het rekenorgaan.

Zonder de eerste eigenschap is het veel moeilijker om berekeningen te ontwerpen, de afwezigheid van de tweede eigenschap introduceert daarenboven allerlei strategische problemen. Bij verkavelde berekeningen worden we in meer of mindere mate met beide moeilijkheden geconfronteerd.

In plaats van de berekening door een enkele von Neumann machine te laten uitvoeren, kan men ook proberen de berekening te verkavelen over een aantal componenten, ieder voor zich met de karakteristieken van de von Neumann machine, en het geheel aangevuld met de mogelijkheid van informatie-uitwisseling tussen de componenten onderling. Een dergelijke verkaveling ontmoeten we aan beide uiteinden van het spectrum: aan het ene uiteinde hebben we de primair geographisch gespreide installatie —typisch een vrij bescheiden aantal, ieder voor zich vrij machtige componenten, maar verspreid over de hele aardbol of zelfs daarbuiten: voor het internationale geldverkeer of de explotatie van interstellaire artefacten— aan het andere uiteinde hebben we de primair veelvuldig actieve installatie —typisch een zeer groot aantal, ieder voor zich niet zo machtige componenten, zo dicht als mogelijk op elkaar gepakt en door de simultane activiteit van de vele componenten samen een zeer snel apparaat vormend: voor de electronische simulatie van windtunnels en de verwerking van stromen signalen waarop zo mogelijk instantaan gereageerd moet kunnen worden—. In beide gevallen worden we er evenwel mee geconfronteerd dat de informatie-uitwisseling tussen de componenten aan ernstige beperkingen is onderworpen. In het geval van de geographisch gespreide installatie is de informatie-uitwisseling tussen de componenten in elk geval duur, vaak ook onbetrouwbaar en soms langzaam —als in het informatieverkeer congestie optreedt— , kortom: redenen genoeg om te proberen de hoeveelheid informatie die wordt uitgewisseld te beperken. In het geval van de veelvuldig actieve installatie —waarbij we rustig aan duizenden componenten mogen denken— wordt de snelste communicatiemogelijkheid tussen twee componenten nog altijd geboden door een die componenten verbindende metaaldraad; bedradingsen aansluitmoeilijkheden —die duizenden componenten zijn ieder voor zich dan wel heel klein!— eisen dan, dat elke component slechts informatie uitwisselt met een zeer gering aantal andere componenten: het connectiepatroon moet technisch realiseerbaar zijn.

*          *

*

Over de veelvuldig actieve installatie nog het volgende. Ten eerste moet de opgave zo zijn, dat het althans mathematisch mogelijk is er een heleboel rekenwerk tegelijkertijd aan te verrichten. Iteratieve berekeningen —eens het paradepaard van Computing Science!— hebben deze karakteristiek in principe niet: in een iteratief rekenproces wordt de uitkomst van de ene iteratiestap als uitgangspunt voor de volgende iteratiestap gebruikt, en als er duizend iteratiestappen moeten worden uitgevoerd, moeten ze essentieel achter elkaar worden uitgevoerd. Mathematische inventiviteit is niet zelden noodzakelijk om de mogelijkheid tot veelvuldig simultane activiteit te scheppen. Aan de beperkingen van het connectiepatroon kunnen we soms tegemoet komen doordat we nog een extra vrijheid hebben: als tussenresultaten, respectievelijk in componenten A en B gevormd, moeten worden samengevoegd, kan A’s tussenresultaat naar B gezonden worden, maar het had ook omgekeerd gekund, en soms is deze vrijheid voldoende. In mijn, vooralsnog bescheiden ervaring was het niet ongebruikelijk dat het rekenproces veel drastischer moest worden omgegooid.

*          *

*

In de geographisch gespreide systemen is die vrijheid van werkindeling minder —de bank in Manilla staat nu eenmaal in Manilla en de werkverdeling tussen sateliet en grondstation is doorgaans ook vrij duidelijk— en ligt in dat opzicht de taak veel duidelijker. Hier hebben we echter meer problemen met het verloren zijn van de eerste eigenschap van de von Neumann machine. In de veelvuldig actieve installatie is het realistisch om de activiteiten van de componenten —althans in een zekere, kleine, tijdssteek— door ÈÈn centraal gegenereerd “tijdsein” te laten synchroniseren en is een uitspraak als “op dat moment is de berekening tot daar gevorderd” zinvol en in ons redeneren bruikbaar. In geographisch gespreide systemen is de reistijd van een boodschap van A naar B , gemeten in de natuurlijke tijdssteek van een component, niet alleen groot —het licht reist maar een voet per nanoseconde!— maar vaak binnen een logisch niet verwaarloosbaar ruime marge ongedefinieerd. Door de ongedefinieerdheid van het begrip “gelijktijdigheid” in verschillende componenten mogen wij niet meer praten over de toestand van het systeem in zijn geheel op dat en dat moment en zijn duidelijk andere bewijstechnieken vereist dan die, welke in de entourage van de von Neumann machine toereikend waren.

8 maart 1979

Plataanstraat 5

5671 AL NUENEN
prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Bart Vreugdenhil

revised
01-Dec-2011

---

## English translation

### Partitioned computations, their possibilities and difficulties

Edsger W.Dijkstra

In its now classical structure the computing automaton consists in the main of two parts, called respectively “arithmetic unit” and “store”. This structure has become known under the name “von Neumann machine”, perhaps in part because the term “Turing machine” already meant something else. Both parts have their very specific function.

In the arithmetic unit all operations —addition, subtraction, multiplication, comparison etc.— take place. Those operations are carried out in it one after the other, and because large computations require an unimaginable number of operations, it is above all a matter of the arithmetic unit being fast. At each moment, however, rather little information resides in the arithmetic unit, sometimes hardly more than the operands and the result of the current operation.

In the store all information is held that is at that moment not actively involved in the computation: it functions as an icebox for intermediate results, to bridge the period from the moment that the intermediate result is formed up to the moment that the intermediate result is needed for the last time in the arithmetic unit.

Both parts are connected with each other in a manner that permits intensive information traffic in both directions. Because the speed of light —only a foot per nanosecond— is an upper bound for the speed of this information traffic, in very fast machines the mutual distance between these two parts must above all not be too great.

The classical von Neumann machine has two properties that are of importance for what follows

1)     each computation consists of a well-defined succession of operations

2)     all information in the store is equally accessible to the arithmetic unit.

Without the first property it is much more difficult to design computations; the absence of the second property moreover introduces all manner of strategic problems. In partitioned computations we are, to a greater or lesser degree, confronted with both difficulties.

Instead of having the computation carried out by a single von Neumann machine, one can also try to partition the computation across a number of components, each on its own with the characteristics of the von Neumann machine, and the whole supplemented with the possibility of information exchange among the components. We encounter such a partitioning at both ends of the spectrum: at the one end we have the primarily geographically dispersed installation —typically a rather modest number of, each on its own, rather powerful components, but spread over the whole globe or even beyond it: for international money traffic or the exploitation of interstellar artefacts— at the other end we have the primarily abundantly active installation —typically a very large number of, each on its own, not so powerful components, packed together as closely as possible and, through the simultaneous activity of the many components together, forming a very fast apparatus: for the electronic simulation of wind tunnels and the processing of streams of signals to which, if at all possible, one must be able to respond instantaneously—. In both cases we are confronted, however, with the fact that the information exchange among the components is subject to serious limitations. In the case of the geographically dispersed installation the information exchange among the components is in any case expensive, often also unreliable and sometimes slow —when congestion occurs in the information traffic—, in short: reasons enough to try to limit the amount of information that is exchanged. In the case of the abundantly active installation —where we may freely think of thousands of components— the fastest possibility of communication between two components is still offered by a metal wire connecting those components; wiring and connection difficulties —those thousands of components are, each on its own, indeed very small!— then demand that each component exchanges information only with a very small number of other components: the connection pattern must be technically realizable.

*          *

*

Concerning the abundantly active installation, the following further. In the first place the task must be such that it is at least mathematically possible to perform a great deal of computational work on it simultaneously. Iterative computations —once the showpiece of Computing Science!— do not in principle have this characteristic: in an iterative computational process the outcome of the one iteration step is used as the starting point for the next iteration step, and if a thousand iteration steps have to be carried out, they must essentially be carried out one after the other. Mathematical inventiveness is not seldom necessary in order to create the possibility of abundantly simultaneous activity. We can sometimes accommodate the limitations of the connection pattern by virtue of having yet an extra freedom: if intermediate results, formed respectively in components A and B, have to be combined, A’s intermediate result can be sent to B, but it could also have been done the other way around, and sometimes this freedom is sufficient. In my, as yet, modest experience it was not unusual that the computational process had to be rearranged much more drastically.

*          *

*

In the geographically dispersed systems that freedom of work-division is less —the bank in Manila simply stands in Manila and the division of work between satellite and ground station is generally also quite clear— and in that respect the task lies much more plainly. Here, however, we have more problems with the loss of the first property of the von Neumann machine. In the abundantly active installation it is realistic to have the activities of the components —at least within a certain, small, time-grain— synchronized by one centrally generated “time signal”, and a statement such as “at that moment the computation has progressed up to there” is meaningful and usable in our reasoning. In geographically dispersed systems the travel time of a message from A to B, measured in the natural time-grain of a component, is not only large —light travels only a foot per nanosecond!— but often undefined within a logically non-negligible wide margin. Owing to the undefinedness of the notion “simultaneity” in different components we may no longer speak about the state of the system as a whole at such-and-such a moment, and clearly other proof techniques are required than those which were adequate in the surroundings of the von Neumann machine.

8 March 1979

Plataanstraat 5

5671 AL NUENEN
prof.dr.Edsger W.Dijkstra

Burroughs Research Fellow

transcribed by Bart Vreugdenhil

revised
01-Dec-2011
