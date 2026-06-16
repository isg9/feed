---
title: "Over standaardroutines (EWD75)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD75.html
published: "1964-01-14T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD75.PDF
---

# Over standaardroutines

Voor een machine met een trommel is het, naar men zich hopenliji denken kan,
een belangrijk criterium, waarnaar men standaardroutines kan onderverdelen, of
zij permanent op een vaste plaats in de kernen aanwezig geacht kunnen worden of
niet.

Onder de permanent aanwezigen vallen in elk geval de zg. coordinatorroutines,
zoals de aanroepen van de P- en V-operaties. In de ALGOL-machine zullen ook de
systeemroutines hierbij horen, in het bijzonder die stukken programma, diete
maken hebben met de aan- of afwezigheid van informatie in de kernen. (Je kunt
uit de aard der zaak informatie "facultatief" op de kernen hebben: als het
programma, dat deze aanwezigheid onderzoekt, ook zelf afwezig kan zijn, dan
krijg je kennelijk rare dingen.) Het is duidelijk, dat wij de totale omvang van
deze routines vanwege hun permanente aanwezigheid scherp in het oog moeten
houden.

Het ALGOL-systeem zal de beschikking krijgen over een uitbreidbare
bibliotheek, nl. van standaardprocedures; ik neem aan, dat deze bibliotheek zich
permanent op de trommel zal bevinden. Bij de incorporering en de interne
formulering treedt direct een moeilijkheid op.

Men kan de bibliotheek op de trommel beschouwen als "moedercopie" en bij
vertaling van een bibliotheek gebruikend ALGOL-programma de gebruikte
bibliotheekprocedures in het objectprogramma opnemen. Deze opname wordt geen
blindelingse copiering; nomenclatuur (identificatie van geheugenbezettingen,
andere bibliotheekprocedures) moet dan vertaald worden. De copie bevat immers
ook de informatie hoe hij -en andere bibliotheekprogramma's- in het
ALGOL-programma zijn ingepast. Op deze wijze is de bibliotheek louter een
vertalerkwestie, waar het run time system nauwelijks iets mee te maken heeft: in
het objectprogramma hoeft niet meer te zien te zijn, dat sommige procedures niet
expliciet gedeclareerd zijn geweest.

Dit is evenwel niet de weg, die wij in wilden slaan. Ten eerste waren we-
terecht of niet- een beetje bang voor de complicatie van de vertaler; ten tweede
betekent dit, dat als twee ALGOL-programma's dezelfde standaardprocedure
gebruiken, dat deze procedure dan inmiddels drie maal in het geheugen staat en
mogelijk tweemaal op de kernen. En de vraag is, waarom zouden we dat doen?

We mikken dus nu op een oplossing, die de bibliotheekroutines gebruikt zoals
ze op de trommel staan. Wat we winnen is een simplificate van de vertaler en de
winst, dat we zuiniger met het geheugen omspringen. Mogelijk winnen we ook, dat
de tekst van de bibliotheekprocedures, die nu immers niet meer door de vertaler
vermalen worden, met wat grotere vrijheden opgesteld kan worden.

De prijs, die we betalen, is vooralsnog onbekend. Ik zie op het ogenblik twee
betalingen.

Doordat verschillende programma's dezelfde kernpagina nu parallel kunnen
willen gebruiken, wordt het concept van een "heilige kernpagina"
gecompliceerder. We kunnen niet meer met een enkel bit volstaan, maar hebben een
teller nodig, die bij heiliging met 1 verhoogd moet worden en bij profanatie met
1 verlaagd moet worden. Alleen als de teller = 0 is, is de pagina vogelvrij.

De tweede prijs is een kerngeheugenbezetting door een
"bibliotheeksinhoudsopgave", die bijhoudt, waar de bibliotheekpagina's zich
bevinden. De lengte van deze tabel is evenredig met de omvang van de
bibliotheek, of de bibliotheek nu gebruikt wordt of niet. Dit is strict genomen
natuurlijk tegen de regels, maar omdat deze tabel waarschijnlijk maar 1 woord
zal beslaan per trommelpagina van waarscijnlijk 512 woorden, dabhten we, dat we
ons dit konden permitteren. ALGOL-programma's identificeren
bibliotheekprocedures -en bibliotheekprocedures identificeren zichzelf en
zichzelf en elkaar-invariant in termen van deze tabel.

Als dit normale procedures zijn, dan krijgen ze bij activering hun
geheugenruimte toegewezen in de stapel van het activerende programma. Dit is
mogelijk en geeft geen aanleiding tot extra complicaties, bij gratie van het
feit, dat de activiteit van een dergelijke bibliotheekprocedure synchroon
verloopt met het aanroepende programma. Van de aanroep van een
bibliotheekprocedure hoeft, voorzover ik zie, de coordinator geen bijzondere
notitie te nemen.

Het bovenstaande is slechts ter inleiding, ter verhoging van de
contrastwerking met wat volgt. Wij zullen nl. de individuele ALGOL-programma's
ook de beschikking moeten kunnen geven over geprogrammeerde
standaardhandelingen, die asynchroon met het ALGOL-programma moeten kunnen
werken. Ik voorzie nu, dat het een drommels verschil zal maken, of een dergelijk
asynchroon proces willekeurig veelvoudig parallel geactiveerd moet kunnen worden
of niet. Als wij ons ervan konden overtuigen, dat dit niet het geval zal zijn,
dan zou ik van ernstige zorgen bevrij zijn.

Straks zal ik, ter illustratie, als intellectuele exercitie en mogelijk voor
later gebruik een voorbeeld proberen uit te werken, dat niet parallel
geactiveerd hoeft te worden. Eerst wil ik proberen aan te duiden, waar ik de
moeilijkheden verwacht, als aan deze voorwaarde niet voldaan is.

Als wij even aannemen, dat elk werkend ALGOL-programma het samenspel
impliceert tussen dit programma en een aan dit ALGOL-programma toegevoegd
simultaan werkend proces, een soort prive-secretaresse, dan betekent de
introductie van een ALGOL-programma de creatie van twee abstracte machines, te
weten het ALGOL-programma en de prive-secretaresse. Ik laat nu even in het
midden of de programmatekst van de prive-secretaresses in veelvoud of in
enkelvoud aanwezig zal zijn. In het ene geval moet ik een
multipliceringsmechanisme maken, in het andere geval een extra
activeringsmechanisme. Wat mij zorgen baart is, dat er tussen ALGOL-programma en
prive-secretaresse informatieverkeer zal moeten plaatsvinden, dat er seinpalen
zullen moeten zijn, die voor beiden toegankelijk zullen moeten zijn. De
introductie van een ALGOL-programma betekent dan ook niet "alleen maar" de
creatie van twee nieuwe abstracte machines, het betekent de creatie van twee op
bijzondere wijze op elkaar afgestemde abstracte machines. Ik kan bijvoorbeeld
niet overzien, of we ons zullen kunnen permitteren, om deze seinpalen
gedurendehun leven op vaste geheugenplaatsen onder te brengen. Zo nee, dan
bekruipt mij plotseling de angst, dat "verschuiven" van die informatie een hel
wordt, omdat meer dan 1 abstracte machine daar weet van moet hebben. Kortom,
"wat en waar" is de common ground waar de twee elkaar kunnen ontmoeten?

Opm. Voor de bibliotheekpracedures hebben we de "common ground" geschapen
door de tijdens run time permanent aanwezige "bibliotheekinhoudsopgave"; tijdens
vertalen komt een soort complement van deze tabel ter sprake, nl. als
"prevulling van de naamlijst": aan de standaardnamen zijn daar dan verwijzingen
toegevoegd naar de bibliotheekinhoudsopgave. Wijziging van de bibliotheek
impliceert in het algemeen een herindeling van het totale bibliotheekgeheugen en
is geen triviale opgave. Wij zijn er essentieel van uitgegaan, dat hergroepering
van de bibliotheek zal gebeuren tijdens executie, of tussen vertaald zijn en
uitvoering, van een of meer ALGOL-programma's. Ziehier een nieuw voordeel van
load and go!

Deze zorgen zijn allemaal aanmerkelijk minder, wanneer we een asynchrone
"manus voor allen" slechts in enkelvoud willen kunnen bespelen. Stel, dat een
ALGOL-programma mogelijkerwijze, maar niet noodzakelijkerwijze, deze manus wil
kunnen gebruiken. (Ik ga straks een en ander illustreren aan de printer; een
programma mag printen, maar hoeft dat natuurlijk niet!). Van deze manus maak je
dan, eens en vooral, een stamgast van je hotel voor abstracte machines. Als hij
klein is, zet je hem op vaste plaatsen op de kernen, anders laat je hem als
ieder ander programma via de paginaadministratie meedraaien. In elk geval kan je
je veroorloven om hem permanent een klein beetje levend geheugen permanent ter
beschikking te stellen voor het informatieverkeer.

(De opmerking is, dat als geen van de lopende ALGOL-programma's van de
printer gebruik wil maken, er dus waarschijnlijk niet zoveel te doen is en dat
je je deze kleine vaste bezetting dus wel kunt permitteren. En er hoeft er maar
eentje te willen printen en je hebt het er al uit.) Op dit moment is er een
ondubbelzinnige, invariante terminologie, waarin de onderscheiden
ALGOL-programma's de manus kunnen toespreken.

Omdat het woord "pagina" al gebruikt is voor de onderverdeling van kern - en
trommelgeheugen, noem ik een pagina, zoals die door de printer bedrukt kan
worden, een formulier, dwz. het stuk van perforatie tot perforatie.

Ik beschouw een de printer als een snel uitvoerorgaan, zo snel, dat

a) ALGOL-programma's hun resultaten zo ongeveer synchroon met de productie op
papier kunnen krijgen, en sterker

b) de printer zelfs meer dan 1 programma bedienen kan.

Het idee is, om de printer steeds formuliersgewijze aan een bepaald programma
te gunnen. Dit impliceert, dat bovenaan elk formulier ongevraagd geprint wordt
een identificatie van het producerende programma, een volgnummer en verder wat
nodig geacht wordt, datum etc.. De "expeditie" moet nu de formulieren maar
afscheuren en sorteren.

Dit impliceert, dat een ALGOL-programma pas de printer maag aanvragen,
wanneer het de gegevens voor een volledig formulier klaar heeft, Immers: zou een
ALGOL-programma de eerste helft van een formulier kunnen bedrukken, voordat de
resultaten voor de tweede helft geproduceerd zijn, dan zou dit programma de
printer heel lang kunnen blokkeren, omdat de productie van de volgende
resultaten wel eens heel lang op zich zou kunnen laten wachten. Elk
ALGOL-programma, dat van de printer gebruik maakt, heeft dus een prive-buffer
met de capaciteit van een formulier. Ald we aannemen, dat op een formulier 64
regels geprint worden, dan komen we met 32 woorden per regel op een buffer van
2048 woorden, dwz. met een paginagrootte van 512 woorden dus 4 pagina's.

Naast de privebuffer heeft elk ALGOL-programma als globale grootheden de
administratieve gegevens over de staat van vulling van de buffer,
formuliernummer, etc. Elk ALGOL-programma begint met deze gegevens passend te
initialiseren als standaard onderdeel van START.

Een ALGOL-programma, dat wil printen bouwt nu, helemaal locaal, een volledig
formulierbeeld op. Hierbij maakt het, neem ik aan, gebruik van in enkelvoud
aanwezige, conversie en opmaakroutines. Het is hier onbelangrijk of deze
permanent op de kernen staan, dan wel als bibliotheekprocedures van de trommel
gehaald kunnen moeten worden. Belangrijk is, dat deze operaties synchroon met
het werkende programma plaatsvinden. Ik neem aan, dat deze "conversie en
opmaakroutine" detecteert, wanneer aan een nieuw formulier begonnen wordt en op
grond daarvan de formulierkop invult en tevens detecteert, wanneer een formulier
vol is.

Zodra een formulier vol is, moet het programma met de buitenwereld gaan
communiceren, dwz. met de formulierprinter. Deze formulierprinter is de
bovengenoemde, geprogrammeerde "manus voor allen".

De synchronisatie met de formulierprinter loopt over twee seinpalen; dit zijn
twee vaste geheugenplaatsen, waarvan ALGOL-programma en formulierprinter beide
kennis moeten dragen. Verder is er de "toonbank", waarover het formulierbeeld
wordt aangeboden. Als we de informatie lijfelijk zouden transporteren, zou dit
een stuk van 2048 plaatsen zijn.

Als we een formulierbuffer beschouwen als een groot array van vier pagina's,
dan hoeft aanbod helemaal niet het lijfelijke transport van 2048 woorden te
impliceren. Immers: die vier pagina's zijn bereikbaar via vier descriptoren, de
zg. TPV's (Trommel Pagina Variabelen). Als we deze vier TPV's transporteren,
bereiken we daarmee, dat de vier gevulde pagina's, die eerst tot het
producerende ALGOL-programma behoorden, nu zijn overgeheveld naar de
formulierprinter.

De organisatie van de grote arrays zal de feitelijke reservering van Pagina's
niet doen bij de declaratie van het array -dan worden alleen de TPV's
geintroduceerd en geinitialiseerd- maar pas bij werkelijk gebruik van het eerste
element van de pagina. Na beeindiging van interesse -zoals bij blokverlating-
worden detrommel-pagina's weer aan de vrije hoop toegevoegd.

Een en ander heeft tot gevolg, dat wij zonder absurde onkosten de individuele
ALGOL-programma's en de formulierprinter grotere buffers zouden kunnen geven;
als wij dat niet doen, zullen wij dus iets van een motivering meeten geven.

Een reden om de individuele programme's buffers van meer dan 1 forniulier te
geven zou de wens kunnen zijn om een programma in staat te stellen de printer
voor meer dan 1 formulier aan zich te binden; wie vier formulieren achter
elkaar, dwz. ongestoord door overige programma's geprint wil hebben, mag de
printer pas aan zich binden, als die vier formulierbeelden geheel geprepareerd
zijn. Hiermee maakt men kunstmatig, dat de aanbiedende programma's in 'bursts'
van meer formulieren aanbieden en het ligt dan ook voor de hand dat de buffer
van de formulierprinter groter gekozen wordt, viz. groot genoeg om in enen op te
kunnen vangen, wat een programma in enen aan zou willen bieden. We zijn echter
niet van plan om een programma meer dan 1 formulier tegelijkertijd aan te laten
bieden, en dus krijgen de individuele programma's een prive-buffer van 1
formulier.

Aldus besloten rijst de vraag, hoe groot we de buffer van de formulierprinter
zullen kiezen. Minstens 1 formulier, nl. het formulier, dat gedrukt wordt. Naar
1 formulier is een bertje weinig: dit betekent immers, dat na voltooiing van dit
formulier de buffer gegarandeerd leeg is en dat voor de coordinator een morele
haast-situatie zou ontstaan als je de printer tussen de formulieren niet wilt
laten stokken. Die morele haastsituatie kan aanmerkelijk verlicht worden door de
formu1ierprinter met een toonbank van twee formulieren te laten werken: die
worden dan wiggel-waggel bedreven.

Met de beslissing, dat de formulierprinter over meer dan een
1-formuliersbuffer zal beschikken, is een wezenlijke uitbreiding gedaan: dit
betekent, dat aanbod oo de toonbank onder controle van een vulwijzer zal moeten
geschieden (waar we anders niet over hoefden te praten) en dat meer dan 1
programma tegelijkertijd een formulier zou kunnen willen aanbieden en dat we
zorgen moeten dat er dan geen ongewenste interferentie op treedt. Breiden we nu
de omvang van de formulierprinterbuffer non uit, dan verandert er aan de
structuur van de programma's niets meer, alleen geheugenindeling en de waarde
van een naar constantes. De vraag is, of we de formulierprinter een grotere
buffer willen geven, Ik geloof van niet.

De overweging is de volgende: als de formulierprinter een grote eigen buffer
heeft, die ongeveer leeg is en als nu alle programma's ongeveer tegelijkertijd
een formulier aanbieden, dan wordt deze buffer grotendeels gevuld en ligt
daarmee voor lang vast, welke programma's door de printer in successie geholpen
zullen worden. Dit volgens de regel "wie lang geleden het eerste kwam, zal later
het eerste malen". Heeft de formulierprinter maar een kleine buffer, dan is deze
ongeveer onmiddellijk vol en de overige programma's staan op een seinpaal te
wachten. Zodra er dan weer ruimte in de buffer van de formulierprinter komt, dan
mag de coordinator beslissen wie een pagina mag aanbieden. Door deze grotere
vrijheid van de coordinator zal het bij kleinere buffer makkelijker zijn om
prioriteit te geven aan bepaalde programma's.

We hebben nog een ander ding overwogen, maar we menen tot de conclusie
gekomen te zijn, dat je dat beslist niet doen moet; de coordinator heeft dan
namelijk nog minder te vertellen. Je kunt als volgt redeneren: ieder programma
heeft zelf (van de programmeur) enig idee van de omvang van zijn
informatieexplosies, die naar de printer moeten. Laat het daarom een locale
buffer introduceren ter grote van een dergelijke "burst" -afgerond op hele
formulieren uit de aard der zaak. Nu kan je het aanbieden van een pagina ook
anders spelen: inplaats van transport van de informatie -of van een descriptor,
wat in dit verband op hetzelfde neer komt- kan je ook de in een ALGOL programma
gevormde formulierinhoud via een kettingrijgerij aanhaken aan de ketting van hef
"nog te printen spul". Dit heeft de organisatorische complicatie, dat na
voltooiing van deze printhandeling dit heugelijk feit niet neutraal betekent "de
printer is weer vrij", maar dat dit doorgemeld moet worden naar het programma,
in welks locale buffer hierdoor weer een formulierplaats vrijgekomen is. Veel
erger is echter, dat wanneer een aantal programma's samen in de machine zit, die
met zijn allen outputbound zijn door de printer, zij geholpen worden naar
evenredigheid van hun eigen bufferomvang: het aanvragen van een grote locale
buffer is dan een manier om je latere mogelijke compagnons in de machine een hak
te zetten. Met deze conclusie zijn we erg blij, want om organisatorische rederen
hadden we er nl. al niet zo'n zin in.

Het volgende, dat besloten moet worden is in welke vorm de printer tussen de
formulieren achtergelaten wordt. De X8 heeft geen mogelijkheid om te testen,
wanneer de perforatie voorbijkomt. Ik neem aan, dat tussen formulieren de
printer achter gelaten wordt met het papier in de positie gereed voor de
bedrukking van de eerste regel. De formulierprinter mag van deze toestand
uitgaan maar moet na gebruik van de printer dus ook zorgvuldig deze toestand
achterlaten! En dit geldt nu niet alleen van de formulierprinter, maar het geldt
voor elk programma, dat "tussendoor" van de snelle regeldrukker gebruik wil
maken.

Op dezelfde manier moeten we beslissen wat we zullen doen met de IFT (+
Inflop) en AFT (+ Acflop) van de printer. Wij hebben ons hierbij door de
controlemogelijkheid laten leiden. (Terwijl ik dit verslag tik, merk ik, dat we
ook anders te werk hadden kunnen gaan; ik beschrijf nu eerst, wat we gedaan
hebben). We zijn er van uitgegaan, dat de formulierprinter gebruik zou maken van
de terugmeldingen, die met het ophogen van IFT gemeld worden; we hebben gesteld,
dat het printen van een formulier, voor zover het de formulierprinter aangaat,
beschouwd moet worden als een onscheidbaar geheel, dat in zijn geheel fout of in
zijn geheel goed gaat. Mogelijke reactie op een fout moet dus kunnen zijn; het
hele formulier opnieuw proberen te printen. Verder hebben we aangenomen, dat er
tussen opeenvolgende formulieren geen vorm van "overlap" zou zijn: de
formulierprinter begint pas aan de regels van het volgende formulier, als het
vorige formulier geheel geprint en gecontroleerd is. Dit impliceert dus, dat
tussen de formulieren het startmagazijn van de snelle regeldrukker even helemaal
leeg is.

Op grond hiervan hebben we gesteld, dat de nuchtere toestand van de
regeldrukker zijn zou AFT = IFT = 0; AFT is duidelijk, er hangen geen
startopdrachten meer; het idee van IFT = 0 kan uitgelegd worden als "er is niets
meer te melden".

Nadat dit besloten is, moet worden afgesproken, in wat voor staat de
formulierprinter de bijbehorende geheugenplaatsen, leegwijzer en startmagazijn
zal aantreffen. Het leek ons verstandig om daaromtrent geen enkele
veronderstelling te maken. Dit betekent, dat bij de aanvang van een formulier de
formulierprinter begint met een leegwijzer in te vullen (op iets, wat er
waarschijnlijk nog staat van het vorige formulier, maar dat doet er niet toe).
We nemen zelfs aan, dat de gegevens in het startmagazijn er niet meer zullen
staan, hoewel die, zie onder, bedoeld zijn om tijdens het formulierprinten niet
te veranderen.

Tenslotte moeten we beslissen, waar het regelbeeld zal staan, dat door de
regeldrukker geprint zal worden. Dit kost ons 32 woorden per regel en we moeten
totaal 4 regels kunnen opbergen. We hebben besloten hier 128 woorden vast voor
te reserveren en dus op het allerlaatst de informatie wel lijfelijk te
transporteren. De overwegingen, die hier toe geleid hebben, waren de volgende.
Als een pagina met regels nog op de kernen staat, staat hij "ergens", dwz. op
een vrij willekeurige kernpagina; als de pagina met regels inmiddels op de
trommel gedumpt is en we hem voor gaats halen met het standaard mechanisme -en
dat willen we natuurlijk- dan komt hij ergens. Maar informatie, die door de
printer asynchroon uit het geheugen geslurpt wordt, moet staan op een uiters
heilige plaats. Zouden wij de pagina, zoals hij in het kerngeheugen staat of
komt, heiligen, dan zou dat impliceren, dat we eventueel midden in het
kerngeheugen voor vrij lange tijd een geheiligde pagina zouden hebben. De
beperkingen, die daaruit voortvloeien, trokken ons niet aan. Vondaar deze
beslissing.

Nu geven we de voorlopige versie van de programma's; zij zijn geschreven met
inbegrip van de controle, de detectie, als het goed is gegaan. Het leek ons nu
wat vroeg om te gaan uitmelken, wat we moeten doen, als deze controle ALARM
geeft.

Relevant grootheden zijn:

integer FPN;
aantal formulieren in FP-buffer (Formulierprinter buffer); dit is een constante, waarschijnlijk dus gelijk 2

integer FPLW,
FPVW; dit zijn leegwijzer, resp. vulwijzer van de FP-buffer deze moeten door het tandenpoetsen aan elkaar gelijk gemaakt worden (door het tandenpoetsen van de formulierprinter); zij worden modulo FPN in het bedrijf opgehoogd.

Om der wille van de beschrijving voer ik in een nieuw type array, een zg
TPV array elementen waarvan fungeren als identifier van een array van 512
integers; ik zal deze elementen met dubbele indicering aanduiden.

Globaal voor formulierprinter en elk ALGOL programma hebben we de toonbank,
alias de formulierprinterbuffer

TPV array FPB[0:4*FPN-1]; deze factor
4 komt omdat we per formulier vier pagina's nodig hebben. Een ALGOL-programma
bouwt zijn formulier op in het locaal formulier

TPV array LF[0:3];

Voor de reductie modulo iets gebruiken we de integer procedure REM.

Synchronisatie tussen een ALGOL-programma en de formulierprinter wordt
gespeeld via twee globale seinpalen:
integer FPL,FPV; deze betekenen
het aantal lege, resp. volle formulieren in FPB. Aanvankelijk (dwz, bij
tandenpoetsen van de formulierprinter wordt FPL:= FPN; FPV:= 0 uitgevoerd).

Aanbod van een formulier aan de formulierprinter geschiedt door het volgende
stukje programma (hier hebben we nog een willekeurige locale integer, zeg i
nodig):

begin X8 doof; P(FPL);
for i:= 0,1,2,3 do
begin FPB[4*FPVW + i]:= LF[i];
LF[i]:= maagdelijk
end;
FPVW:= REM(FPVW + 1, FPN);
V(FPV); X8 horend
end

Commentaar: de X8 wordt hier doof gemaakt om te zorgen dat dit aanbod onder
controle van een unieke vulwijzer plaats vindt; noninterferentie van de ophoging
van FPVW is hiermee ook verzekerd. Dit had "zuiniger" onder controle van een
speciale uitslutingsseinpaal gekund, zuiniger in de zin, dat we dan minder
tijdsrestricties opleggen. Dit accevietje duurt echter gegarabdeerd zo kort, dat
even doof minder werk is. Het feit, dat ook de P- en V-operatie in diifheid
uitgevierd worden, is logisch niet essentieel.

Met "maagdelijk" is bedoeld de constante waarde van een TPV, waarvoor nog
geen pagina gereserveerd is. In dit geval wordt het dus een TPV, waarvoor geen
pagina meer gereserveerd is.

De formulierprinter als hieronder geprogrammeerd neemt van de printer alleen
maar aan, dat IFT en AFT beide = 0 zijn. Het startmagazijn en de leegwijzer van
de printer moeten dus passend geprepareerd worden. Ik gebruik hier de integer
procedure address ter beschrijving van "het adres van".

Naast de genoemde globule grootheden heeft de formulierprinter toegang tot de
integer "leegwijzer", de grootheid, onder controle waarvan de printer asynchroon
de startopdracht aanhaald. Verder is er een globale seinpaal "PV", te weten
printer vrij; deze seinpaal stelt ons in staat desgewenst de printer tussendoor
ook even op een andere manier te gebruiken.

De tekst van de formulierprinter wordt ongeveer als volgt:

formulierprinter:
begin integer leegwijzer i, j, vulwijzer;
integer array startmagazijn [0:7], regelmagazijn [0:127];
P(FPV, PV);
leegwijzer:= address(startmagazijn[7]);
for i:= 0,1,2,3 do startmagazijn[2*i]:= address(regelmagazijn[32*1]) cons;
i:= vulwijzer:= 0;
nieuwe regel:
if 3 < i then begin P(IFT printer);
if startmagazijn[2*vulwijzer + 1] ≠ correct then goto ALARM end;
if i < 63 then
begin for j:= 0 step 1 until 31 do
regelmagazijn[32* vulwijzer + j]:=
FPN[4*FPLW + entier((i+0.5V)/16)][32*REM(i,16) + j];
V(AFT printer) ;
vulwijzer:= REM(vulwijzer + 1, 4)
end;
if i < 68 then goto nieuwe regel;
for i:= 0,1,2,3 do vrijgave pagina(FPB[4*FPLW + i]);
FPLW:= REM(FPLW + 1, FPN);
V(FPL, PV); goto formulierprinter
end

Opm. het begint met initialiseringen, als het startmagazijn en het
regelmagazijn vast gekozen worden, dan zijn dit altijd dezelfde constanten, die
her in evuld moeten worden. Er is geen bezwaar tegen, dat dit van formulier tot
formulier zou weizigen. De test "3 < i" is om de eerste vier keer de controls
te onderdrukken, de test "i < 63" om alleen maar feitelijk aan te bieden, als
er nog iets aan te bieden valt. De procedure "vrijgave pagina" krijgt een tpv
als argument mee; het resultaat is dat de trommelpagina, die via deze TPV
toegankelijk was, nu definitief van de voorraad vrijen wordt toegevoegd. Zou je
zoiets als dit vereeten, dat zou de trommel dichtgroeien met oude
paginabeelden.

De conventie dat de formulierprinter de printer met IFT = 0 aantreft. wat we
in verband met de controle gedaan hebben. lijkt me bij nader inzien questieus.
Nu dwing je min of meer elk ander gebruik nodeloos in dit keurslijf. Beter lijkt
het me om als neutrale toestand AFT = 0 en IFT = 4 (de omvang van het
startmagazijn) te kiezen; dit is niet in tegenspraak, dat de formulierprinter
pas na volledige controle overgaat op het volgende formulier. De
formulierprinter kan aan het einde, na de controle 4 maal V(IFT printer) doen!
Wel ongebruikelijk, maar moeten we er niet aaan wennen?

transcribed by Carl Ludwigson

revised Tue, 2 Oct 2007

---

## English translation

### On standard routines

For a machine with a drum it is, as one may hopefully imagine, an important
criterion by which one can subdivide standard routines, whether they may be
deemed permanently present in a fixed location in the cores or not.

Among the permanently present ones fall in any case the so-called coordinator
routines, such as the invocations of the P- and V-operations. In the ALGOL
machine the system routines will also belong here, in particular those pieces of
program that have to do with the presence or absence of information in the cores.
(By the very nature of the matter you can have information "optionally" in the
cores: if the program that examines this presence can itself also be absent,
then you evidently get strange things.) It is clear that, on account of their
permanent presence, we must keep a sharp eye on the total size of these routines.

The ALGOL system will be provided with an extensible library, namely of
standard procedures; I assume that this library will reside permanently on the
drum. With the incorporation and the internal formulation a difficulty arises at
once.

One can regard the library on the drum as a "master copy" and, when
translating an ALGOL program that uses a library, include the library procedures
used in the object program. This inclusion does not become a blind copying;
nomenclature (identification of memory allocations, other library procedures)
must then be translated. After all, the copy also contains the information about
how it -and other library programs- have been fitted into the ALGOL program. In
this way the library is merely a translator matter, with which the run time
system has hardly anything to do: in the object program it need no longer be
visible that some procedures were not explicitly declared.

This, however, is not the road we wanted to take. In the first place we were
-rightly or not- a little afraid of the complication of the translator; in the
second place this means that if two ALGOL programs use the same standard
procedure, that procedure then stands by now three times in the store and
possibly twice in the cores. And the question is, why would we do that?

So we are now aiming at a solution that uses the library routines as they
stand on the drum. What we gain is a simplification of the translator and the
gain that we deal more economically with the store. We possibly also gain that
the text of the library procedures, which after all are now no longer ground up
by the translator, can be set up with somewhat greater freedoms.

The price we pay is as yet unknown. At the moment I see two payments.

Because various programs can now want to use the same core page in parallel,
the concept of a "sacred core page" becomes more complicated. We can no longer
make do with a single bit, but need a counter that must be increased by 1 upon
sanctification and decreased by 1 upon profanation. Only when the counter = 0 is
the page fair game.

The second price is a core-store occupancy by a "library table of contents"
that keeps track of where the library pages are located. The length of this
table is proportional to the size of the library, whether the library is now
used or not. This is, strictly speaking, of course against the rules, but because
this table will probably occupy only 1 word per drum page of probably 512 words,
we thought that we could permit ourselves this. ALGOL programs identify library
procedures -and library procedures identify themselves and themselves and each
other- invariantly in terms of this table.

If these are normal procedures, then upon activation they are allocated their
memory space in the stack of the activating program. This is possible and gives
no occasion for extra complications, by grace of the fact that the activity of
such a library procedure runs synchronously with the calling program. Of the
invocation of a library procedure the coordinator need, as far as I can see, take
no special note.

The above is merely by way of introduction, to heighten the contrast effect
with what follows. For we shall also have to be able to provide the individual
ALGOL programs with programmed standard operations that must be able to work
asynchronously with the ALGOL program. I now foresee that it will make a deuce of
a difference whether such an asynchronous process must be able to be activated
arbitrarily multiply in parallel or not. If we could convince ourselves that this
will not be the case, then I would be freed of serious worries.

Presently I shall, by way of illustration, as an intellectual exercise and
possibly for later use, try to work out an example that need not be activated in
parallel. First I want to try to indicate where I expect the difficulties if this
condition is not satisfied.

If we assume for a moment that every running ALGOL program implies the
interplay between this program and a simultaneously working process added to this
ALGOL program, a sort of private secretary, then the introduction of an ALGOL
program means the creation of two abstract machines, namely the ALGOL program and
the private secretary. I leave open for the moment whether the program text of
the private secretaries will be present in multiple or in single copy. In the one
case I must make a multiplication mechanism, in the other case an extra
activation mechanism. What worries me is that between the ALGOL program and the
private secretary information traffic will have to take place, that there will
have to be semaphores that will have to be accessible to both. The introduction
of an ALGOL program therefore does not mean "merely" the creation of two new
abstract machines, it means the creation of two abstract machines attuned to each
other in a special way. I cannot, for example, foresee whether we shall be able
to permit ourselves to house these semaphores in fixed memory locations during
their life. If not, then the fear suddenly creeps over me that "shifting" of that
information becomes a hell, because more than 1 abstract machine must have
knowledge of it. In short, "what and where" is the common ground where the two
can meet each other?

Note. For the library procedures we have created the "common ground" by means
of the "library table of contents" that is permanently present during run time;
during translating a sort of complement of this table comes up, namely as the
"pre-filling of the name list": there the standard names then have references
added to them, pointing to the library table of contents. Modification of the
library in general implies a re-layout of the total library store and is no
trivial task. We have essentially proceeded from the assumption that regrouping
of the library will happen during execution, or between being translated and
execution, of one or more ALGOL programs. Behold a new advantage of load and go!

These worries are all considerably less when we want to be able to play an
asynchronous "hand for all" only in single copy. Suppose that an ALGOL program
possibly, but not necessarily, wants to be able to use this hand. (I am going to
illustrate this and that presently with the printer; a program may print, but of
course need not do so!). Of this hand you then make, once and for all, a regular
of your hotel for abstract machines. If it is small, you put it at fixed
locations in the cores; otherwise you let it run along like every other program
via the page administration. In any case you can afford to put permanently a
little bit of live memory permanently at its disposal for the information traffic.

(The remark is that if none of the running ALGOL programs wants to make use of
the printer, then there is probably not so much to do and you can therefore
permit yourself this small fixed occupancy. And only one need want to print and
you have it back out.) At this moment there is an unambiguous, invariant
terminology in which the distinct ALGOL programs can address the hand.

Because the word "page" has already been used for the subdivision of core and
drum store, I call a page, as it can be printed by the printer, a form, that is,
the piece from perforation to perforation.

I regard a the printer as a fast output organ, so fast that

a) ALGOL programs can get their results roughly synchronously with the
production on paper, and, more strongly,

b) the printer can even serve more than 1 program.

The idea is to grant the printer to a particular program always form by form.
This implies that at the top of each form there is printed unrequested an
identification of the producing program, a sequence number and further whatever
is deemed necessary, date etc.. The "dispatch" must now simply tear off and sort
the forms.

This implies that an ALGOL program may request the printer only when it has
the data for a complete form ready. After all: were an ALGOL program able to
print the first half of a form before the results for the second half are
produced, then this program could block the printer for a very long time, because
the production of the next results might well keep one waiting a very long time.
Every ALGOL program that makes use of the printer therefore has a private buffer
with the capacity of a form. If we assume that 64 lines are printed on a form,
then with 32 words per line we arrive at a buffer of 2048 words, that is, with a
page size of 512 words therefore 4 pages.

Besides the private buffer every ALGOL program has, as global quantities, the
administrative data about the state of filling of the buffer, form number, etc.
Every ALGOL program begins by initializing these data suitably as a standard part
of START.

An ALGOL program that wants to print now builds up, entirely locally, a
complete form image. In doing so it makes use, I assume, of singly-present
conversion and layout routines. It is unimportant here whether these stand
permanently in the cores, or whether they may have to be fetched from the drum as
library procedures. What is important is that these operations take place
synchronously with the working program. I assume that this "conversion and layout
routine" detects when a new form is begun and on that basis fills in the form
head, and also detects when a form is full.

As soon as a form is full, the program must start communicating with the
outside world, that is, with the form printer. This form printer is the
above-mentioned, programmed "hand for all".

The synchronization with the form printer runs over two semaphores; these are
two fixed memory locations of which both the ALGOL program and the form printer
must have knowledge. Further there is the "counter", over which the form image is
offered. If we were to transport the information bodily, this would be a piece of
2048 locations.

If we regard a form buffer as a large array of four pages, then offering need
not at all imply the bodily transport of 2048 words. After all: those four pages
are reachable via four descriptors, the so-called TPVs (Drum Page Variables). If
we transport these four TPVs, we thereby achieve that the four filled pages,
which first belonged to the producing ALGOL program, are now transferred to the
form printer.

The organization of the large arrays will not do the actual reservation of
pages at the declaration of the array -then only the TPVs are introduced and
initialized- but only upon actual use of the first element of the page. After
termination of interest -as upon block exit- the drum pages are added back to the
free heap.

This and that has the consequence that we could, without absurd expense, give
the individual ALGOL programs and the form printer larger buffers; if we do not
do so, then we shall therefore have to give something of a motivation.

A reason to give the individual programs buffers of more than 1 form could be
the wish to enable a program to bind the printer to itself for more than 1 form;
whoever wants four forms printed in a row, that is, undisturbed by other
programs, may bind the printer to itself only when those four form images are
entirely prepared. By this one artificially makes the offering programs offer in
'bursts' of several forms, and it is then also obvious that the buffer of the form
printer be chosen larger, viz. large enough to be able to catch in one go what a
program would want to offer in one go. We do not, however, intend to let a program
offer more than 1 form at a time, and so the individual programs get a private
buffer of 1 form.

This having been decided, the question arises how large we shall choose the
buffer of the form printer. At least 1 form, namely the form that is being
printed. Towards 1 form is a little too few: after all, this means that after
completion of this form the buffer is guaranteed empty and that for the
coordinator a moral hurry situation would arise if you do not want to let the
printer stall between the forms. That moral hurry situation can be considerably
relieved by letting the form printer work with a counter of two forms: those are
then operated wiggle-waggle.

With the decision that the form printer will have at its disposal more than a
1-form buffer, an essential extension has been made: this means that offering
onto the counter will have to take place under the control of a fill pointer
(about which we otherwise would not have had to talk) and that more than 1 program
could at the same time want to offer a form and that we must take care that then
no undesired interference occurs. If we now extend the size of the form printer
buffer, then nothing further changes in the structure of the programs, only the
memory layout and the value of a constant. The question is whether we want to give
the form printer a larger buffer. I think not.

The consideration is as follows: if the form printer has a large buffer of its
own that is roughly empty, and if now all programs offer a form at roughly the
same time, then this buffer becomes largely filled and thereby it is fixed for a
long time which programs will be helped by the printer in succession. This
according to the rule "whoever came first long ago, shall grind first later". If
the form printer has only a small buffer, then this is roughly immediately full
and the other programs stand waiting on a semaphore. As soon as there is then
again room in the buffer of the form printer, the coordinator may decide who may
offer a page. By this greater freedom of the coordinator it will, with a smaller
buffer, be easier to give priority to certain programs.

We have considered yet another thing, but we believe we have come to the
conclusion that you must definitely not do that; for then the coordinator has even
less to say. You can reason as follows: every program itself has (from the
programmer) some idea of the size of its information explosions that must go to
the printer. Let it therefore introduce a local buffer of the size of such a
"burst" -rounded to whole forms, by the very nature of the matter. Now you can
also play the offering of a page differently: instead of transport of the
information -or of a descriptor, which in this connection amounts to the same
thing- you can also hook the form content formed in an ALGOL program, via a
chain-stringing, onto the chain of the "stuff still to be printed". This has the
organizational complication that, after completion of this print operation, this
joyful fact does not neutrally mean "the printer is free again", but that this
must be reported through to the program in whose local buffer thereby a form slot
has become free again. Much worse, however, is that when a number of programs sit
together in the machine, which are all together output-bound by the printer, they
are helped in proportion to their own buffer size: requesting a large local buffer
is then a way to do your later possible partners in the machine a bad turn. With
this conclusion we are very glad, for on organizational grounds we already did not
much fancy it.

The next thing that must be decided is in what form the printer is left between
the forms. The X8 has no possibility to test when the perforation comes by. I
assume that between forms the printer is left with the paper in the position ready
for the printing of the first line. The form printer may proceed from this state
but must therefore, after use of the printer, also carefully leave behind this
state! And this now holds not only of the form printer, but it holds for every
program that wants to make use of the fast line printer "in between".

In the same way we must decide what we shall do with the IFT (+ Inflop) and AFT
(+ Acflop) of the printer. We have let ourselves be guided here by the checking
possibility. (While I am typing this report, I notice that we could also have gone
about it differently; I now first describe what we have done). We have proceeded
from the assumption that the form printer would make use of the feedbacks that are
reported with the incrementing of IFT; we have laid down that the printing of a
form, as far as it concerns the form printer, must be regarded as an inseparable
whole that goes wholly wrong or wholly right. A possible reaction to an error must
therefore be able to be: trying to print the whole form again. Further we have
assumed that between successive forms there would be no form of "overlap": the
form printer begins on the lines of the next form only when the previous form is
entirely printed and checked. This therefore implies that between the forms the
start magazine of the fast line printer is for a moment completely empty.

On this basis we have laid down that the sober state of the line printer would
be AFT = IFT = 0; AFT is clear, no start commands are pending any more; the idea
of IFT = 0 can be explained as "there is nothing more to report".

After this has been decided, it must be agreed in what state the form printer
will find the associated memory locations, empty pointer and start magazine. It
seemed to us sensible to make no assumption whatsoever about this. This means that
at the beginning of a form the form printer begins by filling in an empty pointer
(over something that probably still stands there from the previous form, but that
does not matter). We even assume that the data in the start magazine will no
longer stand there, although those, see below, are intended not to change during
the form printing.

Finally we must decide where the line image will stand that is to be printed by
the line printer. This costs us 32 words per line and we must be able to store 4
lines in total. We have decided to reserve here fixedly 128 words for this and
thus, at the very last, to transport the information bodily after all. The
considerations that led to this were the following. If a page with lines still
stands in the cores, it stands "somewhere", that is, in a fairly arbitrary core
page; if the page with lines has meanwhile been dumped to the drum and we fetch it
back for use with the standard mechanism -and that we want of course- then it
comes somewhere. But information that is asynchronously slurped out of the store by
the printer must stand in an utterly sacred location. Were we to sanctify the page
as it stands or comes to stand in the core store, then that would imply that we
would possibly have a sanctified page in the middle of the core store for a fairly
long time. The restrictions that flow from that did not appeal to us. Hence this
decision.

Now we give the provisional version of the programs; they are written
including the checking, the detection of whether it has gone well. It seemed to us
a bit early now to start milking out what we must do if this check gives ALARM.

Relevant quantities are:

integer FPN;
number of forms in the FP-buffer (Form printer buffer); this is a constant,
probably therefore equal to 2

integer FPLW,
FPVW; these are the empty pointer, resp. fill pointer of the FP-buffer; these must
be made equal to each other by the tooth-brushing (by the tooth-brushing of the
form printer); they are incremented modulo FPN in operation.

For the sake of the description I introduce a new type of array, a so-called
TPV array, elements of which function as identifier of an array of 512 integers; I
shall denote these elements with double indexing.

Global for the form printer and every ALGOL program we have the counter, alias
the form printer buffer

TPV array FPB[0:4*FPN-1]; this factor
4 comes about because per form we need four pages. An ALGOL program builds up its
form in the local form

TPV array LF[0:3];

For the reduction modulo something we use the integer procedure REM.

Synchronization between an ALGOL program and the form printer is played via two
global semaphores:
integer FPL,FPV; these mean
the number of empty, resp. full forms in FPB. Initially (that is, at the
tooth-brushing of the form printer, FPL:= FPN; FPV:= 0 is executed).

Offering of a form to the form printer takes place by the following little
piece of program (here we still need an arbitrary local integer, say i):

begin X8 doof; P(FPL);
for i:= 0,1,2,3 do
begin FPB[4*FPVW + i]:= LF[i];
LF[i]:= maagdelijk
end;
FPVW:= REM(FPVW + 1, FPN);
V(FPV); X8 horend
end

Commentary: the X8 is here made deaf to ensure that this offering takes place
under control of a unique fill pointer; non-interference of the incrementing of
FPVW is hereby also assured. This could have been done "more economically" under
control of a special exclusion semaphore, more economically in the sense that we
then impose fewer time restrictions. This little activity, however, lasts
guaranteed so short that being deaf for a moment is less work. The fact that the
P- and V-operation are also carried out in deafness is logically not essential.

By "maagdelijk" (virginal) is meant the constant value of a TPV for which no
page has yet been reserved. In this case it thus becomes a TPV for which no page
is reserved any more.

The form printer as programmed below assumes of the printer only that IFT and
AFT are both = 0. The start magazine and the empty pointer of the printer must
therefore be suitably prepared. I use here the integer procedure address to
describe "the address of".

Besides the global quantities mentioned, the form printer has access to the
integer "empty pointer", the quantity under control of which the printer
asynchronously hooks up the start command. Further there is a global semaphore
"PV", namely printer free; this semaphore enables us, if desired, to use the
printer in between also for a moment in another way.

The text of the form printer becomes roughly as follows:

formulierprinter:
begin integer leegwijzer i, j, vulwijzer;
integer array startmagazijn [0:7], regelmagazijn [0:127];
P(FPV, PV);
leegwijzer:= address(startmagazijn[7]);
for i:= 0,1,2,3 do startmagazijn[2*i]:= address(regelmagazijn[32*1]) cons;
i:= vulwijzer:= 0;
nieuwe regel:
if 3 < i then begin P(IFT printer);
if startmagazijn[2*vulwijzer + 1] ≠ correct then goto ALARM end;
if i < 63 then
begin for j:= 0 step 1 until 31 do
regelmagazijn[32* vulwijzer + j]:=
FPN[4*FPLW + entier((i+0.5V)/16)][32*REM(i,16) + j];
V(AFT printer) ;
vulwijzer:= REM(vulwijzer + 1, 4)
end;
if i < 68 then goto nieuwe regel;
for i:= 0,1,2,3 do vrijgave pagina(FPB[4*FPLW + i]);
FPLW:= REM(FPLW + 1, FPN);
V(FPL, PV); goto formulierprinter
end

Note. it begins with initializations; if the start magazine and the line
magazine are fixedly chosen, then these are always the same constants that have to
be filled in here. There is no objection to this changing from form to form. The
test "3 < i" is to suppress the checking the first four times, the test "i < 63"
is to actually offer only when there is still something to offer. The procedure
"vrijgave pagina" (page release) is passed a tpv as argument; the result is that
the drum page that was accessible via this TPV is now definitively added back to
the stock of free ones. Were you to forget something like this, the drum would
silt up with old page images.

The convention that the form printer finds the printer with IFT = 0, which we
have done in connection with the checking, seems to me on closer reflection
questionable. Now you more or less force every other use needlessly into this
straitjacket. It seems better to me to choose as neutral state AFT = 0 and IFT = 4
(the size of the start magazine); this is not in contradiction with the fact that
the form printer proceeds to the next form only after complete checking. The form
printer can, at the end, after the checking, do V(IFT printer) 4 times! Unusual,
to be sure, but should we not get used to it?

transcribed by Carl Ludwigson

revised Tue, 2 Oct 2007
