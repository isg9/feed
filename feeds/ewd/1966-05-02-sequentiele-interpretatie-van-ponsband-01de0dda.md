---
title: "Sequentiele interpretatie van ponsband (EWD161)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD161.html
published: "1966-05-02T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD161.PDF
---

# Sequentiele interpretatie van ponsband

Dit rapport handelt over de structuur en onderlinge samenhang van vier procedures (of proceduretypen):

integer procedure HEP

voor het aftasten van ponsband

integerproceduretype CHAR

voor het aftasten va een paginabeeld

integer procedure SYM

voor het aftasten van een rij basic
symbols

real procedure NUM

voor het aftasten van een rij numerieke
waarden.

Van het type CHAR beschouwen we er twee, zeg CHARF en CHART voor het aftasten
ven een paginabeeld van flexowriter resp. teleprinter; de uitbreiding naar meer dan
twee verschillende CHAR’s ligt voor de hand. We gaan voorbij aan het feit, dat het
paginabeeldend vermogen van de teleprinter toevallig bijna geheel vervat is in dat
van de flexowriter en beschouwen dus hun respectievelijke paginabeeldende vermogens
als volslagen verschillend.

Als een routine van het type CHAR een band van het betrokken apparaat te verwerken
krijgt, zal deze bij herhaald aanroepen een waardenreeks afleveren, die uniek verslag
aflegt over het bij deze band behorende paginabeeld en wel regel voor regel,
elke regel van links naar rechts. Het zijn de routines van het type CHAR� waarin
de eigenschappen van de diverse hardware ponser/bladschrijvers hun neerslag vinden.
Over deze eigenschappen worden twee veronderstellingen gemaakt

a)         We veronderstellen dat het hardware apparaat beschikt over enige geheugenelementen,
waarvan de beginwaarde (het begin van) het paginabeeld mede kunnen bepalen,
dat aan een band ontleend wordt. Wij noemen heersende case, wagenpositie etc. Hieraan
beantwoordt, dat elke routine van het type CHAR, voordat hij over een paginabeeld
een verslag kan gaan afleggen geactualiseerd (ge�nitialiseerd) moet worden, waarbij
deze mechanische geheugenelementen een bij de actualisering opgegeven beginwaarde
toegedacht krijgen. Na actualisering verslaat CHAR het paginabeeld, dat ontstaat
wanneer

1)
het apparaat in de toestand gebracht zou worden zoals bij de actualisering
van CHAR opgegeven

2)
de band ingelegd wordt met de eerste heptade die door CHAR via HEP
gelezen gaat worden in het leesstation van het apparaat in kwestie.

b)        we veronderstellen de door de hardware van de ponser/bladschrijver gegeven code
zo ingewikkeld, dat in het algemeen geen natuurlijke synchronisatie tussen pagina—
en bandbeeld bestaat, maw. dat als het verslag van CHAR tot een bepaald punt op
de pagina gevorderd is, het plotseling doorlezen van de band via HEP op een in
het algemeen ongedefinieerd moment inhaakt.

Is het gebruik van CHARF en CHART —door de complexiteit van de apparatuur
en door niets anders!— aan enige beperkende regels onderworpen, verder beogen we
de grootst mogelijke algemeenheid. (Als een implementator voor sommige hieronder
te noemen mogelijkheden geen emplooi ziet en ze weglaat, dan is dat zijn zaak: ik
betrek ze in het beeld, om dan de weg tot latere toevoeging open te houden.)

De routine SYM kan op drie manieren aan een band een rij basic symbols ontlenen

1)
een rij basic symbols gedefinieerd op een flexowriterpaginabeeld; SYM zal dan
via CHARF de band aftasten, dit interpretatieproces noemen we “symcharf”.

2)
een rij basic symbols gedefinieerd op een teleprinterpaginebeeld; SYM zal dan
via CHART de band aftasten, dit interpretaieproces noemen we “symchart”.

3)
een rij basic symbols� zonder tussenkomst van een paginabeeld direct op het
bandbeeld gedefinieerd; SYM zal dan via HEP de band aftasten, dit interpretatieproces
noemen we “symhep”.

In het bijzonder zullen we eisen, dat indien via SYM een reeks basic symbols
van een band gelezen wordt, tussentijdse wisselingen tussen de drie interpretatieprocessen
zijn toegestaan, zonder dat het SYM aanroepende programma hier enige
zorg mee heeft. Dit impliceert

1)      dat in een ALGOL tussen te voegen teksten van standaardprocedures een soort
binaire banden zouden kunnen zijn, die SYM via symhep zou kunnen lezen

2)      dat een programma, dat in teleprintercode wordt aangeboden, van een omvattend
blok kan worden voorzien door er een stukje flexowritertekst voor- en achteraan te
plakken.

Evenzo laten wij voor NUM vier verschillende manieren open, waarin hij aan
een band numerieke waarden ontleent

1)       numerieke waarden, gedefinieerd op een rij basic symbols. NUM zal dan via
SYM de band aftasten, wij noemen dit interpretatieproces “numsym”. (Bij activiteit
van numsym doet het niet ter zake of SYM via symcharf� symchart of symhep werkt.)

2)       numerieke waarden gedefinieerd op een flexowriterpaginabeeld; NUM zal dan via
CHARF de band aftasten, wij noemen dit interpretatieproces “numcharf”. (NB. Hierbij
kunnen paginabeelden verwerkt worden, die niet als representatie van basic
symbols beschouwd kunnen worden, bv. octale getallen met onderstreepte cijfers
of zo.)

3)       numerieke waarden gedefinieerd op een teleprinterpagina; NUM zal dan via
CHART de band aftasten, wij noemen dit interpretatieproces “numchart”.

4)       numerieke waarden, zonder tussenkomst van basic symbols of paginabeeld�
direct gedefinieerd op een bandbeeld; NUM zal dan via HEP de band aftasten, dit
interpretatieproces noemen we “numhep”. (Op deze wijze zouden binaire voorstellingen
direct van een band gehaald kunnen worden.)

Wederom moet een programma vele malen NUM kunnen aanroepen, ongeacht de
wisselingen tussen deze vier wijzen van getalopbouw.

Behalve dat gemengde toegankelijkheid van een van de paginabeelden of het
bandbeeld elkaar uitsluiten, proberen we zo liberaal mogelijk te zijn. In het
bijzonder:

a)       Alternering tussen SYM en CHARF (CHART resp. HEP) is definieerbaar mits
SYM werkt volgens symcharf (symchart resp. symhep).

b)       Alternering tussen NUM on CHARF (CHART resp. HEP) is definieerbaar

1)
als NUM werkt volgens numcharf (numchart resp. numhep)

2)
als NUM werkt volgens numsym en SYM voldoet aan voorwaarde a).

c)       Alternering tussen NUM en SYM is definieerbaar

1)
als NUM werkt volgens numsym (aan de werkwijze van SYM is dan geen
beperking opgelegd)

2)
als NUM werkt volgens numcharf (numchart resp. numhep) en SYM werkt
volgens symcharf (symchart resp; symhep}.

We observeren, dat er drie elkaar uitsluitende basisinterpretaties aan de
gang kunnen zijn; welke van de drie vigeert leggen we vast in de waarde van een
aan de informatiestroom toegevoegde integer statusvariabele “basint”.

basint = 1de informatie is gedefinieerd op een bandbeeld� dat niet uitgelegd wordt
als carrier van een paginabeeld (symhep� numhep en algemeen HEP
toegestaan, CHARF en CHART niet van toepassing)

basint = 2 CHARF actueel, dwz. het bandbeeld fungeert alleen als carrier van
een flexowriterpagina� alleen CHARF van toepassing (en HEP alleen
vanuit CHARF).

basint = 3CHART actueel. dwz. het bandbeeld fungeert alleen als carrier van
een teleprinterpagina, alleen CHART van toepassing (en HEP alleen
vanuit CHART).

De statusinteger basint kent nog een vierde waarde

basint = 0geen van de drie genoemde interpretaties vigeert; in het bijzonder,
als in deze toestand informatie van de band moet komen, die alternatief
op teleprinterpagina, flexowriterpagina, dan wel direct op bandbeeld
gedefinieerd kan zijn, dan zal de aanhef van het dan te lezen bandstuk duidelijk
moeten maken, of basint de waarde 1, 2 of 3 moet krijgen.

De routine HEP.

Nadat deze geinitialiseerd is —dwz. de band is ingelegd— geeft de integer
procedure HEP per aanroep de volgende n-ade van de band. Op “EINDE BAND” reageert hi
door totaan de volgende initialisering de waarde “–1” af te leveren. De routine HEP
berekent bij herhaald aanroepen een functie gedefinieerd op het mathematisch model,
dat ik voor een eindige ponsband heb gekozen: een (in de toekomst) oneindige
halfrij “heptaden�� die voor een zeker moment niet en na dat moment louter de
karakteristieke waarde “–1” bevat en aflevert. Voor dit model pleite

1)       dat de oneindigheid de mathematische hanteerbaarheid van HEP alleen maar
ten goede kan komen

2)       dat een mogelijk relevant aspect van de werkelijkheid (nl. einde band)
hierin zijn representatie vindt.

(Ik ben me bewust, dat andere mogelijk relevante aspecten van de werkelijkheid
onder tafel zijn geraakt. De breedte van het papier zou men kunnen willen gebruiken
ter bepaling van de keuze tussen alternatieve codes; de kleur van het papier of
het fabrieksnummer van de flexowriter� waarop een band geponst is, zou men bij het
optreden van lees— of ponsfouten voor diagnostische doeleinden misschien willen
weten!)

Vooralsnog zie ik geen noodzaak om HEP “een voorgift” te kunnen geven. Mocht
deze noodzaak optreden, dan moet een verstandige keuze gemaakt worden tussen de
bijna aequivalente mogelijkheden, die naar aanleiding van CHAR nog ter sprake zullen
komen.

De routines van het type CHAR.

Deze sectie behandelt alleen CHARF� omdat ik met de teleprinter onvoldoende
op de hoogte ben en ik mij, vanwege onbekendheid met standaardpraktijken in het
telexverkeer� nauwelijks bevoegd acht om hier aanbevelingen te doen.

De routine CHARF die ik op het oog heb heeft betrekking op een MC-flexowriter
zonder variabele spatiering� zonder backspace� met vaste instelling van de regel-
opvoer en vaste spatiering van de tabulatorstoppen.

Initialisering van CHARF geschiedt onder opgave van twee parameters, de boolean
“lower case” en de integer “carriage position”� die bij het begin van de regel op
0 gesteld wordt. Daarna levert CHARF per aanroep het verslag van het paginabeeld en
wel per positie op de pagina. (Elke “escaping” aanslag wordt verslagen in viervoud
als hij caseonafhankelijk is (dwz. de spatie) en in achtvoud als hij caseafhankelijk
is, nl. al of niet onderstreept en al of niet doorgestreept). Bij het waardebereik
van CHARF is tevens NLCR —dwz. overgang op een nieuwe regel— begrepen, TAB’s
worden vertaald in een aequivalent aantal spaties, spaties en TAB’s aan het einde
van de regel worden vanwege hun onzichtbaarheid niet doorgegeven.

We kunnen (zie boven) bv— basic symbols hetzij op een Flexowriterpagina�
hetzij op een teleprinterpagina definieren en we hebben geeist van bv. een routine
als SYM, dat hij achter de schermen van de ene code op de andere kan overgaan.

Deze overgangen moeten door de band gestuurd worden. Van elke routine van
type CHAR stel ik me voor, dat er een karakteristieke (zelfafsluitende) ponsing
(of ponscombinatie) is, die behelst, dat hiermee de heersende interpretatie tot
nader order is afgelopen. Voor de procedure CHARF stelde ik me voor hiervoor de
stopcode in de betekenis “EINDE CODE” te gebruiken.

Omdat ik voor routines, die in termen van CHARF werken zeer zeker wil toestaan
dat ze niet zelfafsluitende eenheden van de pagina lezen, zal CHARF wel een voorgift
van een enkele waarde kunnen hebben.

Voor de routine CHARF doemt de volgende rol op.

De toestand “CHARF actueel” —dwz. basint = 2— kan slechts bewerkstelligd worden
door de aanroep van een initialiseringsnrocedure� waarbij de statusvariabelen van
CHARF (te weten “heersende case” en “Waqenpositie”) worden opgegeven. Door de initialiserinqsprocedure
wordt de voorgift ingesteld op “leeg”.

Het geven van een voorgift zou op twee manieren kunnen

1)
door het parameterloze commando “herhaal bij eerstvolgende aanroep de laatst
afgeleverde waarde eerst nog een keer”

2)
door het parameterhebbende commando “geef bij eerstvolgende aanroep eerst de
hierbij opgegeven waarde”.

Omdat de tweede methode wat flexibeler is �en de eerste bovendien slecht
gedefinieerd is onmiddellijk na initialisatie� prefereer ik de tweede versie.

Indien aan CHARF een voorgift gegeven wordt op een moment, dat CHARF niet
actueel is, heeft deze handeling geen effect.

Indien aan CHARF een voorgift gegeven wordt, terwijl een nog niet teruggegeven
voorgift hing, dan overschrijft de laatste voorgift de eerste.

Een neveneffect van CHARF zal moeten zijn� om naar aanleiding van het ontmoeten
van de stopcode de toestand “basint = 2” (dwz. CHARF’s actualiteit) te beeindigen,
Precieser: evenzo als CHARF onzichtbare spaties aan het einde van de regel niet
doorgeeft, zal CHARF ook onzichtbare NLCL’s niet doormelden� wanneer uiteindelijk
een stopcode op de band gevonden wordt. Het optreden van de stopcode wordt aan het
aanroepend programma doorgegeven, doordat CHARF met de waarde “–1” terugkeert,
nu in de betekenis “EINDE CODE”. Indien CHARF wordt aangeroepen tijdens
“basint =: 2” en hij keert terug met de waarde “–1”, dan heeft dit “basint:=0”
als neveneffect. (Dit geldt ook, als deze –1 uit de voorgift komt; hiermee is dus
een manier gegeven om CHARF, als hij actueel mocht zijn� zijn actualiteit
te laten beeindigen.) Als CHARF wordt aangeroepen tijdens “basint ≠ 2 � dan keert
de besturing eveneens terug met CHARF = –1, doch ditmaal zonder neveneffect.

Bij non-actualiteit van CHARF heeft het geven van een voorgift geen (neven)
effect, het aanroepen van CHARF evenmin.

Met deze definitie van het afvallen van de actualiteit heb ik het volgende
bereikt: als SYM van de flexowriterpagina een niet afsluitend symbool leest, gevolgd
door een basic symbol� dat 1 positie beslaat, gevolgd door een stopcode� dan heeft
SYM bij het afleveren van het eerste symhool de volgende positie al gezien. CHARF
heeft misschien de stopcode al gezien, maar nog niet doorgegeven� zodat hij zijn
actualiteit nog niet heeft beeindigd. Dus kan de laatste positiebeschrijving nog
in CHARF worden teruggeduwd.

Als CHARF wegens het doorgeven van in stopcode voor basint de overgang van 2
naar 0 heeft bewerkstelligd, dan is (als de stopcode via de band is gekomen) de
synchronisatie met HEP gedefinieerd: hierna levert een aanroep van HEP de eerste
heptade, die volgt op de stopcode. (En dit is eigenlijk het enige moment, waarop
ik synchronisatie tussen CHAR en daarna HEP expliciet zou willen definieren.
Uitgebreidere synchronisatieregels krijgen gauw iets gekunstelds en bovendien: de hemel
weet, wat voor dol apparaat we morgen in het systeem moeten opnemen!)

Beinvloeding van basint.

Rechtstreeks HEPpen is niet verenigbaar met de actualiteit van CHARF
of CHART� De vraag is, hoe het systeem reageert wanneer men deze (en dergelijke)
regels probeert te schenden. In het bijzonder:

1)  actualisering van CHARF terwijl basint ≠ 0 is

2)  actualisering van CHART terwijl hasint ≠ 0 is

3)  rechtstreeks HEPpen terwijl basint = 2 of = 3 is.

Als basint ≠ 0 is, betekent dit heel expliciet: dat we ons tot nader order tot
een bepaalde bandinterpretatie verplcith hebben, Je kunt zeggen: dan nog een keertje
CHARF of CHART initialiseren is kennelijk misplaatst (en daar de consequentie van
een letale programmafout aan verbinden); je kunt ook zeggen: het herdefinieren van
de bandinterpretatie is dan kennelijk die “nader order” en het zal wel goed wezen.
Uit opportunistische overwegingen neig ik tot het laatste; voor argumenten, dat het
de moeite waard is om dit als letale programmafout te behandelen, sta ik open.

Rechtstreeks HEPpen terwijl basint = 2 of = 3 is, is vreemder, omdat we immers
ons op het standpunt hebben gesteld, dat bij actualiteit van een CHAR de synchronisatie
met HEP in het algemeen ongedefinieerd is. Om dit als letale programmafout
te brandmerken, is me echter te gortig. Daarvoor is het primitivum HFP me te fundamenteel.
Wel betekent rechtstreeks HEPpen� dat een eventuele CHAR�interpretatie
niet meer valide is. Ik stel daarom voor, om HEP als neveneffect te laten hebben

“if basint ≠ 1 then basint := 0” .

(wordt HEP met basint = 2 vanuit CHARF aangeroepen, dan kan CHARF zelf, voor verlating
met �basint := 2” dit neveneffect weer ongedaan maken.)

De initialisering van CHARF en CHART zijn de enige manier om basint = 2 of
= 3 te krijgen; verder zal de programmeur basint = 0 of basint =1 mogen zetten.

De keuze tussen symhep, symcharf en symchart

De keuze tussen symhep, symcharf en symchart hangt af van de heersende waarde
van basint. Maw, wie weet, dat zijn SYM via symcharf zal moeten werken, kan zulks
bereiken, door voor de eerste aanroep van SYM de CHARF te initialiseren; evenzo
kan hij a priori interpretatie volgens symchart of symhep via basint dicteren. Zodoende
is het mogelijk om via SYM flexowriterbanden te lezen, zonder dat deze zich
in hun aanhef als zodanig identificeren. Willen we echter de keuze niet a priori
opleggen, maar van de band laten afhangen, dan kunnen we dit doen door

1) voor de eerste aanroep van SYM de variabele basint op 0 in te stellen.

2) ponsconventies te eisen, zodat aan het begin van de band te zien is, met welke
waarde van basint tot nader order doorgelezen moet worden. SYM kan dan zelf zonodig
CHARF of CHART actualiseren. (Ik zou voor de flexowriteraanhef willen voorstellen
na skippen van blanks erases (en stopcodes?) eerst een NLCR of een casedefinitie,
voor teleprinterbanden na skippen van blanks eerst men CR, een NL of een casedefinitie;
de door SYM ingelaste CHARF- of CHART-initialisatie zal voor de onbekende
parameter een standaardkeuze doen, bv. begin van de regel, lower case dan wel
lettershift.)

Om tot normalisering van deze aanheffen te komen doe ik een beroep op alle belanghebbenden.
Ik verwacht dan, dat iedereen zijn ponserij zo zal instrueren, dat
de aldaar geponste banden —speciaal ALGOL-programma’s en gegevensbanden— conform
die conventie aanheffen, ook al heeft hij dit voor eigen bedrijf niet nodig. Tevens
verwacht ik van elke beperkte implementatie van eigen bedrijf (zeg “alleen Flexowriter
banden”), dat die flexowriterbanden met genormaliseerde aanhef accepteert.

Initialisering van SYM.

Wat betreft de keuze van interpretatieproces is de status van SYM dus geregeld
door basint. We zitten er nog mee, dat ook SYM (juist SYM!) een voorgift van een
enkele waarde moet kunnen hebben. Definieren we het geven van een voorgift door
overschrijving van een eventueel nog hangende voorgift (als hij CHAR) dan kunnen
we SYM in dit opzicht initialiseren door een voorgift te geven en 1 maal SYM aan
te roepen.

Initialisering van NUM.

De interpretatiewijze van SYM is volledig door basint bepaald, dit geldt niet
voor NUM: immers als basint bv. CHARF als actueel aanwijst, dan hebben we
nog de keuze tussen numcharf en numsym via symcharf, NUM
heeft dus nog een boolean statusvariabele, die we “numint” noemen en die
aangeeft of numsym actueel is. Is numint true, dan bepaalt basint slechts de
keuze tussen symcharf, symchart en symhep; is numint false, dan bepaalt basint
tevens de keuze tussen numcharf, numchart en numhep. De boolean “numint” zij in
beide richtingen zetbaar.

Ik stel voor om de derde denkbare toestand (numint ongedefinieerd) te laten
vervallen en in alle twijfelgevallen numint true te zetten. (Aan het begin van een
nieuwe band.) Dit is een keuze, die aan de interpretatiewijze numsym een zekere
voorkeur geeft.

Immers: laat —onder voortdurende actualiteit van CHARF— NUM een tijdlang via
numsym gewerkt hebben en laten we nu op de Flexowriterpagina willen aangeven, dat
van nu af aan tot nader order NUM via numcharf moet werken. Het feit, dat numint
false moet worden, moet worden gemeld door de band, met wie NUM slechts via SYM
contact heeft, Dit kan op verschillende manieren; ik noem

1) door af te spreken, dat het optreden van bepaalde getalscheiders door NUM
geinterpreteerd zal worden als “ga over op numint = false”

2) door een extra basic symbol in te voeren met deze betekenis

3) door alles, wat niet als basic symbol interpreteerbaar is als “nonsense” door
te melden en diezelfde betekenis te geven.

(Ik ga steeds meer voor de eerste mogelijkheid voelen.)

In alle gevallen kunnen we bereiken, dat een band niet via numsym gelezen gaat
worden, door in de aanhef dit aan te geven. Het resultaat is, dat NUM banden leest
via numsym� tenzij andere vermeld. Dit lijkt in overeenstemming met wat we wensen.

De verwerking van “EINDE BAND”.

Tot nog toe hebben we alleen erover gesproken, hoe HEP op einde band reageert.
Daar hebben we afgesproken, dat het optreden van EINDE BAND mogelijk wel tot de
semantische informatie behoort en HEP is niet uitgerust met automatische aanvrage
van volgband. Nu moeten we dit voor de volgende niveaus (CHAR� SYM en NUM)
beslissen.

Ook op het niveau van CHAR zou ik nog geen automatische volgbandaanvrage willen
inbouwen. Sterker: ik vond het hachelijk om de door CHAR veronderstelde codecontinuiteit
zich over physisch gescheiden stukken band te laten uitstrekken en stel
daarom voor CHARF —wanneer hij via HEP “EINDE BAND” detecteerd— op deze detectie
te laten reageren als op de stopcode en “EINDE CODE” te laten doormelden (en dan
basint:= 0 uit te voeren). Het CHAR aanroepend programma kan dan via HEP inspecteren
of er nog meer band is, zo nee, beslissen of het meer wil zien etc.

De consequentie hiervan is de eis, dat als we met SYM in diverse codes willen
kunnen lezen, dan in elk geval op het begin van elke physische band de
code-identificerende aanhef staat. Is aan deze voorwaarde voldaan, dan kunnen we een
lange band uit een aantal deelstukken assembleren door plakken; wanneer alle
deelstukken eindigen met de stopcode� dan kan dit plakproces zelfs zonder er aandacht
aan te besteden of bij die plakjes codewisselingen optreden.

Een en ander voel ik als streke aanwijzing om SYM wel met automatische aanvraag
van volgband te versieren. (Onder het motto: het hoeft tot het SYMmend programma
niet door te dringen, dat die logisch nu zonder meer toegestane plakjes misschien
niet de facto zijn uitgevoerd). En ik stel me zelfs voor om op dit niveau aan het
physisch einde band geen semantische betekenis meer toe te kennen (en dan ook
consequent, dus niet bv. nog een keer doormelden). Dat een programma, dat basic
symbols van een band pulkt dan gegarandeerd ongevoelig is voor de gebruikte code
en voor het physisch aantal banden lijkt me een groter goed dan het verlies van
de mogelijkheid om met SYM een programma te schrijven, dat het aantal basic symbols
op een band telt.

Het is aan SYM om nieuwe band aan te vragen, en wel, wanneer tijdens inspectie
(basint = 0) blijkt, dat HEP de waarde –1 aflevert. Het aanvragen van een nieuwe
band hebben als neveneffect, dat numint = true gezet wordt.

Nog enkele opmerkingen over de synchronisatie.

We nemen aan, dat NUM via numsym de getallen als niet zelfafsluitende eenheden
uit de basic symbol stroom pulkt. (B.v. het teken van het volgende getal kan als
separator gebruikt worden.) Als alleen NUM gebruikt zou worden kan dit op twee
manieren gerealiseerd worden.

1) het teveel gelezen basic symbol wordt in SYM teruggeduwd

2) het teveel gelezen basic symbol wordt in NUM opgeslagen.

Zodra we alternerend NUMmen en SYMmen maakt dit wel verschil. Ik geloof, dat
iedereen methode 1) verkiest.

Als echter SYM in termen van CHARG gedefinieerd is, dan zij nu ook een alternering
tussen NUM en CHARG gedefinieerd. Men lette er op, dat na NUM het CHARFen
het paginabeeld aftast te beginnen achter het teveel gelezen basic symbol!

Wie de band “zonder overslaan” sequentieel wil aftasten zal na NUM eerst een
keer SYM moeten doen en dan pas CHARFen. Een open vraag is, of SYM zijn voorgift
moet behouden bij rechtstreeks CHARFen� CHARTnn of HEPpen. Ik hem geneigd deze
vraag ontkennend te moeten beantwoorden; anders heeft nl na CHARF herhaald SYMmen
niets meer te maken met in successie aftasten en dat kon wel eens “a trouble
shooter’s nightmare” worden.

Het laatste getal.

Lijkt ons de opgave “tel de basic symhols op deze band” wat gezocht, programma’s
die een onbekend aantal getallen moeten verwerken zijn heel bekend. De truc “zet
voorop het aantal” kennen we allemaal en we weten allemaal, hoe hinderlijk deze
kan zijn.

Ik zou me kunnen voorstellen, dat we naast NUM een boolean procedure NUMMARK
introduceren en dit tweetal samen definieren op een willekeurige rij van numerieke
waarden en zg� “marks” (waarbij een mark als getalscheider fungeert).

De boolean procedure NUMMARK geeft antwoord op de vraag of er bij doorlezing
eerst een “mark” komt, zo ja, dan is daarmee deze mark gepasseerd. In de diverse
codes moet dit realiseerbaar zijn, zonder NUM meer statusvariabelen te geven.
(Als we HEP niet met de terugduwfaciliteit uitrusten kost dit ons een heptade per
getal, maar daarover zou ik niet willen vallen).

transcribed
by Martin van der Burgt

revised 20-Nov-2013

---

## English translation

### Sequential interpretation of punched tape

This report deals with the structure and mutual coherence of four procedures (or procedure types):

integer procedure HEP

for the scanning of punched tape

integer procedure type CHAR

for the scanning of a page image

integer procedure SYM

for the scanning of a sequence of basic
symbols

real procedure NUM

for the scanning of a sequence of numerical
values.

Of the type CHAR we consider two, say CHARF and CHART for the scanning
of a page image of flexowriter and teleprinter respectively; the extension to more than
two different CHARs is obvious. We pass over the fact that the
page-imaging capacity of the teleprinter happens to be almost entirely contained in that
of the flexowriter, and we therefore regard their respective page-imaging capacities
as utterly different.

When a routine of the type CHAR is given a tape of the apparatus concerned to process,
then upon repeated invocation it will deliver a sequence of values that gives a unique report
about the page image belonging to this tape, namely line by line,
each line from left to right. It is in the routines of the type CHAR that
the properties of the various hardware punchers/page-writers find their imprint.
Concerning these properties two assumptions are made

a)         We assume that the hardware apparatus disposes of some memory elements,
whose initial value can co-determine the (beginning of the) page image
that is derived from a tape. We mean prevailing case, carriage position etc. To this
corresponds the fact that every routine of the type CHAR, before it can begin to give a report
about a page image, must be actualized (initialized), whereby
these mechanical memory elements are assigned an initial value specified at the actualization.
After actualization CHAR reports the page image that arises
when

1)
the apparatus would be brought into the state as specified at the actualization
of CHAR

2)
the tape is inserted, with the first heptad that is to be
read by CHAR via HEP, into the reading station of the apparatus in question.

b)        we assume the code given by the hardware of the puncher/page-writer
so complicated that in general no natural synchronization between page
and tape image exists, i.e. that when the report of CHAR has progressed to a certain point on
the page, the sudden reading-on of the tape via HEP hooks in at a moment that is in
general undefined.

While the use of CHARF and CHART —through the complexity of the apparatus
and through nothing else!— is subject to some restrictive rules, for the rest we aim at
the greatest possible generality. (If an implementor sees no employment for some of the possibilities
to be mentioned below and omits them, then that is his affair: I
include them in the picture in order thereby to keep open the road to later addition.)

The routine SYM can derive a sequence of basic symbols from a tape in three ways

1)
a sequence of basic symbols defined on a flexowriter page image; SYM will then
scan the tape via CHARF, this interpretation process we call “symcharf”.

2)
a sequence of basic symbols defined on a teleprinter page image; SYM will then
scan the tape via CHART, this interpretation process we call “symchart”.

3)
a sequence of basic symbols defined directly on the tape image, without the intervention
of a page image; SYM will then scan the tape via HEP, this interpretation process
we call “symhep”.

In particular we shall require that if a series of basic symbols is read
from a tape via SYM, intermediate switches between the three interpretation processes
are permitted, without the program invoking SYM having any
concern about this. This implies

1)      that the texts of standard procedures to be inserted in an ALGOL could be a sort of
binary tapes, which SYM could read via symhep

2)      that a program offered in teleprinter code can be provided with an enclosing
block by pasting a piece of flexowriter text at the front and the back of it.

Likewise we leave open for NUM four different ways in which it derives numerical
values from a tape

1)       numerical values, defined on a sequence of basic symbols. NUM will then scan the tape via
SYM, we call this interpretation process “numsym”. (During the activity
of numsym it does not matter whether SYM works via symcharf, symchart or symhep.)

2)       numerical values defined on a flexowriter page image; NUM will then scan the tape via
CHARF, we call this interpretation process “numcharf”. (NB. Hereby
page images can be processed that cannot be regarded as a representation of basic
symbols, e.g. octal numbers with underlined digits
or the like.)

3)       numerical values defined on a teleprinter page; NUM will then scan the tape via
CHART, we call this interpretation process “numchart”.

4)       numerical values, without the intervention of basic symbols or page image,
defined directly on a tape image; NUM will then scan the tape via HEP, this
interpretation process we call “numhep”. (In this way binary representations
could be fetched directly from a tape.)

Again a program must be able to invoke NUM many times, regardless of the
switches between these four manners of number construction.

Apart from the fact that mixed accessibilities of one of the page images or the
tape image exclude one another, we try to be as liberal as possible. In
particular:

a)       Alternation between SYM and CHARF (CHART resp. HEP) is definable provided
SYM works according to symcharf (symchart resp. symhep).

b)       Alternation between NUM and CHARF (CHART resp. HEP) is definable

1)
if NUM works according to numcharf (numchart resp. numhep)

2)
if NUM works according to numsym and SYM satisfies condition a).

c)       Alternation between NUM and SYM is definable

1)
if NUM works according to numsym (no restriction is then imposed on the
way of working of SYM)

2)
if NUM works according to numcharf (numchart resp. numhep) and SYM works
according to symcharf (symchart resp. symhep}.

We observe that three mutually exclusive basic interpretations can be in
progress; which of the three is in force we record in the value of an
integer status variable “basint” added to the information stream.

basint = 1 the information is defined on a tape image that is not interpreted
as a carrier of a page image (symhep, numhep and in general HEP
permitted, CHARF and CHART not applicable)

basint = 2 CHARF current, i.e. the tape image functions only as a carrier of
a flexowriter page, only CHARF applicable (and HEP only
from within CHARF).

basint = 3 CHART current, i.e. the tape image functions only as a carrier of
a teleprinter page, only CHART applicable (and HEP only
from within CHART).

The status integer basint knows yet a fourth value

basint = 0 none of the three named interpretations is in force; in particular,
if in this state information must come from the tape that can alternatively
be defined on a teleprinter page, a flexowriter page, or directly on a tape image,
then the heading of the piece of tape then to be read will have to make clear
whether basint must acquire the value 1, 2 or 3.

The routine HEP.

After this has been initialized —i.e. the tape has been inserted— the integer
procedure HEP gives per invocation the next n-ad of the tape. To “END OF TAPE” it reacts
by delivering, until the next initialization, the value “–1”. The routine HEP
computes, upon repeated invocation, a function defined on the mathematical model
that I have chosen for a finite punched tape: an (in the future) infinite
half-sequence of “heptads” that before a certain moment does not, and after that moment merely does, contain and deliver the
characteristic value “–1”. In favour of this model speak

1)       that the infinity can only benefit the mathematical manageability of HEP

2)       that a possibly relevant aspect of reality (namely end of tape)
finds its representation herein.

(I am aware that other possibly relevant aspects of reality
have fallen under the table. The width of the paper one might wish to use
for the determination of the choice between alternative codes; the colour of the paper or
the factory number of the flexowriter on which a tape was punched one might perhaps wish to
know for diagnostic purposes upon the occurrence of reading or punching errors!)

For the time being I see no necessity to be able to give HEP “a pushback”. Should
this necessity arise, then a sensible choice must be made between the
almost equivalent possibilities that will still come up for discussion in connection with CHAR.

The routines of the type CHAR.

This section deals only with CHARF, because I am insufficiently
acquainted with the teleprinter and, on account of unfamiliarity with standard practices in
telex traffic, I scarcely consider myself competent to make recommendations here.

The routine CHARF that I have in mind relates to an MC-flexowriter
without variable spacing, without backspace, with fixed setting of the line
feed and fixed spacing of the tabulator stops.

Initialization of CHARF takes place under specification of two parameters, the boolean
“lower case” and the integer “carriage position”, which at the beginning of the line is
set to 0. Thereafter CHARF delivers per invocation the report of the page image, namely
per position on the page. (Each “escaping” keystroke is reported fourfold
if it is case-independent (i.e. the space) and eightfold if it is case-dependent,
namely underlined or not and struck through or not). Within the value range
of CHARF, NLCR —i.e. transition to a new line— is also comprised, TABs
are translated into an equivalent number of spaces, spaces and TABs at the end
of the line are, on account of their invisibility, not passed on.

We can (see above) e.g. define basic symbols either on a Flexowriter page
or on a teleprinter page, and we have required of e.g. a routine
such as SYM that it can, behind the scenes, switch from the one code to the other.

These transitions must be steered by the tape. Of every routine of
type CHAR I imagine that there is a characteristic (self-terminating) punching
(or punch combination) which entails that herewith the prevailing interpretation is, until
further notice, finished. For the procedure CHARF I imagined using for this purpose the
stop code in the meaning “END OF CODE”.

Because for routines that work in terms of CHARF I most certainly want to permit
that they read non-self-terminating units of the page, CHARF will indeed be able to have a pushback
of a single value.

For the routine CHARF the following role looms up.

The state “CHARF current” —i.e. basint = 2— can only be brought about
by the invocation of an initialization procedure, whereby the status variables of
CHARF (namely “prevailing case” and “carriage position”) are specified. By the initialization procedure
the pushback is set to “empty”.

The giving of a pushback could be done in two ways

1)
by the parameterless command “at the next invocation, repeat first once more the last
delivered value”

2)
by the parameter-bearing command “at the next invocation, give first the
value specified herewith”.

Because the second method is somewhat more flexible —and the first is moreover poorly
defined immediately after initialization— I prefer the second version.

If a pushback is given to CHARF at a moment when CHARF is not
current, this action has no effect.

If a pushback is given to CHARF while a not-yet-returned
pushback was hanging, then the latter pushback overwrites the former.

A side effect of CHARF will have to be to terminate, in consequence of the encountering
of the stop code, the state “basint = 2” (i.e. CHARF’s currency).
More precisely: just as CHARF does not pass on invisible spaces at the end of the line,
CHARF will also not report invisible NLCLs when finally
a stop code is found on the tape. The occurrence of the stop code is passed on to the
invoking program by CHARF returning with the value “–1”,
now in the meaning “END OF CODE”. If CHARF is invoked during
“basint = 2” and it returns with the value “–1”, then this has “basint := 0”
as a side effect. (This holds also if this –1 comes from the pushback; herewith therefore
a way is given to let CHARF, should it be current, terminate its
currency.) If CHARF is invoked during “basint ≠ 2”, then
the control likewise returns with CHARF = –1, but this time without side effect.

In the case of non-currency of CHARF, the giving of a pushback has no (side)
effect, nor does the invocation of CHARF.

With this definition of the falling-away of the currency I have achieved the following:
if SYM reads from the flexowriter page a non-terminating symbol, followed
by a basic symbol that occupies 1 position, followed by a stop code, then
SYM has, upon delivering the first symbol, already seen the next position. CHARF
has perhaps already seen the stop code, but not yet passed it on, so that it has not yet terminated its
currency. Thus the last position description can still
be pushed back into CHARF.

If CHARF, on account of passing on the stop code, has brought about for basint the transition from 2
to 0, then (if the stop code has come via the tape) the
synchronization with HEP is defined: after this an invocation of HEP delivers the first
heptad that follows upon the stop code. (And this is actually the only moment at which
I would want to define synchronization between CHAR and thereafter HEP explicitly.
More extensive synchronization rules soon acquire something contrived, and moreover: heaven
knows what crazy apparatus we shall have to take into the system tomorrow!)

Influencing of basint.

Direct HEPping is not compatible with the currency of CHARF
or CHART. The question is how the system reacts when one tries to violate these (and similar)
rules. In particular:

1)  actualization of CHARF while basint ≠ 0

2)  actualization of CHART while basint ≠ 0

3)  direct HEPping while basint = 2 or = 3.

If basint ≠ 0, this means quite explicitly: that we have until further notice
committed ourselves to a certain tape interpretation. One can say: initializing
CHARF or CHART then once more is evidently misplaced (and attach to that the consequence of
a lethal program error); one can also say: the redefinition of
the tape interpretation is then evidently that “further notice” and it will be all right.
Out of opportunistic considerations I incline to the latter; to arguments that it
is worth the trouble to treat this as a lethal program error I stand open.

Direct HEPping while basint = 2 or = 3 is stranger, because we have indeed
taken the standpoint that, in the case of currency of a CHAR, the synchronization
with HEP is in general undefined. To brand this as a lethal program error
is, however, too excessive for me. For that the primitive HEP is too fundamental to me.
Direct HEPping does indeed mean that any CHAR interpretation
is no longer valid. I therefore propose to let HEP have as side effect

“if basint ≠ 1 then basint := 0” .

(if HEP is invoked with basint = 2 from within CHARF, then CHARF itself can, before being left
with “basint := 2”, undo this side effect again.)

The initialization of CHARF and CHART are the only way to get basint = 2 or
= 3; further the programmer will be allowed to set basint = 0 or basint = 1.

The choice between symhep, symcharf and symchart

The choice between symhep, symcharf and symchart depends on the prevailing value
of basint. In other words, whoever knows that his SYM will have to work via symcharf can achieve this
by initializing CHARF before the first invocation of SYM; likewise
he can a priori dictate interpretation according to symchart or symhep via basint. In this way
it is possible to read flexowriter tapes via SYM without these
identifying themselves as such in their heading. If, however, we do not want to impose the choice a priori
but to let it depend on the tape, then we can do this by

1) setting the variable basint to 0 before the first invocation of SYM.

2) requiring punching conventions, so that at the beginning of the tape it is to be seen with which
value of basint reading must, until further notice, proceed. SYM can then itself, if necessary,
actualize CHARF or CHART. (For the flexowriter heading I would want to propose
that after skipping blanks, erases (and stop codes?), first an NLCR or a case definition;
for teleprinter tapes after skipping blanks first a CR, an NL or a case definition;
the CHARF or CHART initialization inserted by SYM will, for the unknown
parameter, make a standard choice, e.g. beginning of the line, lower case or
lettershift.)

In order to arrive at a normalization of these headings I make an appeal to all interested parties.
I then expect that everyone will instruct his punching room in such a way that
the tapes punched there —especially ALGOL programs and data tapes— begin in conformity
with that convention, even though he has no need of this for his own operation. At the same time
I expect of every limited implementation of one's own operation (say “flexowriter tapes
only”) that it accepts those flexowriter tapes with normalized heading.

Initialization of SYM.

As regards the choice of interpretation process, the status of SYM is thus regulated
by basint. We are still saddled with the fact that SYM too (precisely SYM!) must be able to have a pushback of a
single value. If we define the giving of a pushback by
overwriting of any pushback still hanging (as with CHAR) then we can
initialize SYM in this respect by giving a pushback and invoking SYM
once.

Initialization of NUM.

The manner of interpretation of SYM is fully determined by basint; this does not hold
for NUM: for if basint e.g. points to CHARF as current, then we have
still the choice between numcharf and numsym via symcharf. NUM
therefore has yet a boolean status variable, which we call “numint” and which
indicates whether numsym is current. If numint is true, then basint determines only the
choice between symcharf, symchart and symhep; if numint is false, then basint determines
also the choice between numcharf, numchart and numhep. The boolean “numint” shall be
settable in both directions.

I propose to let the third conceivable state (numint undefined)
lapse and, in all doubtful cases, to set numint true. (At the beginning of a
new tape.) This is a choice that gives a certain preference to the manner of interpretation numsym.

For: let —under continued currency of CHARF— NUM have worked for a while via
numsym, and let us now wish to indicate on the Flexowriter page that
from now on, until further notice, NUM must work via numcharf. The fact that numint
must become false must be reported by the tape, with which NUM has contact only via SYM.
This can be done in various ways; I mention

1) by agreeing that the occurrence of certain number separators will be interpreted by NUM
as “go over to numint = false”

2) by introducing an extra basic symbol with this meaning

3) by reporting everything that is not interpretable as a basic symbol as “nonsense”
and giving that the same meaning.

(I am coming more and more to feel for the first possibility.)

In all cases we can ensure that a tape is not going to be read via numsym
by indicating this in the heading. The result is that NUM reads tapes
via numsym, unless otherwise stated. This seems in accordance with what we wish.

The processing of “END OF TAPE”.

So far we have spoken only of how HEP reacts to end of tape.
There we agreed that the occurrence of END OF TAPE may indeed belong to the
semantic information, and HEP is not equipped with automatic requesting
of a follow-on tape. Now we must decide this for the following levels (CHAR, SYM and NUM).

At the level of CHAR too I would not yet want to build in automatic follow-on-tape requesting.
Stronger still: I found it precarious to let the code continuity assumed by CHAR
extend over physically separated pieces of tape, and I therefore propose
to let CHARF —when it detects “END OF TAPE” via HEP— react to this detection
as to the stop code and to report on “END OF CODE” (and then
to carry out basint := 0). The program invoking CHAR can then inspect via HEP
whether there is yet more tape, if not, decide whether it wants to see more etc.

The consequence of this is the requirement that if we want to be able to read with SYM
in various codes, then in any case at the beginning of every physical tape there stands
the code-identifying heading. If this condition is satisfied, then we can assemble a
long tape from a number of partial pieces by pasting; when all
partial pieces end with the stop code, then this pasting process can even proceed without paying attention
to whether code switches occur at those splices.

All this I feel as a strong indication to adorn SYM indeed with automatic requesting
of a follow-on tape. (Under the motto: it need not penetrate to the SYMming program
that those logically now unconditionally permitted splices have perhaps
not de facto been carried out). And I even imagine, at this level,
no longer attaching any semantic meaning to the physical end of tape (and then also
consistently, so not e.g. reporting it once more). That a program that picks basic
symbols from a tape is then guaranteed insensitive to the code used
and to the physical number of tapes seems to me a greater good than the loss of
the possibility to write with SYM a program that counts the number of basic symbols
on a tape.

It is up to SYM to request a new tape, namely when during inspection
(basint = 0) it appears that HEP delivers the value –1. The requesting of a new
tape shall have as side effect that numint = true is set.

A few more remarks about the synchronization.

We assume that NUM, via numsym, picks the numbers as non-self-terminating units
out of the basic symbol stream. (E.g. the sign of the next number can be used as
separator.) If only NUM were used, this could be realized in two
ways.

1) the over-read basic symbol is pushed back into SYM

2) the over-read basic symbol is stored in NUM.

As soon as we NUM and SYM alternately, this does make a difference. I believe that
everyone prefers method 1).

If, however, SYM is defined in terms of CHARF, then now also an alternation
between NUM and CHARF is defined. Let it be noted that after NUM the CHARFing
scans the page image, beginning after the over-read basic symbol!

Whoever wants to scan the tape sequentially “without skipping” will, after NUM, have to do SYM
once first and only then CHARF. An open question is whether SYM
must retain its pushback upon direct CHARFing, CHARTing or HEPping. I am inclined to have to answer this
question in the negative; otherwise, namely, after CHARF, repeated SYMming
has nothing more to do with scanning in succession, and that might well become “a trouble
shooter’s nightmare”.

The last number.

If the task “count the basic symbols on this tape” seems to us somewhat far-fetched, programs
that must process an unknown number of numbers are quite familiar. The trick “put
the count at the front” we all know, and we all know how annoying this
can be.

I could imagine that beside NUM we introduce a boolean procedure NUMMARK
and define this pair together on an arbitrary sequence of numerical
values and so-called “marks” (whereby a mark functions as a number separator).

The boolean procedure NUMMARK gives an answer to the question whether, upon reading on,
a “mark” comes first; if so, then this mark is thereby passed. In the various
codes this must be realizable without giving NUM more status variables.
(If we do not equip HEP with the pushback facility, this costs us one heptad per
number, but over that I would not want to make a fuss).

transcribed
by Martin van der Burgt

revised 20-Nov-2013
