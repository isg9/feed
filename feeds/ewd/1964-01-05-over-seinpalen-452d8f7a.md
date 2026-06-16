---
title: "Over seinpalen (EWD74)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD74.html
published: "1964-01-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD74.PDF
---

# Over seinpalen

EWD74 -
1

Over seinpalen

Wij beschouwen een aantal onderling "zwak gekoppelde", in zichzelf sequentiële processen. Onder de "zwakke koppeling" versta ik, dat ze op bepaalde punten rekening met elkaar kunnen moeten houden. Als bv. een aantal processen af en toe wel eens van een of andere faciliteit gebruik wil maken, die maar een proces tegelijk kan bedienen, dan betekent dit, dat de processen wel eens even op elkaar kunnen moeten wachten. Als het ene proces informatie verwerkt, dat door een ander proces geleverd moet worden, dan is het ook duidelijk, dat het eerste op het laatste kan moeten wachten. M.a.w. de processen moeten ten opzichte van elkaar in zekere mate gesynchroniseerd kunnen worden.

Om de processen in de gelegenheid te stellen onderling informatie uit te wisselen over elkaars staat van vordering, is er een gemeenschappelijk geheugen ingevoerd. De elementen van dit geheugen zijn non-negatieve integers, die we seinpalen noemen.

In de individuele processen zijn de voor de onderlinge synchronisatie belangrijke punten gemarkeerd, doordat daar is aangegeven, dat bij passering van zo'n punt op bepaalde seinpalen een bepaalde operatie moet worden uitgevoerd. De individuele processen hebben hier de keuze uit twee operaties, de zg. V-operatie en de zg. P-operatie, welke wij hieronder zullen beschrijven.

In het volgende nemen wij aan, dat S1, S2, S3 etc. namen zijn van toegankelijke seinpalen (niet alle seinpalen hoeven in elk proces toegankelijk te zijn!); de V- en de P-operatie zullen wij schrijven als procedure statement.

De V-operatie ("verhoog" ).

De V-operatie heeft betrekking op minstens 1 seinpaal, dus bv. "V(S1)"
of "V(S1, S2, S3)". Als een van de individuele processen de V-operatie
uitvoert, dan is het effect, dat alle er ditmaal bij opgegeven seinpalen in één ondeelbare handeling met 1 worden verhoogd.

Opm. 1 De toevoeging "in één ondeelbare handeling" bedoelt het volgende uit te drukken. Stel, dat de waarde van de seinpaal S1 = 3 is en dat dan bv. twee van de (simultaan werkende!) processen "tegelijkertijd" de operatie V(S1) willen uitvoeren. Tengevolge van de ondeelbaarheid van de handeling mogen we ons dan voorstellen dat deze twee V-operaties op dezelfde seinpaal in een overigens niet ter zake doende volgorde in successie uitgevoerd worden, zodat na afloop S1 = 5 is en niet een van de ophogingen bv. onder tafel is geraakt.

Opm. 2 De V-operatie met meer dan 1 argument is logisch niet
noodzakelijk, maar wel elegant. In de statement "V(S1, S2)" wordt een
simultane verhoging van beide seinpalen gevraagd; vervangen we dit
in een van de individuele processen door "V(S1);V(S2)", dan vragen
we heel expliciet om ophoging in een bepaalde volgorde. Het zou wel
eens wat minder leuk kunnen zijn om daartoe gedwongen te zijn, als
men liever "neutraal" een aantal seinpalen simultaan wil ophogen.

Opm. 3 Als de V-operatie met meer dan 1 argument wel wordt
opgenomen, dan zullen wij ons voorlopig beperken tot het geval, dat de
erbij opgegeven seinpalen verschillend zijn.

De P-operatie ("Prolaag").

In een individueel proces markeert de P-operatie de tentatieve passering van dit punt. De P-operatie heeft betrekking op 1 of meer seinpalen, dus bv. "P(S1)" of "P(S1, S2, S3)". Als de P-operatie in een van de individuele processen geëntameerd is,

EWD74 -
2

dan is daarmee een operatie begonnen, die slechts beeindigd kan worden op een moment, dat alle erbij opgegeven seinpalen positief zijn. Beëindiging van een P-operatie impliceert, dat alle erbij opgegeven seinpalen met 1 verlaagd worden en ook deze beëindiging geldt als één ondeelbare handeling. Ook voor de P-operatie beperken we ons tot
het geval, dat de erbij opgegeven seinpalen onderling verschillend zijn.
Opm. 1 De P-operatie met meer dan 1 argument is logisch wel
noodzakelijk.

Opm. 2 Vele seinpalen nemen slechts de waarden 0 en 1 aan. In dat
geval fungeert de V-operatie als "baanvak vrijgeven"; de P-operatie, de
tentatieve passering, kan slechts voltooid worden, als de betrokken
seinpaal (of seinpalen) op veilig staat en passering impliceert dan
een op onveilig zetten.

Enige voorbeeldjes voor het gebruik van seinpalen.

Voorbeeld 1.     Als wij een klasje machines (alias processen) Xi hebben (dwz. X0, X1, X2, etc...) en in elk proces komt een kritische sectie voor, kritisch in die zin, dat geen twee kritische secties tegelijkertijd onder behandeling mogen zijn, dan kunnen we dit bereiken met een seinpaal, zeg SX, die in dit simpele geval slechts tweewaardig zal zijn;

SX = 0 zal betekenen: een van de machines Xi is aan zijn kritische sectie bezig

SX = 1 zal betekenen: geen van de machines Xi is aan zijn kritische sectie bezig.

De omschrijving van alle processen is nu gelijkluidend:

"LXi: P(SX); TXi; V(SX); rest proces Xi; goto LXi"

Als we alle processen Xi op hun gelabelde punt (dwz. "aan het begin van de regel") zouden starten met de beginwaarde SX = 2, dan zouden we verwezenlijkt hebben, dat ongeacht de omvang van het klasje processen Xi en nooit meer dan twee simultaan aan hun kritische sectie bezig zouden zijn. Dit is kennelijk een generalisatie van de probleemstelling van wederzijdse uitsluiting. (Het is precies de situatie van bv, n tape decks aan twee kanalen.)

Wij vestigen er de aandacht op, dat de formulering van de individuele processen Xi onafhankelijk is van de omvang van het klasje Xi, iets wat met het ook op de dynamische variatie van deze omvang wel hoogst gewenst is. Ook de maximaal toegestane simultaniteit voor de kritische secties dringt niet tot de formulering van de individuele processen door.

Voorbeeld 2. Nu beschouwen we een groepje machines Xi en een groepje machines Yj, elk met hun kritische sectie TXi resp. TYj. Uitvoering van een critische sectie dient uitvoering van alle andere kritische secties uit te sluiten, maar tevens eisen we, dat de uitvoering van een TX-sectie en een TY-sectie alternerende gebeurtenissen zijn.

We kunnen dit bereiken met twee tweewaardige seinpalen, zeg SX en SY.

SX = 1 betekent, dat nu eerst een TX-sectie aan de beurt is.

SY = 1 betekent, dat nu eerst een TY-sectie aan de beurt is.

De programma's voor de machines luiden:

"LXi: P(SX); TXi; V(SY); proces Xi; goto LXi" en

"LYj: P(SY); TYj; V(SX); proces Yj; goto LYj" .

Als de processen alle "aan het begin van de regel" gestart worden moet SX = 1 en SY = 0 zijn of andersom.

EWD74 -
3

Voorbeeld 3. Tenslotte beschouwen we een klasje machines Xi, die informatie eenheden willen lozen in een cyclische buffer met een capaciteit van N informatie eenheden; voorts een klasje machines Yj, die informatie eenheden uit deze buffer willen verwerken.
Omdat vullen van de buffer administratieve maatregelen met "vulwijzer" impliceert - en voor legen mutatis mutandis hetzelfde geldt - eisen we bovendien, dat vullen slechts door 1 machine Xi tegelijkertijd kan geschieden en evenzo, dat legen slechts door 1 machine Yj tegelijkertijd kan geschieden.

We voeren hiervoor in vier seinpalen:

SX1 = aantal vrije plaatsen in de buffer (aanvankelijk = N)

SX = 0 als een van de machines Xi aan het vullen is, anders = 1

SY1 = aantal gevulde plaatsen in de buffer (aanvankelijk = 0)

SY2 = 0 als een van de machines Yj aan het legen is (anders = 1)

Er zal gelden N - 1 < SX1 + SY1 < N.

De machines zijn nu:

"LXi: P(SX1,SX2); vul volgende plaats van de buffer; V(SY1,SX2);.....; goto
LXi"

"LYj: P(SY1,SY2); leeg volgende plaats van de buffer; V(SX1,SY2);.....; goto
LYj"

Hardware voorzieningen.

Onze cyclische processen gaan we nu in twee groepen onderverdelen. Die van de ene groep heten " de concrete machines", die van de andere groep heten " de abstracte machines".

De "concrete machines" zijn de transput-apparaten. Dit zijn alle (cyclische) processen, die met een eigen tijdsbewustzijn gedurende een zekere periode autonoom hun werk doen. Onder "autonoom" verstaan wij in dit verband onafhankelijk van de besturing van de centrale computer. Behalve transporten van en naar de buitenwereld (paper tape, printer, ponskaarten etc.) valt hieronder ook transporteren tussen kerngeheugen enerzijds en trommel of magneetband anderzijds. Het zijn die processen, die, mits eenmaal door de X8 in gang gezet, simultaan met de werkende centrale computer plaats vinden.

De "abstracte machines" zijn de onderling asynchrone programma's. Aangezien de centrale computer de facto steeds maar met 1 programma bezig is, kunnen wij stellen, dat van de abstracte machine er op elk moment hoogstens eentje werkt. (Het is mogelijk, dat geen van deze programma's werkt: de centrale computer "zit dan in de coördinator". De coördinator wordt niet als een van de abstracte machines beschouwd.)

Zolang seinpalen louter door abstracte machines geselecteerd worden, zijn hiervoor geen speciale hardware voorzieningen nodig (behalve doofheid): elke geheugenplaats kan als geprogrammeerde seinpaal fungeren.

Een seinpaal, die louter door de concrete machines geselecteerd wordt (ik laat even in het midden, of die er zullen zijn, het is heel goed denkbaar) is een aangelegenheid, die zich geheel buiten de X8 afspeelt en hoeft ons op het ogenblik dus ook niet te interesseren.

Een seinpaal, die zowel door een concrete als door een abstracte machine geselecteerd moet kunnen worden, vraagt echter onze speciale aandacht. Het is duidelijk dat hiervoor wel speciale hardware voorzieningen en rigoureuze conventies nodig zijn.

In principe is elke "hardware seinpaal" een (per installatie) vaste geheugen-

EWD74 -
4

plaats; door het transput-apparaat kan daar 1 van worden afgetrokken (externe P-operatie) of 1 bij opgeteld worden (externe V-operatie).
Als de centrale computer deze seinpalen moet ophogen of aflagen, dan moet dit gebeuren met behulp van de additieve uitopdracht. In dit geval is nl. interferentie tussen beide additieve operaties (nl. een intern en een externe) niet mogelijk.

Een complicatie is, dat aan elke hardware seinpaal een flip-flop geassocieerd is, die aangeeft of de bijbehorende seinpaal positief is. (Voor de noodzaak van deze flip-flop, zie later)

We kunnen nu twee soorten hardware seinpaal onderscheiden (als regel heeft elk transput apparaat er van beide soorten eentje):

1) Een startopdrachtentelling. In dit geval verricht de centrale computer de V-operatie, en het transputmechanisme verricht de P-operatie. De geassocieerde flip-flop heet in zo'n geval een Acflop.

2) Een ingrepentelling. In dit geval verricht de centrale computer de P-operatie en het transput-apparaat de V-operatie. De geassocieerde flip-flop heet in zo'n geval een Inflop.

In elke installatie zijn de concrete machines genummerd. Uit het nummer is dan af te leiden welke geheugenplaatsen voor startopdrachten- en ingrepentelling gereserveerd zijn; de bijbehorende Acflop en Inflop zijn onder opgaven van het apparaatnummer bereikbaar.

Wij laten nu zien, hoe de centrale computer bij wijziging van een hardware seinpaal de bijbehorende flip-flop zo nodig kan aanpassen. (Aanpassing van zo'n flip-flop ten gevolge van wijziging door de transput-apparatuur is niet de zorg van de centrale computer, maar van de bouwers.

De V-operatie door de centrale computer.

Zij t de integer, die in het kerngeheugen de seinpaal representeert; de geassocieerde flip-flop noemen we Acflop. Het transput-apparaat zal een volgende startopdracht accepteren, als die er is en het er aan toe is. Het verricht in de voltooiing van de P-operatie op deze seinpaal de handeling, die beschreven kan worden door

"if Acflop then begin t := t - 1; if t
< 0 then Acflop := false".

Deze P-operatie, die beschouwd moet worden als onderdeel van het consumeren van de volgende startopdracht, heeft dus als mogelijk gevolg, dat het opnemen van volgende startopdrachten tot nader order niet meer zal plaats vinden; dit nl. als tengevolge van uitputting van de voorraad Acflop op false gezet is.

Additionele spelregels zijn:

a) dat deze handeling vanuit de computer bezien moet worden als
ondeelbare handeling - dit schept zijn voordelen - maar dan anderzijds
de computer niet over demogelijkheid zal beschikken om deze handeling
over een aantal opdrachten te verbieden - en dit schept zijn problemen.

b) dat bij beïnvloeding van t en Acflop door de computer het pertinent verboden is, als hoe kort ook ten onrechte Acflop true zou zijn. Op dat moment zou nl. het transput-apparaat meteen ten onrechte kunnen gaan werken.

Als de centrale computer - ter melding dat de volgende startinformatie is klaar gezet - op deze seinpaal de V-operatie moet uitvoeren, geschiedde dit door het volgende stukje programma:

EWD74 -
5

"t:= t + 1;
if non Acflop then
begin if 0 < t
then Acflop := true end"
De rechtvaardiging van dit stukje programma is als volg. Na de ophoging wordt Acflop getest; als Acflop true was, dan was dus de telling
t positief en kan deze door de ophoging alleen maar positiever geworden
zijn en eventueel aanpassen van Acflop ligt dus tot nader order op de
weg van de transputmachine. Maar als we de Acflop false
aantreffen, dan kan de transputmachine de P-operatie niet meer
uitvoeren, zodat van die kant t en Acflop ongemoeid gelaten worden en de
centrale computer rustig de test kan uitvoeren.

De P-operatie door de centrale computer.

Weer zullen we de seinpaal aanduiden met t; de bijbehorende flip-flop noemen we nu Inflop. Inflop geve aan, of t positief is. De V-operatie, die door de transputmachine uitgevoerd wordt, kan beschreven worden door:

"t := t + 1; if 0 < t then Inflop := true";

van de centrale computer bezien is dit een ondeelbare, niet tegen te houden handeling.

De voltooiing van de P-operatie zal de centrale computer alleen uitvoeren als Inflop true is; bij gratie van doofheid - waarbij
de waarde van Inflop geen effect heeft, zie later - kan de computer zich
permitteren om Inflop tijdelijk false te zetten.

De voltooiing van de P-operatie door de computer bestaat nu uit het volgende stukje programma:

" t := t - 1;

Inflop := false;

if 0 < t then

Inflop := true"

Essentieel is hierbij aangenomen:

a) dat het geen kwaad kan, als Inflop ten onrechte een tijdje false
is

b) dat als transput-machine en computer "tegelijkertijd" proberen om de
assignment "Inflop := true" uit te voeren, dat dan na afloop
zeker Inflop true zal zijn.

De rechtvaardiging van dit stukje programma is nu als volgt. Elke V-operatie voor de assignment "Inflop := false"  uit bovenstaand programmaatje doet niet te zake, omdat daarna in elk geval Inflop false gezet wordt. Als tijdens de assignment "inflop := false" de telling t positief is, dan zal de computer uiteindelijk de statement "inflop := true" uitvoeren. Door tussengeschoven operaties V kan de telling t alleen maar nog groter worden en dus wordt Inflop correct achter gelaten. Als tijdens de assignment "Inflop := false" de telling t daarentegen
non-positief is, moeten we twee gevallen onderscheiden. In het geval,
dat de telling t niet door tussengeschoven V-operaties positief wordt,
dan zullen zowel de transput-machine als de computer de Inflop verder
ongemoeid laten en Inflop blijft dus terecht =false achter. Als door ondergeschoven V-operaties daarentegen de telling positief wordt, dan zal in elk geval door de transput machine op het moment van omslag de assignment "Inflop := true" uitgevoerd worden ( en
mogelijk ten overvloede ook nog een keer door de computer, nl. als ten
tijde van de test de telling t al positief is ). In dit laatste geval
wordt Inflop dus gegarandeerd correct op true achtergelaten.

EWD74 -
6

Opm. De uitleesbaarheid van de Acflops en de tweezijdigheid van de
zetbaarheid van de Inflops zijn in de X8 geintroduceerd speciaal om deze
programmaatjes mogelijk te maken. Deze hardware is in zekere zin de
extra prijs die we betalen moeten voor de ononderdrukbaarheid van de
autonome operaties op de seinpalen.
De hardware voor de ingreep.

In het volgende zullen we Inflop true ook door = 1, en false door = 0  representeren.

Zoals gezegd zijn de Inflops genummerd. In een installatie met minder dan 27 Inflops zijn deze gezamenlijk als bits van het zg. (1ste) Inflopwoord uitleesbaar met behulp van een speciale communicatieopdracht. (Komen er meer Inflops, dan wordt een 2de Inflopwoord geintroduceerd. Etc etc. Ik verwacht dat er per Inflopwoord hoogstens 26 bits gebruikt zullen worden en dat het tekenbit van een inflopwoord altijd = 0 is met het oog op normering.)

Opm. Welke positie de verschillende Inflops in het Inflopwoord innemen is mij niet bekend: hier kon nog wel eens een beslissing over genomen moeten worden.

Bij elke Inflop hoort een luisterbit; ook dit is een flip-flop, die door de X8 individueel in beide richtingen zetbaar is. Ook de luisterbits zijn via een (of zo nodig meer) zg. "luisterwoord" uitleesbaar. Een luisterbit neemt in het luisterwoord dezelfde positie in als de overeenkomstige Inflop in het Inflopwoord.

Tenslotte is er een flip-flop, die aangeeft of de X8 "doof" dan wel "horend" is. Een ingreep vindt plaats als

a) het collatieresultaat van Inflopwoord(en) en luisterwoord(en) niet
enkel nullen bevat

b) en tevens de machine horend is.

Opm. De doofmakende opdrachten van de X8 werken "instantaan"; het is dus niet mogelijk, dat na de opdracht "maak doof" nog net een interruptie plaats vindt (zoals dit bij de X1 het geval was).

Pro memorie: Dit moet gecontroleerd worden voor de herstellende sprong, die de overgang horend → door kan bewerkstelligen; nagevraagd moet worden of ook de horend makende opdrachten instantaan werken.

De ingreep bestaat daaruit dat:

a) er een speciale subroutinesprong wordt uitgevoerd (met een voor dit
doel gereserveerde labda)

b) in de uitvoering van deze subroutinesprong de normale ophoging van de opdrachtteller onderdrukt wordt en de machine onmiddellijk doof gemaakt wordt.

De ingreepsprong verwijst de besturing naar de "ingreeproutine"; deze begint met registerinhouden e.d. te redden om te zorgen dat het nu onderbroken programma later voortgezet kan worden alsof er niets gebeurd is. Vervolgens zal het de Inflops moeten analyseren om te ontdekken welk extern apparaat zo nodig iets te melden had en wat, etc.

De coördinator.

Een belangrijk onderdeel van de coördinator is de ingreeproutine. Ik verwacht dat de ingreeproutine zozeer met de coördinator verweven zal zijn, dat een gescheiden behandeling nauwelijks mogelijk is.

EWD74 -
7

De coördinator wachtlijst bevat alle programma in de machine, onderverdeeld in twee categorieën, de geblokkeerden en de uitvoerbaren.
Geblokkeerde programmaas zijn programmaas die niet verder kunnen, doordat ze op een seinpaal staan te wachten, op de voltooiing van een voor hun voortzetting noodzakelijke gebeurtenis.

Uitvoerbare programmaas zijn programmaas die wel verder kunnen ware het niet dat de X8 maar 1 van hen verder kan helpen. (Ik neem aan dat het programma dat door de X8 uitgevoerd wordt onder de uitvoerbare gerangschikt blijft: de term "wachtlijst" is dan wat merkwaardig, maar laten we er maar in berusten. Als de besturing echt in de coördinator zit, dan is de term in orde.)

Het effect van de P-operatie kan zijn, dat een programma van uitvoerbaar nu geblokkeerd wordt, het effect van een V-operatie kan zijn, dat een (ander) programma uit de groep der geblokkeerde naar de uitvoerbare wordt overgeheveld. Een programma, dat in actie door een interruptie onderbroken wordt, blijft gerangschikt onder de uitvoerbare!

We kunnen het ook zo zien: als in een programma een P-operatie geïnitieerd wordt, dan was op dat moment het programma in actie, dus uitvoerbaar. Nu zijn er twee gevallen. Of de heersende waarden van de betroffen seinpalen vormen geen beletsel, of zij doen dit wel. In het eerste geval worden zij afgelaagd, de P-operatie is daarmee voltooid en het programma blijft uitvoerbaar. (of het ook in actie blijft is een heel ander chapiter!) In het tweede geval wordt het programma uit de groep der uitvoerbaren gehaald. De groep der geblokkeerde bevat dus alleen alle programmaas die in een geïnitieerde maar niet voltooide P-operatie zijn blijven hangen.

De P-operaties van de abstracte machines worden dus beslist niet als een permanent observeren van de betrokken seinpalen gespeeld, in tegendeel: de abstracte machines, die geblokkeerd zijn, sluimeren. Nu de abstracte machines, eenmaal geblokkeerd zijnde, niet meer gevoelig zijn voor het positief zijn van seinpalen, moet de coördinator gevoelig zijn voor het positief worden van seinpalen. Maw. als 1 of meer seinpalen positief worden, heeft de coördinator de plicht om vast te stellen, of er nu geblokkeerde uit hun blokkade geholpen kunnen worden en zo ja, dan dient de coördinator dit te doen. (Bij de V-operatie met meer dan 1 argument kunnen er dus twee abstracte machines naar de uitvoerbare overgeheveld moeten worden) Hier ligt een vrij stringente plicht voor de coördinator: er moet immers vermeden worden, dat een abstracte machine geblokkeerd is door een geprogrammeerde seinpaal, die inmiddels "onopgemerkt" positief is geworden. De abstracte machine kijkt niet meer, de seinpaal seint niet meer en de boel zit vast!

Het is om bovenstaande redenen, dat wij voor de V-operatie, die tot nog toe als ophoging beschreven is, beslist niet zullen volstaan met een simpele additieve uitopdracht. De geprogrammeerde V-operatie wordt een aanroep van een coördinatorroutine: deze zal de meegegeven seinpalen verhogen en tevens kijken, wat de aan deze verhoging te verbinden consequenties zijn!

Hier zien we inmiddels de verweving van de ingreeproutine en de coördinator. De ingreeproutine sec doet niet veel meer, dan vaststellen op welke (hardware)  seinpaal de V-operatie is uitgevoerd en ook dit zal in de regel tot gevolg hebben, dat een abstracte machine gedeblokkeerd wordt.

De geblokkeerde programmaas bevinden zich in de coördinatorwachtlijst onder opgave van de seinpalen, die bij de voltooiing de P-operatie afgelaagd moeten worden.

EWD74 -
8

Ik stel mij voor, dat de onderverdeling van de wachtlijst geschieden zal niet door verplaatsing maar door kettingrijgen. Ik neem aan dat elke abstracte machine bij creatie een nummer toegekend krijgt - het laagste vrije nummer - en dit nummer gedurende zijn bestaan zal blijven behouden. Ik neem voorts aan, dat onder dit nummer in de wachtlijst direct geselecteerd kan worden.
Ik wilde alle uitvoerbare machines met een ketting aan elkaar rijgen, verder wilde ik bij elke non-positieve seinpaal de beginschakel van een mogelijk niet lege blokkade ketting maken.

Als nu een van de abstracte machines de P-operatie wil uitvoeren, dan worden de meegegeven seinpalen onderzocht. Als een van de seinpalen non-positief bevonden wordt, dan worden de andere seinpalen helemaal niet meer bekeken, want deze abstracte machine wordt meteen aan de blokkadeketting van deze seinpaal gehangen. De opmerking is nl dat voordat deze seinpaal positief is, de P-operatie in elk geval niet voor voltooiing in aanmerking komt.

Als nu een V-operatie op een seinpaal wordt uitgevoerd, dan kan de coördinator nu op vrij snelle wijze de mogelijke consequenties hiervan nagaan. Was de blokkade ketting leeg, dan stond er niets op deze V-operatie te wachten en geen van de geblokkeerde machines komt dus voor deblokkade in aanmerking. Als de blokkadeketting van deze seinpaal niet leeg is, dan onderzoekt men een abstracte machine in de ketting: als de overige seinpalen geen beletsel vormen, dan kan een P-operatie voltooid worden en gaat deze abstracte machine over naar de uitvoerbare. Als een andere seinpaal wel een beletsel vormt, dan gaat de abstracte machine over naar de blokkade ketting van die laatste seinpaal. Repetatur, totdat of de seinpaal weer nul is (succesvolle deblokkade) of de ketting leeg is (of beide). Het is vanzelfsprekend dat door het dusdanig afwerken van de V-operatie het uitgesloten is dat een positieve seinpaal met non-lege blokkadeketting zou overblijven.

Salvo errore et omissione zou de coördinator naar aanleiding van een V-operatie dankzij deze kettingen nu vrij snel moeten kunnen vaststellen of hieraan verdere consequenties verbonden zijn en zo ja, welke.

In dit schema passen de luisterbits voortreffelijk: men maakt de luisterbit van een hardware seinpaal = 1 als zijn blokkadeketting gevuld is, anders = 0. Immers: als de blokkadeketting leeg is, dan staat er in eerste instantie niets op deze seinpaal te wachten. Wat zullen we dan een ingreep introduceren, als deze seinpaal positief wordt? Zo komen de luisterbits naar voren als middel om onnodige ingrepen te onderdrukken. Of dit veel zoden aan de dijk zet, valt nog ernstig te bezien, het zou best eens zo kunnen zijn, dat er haast altijd op een ingreep gewacht wordt.

Transcription by Patrick Tingen.

Last revised on

6 November, 2019

---

## English translation

### On semaphores

EWD74 -
1

On semaphores

We consider a number of mutually "weakly coupled", in themselves sequential processes. By the "weak coupling" I understand that at certain points they may have to take account of one another. If, for example, a number of processes now and then wishes to make use of some facility or other that can serve only one process at a time, then this means that the processes may now and then have to wait a moment for one another. If the one process processes information that has to be supplied by another process, then it is likewise clear that the first may have to wait for the latter. In other words, the processes must be capable of being synchronized to a certain degree with respect to one another.

In order to put the processes in a position to exchange information among themselves about each other's state of progress, a common store has been introduced. The elements of this store are non-negative integers, which we call semaphores.

In the individual processes the points that are important for the mutual synchronization have been marked, in that it is there indicated that on passing such a point a certain operation must be carried out on certain semaphores. The individual processes here have the choice between two operations, the so-called V-operation and the so-called P-operation, which we shall describe below.

In what follows we assume that S1, S2, S3 etc. are names of accessible semaphores (not all semaphores need be accessible in every process!); the V- and the P-operation we shall write as a procedure statement.

The V-operation ("verhoog" [increase]).

The V-operation pertains to at least 1 semaphore, thus for example "V(S1)"
or "V(S1, S2, S3)". If one of the individual processes carries out the V-operation,
then the effect is that all the semaphores specified with it on this occasion are increased by 1 in one indivisible action.

Rem. 1 The addition "in one indivisible action" intends to express the following. Suppose that the value of the semaphore S1 = 3 and that then, for example, two of the (simultaneously operating!) processes wish to carry out the operation V(S1) "at the same time". As a consequence of the indivisibility of the action we may then picture to ourselves that these two V-operations on the same semaphore are carried out in succession in an order that is, for that matter, immaterial, so that afterwards S1 = 5 and not one of the increments has, for example, gone under the table.

Rem. 2 The V-operation with more than 1 argument is logically not
necessary, but it is elegant. In the statement "V(S1, S2)" a
simultaneous increase of both semaphores is requested; if we replace this
in one of the individual processes by "V(S1);V(S2)", then we ask
very explicitly for increase in a particular order. It might
be somewhat less pleasant to be forced to that, if
one would rather "neutrally" increase a number of semaphores simultaneously.

Rem. 3 If the V-operation with more than 1 argument is indeed
incorporated, then we shall for the time being restrict ourselves to the case in which the
semaphores specified with it are distinct.

The P-operation ("Prolaag" [decrease]).

In an individual process the P-operation marks the tentative passing of this point. The P-operation pertains to 1 or more semaphores, thus for example "P(S1)" or "P(S1, S2, S3)". When the P-operation has been initiated in one of the individual processes,

EWD74 -
2

then an operation has thereby been begun that can only be terminated at a moment when all the semaphores specified with it are positive. Termination of a P-operation implies that all the semaphores specified with it are decreased by 1, and this termination too counts as one indivisible action. For the P-operation as well we restrict ourselves to
the case in which the semaphores specified with it are mutually distinct.
Rem. 1 The P-operation with more than 1 argument is indeed logically
necessary.

Rem. 2 Many semaphores take only the values 0 and 1. In that
case the V-operation functions as "release section of track"; the P-operation, the
tentative passing, can only be completed if the semaphore (or semaphores) concerned
stands at safe, and passing then implies
a setting to unsafe.

Some little examples for the use of semaphores.

Example 1.     If we have a little class of machines (alias processes) Xi (i.e. X0, X1, X2, etc...) and in each process there occurs a critical section, critical in the sense that no two critical sections may be under treatment at the same time, then we can achieve this with a semaphore, say SX, which in this simple case will be only two-valued;

SX = 0 will mean: one of the machines Xi is engaged in its critical section

SX = 1 will mean: none of the machines Xi is engaged in its critical section.

The description of all the processes is now identical in wording:

"LXi: P(SX); TXi; V(SX); rest proces Xi; goto LXi"

If we were to start all the processes Xi at their labelled point (i.e. "at the beginning of the line") with the initial value SX = 2, then we would have realized that, regardless of the size of the little class of processes Xi, never more than two would be engaged in their critical section simultaneously. This is evidently a generalization of the problem statement of mutual exclusion. (It is precisely the situation of, e.g., n tape decks on two channels.)

We draw attention to the fact that the formulation of the individual processes Xi is independent of the size of the little class Xi, something which, with a view to the dynamic variation of this size, is indeed highly desirable. Nor does the maximally permitted simultaneity for the critical sections penetrate into the formulation of the individual processes.

Example 2. Now we consider a little group of machines Xi and a little group of machines Yj, each with their critical section TXi and TYj respectively. Execution of a critical section is to exclude execution of all other critical sections, but at the same time we require that the execution of a TX-section and a TY-section be alternating events.

We can achieve this with two two-valued semaphores, say SX and SY.

SX = 1 means that now it is first a TX-section's turn.

SY = 1 means that now it is first a TY-section's turn.

The programs for the machines read:

"LXi: P(SX); TXi; V(SY); proces Xi; goto LXi" and

"LYj: P(SY); TYj; V(SX); proces Yj; goto LYj" .

If the processes are all started "at the beginning of the line" then SX = 1 and SY = 0 must hold, or the other way round.

EWD74 -
3

Example 3. Finally we consider a little class of machines Xi which wish to dump information units into a cyclic buffer with a capacity of N information units; furthermore a little class of machines Yj which wish to process information units out of this buffer.
Because filling of the buffer implies administrative measures with a "fill pointer" - and for emptying mutatis mutandis the same holds - we moreover require that filling can be done only by 1 machine Xi at a time, and likewise that emptying can be done only by 1 machine Yj at a time.

For this we introduce four semaphores:

SX1 = number of free places in the buffer (initially = N)

SX = 0 if one of the machines Xi is filling, otherwise = 1

SY1 = number of filled places in the buffer (initially = 0)

SY2 = 0 if one of the machines Yj is emptying (otherwise = 1)

It will hold that N - 1 < SX1 + SY1 < N.

The machines are now:

"LXi: P(SX1,SX2); vul volgende plaats van de buffer; V(SY1,SX2);.....; goto
LXi"

"LYj: P(SY1,SY2); leeg volgende plaats van de buffer; V(SX1,SY2);.....; goto
LYj"

Hardware provisions.

We are now going to subdivide our cyclic processes into two groups. Those of the one group are called "the concrete machines", those of the other group are called "the abstract machines".

The "concrete machines" are the transput devices. These are all (cyclic) processes which, with a time-awareness of their own, do their work autonomously during a certain period. By "autonomous" we understand in this connection independent of the control of the central computer. Besides transports to and from the outside world (paper tape, printer, punched cards etc.) this also includes transporting between core store on the one hand and drum or magnetic tape on the other. They are those processes which, once set going by the X8, take place simultaneously with the working central computer.

The "abstract machines" are the mutually asynchronous programs. Since the central computer is de facto always busy with only 1 program, we may state that of the abstract machines at every moment at most one is working. (It is possible that none of these programs is working: the central computer "is then in the coordinator". The coordinator is not regarded as one of the abstract machines.)

As long as semaphores are selected purely by abstract machines, no special hardware provisions are needed for this (apart from deafness): every store location can function as a programmed semaphore.

A semaphore that is selected purely by the concrete machines (I leave it open for the moment whether there will be such, it is quite conceivable) is a matter that takes place entirely outside the X8 and therefore need not interest us at the moment either.

A semaphore that must be capable of being selected both by a concrete and by an abstract machine does, however, demand our special attention. It is clear that for this special hardware provisions and rigorous conventions are indeed needed.

In principle every "hardware semaphore" is a (per installation) fixed store

EWD74 -
4

location; by the transput device 1 can be subtracted there (external P-operation) or 1 added (external V-operation).
If the central computer has to increase or decrease these semaphores, then this must be done by means of the additive out-instruction. For in this case interference between the two additive operations (namely an internal and an external one) is not possible.

A complication is that with every hardware semaphore there is associated a flip-flop which indicates whether the corresponding semaphore is positive. (For the necessity of this flip-flop, see later)

We can now distinguish two kinds of hardware semaphore (as a rule every transput device has one of both kinds):

1) A start-instruction count. In this case the central computer performs the V-operation, and the transput mechanism performs the P-operation. The associated flip-flop is in such a case called an Acflop.

2) An intervention count. In this case the central computer performs the P-operation and the transput device the V-operation. The associated flip-flop is in such a case called an Inflop.

In every installation the concrete machines are numbered. From the number it can then be derived which store locations are reserved for start-instruction and intervention counting; the corresponding Acflop and Inflop are accessible upon specification of the device number.

We now show how the central computer can, on changing a hardware semaphore, adapt the corresponding flip-flop if need be. (Adaptation of such a flip-flop as a consequence of change by the transput equipment is not the concern of the central computer, but of the builders.

The V-operation by the central computer.

Let t be the integer that represents the semaphore in the core store; the associated flip-flop we call Acflop. The transput device will accept a next start-instruction, if there is one and it is ready for it. In the completion of the P-operation on this semaphore it performs the action that can be described by

"if Acflop then begin t := t - 1; if t
< 0 then Acflop := false".

This P-operation, which must be regarded as part of the consuming of the next start-instruction, thus has as a possible consequence that the taking-in of subsequent start-instructions will not take place anymore until further notice; this namely if, as a consequence of exhaustion of the supply, Acflop has been set to false.

Additional rules of the game are:

a) that this action, viewed from the computer, must be regarded as an
indivisible action - this creates its advantages - but then on the other hand
the computer will not have at its disposal the possibility of forbidding this action
over a number of instructions - and this creates its problems.

b) that in influencing t and Acflop by the computer it is categorically forbidden that, however briefly, Acflop should wrongly be true. For at that moment the transput device could immediately and wrongly begin to work.

If the central computer - to report that the next start information has been made ready - has to carry out the V-operation on this semaphore, this happened by means of the following little piece of program:

EWD74 -
5

"t:= t + 1;
if non Acflop then
begin if 0 < t
then Acflop := true end"
The justification of this little piece of program is as follows. After the increase Acflop is tested; if Acflop was true, then the count
t was thus positive and can by the increase only have become more
positive, and possible adaptation of Acflop therefore lies until further notice on the
path of the transput machine. But if we find Acflop false,
then the transput machine can no longer carry out the P-operation, so that from that side t and Acflop are left undisturbed and the
central computer can calmly carry out the test.

The P-operation by the central computer.

Again we shall denote the semaphore by t; the corresponding flip-flop we now call Inflop. Let Inflop indicate whether t is positive. The V-operation, which is carried out by the transput machine, can be described by:

"t := t + 1; if 0 < t then Inflop := true";

viewed from the central computer this is an indivisible, unstoppable action.

The completion of the P-operation the central computer will carry out only if Inflop is true; by grace of deafness - in which
the value of Inflop has no effect, see later - the computer can
permit itself to set Inflop false temporarily.

The completion of the P-operation by the computer now consists of the following little piece of program:

" t := t - 1;

Inflop := false;

if 0 < t then

Inflop := true"

It is essentially assumed here:

a) that it can do no harm if Inflop is wrongly false for a while

b) that if the transput machine and the computer try "at the same time" to carry out the
assignment "Inflop := true", then afterwards
Inflop will certainly be true.

The justification of this little piece of program is now as follows. Every V-operation prior to the assignment "Inflop := false" out of the above little program is immaterial, because thereafter Inflop is set false in any case. If during the assignment "inflop := false" the count t is positive, then the computer will eventually carry out the statement "inflop := true". By interposed operations V the count t can only become still larger, and thus Inflop is left behind correctly. If, on the other hand, during the assignment "Inflop := false" the count t is
non-positive, we must distinguish two cases. In the case
in which the count t does not become positive through interposed V-operations,
then both the transput machine and the computer will leave Inflop further
undisturbed, and Inflop thus remains behind, correctly, = false. If, on the other hand, through underslid V-operations the count does become positive, then at any rate the transput machine will at the moment of the change carry out the assignment "Inflop := true" ( and
possibly, superfluously, also once more by the computer, namely if at
the time of the test the count t is already positive ). In this latter case
Inflop is thus guaranteed to be left behind correctly at true.

EWD74 -
6

Rem. The readability of the Acflops and the two-sidedness of the
settability of the Inflops have been introduced in the X8 especially to make these
little programs possible. This hardware is in a certain sense the
extra price we have to pay for the unsuppressibility of the
autonomous operations on the semaphores.
The hardware for the intervention.

In what follows we shall also represent Inflop true by = 1, and false by = 0.

As stated, the Inflops are numbered. In an installation with fewer than 27 Inflops these are jointly readable as bits of the so-called (1st) Inflop word by means of a special communication instruction. (If there come to be more Inflops, then a 2nd Inflop word is introduced. Etc. etc. I expect that per Inflop word at most 26 bits will be used and that the sign bit of an Inflop word is always = 0 with a view to normalization.)

Rem. Which position the various Inflops take in the Inflop word is not known to me: here a decision might still have to be taken.

With each Inflop there belongs a listen bit; this too is a flip-flop, which can be set by the X8 individually in both directions. The listen bits too are readable via a (or if need be several) so-called "listen word". A listen bit takes in the listen word the same position as the corresponding Inflop in the Inflop word.

Finally there is a flip-flop that indicates whether the X8 is "deaf" or "hearing". An intervention takes place if

a) the collation result of Inflop word(s) and listen word(s) does not
contain only zeros

b) and at the same time the machine is hearing.

Rem. The deaf-making instructions of the X8 work "instantaneously"; it is thus not possible that after the instruction "make deaf" an interruption should still just take place (as was the case with the X1).

Pro memoria: This must be checked for the restoring jump that can bring about the transition hearing → deaf; it must be inquired whether the hearing-making instructions too work instantaneously.

The intervention consists in the fact that:

a) a special subroutine jump is carried out (with a labda reserved for this
purpose)

b) in the execution of this subroutine jump the normal increase of the instruction counter is suppressed and the machine is immediately made deaf.

The intervention jump refers the control to the "intervention routine"; this begins by saving register contents and the like in order to ensure that the now interrupted program can later be continued as if nothing had happened. Subsequently it will have to analyse the Inflops in order to discover which external device, if any, had something to report and what, etc.

The coordinator.

An important part of the coordinator is the intervention routine. I expect that the intervention routine will be so interwoven with the coordinator that a separate treatment is hardly possible.

EWD74 -
7

The coordinator waiting list contains all the programs in the machine, subdivided into two categories, the blocked ones and the executable ones.
Blocked programs are programs that cannot proceed because they are waiting on a semaphore, for the completion of an event necessary for their continuation.

Executable programs are programs that could indeed proceed were it not that the X8 can help only 1 of them along. (I assume that the program that is being executed by the X8 remains classified among the executable ones: the term "waiting list" is then somewhat peculiar, but let us resign ourselves to it. If the control really is in the coordinator, then the term is in order.)

The effect of the P-operation may be that a program is now blocked, from being executable; the effect of a V-operation may be that an (other) program is transferred from the group of the blocked ones to the executable ones. A program that, while in action, is interrupted by an interruption remains classified among the executable ones!

We can also view it thus: if in a program a P-operation is initiated, then at that moment the program was in action, hence executable. Now there are two cases. Either the prevailing values of the semaphores concerned form no obstacle, or they do. In the first case they are decreased, the P-operation is thereby completed and the program remains executable. (whether it also remains in action is quite another chapter!) In the second case the program is taken out of the group of the executable ones. The group of the blocked ones thus contains only all those programs that have been left hanging in an initiated but not completed P-operation.

The P-operations of the abstract machines are thus by no means played out as a permanent observing of the semaphores concerned, on the contrary: the abstract machines that are blocked slumber. Now that the abstract machines, once being blocked, are no longer sensitive to the becoming positive of semaphores, the coordinator must be sensitive to the becoming positive of semaphores. In other words, if 1 or more semaphores become positive, the coordinator has the duty of establishing whether blocked ones can now be helped out of their blockade, and if so, then the coordinator is to do this. (With the V-operation with more than 1 argument it may thus be the case that two abstract machines have to be transferred to the executable ones.) Here lies a fairly stringent duty for the coordinator: for it must be avoided that an abstract machine is blocked by a programmed semaphore which has meanwhile "unnoticed" become positive. The abstract machine no longer looks, the semaphore no longer signals, and the whole thing is stuck!

It is for the above reasons that for the V-operation, which up to now has been described as an increase, we shall by no means make do with a simple additive out-instruction. The programmed V-operation becomes a call of a coordinator routine: this will increase the semaphores passed along and also look at what the consequences to be attached to this increase are!

Here we see meanwhile the interweaving of the intervention routine and the coordinator. The intervention routine as such does not do much more than establish on which (hardware) semaphore the V-operation has been carried out, and this too will as a rule have as a consequence that an abstract machine is unblocked.

The blocked programs are located in the coordinator waiting list, with specification of the semaphores which on completion of the P-operation must be decreased.

EWD74 -
8

I imagine that the subdivision of the waiting list will take place not by relocation but by chain-threading. I assume that every abstract machine is allotted a number upon creation - the lowest free number - and will retain this number throughout its existence. I assume furthermore that under this number selection can be made directly in the waiting list.
I wanted to thread all the executable machines together with a chain; furthermore I wanted to make, at each non-positive semaphore, the initial link of a possibly non-empty blockade chain.

If now one of the abstract machines wishes to carry out the P-operation, then the semaphores passed along are examined. If one of the semaphores is found non-positive, then the other semaphores are not examined at all anymore, for this abstract machine is immediately hung onto the blockade chain of this semaphore. The remark is namely that before this semaphore is positive, the P-operation is at any rate not eligible for completion.

If now a V-operation is carried out on a semaphore, then the coordinator can now in a fairly quick manner go through the possible consequences of this. If the blockade chain was empty, then nothing was waiting on this V-operation and none of the blocked machines is therefore eligible for unblocking. If the blockade chain of this semaphore is not empty, then one examines an abstract machine in the chain: if the remaining semaphores form no obstacle, then a P-operation can be completed and this abstract machine passes over to the executable ones. If another semaphore does form an obstacle, then the abstract machine passes over to the blockade chain of that latter semaphore. Repetatur, until either the semaphore is again zero (successful unblocking) or the chain is empty (or both). It goes without saying that by thus disposing of the V-operation it is excluded that a positive semaphore with a non-empty blockade chain should remain.

Salvo errore et omissione the coordinator ought, in response to a V-operation, thanks to these chains, now to be able to establish fairly quickly whether there are further consequences attached to it, and if so, which.

Into this scheme the listen bits fit splendidly: one makes the listen bit of a hardware semaphore = 1 if its blockade chain is filled, otherwise = 0. For indeed: if the blockade chain is empty, then in the first instance nothing is waiting on this semaphore. Why should we then introduce an intervention if this semaphore becomes positive? Thus the listen bits come to the fore as a means of suppressing unnecessary interventions. Whether this is of much avail remains seriously to be seen; it might well be the case that almost always there is waiting on an intervention.

Transcription by Patrick Tingen.

Last revised on

6 November, 2019
