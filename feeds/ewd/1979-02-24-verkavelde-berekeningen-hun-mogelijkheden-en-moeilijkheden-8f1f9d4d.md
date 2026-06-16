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

1)     elke berekening bestaat uit een welgedefinieerde opeenvolging van bewerkingen

2)     alle informatie in het geheugen is gelijkelijk toegankelijk voor het rekenorgaan.

Zonder de eerste eigenschap is het veel moeilijker om berekeningen te ontwerpen, de afwezigheid van de tweede eigenschap introduceert daarenboven allerlei strategische problemen. Bij verkavelde berekeningen worden we in meer of mindere mate met beide moeilijkheden geconfronteerd.

In plaats van de berekening door een enkele von Neumann machine te laten uitvoeren, kan men ook proberen de berekening te verkavelen over een aantal componenten, ieder voor zich met de karakteristieken van de von Neumann machine, en het geheel aangevuld met de mogelijkheid van informatie-uitwisseling tussen de componenten onderling. Een dergelijke verkaveling ontmoeten we aan beide uiteinden van het spectrum: aan het ene uiteinde hebben we de primair geographisch gespreide installatie —typisch een vrij bescheiden aantal, ieder voor zich vrij machtige componenten, maar verspreid over de hele aardbol of zelfs daarbuiten: voor het internationale geldverkeer of de explotatie van interstellaire artefacten— aan het andere uiteinde hebben we de primair veelvuldig actieve installatie —typisch een zeer groot aantal, ieder voor zich niet zo machtige componenten, zo dicht als mogelijk op elkaar gepakt en door de simultane activiteit van de vele componenten samen een zeer snel apparaat vormend: voor de electronische simulatie van windtunnels en de verwerking van stromen signalen waarop zo mogelijk instantaan gereageerd moet kunnen worden—. In beide gevallen worden we er evenwel mee geconfronteerd dat de informatie-uitwisseling tussen de componenten aan ernstige beperkingen is onderworpen. In het geval van de geographisch gespreide installatie is de informatie-uitwisseling tussen de componenten in elk geval duur, vaak ook onbetrouwbaar en soms langzaam —als in het informatieverkeer congestie optreedt— , kortom: redenen genoeg om te proberen de hoeveelheid informatie die wordt uitgewisseld te beperken. In het geval van de veelvuldig actieve installatie —waarbij we rustig aan duizenden componenten mogen denken— wordt de snelste communicatiemogelijkheid tussen twee componenten nog altijd geboden door een die componenten verbindende metaaldraad; bedradingsen aansluitmoeilijkheden —die duizenden componenten zijn ieder voor zich dan wel heel klein!— eisen dan, dat elke component slechts informatie uitwisselt met een zeer gering aantal andere componenten: het connectiepatroon moet technisch realiseerbaar zijn.

*          *

*

Over de veelvuldig actieve installatie nog het volgende. Ten eerste moet de opgave zo zijn, dat het althans mathematisch mogelijk is er een heleboel rekenwerk tegelijkertijd aan te verrichten. Iteratieve berekeningen —eens het paradepaard van Computing Science!— hebben deze karakteristiek in principe niet: in een iteratief rekenproces wordt de uitkomst van de ene iteratiestap als uitgangspunt voor de volgende iteratiestap gebruikt, en als er duizend iteratiestappen moeten worden uitgevoerd, moeten ze essentieel achter elkaar worden uitgevoerd. Mathematische inventiviteit is niet zelden noodzakelijk om de mogelijkheid tot veelvuldig simultane activiteit te scheppen. Aan de beperkingen van het connectiepatroon kunnen we soms tegemoet komen doordat we nog een extra vrijheid hebben: als tussenresultaten, respectievelijk in componenten A en B gevormd, moeten worden samengevoegd, kan A’s tussenresultaat naar B gezonden worden, maar het had ook omgekeerd gekund, en soms is deze vrijheid voldoende. In mijn, vooralsnog bescheiden ervaring was het niet ongebruikelijk dat het rekenproces veel drastischer moest worden omgegooid.

*          *

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
