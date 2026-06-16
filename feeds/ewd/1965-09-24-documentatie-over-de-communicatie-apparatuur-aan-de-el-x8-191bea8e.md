---
title: "Documentatie over de communicatie apparatuur aan de EL X8 (EWD140)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD140.html
published: "1965-09-24T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD140.PDF
---

# Documentatie over de communicatie apparatuur aan de EL X8

Documentatie over communicatie apparatuur aan de EL X8

0. Inleiding

Er
is voorzien in een maximum van 48 zg. Apparaten, genummerd van 0 t/m
47.
Ze
vallen uiteen in twee groepen:
nr.
0 t/m 31: de echte apparaten
nr.
32 t/m 47: de administratieve apparaten.
De
administratieve apparaten verrichten parallel aan de basismachine
zekere handelingen (zoals “tandenpoetsen” en
“foutafhandeling”); deze apparaten hebben een vast
nummer, dat niet installatie-afhankelijk is.
De
echte apparaten in een specifieke installatie ontlenen hun nummer aan
de plaats in de communicatiebekabeling; deze apparaatnummers zijn dus
wel installatie-afhankelijk, kunnen zelfs gewijzigd moeten worden bij
heropstelling.

1.
Communicatie
opdrachten.

Het
opdrachtenrepertoire van de X8 heeft de mogelijkheid 1ste, (2de en
3de) zg. IF-woord in A of S te lezen. Tot nu toe is alleen de layout
van het 1ste IF-woord beslist:

d26 = permanent 0 (
is tekenbit)

d25 = IF app. nr.
39

d18 = IF app. nr.
32

d17 = IF app. nr. 0

d0 = IF app. nr. 17

Bij
de laatste 26 toevoegingen betekent d = 1 : IF = true
(IF = Ingreep Flip Flop).

Het zetten van IF's
door de basismachine geschiedt individueel door opdrachten:

zet IF true

zet IF false

waarbij
het apparaat wordt aangewezen door het adresgedeelte van de opdracht.

N.B. Deze opdrachten
zijn B-modificeerbaar.
Voorts
is aan elk apparaat toegevoegd een zg. LVIF (Luister Vergunning op de
Ingreep Flip Flop), het opdrachten repertoire bevat de opdrachten:

zet LVIF true

zet LVIF false

waarbij
weer het apparaat wordt aangewezen door het adresgedeelte van de
opdracht. Ook deze opdrachten zijn B-modificeerbaar.

Tenslotte is aan elk
apparaat toegevoegd een zg. AF (Actie Flip Flop); deze kan door de
basismachine slechts in 1 richting gezet worden, nl.:

zet AF true

waarbij
weer het apparaat wordt aangewezen door het adresgedeelte van de
opdracht. Ook deze opdrachten zijn B-modificeerbaar.

Tenslotte kent de
basismachine:

zet IV true
(maak horend)

zet IV false
(maak doof)

(IV
= Ingreep Vergunning).

2.
Betekenis
van IF, LVIF, IV en AF

Als
een IF true
is, betekent dit, dat de
basismachine op ten minste 1
terugmelding van het
apparaat in kwestie nog niet heeft gereageerd.
Een
ingreep vindt plaats mits

- IV =
true
and

- Voor
tenminste 1 apparaat geldt “LVIF and
IF”.

Door het zetten van
de LVIF's kan de basismachine reguleren, van welke apparaten het
ingreepmechanisme al dan geen nota neemt. Door het zetten van IV kan
de basismachine (nl., door “maak doof”) tijdelijk
verhinderen door ingrepen onderbroken te worden.
De
uitvoering van de ingreep bestaat uit de uitvoering van de opdracht
op adres 24 in plaats van de volgende opdracht. Op adres 24 moet een
niet stapelende subroutinesprong staan; zijn uitvoering is in twee
opzichten bijzonder:

- Het
ophogen van de opdrachtteller OT wordt onderdrukt (de opdracht is
immers niet onder kontrole van OT aangehaald).

- De
basismachine wordt doof.

De
subroutinesprong in 24 verwijst naar het begin van de ingreeproutine,
die begint met de status quo van het onderbroken programma te redden,
waarna onderzocht wordt, welke IF of IF's voor de ingreep
verantwoordelijk waren. Wat de IF is voor de basismachine is de AF
voor Charon. Zoals IF = true
betekent, dat er voor de
basismachine weer wat te doen is, zo betekent AF = true,
dat er voor het bijbehorende communicatie apparaat weer wat te doen
is, d.w.z. dat er tenminste 1 opdracht voor dit apparaat gegeven is.

3.
Vaste
toekenning van geheugenplaatsen aan apparaten

De
AF meldt voor een apparaat aan Charon de aanwezigheid
van tenminste 1
startopdracht, de IF meldt omgekeerd aan de basismachine de
aanwezigheid
van tenminste 1
terugmelding van dit apparaat. Nadere specificatie (aantal
startopdrachten en welke, aantal terugmeldingen en welke, etc.) vindt
plaats via het kerngeheugen, dat zowel voor de basismachine als voor
Charon toegankelijk is (met prioriteit voor Charon).

Voor deze
communicatie zijn voor elk apparaat vier opeenvolgende
geheugenplaatsen gereserveerd te beginnen bij:

64
+ 4 * apparaatnummer

(Dit
loopt dus van 64 t/m 255). In het volgende zullen deze 4 woorden
worden aangeduid met AR0, AR1, AR2 en AR 3.
De
9 meest significante bits enerzijds en de 18 minst significante bits
anderzijds worden vaak als onafhankelijke bestanddelen beschouwd; het
stuk d26 t/m d18 heet dan “ het tellingdeel”, het stuk
van d17 t/m d0 heet dan “het adresdeel”.

Vooruitlopend geven
wij de algemene functie van de 4 apparaat woorden.

Opm 1.
Deze functies zijn
niet bij alle apparaten van toepassing; uitzonderingen zullen per
apparaat vermeld worden.

Opm 2.
In dit
vooruitlopend overzicht geven wij geven wij slechts een eerste
aanduiding van de betekenissen.

AR0 :

Tellingdeel
:
werkruimte Charon, in het bijzonder gereserveerd voor OK-status.

Adresdeel
: label
van de heersende schakel.

AR1 :

Tellingdeel
: IFT

Adresdeel
:
irrelevant, ≠ (wordt uit compalibiliteitsoverwegingen met de X2
door Charon niet gebruikt)

AR2 :

Tellingdeel
: AFT

Adresdeel
: meestal
werkruimte voor Charon

AR3 :
Werkruimte voor
Charon (vaak gereserveerd voor het zg. “lopende codewoord”).

4. Ketens
van startopdrachten

4.1. Layout
van de startopdracht

Per
apparaat kan men een keten van startopdrachten “aanhangig
maken”. Elke startopdracht – omdat hij alleen in een
keten voorkomt ook wel “schakel” genoemd – bestaat
uit een aantal opeenvolgende geheugenplaatsen. Onder de “label
van een schakel” verstaat men het adres van het tweede woord
van deze schakel.

De
algemene layout van een startopdracht is:

label [–1] :
zg. slotwoord. Vóór
de feitelijke uitvoering van deze startopdracht, is de inhoud van
dit woord voor Charon irrelevant; na afloop van de uitvoering
heeft Charon verslaglegging gedaan over de afloop van de
startopdracht.

label [0] :

Tellingdeel
:
allemaal nullen

Adresdeel
: label
van de volgende schakel

label [1] :
Codewoorden ter
beschrijving van

etc.
de startopdracht;
de layout hiervan is apparaat afhankelijk.

Een startopdracht
wordt dus klaargemaakt in de vorm, die specifiek is voor het apparaat
in kwestie. De startopdracht is niet positioneel gebonden aan het
gebied van het kerngeheugen, waarop de feitelijke informatietransport
betrekking heeft. Omdat voor de labels altijd 18 bits gereserveerd
zijn is er geen hardware restrictie aan hun plaats opgelegd.

4. 2. Het
aanhaken van een startopdracht

In
AFT (0 ≤ AFT ≤ 511) wordt ten behoeve van Charon het aantal
schakels van de startketen genoteerd (inclusief de schakel, die door
Charon in behandeling is). In de Actie Flip Flop AF wordt aangegeven,
of ADT > 0 is.
Nadat
de nieuwe schakel volledig is geprepareerd (met op het door zijn
label aangewezen adres bv. + 0 wegens het ontbreken van een volgende
schakel) moet deze worden aangehaakt.
De
aanhaakorganisatie moet in een zg. vulwijzer VW de label van de
laatst aangehaakte schakel onthouden. Het aanhalingsprotocol is nu in
volgorde:

- 1.
Met
M[VW] := label nieuwe schakel wordt de nieuwe schakel aangehaakt.
(Als er geen “laatste schakel” (meer) is, moet dit worden
onderdrukt); verder wordt VW := label nieuwe schakel.

- Met
een additieve uitopdracht (versie PLUSA of PLUSS) moet AFT met 1
verhoogd worden, waarbij de nieuwe waarde van AFT tevens in het
basismachineregister komt.

- Indien
deze nieuwe AFT-waarde = 1 is moet de basismachine

3.1
het
adresdeel van AR0 door het nieuwe label vervangen (zonder het
tellingdeel te verstoren),

3.2.
vervolgens
AF true zetten.

Opm.
Tijdens
de behandeling van de startopdracht wijst het adresdeel van AR0 naar
deze startopdracht. Als afsluiting laagt Charon AFT met 1 af. Is het
resultaat hiervan > 0, dan vervangt Charon het adresdeel AR0 :=
adresdeel M[adresdeel AR0], d.w.z. gaat over op de volgende schakel.
Is het resultaat van de AFT aflaging = 0 (afgeleverd in de vorm van
negen nullen), dan wordt simultaan AF false
gezet
en de overgang op de volgende schakel wordt onderdrukt. Dit, plus het
feit dat bij niet lege startketen de AFT aflaging op voor de
basismachine oncontroleerbare ogenblikken kan plaatsvinden is de
achtergrond van voornoemde aanhaakriten, waar streng de hand gehouden
moet worden.

4.
3. Het
reageren op een terugmelding

In
IFT wordt ten behoeve van de basismachine genoteerd het aantal
terugmeldingen van Charon, waarop de basismachine nog niet gereageerd
heeft. Voltooiing van een terugmelding wordt door Charon afgesloten
met de ophoging van IFT met 1 en het true
zetten van IF.

De basismachine moet,
in het kader van de reactie op een terugmelding, IFT met 1 aflagen en
zorgen dat IF – die de betekenis “IFT>0” heeft –
passend wordt achtergelaten. Omdat ook hier Charon op voor de
basismachine oncontroleerbare ogenblikken de IFT-ophoging kan doen,
is de IFT aflaging en de bbh IF aanpassing doen de basismachine aan
een streng protocol onderwerpen.
Dit
luidt:

- Zet IF
False

- Laag
IFT met MINA, MINS ( of PLUSA of PLUSS) met 1 af.

2.1.
Doordat
het adresdeel van AF1 ≠ nul is, wordt, als IFT algebraïsch
nul wordt, het tellingdeel van AR1 negen nullen.

- Als de
in het register aangetroffen nieuwe waarde van IFT ≠ 0 is, zet dan
IF true.

Ook aan dit protocol
moet streng de hand gehouden worden.

5.
Het
toetsenbord

Het
toetsenbord is een in voor apparaat, waarvan het nummer
installatie-afhankelijk is.

Het
is een afwijkend apparaat in zoverre de operateur dit invoer apparaat
bedienen kan, zonder dat de basismachine er een startopdracht voor
heeft gegeven.
Wanneer
de basismachine bereid is een volgend karakter van het toetsenbord te
accepteren dient van

AR1 :
het tellingdeel uit
negen nullen te bestaan, het adresdeel ≠
nul te zijn

AF dient true
te zijn.

De aanvangswaarde van
AR0, AR2 en AR3 is irrelevant.

Indrukken
van een toets heeft tot gevolg

- if
AF then
AR0 := karakter else
AR3 := karakter;

- AF := false;

- IFT := IFT + 1 en
IF := true;

Het protocol voor de
reactie op de ingreep van het toetsenbord is als volgt:

L0 : IF := false;
IFT := IFT - 1 annex if IFT > 0 then
begin AR0 := iets negatiefs: goto L0 end;
A := AR0;
if A > 0 then begin AR0 := iets negatiefs; A moet nu als volgend
symbool geaccepteerd worden end;
AF := true; comment nu kan het toetsenbord weer iets in AR0 zetten;

Onder “accepteren
van een symbool” valt onder meer het uitprinten hiervan. Door
pas daarna AF = true
te zetten, kan men ervoor
zorgen, dat de basismachine pas een volgend karakter accepteert,
nadat het voorgaande netjes is verwerkt. Het toetsenbord plaatst de
X8 voor een essentiële haastsituatie: de X8 moet dit karakter zo
snel mogelijk verwerken, maar “AF := true” tot het bitter einde
uitstellen.

6.
Het
teleprinterdrukwerk

De
indeling voor AR0, AR1, AR2 en AR3 is normaal. Het tellingdeel van
AR0 bevat de OK-status, het adresdeel van AR2 wordt niet gebruikt.
Het lopend codewoord in AR3 is gesplitst in tellingdeel en adresdeel
(zie onder). De startschakel bestaat uit 3 woorden, de eerste twee
zijn normaal, de laatste, label [1], bevat het codewoord.

Een startschakel
geeft de mogelijkheid om de woorden uit N opeenvolgende adressen naar
het drukwerk van de teleprinter te sturen. Van elk woord bepalen de
laagste een actie van het drukwerk (Ik neem aan dat in elk van deze
woorden de 21 hoogste bits irrelevant zijn, ik zou ze altijd =0
maken.)

Het
codewoord bepaald lengt en ligging van de uit te voeren waardering,
en wel

tellingdeel codewoord

=
lengte van de rij,
(waarbij tellingdeel = 0 de lengte

=512
voorschrijft; in de lege

startopdracht is dus niet
voorzien)

adresdeel codewoord
= beginadres van de waardering –
1.

Als
de startopdacht door Charon geaccepteerd wordt, wordt het codewoord
uit de heersende schakel in het lopende codewoord (in AR3 dus)
overgenomen.

Per
woordverzending naar het drukwerk neemt Charon tweemaal contact met
het geheugen op. In het eerste geheugencontact met AR3 wordt het
tellingdeel (mod. 512) met 1 verlaagd en wordt het adresdeel ( modulo
218)
met 1 verhoogd. Is het resulterend tellingdeel = 0, dan is het nu nog
volgend woordtransport het laatste van de startopdracht. In het
geheugencontact, volgend op dat met AR3 wordt het woord
getransporteerd, aangewezen door het zojuist in AR3 weggeschreven
adresdeel. Als – door storing – de startopdracht
voortijdig wordt afgebroken, bevindt zich in het tellingdeel van AR3
dus het resterende aantal te printen woorden, in het adresdeel het
adres van het laatste getransporteerde woord.

De
heersende OK-status van het drukwerk wordt onthouden in het
tellingdeel van AR0; na afloop van elke startopdracht wordt dit
tellingdeel overgenomen in het tellingdeel van het slotwoord (label
[–1]) van de schakel in voltooiing. Het adresdeel van het slotwoord
is ongedefinieerd.

Bij
het drukwerk heeft de OK-indicatie slechts 2 mogelijke waarden:

000 000 000      OK

111 111 111      niet
bedrijfsklaar (d.w.z. bv. niet aangesloten)

De
status van het Charon programma voor het drukwerk kent dus twee
toestanden, kortweg aangeduid met OK en non
OK.
Boven
hebben we de normale afwerking van een startopdracht beschreven, als
deze in de toestand OK geaccepteerd wordt. Een startopdracht, die in
de toestand non
OK geaccepteerd wordt,
wordt op één uitzondering na, onmiddellijk symbolisch
afgewerkt. (Onder terugmelding van de non
OK status in het
slotwoord). De ene uitzondering is de zg. HOK-opdracht (Herstel OK
toestand), die normaal in de startketen kan worden opgenomen en
gekarakteriseerd is door een codewoord bestaand uit louter nullen.
(Mede tengevolge hiervan kan men niet
in 1 startopdracht de
inhoud van de woorden op adressen 1 t/m 512 naar het drukwerk sturen:
deze zijn nl. als HOK-opdracht geïnterpreteerd worden. De
HOK-opdracht moet niet opgegeven worden als het apparaat al in de
OK-toestand is.

NB.
Toetsenbord en teleprinterdrukwerk houden elk, onafhankelijk van
elkaar, op mechanische wijze de heersende shift bij. Bij het drukwerk
bepaalt de shift welke van de twee tekens bij een vijfbits
drukcommando op papier verschijnt; bij het toetsenbord bepaalt de
heersende shift welke helft van het toetsenbord geblokkeerd is.

6.1. Hardware
code teleprinter EL X8

(Weggelaten
zie origineel.)
Bij
zwart lint heeft X als
neveneffect het starten van een rood flikkerend attentiesignaal, dat
alleen door de operateur kan worden afgezet.

7.
Het
bandlezer

De
betekenis van de centrale administratie AR0 t/m AR3 is normaal, als
in 3 beschreven.

De
startschakel bestaat uit 3 opeenvolgende woorden:

label [-1] :

slotwoord en wel:

tellingdeel
:
Ok
indicatie

adresdeel
:
undefined

label [0] :

tellingdeel
:
allemaal nullen

adresdeel
:
label
van de volgende schakel

label [+1] :
enig codewoord en
wel

tellingdeel
:

lengte
van de rij (waarbij tellingdeel =0

de lengte =512 voorschrijft; in
de lege start-

opdracht is dus niet voorzien)

adresdeel
:
beginadres van de woordenrij – 1.

Als
de startopdracht door Charon geaccepteerd wordt, wordt het codewoord
uit de heersende schakel in AR3 overgenomen. Per gelezen ponsing
neemt Charon twee maal contact met het geheugen op. In het eerste
geheugencontact met AR3 wordt het tellingdeel (mod. 512) met 1
verlaagd en wordt het adresdeel ( mod. 218)
met 1 verhoogd. Is het resulterend tellingdeel = 0, dan is het nu nog
volgend woordtransport het laatste van de startopdracht. In het
geheugencontact, volgend op dat met AR3 wordt de gelezen ponsing, aan
de meest significante zijde aangevuld met nullen, ingevuld op de
geheugenplaats, die wordt aangewezen door het zojuist in AR3
weggeschreven adresdeel.

Wanneer
de startopdracht voortijdig wordt afgebroken, bevindt zich in AR3 in
het tellinggedeelte het ontbrekende aantal ponsingen, in het
adresdeel het adres van het laatste gevulde woord.
De
heersende OK-status van de bandlezer wordt onthouden in het
tellingdeel van AR0. Aan het einde van de verwerking van een
startopdracht wordt de dan heersende OK-status gecopieerd in het
tellingdeel van het slotwoord van de betrokken schakel.

de OK-status kent
drie waarden:

000 000 000

111 111 111

111
111 110
OK

niet
bedrijfsklaar (d.w.z. bv. niet aangesloten

einde band

De
laatste twee toestanden worden kortweg aangeduid met “non
OK”. Als de
bandlezer non
OK is, worden alle
volgende startopdrachten op één uitzondering na
onmiddellijk symbolisch afgewerkt, onder terugmelding van de
heersende non
OK status in het
slotwoord. De ene uitzondering is de zg. HOK-opdracht (Herstel OK
toestand), die normaal in de starttreden kan worden opgenomen. Mede
tengevolge hiervan kan men niet
in een startopdracht 512
ponsingen willen doen lezen, het opbergen te beginnen bij adres 1.
De HOK-opdracht moet niet
opgegeven worden als de
bandlezer al OK is.

Opm.
1 De
symbolische afwerking verstoord AR3 niet, zodat het gegeven hoeveel
er nog gelezen is, voordat “einde band” het leesproces
afbrak, niet verloren gaat, voordat de volgende leesopdracht
effectief geaccepteerd wordt.
Opm.
2 De
bandlezer leest met een snelheid van 1000 ponsingen per seconde; het
leesproces belegt het geheugen dus voor ½ procent van de tijd.

Opm.
3 De
bandlezer bevat een schakelaar voor verschillende bandbreedten Deze
bandbreedte wordt niet geacht bij te dragen tot de informatie,
programma-technisch is de heersende bandbreedte niet detecteerbaar.

Opm.
4 De
bandlezer leest alle configuraties ongeinterpreteerd, ook “blank
tape” en “all holes”.

De
band passeert in volgorde:
1.
een
mechanische detector voor einde band,
2.
de
rem,
3.
het
leesstation,
4.
de
aandrijving.

De
uiteinden van de band kunnen daardoor niet gelezen worden.

De bandlezer is
uitgerust met een rode en een groene knop. Door op de rode knop te
drukken stopt men de bandlezer onmiddellijk en maakt men de detector
op einde band non-effectief. Door op de groene knop te drukken keert
de bandlezer in zijn normale toestand terug. Het indrukken van de
rode en groene knop dringt als zodanig niet tot de basismachine door:
als een startopdracht onder behandeling is (of komt) duurt het alleen
wat lang, voordat hij wordt afgehandeld.
Wanneer
einde band gedetecteerd wordt, springt de bandlezer instantaan in de
toestand alsof op de rode knop gedrukt is, het indrukken van de
groene knop heeft slechts effect wanneer er een nog voldoende lange
band in de bandlezer ligt.
Opm.
5 Der
rode knop geeft de operateur de gelegenheid in noodsituaties de
bandlezer even te stoppen (knoop of krinkel of wat dan ook) zonder
dat dit tot de logica van het verwerkende programma doordringt. Zelfs
kan hij, als hij voldoende rap is, een aantal banden achter elkaar
als een band aan het verwerkende programma offreren.

Opm.
6 Als
een band tot het einde gelezen is moeten er, voordat een volgende
band over die bandlezer gelezen kan worden, twee dingen gebeurd zijn:

- de
basismachine moet een HOK-opdracht gegeven hebben.

- de
operateur moet het inleggen van de volgende band hebben afgesloten
met het drukken op de groene knop.

De
volgorde van deze twee acties doet niet ter zake.

8.
Het
bandponser

De
centrale administratie op de plaatsen AR0 t/m AR3, evenals de layout
van de startschakel is bij de bandponser als bij het
teleprinterdrukwerk, zie 6. (Teleprinter en bandponser worden nl.
door precies hetzelfde Charonprogramma bediend.) Nu bepalen van elk
woord de laagste 7 bits, wat er geponst wordt. (Ik neem aan, dat de
resterende 20 bits irrelevant zijn, ik zou ze altijd =0 maken.)

Ook de bandponser
heeft een rode en een groene knop. het indrukken van de rode knop
heeft tot gevolg, dat de bandponser instantaan stopt en dat de
selectie voor bandeinde tijdelijk ineffectief is. Door het indrukken
van de groene knop wordt dit weer ongedaan gemaakt.

Als de detectie voor
het bandeinde voor het eerst alarm geeft, is er nog minstens een paar
meter band ter beschildering. In tegenstelling tot de bandlezer, die
bij bandeinde instantaan stopt, maakt de bandponser de heersende
startopdracht rustig af en gaat dan pas over in de toestand “einde
band”, waarin de volgende startopdrachten (if any) symbolisch
worden afgemaakt totaan de eerste HOK-opdracht.
Deze
HOK-opdracht brengt de bandponserij in de afsluittoestand. In de
afsluittoestand worden ponsopdrachten weer normaal geaccepteerd, zij
het dat aan noodkreten van de einde band detectie geen aandacht wordt
geschonken. De afsluittoestand is geïntroduceerd om de
programmeur de gelegenheid te geven eindindicaties op de band te
ponsen; het is echter zijn verantwoordelijkheid om te zorgen, dat
deze eindindicaties niet te lang zijn.
Na
de laatste ponsopdracht van de afsluiting geeft de programmeur een
tweede HOK-opdracht, die dubbel gevolg heeft. Het Charonprogramma
gaat weer in de normale OK-toestand over en de programmeur kan de
startopdrachten voor de volgende band aanbieden. Een tweede effect
van de tweede HOK-opdracht is dat de bandponser zelf in de toestand
raakt, alsof de rode knop ingedrukt is, waarbij drukken op de groene
knop slechts effect heeft nadat een nieuwe band is ingelegd.

Opm.
Met
het aanbieden van de startopdrachten voor de volgende band hoeft het
programma dus niet te wachten, totdat de operateur de band gewisseld
heeft.
De
OK-status geeft vier waarden:

000 000 000
111 111 111

111
111 110

111
111 101
OK

niet
bedrijfsklaar

einde band

afsluittoestand

9.
Regeldrukker

De
centrale administratie (AR9t/m AR3) evenals de driewoords
startschakel (label [-1] t/m label [+1]) hebben de normale functie.
Het laatste woord van de startschakel is het codewoord, dat op nader
te beschrijven wijze aangeeft, wat voor regel gedrukt moet worden.

Ter specificatie van
de startopdracht bouwt men in het geheugen op een aantal
opeenvolgende adressen het zg. regelbeeld op. Plaats en lengte van
het regelbeeld worden gegeven in het codewoord van de startschakel en
wel

codewoord:
tellingdeel
:
aantal woorden dat het regelbeeld bestaat

(1 ≤ dit aantal ≤
36)

adresdeel
:
beginadres van het regelbeeld – 1.

Charon
zendt de woorden van het regelbeeld in volgorde van opklimmend adres
naar de regeldrukker. In elk woord zijn vier zes-bits symbolen gepakt

d26.... d21
1ste symbool

d19.... d14
2de symbool

d12.... d7
3de symbool

d5.... d0
4de symbool

De
drie scheidingsbits d20, d13 en d6 moeten =0 zijn.

De
actie van de regeldrukker laat zich het best beschrijven in termen
van zijn reactie op een van als boven gedefinieerde symbolenrij
(waarvan de lengte altijd een viervoud is.)
Het
allereerste symbool van de rij wordt afwijkend geïnterpreteerd
en is de zg. sprong indicatie (zie onder); daarna specificeren de
symbolen van de rij wat de regeldrukker op die regel moet drukken,
positie per positie in de volgorde van links naar rechts. Het
maximale aantal karakters per regel is 143. Als het regelbeeld in het
geheugen korter is dan 36 woorden, laat de regeldrukker de
ongespecificeerde posities aan het rechtereinde van de regel
onbedrukt. De lengte van de symbolenrij die naar de regeldrukker
gestuurd wordt is per definitie een viervoud; zonodig moet de
programmeur de rij dus aanvullen met 1, 2 of 3 spaties.

De betekenis van de
sprong indicatie (opgevat als een getal van 0 t/m 63) is als volgt:

0
t/m 31
voer het papier op
over het opgegeven aantal regels en druk vervolgens de regel af,
zoals door de rest van het regelbeeld is opgegeven.

32
t/m 39
regelopvoer
gestuurd door een achtgats ponsbandje, druk dan.

40
t/m 63
komt niet voor

(In
geval van formulieren van een vaste lengte kan men in de regeldrukker
een cyclisch geplakt achtgats ponsbandje inleggen, dat met elke regel
over een ponsing getransporteerd wordt. Dit bandje geeft 8 kanalen,
genummerd van 0 t/m 7; de bandgestuurde regelopvoer vindt plaats,
totdat in kanaal nr. “sprong indicatie – 32” een
ponsing in het bandje gedetecteerd wordt. Bij koppeling aan een
rekenautomaat is dit soort faciliteit overbodig, ik ben niet van plan
hem te gebruiken.)

De
betekenis van de symbolen uit de rij na de sprong indicatie volgt uit
de volgende tabel. De symboolwaarde is opgevat als getal van 0 t/m
63. (wals EL X( van THE.)
(tabel
weggelaten, zie origineel document).

Opm.
In de afdruk is de nul smaller dan de letter O.

De normale toestand
is OK d.w.z.

Tellingdeel
AR0
= 000    000    000

slotwoord
= +0     (d.w.z. allemaal nullen)

De
regeldrukker bevat een detector “paper low”; als deze
signaleert, blijft de OK-toestand bestaan – regelprint
opdrachten worden dus normaal geaccepteerd – maar in de
startschakel verschijnt

slotwoord = +1 (d26
= ··· = d1 = 0, d0 = 1)

Geeft
de programmeur nu – d.w.z. terwijl de toestand OK is –
een HOK-opdracht – als steeds in de vorm van een codewoord
gelijk aan louter nullen – dan gaat op de regeldrukker het
lichtje “Change Paper” branden. De regeldrukker raakt
daardoor in de toestand, alsof op de rode knop gedrukt is, drukken op
de groene knop heeft slechts effect als “paper low” geen
alarm geeft, d.w.z. als de operateur zonodig nieuw papier heeft
ingedaan.

Als
de regeldrukker niet is aangesloten dan wordt:

Tellingdeel AR0
= 111 111 111
niet
bedrijfsklaar

slotwoord
= 111 111 111 ,
gevolgd door een ongedefinieerd adresdeel

Deze
toestand geldt als non
OK, en de volgende
startopdrachten ≠ HOK worden symbolisch afgemaakt.

Drie andere oorzaken
kunnen de toestand non
OK tevoorschijn roepen.
Zij laten achter:

tellingdeel AR0
=
111 111 110 en

slotwoord
:
tellingdeel = 111
111 110

adresdeel: d17
= ····· d4
= 0

d3 = 1
als “Yoke
open”

d2 = 1
als papier
gescheurd

d1 = 1
als
pariteitsfout in transmissie

d0 = 1
als tevens
“paper low”

Van de bits d3, d2 en
d1 moet er minstens 1 gelijk zijn aan 1. De pariteits-foutsignalering
d1 = 1 betreft de symbooltransmissie voor de regeldrukker: elk
zesbits symbool wordt nl. voorzien van een pariteitsbit naar de
regeldrukker gestuurd. Bij standaardwalsen, waar minder dan 64
verschillende symbolen in de rij zijn toegestaan betekent d1 = 1
“pariteitsfout in transmissie of onbestaanbaar symbool”.

Bij elke niet-lege
combinatie wordt tevens in d0 = 1 paper low gemeld. In dat geval
herstelt de eerste HOK-opdracht de OK-toestand, maar de indicatie
paper low blijft hangen. Ee tweede HOK-opdracht vanuit de machine
schakelt dan normaal het lichtje “Change Paper” aan.

Opm.
1 Bij
non
OK
door een van de drie genoemde oorzaken kan het reagerende programma
d0 negeren. Blijkt “paper low” te heersen, dan merkt men
dat wel bij de eerstvolgende startopdracht.

Opm.
2 De
voorstelling van zaken, alsof er aan de regeldrukker slechts een rode
en een groene knop zitten, is wat simplistisch: er zitten er meer
aan. Het effect is echter wel, dat na de HOK-opdracht na “paper
low” de basismachine zonder meer de volgende startopdrachten al
kan aanbieden, zij zullen pas verwerkt worden, nadat de operateur
volgend de regels van de kunst nieuw papier heeft ingezet.

Opm.
3 Als
de regelprintopdracht voortijdig is afgebroken bevat het tellingdeel
van AR3 zoals gebruikelijk het aantal resterende (d.w.z. niet
getransporteerde) woorden en het adresdeel het adres van het laatst
getransporteerde woord. Als de regelprintopdracht normaal is
afgewerkt, bevat het tellingdeel 511 en het adresdeel het adres van
“het eerste niet getransporteerde woord”.

10. De
trommel

In
de centrale administratie hebben AR0, AR1 en AR2 de normale
betekenis. AR3 is ongebruikt. Een startopdracht is een schakel
bestaande uit 5 opeenvolgende woorden.

label [–1] :

tellingdeel
:   copie
van OK-status als in tellingdeel van AR0

adresdeel
:   nadere
specificatie slotwoord (zie onder)

label [0] :

tellingdeel
:   000
000 000

adresdeel
:
labeladres volgende schakel.

label [1] :

d26 t/m
d19
:   irrelevant, mits niet alle = 0.

d18 t/m d0
:
beginadres op trommel

label [2] :

d26
= ··· =
d19 = 0

d18
= 0
transportrichting van trommel naar kernen

= 1
transportrichting van kernen naar trommel.

label [3] :

d26
= ··· =
d12 = 0

d11 t/m d0 : lengte
( dus < 4096)

In
tellingdeel van AR0 betekent:

000 000 000
111 111 111

111
111 110
OK

niet
bedrijfsklaar

iets mis

De
laatste twee toestanden geven aanleiding tot symbolisch afwerken
totaan de HOK-opdracht (Gekaraktiseerd door label[1] = + 0; label [2]
en label [3] doen in de HOK-opdracht niet mee).

In
het slotwoord is

d26 t/m d18
: (tellingdeel) :
een copie van tellingdeel van AR3.

d17 = … =
d14
: altijd = 0

d13 = 1
als te laat
geaccepteerd geheugencontact

d12 = 1
als pariteitsfout.

d11 t/m d0
eindstand
aantallenteller (moet = 0 zijn).

d13
= 1 of d12 = 1 impliceert OK-status: 111 111 110.

Ad d13:
De trommel plaatst
het kerngeheugen voor een essentiële haastsituatie. Als deze
(door schromelijke overbelasting!) niet gehonoreerd zou kunnen
worden, wordt d13 = 1 gezet en het transport onderbroken.

Ad d12:
De
trommeltransporten zijn aan de normale pariteitscontrole
onderworpen. Als bij een transport van kernen naar trommel een
woord met onjuiste pariteit in de kernen staat, zal de
transportopdracht alarm geven. Nogmaals proberen is dan
gegarandeerd zinloos.

Opm.
1 In
“label [1]” moet tenminste 1 van de bits =1 zijn om de
opdracht van de HOK-opdracht te kunnen onderscheiden.

Opm.
2 De
ophoging van het lopend trommeladres geschiedt modulo 2 ↑ 19 :
het trommelgeheugen is dus cyclisch gerangschikt.

Opm.
3 Bij
een transport van trommel naar kernen, die er niet zijn, gaat
informatie ongedetecteerd verloren; bij een transport uit kernen die
er niet zin, slaat de pariteitscontrole alarm. Ook het lopend
kernenadres telt rond ( modulo 2 ↑ 18), maar erg interessant is
dit niet.
Opm.
4 In
d 11 t/m d0 vindt men de eindstand van de teller, d.w.z. het aantal
ongetransporteerde woorden. Bij fout geldt het mislukte woord als
getransporteerd: als het laatste woord vindt men, dus d11 = …
= d0 = 0.

11. De klok

De klok is
onveranderljk apparaat nr. 39. Hij werkt onafhankelijk van AF en
heeft drie functies.

11.1
De
eigenlijke klok

Elke
10 insec wordt AR0 modulo 2 ↑ 27 met 1 verhoogd. De cyclus van
dit telproces is van de orde van grootte van twee weken.

11.2
De
kodewekker

Elke
10 insec wordt het adresdeel van AR1 modulo 2 ↑ 18 met 1
verminderd. Als hierdoor AR1 totaal = +0 wordt, wordt in het
tellingdeel van AR1 :000 000 001 gezet en IF wordt true,
m.a.w. de ingreep van de kodewekker kan plaatsvinden. Wie deze
ingreep niet wil hebben, kan hem dus definitief onderdrukken door het
tellingdeel van AR1 meteen ≠ te
maken. De maximale looptijd van de kodewekker is circa 40 minuten.

11.3
De
apparaatwekker

Modulo
512 wordt elke 10 insec het tellingdeel van AR2 met 1 verminderd. Als
het resultaat in het tellingdeel = 0 wordt en het adresdeel van AR2 =
NR ≠ 0
dan wordt:

- van het apparaat met apparaatnummer = NR de LVAF op true gezet,

- in AR2 wordt AR3 overgenomen.

Men
initieert dit proces door eenmalig AR3 en AR2 met hetzelfde te
vullen. Men kan een aangewezen apparaat om de zoveel tijd ergens nota
van laten nemen, met maximum periode van circa vijf seconden.

transcribed
by Martin van der Burgt

revised 2013-06-20

---

## English translation

### Documentation on the communication apparatus attached to the EL X8

Documentation on communication apparatus attached to the EL X8

0. Introduction

Provision
has been made for a maximum of 48 so-called Devices, numbered from 0 through
47.
They
fall into two groups:
nr.
0 through 31: the real devices
nr.
32 through 47: the administrative devices.
The
administrative devices perform, in parallel with the base machine,
certain operations (such as “teeth-brushing” and
“fault handling”); these devices have a fixed
number, which is not installation-dependent.
The
real devices in a specific installation derive their number from
their place in the communication cabling; these device numbers are therefore
indeed installation-dependent, and may even have to be changed upon
re-installation.

1.
Communication
instructions.

The
instruction repertoire of the X8 has the possibility of reading the 1st, (2nd and
3rd) so-called IF-word into A or S. So far only the layout
of the 1st IF-word has been decided:

d26 = permanently 0 (
is the sign bit)

d25 = IF dev. nr.
39

d18 = IF dev. nr.
32

d17 = IF dev. nr. 0

d0 = IF dev. nr. 17

For
the last 26 entries d = 1 means: IF = true
(IF = Intervention Flip Flop).

The setting of IF's
by the base machine takes place individually by means of the instructions:

set IF true

set IF false

whereby
the device is designated by the address part of the instruction.

N.B. These instructions
are B-modifiable.
Furthermore,
to each device there is attached a so-called LVIF (Listening Permission on the
Intervention Flip Flop); the instruction repertoire contains the instructions:

set LVIF true

set LVIF false

whereby
again the device is designated by the address part of the
instruction. These instructions too are B-modifiable.

Finally, to each
device there is attached a so-called AF (Action Flip Flop); this can be
set by the base machine in only 1 direction, namely:

set AF true

whereby
again the device is designated by the address part of the
instruction. These instructions too are B-modifiable.

Finally the base
machine knows:

set IV true
(make hearing)

set IV false
(make deaf)

(IV
= Intervention Permission).

2.
Meaning
of IF, LVIF, IV and AF

If
an IF is true,
this means that the
base machine has not yet reacted to at least 1
report-back from the
device in question.
An
intervention takes place provided

- IV =
true
and

- For
at least 1 device “LVIF and
IF” holds.

By setting
the LVIF's the base machine can regulate which devices the
intervention mechanism does or does not take note of. By setting IV the
base machine can (namely, by “make deaf”) temporarily
prevent itself from being interrupted by interventions.
The
execution of the intervention consists of the execution of the instruction
at address 24 in place of the next instruction. At address 24 there must be a
non-stacking subroutine jump; its execution is special in two
respects:

- The
incrementing of the instruction counter OT is suppressed (the instruction was
indeed not fetched under control of OT).

- The
base machine becomes deaf.

The
subroutine jump in 24 refers to the beginning of the intervention routine,
which begins by saving the status quo of the interrupted program,
after which it is investigated which IF or IF's were
responsible for the intervention. What the IF is to the base machine, the AF
is to Charon. Just as IF = true
means that there is
something for the base machine to do again, so AF = true means
that there is something for the corresponding communication device to do
again, i.e. that at least 1 instruction has been given for this device.

3.
Fixed
assignment of memory locations to devices

The
AF reports for a device to Charon the presence
of at least 1
start instruction; the IF conversely reports to the base machine the
presence
of at least 1
report-back from this device. Further specification (number of
start instructions and which, number of report-backs and which, etc.) takes
place via the core store, which is accessible both to the base machine and to
Charon (with priority for Charon).

For this
communication, for each device four consecutive
memory locations are reserved, beginning at:

64
+ 4 * device number

(This
thus runs from 64 through 255). In what follows these 4 words
will be denoted by AR0, AR1, AR2 and AR 3.
The
9 most significant bits on the one hand and the 18 least significant bits
on the other are often regarded as independent components; the
part d26 through d18 is then called “the count part”, the part
from d17 through d0 is then called “the address part”.

By way of anticipation we
give the general function of the 4 device words.

Rem. 1.
These functions are
not applicable to all devices; exceptions will be mentioned
per device.

Rem. 2.
In this
anticipatory overview we give we give only a first
indication of the meanings.

AR0 :

Count part
:
work space for Charon, in particular reserved for the OK-status.

Address part
: label
of the prevailing link.

AR1 :

Count part
: IFT

Address part
:
irrelevant, ≠ (for reasons of compatibility with the X2
it is not used by Charon)

AR2 :

Count part
: AFT

Address part
: usually
work space for Charon

AR3 :
Work space for
Charon (often reserved for the so-called “running code word”).

4. Chains
of start instructions

4.1. Layout
of the start instruction

Per
device one can “make pending” a chain of start instructions.
Each start instruction – because it occurs only in a
chain also called a “link” – consists
of a number of consecutive memory locations. By the “label
of a link” one understands the address of the second word
of this link.

The
general layout of a start instruction is:

label [–1] :
so-called final word. Before
the actual execution of this start instruction, the content of
this word is irrelevant to Charon; after the completion of the execution
Charon has rendered an account of the outcome of the
start instruction.

label [0] :

Count part
:
all zeros

Address part
: label
of the next link

label [1] :
Code words for the
description of

etc.
the start instruction;
their layout is device-dependent.

A start instruction
is thus prepared in the form which is specific to the device
in question. The start instruction is not positionally bound to the
region of the core store to which the actual information transport
relates. Because for the labels 18 bits are always reserved
there is no hardware restriction imposed on their location.

4. 2. The
hooking-on of a start instruction

In
AFT (0 ≤ AFT ≤ 511) there is recorded, for the benefit of Charon, the number
of links of the start chain (including the link which is being
handled by Charon). In the Action Flip Flop AF it is indicated
whether ADT > 0.
After
the new link has been completely prepared (with, at the address designated by its
label, e.g. + 0 on account of the absence of a next
link) it must be hooked on.
The
hooking-on organization must, in a so-called fill pointer VW, remember the label of the
last link hooked on. The hooking-on protocol is now, in
order:

- 1.
By
M[VW] := label of new link the new link is hooked on.
(If there is no “last link” (any more), this must be
suppressed); furthermore VW := label of new link.

- By
an additive output instruction (version PLUSA or PLUSS) AFT must be
incremented by 1, whereby the new value of AFT also comes into the
base-machine register.

- If
this new AFT-value = 1, the base machine must

3.1
replace the
address part of AR0 by the new label (without disturbing the
count part),

3.2.
subsequently
set AF true.

Rem.
During
the handling of the start instruction the address part of AR0 points to
this start instruction. As a conclusion Charon decrements AFT by 1. If the
result of this is > 0, then Charon replaces the address part AR0 :=
address part M[address part AR0], i.e. proceeds to the next link.
If the result of the AFT decrement = 0 (delivered in the form of
nine zeros), then simultaneously AF is set false
and the transition to the next link is suppressed. This, plus the
fact that with a non-empty start chain the AFT decrement can take place at
moments uncontrollable by the base machine, is the
background of the aforementioned hooking-on rites, which must be strictly
adhered to.

4.
3. The
reacting to a report-back

In
IFT there is recorded, for the benefit of the base machine, the number
of report-backs from Charon to which the base machine has not yet reacted.
Completion of a report-back is concluded by Charon
with the incrementing of IFT by 1 and the setting
of IF true.

The base machine must,
within the framework of the reaction to a report-back, decrement IFT by 1 and
ensure that IF – which has the meaning “IFT>0” –
is left behind suitably. Because here too Charon can do the
IFT-incrementing at moments uncontrollable by the base machine,
the IFT decrement and the corresponding IF adjustment subject the base machine to
a strict protocol.
This
reads:

- Set IF
False

- Decrement
IFT by 1 with MINA, MINS ( or PLUSA or PLUSS).

2.1.
Because
the address part of AF1 ≠ zero, when IFT becomes algebraically
zero the count part of AR1 becomes nine zeros.

- If
the new value of IFT found in the register ≠ 0, then set
IF true.

This protocol too
must be strictly adhered to.

5.
The
keyboard

The
keyboard is an input device whose number is
installation-dependent.

It
is a deviant device insofar as the operator can operate this input device
without the base machine having given a start instruction for it.
When
the base machine is prepared to accept a next character from the keyboard,
the following must hold of

AR1 :
the count part must consist of
nine zeros, the address part must be ≠
zero

AF must be true.

The initial value of
AR0, AR2 and AR3 is irrelevant.

Pressing
a key has as its consequence

- if
AF then
AR0 := character else
AR3 := character;

- AF := false;

- IFT := IFT + 1 and
IF := true;

The protocol for the
reaction to the intervention of the keyboard is as follows:

L0 : IF := false;
IFT := IFT - 1 annex if IFT > 0 then
begin AR0 := something negative: goto L0 end;
A := AR0;
if A > 0 then begin AR0 := something negative; A must now be accepted as the next
symbol end;
AF := true; comment now the keyboard can again put something in AR0;

Under “accepting
a symbol” there falls, among other things, the printing-out of it. By
setting AF = true
only afterwards, one can
ensure that the base machine accepts a next character only
after the preceding one has been neatly processed. The keyboard places the
X8 in an essential hurry situation: the X8 must process this character as
quickly as possible, but postpone “AF := true” to the bitter end.

6.
The
teleprinter printing unit

The
arrangement for AR0, AR1, AR2 and AR3 is normal. The count part of
AR0 contains the OK-status, the address part of AR2 is not used.
The running code word in AR3 is split into a count part and an address part
(see below). The start link consists of 3 words, the first two
are normal, the last, label [1], contains the code word.

A start link
gives the possibility of sending the words from N consecutive addresses to
the printing unit of the teleprinter. Of each word the lowest determine an
action of the printing unit (I assume that in each of these
words the 21 highest bits are irrelevant; I would always make them =0.)

The
code word determines the length and location of the valuation to be carried out,
namely

count part of code word

=
length of the array,
(whereby count part = 0 prescribes the length

=512;
provision is thus not made in the empty

start instruction)

address part of code word
= start address of the valuation –
1.

If
the start instruction is accepted by Charon, the code word
from the prevailing link is taken over into the running code word (in AR3, that is).

Per
word transmission to the printing unit Charon makes contact with
the store twice. In the first memory contact with AR3 the
count part (mod. 512) is decremented by 1 and the address part ( modulo
218)
is incremented by 1. If the resulting count part = 0, then the word transport
still to follow is the last of the start instruction. In the
memory contact following the one with AR3 the word is
transported, designated by the address part just written away
into AR3. If – through a malfunction – the start instruction
is broken off prematurely, then the count part of AR3 contains
the remaining number of words to be printed, and the address part the
address of the last transported word.

The
prevailing OK-status of the printing unit is remembered in the
count part of AR0; after the completion of each start instruction this
count part is taken over into the count part of the final word (label
[–1]) of the link being completed. The address part of the final word
is undefined.

For
the printing unit the OK-indication has only 2 possible values:

000 000 000      OK

111 111 111      not
operational (i.e. e.g. not connected)

The
status of the Charon program for the printing unit thus knows two
states, denoted briefly as OK and non
OK.
Above
we have described the normal completion of a start instruction when
it is accepted in the state OK. A start instruction which is
accepted in the state non
OK is, with one exception,
immediately completed symbolically. (With report-back of the non
OK status in the
final word). The one exception is the so-called HOK-instruction (Restore OK
state), which can normally be included in the start chain and is
characterized by a code word consisting of nothing but zeros.
(Partly as a consequence of this one cannot
in 1 start instruction send the
content of the words at addresses 1 through 512 to the printing unit:
these would namely be interpreted as a HOK-instruction. The
HOK-instruction must not be given if the device is already in the
OK-state.

NB.
Keyboard and teleprinter printing unit each keep track, independently of
each other, mechanically, of the prevailing shift. For the printing unit
the shift determines which of the two characters appears on paper for a five-bit
print command; for the keyboard the prevailing shift determines which
half of the keyboard is blocked.

6.1. Hardware
code teleprinter EL X8

(Omitted,
see original.)
With
a black ribbon X has as
side effect the starting of a red flickering attention signal which
can only be switched off by the operator.

7.
The
tape reader

The
meaning of the central administration AR0 through AR3 is normal, as
described in 3.

The
start link consists of 3 consecutive words:

label [-1] :

final word, namely:

count part
:
Ok
indication

address part
:
undefined

label [0] :

count part
:
all zeros

address part
:
label
of the next link

label [+1] :
the sole code word,
namely

count part
:

length
of the array (whereby count part =0

prescribes the length =512; provision is thus not made in
the empty start-

instruction)

address part
:
start address of the word array – 1.

If
the start instruction is accepted by Charon, the code word
from the prevailing link is taken over into AR3. Per punching read
Charon makes contact with the store twice. In the first
memory contact with AR3 the count part (mod. 512) is decremented
by 1 and the address part ( mod. 218)
is incremented by 1. If the resulting count part = 0, then the word transport
still to follow is the last of the start instruction. In the
memory contact following the one with AR3 the punching read, filled out
on the most significant side with zeros, is filled in at the
memory location designated by the address part just written away
into AR3.

When
the start instruction is broken off prematurely, AR3 contains, in
the count part, the missing number of punchings, in the
address part the address of the last filled word.
The
prevailing OK-status of the tape reader is remembered in the
count part of AR0. At the end of the processing of a
start instruction the OK-status then prevailing is copied into the
count part of the final word of the link concerned.

the OK-status knows
three values:

000 000 000

111 111 111

111
111 110
OK

not
operational (i.e. e.g. not connected

end of tape

The
last two states are denoted briefly as “non
OK”. If the
tape reader is non
OK, all
following start instructions are, with one exception,
immediately completed symbolically, with report-back of the
prevailing non
OK status in the
final word. The one exception is the so-called HOK-instruction (Restore OK
state), which can normally be included in the start chain. Partly
as a consequence of this one cannot
have read 512
punchings in one start instruction, beginning the storing at address 1.
The HOK-instruction must not
be given if the
tape reader is already OK.

Rem.
1 The
symbolic completion does not disturb AR3, so that the datum of how
much has yet been read before “end of tape” broke off the reading process
is not lost before the next read instruction
is effectively accepted.
Rem.
2 The
tape reader reads at a speed of 1000 punchings per second; the
reading process thus occupies the store for ½ percent of the time.

Rem.
3 The
tape reader contains a switch for various tape widths This
tape width is not deemed to contribute to the information;
program-wise the prevailing tape width is not detectable.

Rem.
4 The
tape reader reads all configurations uninterpreted, including “blank
tape” and “all holes”.

The
tape passes, in order:
1.
a
mechanical detector for end of tape,
2.
the
brake,
3.
the
reading station,
4.
the
drive.

The
ends of the tape can therefore not be read.

The tape reader is
equipped with a red and a green button. By pressing the red button
one stops the tape reader immediately and renders the detector
for end of tape ineffective. By pressing the green button the
tape reader returns to its normal state. The pressing of the
red and green button as such does not penetrate to the base machine:
if a start instruction is (or comes) under handling it merely takes
somewhat long before it is dealt with.
When
end of tape is detected, the tape reader jumps instantaneously into the
state as if the red button had been pressed; the pressing of the
green button has effect only when there still lies a sufficiently long
tape in the tape reader.
Rem.
5 The
red button gives the operator the opportunity, in emergency situations, to
stop the tape reader for a moment (a knot or a kink or whatever) without
this penetrating to the logic of the processing program. He can even,
if he is sufficiently quick, offer a number of tapes one after another
to the processing program as a single tape.

Rem.
6 If
a tape has been read to the end, then, before a next
tape can be read over that tape reader, two things must have happened:

- the
base machine must have given a HOK-instruction.

- the
operator must have concluded the inserting of the next tape
with the pressing of the green button.

The
order of these two actions does not matter.

8.
The
tape punch

The
central administration at the locations AR0 through AR3, as well as the layout
of the start link, is the same for the tape punch as for the
teleprinter printing unit, see 6. (Teleprinter and tape punch are namely
served by exactly the same Charon program.) Now the lowest 7 bits of each
word determine what is punched. (I assume that the
remaining 20 bits are irrelevant; I would always make them =0.)

The tape punch too
has a red and a green button. the pressing of the red button
has as its consequence that the tape punch stops instantaneously and that the
selection for end of tape is temporarily ineffective. By pressing
the green button this is undone again.

When the detection for
the end of tape gives alarm for the first time, there are still at least a few
meters of tape available. In contrast to the tape reader, which
stops instantaneously at end of tape, the tape punch calmly completes
the prevailing start instruction and only then passes into the state “end of
tape”, in which the following start instructions (if any) are
completed symbolically up to the first HOK-instruction.
This
HOK-instruction brings the tape-punching into the closing state. In the
closing state punch instructions are again accepted normally, albeit
that no attention is paid to the cries of distress of the end-of-tape detection.
The closing state was introduced to give the
programmer the opportunity to punch end-indications on the tape;
it is however his responsibility to ensure that
these end-indications are not too long.
After
the last punch instruction of the closing the programmer gives a
second HOK-instruction, which has a twofold consequence. The Charon program
passes again into the normal OK-state and the programmer can offer the
start instructions for the next tape. A second effect
of the second HOK-instruction is that the tape punch itself gets into the state
as if the red button had been pressed, whereby pressing the green
button has effect only after a new tape has been inserted.

Rem.
With
the offering of the start instructions for the next tape the
program thus need not wait until the operator has changed the band.
The
OK-status gives four values:

000 000 000
111 111 111

111
111 110

111
111 101
OK

not
operational

end of tape

closing state

9.
Line printer

The
central administration (AR0 through AR3) as well as the three-word
start link (label [-1] through label [+1]) have the normal function.
The last word of the start link is the code word which indicates, in a manner
to be described further, what kind of line must be printed.

For the specification of
the start instruction one builds up in the store, at a number of
consecutive addresses, the so-called line image. Place and length of
the line image are given in the code word of the start link, namely

code word:
count part
:
number of words of which the line image consists

(1 ≤ this number ≤
36)

address part
:
start address of the line image – 1.

Charon
sends the words of the line image, in order of ascending address,
to the line printer. In each word four six-bit symbols are packed

d26.... d21
1st symbol

d19.... d14
2nd symbol

d12.... d7
3rd symbol

d5.... d0
4th symbol

The
three separation bits d20, d13 and d6 must be =0.

The
action of the line printer is best described in terms
of its reaction to a symbol array defined as above
(whose length is always a multiple of four.)
The
very first symbol of the array is interpreted deviantly
and is the so-called skip indication (see below); thereafter the
symbols of the array specify what the line printer must print on that line,
position by position in the order from left to right. The
maximum number of characters per line is 143. If the line image in the
store is shorter than 36 words, the line printer leaves the
unspecified positions at the right end of the line
unprinted. The length of the symbol array which is sent to the line printer
is by definition a multiple of four; if necessary the
programmer must therefore fill out the array with 1, 2 or 3 spaces.

The meaning of the
skip indication (regarded as a number from 0 through 63) is as follows:

0
through 31
advance the paper
over the indicated number of lines and then print the line,
as specified by the rest of the line image.

32
through 39
line advance
controlled by an eight-hole punched tape, then print.

40
through 63
does not occur

(In
the case of forms of a fixed length one can insert into the line printer
a cyclically glued eight-hole punched tape, which is transported over a
punching with each line. This tape gives 8 channels,
numbered from 0 through 7; the tape-controlled line advance takes place
until in channel nr. “skip indication – 32” a
punching is detected in the tape. With coupling to a
computer this kind of facility is superfluous; I do not intend to
use it.)

The
meaning of the symbols of the array after the skip indication follows from
the following table. The symbol value is regarded as a number from 0 through
63. (roller EL X( of THE.)
(table
omitted, see original document).

Rem.
In the printout the zero is narrower than the letter O.

The normal state
is OK, i.e.

Count part
AR0
= 000    000    000

final word
= +0     (i.e. all zeros)

The
line printer contains a detector “paper low”; if this
signals, the OK-state continues to exist – line-print
instructions are thus accepted normally – but in the
start link there appears

final word = +1 (d26
= ··· = d1 = 0, d0 = 1)

If
the programmer now – i.e. while the state is OK –
gives a HOK-instruction – as always in the form of a code word
equal to nothing but zeros – then on the line printer the
light “Change Paper” lights up. The line printer thereby
gets into the state as if the red button had been pressed; pressing
the green button has effect only when “paper low” gives no
alarm, i.e. when the operator has, if necessary, loaded new paper.

If
the line printer is not connected then:

Count part AR0
= 111 111 111
not
operational

final word
= 111 111 111 ,
followed by an undefined address part

This
state counts as non
OK, and the following
start instructions ≠ HOK are completed symbolically.

Three other causes
can bring forth the state non
OK.
They leave behind:

count part AR0
=
111 111 110 and

final word
:
count part = 111
111 110

address part: d17
= ····· d4
= 0

d3 = 1
if “Yoke
open”

d2 = 1
if paper
torn

d1 = 1
if
parity error in transmission

d0 = 1
if also
“paper low”

Of the bits d3, d2 and
d1 at least 1 must be equal to 1. The parity-error signalling
d1 = 1 concerns the symbol transmission for the line printer: each
six-bit symbol is namely provided with a parity bit and sent to the
line printer. With standard rollers, where fewer than 64
different symbols are allowed in the array, d1 = 1 means
“parity error in transmission or non-existent symbol”.

With every non-empty
combination paper low is also reported in d0 = 1. In that case
the first HOK-instruction restores the OK-state, but the indication
paper low remains hanging. A second HOK-instruction from the machine
then switches on, normally, the light “Change Paper”.

Rem.
1 In case of
non
OK
due to one of the three mentioned causes the reacting program may
ignore d0. If it turns out that “paper low” prevails, one will notice
that indeed at the very next start instruction.

Rem.
2 The
presentation of matters, as if there were on the line printer only a red
and a green button, is somewhat simplistic: there are more
on it. The effect is however that after the HOK-instruction following “paper
low” the base machine can without further ado already offer the next
start instructions; they will only be processed after the operator
has, according to the rules of the art, loaded new paper.

Rem.
3 If
the line-print instruction has been broken off prematurely, the count part
of AR3 contains, as usual, the number of remaining (i.e. not
transported) words and the address part the address of the last
transported word. If the line-print instruction has been completed
normally, the count part contains 511 and the address part the address of
“the first non-transported word”.

10. The
drum

In
the central administration AR0, AR1 and AR2 have the normal
meaning. AR3 is unused. A start instruction is a link
consisting of 5 consecutive words.

label [–1] :

count part
:   copy
of OK-status as in count part of AR0

address part
:   further
specification of final word (see below)

label [0] :

count part
:   000
000 000

address part
:
label address of next link.

label [1] :

d26 through
d19
:   irrelevant, provided not all = 0.

d18 through d0
:
start address on drum

label [2] :

d26
= ··· =
d19 = 0

d18
= 0
transport direction from drum to cores

= 1
transport direction from cores to drum.

label [3] :

d26
= ··· =
d12 = 0

d11 through d0 : length
( thus < 4096)

In
the count part of AR0 there means:

000 000 000
111 111 111

111
111 110
OK

not
operational

something wrong

The
last two states give occasion for symbolic completion
up to the HOK-instruction (Characterized by label[1] = + 0; label [2]
and label [3] do not take part in the HOK-instruction).

In
the final word there is

d26 through d18
: (count part) :
a copy of the count part of AR3.

d17 = … =
d14
: always = 0

d13 = 1
if memory contact
accepted too late

d12 = 1
if parity error.

d11 through d0
final state of
count counter (must = 0).

d13
= 1 or d12 = 1 implies OK-status: 111 111 110.

Ad d13:
The drum places
the core store in an essential hurry situation. If this
(through gross overloading!) could not be honored,
d13 = 1 is set and the transport is interrupted.

Ad d12:
The
drum transports are subject to the normal parity check.
If, with a transport from cores to drum, a
word with incorrect parity stands in the cores, the
transport instruction will give alarm. Trying again is then
guaranteed to be pointless.

Rem.
1 In
“label [1]” at least 1 of the bits must be =1 in order to be able to
distinguish the instruction from the HOK-instruction.

Rem.
2 The
incrementing of the running drum address takes place modulo 2 ↑ 19 :
the drum store is thus cyclically arranged.

Rem.
3 With
a transport from drum to cores which are not there, information
is lost undetected; with a transport from cores which are
not there, the parity check sounds the alarm. The running
core address too counts around ( modulo 2 ↑ 18), but this is not very
interesting.
Rem.
4 In
d 11 through d0 one finds the final state of the counter, i.e. the number
of non-transported words. In case of a fault the failed word counts as
transported: if it is the last word one finds, therefore, d11 = …
= d0 = 0.

11. The clock

The clock is
invariably device nr. 39. It works independently of AF and
has three functions.

11.1
The
actual clock

Every
10 insec AR0 is incremented by 1 modulo 2 ↑ 27. The cycle of
this counting process is of the order of magnitude of two weeks.

11.2
The
code alarm

Every
10 insec the address part of AR1 is decremented by 1 modulo 2 ↑ 18.
If thereby AR1 in total becomes = +0, then in the
count part of AR1 :000 000 001 is set and IF becomes true,
in other words the intervention of the code alarm can take place. Whoever does not want this
intervention can thus suppress it definitively by making the
count part of AR1 immediately ≠. The
maximum running time of the code alarm is about 40 minutes.

11.3
The
device alarm

Modulo
512, every 10 insec the count part of AR2 is decremented by 1. If
the result in the count part becomes = 0 and the address part of AR2 =
NR ≠ 0
then:

- of the device with device number = NR the LVAF is set to true,

- in AR2 AR3 is taken over.

One
initiates this process by filling, once, AR3 and AR2 with the same value.
One can have a designated device take note of something every so often,
with a maximum period of about five seconds.

transcribed
by Martin van der Burgt

revised 2013-06-20
