---
title: "Hoger orde adressering (EWD254a)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD02xx/EWD254a.html
published: "1969-01-10T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd02xx/EWD254a.PDF
---

# Hoger orde adressering

Hoger order adressering.

Als men programmaonderdelen of bibliotheekprocedures onafhankelijk wil vertalen impliceert dit de introductie van locale terminologieen (immers: zonder locale terminologieen waren de vertalingen niet onafhankelijk!).

Wie dergelijke onafhankelijk vertaalde stukken wil samenbreien, verplicht zich tot de introductie van een interlocale terminologie, waarin elk globaal object in het hele breiwerk uniek benoemd is.

Om te vermijden, dat teksten in dit samenbreien extensief worden bij gewerkt, voert men tabelletjes in, die steeds een locale terminologie in de interlocale overvoeren. We bereiken hiermee het volgende

1) de samenbreibeslissingen (inbedbeslissingen) zijn in de tabellen gelocaliseerd, wat voor wijzigen bv. belangrijk is.

2) als eenzelfde text in verschillende breisels voorkomt (denk aan bibliotheekprocedures), dan hoeven we slechts het tabelletje in veelvoud in te voeren.

3) als de diepte van adressering niet in de programmatext maar in de entry van de tabelletjes te vinden is, dan hebben we een flexibel middel ter versnelling (indien nodig).

4) elke locale terminologie kan een gesloten adressering zijn, met enige voorzorgen kunnen we de interlocale terminologie ook (grotendeels) gesloten houden.

Je eerste idee is, om elk vertaald stuk te voorzien van een catalogus, waaruit bij samenbreien het locale tabelletje kan worden samengesteld. (Dit zal “naamtechnisch” wel op contractie neerkomen!) Een programma, dat uit vijf onafhankelijk vertaalde stukken wordt opgebouwd bevat dan zes tabellen: eentje voor elke locale terminologie (dat zijn er al vijf) en eentje voor de interlocale terminologie (waarin we voor elk object uniek bv. kunnen bijhouden, waar het zich bevindt).

We doen er goed aan te beseffen, dat we met de introductie van deze zes tabellen al aan het implementeren geslagen zijn! In het programma zijn het de vijf locale terminologieen, die een duidelijk gescheiden identiteit hebben: wat we vragen is een of ander vertaalmechanisme van locale naar interlocale naam (of zo).

Ik wil voorlopig proberen de vijf locale tabellen en de ene interlocale zo gelijk mogelijk te beschouwen en wel om de volgende redenen:

1) als ons breiwerk uit een onafhankelijk vertaald stuk bestaat —stel dat dat denkbaar is— dan kunnen locale en interlocale terminologie samenvallen.

2) als we in “interlocale terminologie” denken aan de nomenclatuur voor segmenten, dan moeten we niet schrikken als sommige entries van de interlocale tabel doorverwijzen naar een met andere programma’s gedeeld universum: ik denk dan de procedurebibliotheek.

3) ik zie niet in, waarom bv. aan-afwezigheidsadministratie van een segment, dat slechts in één locale terminologie benoemd is, niet in de bijbehorende locale tabel gevoerd zou kunnen worden. Dit zou efficienter kunnen zijn.

Zolang men zich beperkt tot vertaal— en linkingproblemen maakt men het zich echter heel makkelijk. Het punt is nl. dat op elk moment dat een (nieuwe) interlocale terminologie moet worden ingevoerd “elk segment er al is”. Je kan die segmenten op een rijtje zetten en dan nummeren. De te identificeren populatie is bekend op het moment dat de identificatoren moeten worden ingevoerd. Wie zich tot deze constellatie beperkt, dreigt te ontgaan dat elk mechanisme, volgens welke we objecten aan een reeds bestaande (en intern geidentificeerde) populatie kunnen toevoegen of er van kunnen afnemen, achter de schermen een algorithme impliceert voor de toekenning van de nieuwe identificator.

Ter onderscheiding van wat we besproken hebben en wat nu komt noem ik wat we gehad hebben statische (locale en interlocale) adressen en wat nu komt dynamische (interlocale en interlocale) adressen. Ik zal ook wel spreken van “statische context” en “dynamische context”.

Bij de dynamische locale adressen hebben we er voor te zorgen, dat in eenzelfde programma hetzelfde dynamische adres in verschillende incarnaties naar verschillende variabelen kan wijzen. Zulks betekent, dat we per incarnatie een tabelletje voor de locale dynamische terminologie invoeren. Er zijn, dacht ik, een aantal redenen om statische en dynamische locale adressen naar aparte tabellen te laten verwijzen. Er is een heel prozaische reden: als we ze in één tabel willen onderbrengen, dan wil je dacht ik de dynamische adressen wel apart hebben —zeg aan het hoge eind— en dat sorteerproces compliceert je vertaalproces een beetje. Er is een tweede reden en die is practischer: als je de zaak steeds in een tabel wilt onderbrengen dan moet je bij elke procedureaanroep dat constante stuk van de statische context, dat bij assemblage is ontstaan, copieren. En als je zoekt kan je, dacht ik, ook nog wel een formulering vinden, die fundamenteler aandoet, nl. dat beide tabellen, ieder voor zich, bedoeld zijn om het systeem opgewassen te maken tegen een heel ander soort combinatorische variatie, nl. verschillende namen voor hetzelfde, versus dezelfde naam voor iets anders.

De interpretatie van locale adressen vergt nu dus de kennis van twee tabellen, de statische en de dynamische. Vanwege verschillende procedureincarnatie’s moeten we de combinatie van eenzelfde statische context met vele dynamische contexten toestaan. Moet men ook omgekeerd de combinatie van eenzelfde dynamische context met verschillende statische contexten toestaan?

Ik heb het geval onderzocht, dat dit niet was toegestaan, dwz. dat elke dynamische context tijdens zijn leven altijd met een vaste statische context gecombineerd zou worden. Toen ik met die analyse klaar was, vroeg ik me echter af, wat er dan wel was overgebleven van het ideaal van “onafhankelijke vertaling van delen”: een en ander lijkt er dan nl. ernstig op, dat elke procedure dan combinerende de dynamische context nl. met de statisch locale terminologie in zijn geheel vertaald moet worden. En aangezien ik er aan gewend ben desgewenst een programma als een procedure te beschouwen, zou elk programma in zijn geheel vertaald moeten worden. Met de restrictie “slechts een statische context per dynamische” lijkt een stukje kind met het badwater weggegooid.

Ik stel me in het volgende voor, dat we geen aparte pagina’s kennen, voor segmenten enerzijds een maximale lengte accepteren, anderzijds niet opzien tegen kleine segmentjes. Er staat ons een dergelijke geheugenbespeling voor ogen, laadt ons proberen het er mee te doen!

In dat geval ligt het voor de hand om te overwegen naast de statisch interlocale tabel ook een dynamisch interlocale tabel in te voeren. Het alternatief is om slechts één interlocale tabel te hebben: wat via de statisch locale tabellen gerefereerd wordt komt dan in de bodem, terwijl daarop de dynamische tabel stack-fashion (stapelsgewijze is natuurlijk goed Nederlands!) groeien en krimpen kan. Dat betekent wel zowat, dat “linking” voltooid moet zijn voordat executie begint! Ik neem dus aan, dat we een aparte dynamische interlocale tabel hebben voor de unieke identiteit van de dynamische segmenten.

Ik stel me voor dat bij blokbinnenkomst voor de locale scalairen een segment geintroduceerd wordt. De verwerking van de arraydeclaratie kan zijn dat (in dit segment) de parameters worden opgezet die subscriptiewaarden omrekenen tot dynamisch interlocale adressen en dat zoveel segmenten worden geintroduceerd als voor dit array nodig. (NB. Als we de dynamisch interlocale tabel stacksgewijze bedrijven, dan kunnen we voor een groot array opeenvolgende interlocale segmentnummers uitdelen; dan rekenen we dus aan het dynamisch-interlokale adres. Een interlocaal segmentnummer wordt dan echt een nummer!) Wat het systeem betreft zijn segmenten voor locale scalairen en voor locale arrays op practisch dezelfde leest geschoeid en dat is hoopgevend.

Nu het moeilijke artikel van de procedureaanroep. Ik moet me in dit stadium beperken tot een eerste verkenning. We zitten met:

1) het meegeven van parameters, inclusief terugkeerinformatie

2) het springen naar een ander punt en eventueel een herdefinitie van de statische context

3) het creeren van een nieuwe geinitialiseerde dynamische context en het hierop overgaan.

Het meegeven van de parameters komt kennelijk ten laste van de calling sequence. In de daar heersende dynamische context kome een parametersegment voor, waarin de calling sequence de gegevens ad 1 invulle (wellicht uitgezonderd de terugkeerinformatie).

De karakterisering van de procedure bevatte de gegevens ad 2 (voor de sprong) en een karakterisering van de dynamische context waaruit de bodem van de te creeren dynamische context geinitialiseerd dient te worden.

Er kan nu een subroutinesprong uitgevoerd worden met als parameters

a1) de procedurekarakterisering (zie vorige alinea)

a2) de naam van het parametersegment (ik stel me voor, dat dit de interlocale naam al is, maar daarvan ben ik niet helemaal zeker)

De subroutinesprong bergt (bv. op een vaste plaats van het parametersegment) de terugkeerinformatie (in elk geval terugkeerplaats en statische context, dynamische context mag wat later) en de sprong naar de eerste opdracht van de procedure wordt uitgevoerd, terwijl de dynamische context van de calling sequence nog vigeert! Onder controle van deze text kan nu een nieuwe dynamische context gecreeerd worden. (Als voor deze creatie de lengte van de dynamische tabel nodig is en deze is een functie van de blokhoogte en van interne nesting van blokken, dan kunnen we deze gegevens, dacht ik, aan het begin van de procedure bij executie bekend veronderstellen.) Ik neem aan, dat gegevens a1 en a2 nog beschikbaar zijn. Uit a1 komt het gegeven uit welke dynamische context de nieuw geprepareerde gevuld moet worden (de proceduretext kan de blokhoogte suppleren). Vervolgens zal— en dat is eigenlijk de overgang van het parametersegment van actueel parametersegment in de calling sequence naar formeel parametersegment van de procedure— de procedure zelf een element (het volgende neem ik aan) van de dynamische tabel in preparatie initialiseren voor het parametersegment. Omdat gegeven a2 nog bekend verondersteld wordt en de procedure zelf de eigen naam van het formele parametersegment kent, zijn hiervoor alle gegevens ter beschikking. Tenslotte kan de procedure de dynamische context in opbouw uitbreiden met een segment voor de eigen localen en moet de dynamische context in opbouw de vigerende worden.

Dit hele stuk initialisatie en introductie van de nieuwe dynamische context kan de procedure dus zelf commanderen. Ik kan me voorstellen dat de ongeinitialiseerde nieuwe dynamische context meteen al “de vigerende” wordt. Initialisatie en introductie van een nieuw segment hebben dan per definitie betrekking op de vigerende dynamische context en dat lijkt wel prettig. Je zit dan wel een tijdje met “verboden dynamische adressen” (ongedefinieerde).

Opm1. Voor apart vertaalde procedureonderdelen mag de statische context voor “externals” van het onderdeel, die verwijzen naar localen van de hele procedure bv., als entry een dynamisch adres bevatten, dat per definitie verwijst naar de op dat moment vigerende dynamische context! Een dynamisch adres is statisch (bij textopbouw, maar ook bij samenbreien) heel goed manipuleerbaar. Dit geeft een belangrijk stuk vrijheid, bv. voor procedures uit de bibliotheek, die je “own’s” wilt geven, geadresseerd in de bodem van de stapel van het aanroepende programma.

Opm2. Dat het parametersegment bij call van dynamische naam verandert lijkt me uiterst gezond.

Opm3. Dat een dynamische context niet meer (zoals in mijn eerste opzet) vast aan één statische context gekoppeld is lijkt me een verbetering. Dit maakt het nl. mogelijk statische en dynamische context onafhankelijk om te zetten, dwz. nadat de statische context is omgezet kan de proceduretext zelf beginnen met de creatie van een nieuwe eigen dynamische context. In mijn eerste opzet moest dat veel gekunstelder.

Opm4. De terugkeerinformatie stel ik me voor te bevatten

1) dynamische context (wijzer)

2) statische context (wijzer)

3) locaal terugkeeradres (in termen van 2.)

N.V. PHILIPS’ COMPUTER INDUSTRIE

APELDOORN

transcribed by Bart Vreugdenhil

revised
08-Jan-2012

---

## English translation

### Higher order addressing

Higher order addressing.

If one wishes to translate program components or library procedures independently, this implies the introduction of local terminologies (after all: without local terminologies the translations would not be independent!).

Whoever wishes to knit together such independently translated pieces obliges himself to introduce an interlocal terminology, in which every global object in the whole knitwork is uniquely named.

In order to avoid that texts get extensively reworked during this knitting-together, one introduces little tables, which each transfer a local terminology into the interlocal one. By this we achieve the following

1) the knit-together decisions (embedding decisions) are localized in the tables, which is important for modification, e.g.

2) if one and the same text occurs in several knittings (think of library procedures), then we need only introduce the little table in multiple copies.

3) if the depth of addressing is to be found not in the program text but in the entry of the little tables, then we have a flexible means of acceleration (if needed).

4) every local terminology can be a closed addressing; with some precautions we can keep the interlocal terminology (largely) closed as well.

Your first idea is to equip every translated piece with a catalogue, from which, during knitting-together, the local little table can be composed. (This will, “name-technically”, amount to contraction!) A program built up from five independently translated pieces then contains six tables: one for each local terminology (that is already five) and one for the interlocal terminology (in which we can, e.g., keep a unique record of where every object is located).

We do well to realize that with the introduction of these six tables we have already set to implementing! In the program it is the five local terminologies that have a clearly separate identity: what we ask for is some translation mechanism or other from local to interlocal name (or such).

For the time being I want to try to regard the five local tables and the one interlocal one as alike as possible, and indeed for the following reasons:

1) if our knitwork consists of a single independently translated piece —supposing that to be conceivable— then local and interlocal terminology may coincide.

2) if by “interlocal terminology” we think of the nomenclature for segments, then we should not be alarmed if some entries of the interlocal table refer onward to a universe shared with other programs: I am thinking then of the procedure library.

3) I do not see why, e.g., the presence/absence administration of a segment that is named in only one local terminology could not be conducted in the corresponding local table. This might be more efficient.

As long as one restricts oneself to translation and linking problems, however, one makes things very easy for oneself. The point is, namely, that at every moment that a (new) interlocal terminology must be introduced “every segment is already there”. You can line those segments up in a row and then number them. The population to be identified is known at the moment that the identifiers must be introduced. Whoever restricts himself to this constellation risks failing to notice that every mechanism, according to which we can add objects to an already existing (and internally identified) population or take them away from it, implies behind the scenes an algorithm for the assignment of the new identifier.

To distinguish between what we have discussed and what now follows, I call what we have had static (local and interlocal) addresses and what now follows dynamic (interlocal and interlocal) addresses. I shall also speak of “static context” and “dynamic context”.

With the dynamic local addresses we have to ensure that within one and the same program the same dynamic address can point to different variables in different incarnations. This means that per incarnation we introduce a little table for the local dynamic terminology. There are, I thought, a number of reasons to let static and dynamic local addresses refer to separate tables. There is a very prosaic reason: if we want to house them in one table, then you want, I thought, to have the dynamic addresses separate —say at the high end— and that sorting process complicates your translation process a bit. There is a second reason and it is more practical: if you always want to house the matter in one table, then at every procedure call you must copy that constant piece of the static context that arose at assembly. And if you search you can, I thought, also find a formulation that feels more fundamental, namely that both tables, each in its own right, are meant to make the system equal to a quite different kind of combinatorial variation, namely different names for the same thing, versus the same name for something else.

The interpretation of local addresses thus now requires knowledge of two tables, the static and the dynamic. On account of different procedure incarnations we must permit the combination of one and the same static context with many dynamic contexts. Must one, conversely, also permit the combination of one and the same dynamic context with different static contexts?

I have examined the case in which this was not permitted, i.e. in which every dynamic context would always, during its life, be combined with a fixed static context. When I was done with that analysis, however, I asked myself what then remained of the ideal of “independent translation of parts”: the whole thing then namely strongly resembles the situation in which every procedure, combining the dynamic context namely with the static local terminology, must be translated in its entirety. And since I am accustomed, if so desired, to regard a program as a procedure, every program would have to be translated in its entirety. With the restriction “only one static context per dynamic one” it seems a bit of the baby is thrown out with the bathwater.

In what follows I imagine that we know no separate pages, that for segments we on the one hand accept a maximal length, on the other hand do not balk at little segments. Some such play of memory is what we have in mind; let us try to make do with it!

In that case it is obvious to consider introducing, alongside the static interlocal table, also a dynamic interlocal table. The alternative is to have only one interlocal table: what is referenced via the static local tables then comes to lie at the bottom, while on top of that the dynamic table can grow and shrink stack-fashion (stapelsgewijze is of course good Dutch!). That does rather mean that “linking” must be completed before execution begins! I therefore assume that we have a separate dynamic interlocal table for the unique identity of the dynamic segments.

I imagine that on block entry a segment is introduced for the local scalars. The processing of the array declaration may be that (in this segment) the parameters are set up which convert subscript values into dynamic interlocal addresses, and that as many segments are introduced as are needed for this array. (N.B. If we operate the dynamic interlocal table stack-fashion, then for a large array we can hand out consecutive interlocal segment numbers; we then compute upon the dynamic-interlocal address. An interlocal segment number then really becomes a number!) As far as the system is concerned, segments for local scalars and for local arrays are cut to practically the same pattern, and that is hopeful.

Now the difficult article of the procedure call. At this stage I must restrict myself to a first reconnaissance. We are faced with:

1) the passing of parameters, including return information

2) the jumping to another point and possibly a redefinition of the static context

3) the creation of a new initialized dynamic context and the transition onto it.

The passing of the parameters evidently falls to the charge of the calling sequence. In the dynamic context prevailing there let there occur a parameter segment, in which the calling sequence fills in the data per 1 (perhaps with the exception of the return information).

The characterization of the procedure contained the data per 2 (for the jump) and a characterization of the dynamic context from which the bottom of the dynamic context to be created is to be initialized.

A subroutine jump can now be executed with as parameters

a1) the procedure characterization (see previous paragraph)

a2) the name of the parameter segment (I imagine that this is already the interlocal name, but of that I am not entirely sure)

The subroutine jump stores (e.g. at a fixed place of the parameter segment) the return information (at any rate return place and static context; dynamic context may come a little later) and the jump to the first instruction of the procedure is executed, while the dynamic context of the calling sequence still prevails! Under the control of this text a new dynamic context can now be created. (If for this creation the length of the dynamic table is needed, and this is a function of the block height and of the internal nesting of blocks, then we can, I thought, assume these data to be known at the beginning of the procedure at execution.) I assume that the data a1 and a2 are still available. From a1 comes the datum from which dynamic context the newly prepared one is to be filled (the procedure text can supply the block height). Next —and this is really the transition of the parameter segment from actual parameter segment in the calling sequence to formal parameter segment of the procedure— the procedure itself will initialize an element (the next one, I assume) of the dynamic table under preparation for the parameter segment. Because datum a2 is still assumed known and the procedure itself knows the own name of the formal parameter segment, all data for this are at our disposal. Finally the procedure can extend the dynamic context under construction with a segment for its own locals, and the dynamic context under construction must become the prevailing one.

This whole piece of initialization and introduction of the new dynamic context can thus be commanded by the procedure itself. I can imagine that the uninitialized new dynamic context immediately already becomes “the prevailing one”. Initialization and introduction of a new segment then by definition pertain to the prevailing dynamic context, and that does seem pleasant. You are then stuck for a while with “forbidden dynamic addresses” (undefined ones).

Rem.1. For separately translated procedure components, the static context for “externals” of the component, which refer e.g. to locals of the whole procedure, may contain as entry a dynamic address, which by definition refers to the dynamic context prevailing at that moment! A dynamic address is, statically (at text construction, but also at knitting-together), very well manipulable. This gives an important piece of freedom, e.g. for procedures from the library to which you want to give “own’s”, addressed in the bottom of the stack of the calling program.

Rem.2. That the parameter segment changes its dynamic name on call seems to me extremely healthy.

Rem.3. That a dynamic context is no longer (as in my first design) fixedly coupled to one static context seems to me an improvement. For this makes it possible to convert static and dynamic context independently, i.e. after the static context has been converted the procedure text itself can begin with the creation of a new own dynamic context. In my first design that had to be much more contrived.

Rem.4. The return information I imagine to contain

1) dynamic context (pointer)

2) static context (pointer)

3) local return address (in terms of 2.)

N.V. PHILIPS’ COMPUTER INDUSTRIE

APELDOORN

transcribed by Bart Vreugdenhil

revised
08-Jan-2012
