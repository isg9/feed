---
title: "Over de sequentialiteit van procesbeschrijvingen (EWD35)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD35.html
published: "1963-01-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD35.PDF
---

# Over de sequentialiteit van procesbeschrijvingen

Het is niet ongebruikelijk, wanneer een spreker zijn voordracht begint met een inleiding. Omdat ik mij hier mogelijk richt tot een gehoor, dat als geheel minder vertrouwd is met de problematiek, die ik wil aansnijden en de terminologie, die ik zal moeten gebruiken, wilde ik in dit geval ter introductie graag twee inleidingen houden, nl. een om de achtergrond van de problemen te schilderen en een tweede, om U een gevoel te geven voor de aard van de logische problemen, die wij tegen het lijf zullen lopen.

Mijn eerste inleiding is een historische. Eerst moet ik U vertellen, wat U zich moet voorstellen bij een sequentiële procesbeschrijving. U kent allen dergelijke procesbeschrijvingen. Beschouw bv. de beschrijving van de constructie van de hoogtelijn uit het hoekpunt C van de driehoek ABC. Deze kan als volgt luiden:

1) trek de cirkel met middelpunt A en straal = AC;

2) trek de cirkel met middelpunt B en straal = BC;

3) trek de rechte, die bepaald is door de snijpunten van de onder 1) en 2) genoemde cirkels.

Dit is een beschrijving ten bate van iemand, wiens "parate kennis" de constructie van de hoogtelijn niet omvat; ten bate van hem is de constructie van de hoogtelijn opgebouwd uit een aantal handelingen van een beperkter repertoire, handelingen, die hij in de opgegeven volgorde, de een na de ander, dwz. sequentieel, kan uitvoeren. Het zal het oplettende lezertje niet ontgaan zijn, dat wij hierbij meer gespecificeerd hebben, dan strikt noodzakelijk: het is wel essentieel, dat handeling 3) pas uitgevoerd wordt, nadat handelingen 1) en 2) voltooid zijn, handelingen 1) en 2) hadden best in de omgekeerde volgorde plaats mogen vinden, sterker, een tekenaar. die twee passers tegelijkertijd bedienen kan, bij wijze van spreken met elke hand één, mag handelingen 1) en 2) simultaan uitvoeren. Maar deze vrijheid is in onze procesbeschrijving geheel onder tafel geraakt.

Nu een numerieker voorbeeld, dat iets meer aansluit bij de denkwereld, waarin de straks te behandelen problemen actueel zijn geworden. Gegeven de waarden van de variabelen a, b, c en d, gevraagd te berekenen de waarde van de variabele x, bepaald door de formule "x:=(a+b)*(c+d)". Wanneer het uitvoeren van deze waardetoekenning niet tot het primitieve repertoire van de rekenaar —en denkt U nu langzamerhand maar aan de rekenmachine— behoort, dan dient dit rekenvoorschrift uit elementairdere stappen opgebouwd te worden, bv.:

1) bereken t1:= a + b;

2) bereken t2:= c + d ,

3) bereken x:= t1 * t2.

Dit rekenvoorschrift bevat precies dezelfde overbepaaldheid als onze hoogtelijn-constructie: de volgorde van de eerste twee stappen doet niet ter zake, zij zouden zelfs simultaan uitgevoerd kunnen worden.

De bovenstaande procesbeschrijving is representatief voor de werkingswijze van meer klassieke rekenautomaten. Het repertoire van elementaire operaties is, omdat voor elke operatie speciale technische voorzieningen vereist zijn, eindig en om financiële redenen zelfs heel beperkt; deze machines danken hun enorme flexibiliteit aan het feit, dat willekeurig ingewikkelde algebraïsche expressies als boven geïllustreerd uitgewerkt kunnen worden door een sequens van dergelijke elementaire operaties. Inhaerent aan deze sequentiële beschrijving van het proces is, dat men meer specificeert, dan strikt noodzakelijk. Dit heeft lange tijd niet gehinderd, maar in de laatste jaren is daar verandering in gekomen, en wel door twee oorzaken: verandering van de structuur van machines en de komst van een nieuwe klasse problemen, waarvoor men deze machines zou willen inzetten.

De klassieke sequentiële machine doet "een ding tegelijk" en verricht de ene functie na de andere. Om der wille van de efficiency is: men voor specifieke functies meer specifieke apparatuur gaan bouwen, maar als je er dan aan vast houdt, dat de verschillende operaties strict sequentieel uitgevoerd zullen worden, dan dringt zich van een machine plotseling het beeld op van een samenspel van N organen, waarvan er op elk discreet ogenblik maar 1 of 2 werken. Als N groot is, betekent dit, dat het merendeel van je machine het merendeel van de tijd stilstaat en dat is kennelijk niet de bedoeling.

De tweede oorzaak is de komst van een nieuwe categorie van problemen; oorspronkelijk werden rekenautomaten alleen gebruikt voor van te voren volledig gedefinieerde processen, en de machine kon het proces vervolgens op zijn eigen houtje, in zijn eigen tempo uitvoeren. Zodra men echter het informatieverwerkende proces samen wil laten spelen met een of ander ander proces —bv. een chemisch proces, dat door de rekenautomaat gestuurd moet worden— dan treden er heel andere problemen op. Aanhakend bij ons voorbeeld van de berekening van x:=(a + b)*(c + d) zijn we dan niet meer in de omstandigheid, dat het er niet meer toe doet, welke som het eerste gevormd wordt. Als de gegevens a,b,c en d op onbekende momenten door de omgeving van de rekenautomaat verschaft worden en het de taak van de machine is, om de waarde van x zo snel mogelijk af te leveren, dan betekent dit, dat de automaat een deelberekening maar vast uit moet voeren, zodra de daarvoor vereiste gegevens binnen zijn: welke optelling men dan eerst uitgevoerd wil hebben, zal afhangen van de volgorde, waarin de gegevens a, b, c en d ter beschikking komen. Dit was mijn eerste inleiding en ik hoop U hiermee een idee gegeven te hebben, van wat U zich bij een sequentiële procesbeschrijving voor moet stellen, en waarom de vraag gesteld is naar minder strict sequentiële procesbeschrijvingen.

Nu mijn tweede inleiding, die bedoeld is om U van meet af aan enig idee te geven van de logische problemen, die we door het verlaten van de stricte sequentialiteit ons op de hals zullen halen.

Evenals in een normaal stuk algebra variabelen met letters, algemeen "namen", aangeduid worden, heeft men in het proces van informatieverwerking aanhoudend de plicht tot identificatie, de plicht om bij de naam het hierdoor benoemde object te vinden. Onder omstandigheden zou men graag willen beschikken over het selectieproces, waarover de juffrouw in de klas beschikt, wanneer zij zegt "Jantje, kom eens voor het bord." Aangenomen, dat de klas als geheel oplet en gehoorzaam is, werkt dit proces feilloos onder de aanname, dat er 1 en slechts 1 Jantje in de klas zit. Het bijzondere van dit selectiemechanisme is, dat het werkt ongeacht het aantal kinderen in de klas. Deze idylle is helaas technisch niet of hoogstens gebrekkig te verwezenlijken, en wij krijgen een probleemstelling, die meer in overeenstemming is met de werkelijkheid, wanneer de kinderen niet spontaan op het noemen van hun naam reageren en de juffrouw, wanneer zij Jantje voor het bord wil hebben, genoodzaakt is om Jantje bij het oor te vatten en daaraan voor het bord te slepen. Als dit nu een ouderwetse school is, waarin ieder kind zijn vaste plaats in de klas heeft, dan kan de juffrouw, als zij aan het begin van het jaar de kinderen op hun plaats gezet heeft, ze voor haar eigen gemak herdopen, en de naam "Jantje" —die voor het vinden van het jongetje niet erg behulpzaam is— vervangen door een identificatie, die blijkens zijn structuur meteen definieert, welk oor ze vatten moet: ze kan het jongetje noemen "derde rij, tweede bank". Dit is in wezen de techniek, die in klassieke automaten bij klassieke toepassingen gevolgd wordt. In modernere toepassingen kan men dit niet meer doen, en moet men bv. opgewassen zijn tegen de problemen, die de juffrouw krijgt, als de kinderen kunnen kiezen, waar ze gaan zitten.

Om het de juffrouw niet onmogelijk te maken, nemen we aan dat elk der onwillige kinderen een naamplaatje opgespeld heeft. AIs de juffrouw nu Jantje voor het bord wil hebben, moet zij dus Jantje opzoeken. De juffrouw werkt sequentieel, kan slechts een naamplaatje tegelijkertijd lezen, en de juffrouw werkt systematisch: zij heeft voor zichzelf de banken op een of andere manier uniek geordend en als zij Jantje wil hebben, dan loopt zij de banken in deze volgorde af, zich steeds afvragend "Ben jij Jantje?", zo nee, dan gaat ze naar de volgende bank. Dit proces werkt feilloos, mits wij kunnen garanderen, dat de kinderen tijdens de rondgang van de juffrouw niet gaan verzitten, mits wij kunnen garanderen, dat de twee processen "kindlocalisatie" en "kinderpermutatie" niet simultaan uitgevoerd worden. Want U begrijpt ook wel, dat Jantje, zodra hij door heeft, dat hij de gezochte figuur is, achter in de klas gaat zitten en zodra de juffrouw de voorste helft van de klas doorzocht heeft, springt hij gauw naar voren. Wij kunnen de juffrouw verfijnen en haar, als ze de hele klas doorzocht heeft en Jantje niet gevonden heeft met frisse moed van voren af aan laten beginnen, maar dit is geen oplossing, want Jantje springt dan gauw weer naar achteren. Nu zou U kunnen opmerken, dat als die permutaties nu niet met zoveel kwaadwilligheid, maar slechts random uitgevoerd werden, dat dan de verwachtingstijd voor het localiseren van Jantje eindig blijft, ja zelfs door zijn vrijheid rond te springen niet groter is geworden. Deze opmerking, hoe hoopvol, brengt geen soulaas. Zij die zich verdiept hebben in de "Theory of computability" weten hoeveel gewicht gehecht wordt aan een eindige, dwz. een gegarandeerd eindigende algorithme, en zij zullen begrijpen, dat een proces, dat potentieel oneindig lang doortolt, geen aantrekkelijk element is. Veel erger is, dat het in de praktijk veelal de vraag is, of wij de permutaties van de kinderen wel als random mogen beschouwen. Als de kans eindig is, dat als de juffrouw een keer door de klas geweest is, de kinderen weer precies zo zitten als toen de juffrouw begon te zoeken, dan kon dit best eens de eerste periode van een zuiver periodiek verschijnsel zijn.

Maar zelfs als wij het risico, dat we Jantje nooit zullen vinden voor lief nemen, dan nog is haar ellende niet te overzien, want naast het risico, dat ze Jantje niet vindt, loopt ze het veel grotere gevaar, dat ze het verkeerde kind voor het bord sleurt, nl. als de kinderen tussen het moment, dat zij voldaan geconstateerd heeft "Zo, jij ben dus Jantje" en het moment, dat zij, op grond van deze constatering haar hand naar het bijbehorend oor heeft uitgestoken, nog gauw van plaats verwisselen. In een informatieverwerkend proces zou dit neerkomen op een ramp. En dit is het einde van mijn tweede inleiding, waarvan ik hoop, dat zij U enig idee gegeven heeft van het soort probleem, dat wij tegen zullen komen.

Aan het einde van mijn eerste inleiding heb ik twee oorzaken genoemd, die voor een groeiende belangstelling in minder strict sequentiële procesbeschrijvingen verantwoordelijk waren. Ik heb ze slechts luchtig aangestipt en het is U misschien niet opgevallen, dat ze aan bijna tegengestelde entourages ontleend waren. In het eerste geval noemde ik de toenemende complexiteit van machines, die nu opgebouwd zijn uit een aantal min of meer autonoom werkende onderdelen en de opgave was, om deze samen één proces uit te laten voeren. In het tweede geval, waar we de automaat in een z.g. "real time application" bekeken, werd het beeld opgeroepen van één enkel apparaat, dat de mogelijkheid had bij een verandering van de externe urgentiesituatie over te schakelen op het nu urgentste van zijn mogelijke taken. Deze twee arrangementen, een samenspel van een aantal machines aan één proces en omgekeerd, één machine zijn aandacht verdelend over een aantal processen, zijn lange tijd als elkaar wezensvreemd beschouwd. Men heeft er zelfs verschillende namen voor uitgevonden, het ene noemt men "parallel programming" en het andere noemt men "multiprogramming maar U moet mij niet meer vragen wat wat, want dat kan ik niet meer onthouden, sinds ik ontdekt heb dat de logische problemen, die zij door de non-sequentialiteit van de procesdefinitie oproepen, in beide gevallen exact dezelfde zijn.

Ik zal mij in het volgende bedienen van het beeld van een aantal, onderling zwak gekoppelde, op zichzelf sequentiële machines, daarbij aansluitend op de eerste entourage. Maar dit is slechts een beschrijvingswijze, want met wat ik in het vervolg een machine zal noemen komt niet noodzakelijkerwijze een discreet aanwijsbaar stuk apparatuur overeen: in het volgende zijn mijn onderscheiden machines mogelijk heel abstract en representeren zij niet meer dan een opzichzelf sequentieel deelproces. Dit komt onder andere daarin tot uiting, dat ik een grote voorliefde zal tonen voor een samenspel van N machines, N willekeurig. Als ik de term machine alleen maar bezigde voor een aanwijsbaar functioneel stuk apparatuur, dan zou ik het aantal machines voor elke installatie als vrij constant mogen beschouwen. Nu echter een machine, als personifiëring van een opzichzelf sequentiële deeltaak, door de gebruiker ad libitum gecreëerd kan worden, is een dergelijke vaste bovengrens niet meer acceptabel. Ik ga dus praten over het samenspel van N machines, N niet alleen willekeurig, maar als het moet zelfs tijdens het proces variabel. Overtollige machines kunnen vernietigd worden, nieuwe machines kunnen naar behoefte er bij gecreëerd worden en in het samenspel der overigen worden opgenomen. Ik hoop, dat U thans op mijn gezag wilt aannemen, danwel na afloop zelf inziet, dat wij ons zonder de algemeenheid te schaden kunnen beperken tot sequentiële machines, die een of ander cyclisch proces uitvoeren.

Laat ons beginnen met een heel eenvoudig probleem. Gegeven twee machines A en B, beide bezig met een cyclisch proces. In de cyclus van machine A komt een zeker critisch traject voor, TA genaamd, en in die van machine B komt een critisch traject TB voor. De opgave is om er voor te zorgen, dat nooit gelijktijdig de beide machines elk aan hun critische traject bezig zijn. (In termen van onze tweede inleiding: machine A zou kunnen zijn de juffrouw, traject TA het selectieproces van een nieuwe leerling, die voor het bord gehaald moet worden, terwijl het traject TB het proces "kinderpermutatie" representeert.) Er mogen geen veronderstellingen gemaakt worden over de relatieve snelheden der machines A en B, de snelheid, waarmee zij werken hoeft zelfs niet constant te zijn. Het is duidelijk, dat wij de wederzijdse uitsluiting van het critische traject slechts kunnen realiseren, als de twee machines op een of andere manier met elkaar kunnen communiceren. Voor deze communicatie stellen wij enig gemeenschappelijk geheugen ter beschikking, dwz. een aantal variabelen, die voor beide machines in beide richtingen toegankelijk zijn, dwz. waaraan beide machines een waarde kunnen toekennen en waarvan beide machines naar de heersende waarde kunnen informeren. Deze twee handelingen, toekennen van een nieuwe waarde en informeren naar de heersende waarde gelden als ondeelbare handelingen, dwz. als beide machines "tegelijkertijd" een waarde aan een gemeenschappelijke variabele willen toekennen, dan is de waarde na afloop of de ene, of de andere toegekende waarde, maar niet een of ander mengsel. Evenzo: als de ene machine informeert naar de waarde van een gemeenschappelijke variabele op het moment, dat de andere machine er een nieuwe waarde aan toekent, dan dringt tot de vragende machine of de oude, of de nieuwe, maar niet een wilde waarde door. Het zal blijken, dat we ons voor dit doel kunnen beperken tot gemeenschappelijke logische variabelen, dwz. variabelen met slechts twee mogelijke waarden, die we in aansluiting op standaardtechnieken met "true" respectievelijk "false" aangeven. Voorts nemen wij aan, dat beide machines slechts naar 1 gemeenschappelijke variabele tegelijkertijd kunnen refereren, en dan ook alleen of om een nieuwe, waarde toe te kennen of om naar de heersende waarde te informeren. In figuur 1 is in blokschemavorm een tentatieve oplossing gegeven voor de structuur van de programma's voor beide machines. Elk blok heeft zijn ingang boven en zijn uitgang onder. Bij een blok met een dubbele uitgang wordt de keuze bepaald door de waarde van de zojuist aangevraagde logische variabele: heeft deze de waarde true, dan is de uitgang rechtsonder bedoeld, heeft deze de waarde false, dan kiezen we de uitgang linksonder.

Er komen in dit schema twee gemeenschappelijke logische variabelen voor, LA en LB. LA betekent: machine A is in zijn critische sectie, LB betekent: machine B is in zijn critische sectie. De schema's in fig 1 zijn duidelijk. In het blok bovenaan wacht machine A indien hij daar aankomt op een ogenblik, dat machine B in sectie TB bezig is, totdat machine B zijn critische traject heeft verlaten, wat door de toekenning "LB:= false" gemarkeerd wordt, waardoor dan de wacht van machine A wordt opgeheven. En omgekeerd. De schema's mogen dan eenvoudig zijn, ze zijn helaas ook fout, want ze zijn wat te optimistisch: ze sluiten nl. niet uit, dat beide machines tegelijkertijd in hun respectievelijke critische trajecten terecht komen. Als, beide machines buiten hun critische traject zijn —zeg ergens in het blanco gelaten blok— dan zijn zowel LA als LB false. Komen ze nu tegelijkertijd in hun bovenste blok, dan vinden ze beide, dat de andere machine hen geen strobreed in de weg legt, en ze gaan beide door en komen tegelijkertijd in hun critische sectie.

We zijn dus wat te optimistisch geweest. De fout is geweest, dat de ene machine niet wist, of de ander al naar zijn staat van vordering informeerde. De schema's van fig.2 maken een veel betrouwbaardere indruk en het is gemakkelijk te verifiëren, dat zij volslagen veilig zijn. Bv. machine A kan slechts aan traject TA beginnen, nadat tijdens het true-zijn van LA geconstateerd is, dat LB false is, en ongeacht hoe snel na deze kennis van machine A dit feit weer obsoleet wordt, doordat machine B gauw zijn bovenste blok "LB:= true" uitvoert, machine A kan dan veilig traject TA ingaan, omdat machine B toch netjes opgehouden wordt totdat LA na afloop van traject TA op false gezet wordt. Deze oplossing is dus volkomen veilig, maar we moeten niet denken, dat we hiermee het probleem opgelost hebben, want deze oplossing is te veilig, er bestaat nl. de kans, dat het hele samenspel van deze machines vastloopt. Als zij tegelijkertijd het bovenste blok van hun schema uitvoeren, dan lopen beide machines in de daarop volgende wacht en blijven tot in eeuwigheid van dagen beleefd voor dezelfde deur staan, zeggende "Na U" ,"Na U". We zijn te pessimistisch geweest.

U ziet, dat het probleem niet triviaal is. Het was mij althans, na deze twee probeersels, helemaal niet zonneklaar, dat er een veilige oplossing bestond, die niet tevens de mogelijkheid van een doodlopende weg in zich hield. Ik heb het probleem toen in deze vorm aan mijn toenmalige collegae van de Rekenafdeling van het Mathematisch Centrum voorgelegd, er bij vertellend, dat ik niet wist, of het oplosbaar was. Aan Dr. T.J. Dekker komt de eer toe als eerste een oplossing gevonden te hebben, die bovendien symmetrisch in de beide machines was. Deze oplossing is weergegeven in fig. 3. Er is een derde logische variabele ingevoerd, nl. AP, die betekent, dat in geval van twijfel machine A prioriteit heeft. Door de asymmetrische betekenis van deze logische variabele zijn gedeelten van deze programma's min of meer elkaars spiegelbeeld, functioneel is de oplossing echter helemaal symmetrisch en is niet de ene machine bevoorrecht boven de ander. Om te bewijzen, dat deze oplossing aan de gestelde eisen voldoet, constateren we eerst, dat de trajecten TA en TB voorafgegaan worden door de sluis, die we in figuur 2 al als veilig herkend hebben. Simultane uitvoering is dus onmogelijk. We hoeven slechts te verifiëren, dat het nu bovendien uitgesloten is, dat beide machines voor hun critische sectie blijven wachten. In deze wachtcycli wordt de waarde van AP niet gewijzigd. Beperken we ons tot het geval, dat AP true is, dan kan machine A slechts blijven cyclen als ook LB true is; onder de voorwaarde AP true kan machine B echter alleen rond blijven lopen in de cyclus, die LB op false zet, dwz. machine A uit zijn cyclus forceert. In het geval AP false verloopt de analyse op dezelfde manier. Omdat AP als logische variabele of true of false is, is daarmee de mogelijkheid van eeuwig op elkaar blijyen wachten eveneens uitgesloten. Q.E.D.

Dekker's oplossing dateert uit 1959 en bijna drie jaar heeft deze oplossing voortgeleefd als "curiositeit", totdat dit soort problematiek aan het begin van 1962 voor mij plotseling weer actueel werd en Dekker's oplossing als uitgangspunt van mijn volgende pogingen gefungeerd heeft.

De probleemstelling was inmiddels wel drastisch veranderd. In 1959 was het de vraag of de beschreven communicatiemogelijkheid tussen twee machines het mogelijk maakte om twee machines zodanig te koppelen, dat de uitvoeringen van de critische trajecten elkaar in de tijd wederzijds uitsloten. In 1962 werd een veel wijdere groep problemen bekeken en werd bovendien de vraagstelling veranderd in "Welke communicatiemogelijkheden tussen machines zijn gewenst, om dit soort spelletjes zo sierlijk mogelijk te spelen?"

Dekker's oplossing heeft op verschillende manieren als uitgangspunt gefungeerd. Omdat het in deze oplossing onduidelijk is, hoe men te werk moet gaan, wanneer het aantal machines uitgebreid wordt, en de opgave blijft om te garanderen, dat er op elk moment hoogstens 1 critische sectie onder behandeling is, was zij een aansporing om naar andere wegen te zoeken. Door een analyse van de moeilijkheden, die Dekker moest overwinnen, konden wij een aanwijzing krijgen wat een hoopvollere weg zou zijn om in te slaan.

De moeilijkheden ontstaan, doordat als een van de machines geinformeerd heeft naar de waarde van een gemeenschappelijke logische variabele, deze informatie onmiddellijk daarna al weer obsoleet kan zijn, nog voordat de informerende machine er effectief op gereageerd heeft, in het bijzonder: nog voordat de informerende machine aan zijn partners heeft kunnen meedelen, dat hij van de waarde van een gemeenschappelijke variabele. "een momentopname gemaakt heeft". De hint, die uit deze bespiegeling gehaald kan worden, is dat men moet zoeken naar machtigere "elementaire" operaties op de gemeenschappelijke variabelen, in het bijzonder naar operaties, waarin voor de informatietransport tussen machine en gemeenschappelijk geheugen niet meer de beperking van het éénrichtingverkeer geldt.

De meestbelovende operatie, die we hebben kunnen bedenken, is de operatie "falsify". Deze informeert naar de waarde van een logische variabele en het feit, dat deze operatie is uitgevoerd wordt in de helft van de gevallen kenbaar gemaakt: de operatie falsify laat de logische variabele, waarop hij geopereerd heeft nl. met de waarde "false" achter.

De operatie "falsify" kan als volgt gedefinieerd worden:

"boolean procedure falsify(S); boolean S;

begin boolean Q; falsify:= Q:= S; if Q then S:= false end" ,

mits we hierbij vermelden, dat de operatie falsify, in weerwil van zijn sequentiële definitie, beschouwd moet worden als elementaire opdracht van het repertoire der machines, waarbij "elementair" in dit geval betekent, dat gedurende de operatie falsify door één van de machines de logische variabele in kwestie ontoegankelijk is voor alle andere machines.

Dat deze operatie een stap in de goede richting is, volgt wel uit het feit, dat we nu meteen de oplossing hebben voor een wolkje machines X1, X2, X3, ..... , elk met zijn critisch traject TXi, die elkaar nu alle in de tijd moeten uitsluiten. We kunnen dit spelen met 1 gemeenschappelijke logische variabele, zeg SX die aangeeft, dat geen van de machines in zijn critische traject bezig is. De programma's vertonen nu alle dezelfde structuur:

"LXi: if non falsify(SX) then goto LXi; TXi; SX:= true; proces Xi; goto LXi"

waarbij we de aandacht er op vestigen, dat in de programma's voor de aparte machines Xi nergens tot uiting komt, hoeveel machines Xi er eigenlijk zijn: we kunnen dus door loutere toevoeging het wolkje machines Xi uitbreiden.

Het geprogrammeerde wachtcyclusje, dat hierin voorkomt, is natuurlijk heel aardig, maar het beantwoordt wat weinig aan ons doel. Een wachtcyclusje is immers de manier om een machine "zonder effect" bezig te houden. Maar als we nu bedenken, dat het merendeel van deze machines door een centrale computer gesimuleerd zal worden, zodat elke actie in de ene machine slechts plaats kan vinden ten koste van de effectieve snelheid van de andere machines, dan is het wel erg begrotelijk om in een wachtcyclusje aandacht van de centrale computer op te gaan eisen voor iets volslagen nutteloos. Zolang het wachtcyclusje niet verlaten kan worden, mag wat ons betreft de snelheid van deze wachtende machine tot nul zakken, Om dit tot uitdrukking te brengen voeren we in plaats van het wachtcyclusje een statement in, een elementaire opdracht van het repertoire van de machines, die heel heel lang kan duren. We geven dit aan met een P (van Passering); vooruitlopend op latere behoeften representeren we de statement "SX:= true" door "V(SX) -met de V van Vrijgave. (Deze terminologie is ontleend aan het spoorwegwezen: in een eerder stadium heetten de gemeenschappelijke logische variabelen "Seinpalen" en als hun naam met een S begint, dan is dat nog een reminiscentie daaraan.) De text van de programma's luidt in deze nieuwe notatie:

"LXi: P(SX); TXi; V(SX); proces Xi; goto LXi" .

De operatie P is een tentatieve passering. Het is een operatie, die slechts beëindigd kan worden op een moment, dat de er bij opgegeven logische variabele de waarde true heeft, beindiging van de operatie P houdt tevens in, dat aan de logische variabele in kwestie weer de waarde false wordt toegekend. Operaties P kunnen wel simultaan plaatsvinden, het beindigen van een P-operatie wordt echter weer beschouwd als een ondeelbare gebeurtenis.

De invoering van de operatie V is meer dan een verkorte schrijfwijze invoeren. Immers: toen wij het wachtcyclusje met behulp van de operatie falsify nog hadden, rustte de detectieplicht ten aanzien van het gelijk true worden van de gemeenschappelijke logische variabelen op de individuele machines, die op deze gebeurtenis stonden te wachten en in die zin was het wachten een tamelijk actieve bezigheid op een betrekkelijk neutrale gebeurtenis. Met de invoering van de P-operatie hebben wij zo ongeveer bereikt, dat wij tegen een machine, die voorlopig niet verder kan, zeggen: "Ga maar slapen, we maken je wel weer wakker, als je verder mag." Maar dit betekent, dat het op true zetten van een gemeenschappelijke logische variabele, waarop misschien een machine staat te wachten, niet meer een neutrale gebeurtenis is als ieder andere waardetoekenning: hier kan nl. het neveneffect aan verbonden moeten worden, dat een van de "slapende" machines "gewekt" moet worden. Het betekent, dat de logische variabelen, die gedurende false-zijn één of meer machines op kunnen houden, niet permanent geobserveerd hoeven te worden, mits bij overgang naar de waarde true een "centrale wekker" op deze overgang attent gemaakt kan worden. Als de centrale wekker per gemeenschappelijke logische variabele een administratie bij houdt, welke machines op elke logische variabele op het ogenblik wachten, dan kan de centrale wekker, mits deze expliciet op het true worden van een dergelijke gemeenschappelijke logische variabele geattendeerd wordt, snel decideren, of er op grond van deze overgang een slapende machine gewekt kan worden en zo ja welke. De operatie V kan nu opgevat worden als een combinatie van het toekennen van de waarde true aan een seinpaal, plus het attenderen van de centrale wekker op deze gebeurtenis. Het is duidelijk, dat deze centrale wekker zijn werk niet kan doen, als we toestaan, dat de gemeenschappelijke logische variabelen ook nog, door normale waardetoekenningen (van het neutrale type "S:= true"), "stiekem", ondershands gewijzigd kunnen worden. Om dit verbod te onderstrepen, noem ik de gemeenschappelijke logische variabelen van nu af aan "seinpalen" en stel ik, dat de enige mogelijkheid, waarop een machine met een bestaande seinpaal zal kunnen communiceren, zal zijn de P-operatie en de V-operatie.

En hiermede is de operatie "falsify", die ons door de aantrekkelijkheid van zijn eenvoud bij de begripsvorming zo behulpzaam is geweest, weer van het toneel verdwenen en de vraag is gerechtvaardigd, of we door de introductie van de seinpaal, die slechts via de operaties P en V toegankelijk is, het kind niet met het badwater hebben weggegooid. Dit is gelukkig niet het geval. Door de operatie "falsify" van het elementaire repertoire van de onderscheiden machines af te voeren en daar de operaties P en V voor in de plaats te geven, hebben wij niets verloren: wil de invoering van de operaties P en V niet een lege geste zijn, dan moet er dus minstens 1 seinpaal zijn, waarop zij kunnen werken. Deze ene seinpaal —noem hem "SG", de Seinpaal Generaal— is echter al voldoende om de operatie falsify op een willekeurige gemeenschappelijke logische variabele toe te voegen aan het repertoire van alle machines. Immers:

Als wij in elke machine definiëren:

"boolean procedure falsify(GB);

boolean GB;

begin boolean Q;

P(SG); falsify:= Q:= GB;

if Q then GB:= false; V(SG)

end"

en wij voorts afspreken, dat elke referentie naar een gemeenschappelijke logische variabele in elke machine ingekapseld zal worden tussen de statements "P(SG)" en "V(SG)" (zodat de assignment "GB1:= true" nu de vorm zal hebben "P(SG); GB1:= true; V(SG)", dan hebben wij hiermee verzekerd dat aan de bijvoorwaarde voldaan is, dat "gedurende de operatie falsify door één van de machines de logische variabele in kwestie ontoegankelijk is voor alle andere machines." We hebben met de ruil dus niet in eigen vlees gesneden.

Wij gaan nu laten zien, dat de operaties P en V in hun toepassingsgebied niet beperkt zijn tot wederzijdse uitsluiting, maar dat we ze ook kunnen gebruiken, om twee machines ten opzichte van elkaar te synchroniseren. Stel, dat wij twee cyclische machines hebben, zeg X en Y, die elk in hun cyclus 'n critische sectie hebben, zeg TX resp. TY, waarbij de volgende eisen gesteld zijn:

1) de uitvoering van de critische sectie TX mag in de tijd niet samenvallen met die van TY;

2) de uitvoeringen van de secties TX en TY moeten alternerende gebeurtenissen zijn (op deze voorwaarde doelden we, toen we spraken van "onderlinge synchronisatie").

Als U zich een entourage wilt voorstellen, waarin men voor dit probleem gesteld wordt, denkt U dan maar aan het volgende. Proces X is een cijferproducerend proces (een rekenproces), proces Y is een cijferconsumerend proces (een schrijfmachine) en deze twee processen zijn gekoppeld door een "toonbank" met de capaciteit van één cijfer. De handeling TX is het neerleggen van het volgende te typen cijfer op een lege toonbank, het stuk TY representeert het afnemen van het cijfer van de volle toonbank. De stricte alternering vloeit voort uit de eis dat enerzijds geen enkel aan de toonbank aangeboden cijfer "weg mag raken" maar inderdaad getypt moet worden, en dat anderzijds -als er een tijdje geen aanbod is- de schrijfmachine niet zijn tijd moet gaan vullen met het nog maar eens typen van het laatste cijfer.

Wij kunnen dit bereiken met twee seinpalen, zeg SX en SY, waarbij: SX betekent, dat nu eerst een sectie TX aan de beurt is, en SY betekent, dat er nu eerst een sectie TY aan de beurt is. Voor machines X en Y luiden de programma's nu:

"LX: P(SX); TX; V(SY); proces X; goto LX"        en

"LY: P(SY); TY; V(SX); proces Y; goto LY"    .

Wij kunnen dit direct op twee verschillende manieren uitbreiden. Indien wij de machine X vervangen door een wolkje machines Xi, die allemaal van dezelfde toonbank gebruik willen maken om hun productie te lozen, en de machine Y vervangen door een wolkje machines, die allemaal bereid zijn, om informatie van de toonbank af te nemen (ongeacht, door welke van de machines Xi het er op is gelegd, het beeld van de schrijfmachines gaat hier dus niet meer op), dan luiden de programma's voor machines Xi en Yj analoog aan de boven gegeven programma's:

"LXi: P(SX); TXi; V(SY); proces Xi; goto LXi"     en

"LYj: P(SY); TYj; V(SX); proces Yj; goto LYj"    .

Hadden we ook nog een groepje machines Zk gehad, en was de opgave geweest, dat de uitvoering van een TX-sectie, een TY-sectie en een TZ-sectie elkaar in die volgorde cyclisch moesten opvolgen, dan hadden de programma's geluid:

"LXi: P(SX); TXi; V(SY); proces Xi; goto LXi"     en

"LYj: P(SY); TYj; V(SZ); proces Yj; goto LYj"     en

"LZk: P(SZ); TZk; V(SX); proces Zk; goto LZk"   .

Wij merken hierbij op, dat in de laatste twee voorbeeldjes de machines Xi gelijk zijn: zij representeren uitsluitend, dat de handelingen TX ergens door geïnterspatiëerd moeten worden, zij laten in het midden, hoe complex deze tussenliggende handelingen gerealiseerd zullen worden.

Een volgende opgave moge aantonen, dat onze operaties P en V in hun huidige vorm nog niet helemaal machtig genoeg zijn. Wij beschouwen daartoe de volgende configuratie. We hebben twee machines A en B, elk met hun critische sectie TA respectievelijk TB; deze secties zijn volledig onafhankelijk van elkaar, maar er is nog een derde machine C in het spel met zijn critische sectie TC en de uitvoering van TC mag in de tijd niet samen vallen met die van TA, noch met die van TB. We kunnen dit doen met twee seinpalen zeg: SA en SB. Voor machines A en B is het programma eenvoudig, nl.

"LA: P(SA); TA; V(SA); proces A; goto LA"      en

"LB: P(SB); TB; V(SB); proces B; goto LB" ,

maar voor machine C is het volgende programma. hoewel veilig, onacceptabel:

"LC: P(SA); P(SB); TC; V(SB); V(SA); proces C; goto LC" .

Immers: machine C zou de statement P(SA) succesvol voltooid kunnen hebben op een moment, dat machine B in sectie TB bezig is. Zolang sectie TB nu nog niet voltooid is, kan machine C niet verder —dat is correct— maar ook machine A kan zijn sectie TA niet meer uitvoeren en dat is niet correct: indirect, via machine C zijn hier de secties TA en TB toch weer gekoppeld. Door de operatie P(SA) van machine C is de seinpaal SA overhaast op false gezet, daarmee machine A onnodig hinderend.

Wij gaan daarom de operaties P en V uitbreiden tot operaties, die we een willekeurig aantal argumenten mee kunnen geven —daarmee de grenzen van ALGOL 60 overschrijdend, maar dat doet er in dit verband bitter weinig toe. Voor de V-operatie is de betekenis duidelijk: het is niet anders dan de simultaneous assignment, die aan alle meegegeven seinpalen gelijktijdig de waarde true toekent. De P-operatie met een aantal argumenten, bv. "P(S1, S2, S3)", is een operatie, die slechts beëindigd kan worden op een ogenblik, dat alle meegegeven seinpalen de waarde true hebben (in ons voorbeeld dus een ogenblik, waarop geldt: "S1 and S2 and S3"), beindiging van een P-operatie houdt dan in, dat aan alle meegegeven seinpalen simultaan de waarde false toegekend wordt. Ook na deze uitbreiding wordt de beindiging van een P-operatie weer beschouwd als een ondeelbare gebeurtenis.

Met deze uitbreiding zijn we in staat machine C zodanig te definiëren, dat het gesignaleerde bezwaar niet meer optreedt:

"LC: P(SA, SB); TC; V(SA; SB); proces C; goto LC".

Tenslotte wil ik, hoewel dit beslist niet de toepassing is, waarvoor de P- en V-operaties geconcipieerd zijn, laten zien, hoe we hiermee het scalair product van twee vectoren kunnen berekenen, zonder dat we ons vast leggen op de volgorde, waarin de producten der elementen bij elkaar opgeteld worden.

Stel, dat we uit willen rekenen de som SIGMA(i,1,5,A[i] * B[i]) —in woorden: de som, voor i lopend van 1 t/m 5, van (A[i] * B[i])—;

stel, dat ingevoerd zijn de 7 seinpalen: Ssom, Sklaar en de vijf seinpalen Sterm[i] met i=1,2,3,4 en 5; alle 7 met de beginwaarde false;

stel, dat inmiddels gecreerd zijn vijf machines van de structuur (i=1,2,3,4,5):

"Lterm[i]: P(Ssom, Sterm[i]);

scapro:= scapro + A[i] * B[i];

n:= n - 1;

if n = 0 then V(Sklaar) else V(Ssom);

goto Lterm[i] " ,

alle bezig aan hun —enige— P-operatie;

Beschouw nu een zesde machine, die onder bovengenoemde voorwaarden juist zal beginnen aan de volgende statements:

".....; n:= 5;

scapro:= 0;

V(Ssom, Sterm[1], Sterm[2], Sterm[3], Sterm[4], Sterm[5]);

"programma, dat genoemde seinpalen, scalairen en vectorelementen
ongemoeid laat";

P(Sklaar);...."

Als de laatste P-operatie van de zesde machine voltooid is, heeft n de waarde 0, scapro heeft de waarde van de gevraagde som en alle seinpalen en de vijf eerstgenoemde machines zijn terug in hun oorspronkelijke toestand.

transcribed by Gerrit Jan Veltink

revised Mon, 22 Mar 2010

---

## English translation

### On the sequentiality of process descriptions

It is not unusual for a speaker to begin his lecture with an introduction. Because I am here possibly addressing an audience that, as a whole, is less familiar with the set of problems I wish to broach and the terminology I shall have to use, I should, in this case, like to give two introductions by way of preface, viz. one to paint the background of the problems and a second one to give you a feeling for the nature of the logical problems that we shall run up against.

My first introduction is a historical one. First I must tell you what you are to picture when one speaks of a sequential process description. You all know such process descriptions. Consider, for example, the description of the construction of the altitude from the vertex C of the triangle ABC. It can read as follows:

1) draw the circle with centre A and radius = AC;

2) draw the circle with centre B and radius = BC;

3) draw the straight line determined by the intersection points of the circles named under 1) and 2).

This is a description for the benefit of someone whose "ready knowledge" does not encompass the construction of the altitude; for his benefit the construction of the altitude is built up out of a number of operations from a more limited repertoire, operations which he can carry out in the prescribed order, one after the other, i.e. sequentially. It will not have escaped the attentive little reader that we have hereby specified more than is strictly necessary: it is indeed essential that operation 3) be carried out only after operations 1) and 2) have been completed, operations 1) and 2) might perfectly well have taken place in the reverse order, more strongly, a draughtsman who can operate two pairs of compasses at the same time, so to speak one with each hand, may carry out operations 1) and 2) simultaneously. But this freedom has, in our process description, vanished entirely under the table.

Now a more numerical example, one that connects somewhat more closely with the world of thought in which the problems to be treated presently have become topical. Given the values of the variables a, b, c and d, it is required to compute the value of the variable x, determined by the formula "x:=(a+b)*(c+d)". When the carrying out of this value assignment does not belong to the primitive repertoire of the calculator —and you may by now be thinking of the computing machine— then this computation rule must be built up out of more elementary steps, e.g.:

1) compute t1:= a + b;

2) compute t2:= c + d ,

3) compute x:= t1 * t2.

This computation rule contains precisely the same overdetermination as our altitude construction: the order of the first two steps is immaterial, they could even be carried out simultaneously.

The above process description is representative of the mode of operation of the more classical computing automata. The repertoire of elementary operations is, because for each operation special technical provisions are required, finite and, for financial reasons, even very limited; these machines owe their enormous flexibility to the fact that arbitrarily complicated algebraic expressions such as the one illustrated above can be worked out by a sequence of such elementary operations. Inherent in this sequential description of the process is that one specifies more than is strictly necessary. For a long time this caused no trouble, but in recent years a change has come about, and indeed through two causes: a change in the structure of machines and the advent of a new class of problems for which one would like to put these machines to use.

The classical sequential machine does "one thing at a time" and performs one function after the other. For the sake of efficiency one has come to build more specific apparatus for specific functions, but if one then holds fast to the requirement that the various operations are to be carried out strictly sequentially, then there suddenly forces itself upon one the image of a machine as an interplay of N organs, of which at each discrete instant only 1 or 2 are working. If N is large, this means that the greater part of your machine stands idle for the greater part of the time, and that is evidently not the intention.

The second cause is the advent of a new category of problems; originally computing automata were used only for processes fully defined in advance, and the machine could then carry out the process on its own, at its own pace. As soon, however, as one wants to have the information-processing process act in concert with some other process —e.g. a chemical process that is to be controlled by the computing automaton— then quite different problems arise. Tying back in to our example of the computation of x:=(a + b)*(c + d), we are then no longer in the situation that it does not matter which sum is formed first. If the data a, b, c and d are supplied at unknown moments by the environment of the computing automaton, and it is the machine's task to deliver the value of x as quickly as possible, then this means that the automaton must just go ahead and carry out a partial computation as soon as the data required for it have come in: which addition one then wants carried out first will depend on the order in which the data a, b, c and d become available. This was my first introduction, and I hope I have thereby given you an idea of what you are to picture when one speaks of a sequential process description, and of why the question has been raised of less strictly sequential process descriptions.

Now my second introduction, which is intended to give you, from the very outset, some idea of the logical problems that we shall bring down upon ourselves by abandoning strict sequentiality.

Just as in an ordinary piece of algebra variables are denoted by letters, in general "names", so in the process of information processing one has the continual duty of identification, the duty to find, given the name, the object thereby designated. Under some circumstances one would gladly have at one's disposal the selection process that the schoolmistress in the class has at her disposal when she says "Johnny, come up to the board." Assuming that the class as a whole is paying attention and is obedient, this process works flawlessly under the assumption that there is 1 and only 1 Johnny in the class. The peculiar thing about this selection mechanism is that it works regardless of the number of children in the class. This idyll is, alas, technically not realizable, or at most realizable in a defective way, and we obtain a statement of the problem that is more in keeping with reality when the children do not respond spontaneously to the calling of their name and the schoolmistress, when she wants Johnny at the board, is compelled to take Johnny by the ear and drag him up to the board by it. Now if this is an old-fashioned school in which every child has its fixed place in the class, then the schoolmistress, if she has put the children in their places at the beginning of the year, can rechristen them for her own convenience, and replace the name "Johnny" —which is not very helpful for finding the boy— by an identification which, by virtue of its structure, immediately defines which ear she is to grab: she can call the boy "third row, second desk". This is essentially the technique that is followed in classical automata in classical applications. In more modern applications one can no longer do this, and one must, for example, be equal to the problems that the schoolmistress gets if the children may choose where they sit.

So as not to make it impossible for the schoolmistress, we assume that each of the unwilling children has pinned on a name tag. If the schoolmistress now wants Johnny at the board, she must therefore look Johnny up. The schoolmistress works sequentially, can read only one name tag at a time, and the schoolmistress works systematically: she has, for herself, ordered the desks uniquely in some manner or other, and when she wants Johnny, she goes down the desks in this order, repeatedly asking herself "Are you Johnny?", if not, then she goes to the next desk. This process works flawlessly, provided we can guarantee that the children do not change seats during the schoolmistress's round, provided we can guarantee that the two processes "child localization" and "child permutation" are not carried out simultaneously. For you will surely understand that Johnny, as soon as he realizes that he is the figure being sought, goes and sits at the back of the class, and as soon as the schoolmistress has searched the front half of the class, he quickly jumps to the front. We can refine the schoolmistress and have her, when she has searched the whole class and not found Johnny, start again from the beginning with fresh courage, but this is no solution, for Johnny then quickly jumps to the back again. Now you might remark that if those permutations were carried out not with so much malice, but merely at random, then the expected time for localizing Johnny would remain finite, indeed would not even have grown larger as a result of his freedom to jump about. This remark, however hopeful, brings no relief. Those who have delved into the "Theory of computability" know how much weight is attached to a finite, i.e. a guaranteed terminating algorithm, and they will understand that a process which potentially keeps spinning on infinitely long is not an attractive element. Much worse is that in practice it is usually questionable whether we may regard the permutations of the children as random at all. If there is a finite probability that, once the schoolmistress has gone through the class one time, the children are seated again exactly as they were when the schoolmistress began searching, then this might very well be the first period of a purely periodic phenomenon.

But even if we accept the risk that we shall never find Johnny, her misery is still beyond surveying, for besides the risk that she does not find Johnny, she runs the much greater danger that she drags the wrong child up to the board, viz. if the children, between the moment at which she has with satisfaction concluded "So, you then are Johnny" and the moment at which she, on the basis of this conclusion, has reached out her hand to the corresponding ear, quickly change places once more. In an information-processing process this would amount to a disaster. And this is the end of my second introduction, of which I hope it has given you some idea of the kind of problem that we shall encounter.

At the end of my first introduction I named two causes that were responsible for a growing interest in less strictly sequential process descriptions. I touched on them only lightly, and it may not have struck you that they were drawn from almost opposite settings. In the first case I named the increasing complexity of machines, which are now built up out of a number of more or less autonomously working components, and the task was to have these together carry out one process. In the second case, where we looked at the automaton in a so-called "real time application", the image was conjured up of a single piece of apparatus that had the possibility, upon a change in the external urgency situation, of switching over to the now most urgent of its possible tasks. These two arrangements, an interplay of a number of machines on one process and, conversely, one machine dividing its attention over a number of processes, were for a long time regarded as essentially foreign to each other. People have even invented different names for them, the one is called "parallel programming" and the other is called "multiprogramming", but you must no longer ask me which is which, for that I can no longer keep straight, ever since I discovered that the logical problems which they call forth through the non-sequentiality of the process definition are in both cases exactly the same.

In what follows I shall avail myself of the image of a number of mutually weakly coupled, in themselves sequential machines, thereby connecting with the first setting. But this is only a manner of description, for to what I shall henceforth call a machine there does not necessarily correspond a discretely identifiable piece of apparatus: in what follows my distinguished machines may be quite abstract and represent nothing more than an in itself sequential partial process. This finds expression, among other things, in the fact that I shall show a great fondness for an interplay of N machines, N arbitrary. If I used the term machine only for an identifiable functional piece of apparatus, then I might regard the number of machines for each installation as fairly constant. Now, however, that a machine, as the personification of an in itself sequential partial task, can be created by the user ad libitum, such a fixed upper bound is no longer acceptable. I am therefore going to talk about the interplay of N machines, N not only arbitrary but, if need be, even variable during the process. Superfluous machines can be destroyed, new machines can be created as needed and taken up into the interplay of the others. I hope that you will now, on my authority, be willing to accept, or else, afterwards, see for yourself, that without harming the generality we can restrict ourselves to sequential machines that carry out some cyclic process or other.

Let us begin with a very simple problem. Given two machines A and B, both engaged in a cyclic process. In the cycle of machine A there occurs a certain critical stretch, called TA, and in that of machine B there occurs a critical stretch TB. The task is to see to it that the two machines are never simultaneously each engaged in its critical stretch. (In terms of our second introduction: machine A might be the schoolmistress, stretch TA the selection process of a new pupil who must be fetched to the board, while the stretch TB represents the process "child permutation".) No assumptions may be made about the relative speeds of the machines A and B, the speed at which they work need not even be constant. It is clear that we can realize the mutual exclusion of the critical stretch only if the two machines can somehow communicate with each other. For this communication we make available some common store, i.e. a number of variables that are accessible to both machines in both directions, i.e. to which both machines can assign a value and of which both machines can inquire the prevailing value. These two operations, assigning a new value and inquiring the prevailing value, count as indivisible operations, i.e. if both machines want "at the same time" to assign a value to a common variable, then the value afterwards is either the one or the other assigned value, but not some mixture or other. Likewise: if the one machine inquires the value of a common variable at the moment that the other machine assigns a new value to it, then there penetrates to the inquiring machine either the old or the new, but not some wild value. It will turn out that for this purpose we can restrict ourselves to common logical variables, i.e. variables with only two possible values, which, in keeping with standard techniques, we denote by "true" and "false" respectively. Furthermore we assume that both machines can refer to only 1 common variable at a time, and then also only either to assign a new value or to inquire the prevailing value. In figure 1 a tentative solution is given, in block-diagram form, for the structure of the programs for both machines. Each block has its entrance at the top and its exit at the bottom. With a block having a double exit the choice is determined by the value of the logical variable just queried: if this has the value true, then the exit at the lower right is meant, if this has the value false, then we choose the exit at the lower left.

There occur in this diagram two common logical variables, LA and LB. LA means: machine A is in its critical section, LB means: machine B is in its critical section. The diagrams in fig. 1 are clear. In the topmost block machine A, if it arrives there at a moment that machine B is engaged in section TB, waits until machine B has left its critical stretch, which is marked by the assignment "LB:= false", whereby machine A's wait is then lifted. And conversely. The diagrams may be simple, but unfortunately they are also wrong, for they are somewhat too optimistic: they namely do not exclude that both machines end up in their respective critical stretches at the same time. If both machines are outside their critical stretch —say somewhere in the block left blank— then both LA and LB are false. If they now arrive at the same time in their topmost block, then both find that the other machine puts not a straw in their way, and they both go on and arrive at the same time in their critical section.

So we have been somewhat too optimistic. The mistake has been that the one machine did not know whether the other was already inquiring about its state of progress. The diagrams of fig. 2 make a much more reliable impression, and it is easy to verify that they are utterly safe. For instance, machine A can only begin stretch TA after it has been ascertained, while LA is true, that LB is false, and regardless of how quickly after this knowledge of machine A this fact becomes obsolete again, because machine B quickly carries out its topmost block "LB:= true", machine A can then safely enter stretch TA, because machine B will nevertheless be held up nicely until LA is set to false after the conclusion of stretch TA. This solution is thus entirely safe, but we must not think that we have thereby solved the problem, for this solution is too safe, there exists namely the possibility that the whole interplay of these machines comes to a standstill. If they carry out the topmost block of their diagram at the same time, then both machines run into the wait that follows it and remain, until eternity of days, politely standing before the same door, saying "After you", "After you". We have been too pessimistic.

You see that the problem is not trivial. To me, at least, after these two attempts, it was not at all crystal clear that there existed a safe solution that did not at the same time harbour within it the possibility of a dead-end road. I then laid the problem in this form before my then colleagues of the Computation Department of the Mathematical Centre, telling them that I did not know whether it was solvable. To Dr. T.J. Dekker belongs the honour of having been the first to find a solution, which moreover was symmetric in the two machines. This solution is rendered in fig. 3. A third logical variable has been introduced, viz. AP, which means that in case of doubt machine A has priority. Through the asymmetric meaning of this logical variable, portions of these programs are more or less each other's mirror image, but functionally the solution is wholly symmetric and the one machine is not privileged over the other. To prove that this solution satisfies the requirements posed, we first observe that the stretches TA and TB are preceded by the lock that we already recognized in figure 2 as safe. Simultaneous execution is thus impossible. We need only verify that it is now, moreover, excluded that both machines remain waiting before their critical section. In these waiting cycles the value of AP is not altered. If we restrict ourselves to the case that AP is true, then machine A can only keep cycling if LB too is true; under the condition AP true, however, machine B can only keep running around in the cycle that sets LB to false, i.e. forces machine A out of its cycle. In the case AP false the analysis runs in the same manner. Because AP, being a logical variable, is either true or false, the possibility of waiting on each other forever is thereby likewise excluded. Q.E.D.

Dekker's solution dates from 1959, and for nearly three years this solution lived on as a "curiosity", until this sort of problem suddenly became topical for me again at the beginning of 1962, and Dekker's solution served as the starting point of my subsequent attempts.

The statement of the problem had in the meantime changed drastically. In 1959 the question was whether the described possibility of communication between two machines made it possible to couple two machines in such a way that the executions of the critical stretches mutually excluded each other in time. In 1962 a much wider group of problems was examined, and moreover the formulation of the question was changed into "Which possibilities of communication between machines are desirable in order to play this kind of game as gracefully as possible?"

Dekker's solution served as a starting point in various ways. Because in this solution it is unclear how one is to proceed when the number of machines is enlarged, while the task remains to guarantee that at each moment at most 1 critical section is under treatment, it was a spur to seek other ways. Through an analysis of the difficulties that Dekker had to overcome, we could obtain an indication of what would be a more hopeful way to take.

The difficulties arise because, when one of the machines has inquired the value of a common logical variable, this information can immediately afterwards already be obsolete again, even before the inquiring machine has effectively reacted to it, in particular: even before the inquiring machine has been able to inform its partners that it has "made a snapshot" of the value of a common variable. The hint that can be drawn from this reflection is that one must seek more powerful "elementary" operations on the common variables, in particular operations in which, for the transport of information between machine and common store, the restriction of one-way traffic no longer holds.

The most promising operation that we have been able to devise is the operation "falsify". It inquires the value of a logical variable, and the fact that this operation has been carried out is made known in half of the cases: the operation falsify namely leaves the logical variable on which it has operated with the value "false".

The operation "falsify" can be defined as follows:

"boolean procedure falsify(S); boolean S;

begin boolean Q; falsify:= Q:= S; if Q then S:= false end" ,

provided we mention here that the operation falsify, in spite of its sequential definition, is to be regarded as an elementary instruction of the repertoire of the machines, where "elementary" in this case means that during the operation falsify by one of the machines the logical variable in question is inaccessible to all other machines.

That this operation is a step in the right direction follows clearly from the fact that we now immediately have the solution for a little cloud of machines X1, X2, X3, ....., each with its critical stretch TXi, which must now all exclude each other in time. We can play this with 1 common logical variable, say SX, which indicates that none of the machines is engaged in its critical stretch. The programs now all exhibit the same structure:

"LXi: if non falsify(SX) then goto LXi; TXi; SX:= true; proces Xi; goto LXi"

whereby we draw attention to the fact that in the programs for the separate machines Xi it nowhere finds expression how many machines Xi there actually are: we can therefore, by mere addition, enlarge the little cloud of machines Xi.

The programmed little waiting cycle that occurs in this is of course quite nice, but it answers our purpose rather poorly. A little waiting cycle is, after all, the way to keep a machine busy "without effect". But if we now consider that the greater part of these machines will be simulated by a central computer, so that every action in the one machine can only take place at the cost of the effective speed of the other machines, then it is rather costly to go and demand attention of the central computer in a little waiting cycle for something utterly useless. As long as the little waiting cycle cannot be left, the speed of this waiting machine may, as far as we are concerned, drop to zero. To bring this to expression, we introduce, in place of the little waiting cycle, a statement, an elementary instruction of the repertoire of the machines, which can last a very, very long time. We denote this with a P (for Passering [passage]); anticipating later needs we represent the statement "SX:= true" by "V(SX)" —with the V of Vrijgave [release]. (This terminology is borrowed from the railways: at an earlier stage the common logical variables were called "Seinpalen" [signal posts/semaphores], and if their name begins with an S, then that is still a reminiscence of that.) The text of the programs reads, in this new notation:

"LXi: P(SX); TXi; V(SX); proces Xi; goto LXi" .

The operation P is a tentative passage. It is an operation that can be terminated only at a moment that the logical variable specified with it has the value true; termination of the operation P also entails that the logical variable in question is again assigned the value false. Operations P can indeed take place simultaneously, but the termination of a P-operation is again regarded as an indivisible event.

The introduction of the operation V is more than introducing an abbreviated notation. For: while we still had the little waiting cycle with the help of the operation falsify, the duty of detection with respect to the becoming-true of the common logical variables rested on the individual machines that stood waiting for this event, and in that sense the waiting was a fairly active occupation upon a relatively neutral event. With the introduction of the P-operation we have more or less achieved that we say to a machine that cannot for the time being proceed further: "Just go to sleep, we'll wake you up again when you may proceed." But this means that the setting-to-true of a common logical variable, for which perhaps a machine stands waiting, is no longer a neutral event like any other value assignment: here, namely, there may have to be attached to it the side effect that one of the "sleeping" machines must be "awakened". It means that the logical variables which, while false, can hold up one or more machines, need not be permanently observed, provided that upon transition to the value true a "central wakener" can be made attentive to this transition. If the central wakener keeps, per common logical variable, a record of which machines are at the moment waiting on each logical variable, then the central wakener can, provided it is explicitly drawn attention to the becoming-true of such a common logical variable, quickly decide whether on the basis of this transition a sleeping machine can be awakened, and if so which one. The operation V can now be conceived of as a combination of the assigning of the value true to a semaphore, plus the drawing of the central wakener's attention to this event. It is clear that this central wakener cannot do its work if we permit the common logical variables also still to be modified "on the sly", under the table, by normal value assignments (of the neutral type "S:= true"). To underline this prohibition, I shall from now on call the common logical variables "semaphores", and I lay it down that the only possibility by which a machine will be able to communicate with an existing semaphore will be the P-operation and the V-operation.

And with this the operation "falsify", which through the attractiveness of its simplicity has been so helpful to us in the formation of concepts, has disappeared again from the scene, and the question is justified whether, through the introduction of the semaphore, which is accessible only via the operations P and V, we have not thrown out the child with the bathwater. This is fortunately not the case. By removing the operation "falsify" from the elementary repertoire of the distinguished machines and giving the operations P and V in its place, we have lost nothing: if the introduction of the operations P and V is not to be an empty gesture, then there must be at least 1 semaphore on which they can work. This one semaphore —call it "SG", the Semaphore General— is, however, already sufficient to add the operation falsify on an arbitrary common logical variable to the repertoire of all machines. For:

If we define in every machine:

"boolean procedure falsify(GB);

boolean GB;

begin boolean Q;

P(SG); falsify:= Q:= GB;

if Q then GB:= false; V(SG)

end"

and we further agree that every reference to a common logical variable in every machine will be encapsulated between the statements "P(SG)" and "V(SG)" (so that the assignment "GB1:= true" will now have the form "P(SG); GB1:= true; V(SG)"), then we have hereby assured that the additional condition is satisfied that "during the operation falsify by one of the machines the logical variable in question is inaccessible to all other machines." With the exchange we have thus not cut into our own flesh.

We are now going to show that the operations P and V are, in their domain of application, not restricted to mutual exclusion, but that we can also use them to synchronize two machines with respect to each other. Suppose that we have two cyclic machines, say X and Y, which each have in their cycle a critical section, say TX and TY respectively, on which the following requirements are imposed:

1) the execution of the critical section TX may not coincide in time with that of TY;

2) the executions of the sections TX and TY must be alternating events (it was this condition we were aiming at when we spoke of "mutual synchronization").

If you wish to picture a setting in which one is confronted with this problem, then just think of the following. Process X is a digit-producing process (a computation process), process Y is a digit-consuming process (a typewriter), and these two processes are coupled by a "counter" with the capacity of one digit. The operation TX is the laying down of the next digit to be typed on an empty counter, the piece TY represents the taking of the digit off the full counter. The strict alternation flows from the requirement that, on the one hand, no digit offered to the counter "may be lost" but must indeed be typed, and that, on the other hand —if for a while there is no supply— the typewriter must not go and fill its time with typing the last digit yet again.

We can achieve this with two semaphores, say SX and SY, where: SX means that now first a section TX is in turn, and SY means that now first a section TY is in turn. For machines X and Y the programs now read:

"LX: P(SX); TX; V(SY); proces X; goto LX"        en [and]

"LY: P(SY); TY; V(SX); proces Y; goto LY"    .

We can extend this directly in two different ways. If we replace the machine X by a little cloud of machines Xi, which all want to make use of the same counter to discharge their production, and replace the machine Y by a little cloud of machines, which are all prepared to take information off the counter (regardless of which of the machines Xi laid it on, so the image of the typewriters no longer holds here), then the programs for machines Xi and Yj read, analogously to the programs given above:

"LXi: P(SX); TXi; V(SY); proces Xi; goto LXi"     en [and]

"LYj: P(SY); TYj; V(SX); proces Yj; goto LYj"    .

Had we also had a little group of machines Zk, and had the task been that the execution of a TX-section, a TY-section and a TZ-section must follow each other cyclically in that order, then the programs would have read:

"LXi: P(SX); TXi; V(SY); proces Xi; goto LXi"     en [and]

"LYj: P(SY); TYj; V(SZ); proces Yj; goto LYj"     en [and]

"LZk: P(SZ); TZk; V(SX); proces Zk; goto LZk"   .

We remark here that in the last two little examples the machines Xi are identical: they represent exclusively that the operations TX must be interspaced somewhere by something, they leave open how complex these intervening operations are to be realized.

A next task may show that our operations P and V, in their present form, are not yet quite powerful enough. To this end we consider the following configuration. We have two machines A and B, each with their critical section TA and TB respectively; these sections are completely independent of each other, but there is still a third machine C in play with its critical section TC, and the execution of TC may not coincide in time with that of TA, nor with that of TB. We can do this with two semaphores, say: SA and SB. For machines A and B the program is simple, viz.

"LA: P(SA); TA; V(SA); proces A; goto LA"      en [and]

"LB: P(SB); TB; V(SB); proces B; goto LB" ,

but for machine C the following program, although safe, is unacceptable:

"LC: P(SA); P(SB); TC; V(SB); V(SA); proces C; goto LC" .

For: machine C might have successfully completed the statement P(SA) at a moment that machine B is engaged in section TB. As long as section TB has not yet been completed, machine C cannot proceed —that is correct— but machine A too can no longer execute its section TA, and that is not correct: indirectly, via machine C, the sections TA and TB are here coupled again after all. Through the operation P(SA) of machine C the semaphore SA has been set to false prematurely, thereby needlessly hindering machine A.

We are therefore going to extend the operations P and V into operations to which we can give an arbitrary number of arguments —thereby overstepping the bounds of ALGOL 60, but that matters bitterly little in this connection. For the V-operation the meaning is clear: it is nothing other than the simultaneous assignment that assigns to all the semaphores given with it the value true at the same time. The P-operation with a number of arguments, e.g. "P(S1, S2, S3)", is an operation that can be terminated only at a moment that all the semaphores given with it have the value true (in our example, then, a moment at which there holds: "S1 and S2 and S3"); termination of a P-operation then entails that to all the semaphores given with it the value false is assigned simultaneously. Even after this extension the termination of a P-operation is again regarded as an indivisible event.

With this extension we are in a position to define machine C in such a way that the objection signalled no longer arises:

"LC: P(SA, SB); TC; V(SA; SB); proces C; goto LC".

Finally I should like, although this is decidedly not the application for which the P- and V-operations were conceived, to show how we can with them compute the scalar product of two vectors without committing ourselves to the order in which the products of the elements are added together.

Suppose that we want to compute the sum SIGMA(i,1,5,A[i] * B[i]) —in words: the sum, for i running from 1 through 5, of (A[i] * B[i])—;

suppose that the 7 semaphores have been introduced: Ssom, Sklaar [Ssum, Sdone] and the five semaphores Sterm[i] with i=1,2,3,4 and 5; all 7 with the initial value false;

suppose that in the meantime five machines have been created of the structure (i=1,2,3,4,5):

"Lterm[i]: P(Ssom, Sterm[i]);

scapro:= scapro + A[i] * B[i];

n:= n - 1;

if n = 0 then V(Sklaar) else V(Ssom);

goto Lterm[i] " ,

all engaged in their —only— P-operation;

Now consider a sixth machine which, under the conditions named above, is just about to begin the following statements:

".....; n:= 5;

scapro:= 0;

V(Ssom, Sterm[1], Sterm[2], Sterm[3], Sterm[4], Sterm[5]);

"program that leaves the named semaphores, scalars and vector elements
undisturbed";

P(Sklaar);...."

When the last P-operation of the sixth machine has been completed, n has the value 0, scapro has the value of the requested sum, and all semaphores and the five first-named machines are back in their original state.

transcribed by Gerrit Jan Veltink

revised Mon, 22 Mar 2010
