---
title: "Notitie over de aansluiting en de programmering van de teleprinter (EWD62)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD62.html
published: "1963-08-23T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD62.PDF
---

# Notitie over de aansluiting en de programmering van de teleprinter

10 juli 1963                                      EWD62
X8 nr. 21.

Notitie over de aansluiting en de programmering van de teleprinter.

1. Hardware

1.1. Hardware voor de teleprinter (dwz. outputorgaan).

De hardware bestaat uit:

een 2-waardige startseinpaal, genaamd "Acflop teleprinter",

een 2-waardige ingreepseinpaal, genaamd "Inflop teleprinter",

een vaste geheugenplaats, die fungeert als buffer met capaciteit van 1 karakter;

het vullen van deze vaste geheugenplaats zal ik in de tekst representeren door

"teleprinter :=....." .

Voor de bediening van het printwerk bestaat dus slechts het minimum: we kunnen geen startopdrachten accumuleren en per startopdracht kunnen we slechts 1 karakter aanbieden. De opmerking is, dat een teleprinter een zo langzaam medium is, dat alles, wat er meer ingebouwd wordt, moeilijk te verdedigen is.

Het aanbieden van een karakter aan de teleprinter geschiedt door de sequens:

"P(Inflop teleprinter);

teleprinter:= karakter;

V(Acflop teleprinter)" .

De P-operatie is wel een aanroep van de Coordinator, de V-operatie kan zonder meer in het programma staan (Nl. Acflop teleprinter:= 1").

1.2. Hardware voor het toetsenbord (dwz. inputorgaan).

Hiervoor zijn twee voorstellen ter sprake geweest; het doel van deze notitie is onder andere om er achter te komen of het ene voorstel preferabel is boven het andere.

1.2.1. Toetsenbord met expliciete "Acflop".

De hardware bestaat uit:

een 2-waardige startseinpaal, genaamd "Acflop toetsenbord",

een 2-waardige ingreepseinpaal, genaamd "Inflop toetsenbord",

een vaste geheugenplaats, die fungeert als buffer met capaciteit van 1 karakter;

het uitlezen van deze geheugenplaats zal ik in het volgende representeren door:

".......:= toetsenbord".

Hier wordt het toetsenbord dus beschouwd als een invoermedium zonder de mogelijkheid van accumulatie van startopdrachten en per startopdracht slechts 1 karakter.

Drukt men op de toets van het toetsenbord, dan stuurt de toetsapparatuur een startpuls, vijf informatiepulsen en een stoppuls naar de "statisizer". De bedoeling van boven gegeven arrangement is, om de statisizer slechts via het toetsenbord te vullen, als op het moment van de startpuls "Acflop toetsenbord = 1" is, terwijl deze vulling zorgt, dat "Acflop toetsenbord := 0" uitgevoerd is, voordat de volgende startpuls zou kunnen komen. Het autonoom transport probeert de inhoud van de statisizer naar de vaste geheugenplaats te transporteren, zodra dit gelukt is, wordt dit door "Inflop toetsenbord := 1" naar de computer gesignaleerd.

Het ruwe schema, dat een karakter van het toetsenbord accepteert bestaat uit

"P(Inflop toetsenbord);

karakter := toetsenbord;

V(Acflop toetsenbord)" .

In het kader van het tandenpoetsen moet de machine dus een keer "V(Acflop toetsenbord)" uitgevoerd hebben om voor deze invoer ontvankelijk te kunnen zijn.

1.2.2. Toetsenbord zonder expliciete "Acflop".

Bij het vullen van de statisizer kan men "Acflop toetsenbord:= 0" uitvoeren bij de afsluitende stoppuls; omdat het autonoom transport practisch instantaan zal plaatsvinden, wordt practisch tegelijkertijd "Inflop toetsenbord:= 1" uitgevoerd.

Omgekeerd: de luistervergunning van de "Inflop toetsenbord" zal slechts = 1 zijn, als er een toetsenbordprogramma daadwerkelijk in zijn operatie

"P(Inflop toetsenbord)"

is blijven hangen.

Als de ingreep effect heeft en dit programma weer in de coordinator in de categorie van de (gedeblokkeerde) onderbroken programma's wordt opgenomen -dwz. als de coordinator beslist, dat dit programma zijn P-operatie voltooid heeft- dan wordt "Inflop toetsenbord:= 0" uitgevoerd (en tevens "luisterbit toetsenbord:= 0", wanneer de wachtketen aan de Inflop toetsenbord hiermee leeg is geworden).

Aannemend, dat dit nu gedeblokkeerde toetsenbordprgramma rap aan de beurt komt -en daar is alles voor te zeggen om er in de coordinator via een prioriteitsregel voor te zorgen, dat dit inderdaad het geval zal zijn- dan zal het tweetal:

"karakter:= toetsenbord;

V(Acflop toetsenbord)"

snel daarna uitgevoerd worden.

De opmerking nu is, dat hieruit volgt, dat deze Acflop en Inflop "bijna altijd" elkaars inverse zullen zijn en dat men zich daarom kan afvragen, of de Acflop toetsenbord -en daarmee de operatie "V(Acflop toetsenbord)"- niet helemaal vervallen kan. We moeten dan wel opnieuw afspreken wat er gebeurt als men in een te hoog tempo op de toetsen drukt.

In het voorstel 1.2.1. had het te snel drukken van de volgende toets geen effect. Nu is het voorstel, om het te snel indrukken van de volgende toets het vorige karakter te laten invalideren. Bv. als volgt.

Het indrukken van een toets heeft altijd ten gevolge, dat een karakter naar de statisizer gezonden wordt. Als dit echter gebeurt op een ogenblik, dat het vorige autonoom transport nog niet heeft plaatsgevonden (kans nihil) of als het autonoom transport plaatsvindt, terwijl "Inflop toetsenbord = 1" is, dan wordt op de geheugenplaats c.n. "toetsenbord" een non-valide karakter aangeboden (bv. numeriek groter dan 31). Hier kan het toetsenbordprogramma dan op testen.

2. Overwegingen bij de vergelijking.

Bij de vergelijking van deze twee oplossingen laat ik de financiele consequentie van een en ander buiten beschouwing: ik ben niet competent om deze te beoordelen en bovendien kon het wel eens niet zo verschrikkelijk veel uitmaken.

Voorts dienen we te bedenken:

a)
dat de kans op "te snel de volgende toets indrukken" naar wij van ganserharte hopen, klein zal zijn;

b)
dat het geen ramp is, als de boodschap in eerste aanleg verkeerd is doorgekomen, daar de operateur toch eerst zal afwachten of de "body" van de boodschap correct is uitgetypt, voordat hij de OK-melding geeft.

Anderzijds dienen we te bedenken:

c)
dat de kans op non-validiteit, omdat het autonoom transport nog niet heeft plaatsgevonden (1.2.2.) heel klein is, maar dat het haastig is op te eisen, dat de reactie op de ingreep onder alle omstandigheden "rap" zal zijn. (We hebben nog nergens essentiele haastsituaties en ik zou me graag de vrijheid voorbehouden, om op gezette tijden -zeg eens in de vijf minuten- de machine een seconde doof te maken voor pak weg herindeling van de informatie op de trommel.) Als dan de bedienende operateur met een kans van 1 op 300 "in de war" zou raken, zou dat toch wel hinderlijk zijn.

d)
dat het gewenst is, dat in de coordinator de behandeling van het toetsenbordprogramma zich in zo weinig mogelijk van die van de andere programma's onderscheidt.

e)
dat het gewenst is, dat de operateur, in twijfel of hij een toets heeft ingedrukt, zo gauw en zo zeker mogelijk weet, waar hij aan toe is.

Ik ga er van uit, dat opnamen van via het toetsenbord ingevoerde karakters en het uitprinten van deze karakters twee karaktersgewijze synchrone processen zullen zijn.

3. Een conventie voor teleprinterreservering.

Als de wachtketting van een hardware seinpaal leeg is -als er geen enkel programma op deze seinpaal wacht- dan zet de coordinator de bijbehorende luisterbit = 0. Deze wordt slechts = 1, als wel een programma op het positief worden van de seinpaal staat te wachten. Dit impliceert, dat ik me ingrepen kan besparen, door zo mogelijk er naar te streven, dat P-operaties op hardware seinpalen zo laat mogelijk gepasseerd worden; is de ingreep op dat ogenblik al binnen, dan kunnen we dat meteen verifieren.

Hieruit volgt als algemene strategie, dat we de passering van een hardware seinpaal in de programma's zo ver mogelijk naar achteren schuiven. Wat aan conventies impliceert, zullen we zo dadelijk zien. (N.B. Ik weet, dat teleprinter-ingrepen zo weinig frequent voorkomen, dat het uitsparen van ingrepen op efficiency-overwegingen hier een zwak argument is; ik wil echter de hele behandeling van de in-en uitvoer-troep zo uniform mogelijk houden, anders verdrinken we in de chaos.)

Het is duidelijk, dat verschillende programma's van de teleprinter gebruik kunnen willen maken; het is kennelijk niet de bedoeling, dat karakters van verschillende bronnen kriskras door elkaar getypt worden: voor het uitprinten van een samenhangende tekst moet een programma de teleprinter dus voor zich reserveren. We voeren hiertoen in de geprogrammeerde seinpaal "teleprinter vrij"; dit is een 2-waardige seinpaal.

Het synchroon met de teleprinter aanbieden van een stuk samenhangende tekst kan nu door twee soorten programma gebeuren -met weglating van alle hier niet irrelevante tellingen-:

type A:
"

L:
P(teleprinter vrij);

P(Inflop teleprinter);

teleprinter:= volgend karakter;

V(Acflop teleprinter);

if B then goto L;

V(teleprinter vrij)"                      of

type B:
"

L:
P(teleprinter vrij);

teleprinter:= volgend karakter;

V(Acflop teleprinter);

P(Inflop teleprinter);

if B then goto L;

V(teleprinter vrij)" .

In het eerste geval wordt de teleprinter vrijgegeven na de laatste tikopdracht, in het tweede geval pas na de terugmelding, dat het volgende karakter alweer getikt kan worden. Het is duidelijk, dat wij slechts een organisatie kunnen opbouwen, mits we hier een duidelijke keuze doen; mijn voorkeur gaat op grond van de eerder gegeven overwegingen uit naar type A. (Deze voorkeur is niet uiterst dwingend; dit is een reden te meer om deze conventie in alle klaarheid en met alle nadruk te formuleren!)

4. Uittikken synchroon met het toetsenbord.

In het volgende veronderstel ik de operatie "P(teleprinter vrij)" uitgevoerd -waarover later meer- zodat niet, terwijl ik aan het "intikken" ben, een machineprogramma me de teleprinter onder de vingers weg kan kapen.

4.1. Met expliciete Acflop voor het toetsenbord.

4.1.1. De meest rigoureuze sluis, die ik me voor kan stellen bestaat uit de volgende cyclus:

"L:
P(Inflop teleprinter);

V(Acflop toetsenbord);

P(Inflop toetsenbord);

karakter:= toetsenbord;

teleprinter:= karakter;

V(Acflop teleprinter);

if B then goto L".

Hier vraagt de machine pas om een volgend karakter van het toetsenbord, als de terugmelding van het typen van het vorige karakter binnen is. Hier weet de operateur drommels goed, waar hij aan toe is, als hij midden in het tikken van een boodschap -doordat iemand binnenkomt of door doofheid van de machine- gestoord wordt. Als de getikte letters van de boodschap alle nog correct zijn, maar hij weet niet precies meer wat hij heeft aangeslagen, dan zijn er drie mogelijkheden. (Ik neem, evenals bij de volgende toetsenbordprogrammaatjes aan, dat zij in doofheid uitgevoerd zullen worden!)

a)
het programma is (in de coordinator) in de eerste P-operatie blijven hangen, hoewel de bijbehorende ingreep al binnen is (de printer staat immers stil);

b)
het programma is in de tweede P-operatie blijven hangen, omdat de ingreep er nog niet is -de operateur moet de volgende toets aanslaan;

c)
het programma is in de tweede P-operatie (in de coordinator) blijven hangen: het voglende karakter is via de statisizer al tot de machine doorgedrongen.

In alle drie de gevallen kan de operateur het volgende karakter aanslaan; in geval a) heeft het geen effect, in geval b) heeft het het gewenste effect en in geval c) heeft het geen effect (niet het ongewenste). Alleen in het geval c) zit er een klein, klein lekje, nl. als de operatie "P(Inflop teleprinter)" voltooid kan worden voordat het karakter goed en wel op het papier staat. Het karakter kan dan twee keer getikt worden.

4.1.2. Een iets minder rigoureuze sluis is:

"L:
P(Inflop toetsenbord, Inflop teleprinter);

karakter:= toetsenbord; teleprinter:= karakter;

V(Acflop teleprinter, Acflop toetsenbord);

if B then goto L".

Hier kan de machine alleen in de P-operatie blijven hangen; als de teleprinter stil staat, is dus "Inflop teleprinter" geen belemmering. Of (a) Inflop toetsenbord is de belemmering of (b) er is geen belemmering, maar de coordinator gunt de machine niet. Als nu weer de tekst voorzover uitgetikt correct is, dan kan de operateur maar een ding doen: het volgende karakter aanslaan en wachten.

In geval (a) is dit normaal het volgende karakter, in geval (b) was dit karakter al binnen en wordt zijn tweede aanslag genegeerd. Voordeel van dit arrangement is, dat 1 keer volgend karakter aanslaan beslist voldoende is. Ook dit sluisje heeft (in geval b) een lekje: vlak voordat hij de toets hernieuwd aanslaat, beeindigt de machine de P-operatie en maakt zij via de V-operatie de statisizer opnieuw ontvankelijk en de letter verschijnt dus twee maal op papier. Het lekje is iets groter, omdat het niet te verkleinen is door de ingreep van het toetsenbord te vertragen. De kans, dat doorgaan van het toetsenbordprogramma en indrukken van de volgende toets niet voldoende coincideren, lijkt me zo klein, dat we dit lekje wel kunnen accepteren. Een voordeel van dit arrangement is, dat in geval van twijfel nog 1 keer de volgende toets indrukken in elk geval op den duur soulaas moet brengen.

Nadeel van dit arrangement is, dat "te snel inslaan" hoegenaamd niet tot de machine doordringt en dat snel inslaan van "a b c" dus "a c" op het papier zou kunnen verschijnen. Ik weeg dit bezwaar niet zo heel erg en wel op grond van de volgende overweging: als "a c" op het papier verschenen is, dan is dat naar voor de operateur, want hij moet het correctieritueel uitvoeren. Als hem dit een paar keer is overkomen, zal hij er verder wel voor oppassen, dat dit hem niet te vaak overkomt: en de gedragslijn, die hij daartoe moet volgen, is duidelijk!

4.2. Zonder expliciete Acflop voor het toetsenbord.

We beschouwen nu de analoge versie van het programma uit 4.1.2.:

"L1:
P(Inflop toetsenbord, Inflop teleprinter);

L2:
karakter:= toetsenbord;

if 31 < karakter then

begin P(Inflop toetsenbord); goto L2 end;

teleprinter:= karakter;

V(Acflop teleprinter);

if B then goto L1".

Wat betreft het te snel aanslaan van de operateur is dit beter. Nemen we weer het te snel aanslaan van "a b c" in die zin, dat b aangeslagen wordt, voordat a correct is verwerkt. In 4.1.2. was de fout van de operateur, dat hij "c" aansloeg, voordat "b" getikt was; in dit geval moet hij, om het nu spaak te laten lopen "c" al aanslaan, voordat de "a" getikt is: de "a" wordt nl. door de te snel gekomen "b" bedorven tot een karakter dat groter dan 31 is.

Laten we nu onderzoeken, hoe precies de operateur weet, waar hij aan toe is, als hij aarzelt wat hij heeft aangeslagen of wat door de machine geweigerd is. Het interessante geval is natuurlijk weer, dat de operateur zich even bedenkt en ziet, dat datgene, wat op papier staat, nog correct is. Het enige, wat hij kan doen is na enige bedenktijd vol goede moed het volgende karakter aanslaan, dat op papier verschijnen moet.

Stond het programma op "Inflop toetsenbord" te wachten, dan was deze actie goed. Nu het geval, dat de volgende karakter al aanwezig was.

Was "Inflop toetsenbord = 1", maar was de machine te lui met haar reactie hierop, dan vernietigt de nieuwe aanslag het al aanwezige volgkarakter en moet dus mettertijd nogmaals op de toets gedrukt worden.

Was inmiddels "Inflop toetsenbord:= 0" voltooid, maar had de coordinator het toetsenbordprogramma nog niet verder laten lopen, dan zal de operateur merken, dat het volgende karakter twee keer getikt wordt.

De kans hierop is te verkleinen, door de coordinator zo te construeren, dat, wanneer de coordinateur de operatie P(Inflop toetsenbord) voltooit, dwz. "Inflop toetsenbord:= 0" uitvoert, het nu toegestane toetsenbordprogramma inderdaad zo snel mogelijk voortgezet wordt. Dit is de prijs, die we betalen voor het dubbele gebruik van "Inflop toetsenbord' en ik vind het niet mooi.

5. Een mengvorm.

Salvo errore et omissione combineert het volgende voorstel de deugden van beide systemen.

Elke keer, dat een toets wordt ingedrukt, wordt een karakter naar de staticiser gezonden; als op het moment van de startpuls de staticiser nog vol mocht zijn, dan neemt de staticiser een non-valide karakter op. (Kans nihil, dit is het zelfde loffelijke perfectionisme als in 1.2.2.)

De inhoud van de staticiser wordt nu in principe naar het geheugen doorgezonden per autonoom woordtransport, afhankelijk echter van de waarde van Acflop toetsenbord en Inflop toetsenbord op dat ogenblik. (De actie van Acflop toetsenbord is dus van het vullen van de staticiser verschoven naar het autonoom transport.)

We onderscheiden nu drie gevallen (het vierde kan zich niet voordoen).

a)
Acflop toetsenbord = 1 en Inflop toetsenbord = 0.

Het karakter wordt zoals het zich in de staticiser bevindt, naar het geheugen doorgezonden; Acflop toetsenbord:= 0; Inflop toetsenbord:= 1);

(Dit als ondeelbare handeling. Dit is het normale geval van karaktertransport.)

b)
Acflop toetsenbord = 0 en Inflop toetsenbord = 1.

Het karakter wordt in "non-valide vorm" in het geheugen neergezet.

(Dit is het geval, dat de tweede toets ingedrukt wordt, terwijl de machine nog niet gereageerd heeft op de ingreep van de vorige. We kunnen dan het vorige karakter nog met "non-valide" overschrijven.)

c)
Acflop toetsenbord = 0 en Inflop toetsenbord = 0.

Het autonoom transport wordt onderdrukt en het karakter raakt "onder tafel".

(Op de ingreep is gereageerd en het is dus te laat, om het vorige karakter nog te herroepen; anderzijds is het tweede karakter wel te vroeg, omdat de machine er nog niet om gevraagd heeft.)

De vierde mogelijkheid komt niet voor, omdat de machine slechts Acflop = 1 zal zetten, als de Inflop = 0 is.

De structuur van het programma is nu als volgt:

"L1:

L2:

P(Inflop toetsenbord, Inflop teleprinter);

karakter:= toetsenbord;

if 31 < karakter then begin V(Acflop toetsenbord); P(Inflop toetsenbord);

goto L2 end;

teleprinter:= karakter;

V(Acflop teleprinter, Acflop toetsenbord);

if B then goto L1".

Wie nu te snel achter elkaar "a b" aanslaat, zal merken, dat er of niets (heel snel) of "a" (iets minder te snel) getypt wordt. In beide gevallen moet hij het eerste ontbrekende karakter aanslaan.

Als nu de operateur aarzelt over wat er gebeurd is, maar de boodschap is, voorzover hij op het papier verschenen is, nog correct, dan kan de operateur na enige bedenktijd alleen maar vol goede moed het ontbrekende karakter (nogmaals?) aanslaan.

De teleprinter staat stil en "Inflop teleprinter" kan dus geen belemmering zijn. We moeten nu drie gevallen onderscheiden.

a)
Als Inflop toetsenbord de belemmering in de P-operatie is, dan wacht het programma op het volgende karakter en het aanslaan ervan is dus correct.

b)
Als Inflop toetsenbord = 1 is, maar de machine is wegens doofheid wat lui met de reactie op deze ingreep, dan maakt de extra aanslag het aanwezige karakter non-valide en is er dus ook niets kapot. (Na verloop van tijd raakt het programma dus in toestand a), dat het karakter nogmaals aangeslagen moet worden.)

c)
Als "Inflop toetsenbord:= 0" door de coordinator voltooid is, maar het toetsenbordprogramma is nog niet aan bod gekomen, dan is het volgende karakter inmiddels onherroepbaar tot het geheugen doorgedrongen. De extra aanslag -de herhaling- deert nu niet, want deze aanslag raakt onder tafel.

Het enige lekje, dat er nu nog in zit is hetzelfde als in het programma 4.1.2. Als de oeprateur een inmiddels ingevoerd (en geaccepteerd) maar nog niet uitgetikt karakter uit ongeduld nog eens aanslaat, precies op het ogenblik, dat de machine de V-operatie uitgevoerd heeft, dan wordt het karakter twee keer ingevoerd. Dit is echter het risico, dat ik geneigd ben om te accepteren.

Aan de problemen van teleprinter-reservering en conventies voor herroepen van ingetikte boodschappen, dan wel OK-melding wilde ik een andere notitie wijden.

PS. Voor de misspelling van "staticiser" in het begin van deze notitie des schrijvers verontschuldigingen.

transcribed by Frank Steggink

revised Sat, 19 Jun 2004

---

## English translation

### Note on the connection and the programming of the teleprinter

10 July 1963                                      EWD62
X8 no. 21.

Note on the connection and the programming of the teleprinter.

1. Hardware

1.1. Hardware for the teleprinter (i.e. output device).

The hardware consists of:

a 2-valued start semaphore, called "Acflop teleprinter",

a 2-valued interrupt semaphore, called "Inflop teleprinter",

a fixed memory location, which functions as a buffer with a capacity of 1 character;

the filling of this fixed memory location I shall represent in the text by

"teleprinter :=....." .

For the operation of the printing work, then, only the minimum exists: we cannot accumulate start commands and per start command we can offer only 1 character. The remark is, that a teleprinter is so slow a medium, that anything more that is built in is hard to defend.

The offering of a character to the teleprinter takes place by the sequence:

"P(Inflop teleprinter);

teleprinter:= karakter;

V(Acflop teleprinter)" .

The P-operation is indeed a call of the Coordinator, the V-operation can stand without further ado in the program (namely Acflop teleprinter:= 1").

1.2. Hardware for the keyboard (i.e. input device).

For this two proposals have come up for discussion; the aim of this note is among other things to find out whether the one proposal is preferable to the other.

1.2.1. Keyboard with explicit "Acflop".

The hardware consists of:

a 2-valued start semaphore, called "Acflop toetsenbord",

a 2-valued interrupt semaphore, called "Inflop toetsenbord",

a fixed memory location, which functions as a buffer with a capacity of 1 character;

the reading out of this memory location I shall represent in the following by:

".......:= toetsenbord".

Here the keyboard is thus regarded as an input medium without the possibility of accumulation of start commands and with only 1 character per start command.

If one presses the key of the keyboard, then the key apparatus sends a start pulse, five information pulses and a stop pulse to the "statisizer". The intention of the arrangement given above is, to fill the statisizer via the keyboard only if at the moment of the start pulse "Acflop toetsenbord = 1", while this filling ensures that "Acflop toetsenbord := 0" has been executed before the next start pulse could come. The autonomous transport attempts to transport the contents of the statisizer to the fixed memory location; as soon as this has succeeded, this is signalled to the computer by "Inflop toetsenbord := 1".

The rough scheme, which accepts a character from the keyboard, consists of

"P(Inflop toetsenbord);

karakter := toetsenbord;

V(Acflop toetsenbord)" .

In the framework of the tooth-brushing the machine must thus once have executed "V(Acflop toetsenbord)" in order to be receptive to this input.

1.2.2. Keyboard without explicit "Acflop".

Upon the filling of the statisizer one can execute "Acflop toetsenbord:= 0" at the closing stop pulse; because the autonomous transport will take place practically instantaneously, "Inflop toetsenbord:= 1" is executed practically simultaneously.

Conversely: the listening permission of the "Inflop toetsenbord" will be = 1 only if a keyboard program has actually got stuck in its operation

"P(Inflop toetsenbord)".

If the interrupt takes effect and this program is again taken up in the coordinator in the category of the (released) interrupted programs -i.e. if the coordinator decides that this program has completed its P-operation- then "Inflop toetsenbord:= 0" is executed (and likewise "listening bit toetsenbord:= 0", when the waiting chain at the Inflop toetsenbord has thereby become empty).

Assuming that this now released keyboard program quickly comes up for its turn -and there is every reason to ensure in the coordinator, via a priority rule, that this will indeed be the case- then the pair:

"karakter:= toetsenbord;

V(Acflop toetsenbord)"

will be executed soon thereafter.

The remark now is, that from this it follows that this Acflop and Inflop will "almost always" be each other's inverse and that one may therefore ask oneself whether the Acflop toetsenbord -and with it the operation "V(Acflop toetsenbord)"- could not be dropped altogether. We must then agree anew what happens if one presses the keys at too high a pace.

In proposal 1.2.1. the too-fast pressing of the next key had no effect. Now the proposal is to let the too-fast pressing of the next key invalidate the previous character. For example as follows.

The pressing of a key always has as its consequence, that a character is sent to the statisizer. If, however, this happens at a moment that the previous autonomous transport has not yet taken place (chance nil) or if the autonomous transport takes place while "Inflop toetsenbord = 1", then at the memory location designated "toetsenbord" a non-valid character is offered (e.g. numerically greater than 31). The keyboard program can then test for this.

2. Considerations in the comparison.

In the comparison of these two solutions I leave the financial consequence of the one and the other out of consideration: I am not competent to judge these and moreover it might well not make so terribly much difference.

Furthermore we ought to bear in mind:

a)
that the chance of "pressing the next key too fast" will, as we wholeheartedly hope, be small;

b)
that it is no disaster if the message has come through wrongly in the first instance, since the operator will after all first wait to see whether the "body" of the message has been typed out correctly before he gives the OK-report.

On the other hand we ought to bear in mind:

c)
that the chance of non-validity, because the autonomous transport has not yet taken place (1.2.2.), is very small, but that it is hasty to demand that the reaction to the interrupt shall under all circumstances be "quick". (We nowhere yet have essential haste situations and I should like to reserve the freedom for myself, at set times -say once in five minutes- to make the machine deaf for a second for, say, a rearrangement of the information on the drum.) If then the operating operator were, with a chance of 1 in 300, to get "muddled", that would after all be rather a nuisance.

d)
that it is desirable that in the coordinator the treatment of the keyboard program differs as little as possible from that of the other programs.

e)
that it is desirable that the operator, in doubt whether he has pressed a key, knows as quickly and as surely as possible where he stands.

I take it as my starting point, that the recording of characters fed in via the keyboard and the printing out of these characters will be two character-wise synchronous processes.

3. A convention for teleprinter reservation.

If the waiting chain of a hardware semaphore is empty -if not a single program is waiting on this semaphore- then the coordinator sets the corresponding listening bit = 0. This becomes = 1 only if a program is indeed waiting for the semaphore to become positive. This implies that I can save myself interrupts by, where possible, striving to have P-operations on hardware semaphores passed as late as possible; if the interrupt is already in at that moment, then we can verify that at once.

From this follows as general strategy, that we push the passing of a hardware semaphore in the programs as far back as possible. What this implies in the way of conventions we shall see in a moment. (N.B. I know that teleprinter interrupts occur so infrequently that the saving of interrupts on efficiency considerations is a weak argument here; I want, however, to keep the whole treatment of the input-and-output mess as uniform as possible, otherwise we drown in chaos.)

It is clear that various programs may want to make use of the teleprinter; it is evidently not the intention that characters from various sources be typed criss-cross through one another: for the printing out of a coherent text a program must therefore reserve the teleprinter for itself. We introduce for this purpose the programmed semaphore "teleprinter vrij"; this is a 2-valued semaphore.

The offering of a piece of coherent text synchronously with the teleprinter can now happen by two sorts of program -with omission of all here not irrelevant counts-:

type A:
"

L:
P(teleprinter vrij);

P(Inflop teleprinter);

teleprinter:= volgend karakter;

V(Acflop teleprinter);

if B then goto L;

V(teleprinter vrij)"                      or

type B:
"

L:
P(teleprinter vrij);

teleprinter:= volgend karakter;

V(Acflop teleprinter);

P(Inflop teleprinter);

if B then goto L;

V(teleprinter vrij)" .

In the first case the teleprinter is released after the last striking command, in the second case only after the report-back that the next character can already be struck again. It is clear that we can build up an organization only provided we make a clear choice here; my preference, on the grounds of the considerations given earlier, goes out to type A. (This preference is not extremely compelling; this is one more reason to formulate this convention with all clarity and with all emphasis!)

4. Typing out synchronously with the keyboard.

In the following I assume the operation "P(teleprinter vrij)" executed -about which more later- so that not, while I am "typing in", a machine program can snatch the teleprinter away from under my fingers.

4.1. With explicit Acflop for the keyboard.

4.1.1. The most rigorous lock that I can imagine consists of the following cycle:

"L:
P(Inflop teleprinter);

V(Acflop toetsenbord);

P(Inflop toetsenbord);

karakter:= toetsenbord;

teleprinter:= karakter;

V(Acflop teleprinter);

if B then goto L".

Here the machine asks for a next character from the keyboard only when the report-back of the typing of the previous character is in. Here the operator knows full well where he stands if he is disturbed in the middle of typing a message -through someone coming in or through deafness of the machine. If the typed letters of the message are all still correct, but he no longer knows exactly what he has struck, then there are three possibilities. (I assume, as with the following little keyboard programs, that they will be executed in deafness!)

a)
the program has got stuck (in the coordinator) in the first P-operation, although the corresponding interrupt is already in (the printer is, after all, standing still);

b)
the program has got stuck in the second P-operation, because the interrupt is not yet there -the operator must strike the next key;

c)
the program has got stuck in the second P-operation (in the coordinator): the next character has already penetrated to the machine via the statisizer.

In all three cases the operator can strike the next character; in case a) it has no effect, in case b) it has the desired effect and in case c) it has no effect (not the undesired one). Only in case c) is there a small, small leak, namely if the operation "P(Inflop teleprinter)" can be completed before the character properly stands on the paper. The character can then be struck twice.

4.1.2. A somewhat less rigorous lock is:

"L:
P(Inflop toetsenbord, Inflop teleprinter);

karakter:= toetsenbord; teleprinter:= karakter;

V(Acflop teleprinter, Acflop toetsenbord);

if B then goto L".

Here the machine can get stuck only in the P-operation; if the teleprinter stands still, then "Inflop teleprinter" is no obstruction. Either (a) Inflop toetsenbord is the obstruction or (b) there is no obstruction, but the coordinator does not grant the machine. If now again the text, insofar as typed out, is correct, then the operator can do only one thing: strike the next character and wait.

In case (a) this is normally the next character, in case (b) this character was already in and its second striking is ignored. The advantage of this arrangement is that striking the next character 1 time is decidedly sufficient. This little lock too has (in case b) a leak: just before he strikes the key anew, the machine ends the P-operation and via the V-operation makes the statisizer receptive again and the letter thus appears twice on paper. The leak is somewhat larger, because it cannot be reduced by delaying the interrupt of the keyboard. The chance that the continuation of the keyboard program and the pressing of the next key do not coincide sufficiently seems to me so small, that we may well accept this leak. An advantage of this arrangement is, that in case of doubt, pressing the next key once more must in any case in the long run bring relief.

A disadvantage of this arrangement is, that "striking too fast" does not penetrate to the machine in any way whatsoever and that the fast striking of "a b c" could therefore cause "a c" to appear on the paper. I do not weigh this objection so very heavily, and indeed on the grounds of the following consideration: if "a c" has appeared on the paper, then that is annoying for the operator, for he has to perform the correction ritual. If this has befallen him a couple of times, he will further take care that this does not befall him too often: and the line of conduct that he must follow to that end is clear!

4.2. Without explicit Acflop for the keyboard.

We now consider the analogous version of the program from 4.1.2.:

"L1:
P(Inflop toetsenbord, Inflop teleprinter);

L2:
karakter:= toetsenbord;

if 31 < karakter then

begin P(Inflop toetsenbord); goto L2 end;

teleprinter:= karakter;

V(Acflop teleprinter);

if B then goto L1".

As regards the operator striking too fast, this is better. Let us again take the too-fast striking of "a b c" in the sense that b is struck before a has been correctly processed. In 4.1.2. the operator's mistake was that he struck "c" before "b" had been typed; in this case he must, in order now to make it go wrong, already strike "c" before the "a" has been typed: the "a" is, namely, by the too-fast-arriving "b", spoiled into a character that is greater than 31.

Let us now investigate how precisely the operator knows where he stands, if he hesitates over what he has struck or what has been refused by the machine. The interesting case is of course again, that the operator thinks for a moment and sees that that which stands on paper is still correct. The only thing he can do is, after some thinking time, in good courage strike the next character, which must appear on paper.

If the program was waiting on "Inflop toetsenbord", then this action was good. Now the case that the next character was already present.

Was "Inflop toetsenbord = 1", but was the machine too lazy with its reaction to this, then the new strike destroys the already present following character and so in time the key must be pressed once more.

Had meanwhile "Inflop toetsenbord:= 0" been completed, but had the coordinator not yet let the keyboard program run on, then the operator will notice that the next character is typed twice.

The chance of this is to be reduced by constructing the coordinator in such a way that, when the coordinator completes the operation P(Inflop toetsenbord), i.e. executes "Inflop toetsenbord:= 0", the now permitted keyboard program is indeed continued as quickly as possible. This is the price that we pay for the double use of "Inflop toetsenbord" and I do not find it pretty.

5. A mixed form.

Salvo errore et omissione the following proposal combines the virtues of both systems.

Each time that a key is pressed, a character is sent to the staticiser; if at the moment of the start pulse the staticiser should still be full, then the staticiser takes up a non-valid character. (Chance nil, this is the same laudable perfectionism as in 1.2.2.)

The contents of the staticiser are now in principle forwarded to the memory per autonomous word transport, dependent however on the value of Acflop toetsenbord and Inflop toetsenbord at that moment. (The action of Acflop toetsenbord has thus shifted from the filling of the staticiser to the autonomous transport.)

We now distinguish three cases (the fourth cannot occur).

a)
Acflop toetsenbord = 1 and Inflop toetsenbord = 0.

The character is forwarded to the memory as it finds itself in the staticiser; Acflop toetsenbord:= 0; Inflop toetsenbord:= 1);

(This as an indivisible action. This is the normal case of character transport.)

b)
Acflop toetsenbord = 0 and Inflop toetsenbord = 1.

The character is put down in the memory in "non-valid form".

(This is the case that the second key is pressed while the machine has not yet reacted to the interrupt of the previous one. We can then still overwrite the previous character with "non-valid".)

c)
Acflop toetsenbord = 0 and Inflop toetsenbord = 0.

The autonomous transport is suppressed and the character goes "under the table".

(The interrupt has been reacted to and it is thus too late to revoke the previous character still; on the other hand the second character is indeed too early, because the machine has not yet asked for it.)

The fourth possibility does not occur, because the machine will set Acflop = 1 only if Inflop = 0.

The structure of the program is now as follows:

"L1:

L2:

P(Inflop toetsenbord, Inflop teleprinter);

karakter:= toetsenbord;

if 31 < karakter then begin V(Acflop toetsenbord); P(Inflop toetsenbord);

goto L2 end;

teleprinter:= karakter;

V(Acflop teleprinter, Acflop toetsenbord);

if B then goto L1".

Whoever now strikes "a b" too fast in succession will notice that either nothing (very fast) or "a" (somewhat less too fast) is typed. In both cases he must strike the first missing character.

If now the operator hesitates over what has happened, but the message, insofar as it has appeared on the paper, is still correct, then the operator can, after some thinking time, only in good courage strike the missing character (once more?).

The teleprinter stands still and "Inflop teleprinter" can thus be no obstruction. We must now distinguish three cases.

a)
If Inflop toetsenbord is the obstruction in the P-operation, then the program is waiting for the next character and the striking of it is thus correct.

b)
If Inflop toetsenbord = 1, but the machine is, on account of deafness, somewhat lazy with the reaction to this interrupt, then the extra strike makes the present character non-valid and so nothing is broken either. (After the passage of time the program thus arrives in state a), that the character must be struck once more.)

c)
If "Inflop toetsenbord:= 0" has been completed by the coordinator, but the keyboard program has not yet come up for its turn, then the next character has meanwhile irrevocably penetrated to the memory. The extra strike -the repetition- now does no harm, for this strike goes under the table.

The only leak that is now still in it is the same as in program 4.1.2. If the operator, out of impatience, strikes once more a character that has meanwhile been fed in (and accepted) but not yet typed out, precisely at the moment that the machine has executed the V-operation, then the character is fed in twice. This is, however, the risk that I am inclined to accept.

To the problems of teleprinter reservation and conventions for the revoking of typed-in messages, or rather the OK-report, I wished to devote another note.

PS. For the misspelling of "staticiser" at the beginning of this note, the writer's apologies.

transcribed by Frank Steggink

revised Sat, 19 Jun 2004
