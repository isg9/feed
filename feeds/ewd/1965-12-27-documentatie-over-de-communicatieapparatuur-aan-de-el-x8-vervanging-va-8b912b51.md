---
title: "Documentatie over de communicatieapparatuur aan de EL X8 (vervanging van EWD140) (EWD149)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD149.html
published: "1965-12-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD149.PDF
---

# Documentatie over de communicatieapparatuur aan de EL X8 (vervanging van EWD140)

7 januari 1966
EWD 149

Documentatie over de communicatie apparatuur aan de EL X8. (Vervanging van EWD140)

0. Inleiding

Er is voorzien in een maximum van 48 zg. Apparaten, genummerd van 0 t/m 47.

Ze vallen uiteen in twee groepen:

nr.0 t/m 31:
de echte apparaten

nr.32 t/m 47:
de administratieve apparaten.

De administratieve apparaten verrichten parallel aan de basismachine
zekere handelingen (zoals “tandenpoetsen” en
“foutafhandeling”); deze apparaten hebben een vast
nummer, dat niet installatie-afhankelijk is.
De echte apparaten in een specifieke installatie ontlenen hun nummer aan
de plaats in de communicatiebekabeling; deze apparaatnummers zijn dus
wel installatie-afhankelijk, kunnen zelfs gewijzigd moeten worden bij
heropstelling.

1. Communicatie opdrachten.

Met elk apparaat zijn drie flipflops geassocieerd.

AF
( = actie Flipflop)

IF
( = Ingreep Flipflop)

LVIF
(= Luistervergunning Ingreep Flipflop).

Zij worden beschouwd als booleans; als hun waarde op bits wordt
afgebeeld, dan wordt true door 1 en false door 0 gerepresenteerd, zodat de logische vermenigvuldiging de
and-functie oplevert.

1.1. Woordsgewijze uitleesopdrachten.

Het opdrachtenrepertoire van de X8 bevat de mogelijkheid het 1ste (2de of 3de) IF- of LVIF-woord uit te lezen.
Tot nu toe is alleen de layout van de 1ste woorden beslist, dwz. de wijze, waarop de bitposities in dit woord aan de apparaten
zijn toegevoegd:

d26
= permanent 0 (dit is het tekenbit)

d25
= toegevoegd aan apparaat nr. 39

:
:

d18
= toegevoegd aan apparaat nr. 32

d17
= toegevoegd aan apparaat nr.   0

:
:

d0
= toegevoegd aan apparaat nr. 17

Bij de woordsgewijze uitleesopdrachten specificeerd men

1) welk register in de uitlezing betrokken is; hier heeft men de keus tussen de registers A en S

2) welk type flipflops uitegelezen moeten worden; hier heeft men de keus tussen de IF's en de LVIF's

3) de keus tussen "schoon in" en "logische vermenigvuldiging"; in het geval
"schoon in" geldt het flipflopwoord zelf als het resultaat, dat bij non-U in het
geselecteerde register verschijnt, in het geval van "logische vermenigvuldiging"
geldt het logische product van het flipflopwoord en de oorspronkelijke inhoud
van het geselecteerde register als het resultaat dat bij non-U in het geselecteerde
register verschijnt

4) 1ste, 2de of 3de flipflopwoord.

De layout van deze opdrachten is als volgt:

d26 t/m d21 = 110 110

A-register

= 111 110

S-register

d14 t/m d9 = 111 101

IF

= 111 110

LVIF

d8 t/m d6 = 000

schoon in

= 001

logische vermenigvuldiging

d5 t/m d0 = 000 000

1ste flipflopwoord

= 000 001

2de flipflopwoord

= 000 010

3de flipflopwoord

Van de adresvarianten is alleen de B-modificatie toegestaan, PZE en UYN werken
normaal. NB. De AF's zijn niet uitleesbaar.

1.2. Bitsgewijze zetopdrachten

De flipflops kunnen individueel gezet worden. Hierbij specificeert men

1) of de nieuwe waarde true of false (1 of 0) moet zijn

2) op welk type flipflop de opdracht betrekking heeft; hier heeft men de keuze
tussen AF, IF en LVIF

3) het apparaatnummer (dwz. het nummer van het apparaat, waarmee deze flipflop
geassocieerd is.)

De layout van deze opdrachten is:

d26 t/m d21 = 110 110

zet flipflop false (=0)

= 111 110

zet flipflop true (=1)

d14 t/m d9 = 111 000

AF

= 111 001

IF

= 111 010

LVIF

d8 t/m d6 = 000

= 001

logische vermenigvuldiging

d5 t/m d0 = 000 000

binaire representatie van het apparaatnummer

Van de adresvarianten is alleen B toegestaan, van de conditievolging alleen
Y en N; U, P, Z, E zijn verboden.

De flipflop IV (Ingreep Vergunning, zie onder) kan gezet worden door de
opdrachten "maak doof" resp. "maak horend", met de coderingen:

d26 t/m d21 = 110 110

IV:= false (=0, maak doof)

= 111 110

IV:= true (=1, maak horend)

d14 t/m d12 = 110

d11 t/m d0 = 000 000 000 000

Hierbij zijn Y en N de enige toegelaten varianten.

2. Betekenis van IF, LVIF, IV en AF.

De IF van een apparaat fungeert ten opzichte van de basismachine als een
soort attentiesignaal; wij zullen later zien onder welke omstandigheden de
voltooiing van een zg. startopdracht voor een communicatieapparaat de assignment
"IF := true (vanzelfsprekend is hier de met het apparaat geassocieerde IF bedoeld)
als neveneffect heeft.

De primaire reactie van de basismachine op zo'n attentiesignaal is de zg.
ingreep, die plaats vindt als:

1) IV (dwz. machine horend) and

2) voor ten minste 1 apparaat geldt "LVIF and IF".

Door het zetten van de LVIF's kan de basismachine reguleren, van welke
apparaten het ingreepmechanisme al dan geen nota neemt; door het zetten van IV (bv.
maak doof) kan de basismachine het optreden van een ingreep generaal verhinderen.

Opm. Wanneer de opdracht (hetzij maak horend, hetzij een herstellende sprong)
de overgang van doof naar horend bewerkstelligt, zal onmiddelijk na deze opdracht
nog geen ingreep optreden, ook al geldt allang voor een of meer appaarten
"LVIF and IF". Na een effectief horend makende opdracht wordt dus gegarandeerd
de volgende opdracht nog aansluitend uitgevoerd, voordat de eerste ingreep kan
optreden. Daarentegen is elke overgang van horend naar doof ten aanzien van de
ingreeponderdrukking onmiddelijk effectief.

De uitvoering van de ingreep bestaat hieruit, dat

1) in plaats van de volgende opdracht de opdracht op adres 24 wordt uitgevoerd
als operand van een DO-opdracht (dwz. de ophoging van OT wordt onderdrukt, omdat
deze opdracht immers niet onder controle van OT is aangehaald)

2) aan het einde van de ingreep wordt de machine doof gemaakt.

Het normale gebruik is, dat de uiteindelijke ingreepopdracht een normale
(dwz. niet stapelende) subroutinesprong is. In de link, die daarbij wordt weggeschreven,
wordt dus nog IV = true genoteerd; het adres in deze link wijst naar
de opdracht uit het onderbroken programma, die net niet meer aan de beurt kwam.

Opm. De doofmaking als onderdeel van de ingreep vindt plaats aan het einde;
zou de uiteindelijke ingreepopdracht (onwaarschijnlijkerwijze) een opdracht blijken te zijn, die de machine horend maakt,
dan worden er aan het einde van
de ingreep twee tegenstrijdige bevelen gegeven ten aanzien van de zetting van IV;
de uiteindelijke waarde van IV moet dan ook als onbepaald beschouwd worden.

In het begin van het ingreepprogramma kan de basismachine —zie 1.1— door inspectie van het logische product van IF— en LVIF-woord onderzoeken, welke IF’s voor deze ingreep verantwoordelijk kunnen zijn geweest.

Wat de IF's zijn voor de basismachine, zijn de AF's voor Charon: wanneer
Charon voor een apparaat geen startopdracht (zie onder) onder behandeling heeft,
legt Charon "AF = true" uit als de aanwezigheid van tenminste ��n nieuwe startopdracht
voor dit apparaat.

3. Vaste toekenning van geheugenplaatsen aan apparaten.

De AF fungeert als attentiesignaal vanuit de basismachine ten behoeve van
een apparaat, de IF fungeert als attentiesignaal vanuit Charon ten behoeve van
de basismachine. De feitelijke informatieuitwisseling tussen basismachine en Charon
vindt plaats via het kerngeheugen, dat zowel voor de basismachine als voor Charon
toegankelijk is (met prioriteit voor Charon).

Voor dit doel zijn voor elk apparaat vier opeenvolgende geheugenadressen gereserveerd,
te beginnen bij adres
64 + 4 * apparaatnummer.

(Dit loopt dus van adres 64 t/m adres 255.) In het volgende zullen deze vier
woorden worden aangeduid met AR0, AR1, AR2 en AR3.

In het volgende zullen van een woord de 9 meest significante bits enerzijds
en de 18 minst significante bits anderzijds vaak als onafhankelijke bestanddelen
beschouwd worden; het stuk d26 t/m d18 heet dan "het tellingdeel", het stuk
van d17 t/m d0 heet dan "het adresdeel". Als wij spreken over de numerieke waarden
van deze delen, dan bedoelen we de non-negatieve getallen met deze 9, resp. 18
bits als binaire representatie.

Vooruitlopend geven wij de algemene functie van de vier woorden, zij het,
dat we slechts een eerste aanduiding van de betekenis kunnen geven.

Opm. Deze betekenissen zijn niet bij alle apparaten van toepassing; uitzonderingen
zullen bij de apparaatsgewijze behandeling vermeld worden.

AR0:

tellingdeel
:
werkruimte Charon, in het bijzonder gereserveerd voor de OK-status

adresdeel:

label van de heersende schakel

AR1:

tellingdeel
:
if IFT then IFT + 512 else IFT

adresdeel
:
irrelevant, mits ≠ 0 (wordt uit compalibiliteitsoverwegingen met de X2 door Charon niet gebruikt)

AR2:

tellingdeel
:
=AFT

adresdeel
:
meestal werkruimte voor Charon

AR3:

Werkruimte voor Charon (vaak gereserveerd voor het zg. "lopende codewoord")

4. Ketens van startopdrachten

4.1. Layout van de startopdracht.

Per apparaat kan men een keten van startopdrachten "aanhangig maken".
Elke startopdracht �omdat ze geketend kunnen worden ook wel "schakel" genoemd�
beslaat een aantal opeenvolgende geheugenplaatsen. Onder de "label van een schakel"
verstaat men het adres van het tweede woord van deze schakel.

De algemene layout van een startopdracht begint als volgt:

label[-1]:

Slotwoord. V��r de feitelijke uitvoering van deze startopdracht, is de
inhoud van dit woord voor Charon irrelevant; na afloop van de uitvoering
heeft Charon in dit woord verslaglegging gedaan over de afloop
van de startopdracht.

label[ 0]:

bij de laatste schakel van een keten voor Charon irrelevant;

bij een schakel, die niet de laatste van de keten is, eist Charon

tellingdeel
: allemaal nullen

adresdeel
: label van de volgende schakel.

label[ 1]:

1ste van een rij van 1 of meer codewoorden ter beschrijving van de
startopdracht; aantal en layout hiervan is apparaat-afhankelijk

De basismachine moet de startopdracht klaarmaken in een vorm, die specifiek
is voor het apparaat in kwestie. De startopdracht is niet positioneel gebonden
aan het apparaat, noch aan het gebied van het kerngeheugen, waarop de feitelijke
informatietransport betrekking zal hebben. Omdat voor de labels altijd 18 bits
gereserveerd zijn, is er geen hardware restrictie aan hun plaats opgelegd.

4.2. Het aanbieden van startopdrachten.

Deze sectie behandelt hoe in het algemeen startopdrachten moeten worden
aangeboden; de opstelling van de startopdracht zelf —in het bijzonder van de
codewoorden— wordt uitgesteld tot de apparaatgewijze gehandeling. Omdat de
regels, die bij het aanbod in acht genomen moeten worden, uiteindelijk hun
oorsprong vinden in het gedrag van Charon, zullen wij eerst beschrijven hoe Charon
overgaat tot het accepteren van een nieuwe startopdracht.

4.2.1. Charongedrag ten aanzien van AF en AFT.

In het volgende zullen wij de activiteiten van Charon kortheidshalve beschrijven
alsof Charon niets anders te doen heeft dan het verzorgen van een enkel
apparaat: het feit dat Charon meer apparaten kan verzorgen betekent dat de hieronder
te beschrijven Charon-reacties niet instantaan hoeven plaats te vinden, nl. niet
wanneer Charon net even met iets anders nuttigs bezig is. Met "rust" van Charon
wordt hieronder bedoeld "rust ten aanzien van dit apparaat".

Als Charon geen startopdrachten onder behandeling heeft, zal hij rusten, zolang AF false is.

Zodra AF true is, zal Charon dan een startopdracht in behandeing nemen, en
wel degene, waarnaar verwezen wordt door het adresdeel van AR0: label van heersende schakel.

Aan het eind van de behandeling van de startopdracht zal Charon als
ondeelbare handeling
AFT := AFT - 1 (mod 512); if AFT = 0 then AF := false

uitvoeren. De ondeelbaarheid van de aftrekking wordt gegarandeerd, doordat Charon
de terugschrijf-halve-cyclus van het kerngeheugen gebruikt om de nieuwe waarde
van AFT in het adresdeel van AR2 terug te schrijven en doordat het kerngeheugen
slechts per hele cyclus aan de verschillende processen gegund wordt. Verder wordt
als onderdeel van de AFT-behandeling de assignment "AF := false" uitgevoerd, als
AFT de waarde = 0 gekregen heeft; anders blijft AF onaangetast.

Als de nieuwe waarde van AFT ≠ 0 was —en AF dus onbeinvoed is gebleven— zal Charon
adresdeel AR0 := M[ adresdeel AR0]

uitvoeren, dwz. de label van de volgende schakel tot "label van heersende schakel"
bevorderen.

Daarna (zie IFT-behandeling, later) wordt de behandeling van de startopdracht
voltooid, deze voltooing aan de basismachine teruggemeld en tenslotte gaat Charon
over in de uitgangstoestand van deze sectie "geen startopdracht onder behandeling".
De vorige is nl. voltooid en de volgende is nog niet in behandeling genomen.

4.2.2. Een enkele schakel voor een passief apparaat.

Bij het simpelste gebruik van Charon biedt de basismachine slechts de volgende
schakel aan, nadat de basismachine (via IFT en IF, zie later) van de voltooiing
van de vorige schakel heeft kennisgenomen en Charon dus (ten aanzien van dit apparaat) in
rust is.

In de aangeboden schakel is de inhoud van "label[0]"label van de volgende
schakel- omdat er geen volgende schakel is, irrelevant.

De basismachine moet er voor zorgen, dat in het adresdeel van AR0 de label
van de aangeboden schakel staat —om Charon in staat te stellen de schakel te vinden—,
moet er voor zorgen, dat AFT = 1 —om te zorgen, dat Charon na afloop AFT = 0
maakt en in de rusttestand overgaat— en tenslotte moet de basismachine AF op true zetten, opdat Charon metterdaad aan de startopdracht begint.

Opm. De heersende label in het adresdeel van AR0 wordt hierbij niet verstoord; als
alle aan te bieden startopdrachten op dezelfde plaats staan, hoeft het tellingdeel
van AR0 van te voren slechts ��n keer ingevuld te worden.

4.2.3. Een keten voor een passief apparaat.

Wil men in plaats van een enkele schakel (zie vorige sectie) een keten van
schakels aan een op dat moment passief apparaat aanbieden, dan moet de basismachine
er voor zorgen, dat het adresdeel van AR0 de label van de eerste schakel bevat,
dat de volgende schakels netjes aan hun voorganger gehaakt zijn, dat AFT gelijk is
aan het aantal schakels —om er voor te zorgen, dat Charon de keten netjes afwerkt
en dan ophoudt— en tenslotte zet men AF op true opdat Charon metterdaad aan de
keten begint.

Opm. In dit geval blijft het adresdeel van AR0, de label van de heersende schakel— niet ongewijzigd.

4.2.4. Het aanhaken van een schakel aan een mogelijk nog actief apparaat.

Wij zullen nu een methode behandelen om een schakel aan te haken aan een keten
die mogelijkerwijs nog niet is afgewerkt. Omdat Charon ook op de AFT, de AF
en de AR0 zou kunnen opereren, moet dit wel met enige voorzorgen geschieden.

We nemen aan dat een zogenaamde vulwijzer "VW" de label van de laatst aangehaakte schakel bevat.
Als de nieuwe label volledig is geprepareerd —label[0] is daarin
irrelevant— vervolgt de basismachine
1) met "M[VW] := label nieuwe schakel" wordt de nieuwe schakel aan de keten
gehaakt (het ingevulde woord bevat in het tellingdeel louter nullen);
2) met "VW" := label nieuwe schakel" wordt de vulwijzer up to date gemaakt.
3) met een additieve uitopdracht in de versie "PLUS" wordt AFT met 1 opgehoogd,
zodat de hierbij gevormde nieuwe waarde van AFT tevens in een register van de
basismachine verschijnt en geinspecteerd kan worden. Voor de ophoging gebruikt
de basismachine een additieve uitopdracht, omdat zodoende gegarandeerd wordt, dat
de ophoging van de basismachine en een mogelijke aflaging door Charon niet met
elkaar op oncontroleerbare (in elk geval ongewenste) wijze met elkaar kunnen
interfereren.
4) indien de door de PLUS-opdracht in het register van de basismachine opgevangen
nieuwe AFT-waarde = 1 is, dan was de oude AFT-waarde dus = 0 en heeft Charon dus
besloten uit deze keten geen nieuwe startopdracht meer aan te halen; de handeling
onder 1) genoemd heeft geen effect gehad en Charon heeft AF op false gezet. In dit
geval vervolgt de basismachine aldus:

4.1.)

in het adresdeel van AR0 wordt de label van de nieuwe schakel ingevuld
zonder het tellingdeel van AR0 te verstoren. We zijn in de toestand, dat
Charon besloten heeft, de keten met rust te laten, wat betekent,
dat Charon het adresdeel van AR0 verder wel met rust zal laten, maar het
tellingdeel van AR0 mogelijkerwijs nog niet. Het invullen van de label
van de nieuwe schakel in het adresdeel van AR0 zonder het bijbehorende
tellingdeel te verstoren kan de basismachine verrichten met een additieve uit-opdracht
bij AR0 op te tellen.

4.2.)

AF := true . NB . De assignment "AF := true" mag alleen uitgevoerd worden
in dit geval, waarin we dus hebben vastgesteld, dat Charon "AF := false"
had uitgevoerd. Wie redeneert "We hebben iets nieuws aangeboden, dus
AF mag in elk gevalop true gezet worden." vergist zich,
want Charon zou denieuwe startopdracht tegen deze tijd al
voltooid kunnen hebben en terecht al AF op false gezet kunnen hebben.

Opm. Het is niet aantrekkelijk om te proberen, de activiteit ten aanzien van een
keten tijdelijk stil te leggen, door (wat de basismachine wel kan) AF := false te
zetten. Charon's beslissing om op grond van AF = true een nieuwe startopdracht
aan te halen laat opzichzelf geen sporen in het kerngeheugen achter. Zo de basismachine
er achter kan komen welke startopdracht nog wel aangehaald is en welke
gegarandeerd niet meer aangehaald zal worden, dan zal dat niet zonder real-time
overwegingen kunnen.

Opm. Handeling 1) dient achterwege te blijven als er nog geen laatst aangehaakte
schakel is (aan het begin bv.) of als de geheugenruimte, destijds door de laatst
aangehaakte schakel ingenomen, inmiddels een heel andere bestemming heeft gekregen (zie 4.3.1).

4.3. De terugmeldingen van voltooiingen.

Als 0 ≤ tellingdeel AR1 ≤ 255, dan wordt het tellingdeel van AR1
geinterpreteerd als IFT; als 256 ≤ tellingdeel AR1 ≤ 511, dan wordt het tellingdeel van AR1
geinterpreteerd als IFT + 512. Het waardebereik van IFT is dus -256 ≤ IFT ≤ 255.
IFT ≥ 0 is gekarakteriseerd door d26=0.

4.3.1. Het gedrag van Charon ten aanzien van IFT en IF.

Charon beeindigt de behandeling van een startopdracht met in een
ondeelbare handeling
IFT := iFT + 1 (mod 512); if IFT > 0 then IF := true

uit te voeren.

De ondeelbaarheid van de additie wordt weer gerealiseerd door in de
terugschrijf-halvecyclus de opgehoogde waarde van IFT terug te schrijven.

Charon verricht deze handeling, nadat AFT is afgelaagd, de label van de
heersende schakel eventueel is opgestapt en slotwoord en OK-status (zie onder)
zijn ingevuld. De IFT-ophoging is de signalering, dat Charon de afgewerkte
schakel niet meer zal raadplegen; de basismachine mag hierna de door de afgewerkte
schakel belegde geheugenplaatsen dus een andere bestemming geven.

4.3.2. IFT-aflaging door de basismachine.

Tegenover de IFT-ophoging door Charon moeten IFT-aflagingen van de
basismachine staan. Door deze met behulp van een additieve uitopdracht uit te voeren,
verzekert men, dat deze aflagingen niet interfereren met de eventuele ophogingen
door Charon. Doordat het adresdeel van AR1 ≠ 0 is, bereikt men, dat IFT = 0
door deze additieve uitopdracht inderdaad correct met een tellingdeel uit
louter nullen wordt afgeleverd.

Het gedrag van Charon is er op gericht om "IF" de betekenis te geven
"IFT > 0": als IF deze betekenis heeft, blijft hij bij Charon-verhogingen
van IFT (aangenomen, dat de capaciteit van IFT niet overschreden wordt)
gehandhaafd. Als de basismachine IFT gewijzigd heeft en men stelt er prijs op,
dat daarna IF weer "IFT > 0" zal betekenen, dan kan de basismachine als volgt
te werk gaan:

" IF := false;

if IFT > 0 then IF := true"

De analyse "IFT > 0" kan worden uitgevoerd door te testen, of van AR1
d26 = 0 en het tellingdeel ≠ 0 is. Nauwkeurige analyse leert, dat er geen bezwaar
tegen zou zijn, wanneer voor deze test het tellingdeel van AR1 tweemaal uit het
geheugen geselecteerd zou worden, ook al zou Charon onderhand IFT ophogen.
(De veronderstelling, dat de basismachine na "IF := false" IFT ongemoeid laat,
is essentieel; het protocol, zoals in EWD140 gegeven is fout, wanneer IFT ook
negatieve waarden mag aannemen!)

Wanneer men begint met IFT = 0 en als onderdeel van het kennisnemen van
een voltooing van een startopdracht IFT met 1 aflaagt, dan heeft IFT de betekenis
van het aantal reeds voltooide schakels, waarvan de basismachine nog geen
kennis heeft genomen. Door IFT een negatieve voorgift te geven, kan men desgewenst
bereiken, dat IF pas true gezet wordt (en de ingreep tengevolge hiervan
dus pas optreedt) na voltooing van de zoveelste startopdracht uit de keten.
In al deze gevallen mogen we veilig aannemen, dat het beperkte bereik van de
AFT en de IFT nimmer een beperking zal zijn.

5. Het toetsenbord.

Het toetsenbord is een invoerapparaat, waarvan het nummer
installatie-afhankelijk is.

Het is een ernstig afwijkend apparaat: het verricht zijn activiteit nl. niet
op commando van de door de basismachine gegeven startopdrachten, maar doordat de
operateur een toets aanslat. Charon maakt bij het toetsenbord geen gebruik van AR2.

Als de operateur een toets aanslaat, verricht Charon:

if AF and LVAF then AR0 := karakter else AR3 := karakter; AF := false;

IFT := IFT + 1 (mod 512); if 0 then IF := true

(LVAF is een interne boolean van Charon, die in dit geval permanent = true
geacht wordt te zijn; zie verder "Het tandenpoetsen.".)

Door aanvankelijk IFT (= tellingdeel AR1) = 0 te maken en (in overeenstemming
met de normale betekenis) IF = false te zetten en LVIF = true, bereikt men
dat de basismachine via het ingreepmechanisme van het indrukken van een toets
op de hoogte wordt gesteld, zodra de basismachine horend is. Door bovendien
AF = true te zetten, verzorgt men, dat het eerste karakter in AR0 komt en eventueel
volgende karakters (met overschrijving van de vorige) in AR3 terechtkomen.

(Het vullen van AR3 geschiedt, wanneer de basismachine niet tijdig op de
ingreep na het vorige karakter gereageerd heeft; in het onwaarschijnlijkegeval,
dat Charon nog geen kans heeft gezien, om het vorige karakter in het geheugen te
plaatsen, hebben de te snel op elkaar volgende aanslagen het effect, dat Charon
als bovenbeschreven ��n karakter in het geheugen plaatst, maar nu
een zg. non-valide met een getalwaarde groter dan 31.)

L. Zwanenburg heeft een programma gemaakt, waarin de tijd in alternerende
perioden verdeeld wordt;

periode a): verwerking van een enkel of verwerping van een of meer karakters,
afgesloten door de indicatie, dat nu de volgende aanslag mag plaatsvinden;

periode b): de tijd, waarin een aanslag mag plaatsvinden, afgesloten door de
reactie op de ingreep;

en wel zodanig, dat in de periode a) slechts dan een karakter wordt verwerkt, als
in de voorgaande periode a) geen en in de daaropvolgende periode b) precies
��n aanslag heeft plaatsgevonden.

Bij de reactie op de ingree�p van het toetsenbord gaat de machine dan als volgt te werk

L0: IF := false; IFT := IFT - 1; if IFT ≠ 0 then

begin AR0 := iets negatiefs; goto L0 end;

A := AR0; if A > 0 then

begin AR0 := iets negatiefs;

A is nu het volgende karakter, dat

geaccepteerd wordt end

else er zijn karakters verworpen, wat

desgewenst kan worden aangegeven;

AF := true;

comment nu kan de operateur weer een karakter in

AR0 krijgen;6. Het teleprinterdrukwerk.

De indeling voor AR0, AR1, AR2 en AR3 is normaal (zie 3.); het adresdeel van
AR2 wordt niet gebruikt. Het lopend codewoord in AR3 is gesplitst in tellingdeel en adresdeel (zie onder).

Een startschakel geeft de mogelijkheid om de woorden uit N opeenvolgende
adressen (1 ≤ N ≤ 512) in volgorde naar het drukwerk te sturen. Van elk woord
bepalen de laagste zes bits een actie van het drukwerk; de 21 hoogste bits van deze woorden zijn irrelevant.

De startschakel bestaat uit drie woorden; de eerste twee (label [-1] en label[0])
zijn normaal (zie 4.1.), het derde woord (label[1]) bevat het zg. codewoord.
Het codewoord bepaalt lengte en ligging van de naar het drukwerk te zenden
woordenrij en wel:

tellingdeel codewoord =
lengte van de rij, (waarbij tellingdeel = 0 de rijlengte =512 voorschrijft; zodat dus niet in de lege startopdracht is voorzien)

adresdeel codewoord  =beginadres van de woordenrij � 1.

Als de startopdacht door Charon geaccepteerd wordt, wordt het codewoord
uit de heersende schakel in het lopende codewoord (in AR3 dus) overgenomen. Per
woordverzending naar het drukwerk neemt Charon tweemaal contact met het geheugen
op. In het eerste geheugencontact met AR3 wordt het tellingdeel (mod. 512) met 1
verlaagd en wordt het adresdeel ( modulo 2 ↑ 18) met 1 verhoogd. Is het resulterend
tellingdeel = 0, dan is het nu nog volgende woordtransport naar het drukwerk
het laatste van de startopdracht. In het geheugencontact, volgend op dat met
AR3 wordt het woord getransporteerd, aangewezen door het zojuist in AR3 weggeschreven
adresdeel. De afsluitingsriten van de schakelbehandeling ( zoals AFT-aflaging
etc. en IFT-ophoging etc.) vinden in Charon plaats, zodra het laatste
woord naar het drukwerk is gezonden. Charon meldt de startopdracht dus als
voltooid, terwijl het drukwerk nog 100 msec bezig is met het feitelijk drukken van
het laatst aangeboden karakter. Daar de feitelijke woordtransporten naar het
drukwerk steeds pas plaatsvinden, nadat Charon van de teleprinter het signaal
ontvangen heeft, dat het volgende woord gestuurd mag worden, is een en ander
(zoals te verwachten) geen beletsel om meteende volgende startopdracht aan te
bieden, noch voor Charon, om deze alvast te accepteren.

Daar de afwerking van de startopdracht afhankelijk is van de terugsignalering
van het drukwerk naar Charon, zou een startopdracht nimmer eindigen, als deze
terugsignalering achterwege blijft, zoals dat het geval zou zijn, als het drukwerk
helemaal niet is aangesloten. Charon initieert om deze reden alleen het transport
van een woordrij, nadat hij (naar eer en geweten) geverifieerd heeft, dat het
drukwerk volgens de regels van de kunst is aangesloten en naar behoren
functioneert.

Een en ander wordt als volgt gerealiseerd. Het Charonprogramma, dat het
drukwerk verzorgt, kent twee toestanden, genaamd "OK resp. "NBK" (Niet Bedrijfs
Klaar). Vlak voordat dit programma in de toestand van OK een nieuwe startopdracht
zal gaan aanhalen, test Charon, of het drukwerk goed is aangesloten; zo ja, dan
wordt de startopdracht als boven afgehandeld, zo nee, dan gaat het verzorgingsprogramma
prompt over in de toestand NBK, beschouwt deze opdracht als
afgehandeld en gaat over tot de afsluitingsriten. Als het verzorgingsprogramma
zich in het begin van het aanhalen van de volgende schakel al in de toestand
NBK bevindt, dan worden alle startopdrachten zg. symbolisch afgewerkt —dwz.
Charon gaat meteen tot de afsluitingsriten over— met uitzondering van de zg.
HOK-opdracht (Herstel OK-toestand). De uitvoering van de HOK-opdracht bestaat
nl. daaruit, dat het verzorgingsprogramma van het drukwerk in de toestand OK
gebracht wordt.

De twee toestanden van het verzorgingsprogramma worden onderscheiden in het
tellingdeel van AR0 en wel:

tellingdeel AR0
= 000 000 000
betekent OK

= 111 111 111
betekent NBK.

De afsluitriten van een startopdracht omvatten tevens het invullen van het
zg. slotwoord (in label[-1]); in het tellingdeel van het slotwoord van de
schakel in voltooing vult Charon een copie van het tellingdeel van AR0 in; de waarde,
die aan het adresdeel van het slotwoord wordt toegekend is ongedefinieerd.

De HOK-opdracht wordt gecodeerd door een codewoord (label[1]), bestaande uit
louter nullen. De uitgevoerde HOK-opdracht krijgt in zijn schakel een slotwoord,
bestaande uit louter nullen.

Opm 1. Het codewoord = + 0 wordt altijd als een HOK-opdracht en nimmer als
transportopdracht uitgelegd. Hierdoor is het niet mogelijk om in ��n
startopdracht een woordenrij van 512 woorden te transporteren, te beginnen op adres 1.

Opm 2. De HOK-opdracht mag gegeven worden, terwijl het verzorgingsprogramma zich
reeds in de toestand OK bevindt; hij heeft dan geen effect. Het aanhaken van
dergelijke "overbodige" HOK-opdrachten aan nog actieve ketens moet in het algemeen
niet gestimuleerd worden: in het algemeen (dwz. bij ingewikkeldere apparaten) kan
nl. de OK-toestand bij de uitvoering van de nog voorafgaande schakels verlaten
worden en de oorspronkelijk overbodige HOK-opdracht zou dan plotseling wel effect
krijgen, en waarschijnlijk een ongewenst effect!

Opm 3. Het codewoord van de startschakel, waaraan het verzorgingsprogramma in
de OK-toestand zou willen beginnen, maar wat niet doorgaat, omdat het drukwerk
zich niet bevredigend aan Charon voordoet, zodat de overgang naar NBK
plaatsvindt, wordt niet geraadpleegd. Als dit een HOK-opdracht was, raakt het
verzorgingsprogramma dus toch in de NBK-toestand en deze HOK-opdracht krijgt dan
niet het slotwoord = +0.

6.1. Controlelichten op het teleprinterdrukwerk.

6.1.1. Rode en groene knop.

De besturing van het teleprinterdrukwerk kent twee toestanden, genaamd
"rood" respectievelijk "groen". De heersende toestand wordt aangegeven door het
branden van een rood, resp. groen controlelichtje. Verder bevinden zich
bij de teleprinter een rode en een groene knop; door een van deze beide in te
drukken kan men de teleprinter in de overeenkomstige toestand brengen.

Boven hebben wij de teleprinter beschreven in de groene toestand.
In de rode toestand zendt de teleprinter geen synchronisatiesignalen naar Charon.
Zodoende kan men door op de rode knop te drukken het teleprinterdrukwerk even
stilleggen —bv. om het papier te verschikken— en daarna, door het drukken op de
groene knop het teleprinterdrukwerk weer vrijgeven en wel zonder dat deze ingreep
tot Charon (en dus evenmin tot de basismachine) is doorgedrongen. (Het drukwerk
heeft zich even gedragen als een heel langzaam apparaat, maar "real time"
overwegingen spelen geen rol in de logica van het samenspel tussen basismachine en
Charon, noch tussen Charon en de randapparatuur.)

Zodra Charon metterdaad wacht op een dergelijke synchronisatiesignalering,
die voorlopig achterwege blijft, omdat het apparaat zich in de rode toestand
bevindt, wordt de operateur hierop attent gemaakt, doordat het bijbehorende rode
lampje gaat flikkeren.

6.1.2. Het papiercontact.

Als in het teleprinterdrukwerk het papier dreigt op te raken, wordt dit
mechanisch gedetecteerd. Dit heeft tot gevolg, dat bij einde van het papier

1) een speciaal waarschuwingslichtje op de teleprinter geat branden

2) de besturing van het teleprinterdrukwerk overgaat in de rode toestand.

6.1.3. Het attentiesignaal.

Als het ijzeren kruis is gedrukt en 100 msec later de lintkleur zwart is,
dan gaat een wit attentiesignaal flikkeren, dat door de opereteur kan worden
uitgezet. Door het ijzeren kruis nooit door een lintkleurverandering te laten
volgen, kan men er voor zorgen, dat het flikkerlicht alleen na een zwart ijzeren
kruis volgt.

7. De bandlezer.

De betekenis van de centrale administratie AR0 t/m AR3 is normaal (zie 3.);
de layout van de startschakel, die drie woorden omvat is als in 4.1 beschreven,
de betekenis van het codewoord in label[1] is:

label[1]:
codewoord

als codewoord bestaat uit louter nullen, dan HOK-opdracht, anders

tellingdeel =
lengte van de rij (waarbij tellingdeel = 0

de lengte = 512 voorschrijft)

adresdeel =
beginadres van de woordenrij - 1

Het Charonprogramma, dat de bandlezer verzorgt kent drie toestanden, genaamd
OK, NBK (Niet Bedrijfs Klaar) en EB (Einde Band). De toestand van het verzorgingsprogramma
wordt ten bate van Charon in het tellingdeel van AR0 bijgehouden. Ten
bate van de basismachine, die na afloop van elke startopdracht moet kunnen vaststellen,
hoe deze is uitgevoerd, wordt in het tellingdeel van elk slotwoord de
op het moment van de invulling heersende toestand genoteerd. In beide tellingdelen
geldt de codering:

000 000 000

OK

111 111 111

NBK
(Nist Bedrijfs Klaar)

111 111 110

EB
(Einde Band)

De overgang naar de toestand NBK vindt (zie 6.) weer plaats, wanneer
het verzorgingsprogramma, vlak voordat het in de toestand OK een startopdracht
zou aanhalen, de aanwezigheid van de correct aangesloten bandlezer niet
naar tevredenheid kan vaststellen. Van deze startopdracht wordt het codewoord
dan niet geraadplaagd en Charon gaat direct over tot de afsluitingsriten (met
vermelding van de inmiddels heersende NBK in het slotwoord).

In de toestanden NBK en EB worden startopdrachten symbolisch afgewerkt, dwz.
Charon kijkt, of het codewoord de HOK-opdracht is, zoja, dan gaat het verzorgingsprogramma
over in de toestand OK (met vermelding van een slotwoord = +0), zo nee,
dan wordt de opdracht meteen afgesloten met vermelding van de toestand
(NBK of EB) in het slotwoord.

Wanneer de OK-status geldt en de bandlezer lijkt zijn correcte aanwezigheid
te bevestigen, dan wordt de nieuwe startopdracht normaal verwerkt. Is hij een
(overbodige) HOK-opdracht, dan wordt hij als zodanig uitgevoerd, anders wordt
het feitelijk bandlezen geinitieerd. Bij de acceptering van de startopdracht wordt
het codewoord uit de heersende schakel in AR3 overgenomen. Per gelezen ponsing
neemt Charon tweemaal contact met het geheugen op. In het eerste geheugencontact
met AR3 (het zg. lopende codewoord) wordt het tellingdeel (mod 512) met 1 verlaagd
en het adresdeel (mod 218) met 1 verhoogd. Is het resulterende tellingdeel
= 0, dan is het nu volgende woordtransport het laatste van de startopdracht. In
het geheugencontact, volgend op dat met AR3 wordt de gelezen ponsing, aan de meest
significante zijde aangevuld met nullen, ingevuld op de geheugenplaats, die wordt aangewezen door het zojuist in AR3 weggeschreven adresdeel. Tevens wordt
bij het lezen van elke ponsing geinspecteerd of de detectie van het einde van de
band alarm geeft, zo ja, dan gaat het verzorgingsprogramma tot de afsluitingsriten
van deze startopdracht over, na eerst de toestand EB heersend te hebben gemaakt.

Opm. Er wordt dus altijd tenminste 1 ponsing ten gevolge van de geaccepteerde
transportopdracht gelezen. Na elke ponsing wordt eerst gevraagd naar het bandeinde�
zo nee, dan wordt pas getest, of de afsluiting tengevolge van tellingdeel = 0
aan de beurt is. Als het slotwoord EB vermeldt, kan de startopdrecht dus
wel volledig zijn uitgevoerd! Het symbolisch afwerken verstoort AR3 niet:
wanneer de startopdracht op grond van Einde Band is beeindigd� bevindt zich in
het tellingdeel van AR3 het ontbrekende aantal pons�ngen, in het adresdeel het
adres van het laatst gevulde woord.

De bandlezer leest met een snelheid van 1000 ponsingen per seconde; afgezien
van initiering� en afsluitriten belegt het leesproces het geheugen van de
machine voor .625 procent van de tijd.

De bandlezer kan ingesteld worden op het lezen van 5-, 7- of 8-gats band.
Deze bandbreedte wordt niet geacht bij te dragen tot de informatie, programmatechnische
is de heersende bandbreedte niet detecteerbaar. In het geheugen wordt
een gat in een ponsing door een 1 en geen gat door een 0 voorgesteld; de individuele
posities in een ponsing worden in volgorde op de minst significante bitposit�es
van het woord afgebeeld, en wel zo, dat de sporen, toegevoegd aan d2 en d3 aan
beide zijden van de 'sprocket holes" liggen. De bandlezer leest alle configuaties
onge�nterpreteerd, ook "blank tape" en "all holes�.

NB. In de bandlezer zitten buffers voor twee ponsingen. Dit heeft tot gevolg,
dat na bandverw�sselen de eerste twee ponsingen� die tot Charon doordringen,
de laatste twee van de vorige band zijn. Na aanzetten van de machine zijn de
eerste twee gelezen ponsingen ongedefinieerd.

De band passeert in volgorde

1) een detector voor het einde van de band

2) de rem

3) het leesstation

4) de aandrijving.

De uiteinden van een band kunnen daardoor niet gelezen worden.

De bandlezer is eveneens uitgerust met een rode en een groene knop en bijbehorende
controlelichtjes; ook hier flikkert het rode licht als Charon metterdaad
wacht op een achterwege blijvend synchronisatiesignaal van de bandlezer
("een hangende startopdracht"). Wanneer Charon op grond van het tot hem
doorgedrongen signaal "Einde Band" een startopdracht afbreekt voert hij tevens de
bandlezer in kwestie over in de rode toestand, het indrukken van de groene
knop heeft slechts effect, wanneer er nog een voldoend lange band in de bandlezer
ligt. Het indrukken van de rode en de groene knop dringt als zodanig niet tot
de machine door, evenmin als het band verwisselen in een rode bandlezer.

Als er over een bandlezer een band tot en met het einde gelezen is, moeten
er, voordat een volgende band over die bandlezer gelezen kan worden, twee dingen
gebeurd zijn:

1) de basismachine moet een HOK-opdracht aan deze bandlezer hebben aangeboden

2) de operateur moet een nieuwe band hebben ingelegd en dit met het drukken op
de groene knop daarna hebben bekrachtigd.

De volgorde van deze twee gebeurtenissen doet niet ter zake.
8. De bandponser.

De centrale administratie op de plaatsen AR0 t/m AR3, evenals de layout
van de startschakel zijn bij de bandponser even normaal als bij het teleprinter drukwerk
(zie 6). Teleprinterdrukwerk en bandponser worden nl. door identieke
Charon-programma's verzorgd. Nu bepalen de laagste 7 bits van het woord, dat
naar de ponser gestuurd wordt, wat er wordt geponst� de overige 20 zijn irrelevant.
De toevoeging van de bitposities in het woord aan de sporen op de band is
als bij de bandlezer.

Het Charonprogramma� dat de bandponser verzorgt, kent vier toestanden,
genaamd OK, NBK (Niet Bedrijfs Klaar)� EB (Einde Band) en ASL (AfSLuit�ng). De
toestand van het verzorgingsprogramma wordt ten bate van Charon in het tellingdeel
van AR0 bijgehouden. Ten bate van de basismachine� die na afloop van elke
startopdracht moet kunnen vaststellen, hoe deze is uitgevoerd, wordt in het
tellingdeel van elk slotwoord de op het moment van invulling heersende toestand
genoteerd. In beide tellingdelen geldt de codering:

000 000 000

OK

111 111 111

NBK (Niet Bedrijfs Klaar)

111 111 110

EB (Einde Band)

111 111 101

ASL (AfSLuit�ng)

De overgang naar de toestand NOK vindt (zie 6) weer slechts plaats, wanneer
het verzorgingsprogramma� vlak voordat het in de toestand OK een startopdracht
zou gaan aanhalen, de aanwezigheid van een correct aangesloten bandponser niet
naar tevredenheid kan vaststellen. Van deze startopdracht wordt het codewoord
niet geraadpleegd en Charon gaat direct over tot de afsluitingsriten (met vermelding
van de inmiddels heersende NBK in het slotwoord).

De overgang naar de toestand EB vindt plaats, wanneer het verzorgingsprogramma
in de OK�toestand aan een nieuwe startopdracht wil gaan beginnen, de bandponser
zijn correcte aanwezigheid lijkt te bevestigen maar de detector op het einde van
de band alarm geeft. Van deze startopdracht wordt het codewoord niet geraadpleegd
en Charon gaat direct over tot de afsluitingsriten (met vermelding van
de inmiddels heersende EB in het slotwoord).

In de toestanden NBK en EB worden startopdrachten symbolisch afgewerkt,
dwz. Charon kijkt, of het codewoord de HOK-opdracht is, zo nee, dan wordt de
opdracht meteen afgesloten met vermelding van de heersende toestand (NBK of EB)
in het slotwoord, zo ja, dan bewerkt de HOK-opdracht bij NBK de overgang naar
OK (met slotwoord = + 0) en bij EB de overgang naar ASL (met vermelding van de
inmiddels heersende ASL in het slotwoord).

Niet symbolische afwerking treedt op, wanneer het verzorgingsprogramma
in de toestand OK of ASL aan een nieuwe startopdracht wil beginnen. De enige
verschillen tussen deze twee toestanden zijn

1)
de test van correct aangesloten zijn blijft bij ASL achterwege, de overgang
ASL → NBK komt dus niet voor

2)
de test van alarm van de detector van het einde van de band blijft
bij ASL achterwege, de overgang ASL → EB komt dus niet voor

3)
bij ASL heeft de HOK�opdracht het neveneffect, dat de bandponser in de rode
toestand komt en een lichtje "Band verwisselen" op de bandponser gaat branden.
Voor de verwerking van de feitelijke ponsopdracht (in toestand OK of ASL) verwijzen
we naar sectie 6, het teleprinterdrukwerk.

Als het einde van de band nadert, krijgt de laatst normaal verwerkte ponsopdracht
nog de OK�melding in het slotwoord, van de eerste met de EB-melding is
niets meer geponst. Met een HOK�opdracht krijgt de basismachine het Charonprogramma
in de afsluitingstoestand� waarin alarm van de einde�band�detect�e genegeerd
wordt. De programmeur heeft dan nog de gelegenheid (minstens anderhalve meter
band) om een speciale afsluitcombinatie op de band te ponsen (deze mag uit meer
dan een startschakel zijn opgebouwd). Een volgende HOK�opdracht brengt het verzorgingsprogramma weer in de OK�toestand, (en wat de basismach�ne
betreft kunnen de volgende startopdrachten alweer aangeboden worden), de bandponser
is echter nog rood, totdat de operateur een nieuwe rol heeft ingelegd
(inclusief het resetten van het EB-contact) en op de
groene knop heeft gedrukt.

Opm. In de rode toestand wordt alarm van de einde�band�detector niet aan Charon
doorgegeven. Wanneer na de tweede HOK�opdracht de basismachine alweer opdrachten
voor de volgende band aanbiedt, kan als de operateur nog geen band verwisseld heeft
het einde van de eerste band dus niet voor dat van de tweede aangezien worden.
Het drukken op de groene knop heeft geen effect� wanneer het lichtje "Band Verwisselen"
nog brandt.

9. De klok.

De klok is onveranderlijk apparaat nr. 39. Hij werkt onafhankelijk van de
AF en heeft drie functies.

9.1. De eigenlijke klok.

Elke 10 msec wordt AR0 (mod 227) met 1 verhoogd. De cyclus van dit
telproces is van de orde van grootte van twee weken.
Gelijkzetten van de klok: tandenpoetsen (NLVAF and NLVCA), gelijkzetten, poetsen (NLVAF and LVCA)

9.2. De kookwekker.

Elke 10 msec wordt het adresdeel van AR1 (mod 218) met 1 verminderd. Als
AR1 hierdoor totaal = + 0 wordt � wordt het tellingdeel van AR1 =000 000 001
gezet en IF:= true, maw. de ingreep van de kookwekker kan plaatsvinden. Wie deze
ingreep niet wil hebben, kan hem dus definitief onderdrukken door het tellingdeel
van AR1 meteen ≠ 0 te maken. De maximale looptijd van de kookwekker is
circa 40 minuten.

9.3. De apparaatwekker.

Elke 10 msec wordt het tellingdeel van AR2 met 1 (mod 512) verminderd.
Als het resulterende tellingdeel = 0 wordt en het adresdeel van AR2 = NR ≠ 0
is, dan wordt

1) van het apparaat met apparaatnummer = NR de LVAF = true gezet

2) AR3 in AR2 overgenomen.

Men initieert dit proces door eenmalig AR2 en AR3 met hetzelfde te vullen.
Men kan met de apparaatwekker een aangewezen apparaat om de zoveel tijd ergens
nota van laten nemen, met een maximum periode van circa vijf seconden, een faciliteit,
die in sommige on line toepassingen van interesse is. Door in AR2 bv.
+ 0 in te vullen schakelt men de apparaatwekker uit.

10. Het aanschakelen van de machine.

Als de machine wordt aangezet, dan komt hij in de toestand, die ook te
bereiken is door het indrukken van LS (Laatste Schakelaar) op het wegklapbare
gedeelte van het bedieningspaneel. Dit heeft de volgende effecten:

Alle randapparatuur komt in de rode toestand.

Charon komt in zijn rusttoestand en doet niets.

De basismachine komt in de zg. statische stop, dwz. doet ook niets en is
zelfs voor het ingreepmechanisme niet ontvankelijk.

De regulerende flipflops hebben na afloop de volgende waarden

IV = true (Ingreep Vergunning)

OV = false (Onderzoek Vergunning)

alle AF = false

alle IF = false

alle LVIF = true

alle LVAF = true (Luister Vergunning AF, interne Charon flipflops)

alle LVCA = false (Luister Vergunning Communicatie Aankondiging, eveneens interne
flipflops van Charon)

In deze installatietoestand (en alleen in deze installatietoestand) heeft
het indrukken van hetzij IP (Initieel Programma) hetzij NB(Normaal Begin) effect. De bespreking
van het effect van de toets NB wordt uitgesteld tot de behandeling van het
tandenpoetsen.

10.1 Het Prille Begin.

Het indrukken ven de toets IP heeft tot effect. dat de basismachine overgaat
in de dynamische stoptoestand, dwz. hij doet nog steeds niets maar is nu wel
voor ingrepen ontvankelijk; daar (zie 10) alle IF�s false zijn, treedt het ingreepmechanisme
voorlopig niet in werking. In T wordt de waarde 218 - 1 gezet.

Verder wordt Charon gestart in een speciaal programma —genaamd het prille
begin— dat via een vaste, installatieafhankelijke bandlezer op een heel speciale
manier band gaat lezen.
Op deze band komen woorden voor, als volgt gecodeerd in vier opeenvolgende
heptades; de leesrichting is van boven naar beneden, de positie van de sprocket
holes is door wat extra ruimte weergegeven. Weer stelt een gat een 1 voor.

1 d26 d25 d24 d23 d22 d21
d20 d19 d18 d17 d16 d15 d14
d13 d12 d11 d1O d9 d8 d7
d6 d5 d4 d3 d2 d1 d0
Voor en achter aan de band kieze men een behoorlijk stuk blank.
Het leesprogramma skipt eventuele extra blanke heptades voor elk woord; het
eerste woord, dat van de band gelezen wordt, wordt ge�nterpreteerd als codewoord,
zij het dat de opeenvolgende woordtransporten naar het geheugen nu niet
betrekking hebben op enkele heptades naar op hele woorden. Zodra deze startopdracht
is afgelopen keert het prille begin terug naar de toestand, waarin het
eerst volgende van de band gelezen woord weer als codewoord wordt ge�nterpreteerd.

Het IP-lezen wordt door Charon beeindigd� zodra hij een codewoord
bestaande uit louter nullen zou vinden, dwz. op het goede moment de configuratie

1000 000
0000 000
0000 000
0000 000
Het beeindigen heeft voor Charon ten gevolge� dat hij in de normale
rusttoestand ("niets te doen") overgaat(tijdens het IP-lezen was zijn toestand
nog wel een beetje bijzonder); de betrokken bandlezer stopt en blijft groen
met LVAF = true (poetsen is niet noodzakelijk). Tenslotte zet Charon
onafhankelijk van IFT van de bijbehorende bandlezer de IF = true. Tenzij SVA
(Stop Volgende Adres) in staat, vindt in de basismachine nu onmiddelijk
de normale ingreep plaats via M[24]. (Staat SVA wel in, dan blijft de
�ngreep hangen met in de opdrachtteller allemaal enen (2↑18 - 1), totdat de
machine wordt doorgestart.)

NB. Aangezien het lezen van het prille begin aan geen enkele vorm van
pariteitscontrole is onderworpen� verdient het aanbeveling, met het prille
begin niet al te veel meer dan nodig in te lezen.

10.2 De tandenpoetser.

De tandenpoetser is voor alle installaties apparaat nr. 38. Men start de
tandenpoetser door (na aanbieding van een startopdracht) zijn AF op true te
zetten, hij meldt voltooiing van zijn actie door zijn IF op true te zetten.
Dit alles gaat buiten AFT en IFT om, die bij de tendenpoetser niet bestaan.
(AR1, AR2 en AR3 worden niet gebruikt door het tandenpoetsprogramma).

Startopdrachten voor de tandenpoetser kunnen niet geketend worden, men
plaatst ze stuk voor stuk in AR0. De betekenis van de bits van AR0 is dan:

d26
t/m d20:

alle = 0LVCA =: false

niet
alle = 0LVCA =: true

d19
t/m d18:
beide permanent = 0

d17
t/m d11:

alle = 0LVAF =: false

niet
alle = 0LVAF =: true

d10
t/m d6:
irrelevant

d5
t/m d0:
nummer van het te poetsenapparaat.

Het poetsen van een apparaat betekent

1) dat de electronische besturing van het apparaat in de �thuisstand" wordt
gebracht

2) dat de LVAF en de LVCA voor dit apparaat gezet worden volgens specificatie
van de startopdracht.

De LVAF's en LVCA's zijn interne flipflops van Charon en voor elk apparaat
is er zo'n paar. De LVAF (Luister Vergunning AF) bestuurt of Charon een actie
zal ondernemen op het true zijn van de bijbehorende AF; de LVCA (Luister Vergunnning
Communicatie Aankondiging) bestuurt, of Charon een actie zal ondernemen op
de aanwezigheid van de bijbehorende zg. Communicatie Aankondiging (een "synchronisatiesignalering"
van het aparaat naar Charon).

Voor de klok moet LVCA true zijn en is de LVAF irrelevant, voor het
toetsenbord moeten LVCA en LVAF beide true zijn, voor de tandenpoetser moet
LVAF true zijn (laat hem dus zo); voor "normale apparaten" (teleprinterdrukwerk,
bandponser� bandlezer� regeldrukker� plotter, trommel) moet LVAF true
en LVCA false zijn.

NB. Het tandenpoetsen verzorgt geen initiering van de centrale administraties,
dit moet dus rechtstreeks door de basismachine gedaan worden

11. De foutbehandeling.

De foutbehandeling is altijd apparaat nr. 37. De betekenis van de centrale
administratie is

AR0:basisadres centrale administratie eerste fout

AR1tellingdeel: IFT

adresdeel : ongelijk nul

AR2ongebruikt

AR3:basisadres centrale administratie volgende fout.

Zodra in een van de Charonprogramma's een pariteitsfout wordt gesignaleerd
bij het aanhalen van een woord uit het geheugen, dan wordt

1) het woord met de foute pariteit in het geheugen teruggeschreven, net zoals
het daar werd aangetroffen

2) het heersende Charonprogramma, na LVCA:= LVAF:= false (zodat het tot nader
order niet meer actief zal worden) onmiddellijk onderbroken

3) onder meegave van het adres van de AR0 behorende bij het onderbroken Charonverzorgingsprogramma
wordt de foutbehandel�ng onmiddellijk gestart door
het true maken van de LVCA.

Dit basisadres van de centrale administratie wordt door de foutbehandeling
verwerkt als een karakter van het toetsenbord door het toetsenbordprogramma:

if AF and LVAF then AR0:= basisadres else AR3 := basisadres; AF := false;

IFT:= IFT + 1 (mod 512); if 0 then IFT:= true

De reactie op de ingreep ven de foutbehandeling stelt de besismchine voor
een essentiele haastsituatie; bij cumulatie van fouten raakt de basismachine het
spoor bijster.

Opmerking. Als er bij de tandenpoetser een pariteitsfout optreedt, is men
op LS aangewezen om weer greep op de machine te krijgen!

- - - - - - - - - - - - - - - - - - - -- - - - - - - - - - -- - - - -

Toevoeging en 10.2. Na aanzetten of LS kan men, behalve op IP ook drukken op NB
Men forceert dan IF T := 218 - 1 van de tandenpoetser true en krijgt een ingreep via M[24].

Transcribed
by Martin van der Burgt

Revised
16-Aug-2013
