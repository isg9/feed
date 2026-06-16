---
title: "Documentatie over de communicatieapparatuur aan de EL X8 (vervanging van EWD140) (EWD149)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD149.html
published: "1965-12-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD149.PDF
---

# Documentatie over de communicatieapparatuur aan de EL X8 (vervanging van EWD140)

7 januari 1966
EWD 149

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
= toegevoegd aan apparaat nr.   0

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

adresdeel codewoord  =beginadres van de woordenrij � 1.

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

---

## English translation

### Documentation on the communication equipment of the EL X8 (replacement of EWD140)

7 January 1966
EWD 149

Documentation on the communication equipment of the EL X8. (Replacement of EWD140)

0. Introduction

A maximum of 48 so-called Devices has been provided for, numbered 0 through 47.

They fall into two groups:

no. 0 through 31:
the real devices

no. 32 through 47:
the administrative devices.

The administrative devices perform certain operations in parallel with the basic machine (such as "tooth-brushing" and "fault-handling"); these devices have a fixed number, which is not installation-dependent.
The real devices in a specific installation derive their number from the place in the communication cabling; these device numbers are therefore installation-dependent, may even have to be changed upon re-installation.

1. Communication instructions.

Three flipflops are associated with each device.

AF
( = Action Flipflop)

IF
( = Interrupt Flipflop)

LVIF
(= Listening-Permission Interrupt Flipflop).

They are regarded as booleans; if their value is mapped onto bits, then true is represented by 1 and false by 0, so that the logical multiplication yields the and-function.

1.1. Word-wise read-out instructions.

The instruction repertoire of the X8 contains the possibility of reading out the 1st (2nd or 3rd) IF- or LVIF-word.
Up to now only the layout of the 1st words has been decided, i.e. the manner in which the bit positions in this word are assigned to the devices:

d26
= permanently 0 (this is the sign bit)

d25
= assigned to device no. 39

:
:

d18
= assigned to device no. 32

d17
= assigned to device no.   0

:
:

d0
= assigned to device no. 17

For the word-wise read-out instructions one specifies

1) which register is involved in the read-out; here one has the choice between the registers A and S

2) which type of flipflops are to be read out; here one has the choice between the IF's and the LVIF's

3) the choice between "clean in" and "logical multiplication"; in the case of "clean in" the flipflop word itself counts as the result, which upon non-U appears in the selected register; in the case of "logical multiplication" the logical product of the flipflop word and the original content of the selected register counts as the result that upon non-U appears in the selected register

4) 1st, 2nd or 3rd flipflop word.

The layout of these instructions is as follows:

d26 through d21 = 110 110

A-register

= 111 110

S-register

d14 through d9 = 111 101

IF

= 111 110

LVIF

d8 through d6 = 000

clean in

= 001

logical multiplication

d5 through d0 = 000 000

1st flipflop word

= 000 001

2nd flipflop word

= 000 010

3rd flipflop word

Of the address variants only the B-modification is permitted, PZE and UYN work normally. NB. The AF's are not readable.

1.2. Bit-wise set instructions

The flipflops can be set individually. Here one specifies

1) whether the new value must be true or false (1 or 0)

2) which type of flipflop the instruction pertains to; here one has the choice between AF, IF and LVIF

3) the device number (i.e. the number of the device with which this flipflop is associated.)

The layout of these instructions is:

d26 through d21 = 110 110

set flipflop false (=0)

= 111 110

set flipflop true (=1)

d14 through d9 = 111 000

AF

= 111 001

IF

= 111 010

LVIF

d8 through d6 = 000

= 001

logical multiplication

d5 through d0 = 000 000

binary representation of the device number

Of the address variants only B is permitted, of the condition-following only Y and N; U, P, Z, E are forbidden.

The flipflop IV (Interrupt Permission, see below) can be set by the instructions "make deaf" and "make hearing" respectively, with the codings:

d26 through d21 = 110 110

IV:= false (=0, make deaf)

= 111 110

IV:= true (=1, make hearing)

d14 through d12 = 110

d11 through d0 = 000 000 000 000

Here Y and N are the only permitted variants.

2. Meaning of IF, LVIF, IV and AF.

The IF of a device functions, with respect to the basic machine, as a kind of attention signal; we shall see later under which circumstances the completion of a so-called start instruction for a communication device has the assignment "IF := true (of course the IF associated with the device is meant here) as a side effect.

The primary reaction of the basic machine to such an attention signal is the so-called interrupt, which takes place if:

1) IV (i.e. machine hearing) and

2) for at least 1 device "LVIF and IF" holds.

By the setting of the LVIF's the basic machine can regulate of which devices the interrupt mechanism does or does not take note; by the setting of IV (e.g. make deaf) the basic machine can generally prevent the occurrence of an interrupt.

Rem. When the instruction (be it make hearing, or be it a restoring jump) effects the transition from deaf to hearing, no interrupt will yet occur immediately after this instruction, even if "LVIF and IF" has long held for one or more devices. After an instruction that effectively makes hearing, the next instruction is therefore guaranteed still to be executed immediately afterwards, before the first interrupt can occur. By contrast, every transition from hearing to deaf is immediately effective with respect to the suppression of the interrupt.

The execution of the interrupt consists of the fact that

1) instead of the next instruction the instruction at address 24 is executed as the operand of a DO-instruction (i.e. the incrementing of OT is suppressed, since this instruction was after all not fetched under the control of OT)

2) at the end of the interrupt the machine is made deaf.

The normal usage is that the final interrupt instruction is a normal (i.e. non-stacking) subroutine jump. In the link that is thereby written away, IV = true is therefore still recorded; the address in this link points to the instruction from the interrupted program that just no longer got its turn.

Rem. The making-deaf as part of the interrupt takes place at the end; should the final interrupt instruction (improbably) turn out to be an instruction that makes the machine hearing, then at the end of the interrupt two contradictory commands are given with respect to the setting of IV; the final value of IV must therefore be regarded as undetermined.

At the beginning of the interrupt program the basic machine can —see 1.1— by inspection of the logical product of the IF- and LVIF-word investigate which IF's may have been responsible for this interrupt.

What the IF's are for the basic machine, the AF's are for Charon: when Charon has no start instruction (see below) under treatment for a device, Charon interprets "AF = true" as the presence of at least one new start instruction for this device.

3. Fixed assignment of memory locations to devices.

The AF functions as an attention signal from the basic machine for the benefit of a device, the IF functions as an attention signal from Charon for the benefit of the basic machine. The actual exchange of information between the basic machine and Charon takes place via the core store, which is accessible both to the basic machine and to Charon (with priority for Charon).

For this purpose four consecutive memory addresses have been reserved for each device, beginning at address
64 + 4 * device number.

(This therefore runs from address 64 through address 255.) In what follows these four words will be denoted by AR0, AR1, AR2 and AR3.

In what follows the 9 most significant bits on the one hand and the 18 least significant bits on the other hand of a word will often be regarded as independent components; the part d26 through d18 is then called "the count part", the part from d17 through d0 is then called "the address part". When we speak of the numerical values of these parts, we mean the non-negative numbers with these 9, respectively 18 bits as binary representation.

By way of anticipation we give the general function of the four words, albeit that we can only give a first indication of the meaning.

Rem. These meanings do not apply to all devices; exceptions will be mentioned in the device-by-device treatment.

AR0:

count part
:
working space Charon, in particular reserved for the OK-status

address part:

label of the prevailing link

AR1:

count part
:
if IFT then IFT + 512 else IFT

address part
:
irrelevant, provided ≠ 0 (is not used by Charon for reasons of compatibility with the X2)

AR2:

count part
:
=AFT

address part
:
mostly working space for Charon

AR3:

Working space for Charon (often reserved for the so-called "current code word")

4. Chains of start instructions

4.1. Layout of the start instruction.

For each device one can "lay pending" a chain of start instructions.
Each start instruction —because they can be chained also called "link"— occupies a number of consecutive memory locations. By the "label of a link" one understands the address of the second word of this link.

The general layout of a start instruction begins as follows:

label[-1]:

Closing word. Before the actual execution of this start instruction, the content of this word is irrelevant to Charon; after the conclusion of the execution Charon has made a report in this word about the outcome of the start instruction.

label[ 0]:

at the last link of a chain irrelevant to Charon;

at a link that is not the last of the chain, Charon requires

count part
: all zeros

address part
: label of the following link.

label[ 1]:

1st of a row of 1 or more code words for the description of the start instruction; their number and layout is device-dependent

The basic machine must prepare the start instruction in a form that is specific to the device in question. The start instruction is not positionally bound to the device, nor to the region of the core store to which the actual information transport will pertain. Because 18 bits are always reserved for the labels, no hardware restriction is imposed on their place.

4.2. The offering of start instructions.

This section treats how start instructions must in general be offered; the composition of the start instruction itself —in particular of the code words— is postponed to the device-by-device treatment. Because the rules that must be observed at the offering ultimately find their origin in the behaviour of Charon, we shall first describe how Charon proceeds to the accepting of a new start instruction.

4.2.1. Charon behaviour with respect to AF and AFT.

In what follows we shall describe the activities of Charon, for the sake of brevity, as if Charon has nothing else to do than serving a single device: the fact that Charon can serve more devices means that the Charon reactions to be described below need not take place instantaneously, namely not when Charon is just busy for a moment with something else useful. By "rest" of Charon is understood below "rest with respect to this device".

If Charon has no start instructions under treatment, he will rest as long as AF is false.

As soon as AF is true, Charon will then take a start instruction into treatment, namely the one referred to by the address part of AR0: label of prevailing link.

At the end of the treatment of the start instruction Charon will execute as an indivisible operation
AFT := AFT - 1 (mod 512); if AFT = 0 then AF := false

The indivisibility of the subtraction is guaranteed in that Charon uses the rewrite-half-cycle of the core store to write the new value of AFT back into the address part of AR2 and in that the core store is granted to the various processes only per whole cycle. Further, as part of the AFT-treatment the assignment "AF := false" is executed if AFT has obtained the value = 0; otherwise AF remains untouched.

If the new value of AFT was ≠ 0 —and AF therefore has remained uninfluenced— Charon will execute
address part AR0 := M[ address part AR0]

i.e. promote the label of the following link to "label of prevailing link".

Thereafter (see IFT-treatment, later) the treatment of the start instruction is completed, this completion is reported back to the basic machine, and finally Charon passes into the initial state of this section "no start instruction under treatment". The previous one is namely completed and the next one has not yet been taken into treatment.

4.2.2. A single link for a passive device.

In the simplest use of Charon the basic machine offers the next link only after the basic machine (via IFT and IF, see later) has taken note of the completion of the previous link and Charon is therefore (with respect to this device) at rest.

In the offered link the content of "label[0]" —label of the following link— is, because there is no following link, irrelevant.

The basic machine must see to it that in the address part of AR0 there stands the label of the offered link —in order to enable Charon to find the link—, must see to it that AFT = 1 —in order to ensure that Charon afterwards makes AFT = 0 and passes into the resting state— and finally the basic machine must set AF to true, in order that Charon actually begin upon the start instruction.

Rem. The prevailing label in the address part of AR0 is here not disturbed; if all start instructions to be offered stand in the same place, the count part of AR0 need only be filled in once beforehand.

4.2.3. A chain for a passive device.

If, instead of a single link (see previous section), one wishes to offer a chain of links to a device that is passive at that moment, then the basic machine must see to it that the address part of AR0 contains the label of the first link, that the following links are neatly hooked onto their predecessor, that AFT equals the number of links —in order to ensure that Charon works through the chain neatly and then stops— and finally one sets AF to true in order that Charon actually begin upon the chain.

Rem. In this case the address part of AR0 —the label of the prevailing link— does not remain unchanged.

4.2.4. The hooking-on of a link to a possibly still active device.

We shall now treat a method for hooking a link onto a chain that has possibly not yet been worked through. Because Charon might also operate on the AFT, the AF and the AR0, this must indeed be done with some precautions.

We assume that a so-called fill-pointer "VW" contains the label of the most recently hooked-on link.
When the new label is fully prepared —label[0] is in it irrelevant— the basic machine proceeds
1) with "M[VW] := label new link" the new link is hooked onto the chain (the filled-in word contains in the count part nothing but zeros);
2) with "VW := label new link" the fill-pointer is brought up to date.
3) with an additive out-instruction in the version "PLUS" AFT is incremented by 1, so that the new value of AFT thereby formed also appears in a register of the basic machine and can be inspected. For the incrementing the basic machine uses an additive out-instruction, because thereby it is guaranteed that the incrementing by the basic machine and a possible decrementing by Charon cannot interfere with each other in an uncontrollable (in any case undesirable) manner.
4) if the new AFT-value caught by the PLUS-instruction in the register of the basic machine is = 1, then the old AFT-value was therefore = 0 and Charon has therefore decided to fetch no more new start instruction from this chain; the operation mentioned under 1) has had no effect and Charon has set AF to false. In this case the basic machine proceeds thus:

4.1.)

in the address part of AR0 the label of the new link is filled in without disturbing the count part of AR0. We are in the state that Charon has decided to leave the chain at rest, which means that Charon will indeed leave the address part of AR0 at rest henceforth, but possibly not yet the count part of AR0. The filling-in of the label of the new link into the address part of AR0 without disturbing the accompanying count part can be performed by the basic machine by adding, with an additive out-instruction, to AR0.

4.2.)

AF := true . NB . The assignment "AF := true" may only be executed in this case, in which we have thus established that Charon had executed "AF := false". Whoever reasons "We have offered something new, so AF may in any case be set to true." is mistaken, for Charon could have already completed the new start instruction by this time and could rightly have already set AF to false.

Rem. It is not attractive to try to lay the activity with respect to a chain temporarily idle by (which the basic machine can indeed do) setting AF := false. Charon's decision to fetch a new start instruction on the grounds of AF = true leaves of itself no traces in the core store. If the basic machine can find out which start instruction has still been fetched and which is guaranteed no longer to be fetched, then that will not be possible without real-time considerations.

Rem. Operation 1) should be omitted if there is not yet a most recently hooked-on link (at the beginning, e.g.) or if the memory space, occupied at the time by the most recently hooked-on link, has meanwhile received a quite different destination (see 4.3.1).

4.3. The reports of completions.

If 0 ≤ count part AR1 ≤ 255, then the count part of AR1 is interpreted as IFT; if 256 ≤ count part AR1 ≤ 511, then the count part of AR1 is interpreted as IFT + 512. The value range of IFT is therefore -256 ≤ IFT ≤ 255. IFT ≥ 0 is characterized by d26=0.

4.3.1. The behaviour of Charon with respect to IFT and IF.

Charon ends the treatment of a start instruction by executing in an indivisible operation
IFT := iFT + 1 (mod 512); if IFT > 0 then IF := true

The indivisibility of the addition is again realized by writing back the incremented value of IFT in the rewrite-half-cycle.

Charon performs this operation after AFT has been decremented, the label of the prevailing link has possibly been stepped up, and closing word and OK-status (see below) have been filled in. The IFT-incrementing is the signalling that Charon will no longer consult the worked-through link; the basic machine may hereafter therefore give a different destination to the memory locations occupied by the worked-through link.

4.3.2. IFT-decrementing by the basic machine.

Over against the IFT-incrementing by Charon there must stand IFT-decrementings by the basic machine. By performing these with the aid of an additive out-instruction, one ensures that these decrementings do not interfere with the possible incrementings by Charon. By virtue of the address part of AR1 being ≠ 0, one achieves that IFT = 0 is indeed correctly delivered by this additive out-instruction with a count part of nothing but zeros.

The behaviour of Charon is directed at giving "IF" the meaning "IFT > 0": if IF has this meaning, it is maintained under Charon-incrementings of IFT (assuming that the capacity of IFT is not exceeded). If the basic machine has changed IFT and one sets store by the fact that IF will thereafter again mean "IFT > 0", then the basic machine can proceed as follows:

" IF := false;

if IFT > 0 then IF := true"

The analysis "IFT > 0" can be performed by testing whether of AR1 d26 = 0 and the count part is ≠ 0. Precise analysis teaches that there would be no objection if for this test the count part of AR1 were selected twice from the store, even should Charon meanwhile increment IFT. (The supposition that the basic machine leaves IFT alone after "IF := false" is essential; the protocol as given in EWD140 is wrong if IFT may also assume negative values!)

When one begins with IFT = 0 and, as part of taking note of a completion of a start instruction, decrements IFT by 1, then IFT has the meaning of the number of already completed links of which the basic machine has not yet taken note. By giving IFT a negative pre-loading, one can if desired achieve that IF is only set to true (and the interrupt consequently therefore only occurs) after completion of the so-manieth start instruction of the chain. In all these cases we may safely assume that the limited range of the AFT and the IFT will never be a limitation.

5. The keyboard.

The keyboard is an input device whose number is installation-dependent.

It is a seriously deviant device: it performs its activity namely not on command of the start instructions given by the basic machine, but by the operator striking a key. Charon makes no use of AR2 with the keyboard.

When the operator strikes a key, Charon performs:

if AF and LVAF then AR0 := character else AR3 := character; AF := false;

IFT := IFT + 1 (mod 512); if 0 then IF := true

(LVAF is an internal boolean of Charon, which in this case is deemed to be permanently = true; see further "The tooth-brushing.".)

By initially making IFT (= count part AR1) = 0 and (in accordance with the normal meaning) setting IF = false and LVIF = true, one achieves that the basic machine is informed via the interrupt mechanism of the pressing of a key as soon as the basic machine is hearing. By additionally setting AF = true, one arranges that the first character comes into AR0 and possible following characters (with overwriting of the previous ones) end up in AR3.

(The filling of AR3 happens when the basic machine has not reacted in time to the interrupt after the previous character; in the improbable case that Charon has not yet had a chance to place the previous character in the store, the strikes following each other too quickly have the effect that Charon, as described above, places one character in the store, but now a so-called non-valid one with a numerical value greater than 31.)

L. Zwanenburg has made a program in which time is divided into alternating periods;

period a): processing of a single character or rejection of one or more characters, concluded by the indication that now the next strike may take place;

period b): the time in which a strike may take place, concluded by the reaction to the interrupt;

and indeed in such a way that in period a) a character is processed only if in the preceding period a) no, and in the subsequent period b) exactly one strike has taken place.

In the reaction to the interrupt of the keyboard the machine then proceeds as follows

L0: IF := false; IFT := IFT - 1; if IFT ≠ 0 then

begin AR0 := something negative; goto L0 end;

A := AR0; if A > 0 then

begin AR0 := something negative;

A is now the next character that

is accepted end

else characters have been rejected, which

can be indicated if desired;

AF := true;

comment now the operator can again get a character into

AR0;6. The teleprinter printing.

The arrangement for AR0, AR1, AR2 and AR3 is normal (see 3.); the address part of AR2 is not used. The current code word in AR3 is split into count part and address part (see below).

A start link gives the possibility of sending the words from N consecutive addresses (1 ≤ N ≤ 512) in order to the printing. Of each word the lowest six bits determine an action of the printing; the 21 highest bits of these words are irrelevant.

The start link consists of three words; the first two (label [-1] and label[0]) are normal (see 4.1.), the third word (label[1]) contains the so-called code word. The code word determines length and location of the row of words to be sent to the printing, namely:

count part code word =
length of the row, (where count part = 0 prescribes the row length = 512; so that the empty start instruction is therefore not provided for)

address part code word  = start address of the row of words − 1.

When the start instruction is accepted by Charon, the code word from the prevailing link is taken over into the current code word (in AR3 therefore). Per word transmission to the printing Charon takes contact with the store twice. In the first store contact with AR3 the count part (mod. 512) is decreased by 1 and the address part (modulo 2 ↑ 18) is increased by 1. If the resulting count part = 0, then the word transport to the printing that still follows now is the last of the start instruction. In the store contact following that with AR3 the word is transported that is indicated by the address part just written away in AR3. The concluding rites of the link treatment (such as AFT-decrementing etc. and IFT-incrementing etc.) take place in Charon as soon as the last word has been sent to the printing. Charon therefore reports the start instruction as completed while the printing is still busy for 100 msec with the actual printing of the last offered character. Since the actual word transports to the printing always take place only after Charon has received from the teleprinter the signal that the next word may be sent, all this (as is to be expected) is no impediment to offering the next start instruction at once, nor for Charon to accept it in advance.

Since the working-through of the start instruction depends on the back-signalling from the printing to Charon, a start instruction would never end if this back-signalling were to be absent, as would be the case if the printing were not connected at all. For this reason Charon initiates the transport of a row of words only after it has (in good faith) verified that the printing is connected according to the rules of the art and functions properly.

All this is realized as follows. The Charon program that serves the printing knows two states, called "OK" respectively "NBK" (Not Operationally Ready). Just before this program is going to fetch a new start instruction in the state of OK, Charon tests whether the printing is properly connected; if so, then the start instruction is handled as above, if not, then the service program promptly passes into the state NBK, regards this instruction as handled, and passes to the concluding rites. If the service program already finds itself in the state NBK at the beginning of the fetching of the next link, then all start instructions are so-called symbolically worked through —i.e. Charon passes at once to the concluding rites— with the exception of the so-called HOK-instruction (Restore OK-state). The execution of the HOK-instruction consists namely of the fact that the service program of the printing is brought into the state OK.

The two states of the service program are distinguished in the count part of AR0, namely:

count part AR0
= 000 000 000
means OK

= 111 111 111
means NBK.

The concluding rites of a start instruction also comprise the filling-in of the so-called closing word (in label[-1]); in the count part of the closing word of the link in completion Charon fills in a copy of the count part of AR0; the value that is assigned to the address part of the closing word is undefined.

The HOK-instruction is coded by a code word (label[1]) consisting of nothing but zeros. The executed HOK-instruction gets in its link a closing word consisting of nothing but zeros.

Rem 1. The code word = + 0 is always interpreted as a HOK-instruction and never as a transport instruction. Hereby it is not possible to transport in one start instruction a row of words of 512 words, beginning at address 1.

Rem 2. The HOK-instruction may be given while the service program already finds itself in the state OK; it then has no effect. The hooking-on of such "superfluous" HOK-instructions to still active chains should in general not be encouraged: in general (i.e. with more complicated devices) the OK-state can namely be left during the execution of the still preceding links, and the originally superfluous HOK-instruction would then suddenly get an effect after all, and probably an undesired effect!

Rem 3. The code word of the start link upon which the service program would like to begin in the OK-state, but which does not go through because the printing does not present itself satisfactorily to Charon, so that the transition to NBK takes place, is not consulted. If this was a HOK-instruction, the service program thus gets into the NBK-state anyway and this HOK-instruction then does not get the closing word = +0.

6.1. Indicator lights on the teleprinter printing.

6.1.1. Red and green button.

The control of the teleprinter printing knows two states, called "red" respectively "green". The prevailing state is indicated by the burning of a red, respectively green indicator light. Furthermore there are at the teleprinter a red and a green button; by pressing one of these two one can bring the teleprinter into the corresponding state.

Above we have described the teleprinter in the green state. In the red state the teleprinter sends no synchronization signals to Charon. Thus one can, by pressing the red button, lay the teleprinter printing idle for a moment —e.g. to adjust the paper— and thereafter, by pressing the green button, release the teleprinter printing again, and indeed without this intervention having penetrated to Charon (and therefore neither to the basic machine). (The printing has behaved for a moment like a very slow device, but "real time" considerations play no role in the logic of the interplay between the basic machine and Charon, nor between Charon and the peripheral equipment.)

As soon as Charon is actually waiting for such a synchronization signalling that for the time being is absent because the device finds itself in the red state, the operator is made attentive to this by the accompanying red lamp beginning to flicker.

6.1.2. The paper contact.

If in the teleprinter printing the paper threatens to run out, this is detected mechanically. This has the result that at the end of the paper

1) a special warning light on the teleprinter begins to burn

2) the control of the teleprinter printing passes into the red state.

6.1.3. The attention signal.

If the iron cross has been printed and 100 msec later the ribbon colour is black, then a white attention signal begins to flicker, which can be turned off by the operator. By never letting the iron cross be followed by a ribbon-colour change, one can arrange that the flicker light follows only after a black iron cross.

7. The tape reader.

The meaning of the central administration AR0 through AR3 is normal (see 3.); the layout of the start link, which comprises three words, is as described in 4.1, the meaning of the code word in label[1] is:

label[1]:
code word

if code word consists of nothing but zeros, then HOK-instruction, otherwise

count part =
length of the row (where count part = 0

prescribes the length = 512)

address part =
start address of the row of words - 1

The Charon program that serves the tape reader knows three states, called OK, NBK (Not Operationally Ready) and EB (End of Tape). The state of the service program is, for the benefit of Charon, kept in the count part of AR0. For the benefit of the basic machine, which after the conclusion of each start instruction must be able to ascertain how it has been executed, the state prevailing at the moment of filling-in is recorded in the count part of each closing word. In both count parts the coding holds:

000 000 000

OK

111 111 111

NBK
(Not Operationally Ready)

111 111 110

EB
(End of Tape)

The transition to the state NBK takes place (see 6.) again when the service program, just before it would fetch a start instruction in the state OK, cannot satisfactorily ascertain the presence of the correctly connected tape reader. Of this start instruction the code word is then not consulted and Charon passes directly to the concluding rites (with mention of the meanwhile prevailing NBK in the closing word).

In the states NBK and EB start instructions are worked through symbolically, i.e. Charon looks whether the code word is the HOK-instruction; if so, then the service program passes into the state OK (with mention of a closing word = +0); if not, then the instruction is closed off at once with mention of the state (NBK or EB) in the closing word.

When the OK-status holds and the tape reader seems to confirm its correct presence, then the new start instruction is processed normally. If it is a (superfluous) HOK-instruction, then it is executed as such, otherwise the actual tape reading is initiated. Upon the acceptance of the start instruction the code word from the prevailing link is taken over into AR3. Per read punching Charon takes contact with the store twice. In the first store contact with AR3 (the so-called current code word) the count part (mod 512) is decreased by 1 and the address part (mod 218) is increased by 1. If the resulting count part = 0, then the word transport now following is the last of the start instruction. In the store contact following that with AR3 the read punching, supplemented on the most significant side with zeros, is filled in at the memory location indicated by the address part just written away in AR3. Likewise, at the reading of each punching it is inspected whether the detection of the end of the tape gives alarm; if so, then the service program passes to the concluding rites of this start instruction, after first having made the state EB prevailing.

Rem. There is therefore always at least 1 punching read as a consequence of the accepted transport instruction. After each punching the end of tape is first asked for; if not, then only is it tested whether the closing-off as a consequence of count part = 0 is due. If the closing word mentions EB, the start instruction may therefore indeed have been fully executed! The symbolic working-through does not disturb AR3: when the start instruction has been ended on the grounds of End of Tape, there is in the count part of AR3 the missing number of punchings, in the address part the address of the last filled word.

The tape reader reads at a speed of 1000 punchings per second; apart from initiation and concluding rites, the reading process occupies the store of the machine for .625 percent of the time.

The tape reader can be set to the reading of 5-, 7- or 8-hole tape. This tape width is not deemed to contribute to the information; from the program-technical point of view the prevailing tape width is not detectable. In the store a hole in a punching is represented by a 1 and no hole by a 0; the individual positions in a punching are mapped in order onto the least significant bit positions of the word, and indeed in such a way that the tracks assigned to d2 and d3 lie on both sides of the 'sprocket holes". The tape reader reads all configurations uninterpreted, also "blank tape" and "all holes".

NB. In the tape reader there are buffers for two punchings. This has the result that after tape changing the first two punchings that penetrate to Charon are the last two of the previous tape. After switching on the machine the first two read punchings are undefined.

The tape passes in order

1) a detector for the end of the tape

2) the brake

3) the reading station

4) the drive.

The ends of a tape can therefore not be read.

The tape reader is likewise equipped with a red and a green button and accompanying indicator lights; here too the red light flickers when Charon is actually waiting for an absent synchronization signal from the tape reader ("a hanging start instruction"). When Charon, on the grounds of the signal "End of Tape" that has penetrated to it, breaks off a start instruction, it likewise brings the tape reader in question over into the red state; the pressing of the green button has effect only when there still lies a sufficiently long tape in the tape reader. The pressing of the red and the green button does not as such penetrate to the machine, no more than does the changing of tape in a red tape reader.

When a tape has been read up to and including the end over a tape reader, two things must have happened before a following tape can be read over that tape reader:

1) the basic machine must have offered a HOK-instruction to this tape reader

2) the operator must have laid in a new tape and afterwards confirmed this by pressing the green button.

The order of these two events does not matter.
8. The tape punch.

The central administration in the locations AR0 through AR3, as well as the layout of the start link, are with the tape punch just as normal as with the teleprinter printing (see 6). Teleprinter printing and tape punch are namely served by identical Charon programs. Now the lowest 7 bits of the word that is sent to the punch determine what is punched; the remaining 20 are irrelevant. The assignment of the bit positions in the word to the tracks on the tape is as with the tape reader.

The Charon program that serves the tape punch knows four states, called OK, NBK (Not Operationally Ready), EB (End of Tape) and ASL (Closing-Off). The state of the service program is, for the benefit of Charon, kept in the count part of AR0. For the benefit of the basic machine, which after the conclusion of each start instruction must be able to ascertain how it has been executed, the state prevailing at the moment of filling-in is recorded in the count part of each closing word. In both count parts the coding holds:

000 000 000

OK

111 111 111

NBK (Not Operationally Ready)

111 111 110

EB (End of Tape)

111 111 101

ASL (Closing-Off)

The transition to the state NOK takes place (see 6) again only when the service program, just before it would fetch a start instruction in the state OK, cannot satisfactorily ascertain the presence of a correctly connected tape punch. Of this start instruction the code word is not consulted and Charon passes directly to the concluding rites (with mention of the meanwhile prevailing NBK in the closing word).

The transition to the state EB takes place when the service program in the OK-state wishes to begin upon a new start instruction, the tape punch seems to confirm its correct presence but the detector at the end of the tape gives alarm. Of this start instruction the code word is not consulted and Charon passes directly to the concluding rites (with mention of the meanwhile prevailing EB in the closing word).

In the states NBK and EB start instructions are worked through symbolically, i.e. Charon looks whether the code word is the HOK-instruction; if not, then the instruction is closed off at once with mention of the prevailing state (NBK or EB) in the closing word; if so, then the HOK-instruction effects with NBK the transition to OK (with closing word = + 0) and with EB the transition to ASL (with mention of the meanwhile prevailing ASL in the closing word).

Non-symbolic working-through occurs when the service program in the state OK or ASL wishes to begin upon a new start instruction. The only differences between these two states are

1)
the test of being correctly connected is absent with ASL, the transition ASL → NBK therefore does not occur

2)
the test of alarm of the detector of the end of the tape is absent with ASL, the transition ASL → EB therefore does not occur

3)
with ASL the HOK-instruction has the side effect that the tape punch comes into the red state and a light "Change tape" on the tape punch begins to burn.
For the processing of the actual punch instruction (in state OK or ASL) we refer to section 6, the teleprinter printing.

When the end of the tape approaches, the last normally processed punch instruction still gets the OK-report in the closing word; of the first one with the EB-report nothing more has been punched. With a HOK-instruction the basic machine gets the Charon program into the closing-off state, in which alarm of the end-of-tape-detection is ignored. The programmer then still has the opportunity (at least one and a half metres of tape) to punch a special closing combination onto the tape (this may be built up out of more than one start link). A following HOK-instruction brings the service program again into the OK-state, (and as far as the basic machine is concerned the following start instructions can already be offered again); the tape punch, however, is still red until the operator has laid in a new roll (including the resetting of the EB-contact) and has pressed the green button.

Rem. In the red state alarm of the end-of-tape-detector is not passed on to Charon. When after the second HOK-instruction the basic machine again offers instructions for the following tape, then if the operator has not yet changed tape the end of the first tape cannot be mistaken for that of the second. The pressing of the green button has no effect when the light "Change Tape" is still burning.

9. The clock.

The clock is invariably device no. 39. It works independently of the AF and has three functions.

9.1. The actual clock.

Every 10 msec AR0 (mod 227) is increased by 1. The cycle of this counting process is of the order of magnitude of two weeks.
Setting the clock: tooth-brushing (NLVAF and NLVCA), setting, brushing (NLVAF and LVCA)

9.2. The egg-timer.

Every 10 msec the address part of AR1 (mod 218) is decreased by 1. If AR1 thereby becomes totally = + 0, the count part of AR1 is set = 000 000 001 and IF:= true, in other words the interrupt of the egg-timer can take place. Whoever does not want this interrupt can therefore definitively suppress it by making the count part of AR1 immediately ≠ 0. The maximum running time of the egg-timer is about 40 minutes.

9.3. The device-alarm.

Every 10 msec the count part of AR2 is decreased by 1 (mod 512). If the resulting count part becomes = 0 and the address part of AR2 = NR ≠ 0, then

1) of the device with device number = NR the LVAF is set = true

2) AR3 is taken over into AR2.

One initiates this process by filling AR2 and AR3 once with the same thing. With the device-alarm one can have a designated device take note of something every so often, with a maximum period of about five seconds, a facility that is of interest in some on-line applications. By filling in AR2 e.g. + 0 one switches off the device-alarm.

10. The switching-on of the machine.

When the machine is switched on, it comes into the state that can also be reached by pressing LS (Last Switch) on the fold-away part of the operating panel. This has the following effects:

All peripheral equipment comes into the red state.

Charon comes into its resting state and does nothing.

The basic machine comes into the so-called static stop, i.e. likewise does nothing and is not even receptive to the interrupt mechanism.

The regulating flipflops have afterwards the following values

IV = true (Interrupt Permission)

OV = false (Investigation Permission)

all AF = false

all IF = false

all LVIF = true

all LVAF = true (Listening Permission AF, internal Charon flipflops)

all LVCA = false (Listening Permission Communication Announcement, likewise internal flipflops of Charon)

In this installation state (and only in this installation state) the pressing of either IP (Initial Program) or NB (Normal Begin) has effect. The discussion of the effect of the key NB is postponed to the treatment of the tooth-brushing.

10.1 The Earliest Beginning.

The pressing of the key IP has the effect that the basic machine passes into the dynamic stop state, i.e. it still does nothing but is now indeed receptive to interrupts; since (see 10) all IF's are false, the interrupt mechanism does not for the time being come into operation. In T the value 218 - 1 is set.

Furthermore Charon is started in a special program —called the earliest beginning— that reads tape in a very special way via a fixed, installation-dependent tape reader.
On this tape there occur words, coded as follows in four consecutive heptads; the reading direction is from top to bottom, the position of the sprocket holes is rendered by some extra space. Again a hole represents a 1.

1 d26 d25 d24 d23 d22 d21
d20 d19 d18 d17 d16 d15 d14
d13 d12 d11 d1O d9 d8 d7
d6 d5 d4 d3 d2 d1 d0
At the front and back of the tape one should choose a decent stretch of blank.
The reading program skips possible extra blank heptads before each word; the first word that is read from the tape is interpreted as code word, albeit that the consecutive word transports to the store now pertain not to single heptads but to whole words. As soon as this start instruction has finished, the earliest beginning returns to the state in which the next word read from the tape is again interpreted as code word.

The IP-reading is ended by Charon as soon as it would find a code word consisting of nothing but zeros, i.e. at the right moment the configuration

1000 000
0000 000
0000 000
0000 000
The ending has for Charon the consequence that it passes into the normal resting state ("nothing to do") (during the IP-reading its state was indeed still a bit special); the tape reader in question stops and remains green with LVAF = true (brushing is not necessary). Finally Charon sets, independently of IFT of the accompanying tape reader, the IF = true. Unless SVA (Stop Next Address) is in, the normal interrupt now takes place immediately in the basic machine via M[24]. (If SVA is in, then the interrupt remains hanging with all ones (2↑18 - 1) in the instruction counter, until the machine is restarted.)

NB. Since the reading of the earliest beginning is not subjected to any form of parity check, it is advisable not to read in too much more than necessary with the earliest beginning.

10.2 The tooth-brusher.

The tooth-brusher is for all installations device no. 38. One starts the tooth-brusher by (after offering of a start instruction) setting its AF to true; it reports completion of its action by setting its IF to true. All this goes around AFT and IFT, which do not exist with the tooth-brusher. (AR1, AR2 and AR3 are not used by the tooth-brushing program).

Start instructions for the tooth-brusher cannot be chained; one places them one by one in AR0. The meaning of the bits of AR0 is then:

d26
through d20:

all = 0 LVCA =: false

not
all = 0 LVCA =: true

d19
through d18:
both permanently = 0

d17
through d11:

all = 0 LVAF =: false

not
all = 0 LVAF =: true

d10
through d6:
irrelevant

d5
through d0:
number of the device to be brushed.

The brushing of a device means

1) that the electronic control of the device is brought into the "home position"

2) that the LVAF and the LVCA for this device are set according to the specification of the start instruction.

The LVAF's and LVCA's are internal flipflops of Charon, and for each device there is such a pair. The LVAF (Listening Permission AF) controls whether Charon will undertake an action upon the being-true of the accompanying AF; the LVCA (Listening Permission Communication Announcement) controls whether Charon will undertake an action upon the presence of the accompanying so-called Communication Announcement (a "synchronization signalling" from the device to Charon).

For the clock LVCA must be true and the LVAF is irrelevant; for the keyboard LVCA and LVAF must both be true; for the tooth-brusher LVAF must be true (so leave it so); for "normal devices" (teleprinter printing, tape punch, tape reader, line printer, plotter, drum) LVAF must be true and LVCA false.

NB. The tooth-brushing provides no initiation of the central administrations, this must therefore be done directly by the basic machine

11. The fault-handling.

The fault-handling is always device no. 37. The meaning of the central administration is

AR0: base address central administration first fault

AR1 count part: IFT

address part : unequal to zero

AR2 unused

AR3: base address central administration next fault.

As soon as in one of the Charon programs a parity fault is signalled at the fetching of a word from the store, then

1) the word with the wrong parity is written back into the store, just as it was found there

2) the prevailing Charon program is, after LVCA:= LVAF:= false (so that it will until further order no longer become active), immediately interrupted

3) under transmission of the address of the AR0 belonging to the interrupted Charon service program, the fault-handling is immediately started by the making-true of the LVCA.

This base address of the central administration is processed by the fault-handling as a character of the keyboard is by the keyboard program:

if AF and LVAF then AR0:= base address else AR3 := base address; AF := false;

IFT:= IFT + 1 (mod 512); if 0 then IFT:= true

The reaction to the interrupt of the fault-handling places the basic machine before an essential hurry-situation; at cumulation of faults the basic machine loses the trail.

Remark. If a parity fault occurs with the tooth-brusher, one is thrown back upon LS to get a grip on the machine again!

- - - - - - - - - - - - - - - - - - - -- - - - - - - - - - -- - - - -

Addition to 10.2. After switching on or LS one can, besides IP, also press NB. One then forces IF T := 218 - 1 of the tooth-brusher true and gets an interrupt via M[24].

Transcribed
by Martin van der Burgt

Revised
16-Aug-2013
