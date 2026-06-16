---
title: "Het vectorgeheugen (EWD41)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD41.html
published: "1963-02-07T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD41.PDF
---

# Het vectorgeheugen

EWD 41

DUTCH-ENGLISH DICTIONARY OF PRINCIPAL KEYWORDS

Boom, Tak, Twijg Tree, Branch, Twig

Kernengeheugen Core storage

Seinpaal Semaphore

Trommelgeheugen Drum storage

Het vectorgeheugen.

In de meeste multi-programmeringssystemen, die ik gezien heb, moet elk individueel programma zijn geheugenbehoefte opgeven in de vorm van "zoveel aansluitende geheugenplaatsen". Dynamische wijziging van dit aantal stoot meestal op moeilijkheden - vooral als een programma in de loop van zijn uitvoering tot de ontdekking zou komen, dat het eigenlijk wat meer zou willen hebben.     Mijn opmerking is de volgende: elk programma drukt hier zijn behoefte aan geheugen uit als behoefte aan een vector van gegeven lengte en zodra een aantal van deze programma's samen in eenzelfde geheugen georganiseerd worden, dan beschikt deze overkoepelende organisatie dus over een of andere techniek om een aantal vectoren in een lineair geheugen onder te brengen. Ik zou de koppeling "een individueel, sequentiëel programma versus een enkele vector" willen laten vervallen en elk individueel programma de mogelijkheid willen geven, zijn geheugenbehoefte in termen van zo veel vectoren als het maar wil, uit te laten drukken.

Dit heeft enkele potentiële voordelen. Teneerste is het een grote uitzondering, wanneer de behoefte aan geheugen, die door een bepaald proces gesteld wordt, zich op de meest natuurlijke wijze als een enkele vector laat uitdrukken. Men zou, ook bij uni-programmering, van een dergelijke faciliteit veel plezier kunnen hebben. Tentweede schept dit de mogelijkheid tot een wat natuurlijkere eenheid, waarin hoeveelheden informatie van random access geheugen naar, zeg trommel, gedumpt kunnen worden. Tenderde zijn de individuele programma's zo evident niet in termen van physische adressen geformuleerd, dat verwacht mag worden, dat herindeling van het physische geheugen met grotere vrijheid kan geschieden.

In het volgende zal de aanwezigheid van een "tweeslachtig geheugen" (mijn vertaling van "two level store") nauwelijks ter sprake komen. Over de strategie, volgens welke de informatie over beide vormen van geheugen verdeeld zal moeten worden, zal ik het niet hebben; over de administratie van de trommelindeling evenmin.

Mijn voornaamste zorgen zijn voorlopig:

(a) dat de programma's in hun formulering - dwz. de manier, waarop ze in het kerngeheugen gerepresenteerd worden - zich beperken tot het "eigene", voor zichzelf wezenlijke; maw. dat deze formulering onafhankelijk is van de wijze, waarop dit programma in zijn omgeving is ingebed;
(b) dat de effectieve orde der adressering niet explodeert;
(c) dat de administratieve taken, die voortvloeien uit de plicht bij herindeling van het geheugen de administratie bij te werken, duidelijk omschreven blijft;
(d) dat duidelijk is, welke de "korrelgrootte" is van de programma's, dwz. welke handelingen niet door herindeling van het geheugen verstoord mogen worden. We zouden in practische moeilijkheden kunnen geraken, als de uitvoering van een korrel willekeurig lang zou mogen duren, we zouden een contradictie krijgen, als de uitvoering van een korrel ... herindeling van het geheugen noodzakelijk zou maken!
(e) tenslotte een practische zorg: het vinden van een terminologie om hierover te kunnen praten.

Ter vermijding van een babylonische spraakverwarring eerst de volgende afspraak: ik zal het woord "adres" alleen gebruiken in de betekenis van "physisch adres", de bitrij, die het selectieregister van een geheugen ingestuurd kan worden. Elk programma zal zijn elementen identificeren als "nummer zoveel van een vector" en dit rangnummer zal ik een "index" noemen.

Een vector ter lengte N bestaat uit N elementen, onderling onderscheiden door een indexwaarde i; hierbij gelden de ongelijkheden  0 <= i <=  N - 1. De identificatie van de vector - noodzakelijk indien er in een gegeven context meer vectoren voorkomen - is mogelijk, doordat aan elke vector een zg. "basis" is toegekend en een basis, als element van een andere vector weer door een rangnummer identificeerbaar is.

(De bedoeling is, dat een vector in kerngeheugen een aantal consecutieve geheugenplaatsen zal beslaan; de basis van deze vector bevat dan onder andere beginadres en lengte van de er op steunende vector.)

Het bestaan van een basis is het recht om over een vector te mogen praten; de identificatie (dwz. de indexrij) van de basis fungeert als identificatie van deze vector. Hel zal later duidelijk worden, dat een basis - hoewel het een indiceerbaar element van een vector is - niet beschouwd mag worden als een willekeurig element, dat informatie representeert, die beschouwd mag worden als subject matter van het proces. (Een gedeelte van de basis zou zijn het beginadres van de er op steunende vector, maar physische adressen hebben in de context van het programma geen betekenis!) Wij hebben dus twee typen elementen: elementen, waarop het programma vrijelijk opereren mag (getallen) en bases. Om een aantal redenen leggen wij ons de beperking op, dat er geen gemengde vectoren zullen zijn: alle elementen van dezelfde vector moeten van hetzelfde type zijn. Zijn de elementen getallen, dan spreken we zo nodig over een "twijg", zijn de elementen bases van andere vectoren, dan noemen we de vector een "tak". (Een basis bevat, behalve lengte en beginadres, ook een specificatie van het type van de er op steunende vector.)

Elke context veronderstelt de toegankelijkheid van een "impliciete basis", nl. de basis van de vector, waarin de eerste index van een indexrij als rangnummer geinterpreteerd zal worden.

Toelaatbare operaties op vectorbases zijn: definitie van het type van de er op steunende vector; definitie van de lengte van de er op steunende vector.

Als een bestaande vector verlengd wordt, worden er enige "ongedefiniëerde" elementen aan toegevoegd; als het de verlenging van een tak is, worden er dus "ongedefiniëerde" bases toegevoegd. Een ongedefiniëerde basis bevat op de plaats de typespecificatie de notitie "nog ongespecificeerd". Na deze introductie van de basis mag het type van de er op steunende vector één keer gedefiniëerd worden. (We sluiten typeveranderingen dus uit voor bestaande vectoren, ook in het geval, dat de vector op dat ogenblik de lengte 0 had.) Een ongespecificeerde vector is iets anders dan een lege vector: hoewel er alleen maar "niks" uit zou kunnen komen, heb ik toch bezwaar tegen optellingen als "0 koeien plus 0 ganzen".

Als een bestaande vector verkort wordt, gaat er een aantal van de hoogstgenummerde elementen van deze vector verloren; was de vector een tak, dan gaan met de bases, die afgevoerd worden, alle vectoren verloren, die daar op hebben gerust (en als daar weer takken onder waren, alle vectoren, die daar weer op rusten etc.)

(De in de eervorige alinea genoemde restrictie wat betreft constantheid van type, kan men ondervangen, door als extra operatie op een vectorbasis in te voeren: het ongespecificeerd maken. Zo er al iets op deze basis rustte, gaat dat bij de ongespecificeerdmakerij verloren. Andere oplossing is, om deze ongespecificeerdmakerij geimpliceerd te laten zijn bij typewisseling. Of dit allemaal erg belangrijk is, waag ik te betwijfelen.)

Operaties op vectorbases geschieden onder opgave van de indexrij, die de basis in kwestie identificeert; is deze indexrij leeg, dan is per definitie de impliciete basis bedoeld. Context zal dus in het algemeen beginnen, de impliciete basis te definiëren als basis van een tak van zekere lengte. Daarmee is een willekeurig aantal nieuwe bases geintroduceerd en daarmee is de mogelijkheid geschapen, om nieuwe vectoren te introduceren.

Referentie naar element i van een vector met lengte N, waarbij i niet voldoet aan  0 <= i <= N - 1  is onzin en heeft geen betekenis: het is dan ook de bedoeling dat elke selectie van een vectorelement impliceert, dat er getest wordt, of er aan deze ongelijkheden voldaan is. (Je eist van elk programma een zekere mate van consistentie; is hieraan voldaan, dan heeft de gebruikelijke vorm van "Memory protect" om het programma van meneer Jansen te beschermen tegen de capriolen van meneer Pieterse geen enkele zin meer. Immers, meneer Pieterse beschikt niet meer over de terminologie om meneer Jansen te hinderen.)

Het is duidelijk, hoe elke context op deze manier zijn locale variabelen op uiterst flexibele manier kan invoeren en bespelen. Op de impliciete basis wordt een boompje geplant, dat tijdens zijn bestaan kan groeien en besnoeid kan worden. Nu wordt het echter de hoogste tijd om te zien, hoe onze context met het locale boompje past in zijn omgeving. De impliciete basis, waarover wij gesproken hebben, is nl. geen absoluut machinegegeven: als wij een zo grote stap terug doen, dat wij het gehele geheugen van de machine kunnen overzien, dan is wat intern, dwz. in een gegeven context als impliciete basis fungeert, extern een element, dat zijn plaats in de totale boom heeft en als ieder ander element geidentificeerd wordt door een indexrij van een of andere lengte.

Bovendien: zonder uitzondering kunnen we stellen, dat elke context niet "self-contained" zal zijn, maar slechts kan werken bij gratie van voorafgaande specificatie van een aantal parameters. Een programma, dat een van de communicatieapparaten gebruikt, zal dit communicatieapparaat als "formeel" apparaat toespreken; bij activering moet er als overeenkomstige actuele parameter een specifiek apparaat meegegeven worden. Ook dit valt allemaal onder "inpassing in de omgeving".

Om de samenwerking van een gegeven context te bewerkstelligen met grootheden (vectoren, communicatieapparaten), die intern een formele naam hebben en extern een naam, die is uitgedrukt in een terminologie, die intern geen betekenis heeft, heb ik na rijp beraad en lange aarzeling een nieuw soort vector ingevoerd, de zg. "parametervector".

Bepaalde context kan slechts werken, als zij de beschikking heeft over en de impliciete basis en de bijbehorende parametervector. Omdat dit een onafscheidelijk tweetal is, stel ik voor ze samen in dezelfde vector onder te brengen. (De overige vectoren, takken en twijgen, waren homogeen. We voeren dus nu in de parametertak, een vector met een heel specifieke indeling, bv. het nulde element is de impliciete basis, het eerste element is de heersende waarde van de opdrachtteller en de overige elementen zijn parameters. Het zou me niet verbazen, als het begin van de parametervector een heel geschikte bergplaats was voor nog wat meer impliciete "toestandsvariabelen".) Algemeen begint de parametervector met een vast aantal (minstens 1) vectorbases, eventueel een vast aantal elementen voor toestandsvariabelen, en tenslotte een aantal parameters. Als een vector parametervector is, dan is dat in zijn basis vermeld. In het programma wordt naar de formele parameters verwezen door
(a) een indicatie, dat het hier een formele parameter betreft; dit selecteert de parametervector;
(b) een index, die de parameter specificeert.

Het idee is, dat een gegeven context A een context B als volgt activeert. Context A reserveert een parametervector, vult daarin alle parameters in en laat dan de machine overspringen naar de nieuwe parametervector annex nieuwe impliciete basis.

Ik wil nu speciaal even nagaan, hoe we een bestaande vector aan een nieuwe context als parameter meegeven. Ik wil dit doen, door op de bijbehorende plaats van de parametervector een copie van de basis van de vector in te vullen. Dit is heel griezelig, maar voor mijn gevoel moet ik dit doen.

Laten we even teruggaan naar het idee van de impliciete basis: een bepaald programma is geformuleerd in termen van zijn impliciete basis en spreekt zich er niet over uit, hoe lang de indexrij is, die in de externe wereld deze impliciete basis volledig identificeert. De selectie tengevolge van een indexrij in specifieke context zou dus betekenen, dat je deze indexrij in gedachten vooraf zou moeten laten gaan door de indexrij, die nodig is om in de omgeving de impliciete basis te vinden. Dat zou betekenen, dat de efficiency, waarmee een programma uitgevoerd wordt afhangt van de afstand tot de wortel van de boom en dat was niet de bedoeling.

Zodra ik echter vectorbases copiëer, dan moet ik er wel voor zorgen, dat bij wijziging van deze basis - verschuiving van de er op steunende vector bv. - dit niet alleen in de originele, unieke basis, maar ook tot zijn copieën doordringt.

Het idee van de aparte parametervector is ingevoerd om bij wijziging van een basis te weten, dat je slechts parametervectoren hoeft te scannen om te kijken, of je elementen tegenkomt, die mee gewijzigd moeten worden. Ik geloof, dat dit sneller uitkomt, dan bij elke copiëring bij de bron vermelden, waar de copie staat, te meer daar je bij een basis met een enkel nog wel bezuiniging kunt bereiken. (Je kunt bij elke basis een "gecopiëerd-bit" invoeren. De ene waarde betekent: deze basis is beslist niet gecopiëerd, de andere waarde betekent: deze basis is mogelijk wel gecopiëerd. Bij elke copiëring van een basis vermeld je dit feit bij deze basis. Een basis kan op deze manier op een gegeven ogenblik een aantal copieën hebben, maar dat tel je niet. Als sommige van deze copieën door vectorverkorting verdwijnen, dan laat je dat verdwijnen voorlopig onopgemerkt. (Zolang de basis niet gewijzigd wordt, doet de mogelijke copiëring er ook niet zo veel toe.) Pas als je de vector gaat verschuiven, kijk je naar de copiëringsbit. Als deze staat in de stand "mogelijk een copie", dan onderzoek je de mogelijke parametervectoren, om te kijken of je wat hebt bij te werken. Vind je inderdaad copieën, dan werk je deze bij, anders zet je de "gecopiëerdbit" weer terug.)

Zolang de ene context slechts sequentiëel andere contexten activeert - de ene procedure slechts aanroept als de vorige helemaal klaar is - kan deze activerende context volstaan met slechts een enkele parametervector. Ook voor het geval, dat een bepaalde context wil beschikken over meer parametervectoren, is er, als ik me niet vergis, wel een acceptabele conventie te vinden, zodat in een bepaalde context de parametervectoren gemakkelijk te vinden zijn.

Als je een subroutine aanroept, moet je daarbij vermelden, welke subroutine je dan aanroept. Evenzo wilde ik bij activering van een bepaald programma dit programma meegeven als parameter van het activeringsmechanisme. Ik beschouw een programma als een vector en we spreken bv. af, dat we de basis van deze vector copiëren op de plaats van de eerste parameter, voordat we de machine laten overgaan op de nieuwe parametervector.

Tot slot een paar opmerkingen.

Het in parallelle programmering activeren van een nieuw programma kun je beschouwen als een operatie, waarvan het "gewoon aanroepen van een subroutine" een bijzonder geval is. Eén van de parameters laat je een seinpaal zijn, die voor de activering (voor de aanroep) door het buitenste programma op FALSE gezet wordt. Vervolgens activeert het buitenste programma de subroutine en blijft dan hangen op een passering van deze seinpaal. De subroutine eindigt met zijn voltooiing aan te geven door de seinpaal in kwestie op TRUE te zetten, zodat het aanroepende programma pas doorgaat, als de subroutine voltooid is. Na doorgang kan het aanroepende programma deze activering - deze wachtende machine - weer van de masterwachtlijst afvoeren. Ik betwijfel, of dit de meest practische manier is om subroutines aan te roepen, maar het is wel verhelderend, dat het in wezen niets anders is dan het toevoegen van een nieuw programma aan de wachtlijst.

Ik ben er in stilte van uitgegaan, dat de aanwezigheid van een vector in de kernen zou impliceren, dat zijn basis zich ook op de kernen bevindt. Liefst hield ik alle takken op de kernen, ook de parametertakken. Parametertakken allemaal op de kernen houden is daarom wel prettig, omdat verschuiving kan impliceren, dat je in parametertakken ver van de wortel nog correcties moet aanbrengen. Als het noodzakelijk is, om parametertakken op de trommel te dumpen, dan is daar overigens nog nog wel wat aan te doen. Dan moet je bij elke parameter in de tak tevens vermelden, welke indexrij in de context, die de parameter gevuld heeft, deze parameter heeft aangehaald. Dan moet je bij terughalen van de parameterrij van de trommel je realiseren, dat je nu een parametertak van de trommel haalt, dat daarop correcties eventueel niet zijn bijgehouden, maar je hebt dan alle gegevens op kernen, om de parameterrij, die een tijd lang op de trommel "geslapen heeft" weer up to date te maken.

In welke terminologie de wachtende programma's op de wachtlijst van de master onthouden moeten worden is voor mij nog een open vraag; hetzelfde geldt voor de geprogrammeerde seinpalen.

Ik heb vergeten te vermelden, dat ik met opzet het uit te voeren programma als parameter aan het activeringsmechanisme meegeef, en dat ik een dergelijk programma beschouw als een constante vector. (Hier kun je voor zorgen, door in het programma zelf niet de terminologie ter beschikking te stellen, waarmee het zich zou kunnen overschrijven. Onze ALGOL-programma's overschrijven zichzelf ook niet.) Het is nl. heel aantrekkelijk, om een programma, dat output-limited is, in een aantal activeringen tegelijkertijd aan het werk te hebben, zonder het programma meer dan eens op de kernen te hebben staan. (Wat dachten we van het vertaalproces? Of het "consoleprogramma" als je meer dan één console hebt?)

De grondgedachte bij de kernenbezetting is, dat een vector helemaal wel, of helemaal niet op de kernen te vinden is. Dat noteer je dan in de basis. Dat je op deze manier geen vectoren in zou kunnen voeren, die langer zijn dan de omvang van het kernengeheugen, is een hinderlijkheid, die echter automatisch op te lossen is (door een impliciete ordeverhoging; dat dit gebeurd is, kun je eveneens in de basis aangeven.)

Tenslotte: ik heb zo'n gevoel, dat de transporttijden van en naar langzaam geheugen wel eens een bottleneck zouden kunnen worden. Als je een vector uit het langzame geheugen hebt overgenomen, kun je met een bit in de basis aangeven, dat de copie nog "gaaf" is; schrijf je in die vector, of verander je zijn lengte, dat zet je die bit in de andere stand. Op die manier kun je je vrijwat nodeloos terugtransporteren wel besparen. Als de tijd tussen aanvraag en begin van het feitelijke transport lang is vergeleken bij de feitelijke transporttijd (omdat de trommel in de verkeerde stand staat) kon het wel eens de moeite lonen, om aangevraagde transporten even te verzamelen en desnoods via programma te sorteren. Dit kan wel eens zoveel te meer noodzakelijk zijn, omdat een heleboel vectoren kort zullen zijn vergeleken bij de capaciteit van een heel trommelspoor.

Slotopmerkingen.

Ik ben mij maar al te goed bewust, dat in het voorafgaande geen afgerond plan is beschreven. Integendeel: dit laat nog zoveel vraagtekens over, dat uitwerking een considerabele hoeveelheid werk zal zijn. Voordat ik mij daar in stort, wil ik graag de opinie van wat andere mensen hebben, hoe uitvoerbaar of hoe heilloos dit hun voorkomt.

Ik kan me voorstellen, dat de lezer het gevoel heeft een kijkje achter de schermen van andermans nachtmerrie gekregen te hebben. Het misschien niet overtuigende verweer, dat ik daartegenover stellen kan, is, dat toen het ALGOL-complex voor de X1 in zijn huidige vorm begon te dagen, ons de eerste drie weken ook de rillingen over de rug liepen. Het kan achteraf wel eens heel goed geweest zijn, dat wij slechts etappegewijze met de consequenties van die aanpak geconfronteerd zijn geworden; anders had het ons meer moeite gekost de euvele moed op te brengen. In mijn niet zeldzame momenten van twijfel put ik wel moed (of moet ik zeggen: overmoed?) uit die ervaring.

Er is nog een reden, waarom ik dit plan nu graag aan anderen wil voorleggen. Van het hele veld problemen, heb ik er een paar uitgekozen en er over gedacht, hoe je die zou kunnen oplossen. Ik heb me daar alleen maar in kunnen verdiepen, door er maar eens van uit te gaan dat de rest wel oplosbaar is. Je hoopt natuurlijk, dat in de keuze van waar je eerst naar kijkt, je intuitie je niet bedriegt, maar je loopt natuurlijk altijd de kans, dat je een paar heel wezenlijke moeilijkheden onbewust naar achteren schuift omdat je er geen verstandig woord over weet te zeggen.

Vandaar, dat dit stuk besloten wordt met een verzoek om commentaar.

E.W.Dijkstra

transcribed by Johan E. Mebius
revised Sun, 28 Mar 2010

---

## English translation

### The vector store

EWD 41

DUTCH-ENGLISH DICTIONARY OF PRINCIPAL KEYWORDS

Boom, Tak, Twijg Tree, Branch, Twig

Kernengeheugen Core storage

Seinpaal Semaphore

Trommelgeheugen Drum storage

The vector store.

In most multi-programming systems that I have seen, each individual program must state its store requirement in the form of "so many contiguous storage locations". Dynamic alteration of this number usually runs into difficulties — especially if a program were to discover, in the course of its execution, that it would actually like to have somewhat more.     My observation is the following: here each program expresses its need for store as a need for a vector of given length, and as soon as a number of such programs are organized together in one and the same store, then this overarching organization evidently disposes of some technique or other for accommodating a number of vectors in a linear store. I should like to drop the coupling "an individual, sequential program versus a single vector" and to give each individual program the possibility of expressing its store requirement in terms of as many vectors as it likes.

This has a few potential advantages. In the first place it is a great exception when the need for store posed by a particular process most naturally expresses itself as a single vector. One could, in uni-programming too, derive much pleasure from such a facility. In the second place this creates the possibility of a somewhat more natural unit in which quantities of information can be dumped from random access store to, say, drum. In the third place the individual programs are so evidently not formulated in terms of physical addresses, that one may expect that rearrangement of the physical store can take place with greater freedom.

In what follows the presence of a "two-natured store" (my translation of "two level store") will scarcely come up for discussion. I shall not speak about the strategy according to which the information will have to be divided over both forms of store; nor about the administration of the drum layout.

For the time being my principal concerns are:

(a) that the programs in their formulation — i.e. the manner in which they are represented in the core store — confine themselves to what is "their own", essential to themselves; in other words, that this formulation is independent of the way in which this program is embedded in its environment;
(b) that the effective order of the addressing does not explode;
(c) that the administrative tasks, which arise from the duty to update the administration upon rearrangement of the store, remain clearly defined;
(d) that it is clear what the "grain size" of the programs is, i.e. which operations may not be disturbed by rearrangement of the store. We could get into practical difficulties if the execution of a grain were allowed to last arbitrarily long; we would get a contradiction if the execution of a grain ... were to make rearrangement of the store necessary!
(e) finally a practical concern: the finding of a terminology in which to be able to talk about this.

To avoid a babylonian confusion of tongues, first the following convention: I shall use the word "address" only in the meaning of "physical address", the bit string which can be fed into the selection register of a store. Each program will identify its elements as "number such-and-such of a vector" and this rank number I shall call an "index".

A vector of length N consists of N elements, mutually distinguished by an index value i; here the inequalities  0 <= i <=  N - 1 hold. The identification of the vector — necessary if more than one vector occurs in a given context — is possible because to each vector a so-called "base" has been assigned, and a base, being an element of another vector, is in turn identifiable by a rank number.

(The intention is that a vector in core store shall occupy a number of consecutive storage locations; the base of this vector then contains, among other things, the starting address and the length of the vector resting upon it.)

The existence of a base is the right to be allowed to speak about a vector; the identification (i.e. the index string) of the base functions as the identification of this vector. It will become clear later that a base — although it is an indexable element of a vector — may not be regarded as an arbitrary element representing information that may be considered as the subject matter of the process. (A part of the base would be the starting address of the vector resting upon it, but physical addresses have no meaning in the context of the program!) We thus have two types of elements: elements upon which the program may freely operate (numbers) and bases. For a number of reasons we impose upon ourselves the restriction that there shall be no mixed vectors: all elements of the same vector must be of the same type. If the elements are numbers, then when necessary we speak of a "twig"; if the elements are bases of other vectors, then we call the vector a "branch". (A base contains, besides length and starting address, also a specification of the type of the vector resting upon it.)

Every context presupposes the accessibility of an "implicit base", namely the base of the vector in which the first index of an index string will be interpreted as a rank number.

Admissible operations on vector bases are: definition of the type of the vector resting upon it; definition of the length of the vector resting upon it.

When an existing vector is lengthened, a number of "undefined" elements are added to it; if it is the lengthening of a branch, then "undefined" bases are thus added. An undefined base contains, in the place of the type specification, the notation "not yet specified". After this introduction of the base, the type of the vector resting upon it may be defined one time. (We thus exclude type changes for existing vectors, even in the case in which the vector at that moment had length 0.) An unspecified vector is something other than an empty vector: although only "nothing" could come out of it, I nevertheless object to additions such as "0 cows plus 0 geese".

When an existing vector is shortened, a number of the highest-numbered elements of this vector are lost; if the vector was a branch, then along with the bases that are removed all vectors are lost which rested upon them (and if there were again branches among those, all vectors which in turn rest upon those, etc.)

(The restriction mentioned in the paragraph before last regarding constancy of type can be circumvented by introducing, as an extra operation on a vector base: the making-unspecified. If something already rested upon this base, that is lost in the making-unspecified business. Another solution is to let this making-unspecified be implied upon type change. Whether all this is very important, I venture to doubt.)

Operations on vector bases take place upon statement of the index string which identifies the base in question; if this index string is empty, then by definition the implicit base is meant. A context will thus in general begin by defining the implicit base as the base of a branch of a certain length. With that an arbitrary number of new bases has been introduced, and with that the possibility has been created of introducing new vectors.

Reference to element i of a vector of length N, where i does not satisfy  0 <= i <= N - 1 , is nonsense and has no meaning: it is therefore the intention that every selection of a vector element implies that a test is made of whether these inequalities are satisfied. (One demands of every program a certain measure of consistency; if this is satisfied, then the usual form of "Memory protect" — to protect the program of Mr. Jansen against the capers of Mr. Pieterse — no longer has any sense at all. For Mr. Pieterse no longer disposes of the terminology with which to hinder Mr. Jansen.)

It is clear how every context in this manner can introduce and play upon its local variables in an extremely flexible manner. Upon the implicit base a little tree is planted, which during its existence can grow and be pruned. But now it becomes high time to see how our context with the local little tree fits into its environment. The implicit base, of which we have spoken, is namely no absolute machine datum: if we take so large a step back that we can survey the entire store of the machine, then what internally — i.e. in a given context — functions as the implicit base is externally an element which has its place in the total tree and, like every other element, is identified by an index string of some length or other.

Moreover: without exception we can assert that every context will not be "self-contained", but can only work by grace of a preceding specification of a number of parameters. A program that uses one of the communication devices will address this communication device as a "formal" device; upon activation a specific device must be supplied as the corresponding actual parameter. This too all falls under "fitting into the environment".

In order to bring about the cooperation of a given context with quantities (vectors, communication devices) which internally have a formal name and externally a name expressed in a terminology that has no meaning internally, I have, after mature deliberation and long hesitation, introduced a new sort of vector, the so-called "parameter vector".

A particular context can only work if it has at its disposal both the implicit base and the associated parameter vector. Because this is an inseparable pair, I propose to accommodate them together in the same vector. (The other vectors, branches and twigs, were homogeneous. We thus now introduce, in the parameter branch, a vector with a very specific layout, e.g. the zeroth element is the implicit base, the first element is the prevailing value of the order counter, and the remaining elements are parameters. It would not surprise me if the beginning of the parameter vector were a very suitable storage place for yet some more implicit "state variables".) In general the parameter vector begins with a fixed number (at least 1) of vector bases, possibly a fixed number of elements for state variables, and finally a number of parameters. If a vector is a parameter vector, then that is stated in its base. In the program the formal parameters are referred to by
(a) an indication that a formal parameter is concerned here; this selects the parameter vector;
(b) an index which specifies the parameter.

The idea is that a given context A activates a context B as follows. Context A reserves a parameter vector, fills in all the parameters in it, and then lets the machine jump over to the new parameter vector together with the new implicit base.

I now want to examine specially for a moment how we supply an existing vector to a new context as a parameter. I want to do this by filling in, at the appropriate place of the parameter vector, a copy of the base of the vector. This is very eerie, but to my feeling I must do this.

Let us go back for a moment to the idea of the implicit base: a particular program is formulated in terms of its implicit base and does not pronounce upon how long the index string is which in the external world fully identifies this implicit base. The selection consequent upon an index string in a specific context would thus mean that you would in thought have to precede this index string by the index string needed to find the implicit base in the environment. That would mean that the efficiency with which a program is executed depends on the distance to the root of the tree, and that was not the intention.

As soon as I copy vector bases, however, I must then make sure that upon alteration of this base — displacement of the vector resting upon it, e.g. — this penetrates not only into the original, unique base, but also into its copies.

The idea of the separate parameter vector has been introduced in order to know, upon alteration of a base, that one need only scan parameter vectors to see whether one encounters elements that must be altered along with it. I believe that this works out faster than recording, at the source upon each copying, where the copy stands, the more so since with a base one can still achieve some economy with a single one. (One can introduce with every base a "copied-bit". The one value means: this base has definitely not been copied; the other value means: this base has possibly been copied. Upon every copying of a base you record this fact with this base. A base can in this manner at a given moment have a number of copies, but one does not count those. If some of these copies disappear through vector shortening, then one lets that disappearance go unnoticed for the time being. (As long as the base is not altered, the possible copying does not matter all that much either.) Only when you are going to displace the vector do you look at the copying bit. If it stands in the position "possibly a copy", then you investigate the possible parameter vectors to see whether you have something to update. If you do indeed find copies, then you update these; otherwise you set the "copied bit" back again.)

As long as the one context only sequentially activates other contexts — only calls the one procedure when the previous one is entirely finished — this activating context can make do with only a single parameter vector. Also for the case in which a particular context wishes to dispose of more than one parameter vector, there is, if I am not mistaken, an acceptable convention to be found, such that in a particular context the parameter vectors are easily found.

When you call a subroutine, you must state therewith which subroutine you are then calling. Likewise I wanted, upon activation of a particular program, to supply this program as a parameter of the activation mechanism. I regard a program as a vector and we agree, e.g., that we copy the base of this vector into the place of the first parameter, before we let the machine pass over to the new parameter vector.

In conclusion a few remarks.

The activating of a new program in parallel programming can be regarded as an operation of which the "ordinary calling of a subroutine" is a special case. You let one of the parameters be a semaphore which, prior to the activation (prior to the call), is set to FALSE by the outermost program. Subsequently the outermost program activates the subroutine and then remains hanging on a passage of this semaphore. The subroutine ends by indicating its completion by setting the semaphore in question to TRUE, so that the calling program only proceeds when the subroutine is completed. After passage the calling program can remove this activation — this waiting machine — again from the master waiting list. I doubt whether this is the most practical way to call subroutines, but it is illuminating that it is essentially nothing other than the adding of a new program to the waiting list.

I have tacitly assumed that the presence of a vector in the cores would imply that its base is also located in the cores. By preference I would keep all branches in the cores, the parameter branches too. Keeping all parameter branches in the cores is pleasant precisely because displacement can imply that you still have to make corrections in parameter branches far from the root. If it is necessary to dump parameter branches onto the drum, then there is incidentally still something to be done about that. Then with every parameter in the branch you must also state which index string in the context that filled the parameter has cited this parameter. Then upon fetching back the parameter string from the drum you must realize that you are now fetching a parameter branch from the drum, that corrections to it may possibly not have been kept up, but you then have all the data in the cores to bring the parameter string, which for a while has "slept" on the drum, up to date again.

In what terminology the waiting programs are to be remembered on the master's waiting list is still an open question for me; the same holds for the programmed semaphores.

I have forgotten to mention that I deliberately supply the program to be executed as a parameter to the activation mechanism, and that I regard such a program as a constant vector. (You can ensure this by not making available, within the program itself, the terminology with which it could overwrite itself. Our ALGOL programs do not overwrite themselves either.) It is namely very attractive to have a program which is output-limited at work in a number of activations simultaneously, without having the program standing in the cores more than once. (What do we think of the translation process? Or the "console program" if you have more than one console?)

The basic thought regarding the core occupation is that a vector is either to be found entirely in the cores, or not at all. That you then note in the base. That in this manner one could not introduce vectors that are longer than the size of the core store is an annoyance which can, however, be solved automatically (by an implicit order increase; that this has been done you can likewise indicate in the base.)

Finally: I have a feeling that the transport times to and from slow store might well become a bottleneck. When you have taken over a vector from the slow store, you can indicate with a bit in the base that the copy is still "intact"; if you write into that vector, or alter its length, then you set that bit into the other position. In that manner you can save yourself quite a lot of needless transporting-back. If the time between request and the beginning of the actual transport is long compared to the actual transport time (because the drum stands in the wrong position), it might well be worth the trouble to collect requested transports for a moment and, if need be, to sort them by program. This may be so much the more necessary because a great many vectors will be short compared to the capacity of a whole drum track.

Closing remarks.

I am only too well aware that in the foregoing no rounded-off plan has been described. On the contrary: this leaves so many question marks that elaboration will be a considerable quantity of work. Before I plunge into it, I should like to have the opinion of some other people as to how feasible or how disastrous this seems to them.

I can imagine that the reader has the feeling of having gotten a peek behind the scenes of someone else's nightmare. The perhaps not convincing defense that I can set against that is that when the ALGOL complex for the X1 began to dawn in its present form, the shivers ran down our backs too for the first three weeks. It may, in retrospect, have been very good that we were confronted with the consequences of that approach only stage by stage; otherwise it would have cost us more effort to summon up the audacious courage. In my not infrequent moments of doubt I do draw courage (or should I say: over-courage?) from that experience.

There is yet another reason why I should now like to lay this plan before others. Of the whole field of problems I have selected a few and thought about how one might solve those. I have only been able to immerse myself in them by assuming, for once, that the rest is indeed solvable. One hopes of course that, in the choice of what one looks at first, one's intuition does not deceive one, but one of course always runs the risk that one unconsciously pushes a few very essential difficulties to the back because one knows no sensible word to say about them.

Hence this piece is concluded with a request for comment.

E.W.Dijkstra

transcribed by Johan E. Mebius
revised Sun, 28 Mar 2010
