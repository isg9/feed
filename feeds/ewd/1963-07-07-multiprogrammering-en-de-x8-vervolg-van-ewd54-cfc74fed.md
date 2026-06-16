---
title: "Multiprogrammering en de X8 (Vervolg van EWD54) (EWD57)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD57.html
published: "1963-07-07T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD57.PDF
---

# Multiprogrammering en de X8 (Vervolg van EWD54)

EWD 57

MULTIPROGAMMERING EN DE X8

DIRECTE TOEGANG TOT DELEN EN SECTIES  -  DIRECT ACCESS TO INSTALMENTS AND SECTIONS

DEEL 1: EWD51

0
1
1.1
1.2
1.3
2
2.1
2.2
2.3

DEEL 2: EWD54

2.4
3
3.1
3.2

DEEL 3: EWD57

3.3
4
4.0
4.1
4.2
4.3
4.4

DUTCH-ENGLISH DICTIONARY OF PRINCIPAL KEYWORDS

Seinpaal
Semaphore

Verhogen (V)
To raise, To
increase

Prolagen (P),

a neologism
coming from
To try and lower,

Probeer te verlagen
To try and decrease

Ingreep
Interrupt,
Interruption

Kernen
Core storage,

Main storage

Trommelgeheugen
Drum storage

Multiprogrammering
en de X8, aflevering 3.

(Vervolg van EWD51 en EWD54)

Juni 1963

X8 Nr. 19.

3.3.
De functie van de luisterbits. (7 juni 1963)

Een ingreep (zie 2.3., EWD51 - 7) vindt slechts plaats als

(a) de machine horend is

(b) het collatieresultaat van ingreepwoord(en) en (overeenkomstige) luisterwoord(en) ≠ 0 is.

Door een luisterbit = 0 te maken, kan men dus bereiken, dat de bijbehorende seinpaal, als de laatst allang positief is en de bijbehorende ingreepflipflop allang = 1 gezet is, tot nader order toch geen ingreep teweegbrengt.

Men is kennelijk niet geinteresseerd in het positief worden van een seinpaal, als er niets op staat te wachten. Speciaal bij een hardware seinpaal, waarvan het positief zijn in principe een ingreep teweegbrengt, is het begrotelijk, als deze ingreep plaats vindt en de coordinator niets anders doen kan den gedesinteresseerd van dit feit nota nemen, gedesinteresseerd, omdat aan het hele mogelijkheidspatroon niets verandert. Een ingreep kost nl. wel tijd (en we zullen aanhoudend alert moeten
blijven om deze tijd niet te veel te laten toenemen).

Het idee is nu, om de coordinator ervoor te laten zorgen, dat de luisterbit van een hardware seinpaal aangeeft, of er op deze seinpaal iets staat te wachten of niet: als de wachtketen, die aan de hardware seinpaal hangt (zie 3.2., EWD54 - 8 e.v.)., leeg is, zal de bijbehorende luisterbit = 0 zijn, anders = 1. Het is de taak van de tandenpoetserij om initieel
hiervoor te zorgen, het is de taak van de coordinator om dit steeds bij te houden.

Het gebruik van de luisterbit is hiermee duidelijk. Hoe effectief het is, kan ik op geen stukken na bekijken. Algemeen geldt, dat het effectiever zal zijn

(a) als abstracte machines zo mogelijk liever aan non-hardware seinpalen gehangen worden.

(b) als de cumulatie van ingrepen hoger kan zijn. Dit laatste houdt verband met de omvang van het startmagazijn.

We moeten hier in elk geval naar kijken en wel om twee redenen: ten eerste geeft dit "letten op" waarschijnlijk nauwelijks aanleiding tot kostenverhoging -in hardware, tijd of aantal opdrachten van de coordinator-; ten tweede moeten we ons bewust zijn, dat het uitsparen van onnodige ingrepen hier zou werken, als de computer niet op de transputapparaten zou hoeven te wachten, dwz. als de hele fabriek "processor limited" zou zijn, dwz. juist de omstandigheid, dat het
alle zin heeft, om zuinig met computertijd om te springen!

Een laatste opmerking is, dat de tijdsspendering per ingreep een betrekkelijk vast bedrag is: dit gaat procentueel des te zwaarder tellen, naarmate je randapparatuur frequenter V-operaties uitvoert. (We moeten niet vervallen in de fout, die ik bij de X1 gemaakt heb, waar het ingreepprogramma voor de bandlezer afgestamd was op de 150 char./sec van de Ferranti-bandlezer -onzaliger nagedachtenis!- met het droeve resultaat, dat het de EL1000 vanwege zijn nu niet meer negligeabele reactietijd
niet meer bij kon sloffen!)

4. Automatische transporten tussen langzaam en snel geheugen.

4.0. Inleiding.

Een ander niet onbelangrijk probleem, nl. dat van de toevoeging of afvoering van abstracte machines laat ik voorlopig met opzet liggen, en wel om de volgende redenen. Binnen afzienbare tijd hoop ik de documentatie onder ogen te krijgen van hoe andere systemenm waarin dit probleem in een zekere mate van algemeenheid is opgelost. Daarin is ongetwijfeld een oplossing gevonden voor de toekenning van randapparatuur aan programma's; behalve concrete objecten zullen wij ook abstracte objecten
-nieuw in te voeren seinpalen, bv.- moeten toekennen. Door de grotere flexibiliteit, waar we op mikken, zal het voor ons wat minder eenvoudig liggen, maar het zou onverstanding zijn om te schromen, van andermans ervaring te profiteren. De tweede reden is, dat de administratieve plichten bij toevoeging van nieuwe abstracte machines wel eens medebepaald kon worden door de structuur van deze machines. Deze laatste kon wel eens drastisch beinvloed worden door de manier, waarop we de automatische transporten tussen
langzaam en snel geheugen denken te regelen. Dit laatste is dus fundamenteler -en ook moeilijker; vandaar dat ik dat nu eerst te lijf wil. Ter inleiding een historisch overzicht van drie relevante stappen.

4.1. De ARMAC.

Ik herinner me nog heel goed, hoe we wild enthousiast waren, toen we het ontwerp van de ARMAC klaar hadden. In zijn laatste dagen, met de X1 aan de andere kant van de gang, hebben we wel eens medelijdend de schouders over het oude beestje opgehaald. In retrospectie was dit enthousiasme uit 1955 toch niet zo ongerechtvaardigd!

In 't kort kwam de organisatie van de ARMAC, die in wezen een trommelmachine was, op het volgende neer. De trommel was ingedeeld in sporen van 32 woorden per spoor. Opdrachtselectie direct van de trommel was echter onmogelijk: de machine was uitgerust met een buffer op kernen met de capaciteit van 1 spoor en opdrachtselectie bestond uit

(1) verificatie, dat het goede spoor in de buffer gecopieerd stond

(2) selectie van de overeenkomstige plaats in de buffer.

Als aan de eerste voorwaarde niet voldaan was, werd het rekenproces zonder pardon 1 trommelomwenteling opgehouden, waarin de buffer met het nieuwe spoor gevuld werd. Om dat alles mogelijk te maken, waren 7 additionele flipflops ingevoerd (de trommel bevatte 128 sporen, de adreslengte was 7 + 5 = 12 bits) waarin het nummer van het trommelspoor onthouden werd, dat in de buffer gecopieerd stond. Bij elke opdrachtselectie werd getest op gelijkheid van deze 7 bits en de 7 meest significante bits van de opdrachtteller.

Als additionele feature werd bij elke getalselectie ditzelfde 7-tal vergeleken met de 7 meest significante adresbits in het opdrachtregister: in geval van gelijkheid werd bij lezen niet gewacht, tot de aangewezen trommelplaats onder de koppen doordraaide, maar werd meteen uit de overeenkomstige plaats uit de buffer geselecteerd; bij schrijven werd dan wel op de trommel geschreven, maar tevens werd de kopie in de buffer bijgehouden.

Deze additionele hardware heeft zijn geld meer dan opgebracht. Let wel, dat de comparatie met de 7 bits, die aangaven, welk spoor in de buffer stond, in elke gewone opdracht twee keer werd uitgevoerd: eerst in de opdrachtcyclus, daarna in de getalcyclus. In een conventionele Von Neumann-machine kon je ook moeilijk anders.

Op de keper beschouwd waren het merendeel van deze comparaties overbodig: logisch was het toelaatbaar -en dat was toen in zekere zin geavanceerd- om de ARMAC als boven beschreven non-track-bewust te bedrijven. In de praktijk liet je dit wel uit je hoofd: je besteedde dagen om net zo lang aan de subroutine te fiegelen, totdat je hem in 32 woorden geperst had! En alle hoofdprogramma's hadden op den duur ook alle constanten in het spoor, waar ze gebruikt werden; je moest wel.

Gaan we er nu van uit, dat de machine track-bewust geprogrammeerd wordt, dan komen we tot de verrassende ontdekking, dat alle comparaties met de 7 bits weggepraat kunnen worden en daarmee de 7 flipflops zelf.

Het enige dat je nodig hebt zijn speciale sprong- lees- en schrijfopdrachten van en naar plaats zoveel van de buffer. De gewone sprongopdrachten kan je dan altijd hernieuwde buffervulling laten impliceren. De enige vraag is dan nog, hoe je van het ene spoor naar het andere overloopt. Je had kunnen detecteren op overloop naar de 7de bit van de opdrachtteller bij dezelfde ophoging; een alternatieve oplossing is om expliciet aan het einde van een spoor naar het begin van het volgende spoor
te springen -iets dat we in de praktijk haast altijd deden, omdat je toch over een paar constanten heen moest springen.

Op deze manier bekeken is de automatische 7-bits comparatie dus eigenlijk, hoewel mooi, volslagen overbodig: de programmeur kon weten hoe de uitslag zou zijn (en hij deed er goed aan dit te weten).

4.2. ATLAS.

Een volledig beeld van de ATLAS heb ik niet; toen ik het wilde weten, kon ik de documentatie niet te pakken krijgen. Wat mij ervan bijgebleven is, is dat elk programma zijn eigen sporen op de trommel via een of andere techniek van relatieve adressering kan blijven nummeren van 0, 1, 2, 3, ...; tijdens executie zijn daar physische sporen aan toegevoegd. Hoe precies, weet ik niet; het is -althans nu- voor ons ook niet belangrijk.

Belangrijk is, dat de ATLAS in mijn geheugen beklijfd is als een ARMAC met een aantal buffers. (Ik geloof 32 [,] of 64 als je rijk was.) Voor de programmeur van ieder individueel programma is het niet toegankelijk, welke sporen zich op elk moment in de buffers bevinden. Hij kan vrijelijk als in een echte Von Neumann-machine naar elk woord in zijn programma (opdrachten + werkroumtes) refereren. Hardware detecteert of het geselecteerde spoor in een van de buffers staat; zoja, dan wordt het
overeenkomstige woord van die buffer genomen, zo nee, dan wordt een van de buffers vrijgemaakt en wordt daarin het gevraagde spoor gecopieerd, waarna deze berekening door kan gaan. Dit alles in hardware uitgevoerd.

Dit project is in meer dan 1 opzicht angstig.

In de eerste plaats moeten de comparaties met het ene gevraagde en de 32 (of 64) aanwezige sporen simultaan uitgevoerd worden en de hiervoor benodigde hardware is niet kinderachtig. Het moet allemaal rap gebeuren, want anders campareer je het grootste deel van de tijd.

In de tweede plaats -hoewel dit geloof ik niet inhaerent aan het systeem is- is de strategie volgens welke een buffer vrijgemaakt wordt, ook in de hardware vastgelegd.

Bij de ARMAC hadden we dit strategieprobleem niet: er was maar 1 buffer en een keuze van 1 uit 1 geeft weinig vrijheid.

4.3. GIER ALGOL.

De GIER is een machine met een kerngeheugen van zeg 100 woorden en een 12 keer zo grote trommel met sporen van ongeveer 40 woorden (Precies weet ik het niet, het is in elk geval een raar getal.)

De implementatie van ALGOL op deze machine is voor wat de kerngeheugentoewijzing voor tussenresultaten [betreft] geent op de X1-implementatie, voor opdrachten op de ATLAS, met dit verschil, dat het objectprogramma (uiteindelijk) wel trackbewust geformuleerd wordt.

Karakteristieke eigenschappen van de GIER zijn in dit verband dat het kerngeheugen echt wel klein is, dat de machine snel is en tijdens tracktransporten zelf wel door kan werken. De machine heeft geen interrupt features, multiprogrammering komt niet ter sprake.

De oplossing van Peter Naur, J�rn Jensen c.s. is in wezen als volgt.

Een zo klein mogelijk hoekje van het kerngeheugen is gereserveerd voor het "run time system", dwz. de administratie (programma + gegevens) welke programmasporen in het kerngeheugen staan en waar. Er is alles aan gedaan om dit gedeelte zo compact mogelijk te houden, iets dat je aan begenadigd codeur als Jensen gerust kunt overlaten.

De rest van het kerngeheugen is nu beschikbaar voor variabelen en programma. De programmeur heeft de plicht om te zorgen, dat het aantal variabelen de 500 tot 700 niet overschrijdt. Als hij meer variabelen bespelen wil, kan hij dit, mits hij expliciet -via uit de bibliotheek beschikbare standaardprocedures- variabelen expliciet van en naar de trommel transporteert. De variabelen in het kerngeheugen worden gestapeld, de rest is -buiten controle van de programmeur maar onder controle van
het run time system- ter beschikking voor copieen van programmasporen. Gedurende die periodes, dat het aantal variabelen klein is, is er meer ruimte in het kerngeheugen voor programma en wordt dus minder tijd aan tracktransporten verspeeld. (Een gedeelte van die tijd maakt hij ten nutte door een gecompliceerde administratie bij te werken.)

De werkwijze in het GIER ALGOL system lost voor ons niet alles op: het houdt geen rekening met multiprogrammering en legt de transportlast voor variabelen nog op de schouders van de programmeurs.

Voorts moeten we een vrij wezenlijk verschil niet uit het oog verliezen: bij de GIER moest men ervan uitgaan, dat als regel het kerngeheugen te klein zou zijn om opdrachten en variabelen van een programma alle te bevatten; bij de X8 mogen we er dunkt me van uitgaan, dat het kerngeheugen in de regel daarvoor wel groot genoeg zal zijn.

4.4. Paginaindeling.

Het idee is dus om de informatie op de trommel in "pagina's" in te delen en paginaasgewijs al of niet in het kerngeheugen te laten zitten. Wil dit aanleiding geven tot een redelijk vlotte afwerking, dan moet aan minstens drie voorwaarden voldaan zijn.

(a) Zolang de benodigde informatie wel een plaats in het kerngeheugen heeft, moet er zo min mogelijk tijd veloren gaan door verificatie hiervan.

(b) De informatie moet volgens een dynamisch zinvol criterium over de pagina's verdeeld zijn.

(c) De strategie volgens welke "oude pagina's" uit het kerngeheugen verwijderd worden, moet door een of andere (dynamische) bespiegeling waarschijnlijk gemaakt kunnen worden.

4.4.1. De verificatie. (ad 4.4 sub a).

Bij de ARMAC geschiedde de verificatie door weinig, bij de ATLAS door veel hardware, die simultaan met het rekenproces in actie is. Dit in tegenstelling tot GIER ALGOL, waar gebruikt is, dat veel van deze verificaties weggepraat kunnen worden. Het meest vor de hand liggende feit om te gebruiken is wel, dat als een bepaalde opdracht zich in het kerngeheugen bevindt, dat dan de hele bijbehorende pagina zich in het kerngeheugen bevindt.

Het streven om het werkende programma zo min mogelijk te vertragen, zolang de benodigde informatie op de kernen staat is echter meer dan uitbuiten van evidenties om het aantal verificaties te drukken: we willen er tevens voor zorgen, dat die verificaties, die nog wel uitgevoerd worden, zo min mogelijktijd vergen. Dit is een strategisch uitgangspunt, slechts te rechtvaardigen door de veronderstelling dat een kerngeheugen van 32 K zo groot is, dat het voor vele problemen -dan wel voor lange
tijd- toereikend zal zijn. Een kerngeheugen van 32 K lijkt mij zo groot, dat het zinvol is om te eisen, dat alle programma's die daaraan genoeg hebben, zo min mogelijk gehinderd worden door hun permissie die grens te overschrijden. (Hier scheiden wegen zich heel duidelijk van die van GIER ALGOL, dat in een kerngeheugen van 1000 40-bitswoorden gespeeld moet worden.)

In ons geval betekent dit, dat we, zodra de kerngeheugentoewijzing verandert, bereid moeten zijn om de consequenties hiervan uitgebreid na te gaan en op allerlei plaatsen passende notities te maken: we moeten de dynamische verificaties zoveel mogelijk prepareren.

Een enkel voorbeeld moge dit toelichten. Stel, dat we een minimumadministratie bijhouden, bestaande uit een lijstje van welke pagina's zich waar in het kerngeheugen bevinden. De vraag of een willekeurige pagina zich in het kerngeheugen bevindt, zou dan wel eens een tamelijk tijdrovende affaire kunnen zijn. Naarmate de opdrachtpagina's kleiner zijn, zal het dynamisch vaker voorkomen, dat een sprong naar de volgende -of daaropvolgende- pagina uitgevoerd moet worden. De hieruit voortvloeiende
verificatieplicht kan je drastisch vereenvoudingen door bij elke pagina in kerngeheugen te vermelden of, en zo ja waar, zich de volgende pagina in het kerngeheugen bevindt. Aan de coordinator is dan natuurlijk wel de zoete plicht om bij de wijzigingen deze notities steeds bij te werken.

Het idee van "de volgende opdracht" kan gebruikt worden om vele verificaties overbodig te maken, het idee van "de volgende pagina" om een aantal verificaties, die nog wel uitgevoerd moeten worden, te versnellen. Het versnellen van verificaties door passende preparatie zullen wij, als ik me niet vergis, nog wel vaker tegenkomen; er is nl. een tweede reden, waarom de techniek veelbelovend is. Als een verificatie een negatief resultaat oplevert, moet er getransporteerd
worden: het proces in kwestie kan dus niet verder. Door multiprogrammering bestaat er wel de mogelijkheid, dat er iets anders kan gaan lopen, maar [niet] de zekerheid, dat dat andere niet de Dummy Abstract Machine (zie 3.2., EWD54-7) wordt. Een zinvoller zoethoudertje voor de computer is altijd beter; de coordinator is dan de meest voor de hand liggende candidaat.

4.4.2. Dynamisch zinvolle indeling. (12 juni 1963)

De drie beschreven voorbeelden werken met pagina's van uniforme lengte; bij de X8, waar de lengte van trommeltransporten binnen zeer ruime grenzen vrij kiesbaar is, zijn we dus niet aan vaste paginalengte gebonden. Het is dus redelijk, om zich af te vragen, of we de lengte van elke individuele pagina zich kunnen laten voegen naar de structuur van de informatie, die hij herbergt. Voordat ik nu de mogelijkheden van uniforme versus non-uniforme paginalengtes wil vergelijken, eerst een opmerking.
Als we uniforme paginalengte kiezen, dan is het duidelijk, dat we voor deze uniforme paginalengte een keuze moeten doen -hoe is een andere zaak. De opmerking, die ik maken wilde is, dat we in geval van non-uniforme paginalengte ook een dergelijke quantitatieve keuze moeten doen. Doen we dat niet en kiezen we onze pagina's zo, dat alles dat bij elkaar hoort op 1 enkele pagina komt, dan wordt natuurlijk de uitkomst, dat we het hele programma op een enkele voldoend lange pagina onder gaan brengen. Dat was nu niet
de bedoeling. De enige manier, waarop we de non-uniforme pagina-indeling kunnen aanbrengen, lijkt de volgende. Men neme een zekere ideale paginalengte in het oog. Is het programma daarvoor te lang, dan verdeelt men het programma op de meest natuurlijke manier in stukken; zijn die stukken nog te lang, dan kijkt men naar de interne structuur van deze stukken etc. etc. totdat de hele boel in passende mootjes is opgehakt.

In het geval van uniforme paginalengte ga je, zolang je niet slim probeert te wezen, anders te werk. Het allergrofste is, dat je de tekst van het programma gewoon in vaste stukken van om de zoveel opdrachten ophakt. Een voor de hand liggende verfijning is, om de constanten, die in een pagina gebruikt worden over de pagina op te zamelen en achteraan in dezelfde pagina op te nemen. (Bij dit groeperen moet je even voorzichtig zijn: als je nog ruimte voor een enkele hebt en die opdracht heeft
net een nieuwe constante nodig, dan lukt het niet meer!) Tot zover is GIER ALGOL gegaan en verder niet; probeer je veel verder te gaan, dan krijg je steeds meer overwegingen, die bij de non-uniforme paginaindeling van meet af aan een rol moeten spelen.

En dit soort overwegingen is bedenkelijk, omdat op louter lexicographische gronden niet alle gegevens ter beschikking zijn, die uitmaken, of iets dynamisch zinvol is of niet. Het angstige is, dat het gevolg van vermeende slimheid zelfs averechts kan werken. Een enkel voorbeeld moge dit toelichten: stel dat we in een programma eerst een cyclus A hebben, gevolgd door een cyclus B met een binnencyclus C. Nu beschouwen we twee mogelijke indelingen: cyclus A zit schrijlings over een paginagrens
en cyclus B kan nog in zijn geheel in de tweede pagina; het andere arrangement is, dat we de programmatekst wat doorschuiven, zodat cyclus A in zijn geheel aan het begin van een pagina komt te liggen en nu cyclus B schrijlings over een paginagrens ligt, maat zo, dat de binnencyclus C nog in dezelfde pagina als A past. Op grond van het bekende regeltje "optimalisering is vooral van belang bij binnenste cycli" lijkt het tweede beter, want zowel A als C zitten nu niet meer schrijlings. Maar als A vele
malen minder doorlopen zal worden dan B, dan is de eerste oplossing dynamisch toch echt te preferen! Een en ander ter illustratie van het feit, dat we met minimaliseringen  van pagina-overgangen erg voorzichtig moeten zijn. De ervaring heeft bovendien geleerd, dat dergelijke groepeerderijen, die al gauw op uitgebreide analyses van de samenhang, de "cross-references" in de aangeboden tekst neerkomen, vertaaltijden makkelijk tot een veelvoud doen exploderen. Het dubieuze effect en mijn voorliefde
voor "load and go" maken, dat ik me niet erg aangetrokken kan voelen tot dergelijke optimaliseringen. Het ontaardt gauw in een poging tot raffinement op het verkeerde ogenblik.

Wat kan je wel doen? Wel, het onderbrengen van constantes in de pagina, waarin ze gebruikt worden, is een duidelijk voorbeeld. Het kost haast niets en het succes is verzekerd!

Immers: als we de opeenvolgende opdrachten samen in een pagina onderbrengen, dan wete we, dat we niet zo gek zitten, omdat na een opdracht interesse in de volgende opdracht een heel waarschijnlijk gebeuren is. Als we nu in onze tekst " ...* 2.5" tegenkomen, dan is de uit te voeren handeling "met 2.5 vermenigvuldigen" en als we dat in onze machinecode in twee losse stukken splitsen, dan is deze splitsing iets artificieels. Constanten in dezelfde pagina onderbrengen
is dus niet: de aangeboden tekst "handig groeperen", maar "niet nodeloos ophakken". De vertaler heeft niet de informatie vernietigd, dat het hier om een constante operatie ging. En wat we in elk geval kunnen doen is proberen om het effect van de informatievernietiging door de vertaler te beperken, te mitigeren.

De opmerking, dat je verificatieplichten kunt verkorten door bij elke pagina in kerngeheugen te noteren (zie 4.4.1.) waar je de volgende kunt vinden, is op hetzelfde soort overwegingen geent. Je zult dit eerst raadplegen van de volgende nl. vooral doen bij die sprongopdrachten, die niet in de tekst voorkomen, maar die de vertaler er bij gemaakt heeft. En dat zijn nu juist bijna allemaal voorwaartse sprongen; denk aan het overlopen naar de volgende pagina of de vertaling
van "if..then..else..". Er is geen enkele reden om de if-clause aanleiding te laten geven tot hetzelfde stuk opjectprogramma, dat je gekregen zou hebben als de sourcetekst het met expliciete sprongen -die "overal heen" kunnen zijn- geformuleerd had.

Ik hoop hier meer voorbeelden van te kunnen vinden.

Ik ga er nu maar van uit, dat we "opdrachten + constanten" samen zullen indelen in pagina's van vaste lengte. Blijft natuurlijk de vraag welke lengte.

Als je aanneemt, dat uiteindelijk het kerngeheugen in het merendeel van de gevallen groot genoeg is, dan is er een prae voor lange pagina's. Immers, het aantal malen, dat verificatie uitgevoerd moet worden is dan minder, omdat je vaker in dezelfde pagina blijft en verificatie dus achterwege kan blijven. Bovendien: hoe minder pagina's er in het spel zijn, hoe goedkoper de verificatie. Zodra je ook ernstig rekening moet houden met hetgeval, dat je kerngeheugen te klein blijkt, dan moet
je trommeltransporten vermijden en een van de manieren om dat te doen is zo zuinig mogelijk met je kerngeheugen omspringen. Je verspilt natuurlijk als je maar in een stukje van een pagina geinteresseerd bent, de ruimte, die de rest van de pagina onherroepelijk inneemt. Ik denk dat op het ogenblik -12 juni 1963- over pagina's van 256 opdrachten. (Mijn idee over de ideale paginalengte is tot 256 gegroeid; als het zo doorgaat, zou ik nog wel eens bij 512 terecht kunnen komen.)

De volgende overwegingen zijn voor mij wel verhelderend geweest om in te zien, welke factoren in het spel waren. Als het kerngeheugen voor het totale programma te klein is, dan kunnen we slechts lange tijd trommeltransporten uitsparen, als de besturing in een gedeelte van de programmatekst rondcyclet. Als prototype van de situatie, waarin winst is te boeken, bekijken we dus een cyclus. Dynamisch beschouwd bestaat een cyclus uit een opeenvolging van referenties naar een aantal door het
programma verspreid liggende stukjes consecutief programma (de uitgeschreven cyclus, daarin gebruikte subroutines, formele parameters etc.); de vraag, die we kunnen stellen is "Als al deze stukjes M opdrachten lang zijn, wat is dan de verwachtingswaarde van de geheugenbezetting bij een paginalengte N?". Je zoudt N zo willen kiezen, dat die verwachtingswaarde minimaal is. Dat is hij natuurlijk als we N = 1 kiezen, maar dat is kennelijk niet de bedoeling: sterker, we gaan ervan uit, dan N << M onpractisch
is, omdat het begrip "pagina" dan onvoldoende is om zo'n stuk van M opdrachten bij elkaar te laten horen. De kans dat het stuk van M opdrachten binnen 1 pagina komt te liggen is nu p1 = (N - M)/N, de kans, dat het stuk schrijlings over een paginagrens komt is p2 = M/N. De verwachtingswaarde van de geheugenbezetting is nu N*p1 + 2*N*p2 = N + M. Onder de bijvoorwaarde M < N vind je dus dat de ideale paginalengte N = M zou moeten zijn en een cyclus stouw je gemiddeld in twee keer zo veel geheugen als
je eigenlijk nodig zou hebben. (De kans, dat je twee stukken M in dezelfde pagina zou aantreffen is verwaarloosd.)

Op deze manier bezien is de paginalengte van circa 40 woorden in GIER ALGOL heel goed verdedigbaar: M = 40 lijkt redelijk afgestemd op allerlei standaard subroutines, de stukjes tekst, die actuele parameters vastleggen etc. Het is ook duidelijk, dat wij een grotere paginalengte moeten invoeren.

4.4.3. Voorstel tot versnelling van de verificatie. (19 juni 1963)

Het volgende handelt alleen over de pagina's van uniforme lengte, die ik denk te gebruiken voor het bergen van opdrachten + constanten (dwz. het statische gedeelte van de procesbeschrijving). Om spraakverwarring te voorkomen spreek ik van kernpagina's en trommelpagina's; een kernpagina is een stuk kerngeheugen, bedoeld om een copie van een trommelpagina te herbergen.

Ik ga in hoofdzaak uit van een paginalengte van 256 woorden -de consequenties der onderscheiden technieken zal ik af en toe trloops ook voor pagins's van 512 woorden vermelden-, een kerngeheugen van 32 K en 1 trommel van 512 K.

Het kerngeheugen bergt dan maximaal 128 pagina's, de trommel maximal 2048. Ik neem aan, dat de coordinator er hoge prijs op stelt om per kernpagina enige historie bij te houden, welke trommelpagina er in gecopieerd staat, wanneer er voor het laatst naar gerefereerd is etc. Laten wen aannemen, dat voor deze administratie een woord per kernpagina genoeg is. We hebben dan 128 woorden "kernpaginatabel".

Deze tabel mag dan ideaal zijn om bij een gegeven kernpagina vast te stellen, welke trommelpagina er in gecopieerd staat, hoewel voldoende is deze administratie nauwelijks adequaat om de omgekeerde vraag rap te beantwoorden "Staat trommelpagina [nummer] zo en zo soms ergens op de kernen gecopieerd en zo ja, waar?".

Als we gewoon het lijstje afgaan om te kijken of hij er bij is, dan kost dit bv. 4 geheugencycli per test; aangenomen, dat de helft van het geheugen met uniforma pagina's gevuld is, dan moeten we minimaal 1, maximaal 64 tests aanleggen en komen we op een gemiddelde van 130 geheugencycli per verificatie. (Als het kerngeheugen wat te klein is en we vaak nul op het rekest zullen krijgen, ligt dit gemiddelde hoger, omdat je om afwezigheid te kunne constateren, de hele boel doormoet.) Dit
zou, als we er niets aan doen wel eens 25 procent van de rekentijd kunnen gaan vergen. (Dit is geschat op een verificatie om de 40 opdrachten van gemiddeld 10 geheugencontacten of 50 van gemiddeld 8.) In elk geval te veel. Ik wil dit nu op twee manieren bestrijden:

(1) onderzoeken hoe door extra geheugenruimte en eventueel extra werkzaamheden bij verandering der kerngeheugentoewijzing deze algemene test versneld kan worden

(2) onderzoeken hoe we, zolang de kerngeheugentoewijzing niet verandert het aantal van deze algemene verificaties kunne drukken.

Analoog aan de kernpaginatabel kunnen we een trommelpaginatabel invoeren.

Die zou dan 2048 woorden beslaan en in elk woord zouden we kunnen noteren of en zo ja, waar de trommelpagina in de kernengecopieerd staat. Dit maakt de omgekeerde vraag inderdaad rap beantwoordbaar, maar tegen niet verwaarloosbare kosten. Bij wijziging in de kerngeheugentoewijzing moet de coordinator behalve de kernpaginatabel ook de trommelpaginatabel bijwerken, deze werkzaamheden zijn verwaarloosbaar, de geheugenbezetting van 2048 woorden maximaal is echter iets, waartegen ik wel een
beetje hik. Bedenkend, dat je slechts vraagt naar een 7-bits antwoord, kan je met drie in een woord de omvang van deze tabel tot 683 maximaal drukken, maar dit wordt "trading speed for space".(Bij pagina's van 512 woorden heb je er maximaal 1024 op de trommel en maximaal 64 inhet kerngeheugen: 1024 6-bits antwoorden kunnen op 256 woorden worden afgebeeld. De versnelling door de trommelpaginatabel is nu minder indrukwekkend, omdat scannen van de paginatabel dan ook gemiddeld twee keer zo snel is afgelopen.
Hier zien we overigens mooi, hoe grotere pagina's de administratie drukken.)

Een alternatief om de comprimatie -van 11 tot 7 of van 10 tot 6 bits- te verrichten is om in te voeren een lijstje van de in het kerngeheugen gecopieerde trommelpagina's, naar opklimmend paginanummer gerangschikt. Dit vergt bij verwijdering en toevoeging enige extra administratie -wat geschuif- maar de vraag van het "waar" kan nu met een logarithmische zoekerij, die in gemiddeld 6 (of 5) slagen beeindigd is. De tijdsduur, die voor de logarithmische
zoekerij nodig is is iets groter -volgens voorlopige probeersels- dan die nodig voor het raadplegen van de gecomprimeerde trommelpaginatabel (30 tot 40 geheugencontacten) kortom: op de 400 nog te veel om te licht over te denken.

Je kunt ook de kernpagina's aan een ketting rijgen, aftastbaar in de volgorde van laatste referentie. Het programma, dat zo'n ketting afwandelt is een snoesje, 6 geheugencontacten per slag, als ik me niet heb vergist. Zolang je in een paar pagina's je rondjes draait, gaat de verificatie leuk en kan deze methode met de andere concurreren, maar als je er meer dan vijf, zes schakels af moet tasten, wordt het tijdrovend en moet je rekening gaan houden met maximale zoektijden van bove de milliseconde
(zeg 400 geheugencontacten), nl. voor een zo [lang] vergeten pagina, dat hij al bijna op de nominatie staat om overschreven te worden. Het effect zou zijn, dat bij een plotselinge wisseling de machine "wat op zijn kop moet krabben" zich afvragende waar dat en dat ook al weer stond. In verband met multiprogrammering lijkt me dit niet aantrekkelijk. Een technisch nadeel (dat door scannen van de kernpaginatabel gedeeld wordt) is bovendien het volgende: als een trommelpagina niet op de kernen aanwezig
is, ontdek je dat pas, nadat je de hele ketting, resp. tabel hebt afgewerkt en je trommeltransport kan pas daarna gestart worden. Liever een snellere detectie, dat het mis zit en administratief gehannes, nadat het transport vast in gang gezet is. Als generale administratie is het ketting rijgen dus niet bruikbaar; hoogstens kunnen we voor de vier jongste pagina's bv. in een klein cyclisch magazijntje bijhouden om kleine cycli wat te helpen, als dit nodig mocht zijn. Ik hoop van niet.

Vooralsnog zie ik het meeste heil in de naar opklimmend trommelpaginanummer gesorteerde aanwezigheidslijst, en wel voornamelijk omdat deze lijst evenredig lang is met de kernpaginatabel, zodat ik de twee door elkaar kan vlechten. Ik moet de alternatieven gewoon uitwerken. Hoe zwaar we ruimte tegen tijd laten wegen, hangt er mede van af, hoe frequent we proberen moeten om via deze administratie bij een gegeven trommelpaginanummer een kernpaginanummer te vinden. Vooralsnog vergt dit naar
schatting 10 procent van de rekentijd, ook als alles in het kerngeheugen zit. En dat vind ik nog wat veel.

Nu de tweede mogelijkheid: de opmerking is nl. dat het niet nodig is om de vraag "waar staat trommelpagina nummer zoveel" zo rap te kunnen beantwoorden, als we een mechanisme kunnen bedenken, waardoor het aantal malen, datdeze vraag gesteld  wordt, maar genoeg gedrukt wordt.

Voor sprongen naar de volgende pagina is de oplossing gevonden. Het feitenmateriaal, dat hiervoor door de centrale administratie bijgehouden moet worden is onafhankelijk van de omvang van het programma en onafhankelijk van de intensiteit van interpaginareferenties en tenslotte altijd up to date.

Om verdere verificaties te onderdrukken heb ik allerlei dingen geprobeerd; de grote moeilijkheid was, dat je op grond van aanwezigheid dat gegeven wel overal rond kunt strooien, maar dat het heel moeilijk wordt om na te gaan, wat je dan allemaal herroepen moet, mocht zo'n pagina verdwijnen. Wat wij echter moeten exploiteren is het volgende: we moeten zorgen, dat de vraag "welke trommelpagina staat in die en die kernpagina?" rap beantwoordbaar is. Dat kan en uit het volgende
zal blijken, dat het de moeite loont om daar alles en alles aan te doen

Laten we als specifiek voorbeeld eens beschouwen het meegeven van de link aan een procedure (het meegeven van het beginadres van een impliciete subroutine als actuele parameter is van hetzelfde laken een pak). Het is duidelijk, dat we dit terugkeeradres invariant, dwz. in termen van trommelpaginanummer moeten meegeven, zeg om de gedachten te bepalen een adres in trommelpagina 287. Het is echter onverstandig om bij de Return de centrale administratie zonder meer te confronteren met de
vraag "waar staat trommelpagina 287?". We kunnen er allicht op gokken, dat die nog op dezelfde kernpagina staat. Wat we dus moeten doen is als terugkeeradres twee dingen meegeven: een invariant trommeladres en een kernadres, het laatste als suggestie! De localisatie van een trommelpagina kunnen we beschouwen als een tijdrovend sommetje, waarvan we het antwoord heel makkelijk kunnen controleren en waarvoor we een schatting hebben, die in 95 procent van de gevallen raak zal zijn, nl. als de oude pagina
nog op zijn plaats staat. Als dat niet meer het geval is, dan is er in de tussentijd zoveel gebeurd, dat de verificatietest er nog wel bij kan.

transcribed by Johan E. Mebius

revised Thu, 29 Mar 2007


---

## English translation

### Multiprogramming and the X8 (Continuation of EWD54)

EWD 57

MULTIPROGRAMMING AND THE X8

DIRECT ACCESS TO INSTALMENTS AND SECTIONS  -  DIRECT ACCESS TO INSTALMENTS AND SECTIONS

PART 1: EWD51

0
1
1.1
1.2
1.3
2
2.1
2.2
2.3

PART 2: EWD54

2.4
3
3.1
3.2

PART 3: EWD57

3.3
4
4.0
4.1
4.2
4.3
4.4

DUTCH-ENGLISH DICTIONARY OF PRINCIPAL KEYWORDS

Seinpaal
Semaphore

Verhogen (V)
To raise, To
increase

Prolagen (P),

a neologism
coming from
To try and lower,

Probeer te verlagen
To try and decrease

Ingreep
Interrupt,
Interruption

Kernen
Core storage,

Main storage

Trommelgeheugen
Drum storage

Multiprogramming
and the X8, instalment 3.

(Continuation of EWD51 and EWD54)

June 1963

X8 No. 19.

3.3.
The function of the listening bits. (7 June 1963)

An interrupt (see 2.3., EWD51 - 7) takes place only if

(a) the machine is listening

(b) the collation result of interrupt word(s) and (corresponding) listening word(s) ≠ 0.

By setting a listening bit = 0, one can thus achieve that the associated semaphore, even if the latter has long been positive and the associated interrupt flipflop has long been set = 1, nevertheless brings about no interrupt until further notice.

One is evidently not interested in a semaphore becoming positive if there is nothing waiting on it. Especially in the case of a hardware semaphore, whose being positive in principle brings about an interrupt, it is regrettable if this interrupt takes place and the coordinator can do nothing other than take note of this fact disinterestedly, disinterestedly because nothing in the whole pattern of possibilities changes. An interrupt does, after all, cost time (and we shall have to remain continually alert
to keep this time from increasing too much).

The idea now is to let the coordinator see to it that the listening bit of a hardware semaphore indicates whether or not something is waiting on this semaphore: if the waiting chain that hangs on the hardware semaphore (see 3.2., EWD54 - 8 ff.) is empty, the associated listening bit will be = 0, otherwise = 1. It is the task of the tooth-brushing routine to see to this initially,
it is the task of the coordinator to keep it up to date at all times.

The use of the listening bit is hereby clear. How effective it is I cannot by any means assess. In general it holds that it will be more effective

(a) if abstract machines are, where possible, preferably hung on non-hardware semaphores.

(b) if the accumulation of interrupts can be higher. This last is connected with the size of the start magazine.

We must at any rate look at this, and for two reasons: firstly, this "paying attention" probably gives hardly any occasion for a cost increase -in hardware, time, or number of instructions of the coordinator-; secondly, we must be aware that the saving of unnecessary interrupts would take effect here if the computer did not have to wait on the transput devices, i.e. if the whole factory were "processor limited", i.e. precisely the circumstance in which it
makes every sense to be economical with computer time!

A final remark is that the time expenditure per interrupt is a relatively fixed amount: this counts the more heavily, in percentage terms, the more frequently your peripheral equipment carries out V-operations. (We must not fall into the mistake I made with the X1, where the interrupt program for the tape reader was geared to the 150 char./sec of the Ferranti tape reader -of unblessed memory!- with the sad result that, on account of its now no longer negligible reaction time, it could
no longer keep up with the EL1000!)

4. Automatic transports between slow and fast store.

4.0. Introduction.

Another not unimportant problem, namely that of the addition or removal of abstract machines, I deliberately leave aside for the time being, and for the following reasons. Within the foreseeable future I hope to get sight of the documentation of how other systems, in which this problem has been solved in a certain degree of generality. In these a solution has undoubtedly been found for the assignment of peripheral equipment to programs; besides concrete objects we shall also have to assign abstract objects
-newly to be introduced semaphores, for example-. On account of the greater flexibility we are aiming at, it will lie somewhat less simply for us, but it would be unwise to shrink from profiting from another's experience. The second reason is that the administrative duties upon the addition of new abstract machines might well be co-determined by the structure of these machines. This last might well be drastically influenced by the way in which we intend to arrange the automatic transports between
slow and fast store. This last is therefore more fundamental -and also more difficult; hence I now wish to tackle that first. By way of introduction, a historical survey of three relevant steps.

4.1. The ARMAC.

I still remember very well how wildly enthusiastic we were when we had the design of the ARMAC ready. In its final days, with the X1 on the other side of the corridor, we did sometimes shrug our shoulders pityingly over the old little beast. In retrospect this enthusiasm from 1955 was after all not so unjustified!

In short, the organization of the ARMAC, which was essentially a drum machine, came down to the following. The drum was divided into tracks of 32 words per track. Instruction selection directly from the drum was however impossible: the machine was equipped with a buffer on cores with the capacity of 1 track, and instruction selection consisted of

(1) verification that the right track stood copied in the buffer

(2) selection of the corresponding place in the buffer.

If the first condition was not satisfied, the computing process was, without pardon, held up for 1 drum revolution, in which the buffer was filled with the new track. To make all of that possible, 7 additional flipflops had been introduced (the drum contained 128 tracks, the address length was 7 + 5 = 12 bits) in which the number of the drum track that stood copied in the buffer was remembered. At each instruction selection a test was made for equality of these 7 bits and the 7 most significant bits of the instruction counter.

As an additional feature, at each number selection this same set of 7 was compared with the 7 most significant address bits in the instruction register: in case of equality, on reading one did not wait until the indicated drum place rotated under the heads, but selection was made at once from the corresponding place in the buffer; on writing, writing to the drum did take place, but at the same time the copy in the buffer was kept up to date.

This additional hardware more than paid for itself. Note that the comparison with the 7 bits, which indicated which track stood in the buffer, was carried out twice in each ordinary instruction: first in the instruction cycle, then in the number cycle. In a conventional Von Neumann machine one could hardly do otherwise.

Strictly considered, the majority of these comparisons were superfluous: logically it was admissible -and that was at the time in a certain sense advanced- to run the ARMAC as described above in a non-track-conscious manner. In practice one put this out of one's head: you spent days fiddling at the subroutine for just as long as it took until you had squeezed it into 32 words! And all main programs in the long run also had all constants in the track where they were used; you simply had to.

If we now assume that the machine is programmed track-consciously, then we arrive at the surprising discovery that all comparisons with the 7 bits can be talked away, and with them the 7 flipflops themselves.

The only thing you need are special jump, read, and write instructions to and from such-and-such a place of the buffer. The ordinary jump instructions you can then always let imply renewed filling of the buffer. The only question that then remains is how you pass over from the one track to the other. You could have detected on overflow into the 7th bit of the instruction counter at the same incrementing; an alternative solution is to jump explicitly at the end of a track to the beginning of the next track
-something we did in practice almost always, because you had to jump over a couple of constants anyway.

Viewed in this way, the automatic 7-bit comparison is thus really, though pretty, utterly superfluous: the programmer could know what the outcome would be (and he did well to know this).

4.2. ATLAS.

A complete picture of the ATLAS I do not have; when I wanted to know it, I could not get hold of the documentation. What has stuck with me of it is that each program can keep numbering its own tracks on the drum, via some technique of relative addressing, from 0, 1, 2, 3, ...; during execution physical tracks are added to these. How precisely, I do not know; it is -at least now- not important for us either.

What is important is that the ATLAS has lodged in my memory as an ARMAC with a number of buffers. (I believe 32 [,] or 64 if you were rich.) For the programmer of each individual program it is not accessible which tracks are at any moment in the buffers. He can freely, as in a real Von Neumann machine, refer to any word in his program (instructions + working spaces). Hardware detects whether the selected track stands in one of the buffers; if so, then the
corresponding word of that buffer is taken; if not, then one of the buffers is freed and into it the requested track is copied, after which this computation can proceed. All of this carried out in hardware.

This project is in more than 1 respect frightening.

In the first place the comparisons of the one requested with the 32 (or 64) present tracks must be carried out simultaneously, and the hardware required for this is no trifle. It must all happen rapidly, for otherwise you are comparing for the greatest part of the time.

In the second place -although I believe this is not inherent to the system- the strategy according to which a buffer is freed is also laid down in the hardware.

With the ARMAC we did not have this strategy problem: there was only 1 buffer, and a choice of 1 out of 1 gives little freedom.

4.3. GIER ALGOL.

The GIER is a machine with a core store of, say, 100 words and a drum 12 times as large with tracks of about 40 words (I do not know precisely, it is at any rate an odd number.)

The implementation of ALGOL on this machine is, as regards the core store allocation for intermediate results, grafted onto the X1 implementation, for instructions onto the ATLAS, with this difference, that the object program is (ultimately) formulated track-consciously after all.

Characteristic properties of the GIER are, in this connection, that the core store really is quite small, that the machine is fast and can itself carry on working during track transports. The machine has no interrupt features, multiprogramming does not come up for discussion.

The solution of Peter Naur, Jørn Jensen and colleagues is essentially as follows.

A corner of the core store as small as possible is reserved for the "run time system", i.e. the administration (program + data) of which program tracks stand in the core store and where. Everything has been done to keep this part as compact as possible, something you can safely leave to a gifted coder like Jensen.

The rest of the core store is now available for variables and program. The programmer has the duty to see to it that the number of variables does not exceed the 500 to 700. If he wishes to manipulate more variables, he can do so, provided he explicitly -via standard procedures available from the library- transports variables explicitly to and from the drum. The variables in the core store are stacked, the rest is -outside the control of the programmer but under the control of
the run time system- available for copies of program tracks. During those periods in which the number of variables is small, there is more room in the core store for program, and hence less time is wasted on track transports. (Part of that time he turns to good account by updating a complicated administration.)

The method of working in the GIER ALGOL system does not solve everything for us: it makes no allowance for multiprogramming and still lays the transport burden for variables on the shoulders of the programmers.

Furthermore we must not lose sight of a fairly essential difference: with the GIER one had to start from the assumption that, as a rule, the core store would be too small to contain all the instructions and variables of a program; with the X8 we may, it seems to me, start from the assumption that the core store will, as a rule, be large enough for that.

4.4. Page division.

The idea is thus to divide the information on the drum into "pages" and to let it sit, page by page, in the core store or not. If this is to give occasion for a reasonably brisk handling, then at least three conditions must be satisfied.

(a) As long as the required information does have a place in the core store, as little time as possible must be lost through verification of this.

(b) The information must be distributed over the pages according to a dynamically meaningful criterion.

(c) The strategy according to which "old pages" are removed from the core store must be capable of being rendered probable by some (dynamic) reflection.

4.4.1. The verification. (concerning 4.4 under a).

With the ARMAC the verification took place by means of little hardware, with the ATLAS by means of much hardware, which is in action simultaneously with the computing process. This in contrast to GIER ALGOL, where use is made of the fact that many of these verifications can be talked away. The most obvious fact to use is surely that, if a particular instruction is in the core store, then the whole associated page is in the core store.

The striving to delay the running program as little as possible, as long as the required information stands on the cores, is however more than exploiting self-evident facts to reduce the number of verifications: we also want to see to it that those verifications which are still carried out demand as little time as possible. This is a strategic point of departure, justifiable only by the supposition that a core store of 32 K is so large that for many problems -or for a long
time- it will be sufficient. A core store of 32 K seems to me so large that it is meaningful to require that all programs which can make do with it be hindered as little as possible by their permission to cross that boundary. (Here the ways part very clearly from those of GIER ALGOL, which must be played in a core store of 1000 40-bit words.)

In our case this means that we, as soon as the core store allocation changes, must be prepared to go into the consequences of this extensively and to make appropriate notes in all sorts of places: we must prepare the dynamic verifications as much as possible.

A single example may illustrate this. Suppose we keep a minimum administration, consisting of a little list of which pages are where in the core store. The question whether an arbitrary page is in the core store might then well be a rather time-consuming affair. The smaller the instruction pages are, the more often it will dynamically occur that a jump to the next -or subsequent- page must be carried out. The resulting
verification duty you can drastically simplify by noting at each page in core store whether, and if so where, the next page is in the core store. The coordinator then of course has the sweet duty of keeping these notes up to date at the changes.

The idea of "the next instruction" can be used to make many verifications superfluous, the idea of "the next page" to speed up a number of verifications which still must be carried out. The speeding up of verifications by appropriate preparation we shall, if I am not mistaken, encounter more often still; there is, namely, a second reason why the technique is promising. If a verification yields a negative result, a transport must take
place: the process in question can thus not proceed. Through multiprogramming there does exist the possibility that something else can start running, but [not] the certainty that that other thing does not become the Dummy Abstract Machine (see 3.2., EWD54-7). A more meaningful pacifier for the computer is always better; the coordinator is then the most obvious candidate.

4.4.2. Dynamically meaningful division. (12 June 1963)

The three described examples work with pages of uniform length; with the X8, where the length of drum transports is freely choosable within very wide limits, we are thus not bound to a fixed page length. It is therefore reasonable to ask oneself whether we can let the length of each individual page conform to the structure of the information it harbours. Before I now wish to compare the possibilities of uniform versus non-uniform page lengths, first a remark.
If we choose uniform page length, then it is clear that we must make a choice for this uniform page length -how is another matter. The remark I wished to make is that in the case of non-uniform page length we must also make such a quantitative choice. If we do not do that, and choose our pages such that everything that belongs together comes onto 1 single page, then the outcome will of course be that we shall accommodate the whole program on a single sufficiently long page. That was not
the intention. The only way in which we can effect the non-uniform page division seems to be the following. One takes a certain ideal page length in mind. If the program is too long for that, then one divides the program in the most natural way into pieces; if those pieces are still too long, then one looks at the internal structure of these pieces, etc. etc. until the whole lot has been chopped up into suitable chunks.

In the case of uniform page length you proceed, as long as you do not try to be clever, differently. The very crudest is that you simply chop up the text of the program into fixed pieces of so-many instructions each. An obvious refinement is to gather up the constants used in a page across the page and to include them at the back in the same page. (With this grouping you must be a bit careful: if you still have room for a single one and that instruction
just happens to need a new constant, then it no longer works out!) Thus far GIER ALGOL went and no further; if you try to go much further, then you get more and more considerations which, in the non-uniform page division, must play a part from the very outset.

And this sort of consideration is questionable, because on purely lexicographic grounds not all the data are available which determine whether something is dynamically meaningful or not. The frightening thing is that the consequence of supposed cleverness can even work in reverse. A single example may illustrate this: suppose that in a program we first have a loop A, followed by a loop B with an inner loop C. Now we consider two possible divisions: loop A sits astride a page boundary
and loop B can still fit in its entirety in the second page; the other arrangement is that we shift the program text along a bit, so that loop A comes to lie in its entirety at the beginning of a page and now loop B lies astride a page boundary, but in such a way that the inner loop C still fits in the same page as A. On the grounds of the well-known little rule "optimization is above all of importance for innermost loops", the second seems better, for both A and C now no longer sit astride. But if A will
be traversed many times less than B, then the first solution is dynamically really to be preferred after all! All this by way of illustration of the fact that we must be very careful with minimizations of page transitions. Experience has moreover taught that such grouping efforts, which soon come down to extensive analyses of the coherence, the "cross-references" in the offered text, easily make translation times explode to a multiple. The dubious effect and my predilection
for "load and go" make it that I cannot feel very much attracted to such optimizations. It soon degenerates into an attempt at refinement at the wrong moment.

What can you do then? Well, the accommodating of constants in the page in which they are used is a clear example. It costs almost nothing and success is assured!

For: if we accommodate the successive instructions together in a page, then we know that we are not sitting so foolishly, because after an instruction interest in the next instruction is a very probable occurrence. If we now encounter in our text " ...* 2.5", then the action to be carried out is "multiply by 2.5", and if we split that into two separate pieces in our machine code, then this splitting is something artificial. Accommodating constants in the same page
is thus not: "conveniently grouping" the offered text, but "not needlessly chopping up". The translator has not destroyed the information that this concerned a constant operation. And what we can at any rate do is to try to limit, to mitigate, the effect of the destruction of information by the translator.

The remark that you can shorten verification duties by noting at each page in core store (see 4.4.1.) where you can find the next one is grafted onto the same sort of considerations. You will do this first consulting of the next one, namely, above all for those jump instructions which do not occur in the text, but which the translator has made up in addition. And those are precisely almost all of them forward jumps; think of the passing over to the next page, or of the translation
of "if..then..else..". There is no reason whatsoever to let the if-clause give occasion to the same piece of object program that you would have got if the source text had formulated it with explicit jumps -which can be "to anywhere".

I hope to be able to find more examples of this.

I now simply assume that we shall divide "instructions + constants" together into pages of fixed length. There remains of course the question of which length.

If you assume that ultimately the core store is in the majority of cases large enough, then there is an advantage for long pages. For the number of times that verification must be carried out is then fewer, because you stay more often in the same page and verification can thus be omitted. Moreover: the fewer pages are in play, the cheaper the verification. As soon as you must also seriously reckon with the case that your core store turns out too small, then you must
avoid drum transports, and one of the ways to do that is to be as economical as possible with your core store. You waste, of course, if you are interested in only a little piece of a page, the room that the rest of the page irrevocably occupies. I am thinking at the moment -12 June 1963- of pages of 256 instructions. (My idea about the ideal page length has grown to 256; if it keeps on like this, I might well end up at 512.)

The following considerations have been illuminating for me in seeing which factors were in play. If the core store is too small for the total program, then we can only save drum transports for a long time if the control cycles around within a part of the program text. As prototype of the situation in which gain is to be booked we therefore look at a loop. Dynamically considered, a loop consists of a succession of references to a number of little pieces of consecutive program scattered through the
program (the written-out loop, subroutines used therein, formal parameters etc.); the question we can pose is "If all these little pieces are M instructions long, what then is the expectation value of the store occupation at a page length N?". You would want to choose N such that that expectation value is minimal. It is of course minimal if we choose N = 1, but that is evidently not the intention: rather, we start from the assumption that N << M is impractical,
because the concept "page" is then insufficient to let such a piece of M instructions belong together. The chance that the piece of M instructions comes to lie within 1 page is now p1 = (N - M)/N, the chance that the piece comes to lie astride a page boundary is p2 = M/N. The expectation value of the store occupation is now N*p1 + 2*N*p2 = N + M. Under the side condition M < N you thus find that the ideal page length ought to be N = M, and a loop you cram on average into twice as much store as
you would actually need. (The chance that you would find two pieces M in the same page has been neglected.)

Viewed in this way, the page length of about 40 words in GIER ALGOL is very well defensible: M = 40 seems reasonably attuned to all sorts of standard subroutines, the little pieces of text which fix actual parameters etc. It is also clear that we must introduce a larger page length.

4.4.3. Proposal for speeding up the verification. (19 June 1963)

The following deals only with the pages of uniform length which I intend to use for the storing of instructions + constants (i.e. the static part of the process description). To avoid confusion of speech I speak of core pages and drum pages; a core page is a piece of core store, intended to harbour a copy of a drum page.

I start mainly from a page length of 256 words -the consequences of the distinguished techniques I shall now and then also mention in passing for pages of 512 words-, a core store of 32 K and 1 drum of 512 K.

The core store then harbours at most 128 pages, the drum at most 2048. I assume that the coordinator sets great store by keeping some history per core page, of which drum page stands copied in it, when it was last referred to etc. Let us assume that one word per core page is enough for this administration. We then have a 128-word "core page table".

This table may then be ideal for establishing, given a core page, which drum page stands copied in it, but however sufficient, this administration is hardly adequate to answer rapidly the reverse question "Is drum page [number] such-and-such perhaps copied somewhere on the cores and, if so, where?".

If we simply run down the little list to see whether it is among them, then this costs, say, 4 store cycles per test; assuming that half of the store is filled with uniform pages, then we must lay out at least 1, at most 64 tests, and we arrive at an average of 130 store cycles per verification. (If the core store is somewhat too small and we shall often get nil returns on our request, this average lies higher, because to be able to ascertain absence you must go through the whole lot.) This
might well, if we do nothing about it, come to demand 25 percent of the computing time. (This is estimated on a verification every 40 instructions of on average 10 store contacts, or 50 of on average 8.) In any case too much. I now wish to combat this in two ways:

(1) to investigate how, by means of extra store room and possibly extra labour upon a change of the core store allocation, this general test can be speeded up

(2) to investigate how we, as long as the core store allocation does not change, can reduce the number of these general verifications.

Analogously to the core page table we can introduce a drum page table.

That would then occupy 2048 words and in each word we could note whether and, if so, where the drum page stands copied in the cores. This indeed makes the reverse question rapidly answerable, but at non-negligible cost. Upon a change in the core store allocation the coordinator must, besides the core page table, also update the drum page table; these labours are negligible, but the store occupation of 2048 words at most is, however, something at which I do
hiccup a bit. Bearing in mind that you ask only for a 7-bit answer, you can, with three to a word, reduce the size of this table to 683 at most, but this becomes "trading speed for space". (With pages of 512 words you have at most 1024 of them on the drum and at most 64 in the core store: 1024 6-bit answers can be mapped onto 256 words. The speed-up by the drum page table is now less impressive, because scanning of the page table is then also finished on average twice as fast.
Here, incidentally, we see nicely how larger pages reduce the administration.)

An alternative for performing the compression -from 11 to 7, or from 10 to 6 bits- is to introduce a little list of the drum pages copied in the core store, ranked by ascending page number. This requires, upon removal and addition, some extra administration -some shuffling- but the question of the "where" can now be done with a logarithmic search which is concluded in on average 6 (or 5) rounds. The time duration required for the logarithmic
search is somewhat greater -according to provisional trials- than that needed for consulting the compressed drum page table (30 to 40 store contacts); in short: at 400, still too much to think lightly about.

You can also string the core pages on a chain, scannable in the order of last reference. The program that walks down such a chain is a little gem, 6 store contacts per round, if I have not made a mistake. As long as you are turning your circles in a couple of pages, the verification goes nicely and this method can compete with the others, but if you have to scan more than five, six links of it, it becomes time-consuming and you must begin to reckon with maximum search times of above the millisecond
(say 400 store contacts), namely for a page so [long] forgotten that it already stands almost on the nomination to be overwritten. The effect would be that, on a sudden change-over, the machine "has to scratch its head a bit", asking itself where this and that stood again. In connection with multiprogramming this seems to me not attractive. A technical disadvantage (which is shared by scanning of the core page table) is moreover the following: if a drum page is not present on the cores,
you discover that only after you have worked through the whole chain, respectively table, and your drum transport can only thereafter be started. Rather a faster detection that something is wrong and administrative fiddling after the transport has already been set in motion. As general administration the chain-stringing is thus not usable; at most we can keep, for the four youngest pages for example, a small cyclic little magazine, to help small loops a bit, if this should be necessary. I hope not.

For the time being I see the most salvation in the presence list sorted by ascending drum page number, and indeed mainly because this list is proportionally as long as the core page table, so that I can braid the two together. I must simply work out the alternatives. How heavily we let room weigh against time depends partly on how frequently we must try to find, via this administration, a core page number for a given drum page number. For the time being this demands, by
estimate, 10 percent of the computing time, even if everything is in the core store. And that I still find rather much.

Now the second possibility: the remark is, namely, that it is not necessary to be able to answer the question "where does drum page number such-and-such stand" so rapidly, if we can devise a mechanism by which the number of times this question is posed is sufficiently reduced.

For jumps to the next page the solution has been found. The factual material that must be kept up to date for this by the central administration is independent of the size of the program and independent of the intensity of inter-page references and, finally, always up to date.

To suppress further verifications I have tried all sorts of things; the great difficulty was that, on the grounds of presence, you can indeed scatter that datum around everywhere, but that it becomes very difficult to ascertain what you must then all revoke, should such a page disappear. What we must, however, exploit is the following: we must see to it that the question "which drum page stands in such-and-such a core page?" is rapidly answerable. That can be done, and from the following
it will appear that it is worth doing everything and anything for that

Let us, as a specific example, consider the passing of the link to a procedure (the passing of the start address of an implicit subroutine as actual parameter is cut from the same cloth). It is clear that we must pass this return address invariantly, i.e. in terms of drum page number, say, to fix our thoughts, an address in drum page 287. It is however unwise, upon the Return, to confront the central administration straight away with the
question "where does drum page 287 stand?". We may well gamble that it still stands on the same core page. What we must therefore do is to pass, as return address, two things: an invariant drum address and a core address, the latter as a suggestion! The localization of a drum page we can regard as a time-consuming little sum, whose answer we can very easily check and for which we have an estimate that in 95 percent of the cases will be on the mark, namely if the old page
still stands in its place. If that is no longer the case, then so much has happened in the meantime that the verification test can still be done in addition.

transcribed by Johan E. Mebius

revised Thu, 29 Mar 2007
