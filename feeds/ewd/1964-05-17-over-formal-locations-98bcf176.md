---
title: "Over formal locations (EWD89)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD89.html
published: "1964-05-17T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD89.PDF
---

# Over formal locations

8 juni 1864

Over formal locations

(Résumé van de besprekingen van de werkgroep, gehouden op 8 juni 1964)

Het volgende is een résumé, dat ik uit mijn blote hoofd opstel; in hoeverre het waarheidsgetrouw is, kan ik dus niet garanderen.

We hebben bij de kop gevat het geval, dat de formele parameter gespecificeerd is als real of integer. We zullen dit noemen "arithmetic": we hebben nl. besloten de vertaler tussen deze twee specificaties geen onderscheid te laten maken en altijd het type van de actuele parameter — mits natuurlijk real of integer — te laten prevaleren.

De prijs die we bereid zijn te betalen, bestaat zo op het oog uit vrij omvangrijke formal locations en een analyse van de meegegeven actuele parameters bij binnenkomst in de procedure.

Over de omvang van de formal locations: wij komen zo op het eerste gezicht op vier woorden per parameter.

Over de test: wij stellen ons voor, dat de meegegeven actuele parameters getest worden, d.w.z. aan een controle onderworpen worden, die op grond van de specificaties aan gelegd kan worden (wij stellen specificaties verplicht). Tot nog toe hebben wij ons geen zorgen gemaakt over de snelheid, waarmee deze test aangelegd kan worden ("Is het dit, of dat, of dat etc." Als je eerst vraagt naar het meest voorkomende type actuele parameter, dan is het waarschijnlijk nog niet eens zo tijdrovend.)

Wij nemen ons voor om deze acceptabiliteitstest zover door te voeren, dat daarna de tests, die nog dynamisch uitgevoerd moeten worden, een soort van sinecure zijn.

Omdat de type-overgangen speciale complicaties met zich meebrengen, gaan we het daar eerst over hebben.

We stellen ons voor, dat de arithmetiek altijd in het drijvend register F wordt uitgevoerd. (Uitgezonderd misschien n := n + 1 of zoiets doms.)

Om een integer als operand, als primary, in een expressie te betrekken, is dankzij de speciale representatie van drijvende getallen in de X8 geen probleem. Moeilijker is de assignment aan een integer variabele. Dit kan op twee manieren moeiljkheden opleveren: het getal kan niet geheel zijn, het getal kan (na evenetuele afronding!) te groot zijn.

Als F geïntegreerd moet worden, lassen we in het programma in:

U, A = M[57], Z kop F = 0?

N, SUBCD(INTEGREER)

waarin de subroutine "INTEGREER" — hiervoor moeten we een kortere naam bedenken — luidt (begint met)

INTEGREER:       F + AFR                    AFR = 3 * 2 ↑ 38

F - AFR

U, A = M[57], Z

Y, GOTOR(MC[-1]) →  en terug

________________________  capaciteitsoverschrijding

Opm. De constante AFR is zo gekozen, dat mits F van te voren niet te groot is (abs. < 2 ↑ 38) de som F + AFR met de binaire komma naaar rechts geechelonneerd staat. En passant verzorgt dan het fatsoen van de rekencursist van de EL-X8 de correcte afronding. Vervolgens wordt AFT er weer afgetrokken. Nu is F in elk geval geheel. F kan nog te groot zijn (veel te groot zelfs); vandaar dat de kop van F weer aan een nultest onderworpen wordt.

Een en ander kost ons in het hoofdprogramma twee opdrachten per integer assignment aan plaatsruimte.

Aan tijd

in het normale geval (geen afronding nodig): 2 geh. cont

als transferfunctie nodig (wel afronding): 12 geh. cont. + exces tijd voor drijvende optelling

Ik verwacht, dat afronding zo weinig voorkomt, dat ik er weinig voor voel om — als standaardpraktijk tenminate — ten koste van vier extra opdrachten in het objectprogramma de afronding vier geheugencontacten te versnellen.

U, A = M[57], 4

Y, JUMP (4)          →

F + AFR

F - AFR

U, A = M[57], Z

N, GOTO              →           capaciteitsoverschrijding

→

--------------

Tot zover het probleem van afronding en range analyse. (In geval van capaciteitsoverschrijding zullen we de plaats waar ook nog wel een beetje moeten aangeven. Daar mogen we later dan over denken)

We keren nu terug tot de formal locations.

Het type van de actuele parameter deert ons, zoals bekend, weinig, wanneer de formele in de expressie-situatie uitgewerkt moet worden. Als hij aan de linkerkant van de assignment voorkomt, treedt er nog meer ellende op, zodra we een multiple assignment hebben. Immers:

als een formele linkerkant in zijn eentje voorkomt, heeft een integer actuele tot gevolg, dat er afgerond kan moeten worden;

als een formele linkerkant, samen met een real linkerkant voorkomt, moet een integer actuele tot gevolg hebben, dat er een foutindicatie gegeven wordt, omdat in een multiple assignment de linkerkanten van hetzelfde type moeten zijn;

als in een multiple assignment alle linkerkanten formeel zijn — dankzij de binnenprocedure kunnen dit formelen van verschillende hoogte zijn! — moet de foutindicatie bij verschillende types gegeven worden, en afronding moet bij (homogeen) integer actuelen ingelost worden.

We hebben dus besloten om de multiple formal assignment voorlopig te laten voor wat hij is, d.w.z. het normale geval niet door het bestaan van de multiple formal assignment te laten vertragen. (We hebben voor de multiple formal assignment wel een oplossing, maar die is dan maar een beetje tijdrovend.)

Vast staat, dat we de assignment aan de nonformele scalar exceptioneel zullen behandelen. Als de vertaler in een scan van links naar recht de assignment statement leest, worden de nonformele scalaire linkerkanten opgezouten en op homogeneiteit van type onderzocht (tezamen met de nonformele subscripted linkerkanten, die van hetzelfde type moeten zijn, maar wel meteen aanleiding tot "programma" geven.)

Als de vertaler het programma gegenereerd heeft, dat de expressie uitrekent in het F-register, test hij, of het verzamelde type van de linkerkant van type integer is, zo ja, dan wordt de integratie ingelast. Vervolgens genereert de vertaler voor elke opgezouten nonformele linkerkant een Mp[q] := F of een Mp[q] := G, afhankelijk van het gevonden type. Tenslotte komen de subscripted left hand sides aan bod. (waarover later vast nog een hele hoop meer). Tot zover was er niets formeel.

Nu gaan we bekijken geval van de single formal assignment.

Hier staat de taal als actuele parameter toe:

1e een real scalar
2e een integer scalar
3e een subscripted real
4e een subscripted integer
5e een formal.

De laatste mogelijkheid confronteert ons echter met niets nieuws, want het meegeven van een reeds formele parameter als actuele is niet meer dan copieren van de formal locations; de laatste mogelijkheid geeft dus geen nieuwe vrijheidsgraad aan de formal locations.

De eerste opmerking is, dat we bij een formele linkerkant eerst deze linkerkant moeten analyseren en zelfe moeten uitwerken, in het geval het een subscripted variable is. De uitwerking van de expressie kan immers als neveneffect hebben, dat de subscript evaluation na afloop andere waarden op zou leveren. (Denk maar aan "R[read] := read".)

Om die reden zullen we de formele linkerkant eerst moeten "uitwerken", d.w.z. indentiteit van de variabele moeten vaststellen. (In geval van meer formele en/of subscripted linkerkanten werken we deze af in volgorde van links naar rechts.

Het uitwerken van een formele linkerkant zal aanleiding geven tot een stapelvulling, onder controle waarvan de waarde van de expressie, wanneer deze eenmal in het F-register gevormd is, zal worden weggenschreven. En het is van deze kant, dat we het systeem opbouwen: eerst stellen we vast, wat een plezierige stapelvulling is, opdat dit wegschrijven vlot en snel zal gaan. Hierna stellen we vast, wat geschikte vullingen van formal locations zijn, om bij de uitwerking van de formele linkerkant deze gewenste stapelvulling tot stand te brengen. De stapelvulling ten gevolge van uitwerking van een formele linkerkant noemen we een "linkerkantwaarde".

Op het moment, dat de wegschrijving moet plaatsvinden, staat de waarde in F en de linkerkantwaarde is het top-element van de stapel.

We zullen voor de verschillende gevallen vershillende gestructueerde linkerkantwaarden moeten introduceren. Deze verschillende structuren zijn echter gekoppeld door de eis, dat de feitelijke assignments moeten geschichten op grond van precies dezelfde indicatie in het (aan de body ontleende) objectprogramma. We kiezen deze indicatie zo, dat

a)  de in beslag genomen ruimte in het objectprogramma niet te groot is

b) het meest voorkomende geval — actuele een real in de stapel zo vlot mogelijk verwerkt wordt.

Één opdracht in het objectprogramma is kennelijk wel het minimum en we gaan exploreren, wat de gevolgen zijn, als we hiervoor kiezen

"DO(MC[-1])"

d.w.z. een execute op de top van de stapel met impliciete aflaging van het B-register.

We gaan nu na, wat in de verschillende gevallen de aangemeten linkerkantwaarde moet zijn.

1. Een real op een vast physisch adres "a". Dit doet zich voor als de real een scalar in de stapel is, of een element van een klein array. Bladzijden van grote arrays zullen immers in het algemeen niet op heilige pagina's staan.

1.1. a < 32768

In dit geval bestaat de linkerkantwaarde uit een enkel woord (bij de weergave van stapels is de stapelbodem boven!):  B - 1: M[a] := F
B : ---------

Tijsduur 4 geheugencontacten.

1.2. a ≥ 32768
Dit geval, dat zich alleen voordoet als men beschikt over een geheugen, dat groter is dan 32K behandelen we volledigheidshalve wel. We kunnen dan niet volstaan met een linkerkantwaarde van een enkel woord. Dan moet dus b.v. B met meer dan 1 worden afgelaagd, dit kan door het topwoord van de linkerkantwaarde een subroutinesprong te laten zijn. Een mogelijke structuur van de linkerkantwaarde is:  B - 2:             + a
B - 1: SUBCD(:q1)
B      : ----------

Waarbij adres :q1 de entry van het systeem is en wel

q1:  S = MC[-2]
MS[0] = F
GOTOR(MC[0]) ⇒  S := a, B := B - 1
berg
B := B - 1 en terug.

Opm. Wij vestigen er de aandacht op, dat hier wel een stukje gemeen programmeren wordt weggeven. De DO-operatie laagt eerst B af; de uiteindelijke opdracht blijkt een stapelende subroutinesprong te zijn, die zichzelf met de link overschrijft en B weer met 1 ophoogt. De subroutine bestaat uit drie opdrachten, waarin twee stapelverwijzingen de twee nodige B-aflagingen impliciet bewerkstelligen.

Tijdsduur: 10 geheugencontacten.

2.  Een integer op adres a
Ook bij integers kan het geval a < 32768 apart behandeld worden; het verschil is echter gemarkeerd.

2.1. a < 32768
De structuur van de linkerkantwaarde kan zijn  B - 2 : M[a] = G
B - 1 : SUBCD(:q2)
B       : ----------

Met q2: U, S = M[57], Z

Y, DO(MC[-2])
Y, GOTOR(MC[)])
F + AFR
F - AFR
U, S = M[57], Z
Y, DO(MC[-2])
Y, GOTOR(MC[0])
--------------

→ zonder meer acceptabel

→ na afronding acceptabel
capaciteitsoverschrijding

Tijdsduur minimaal 9 geheugencontacten.

2.2. a ≥ 32768
De structuur van de linkerkantwaarde kan zijn:  B - 2 : + a
B - 1 : SUBCD(:q3)
B       : ----------

met q3:      S = MC[-2]
U, S = M[57], Z
Y, MS[0] = G
Y, GOTOR(MC[0]) → zonder meet acceptabel
.
.
.
. etc.
.
.

Tijdsduur minimaal 10 geheugencontacten.

3.  Een real op een bladzijde
De linkerkantwaarde beslaat weer twee woorden  B - 2 :  pakking van physisch adres van de BZV en regelnummer
B - 1 :  SUBCD(:q4)
B       :  ----------

De wijze, waarop de subroutine, die op :q4 begint toegang heeft tot de gepakte gegevens is na de vorige routines nu wel duidelijk. Wij merken op, dat wanneer het uit tijdsoverwegingen voldoende gunstiger uitkomt, we hier een linkerkant ook wel drie woorden kunnen laten beslaan, door nl. het physisch adres van de BZV en het regelnummer gescheiden, ieder in een eigen woord op te bergen.

4. Een integer op een bladzijde  B - 2 : pakking van physisch adres van de BZV en regelnummer
B - 1 : SUBCD(:q5)
B       : ----------

De routine, die bij q5 begint, begint (zie 2) af te ronden, en verloopt verder analoog aan die van q4. Ook hier kunnen desgewenst de hier gepakte gegevens in twee aparte woorden geborgen.

Uitwerking van formele variabelen

Wij moeten nu, voor elk type van actuele parameter gaan vaststellen, welke structuur de formal locations zullen vertonen. De formal locations bepalen de actuele parameter, die in het algemeen op twee wijzen kan moeten worden uitgewerkt, nl. als linkerkant en als rechterkant. In het laatste geval stellen we als eis, dat de waarde in het F-register verschijnt, in het eerste geval, dat de linkerkantwaarde, als boven beschreven, aan de stapel wordt toegevoegd.

We gaan uit van vier formal locations, in volgorde van opklimmend geheugenadres beschreven als f[0], f[1], f[2], en f[3]. We spreken af, dat uitwerking als rechterkant in het objectprogramma van de body gecodeerd zal worden als

DOS (f[0])

en uitwerking als linkerkant als

DOS (f[1])

Opm. 1. De formal locations zullen, als grootheden in de stapel, via de dynamische adressering geadresseerd worden.

Opm. 2. Het meegeven van een formele parameter als actuele willen we effectueren, door "ongezien" copieren van de formal locations. We moeten er dus voor zorgen, dat de structuur van de formal locations zodanig is, dat ze tegen verplaatsing (nl. copiering elders) bestand zijn.

We volgen nu dezelfde indeling.

1. Een real op een vast physisch adres a

1.1. a < 32768 f[0] : F = M[a]
f[1] : SUBCD(:q6)
f[2] : M[a] = F
f[3] : voor alsnog ongebruikt.

Opm. f[3] hebbe wel de goede pariteit in verband met copiering van de formal locations.

De subroutine, die op q6 begint, is even wat gecompliceerd, omdat de link boven op de stapel komt en het juist deze plaats is, die gevuld moet worden. Dit impliceert "redden van de link".

De systeemroutine luidt

q6 :

G = MS[1]

S = M[B-1]  red link

M[B-1] = G

GOTOR(M[60])      ⇒ terug via S-register.

Tijdsduur voor linkerkantuitwerking: 10 geheugencontacten.

De rechterkantuitwerking spreekt voor zichzelf en duurt 4 geheugencontacten.

1.2. a ≥ 32768 Volledigheidshalve geven we de formal locations voor een vast gesitueerde real boven de 32 K:
f[0] SUBCD(:q7)
f[1] SUBCD(:q8)
f[2] + a
f[3] SUBCD(:q3)
Hier geeft q8 de systeemroutine aan:

q8 :

F = MS[1]

S = MC[-1]

MC[0] = F

GOTOR(M[60])

een routine, die grote overeenkomst vertoont met q6; de netto verhoging van B met 1 wordt door - 1 + 2 bewerkstelligd. De tijdsduur voor linkerkantuitwerking is nu 12 geheugencontacten.

De systeemroutine q7 luidt

q7 :

S = MS[2]

F = MS[0]

GOTOR(MC[-1])

en vergt 10 geheugencontacten.

2. Een integer op een vast physisch adres a

2.1. a < 32768

f[0] G = M[a]
f[1] SUBCD(:q8)
f[2] M[a] = G
f[3] SUBCD(:q2)

Hierbij wordt dezelfde systeemroutine gebruikt (nl. q8) als in geval 1.2. Uitwerking van een linkerkant kost 12 geheugencontacten, uitwerking van een rechterkant 3

2.2. a ≥ 32768
Volledigheidshalve geven we dit geval weer
f[0] SUBCD(:q9)
f[1] SUBCD(:q8)
f[2] + a
f[3] SUBCD(:q3)

waarbij q9 (analoog aan q7) luidt     q9 :

S = MS[2]
G = MS[0]
GOTOR(MC[-1])

Uitwerking van een rechterkant duurt dan 9 geheugencontacten, die van een linkerkant als gebrukelijk 12.

3. Een real op een bladzijde

De actuele parameter is dan een subscripted element. We nemen dat deze gegeven is door een impliciete IS, die (zo ongeveer) het element localiseert in termen van physisch adres van de BZV en regelnummer. Aanwezigheidsanalyse van de bladzijde wordt door de impliciete subroutine nog niet gepleegd.

De impliciete subroutine wordt in de formal locations gekarakteriseerd door een "invariant beginadres". We laten in het midden of hierin de BZV van de pagina gegeven is door zijn physische adres, dan wel door zijn adressering als locale van het buitenste blok (eigen bladzijde) of de bibliotheeklijst. Physisch adres is waarschijnlijk het snelste.

De inhoud van de formal locations is dan

f[0]      SUBCD(q10 of q11)
f[1]      SUBCD(q12 of q13)
f[2]       invariant beginadres implicitie subroutine
f[3]      context D.

Routines q10 en q12 zijn bedoeld voor het geval, dat de IS simpel is, wat o.a. impliceert, dat de pagina van de body niet geprofaneerd hoeft te worden en er in de stapel per definitie voldoende anonieme ruimte zal zijn. In dit geval kan de machinelink als terugadres gehandhaafd worden en hoeft alleen de heersende D aan de terugkeergegevens toegevoegd te worden. Anders (q11 en q13) moet de link in invariant terugkeeradres terugvertaald worden en de pagina van de body geprofaneerd worden.

Onder controle van S hebben deze routines toegang tot de IS en de context D, die ingevuld moet worden om de IS correct te kunnen uitvoeren. Verder moet de bladzijde waar de IS begint heilig aangevraagd worden. (Als hij al heilig aanwezig is, betekent dit al, dat HHT = 2 wordt!)

Na afloop van de IS keert de besturing terug in de systeemroutine. Afhankelijk van linker of rechterkant wordt of alleen de pakking van BZV-adres en regelnummer — door de IS afgeleverd — op de top van de stapel gezet en dit afgedekt met de constante "SUBCD(:q4)" of wordt selectie gepleegd — inclusief aanwezigheidscontrole — en de gevraagde real in F gezet.

4.  Een integer op een bladzijde De inhoud van de formal locations is dan

f[0]      SUBCD(q14 of q15)
f[1]      SUBCD(q16 of q17)
f[2]      invariant beginadres impliciete subroutine
f[3]      context D.

Deze routines verschillen slechts van de vorige vier, doordat bij linkerkantuitwerking de pakking met "SUBCD(:q5)" wordt afgedekt en bij rechterkantuitwerking alleen het geselecteerde woord in G geplaaatst wordt.

Opm. 1  Zodra we ook "kleine arrays" in de stapel willen toestaan, dan zullen (kunnen) de laatste acht subroutines na terugkeer uit IS inspecteren, of het aangewezen element een van een klein array is. In dat geval is de linkerkantwaarde als van type 1 of 2, de rechterkantwaarde is dan triviaal.

Opm. 2 Deze acht subroutines lijken verdacht veel op elkaar en het ligt dus voor de hand om ze van gemeenschappelijke routines gebruik te laten maken. Men lette erop, dat alle status informatie dan op de stapel moet komen, omdat ze genest en parallel gearchiveerd kunnen worden — d.w.z. door verschillende programma's door elkaar gebruikt kunnen worden.

Hiermede zijn alle actuele parameters, corresponderend met een specificatie arithmetic behandeld voorzover zij een bestaanbare linkerkantwaarde hebben. De overige zijn de constante, de expressie en de arithmetic procedure SS (Self Supporting, d.w.z. een procedure identifier behorend bij een procedure zonder parameter).

Bij binnenkomst in de procedure zullen de formal locations op aanwaaardbaarheid getest moeten worden; hier wordt niet getest, of een arithmetic actual ook inderdaad een linkerkantwaarde heeft. Daarom zullen wij deze formal locations (met name f[1]) zo vullen, dat als de body probeert, hun linkerkantwaarde uit te werken, dit alsnog tot protest aanleiding geeft.

5.  De real constante      f[0]
f[1] F = MS[2]
Alarmsprong

f[2]    ⎱
f[3]    ⎰  waarde van de constante

Tijdsduur rechterkantuitwerking: 4 geheugencontacten.

6. De integer constante (Dit is misschien een beetje te veel eer!)

f[0]
f[1]
f[2]
f[3]  G = MS[3]
Alarmsprong
invariant beginadres impliciete subroutine
waarde integer constante

Tijdsduur rechterkantuitwerking: 3 geheugencontacten.

7. De arithmetische expressie    f[0]
f[1]
f[2]
f[3] SUBCD(q18 of q19)
Alarmsprong
invariant beginadres impliciete subroutine
context D.

Het onderscheid tussen de systeemroutines q18 of q19 is weer of de impliciete subroutine simpel is of niet.

8. De enkele type procedure identifier, die als expressie op mag treden De inhoud van de formal location wordt, zodra we goed en wel ons in de body befinden

f[0]
f[1]
f[2]
f[3] SUBCD(:q20)
Alarmsprong
invariant beginadres van de procedure
context D

De moeilijkheid hier is, dat de procedure ook expliciet als procedure aangeroepen kan worden. Nu stellen wij one voor om bij de call de formal locations van de parameters te vullen, vervolgens dit aantal mee te geven en dan naar de (nog "ongeziene") procedure te springen, die begint te testen of het aantal meegegeven parameters correct is. De body van de procedure zonder parameters begint dus te testen of hij inderdaad geen parameters heeft meegekregen.

De functie van de systeemroutine q20 is om het stukje calling sequence behelzends "nulparameters" dynamisch in te lassen, voordat volgens alle regels van de kunst de procedure geactiveerd wordt.

De formal location vulling met SUBCD(:q20) kan echter slechts op twee manieren to stand komen

1e Door een arithmetic formal parameter — waarbij de actuele een procedure identifier was — als actuele parameter door te geven. Doorgeven is immers copieren van de formal locations.

2e Door een arithmetic procedure identifier als actuele parameter mee te geven, waar de formele als arithmetic gespecificeerd blijkt te zijn. Het meegeven van een procedure identifier als actuele parameter geschiedt door te vertaler:

1e

zonder bewustzijn van het aantal parameters, dat deze procedure vergt

2e

zonder bewustzijn of de overeenkomstige formele als procedure of als arithmetic gespecificeerd is.

Als de formele als procedure gespecificeerd is, dan inspecteert de body de formal locations om te kijken of er een procedure van het goede type is meegegeven. Bij gebruik van deze als procedure gespecificeerde parameter zal de callside het feitelijk aantal parameters (kan = 0 zijn) expliciet aangeven.

Als de formele parameter als arithmetic gespecificeerd is, dan wordt als overeenkomstige actuele een procedure identifier van het goede type geaccepteerd, maar de formal locations worden gewijzigd door de invulling van "SUBCD(:q20)" in f[0]. Dit proces vindt plaats ongeacht het aantal parameters, dat de meegegeven procedure graag zou willen hebben. Bij dynamisch gebruik wordt dit in de procedure getest; q20 zorgt nl. voor expliciete meegave van "nulparameters". (Ook dan hoeft de Alarmsprong pas in de formal location ingevuld te worden: bij een procedure, die als zodanig gespecificeerd is, is een alarmsprong immers een open deur!)

transcribed by Carl Ludwigson
revised Thu, 4 Nov 2010

---

## English translation

### On formal locations

8 June 1864

On formal locations

(Résumé of the discussions of the working group, held on 8 June 1964)

The following is a résumé that I draw up off the top of my head; to what extent it is faithful to the truth I cannot therefore guarantee.

We have tackled head-on the case in which the formal parameter is specified as real or integer. We shall call this "arithmetic": we have namely decided to let the translator make no distinction between these two specifications and always let the type of the actual parameter — provided of course it is real or integer — prevail.

The price we are prepared to pay consists, on the face of it, of rather sizable formal locations and an analysis of the supplied actual parameters upon entry into the procedure.

On the size of the formal locations: at first sight we arrive at four words per parameter.

On the test: we envisage that the supplied actual parameters are tested, i.e. subjected to a check that can be set up on the basis of the specifications (we make specifications obligatory). Up to now we have not worried about the speed with which this test can be set up ("Is it this, or that, or that, etc." If you ask first about the most commonly occurring type of actual parameter, then it is probably not even all that time-consuming.)

We intend to carry this acceptability test through to the point where the tests that must still be carried out dynamically are a kind of sinecure.

Because the type transitions bring along special complications, we shall discuss those first.

We envisage that the arithmetic is always carried out in the floating register F. (Excepting perhaps n := n + 1 or some such silly thing.)

To involve an integer as an operand, as a primary, in an expression is, thanks to the special representation of floating numbers in the X8, no problem. More difficult is the assignment to an integer variable. This can cause difficulties in two ways: the number may not be whole, the number may (after possible rounding!) be too large.

If F must be integrated, we insert into the program:

U, A = M[57], Z kop F = 0?

N, SUBCD(INTEGREER)

in which the subroutine "INTEGREER" — for this we must devise a shorter name — reads (begins with)

INTEGREER:       F + AFR                    AFR = 3 * 2 ↑ 38

F - AFR

U, A = M[57], Z

Y, GOTOR(MC[-1]) →  and back

________________________  capacity overflow

Note. The constant AFR is so chosen that, provided F is beforehand not too large (abs. < 2 ↑ 38), the sum F + AFR stands echeloned with the binary point to the right. En passant the propriety of the arithmetic apprentice of the EL-X8 then takes care of the correct rounding. Subsequently AFT is subtracted off again. Now F is in any case whole. F may still be too large (much too large even); hence the head of F is again subjected to a zero test.

All of this costs us in the main program two instructions per integer assignment in space.

In time

in the normal case (no rounding needed): 2 mem. cont

if transfer function needed (rounding does occur): 12 mem. cont. + excess time for floating addition

I expect that rounding occurs so seldom that I feel little inclined to — as standard practice at least — at the cost of four extra instructions in the object program speed up the rounding by four memory contacts.

U, A = M[57], 4

Y, JUMP (4)          →

F + AFR

F - AFR

U, A = M[57], Z

N, GOTO              →           capacity overflow

→

--------------

So much for the problem of rounding and range analysis. (In case of capacity overflow we shall also have to indicate the location somewhat. We may think about that later.)

We now return to the formal locations.

The type of the actual parameter troubles us, as is known, little when the formal must be worked out in the expression situation. If it occurs on the left-hand side of the assignment, still more misery arises as soon as we have a multiple assignment. For indeed:

if a formal left-hand side occurs on its own, an integer actual has the consequence that rounding may have to be done;

if a formal left-hand side occurs together with a real left-hand side, an integer actual must have the consequence that an error indication is given, because in a multiple assignment the left-hand sides must be of the same type;

if in a multiple assignment all left-hand sides are formal — thanks to the inner procedure these can be formals of different height! — the error indication must be given in case of differing types, and rounding must be discharged in case of (homogeneous) integer actuals.

We have therefore decided to leave the multiple formal assignment for the time being for what it is, i.e. not to let the normal case be slowed down by the existence of the multiple formal assignment. (We do have a solution for the multiple formal assignment, but it is then just a bit time-consuming.)

It stands firm that we shall treat the assignment to the nonformal scalar exceptionally. When the translator, in a scan from left to right, reads the assignment statement, the nonformal scalar left-hand sides are salted away and examined for homogeneity of type (together with the nonformal subscripted left-hand sides, which must be of the same type, but do immediately give rise to "program".)

When the translator has generated the program that computes the expression in the F-register, it tests whether the collected type of the left-hand side is of type integer; if so, the integration is inserted. Subsequently the translator generates, for each salted-away nonformal left-hand side, an Mp[q] := F or an Mp[q] := G, depending on the type found. Finally the subscripted left hand sides come up. (about which surely a whole lot more later). Up to this point nothing was formal.

Now we are going to look at the case of the single formal assignment.

Here the language allows as actual parameter:

1st a real scalar
2nd an integer scalar
3rd a subscripted real
4th a subscripted integer
5th a formal.

The last possibility, however, confronts us with nothing new, for the supplying of an already formal parameter as actual is nothing more than copying the formal locations; the last possibility thus gives no new degree of freedom to the formal locations.

The first remark is that, with a formal left-hand side, we must first analyse this left-hand side and even must work it out, in case it is a subscripted variable. The working-out of the expression may indeed have as side effect that the subscript evaluation would afterwards yield other values. (Just think of "R[read] := read".)

For that reason we shall first have to "work out" the formal left-hand side, i.e. have to establish the identity of the variable. (In case of more formal and/or subscripted left-hand sides we deal with these in order from left to right.

The working-out of a formal left-hand side will give rise to a stack filling, under control of which the value of the expression, once it has been formed in the F-register, will be written away. And it is from this side that we build up the system: first we establish what is a pleasant stack filling, so that this writing-away will go smoothly and quickly. After this we establish what are suitable fillings of formal locations so as to bring about, in the working-out of the formal left-hand side, this desired stack filling. The stack filling resulting from the working-out of a formal left-hand side we call a "left-hand-side value".

At the moment when the writing-away must take place, the value stands in F and the left-hand-side value is the top element of the stack.

We shall, for the different cases, have to introduce differently structured left-hand-side values. These different structures are, however, coupled by the requirement that the actual assignments must take place on the basis of precisely the same indication in the (body-derived) object program. We choose this indication such that

a)  the space taken up in the object program is not too large

b) the most commonly occurring case — actual a real in the stack is processed as smoothly as possible.

One instruction in the object program is evidently the minimum, and we shall explore what the consequences are if we choose for this

"DO(MC[-1])"

i.e. an execute on the top of the stack with implicit lowering of the B-register.

We now examine what, in the different cases, the fitted left-hand-side value must be.

1. A real at a fixed physical address "a". This occurs when the real is a scalar in the stack, or an element of a small array. Pages of large arrays will indeed in general not stand on sacred pages.

1.1. a < 32768

In this case the left-hand-side value consists of a single word (in the depiction of stacks the bottom of the stack is at the top!):  B - 1: M[a] := F
B : ---------

Duration 4 memory contacts.

1.2. a ≥ 32768
This case, which occurs only if one has at one's disposal a memory larger than 32K, we do treat for the sake of completeness. We then cannot make do with a left-hand-side value of a single word. Then B must e.g. be lowered by more than 1; this can be done by letting the top word of the left-hand-side value be a subroutine jump. A possible structure of the left-hand-side value is:  B - 2:             + a
B - 1: SUBCD(:q1)
B      : ----------

where address :q1 is the entry of the system, namely

q1:  S = MC[-2]
MS[0] = F
GOTOR(MC[0]) ⇒  S := a, B := B - 1
store
B := B - 1 and back.

Note. We draw attention to the fact that here a bit of cunning programming is indeed brought off. The DO-operation first lowers B; the final instruction turns out to be a stacking subroutine jump that overwrites itself with the link and raises B again by 1. The subroutine consists of three instructions, in which two stack references bring about the two necessary B-lowerings implicitly.

Duration: 10 memory contacts.

2.  An integer at address a
With integers too the case a < 32768 can be treated separately; the difference, however, is marked.

2.1. a < 32768
The structure of the left-hand-side value can be  B - 2 : M[a] = G
B - 1 : SUBCD(:q2)
B       : ----------

With q2: U, S = M[57], Z

Y, DO(MC[-2])
Y, GOTOR(MC[)])
F + AFR
F - AFR
U, S = M[57], Z
Y, DO(MC[-2])
Y, GOTOR(MC[0])
--------------

→ acceptable without more ado

→ acceptable after rounding
capacity overflow

Duration at least 9 memory contacts.

2.2. a ≥ 32768
The structure of the left-hand-side value can be:  B - 2 : + a
B - 1 : SUBCD(:q3)
B       : ----------

with q3:      S = MC[-2]
U, S = M[57], Z
Y, MS[0] = G
Y, GOTOR(MC[0]) → acceptable without more ado
.
.
.
. etc.
.
.

Duration at least 10 memory contacts.

3.  A real on a page
The left-hand-side value again occupies two words  B - 2 :  packing of physical address of the BZV and line number
B - 1 :  SUBCD(:q4)
B       :  ----------

The manner in which the subroutine that begins at :q4 has access to the packed data is, after the previous routines, now surely clear. We note that, when on grounds of timing it comes out sufficiently more favorably, we can here also let a left-hand side occupy three words, namely by storing the physical address of the BZV and the line number separately, each in its own word.

4. An integer on a page  B - 2 : packing of physical address of the BZV and line number
B - 1 : SUBCD(:q5)
B       : ----------

The routine that begins at q5 begins (see 2) by rounding, and proceeds further analogously to that of q4. Here too the data packed here can, if desired, be stored in two separate words.

Working-out of formal variables

We must now, for each type of actual parameter, establish what structure the formal locations will exhibit. The formal locations determine the actual parameter, which in general may have to be worked out in two ways, namely as left-hand side and as right-hand side. In the latter case we lay down as requirement that the value appears in the F-register, in the former case that the left-hand-side value, as described above, is added to the stack.

We start from four formal locations, described in order of ascending memory address as f[0], f[1], f[2], and f[3]. We agree that working-out as right-hand side in the object program of the body shall be coded as

DOS (f[0])

and working-out as left-hand side as

DOS (f[1])

Note 1. The formal locations will, as quantities in the stack, be addressed via the dynamic addressing.

Note 2. The supplying of a formal parameter as actual we wish to effect by "unseen" copying of the formal locations. We must therefore see to it that the structure of the formal locations is such that they are proof against displacement (namely copying elsewhere).

We now follow the same subdivision.

1. A real at a fixed physical address a

1.1. a < 32768 f[0] : F = M[a]
f[1] : SUBCD(:q6)
f[2] : M[a] = F
f[3] : for the time being unused.

Note. f[3] should have the right parity in connection with copying of the formal locations.

The subroutine that begins at q6 is just a bit complicated, because the link comes on top of the stack and it is precisely this place that must be filled. This implies "saving the link".

The system routine reads

q6 :

G = MS[1]

S = M[B-1]  save link

M[B-1] = G

GOTOR(M[60])      ⇒ back via S-register.

Duration for left-hand-side working-out: 10 memory contacts.

The right-hand-side working-out speaks for itself and lasts 4 memory contacts.

1.2. a ≥ 32768 For the sake of completeness we give the formal locations for a fixedly situated real above the 32 K:
f[0] SUBCD(:q7)
f[1] SUBCD(:q8)
f[2] + a
f[3] SUBCD(:q3)
Here q8 indicates the system routine:

q8 :

F = MS[1]

S = MC[-1]

MC[0] = F

GOTOR(M[60])

a routine that exhibits great resemblance to q6; the net raising of B by 1 is brought about by - 1 + 2. The duration for left-hand-side working-out is now 12 memory contacts.

The system routine q7 reads

q7 :

S = MS[2]

F = MS[0]

GOTOR(MC[-1])

and requires 10 memory contacts.

2. An integer at a fixed physical address a

2.1. a < 32768

f[0] G = M[a]
f[1] SUBCD(:q8)
f[2] M[a] = G
f[3] SUBCD(:q2)

Here the same system routine is used (namely q8) as in case 1.2. Working-out of a left-hand side costs 12 memory contacts, working-out of a right-hand side 3

2.2. a ≥ 32768
For the sake of completeness we give this case again
f[0] SUBCD(:q9)
f[1] SUBCD(:q8)
f[2] + a
f[3] SUBCD(:q3)

where q9 (analogous to q7) reads     q9 :

S = MS[2]
G = MS[0]
GOTOR(MC[-1])

Working-out of a right-hand side then lasts 9 memory contacts, that of a left-hand side as usual 12.

3. A real on a page

The actual parameter is then a subscripted element. We assume that this is given by an implicit IS that (roughly) localizes the element in terms of physical address of the BZV and line number. Presence analysis of the page is not yet performed by the implicit subroutine.

The implicit subroutine is characterized in the formal locations by an "invariant start address". We leave open whether herein the BZV of the page is given by its physical address, or rather by its addressing as a local of the outermost block (own page) or the library list. Physical address is probably the fastest.

The content of the formal locations is then

f[0]      SUBCD(q10 or q11)
f[1]      SUBCD(q12 or q13)
f[2]       invariant start address implicit subroutine
f[3]      context D.

Routines q10 and q12 are intended for the case in which the IS is simple, which among other things implies that the page of the body need not be profaned and there will by definition be sufficient anonymous space in the stack. In this case the machine link can be maintained as return address and only the prevailing D need be added to the return data. Otherwise (q11 and q13) the link must be translated back into an invariant return address and the page of the body must be profaned.

Under control of S these routines have access to the IS and the context D, which must be filled in to be able to execute the IS correctly. Furthermore the page where the IS begins must be requested sacredly. (If it is already present sacredly, this already means that HHT = 2 becomes!)

After completion of the IS the control returns into the system routine. Depending on left-hand or right-hand side, either only the packing of BZV-address and line number — delivered by the IS — is placed on the top of the stack and this covered with the constant "SUBCD(:q4)", or selection is performed — including presence check — and the requested real placed in F.

4.  An integer on a page The content of the formal locations is then

f[0]      SUBCD(q14 or q15)
f[1]      SUBCD(q16 or q17)
f[2]      invariant start address implicit subroutine
f[3]      context D.

These routines differ from the previous four only in that, on left-hand-side working-out, the packing is covered with "SUBCD(:q5)" and, on right-hand-side working-out, only the selected word is placed in G.

Note 1  As soon as we also wish to permit "small arrays" in the stack, the last eight subroutines will (can) inspect, after return from IS, whether the indicated element is one of a small array. In that case the left-hand-side value is as of type 1 or 2, the right-hand-side value is then trivial.

Note 2 These eight subroutines look suspiciously much alike, and it is therefore obvious to let them make use of common routines. One should note that all status information must then come onto the stack, because they can be nested and archived in parallel — i.e. can be used intermingled by different programs.

With this all actual parameters corresponding to a specification arithmetic have been treated insofar as they have a possible left-hand-side value. The remaining ones are the constant, the expression, and the arithmetic procedure SS (Self Supporting, i.e. a procedure identifier belonging to a procedure without parameters).

Upon entry into the procedure the formal locations will have to be tested for acceptability; here it is not tested whether an arithmetic actual indeed also has a left-hand-side value. Therefore we shall fill these formal locations (in particular f[1]) such that, if the body tries to work out their left-hand-side value, this still gives rise to protest.

5.  The real constant      f[0]
f[1] F = MS[2]
Alarm jump

f[2]    ⎱
f[3]    ⎰  value of the constant

Duration right-hand-side working-out: 4 memory contacts.

6. The integer constant (This is perhaps a bit too much honor!)

f[0]
f[1]
f[2]
f[3]  G = MS[3]
Alarm jump
invariant start address implicit subroutine
value integer constant

Duration right-hand-side working-out: 3 memory contacts.

7. The arithmetic expression    f[0]
f[1]
f[2]
f[3] SUBCD(q18 or q19)
Alarm jump
invariant start address implicit subroutine
context D.

The distinction between the system routines q18 or q19 is again whether the implicit subroutine is simple or not.

8. The single type procedure identifier that may occur as an expression The content of the formal location becomes, as soon as we find ourselves well and truly in the body

f[0]
f[1]
f[2]
f[3] SUBCD(:q20)
Alarm jump
invariant start address of the procedure
context D

The difficulty here is that the procedure can also be called explicitly as a procedure. Now we propose, at the call, to fill the formal locations of the parameters, then supply this number, and then jump to the (still "unseen") procedure, which begins by testing whether the number of supplied parameters is correct. The body of the parameterless procedure thus begins by testing whether it has indeed been supplied no parameters.

The function of the system routine q20 is to insert dynamically the bit of calling sequence comprising "zero parameters", before the procedure is activated according to all the rules of the art.

The formal-location filling with SUBCD(:q20) can, however, come about in only two ways

1st By passing on, as actual parameter, an arithmetic formal parameter — where the actual was a procedure identifier. Passing on is indeed copying of the formal locations.

2nd By supplying an arithmetic procedure identifier as actual parameter, where the formal turns out to be specified as arithmetic. The supplying of a procedure identifier as actual parameter is done by the translator:

1st

without awareness of the number of parameters that this procedure requires

2nd

without awareness of whether the corresponding formal is specified as procedure or as arithmetic.

If the formal is specified as procedure, then the body inspects the formal locations to see whether a procedure of the right type has been supplied. On use of this parameter specified as procedure, the call side will explicitly indicate the actual number of parameters (can = 0).

If the formal parameter is specified as arithmetic, then a procedure identifier of the right type is accepted as the corresponding actual, but the formal locations are modified by the filling-in of "SUBCD(:q20)" in f[0]. This process takes place regardless of the number of parameters that the supplied procedure would like to have. On dynamic use this is tested in the procedure; q20 namely takes care of explicit supply of "zero parameters". (Even then the Alarm jump need only be filled into the formal location: with a procedure that is specified as such, an alarm jump is indeed an open door!)

transcribed by Carl Ludwigson
revised Thu, 4 Nov 2010
