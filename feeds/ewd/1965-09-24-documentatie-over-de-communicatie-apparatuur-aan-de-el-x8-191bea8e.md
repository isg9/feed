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
pas daarna AF = true
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

000 000 000      OK

111 111 111      niet
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
= 000    000    000

slotwoord
= +0     (d.w.z. allemaal nullen)

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
:   copie
van OK-status als in tellingdeel van AR0

adresdeel
:   nadere
specificatie slotwoord (zie onder)

label [0] :

tellingdeel
:   000
000 000

adresdeel
:
labeladres volgende schakel.

label [1] :

d26 t/m
d19
:   irrelevant, mits niet alle = 0.

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
