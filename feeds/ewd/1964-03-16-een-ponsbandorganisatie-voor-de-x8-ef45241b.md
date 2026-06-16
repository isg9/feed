---
title: "Een ponsbandorganisatie voor de X8 (EWD82)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD82.html
published: "1964-03-16T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD82.PDF
---

# Een ponsbandorganisatie voor de X8

EWD082     X-8 nr. 38.

Een ponsbandorganisatie van de X8.

Motto: "Wie het kleine niet eert,

is het grote niet waard."

Inleiding.

In dit rapport zal het hoofdaccent vallen op het lezen van ponsband. Waarom het ponsen minder grote vragen op hoeft te werpen, zal in de loop hiervan duidelijk worden.

De bandleesopdracht van de X8 is volledig code-onafhankelijk, elke configuratie van al of niet gaatjes dringt tot de machine door en in zekere zin "is er geen probleem", nl. in de zin, dat we de verwerking van deze banden kunne programmeren zoals we willen. Omgekeerd geeft dit ons wel de plicht om uit de baaierd van mogelijkheden dan te kiezen, wat we willen. Een groot gedeelte van dit rapport houdt zich met deze keuze (en de motiviering daarvan) bezig.

Het leidende principie is, dat het voor de gebruiker zo gemakkelijk mogelijk gemaakt moet worden. Dit betekent onder andere dat wanneer een gebruiker opeenvolgende getallen van een ponsband wil lezen, dat hij dan een intern passende identificatie van numerieke waarden in zijn proces wil opnemen, zonder dat hij zich bewust hoeft te zijn van de conventies, volgens welke deze getallen op de band staan. Deze situatie is geheel analoog aan zijn instelling tegenover variabelen van type integer of real: voor hem karakteriseren deze waarden, hoe is zijn zorg niet; de interne representatie hoeft niet eens uniek te zijn. Bij een dergelijke wijze van bandlezen dienen allerlei "irrelevantia", voordat de numerieke waarde tot het programma doordringt, dus onder tafel geraakt te zijn.

De complicatie hierbij is echter, dat wij niet a priori decreteren kunne, wat voor de programmeur irrelevant is. Ponsbanden immers leiden een zelfstandig bestaan buiten de machine, zij kunnen komen van automatische meetapparatuur, zij kunnen voor ponsbandgestuurde machinerie gemaakt moeten worden. Waar wij dus op aansturen is een organisatie, die enerzijds het gemak dient, maar die anderzijds desgewenst de programmeur de volledige controle geeft, die hij nodig heeft om ponsband als "general purpose medium" te kunnen gebruiken.

Een volgende complicatie is het gemengde gebruik, dat men maken wil van het physische "einde band". "Einde band" is door de X8 detecteerbaar, de consequenties, die hieraan verbonden dienen te worden, kunnen van geval tot geval echter aanzienlijk verschillen.

Er zijn programma's, waarin einde band een essentieel onderdeel van de inkomende informatie stroom is, bv. een programma, dat een willekeurige band reproduceert. (Het is in dit licht jammer, dat de ponser niet beschikt over een door de X8 bestuurbaar afsnijertje!)

In andere organisaties wil men liever aan einde band geen betekenis toekennen. Als een programma vraagt om een heel lange band, die vb. uit twee stukken is opgebouwd, dan is het natuurlijk prettig, dat de EL 1000 zorgvuldig gemaakte plakjes slikt, maar het zou jammer zijn, als dit de operateur ook inderdaad tot plakken zou verplichten. Als de operateur om een of andere reden prefereert om zo'n band in twee stukken in te leggen, zou dit moeten mogen zonder dat daarbij een extra last plicht tot plakken wil geven, evenmin wil men de operateur de plicht tot knippen -dwz. het verbod tot plakken- opleggen, althans in het merendeel der gevallen, waarin hierdoor geen onduidelijkheden kunnen ontstaan. (Het is duidelijk, dat men achteraan een band, waarop "einde band" een essentieel onderdeel van de informatie is, niet straffeloos een andere band mag aanplakken.)

Het idee van de standaardband en "de open bek".

We gaan er vanuit, dat het merendeel der ALGOL-programma's zo is, dat, zo zij om getallenmateriaal vragen,

- zij niet hoeven te reageren op "einde band"
- dit materiaal van te voren bijgeleverd kan worden.

Het (operationeel) normale geval zij, dat deze band met getallenmateriaal dan achter de programmatext aangeplakt wordt (c.q. meteen erachteraan geponst is). Wij noemen een dergelijke band met te verwerken materiaal "de eigen band" van dit programma.

Hier geeft het begin van de band -nl. de programmatext- aan hoe de rest -nl. het getallenmateriaal- verwerkt moet worden. Wij willen dit idee een stap uitbreiden en spreken af, dat vooraan de text een herkenbare standaardkreet, zeg algol voorkomt. Dit is een metakarakter, dat behelst, dat de eropvolgende ponsingen als de etxt[sic] van een ALGOL-programma geinterpreteerd moeten worden. Deze interpretatie (vertaling, gevolgd door executie) is een standaardverwerking; het metakarakter algol is op de band geintroduceerd, omdat ik wel ruimte zie voor uitbreidingen van het aantal standaardverwerkingen. Al deze standaadverwerkingen zullen via een bijbehorend metakarakter geactiveerd kunnen worden.

Een band, die zichzelf aan een dergelijke standaardverwerking doet onderwerpen noemen we een standaardband. Het doel, waarnaar wij streven is om het overgrote deel der operateurshandelingen te gieten in het keurslijf "verwerking van een standaardband."

Hiertoe voeren wij in het concept van de "bandlezer met de open bek". De bedoeling is, dat de operateur voor de verwerking van een standaardband niet meer hoeft te doen dan zo'n standaardband inleggen in een bandlezer en op de groene knop drukken. De toestand, waarin een bandlezer aldus op standaardwijze op een standaardband reageert noemen we "met open bek".

Het is de bedoeling, dat alle banden, eenmaal ingelegd, in principe op volle snelheid -dwz. ongeacht het tempo van feitelijke verwerking- ingelezen worden. Als de lezer sneller loopt dan de feitelijke verwerking, dan kan de rest op de trommel gedumpt worden. Zodra "einde band" gesignaleerd is, wordt de lezer vrijgegeven en blijft deze achter met de bek open.

N.B. We eisen niet, dat "einde band" gesignaleerd is, voordat de feitelijke verwerking begint. Als het dringen in het geheugen is, dan moet de coordinator kunnen kiezen en "lezerbezetting" boven "geheugenbezetting" kunnen prefereren.

Hieraan zitten al enige haken en ogen, omdat bandlezers niet uitsluitend voor de standaardverwerking van standaardbanden gebruikt zullen worden. (Er bestaat ook non-standardverwerking van wat er overigens bedriegelijk als standaardband uitziet!)

De prijs, die we voor de gemakkelijke verwerking van standaardbanden moeten betalen bestaat uit enige extra hocus-pocus voor het non-standaardlezen. Men moet nl. een veilig sluisje maken voor het geval, dat de operateur te goeder trouw een standaardband inlegt in een lezer, juist op hetzelfde moment, dat de X8 beslist, deze lezer voor een ander doel te gaan gebruiken.

In ruimte aanleg kan een eenbandlezer zich in drie toestanden bevinden:

toestand 0: vrij

toestand 1: voor non-standaardwerk aangevraagd

toestand 2: bezet

In toestand 0 hangt er een tamelijk symbolische startopdracht voor 1 enkele heptade, die alleen maar gegeven is om het inleggen van een nieuwe band en het indrukken van de groene knop tot de X8 te laten doordringen. (Ik neem aan, dat de heptade zelf wellicht genegeerd zal worden.)

In toestand 1 heeft de X8 besloten, dat de bandlezer voor non-standaardwerk gebruikt gaat worden; hiervan is via de verreschrijver melding gedaan. De bandlezer zal niet verder lopen, voordat de operateur op deze melding geantwoord heeft, dat er geen verkeerde band in de bandlezer ligt.

In toestand 2 is de bandlezer definitief ingeschakeld voor een welomlijnde (al of niet standaard) taak.

Vindt nu een ingreep plaats van een bandlezer, die zich in toestand 0 bevindt, dan wordt deze band als standaardband verder gelezen. In doofheid vindt, als reactie op de symbolische heptade, de overgang naar toestand 2 plaats (en er wordt genoteerd, dat we nu standaardbandlezen). De volgende ingrepen vinden per definitie plaats in toestand 2.

Vindt een (eerste) ingreep plaats in toestand 1, dan wordt er tot nader order gen band meer gelezen. De operateur heeft dan nl. een standaardband ingelegd in de periode, dat de X8 de lezer al voor een ander doel besteed heeft, maar de operateur nog niet geantwoord heeft, van deze beslissing kennisgenomen te hebben. De operateur merkt dit o.a. doordat van de band, waarvan hij verwacht, dat hij naarbinnen zal schieten, maar 1 heptade gelezen wordt.

Het operateursantwoord -behelzend, dat er geen ongevraagde band in de lezer ligt, bewerkstelligt de overgang van toestand 1 naar toestand 2, een volledige specificatie van wat er dan wel moet gebeuren en tenslotte het vullen van het startmagazijn van de lezer in kwestie. Ligt de band al in, dan gaat hij lopen; de operateur mag het antwoord ook vast geven, als de lezer leeg is, waarna de band gaat lopen, zodra hij is ingelegd.

Het concept van de bandvariabele.

Als een band "synchroon met het physisch leesproces" verwerkt wordt, zoals bij ARMAC en X1, dan fungeert de ligging van de band in de lezer als geheugenelement. Tegen de tijd, dat we het bandbeeld in het geheugen hebben opgeslagen, moeten we dit "heptadewijzertje", dat het sequentieel aftasten van het bandbeeld bestiert, in het geheugen opnemen. Dit heptadewijzer is een belangrijke constituent van de zg. bandvariabele.

Voorts bevat een bandvariabele het gegeven, of het physisch leesproces al beeindigd is of niet. Zo nee, dan bevat hij een verwijzing naar de physische lezer, de lezeradministratie bevat op zijn beurt een verwijzing naar de bandvariabele. De koppeling "bandvariabele - lezer", die bestaan blijft, totdat de X8 voor deze lezer het signaal "einde band" gevonden heeft, wordt dus wederzijds geboekt. Zodra het signaal "einde band" tot de X8 doordringt, wordt deze koppeling verbroken. De bandvariabele heeft er dan geen weet meer van, via welke lezer deze band is binnengekomen, de lezer zelf gaat over in toestand 0 met de startopdracht voor de enkele heptade in het startmagazijn. (Dankzij het symbolisch afwerken van eventuele startopdrachten in het magazijn, zodra einde band gesignaleerd is, is het mogelijk om te zorgen, dat dit de enige startopdracht in het magazijn is.)

Een bandvariabele maakt altijd deel uit van een programma (uitgezonderd misschien bij de allereerste analyse van het metakarakter voorop een standaardband). Tijdens vertaling is de bandvariabele een grootheid van de vertaler, aan het einde van de vertaling fungeert de dan heersende waarade van de bandvariabele als initialisering van de eigen-bandvariabele van het vertaalde programma. Dit overhevelen van bandvariabele van vertaler naar vertaald programma dient met enige zorg te geschieden: omdat de bandlezer het leesproces nog niet voltooid hoeft te hebben, bergt een bandvariabele dus een mogelijke synchronisatiebeperking in zich. Het is deze synchronisatiebeperking, die ook van vertaler naar vertaald programma overgegeven moet worden.

Gestroomlijnd lezen en de rol van metakarakters.

Wanneer wij meer dan 1 programma simultaan in de machine hebben zitten, dan hebben wij zoals bekend er voor te zorgen, dat het ene programma niet zo fout kan zijn, dat daardoor het andere programma verstoord wordt. Deze garantie impliceert beperkingen ten aanzien van de structuur der individuele programma's. Ik heb me steeds op het standpunt gesteld, dat elke gebruiker, die deze beperkingen niet accepteren kan omdat hij in machinecode programeren wil, de machine dan maar in uniprogrammering bedrijven moet.

Een analoge situatie komen we tegen, wanneer we twee programma's, die verder niets met elkaar te maken hebben, achter elkaar op eenzelfde band willen invoeren. Het verwerkende systeem heeft dan de plicht om vast te stellen, waar de text van het tweede programma begint. De opsteller van het eerste programma mag dan niet de mogelijkheid hebben een zodanige fout te maken, dat het correct vaststellen van de plaats van deze caesuur niet meer een betrouwbare operatie is. Om veilig twee programma's achterelkaar[sic] op dezelfde band in te kunnen voeren, moet het eerste programma zekere spelregels in acht nemen, geheel analoog aan de boven gesignaleerde situatie. Uit de aard der zaak zullen wij ernaar streven, deze restricties zo min mogelijk knallend te maken.

Om het leven niet nodeloos te compliceren, veronderstel ik in het volgende dat een programma slechts op twee wijzen contact met de eigen band kan opnemen:

- lees volgende ponsing
- lees volgend getal.

In geval a) heeft de programmeur toegang tot ieder bit van de ingevoerde band en hij kan hierop naar eigen goeddunken reageren. Een programma, dat de ponsingen als zodanig van de eigen band leest is als het programma in machinecode; het is niet toegestaan om achter de eigen band van dit programma de text van een onafhankelijk volgend programma te plakken. (Wie rot zou willen doen, schrijft dan een ALGOL-programma, waarvan het enige effect is, dat het volgende ALGOL-programma op de hand wordt overgesslagen!)

Een programma, dat zich in zijn contact met de eigen band echter beperkt tot "lees volgend getal" kan door het systeem onder de duim gehouden worden. Ik beschouw hier "lees volgend getal" als een nieuw primitivum en niet als een subroutine opgebouwd met "lees volgende ponsing" in die zin, dat gebruik van "lees volgende getal" a fortiori gebruik van "lees volgende ponsing" zou impliceren. Hieraan verbinden wij twee consequenties.

Wij beschouwen "lees volgend getal" als een niet nader gespecificeerd proces, dat bij elke activering een volgende getalwaarde van de band distilleert (of alarm geeft). In het verwerkende programma is bv. niet te achterhalen, hoe deze getallen op de band gegeven zijn. Ze mogen via een flexowriter in decimalen geponst zijn, ze mogen wat mij betreft ook (door een vroeger ALGOL-programma) binair geponst zijn. Over het aantal mogelijkheden wil ik me nu niet uitlaten. In elk geval, bevat deze band, wat we vroeger controlecombinaties, wat we nu misschien "syntactische structuur" noemen.

De tweede consequentie is, dat we deze syntatische structuur zo kunnen maken, dat ook de omvang van de informatiehoeveelheid ondubbelzinnig -en wel buiten de invloedsfeer van de programmeur- vastgesteld kan worden. Een programma, dat zich in zijn contact met de eigen band beperkt tot die vormen, waarin deze eindvaststelling aldus geschieden kan, leest zijn eigen band "gestroomlijnd".

Als de eigen band voor gestroomlijnd lezen bestemd is en op de flexowriter gemaakt is, dan bestaat deze syntatische structuur naast de interne structuur van de getallen uit conventies ter getalscheiding, waarschijnlijk door middel van welgedefinieerde separatoren. Ik stel me voor een dergelijke band te doen afsluiten door een soort "superseparator", zeg het metakarakter enddata. Het gestroomlijnd lezen moet dan een zwaar controlerend proces zijn, dat o.a. onder geen beding het metakarakter enddata ongemerkt zou mogen laten voorbijgaan. Als het detectieproces voor het einde van de eigen band aldus tegen fouten van de programmeur is afgeschermd, mag de eigen band door een nieuwe, onafhankelijke band geolgd[sic] worden.

Een en ander suggereert, om de vertaler statisch de programma's in drie categorieen in te laten delen

- programma's, waarin geen eigen band gelezen wordt
- programma's, waarin de eigen band slechts gestroomlijnd gelezen wordt
- programma's, waarin de eigen band (slechts) ongestroomlijnd gelezen wordt.

Alleen programma's uit de categorieen I en II staan een achteraangeplakt volgend programma toe (dwz. naar wij hopen bijna allemaal).

Opm.1. Het vaststellen in welke categorie een programma valt, dient statisch te geschieden. Immers, als we het dynamisch afwachten, of en zo ja, hoe een programma contact met de eigen band opneemt, dan zouden we bedrogen uitkomen, zodra een programma uit categorie III ten onrechte zijn activiteit beeindigt, nog voordat contact met de eigen band is opgenomen.

Opm.2. Naar aanleiding van de haakjes om "slechts" in de omschrijving van III. Misschien haal ik deze weg, en sta ik niet toe, dat een band deels gestroomlijnd, deels ongestroomlijnd gelezen wordt. Dit gemengd lezen hinkt nl. erg op twee gedachten. Ik voorzie, dat pogingen om onder alle omstandigheden de semantiek bij overgang te definieren aanleiding geven tot een rommeltje, dat geen gebruiker het vertrouwen geeft geen vergissingen gemaakt te hebben. Liever breidt ik de mogelijheden van gestroomlijnd lezen wat uit.

Buffering.

Wanneer een band gelezen wordt, waarvan de feitelijke verwerkingssnelheid aanmerkelijk lager ligt dan die van het leesproces, dan zal, als er nog voldoende ruimte op de trommel is, de bandlezer moeten kunnen doorlopen en moet het bandbeeld in het geheugen opgebouwd worden.

Als we -even wat royaal rekend- zeggen, dat we octaden lezen, waarvan we er 3 in een woord kunnen pakken, dan bergen we dus 1500 octaden in een pagina van 512 woorden. Een volledige rol -300 meter a 400 ponsingen per meter- bevat 120.000 ponsingen en vult dus 80 pagina's, 8 procent van de trommel. Dit is alleszins acceptabel en we mogen verwachten, dat de lezer dus inderdaad haast altijd full-speed zal doorlopen. Door de snelheid van de schone schuifopdracht kan dit pakken dunkt me ook heel snel; van het idee om banden ongepakt op de trommel op te slaan, schrik ik nog wel een beetje. (Als de prijs van het pakken tegenvalt, kunnen we altijd nog overwegen, om als een band teveel ruimte in dreigt te nemen, pas dan er toe over te gaan om de rest te pakken.)

Hoe een en ander georganiseerd gaat worden is nog niet zo duidelijk. We hebben enige exploratie gepleegd van het volgende arrangement. Op grond van de overweging, dat een voorrekend leesproces impliceren gaat, dat het verwerkende programma het vervolg van het bandbeeld via de trommel gaat aanhalen, dachten we, dat het een eenvoudigere organisatie zouden krijgen, als we afspraken, dat het informatietransport van pakkingsprogramma naar verwerkende programma altijd via de trommel zou lopen. De verwachte simplificatie viel niet mee, integendeel; het zat not niet eens helemaal rond en toen was het al ber[sic] ingewikkeld.

We sullen nu proberen om een arrangement uit te werken, waarbij assemblage- en verwerkingsproces communiceren via een soort cyclische buffer, waarin de elementen "paginadescriptoren" zijn en zullen kijken, of we dit kunnen spelen, onafhankelijk ervan of de pagina's zelf zich nog op de kernen bevinden. De bedoeling is, dat indien bandbeeldpagina's naar en van de trommel getransporteerd worden, ze zich in niets onderscheiden van andere pagina's, die een de autonome trommeltransport onderworpen zijn. Misschien mode, omdat we dit nog niet hebben uitgewerkt, verwachten we hier veel van.

Ook bij output via ponsband willen we bufferen en we stellen ons voor, dat er normaliter pas een ponser voor een band gereserveerd wordt, als het volledige bandbeeld in het geheugen is opgebouwd. Hiermee hopen we drie dingen te bereiken.

Ten eerste zal hierdoor een programma een bandponser zo kort mogelijk aansluitend bezetten. Ten tweede is nu de lengte van de band bekend, voordat het ponsproces begint. Als de coordinator ingelicht wordt, wanneer er een nieuwe rol in een van de ponsers is ingelegd (en wanneer de operateur wat bescheiden met run out omgaat) dan kan de coordinator bijhouden, hoeveel papier er nog op de rol behoortte[sic] zitten. Er kan dus voor gezord worden, dat niet halverwege de band het papier plotseling op is. Ten derde kan men dit bandbeeld twee maal opbouwen, nl. eerst om te ponsen, en later om te controleren. Dit controleproces willen wij beschouwen als een van de standaardhandelingen, die geactiveerd kan worden, door een band met een passend metakarakter in een open bek te leggen. Dit extra metakarakter zal het ponsproces er ongevraagd vooraan toevoegen, als het zijn werk gedaan heeft, kan de operateur het afscheuren. Het succesvol beeindigen van het controleproces kan de geheugenruimte van het bandbeeld vrijgeven.

transcribed by David J. Brantley

latest revision Sat, 29 May 2004

---

## English translation

### A punched-tape organization for the X8

EWD082     X-8 no. 38.

A punched-tape organization of the X8.

Motto: "He who does not honour the small,

is not worthy of the great."

Introduction.

In this report the main emphasis will fall on the reading of punched tape. Why the punching need raise fewer large questions will become clear in the course of this.

The tape-reading instruction of the X8 is fully code-independent; every configuration of holes or no holes penetrates to the machine and in a certain sense "there is no problem", namely in the sense that we can program the processing of these tapes as we wish. Conversely, this does impose upon us the duty to choose, out of the welter of possibilities, what we want. A large part of this report concerns itself with this choice (and the motivation thereof).

The guiding principle is that things must be made as easy as possible for the user. This means among other things that when a user wishes to read successive numbers from a punched tape, he then wishes to incorporate an internally fitting identification of numerical values into his process, without his having to be aware of the conventions according to which these numbers stand on the tape. This situation is entirely analogous to his attitude towards variables of type integer or real: for him these characterize values; how is none of his concern; the internal representation need not even be unique. With such a manner of tape reading all kinds of "irrelevantia" must therefore have disappeared under the table before the numerical value penetrates to the program.

The complication here, however, is that we cannot a priori decree what is irrelevant to the programmer. Punched tapes, after all, lead an independent existence outside the machine; they may come from automatic measuring apparatus, they may have to be made for tape-controlled machinery. What we are therefore aiming at is an organization which on the one hand serves convenience, but which on the other hand, if desired, gives the programmer the full control he needs in order to be able to use punched tape as a "general purpose medium".

A further complication is the mixed use one wishes to make of the physical "end of tape". "End of tape" is detectable by the X8; the consequences that ought to be attached to it can, however, differ considerably from case to case.

There are programs in which end of tape is an essential part of the incoming information stream, e.g. a program that reproduces an arbitrary tape. (In this light it is a pity that the punch does not have a little cutter controllable by the X8!)

In other organizations one would rather attach no meaning to end of tape. If a program asks for a very long tape that is, for example, built up out of two pieces, then it is of course pleasant that the EL 1000 swallows carefully made splices, but it would be a pity if this also actually obliged the operator to splice. If the operator for one reason or another prefers to load such a tape in two pieces, this ought to be permitted without thereby wanting to impose an extra burden of an obligation to splice; nor does one wish to impose upon the operator the obligation to cut -i.e. the prohibition against splicing- at least in the majority of cases, in which no ambiguities can arise from it. (It is clear that one may not, with impunity, splice another tape onto the back of a tape on which "end of tape" is an essential part of the information.)

The idea of the standard tape and "the open mouth".

We start from the assumption that the majority of ALGOL programs are such that, if they ask for numerical material,

- they need not react to "end of tape"
- this material can be supplied in advance.

The (operationally) normal case shall be that this tape with numerical material is then spliced onto the back of the program text (or, as the case may be, punched immediately after it). We call such a tape with material to be processed "the own tape" of this program.

Here the beginning of the tape -namely the program text- indicates how the rest -namely the numerical material- is to be processed. We wish to extend this idea a step further and agree that at the front of the text a recognizable standard cry, say algol, occurs. This is a metacharacter which implies that the punchings following it are to be interpreted as the text[sic] of an ALGOL program. This interpretation (translation, followed by execution) is a standard processing; the metacharacter algol has been introduced onto the tape because I do see room for extensions of the number of standard processings. All these standard processings will be activatable via an associated metacharacter.

A tape which subjects itself to such a standard processing we call a standard tape. The aim we are striving for is to force the overwhelming majority of operator actions into the straitjacket of "processing of a standard tape."

To this end we introduce the concept of the "tape reader with the open mouth". The intention is that, for the processing of a standard tape, the operator need do no more than load such a standard tape into a tape reader and press the green button. The state in which a tape reader thus reacts in the standard manner to a standard tape we call "with open mouth".

The intention is that all tapes, once loaded, are in principle read in at full speed -i.e. regardless of the pace of actual processing. If the reader runs faster than the actual processing, then the rest can be dumped onto the drum. As soon as "end of tape" has been signalled, the reader is released and remains behind with its mouth open.

N.B. We do not require that "end of tape" be signalled before the actual processing begins. If there is congestion in the store, then the coordinator must be able to choose and to be able to prefer "reader occupancy" over "store occupancy".

There are already some snags to this, because tape readers will not be used exclusively for the standard processing of standard tapes. (There also exists non-standard processing of what otherwise deceptively looks like a standard tape!)

The price that we have to pay for the easy processing of standard tapes consists of some extra hocus-pocus for the non-standard reading. One must namely make a safe little lock-gate for the case in which the operator, in good faith, loads a standard tape into a reader at precisely the same moment that the X8 decides to start using this reader for another purpose.

In broad outline a single-tape reader can find itself in three states:

state 0: free

state 1: requested for non-standard work

state 2: occupied

In state 0 there hangs a rather symbolic start instruction for 1 single heptade, which is given only in order to let the loading of a new tape and the pressing of the green button penetrate to the X8. (I assume that the heptade itself will perhaps be ignored.)

In state 1 the X8 has decided that the tape reader is going to be used for non-standard work; this has been reported via the teleprinter. The tape reader will not run on before the operator has answered this report that there is no wrong tape lying in the tape reader.

In state 2 the tape reader has been definitively engaged for a well-defined (standard or not) task.

If now an interrupt occurs from a tape reader that finds itself in state 0, then this tape is read on as a standard tape. In deafness, as a reaction to the symbolic heptade, the transition to state 2 takes place (and it is noted that we are now doing standard-tape reading). The following interrupts take place by definition in state 2.

If a (first) interrupt occurs in state 1, then until further notice no more tape is read. The operator has namely loaded a standard tape during the period in which the X8 has already allocated the reader for another purpose, but the operator has not yet answered that he has taken note of this decision. The operator notices this among other things from the fact that of the tape, which he expects will shoot inward, only 1 heptade is read.

The operator's answer -implying that there is no unrequested tape lying in the reader- effects the transition from state 1 to state 2, a complete specification of what must then in fact happen, and finally the filling of the start magazine of the reader in question. If the tape is already loaded, then it starts running; the operator may also give the answer in advance, while the reader is empty, after which the tape starts running as soon as it has been loaded.

The concept of the tape variable.

If a tape is processed "synchronously with the physical reading process", as with ARMAC and X1, then the position of the tape in the reader functions as a memory element. By the time we have stored the tape image in the store, we must incorporate into the store this "little heptade pointer", which governs the sequential scanning of the tape image. This heptade pointer is an important constituent of the so-called tape variable.

Furthermore a tape variable contains the datum of whether the physical reading process has already terminated or not. If not, then it contains a reference to the physical reader; the reader administration in its turn contains a reference to the tape variable. The coupling "tape variable - reader", which remains in existence until the X8 has found the signal "end of tape" for this reader, is thus booked mutually. As soon as the signal "end of tape" penetrates to the X8, this coupling is broken. The tape variable then no longer has any knowledge of via which reader this tape came in; the reader itself passes into state 0 with the start instruction for the single heptade in the start magazine. (Thanks to the symbolic dispatching of any start instructions in the magazine as soon as end of tape has been signalled, it is possible to ensure that this is the only start instruction in the magazine.)

A tape variable always forms part of a program (except perhaps in the very first analysis of the metacharacter at the front of a standard tape). During translation the tape variable is a quantity of the translator; at the end of the translation the then prevailing value of the tape variable functions as the initialization of the own-tape variable of the translated program. This transfer of the tape variable from translator to translated program must be done with some care: because the tape reader need not yet have completed the reading process, a tape variable therefore harbours within itself a possible synchronization restriction. It is this synchronization restriction that must also be handed over from translator to translated program.

Streamlined reading and the role of metacharacters.

When we have more than 1 program sitting simultaneously in the machine, then we have, as is known, to ensure that the one program cannot be so faulty that thereby the other program is disturbed. This guarantee implies restrictions with respect to the structure of the individual programs. I have always taken the standpoint that every user who cannot accept these restrictions because he wants to program in machine code must then just run the machine in uniprogramming.

An analogous situation we encounter when we wish to feed in two programs, which otherwise have nothing to do with one another, one after the other on one and the same tape. The processing system then has the duty to establish where the text of the second program begins. The composer of the first program must then not have the possibility of making such an error that the correct establishing of the position of this caesura is no longer a reliable operation. In order to be able safely to feed two programs one after the other[sic] on the same tape, the first program must observe certain rules of the game, entirely analogous to the situation signalled above. By the nature of the matter we shall strive to make these restrictions as little jarring as possible.

In order not to complicate life needlessly, I assume in the following that a program can make contact with the own tape only in two ways:

- read next punching
- read next number.

In case a) the programmer has access to every bit of the fed-in tape and he can react to this at his own discretion. A program that reads the punchings as such from the own tape is like the program in machine code; it is not permitted to splice onto the back of the own tape of this program the text of an independent following program. (Whoever would want to behave nastily then writes an ALGOL program whose only effect is that the following ALGOL program is skipped over by hand!)

A program which, in its contact with the own tape, however, restricts itself to "read next number" can be kept under the thumb by the system. I regard "read next number" here as a new primitive and not as a subroutine built up with "read next punching" in the sense that use of "read next number" would a fortiori imply use of "read next punching". To this we attach two consequences.

We regard "read next number" as a not further specified process which, on each activation, distils a next numerical value from the tape (or gives an alarm). In the processing program it is for example not to be ascertained how these numbers are given on the tape. They may be punched via a flexowriter in decimals; for all I care they may also (by an earlier ALGOL program) have been punched binarily. About the number of possibilities I do not wish to commit myself now. In any case, this tape contains what we formerly called check combinations, what we now perhaps call "syntactic structure".

The second consequence is that we can make this syntactic structure such that also the extent of the quantity of information can be established unambiguously -and indeed outside the sphere of influence of the programmer. A program which, in its contact with the own tape, restricts itself to those forms in which this end-establishment can thus take place, reads its own tape "in streamlined fashion".

If the own tape is intended for streamlined reading and has been made on the flexowriter, then this syntactic structure consists, besides the internal structure of the numbers, of conventions for number separation, probably by means of well-defined separators. I imagine having such a tape closed off by a kind of "superseparator", say the metacharacter enddata. The streamlined reading must then be a heavily checking process which among other things must under no circumstances let the metacharacter enddata pass by unnoticed. If the detection process for the end of the own tape is thus shielded against errors of the programmer, then the own tape may be followed[sic] by a new, independent tape.

All this suggests letting the translator statically divide the programs into three categories

- programs in which no own tape is read
- programs in which the own tape is read only in streamlined fashion
- programs in which the own tape is read (only) in non-streamlined fashion.

Only programs from categories I and II permit a following program spliced onto the back (i.e. as we hope, almost all of them).

Remark 1. The establishing of which category a program falls into must take place statically. For if we await dynamically whether and, if so, how a program makes contact with the own tape, then we would come out deceived as soon as a program from category III wrongly terminates its activity before contact with the own tape has been made.

Remark 2. With reference to the brackets around "only" in the description of III. Perhaps I shall remove these, and not permit a tape to be read partly in streamlined and partly in non-streamlined fashion. This mixed reading namely limps very much on two thoughts. I foresee that attempts to define the semantics at the transition under all circumstances give rise to a mess that gives no user the confidence of not having made mistakes. I would rather extend the possibilities of streamlined reading somewhat.

Buffering.

When a tape is read whose actual processing speed lies considerably lower than that of the reading process, then, if there is still sufficient room on the drum, the tape reader must be able to run on and the tape image must be built up in the store.

If we say -reckoning a bit generously- that we read octads, of which we can pack 3 in a word, then we store 1500 octads in a page of 512 words. A complete roll -300 metres at 400 punchings per metre- contains 120,000 punchings and thus fills 80 pages, 8 percent of the drum. This is in every respect acceptable and we may expect that the reader will therefore indeed almost always run on at full speed. Owing to the speed of the clean shift instruction this packing can also, it seems to me, be done very fast; at the idea of storing tapes unpacked on the drum I still recoil a little. (If the price of the packing turns out disappointing, we can always still consider, when a tape threatens to take up too much room, only then proceeding to pack the rest.)

How all this is going to be organized is not yet so clear. We have carried out some exploration of the following arrangement. On the basis of the consideration that a read process running ahead is going to imply that the processing program is going to fetch the continuation of the tape image via the drum, we thought that we would get a simpler organization if we agreed that the information transport from packing program to processing program would always run via the drum. The expected simplification did not work out; on the contrary, it was not even quite worked out completely and already it was very[sic] complicated.

We shall now try to work out an arrangement in which the assembly and processing processes communicate via a kind of cyclic buffer, in which the elements are "page descriptors", and shall see whether we can play this regardless of whether the pages themselves still find themselves in the cores. The intention is that if tape-image pages are transported to and from the drum, they distinguish themselves in nothing from other pages that are subjected to the autonomous drum transport. Perhaps fashionably[sic], because we have not yet worked this out, we expect much of this.

With output via punched tape too we wish to buffer, and we imagine that normally a punch is only reserved for a tape when the complete tape image has been built up in the store. With this we hope to achieve three things.

In the first place a program will thereby occupy a tape punch for as short a continuous time as possible. In the second place the length of the tape is now known before the punching process begins. If the coordinator is informed when a new roll has been loaded into one of the punches (and when the operator deals somewhat modestly with run out), then the coordinator can keep track of how much paper ought still to be on the roll. It can thus be ensured that the paper does not suddenly run out halfway through the tape. In the third place one can build up this tape image twice, namely first to punch, and later to check. This checking process we wish to regard as one of the standard actions which can be activated by laying a tape with a fitting metacharacter into an open mouth. This extra metacharacter the punching process will add at the front unrequested; when it has done its work, the operator can tear it off. The successful termination of the checking process can release the store space of the tape image.

transcribed by David J. Brantley

latest revision Sat, 29 May 2004
