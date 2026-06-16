---
title: "Over paginaadministratie (EWD84)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD84.html
published: "1964-04-02T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD84.PDF
---

# Over paginaadministratie

EWD 84

15 mei 1964

Over paginaadministratie.

0. Enige terminologie.

Om spraakverwarring te voorkomen, voeren we in de begrippen "pagina" en "bladzijde". Een bladzijde is een hoeveelheid informatie van 512 woorden, een pagina is een stuk van een van de geheugens, bestemd om, indien gebruikt, een bladzijde te herbergen. Een pagina beslaat dus 512 woorden. De bladzijde is dus de informatiehoeveelheid, die in een pagina "past".

In de programma's zijn het de bladzijden, die een duidelijke identiteit hebben. De text van het objectprogramma wordt in bladzijden onderverdeeld, bladzijden, wier bestaan begint in de loop van het compilatieproces en eindigt na voltooiing van het programma. Grote arrays zullen hun elementen eveneens in bladzijden ingedeeld krijgen, in principe beginnen deze bladzijden hun bestaan bij de uitvoering van de arraydeclaratie, ze eindigen hun bestaan bij de definitieve verlating van het blok in kwestie.

De stapel wordt enigzins anders behandeld; deze zal wel steeds bestaan uit een aantal bladzijden, maar om de elementen van deze bladzijden te adresseren zal een ander mechanisme geschapen worden dan het mechanisme, nodig om een element te selecteren uit een algemene bladzijde (programmatext of groot array). De elementen in de stapel zullen nl. tijdens hun levensduur slechts op een vaste plaats in de kernen kunnen staan, zodat we deze elementen gedurende hun bestaan kunnen karakteriseren door hun physisch kernenadres.

Elke bladzijde (althans de non-stapelbladzijden) ontleent zijn identiteit aan een hem toegevoegde grootheid, een zg. BZV (BladZijde Variabele). Doorgaans zijn de BZV's gesitueerd in de stapel van het programma, waarin deze bladzijden voorkomen. (Dit geldt ten duidelijkste voor de bladzijden, die de programmatext uitmaken; de bijbehorende BZV's kunnen we beschouwen als globale variabelen van dit programma. Dit geldt ook voor de BZV's voor array-bladzijden; deze zijn lokale grootheden van het blok, waarin het array gedeclareerd is.) Er komen in het samenspel ook andere bladzijden voor, bv. de bladzijden, die de bibliotheek van standaardprocedures, die permanent op de trommel aanwezig geacht wordt; deze kunnen kennelijk niet exclusief in de stapel van 1 programma ondergebracht worden. Immers, de bibliotheek is een gedeelte van het universum, waarin de verschillende programma's samen in ingebed worden, de bibliotheek overtreft, omvat, in levensduur ook de individuele programma's. Dit vindt zijn neerslag daarin, dat er naast wat de programma's prive nodig hebben, op een vaste plaats in het kerngeheugen een rijtje BZV's ondergebracht zal worden, de "bibliotheek BZV's", waartoe ieder individueel programma, bij gratie van het feit, dat ze permanent op een vaste plaats te vinden zullen zijn, via directe adressering toegang toe heeft. Een soortgelijk arrangement zal gemaakt worden om het mogelijk te maken bladzijden van het ene proces over te hevelen naar een ander proces; ik heb hier het oog op input-output-buffers.

Pagina's hebben we uit de aard der zaak in twee soorten, nl. "kernpagina's" en "Trommelpagina's". In beide media beslaan ze 512 opeenvolgende geheugenwoorden. De BZV's heten "variabelen" omdat ze onder andere vastleggen, in welke pagina(s) de bladzijde in kwestie te vinden is. Wij zullen straks uitgebreid en in detail beschrijven, welke informatie in de BZV's vastgelegd wordt en hoe. Bij die afspraak hebben twee dingen als leidraad gediend: het meest voorkomende geval moet zo snel mogelijk herkend kunnen worden (een tijdsoverweging dus) en de BZV moet zo min mogelijk ruimte in beslag nemen (een overweging van geheugenruimte dus). Dit laatste is belangrijk, omdat de BZV's —zoals wij later zullen zien— permanent in het kerngeheugen zullen staan en aan elke voorkomende bladzijde een BZV is toegevoegd.

Ook de pagina's worden door additionele "variabelen" nader beschreven. Voor de trommelpagina's - wat er heel veel zijn - is dit slechts een enkel bit, dat aangeeft of deze trommelpagina vrij, dan wel bezet is. Voor de kernpagina is deze statusinformatie aanmerkelijk omvangrijker, maar daar deert dit minder, omdat het maximale aantal kernpagina's zoveel kleiner is. De informatie, die de status van een kernpagina beschrijft heet een KIE (Kern Inhoud Element). Een KIE beslaat tot op heden vier woorden. (We merken en passant op, dat dit bijna een procent is van de geheugenruimte, die door de kernpagina zelf wordt ingenomen; bij veel kleinere kernpagina's zou de voor de KIE's benodigde geheugenruimte zich gaan wreken.) Wat een KIE precies allemaal aangeeft en hoe, zullen we eveneens in detail bespreken.

1. Classificatie van kernpagina's.

In het volgende zal een classificatie gegeven worden van de hoofdtoestanden, die we bij een kernpagina onderscheiden. Deze classificatie geeft aan (steeds), welke rol deze kernpagina op dit moment speelt. In de individuele programma's is deze rol vrij duidelijk, we moeten ons echter realiseren, dat een kernpagina ook betrokken kan zijn in een trommeltransportopdracht, of om de bladzijde, die hij herbergde op de trommel te dumpen, of om de inhoud van een trommelpagina over te nemen. Deze actie's worden niet zozeer door de individuele programma's als door de coordinator ondernomen. De tijdsduur van deze transporten is geenszins verwaarloosbaar en we moeten de status van een kernpagina "in een transport betrokken" dus expliciet invoeren. Hoeveel onderscheidingen we hierin willen aanbrengen, welke facetten tijdens zo'n transport van belang zijn, hangt van de coordinator af. Dat de hieronder te beschrijven indeling van kernpagina's zinvol is, is dus eigenlijk niet aan te tonen zonder ook in detail dat stuk van de coordinator te beschrijven, dat hier relevant is. We zullen dit toch proberen, dit rapport moge dan dienen als "underlying information" voor de beschrijving van de betrokken coordinator-mechanismen. (Zoals beschreven zijn in de blokschema's van het document "bladzorg" d.d. 1–3 mei 1964.)

Ik ben mij bewust, dat de onder te geven indeling niet dwingend is; zij is de neerslag van weloverwogen (zo men pejoratief wil zijn "willekeurige") beslissingen. Het beste, waarnaar ik dus naar streven kan is waarschijnlijk maken, dat de genomen beslissingen de weg banen tot een hanteerbare organisatie!

Ter introductie tot het soort moeilijkheden moge de volgende "PTT-gelijkenis" dienen. De PTT heeft op zich genomen, om brieven te bezorgen, bezorgt deze niet instantaan maar weet drommels goed, dat deze brieven een eindige reistijd hebben. Als alle geadresseerden het eeuwige leven hadden en nooit verhuisden, had de PTT geen probleem. Gegeven is echter het feit, dat geadresseerden slechts per adres slechts een eindige verblijfstijd doorbrengen. Het probleem van briefbezorging zou heel andere vormen aannemen, asl de gemiddelde verblijfstijd op eenzelfde adres eens klein was ten opzichte van de gemiddelde reistijd van een brief! (Soms vraag ik me af, of de PTT zich wel bewust is, hoezeer ze in dezen door het oog van de naald gekropen zijn.)

Soortgelijke moeilijkheden hebben wij, doordat trommeltransporten niet instantaan plaats vinden, ook. Tijdens de periode, dat een bladzijde tussen langzaam en snel geheugen getransporteerd wordt, kunnen er allerlei dingen mee gebeuren. De bladzijde kan ophouden te bestaan. Het kan ook gebeuren, dat terwijl we bezig zijn om deze bladzijde op de trommel te dumpen, er hernieuwd interesse in ontstaat. Maw. een trommeltransport wordt ondernomen op grond van een of andere beslissing (dat een bladzijde naar het schijnt niet erg interessant is bv.) maar tijdens de uitvoering van een dergelijk transport kunnen zich omstandigheden voordoen, waardoor men deze beslissing eigenlijk wil herroepen, resp. aan het transport een andere betekenis wil geven. De eerste beslissing, die wij daarom genomen hebben is, dat wij voor de roltoekenning van een kernpagina, die in een transport betrokken gaat worden, geen beslissing zullen nemen, die verder in de toekomst reikt dan de voltooiing van dit transport.

Het komt er op neer, dat we niet het volgende arrangement gevolgd hebben. Als een van de programma's een levendige belangstelling in een bladzijde, zeg bladzijde A, toont, die op dat moment zich niet in de kernen bevindt, dan zoekt men uit, welke bladzijde, die zich wel op de kernen bevindt, het oninteressantst lijkt. Dit zij bladzijde B. Men verwijdert nu bladzijde B uit de kernen (als hij niet ook nog op de trommel staat, impliceert dit een dumpend transport naar de trommel) en geeft vervolgens een transportopdracht om bladzijde A op de vrijgemaakte kernpagina over te nemen. Dit arrangement dwingt ons dus om ten aanzien van de kernpagina in kwestie twee transporten in de toekomst te kijken en dit hebben wij niet gekozen.

In plaats daarvan maken wij het systeem zo, dat er in principe wel een vrije kernpagina is. Ontstaat er nu behoefte aan bladzijde A, dan wordt een transportopdracht naar deze vrije kernpagina gegeven; vervolgens wordt geanalyseerd of door deze bezetting het aantal vrije kernpagina's angstig laag geworden is, zo ja, dan wordt gekeken welke het oninteressantst is en deze wordt verwijderd uit de kernen; dit laatste heeft dus mogelijk een dumpend transport ten gevolge.

De redenen, dat wij de tweede methode gekozen hebben, zijn de volgende. Nogmaals, wij zijn ons bewust, dat ze niet dwingend zijn. In het eerste geval moet je eerst de minst interessante kiezen, zodoende ruimte maken voor wat je nodig hebt. Dit lijkt in twee opzichten ongunstig. Het keuzeproces, welke het minst interessant is, kon wel eens een beetje tijdrovend zijn en bovendien komt er (mogelijk) eerst het ruimtemakend transport voordat het transport van trommel naar kernen plaats kan vinden. Het programma, dat bladzijde A nodig had, wordt daardoor gegarandeerd vrij lang opgehouden. Dat is allemaal goed in multiprogrammering, als je veel andere programma's hebt, maar dat is natuurlijk niet iets om op te bouwen. In het gekozen arrangement kan het transport voor bladzijde A meteen gegeven worden, zodat het programma, dat bladzijde A nodig heeft, zo gauw mogelijk weer door kan gaan. Het is nu de vraag, of de machine nog een ander nuttig programma op de pennen heeft en dit is dus het geknipte moment om de "minstinteressante" er uit te kiezen en maar vast ruimte te maken.

Een ander gevolg is zuiver organisatorisch: in het startmagazijn voor de trommel zal op elk moment voor een bepaalde kernpagina hoogstens 1 startopdracht staan, in het transport is elke kernpagina hoogstens met 1 bladzijde betrokken. Dit betekent, dat we de informatie over wat op komst is, wat weggetransporteerd wordt etc. per kernpagina kunnen bijhouden. dwz. dat we de omvang van een KIE klein kunnen houden. Hoewel we alternatieven niet tot het eind toe hebben uitgewerkt (we hebben pogingen ondernomen, maar raakten, mogelijk door gebrek aan ervaring, vast), hebben we het gevoel, dat het hier voor te stellen systeem onder andere zijn eenvoud ontleend aan het feit, dat een kernpagina in de tijd, dat hij vrijgemaakt zit te worden, niet al vast een andere bestemming gekregen heeft.

Nu gaan we de classificatie van de kernpagina's geven. Een kernpagina kan zich in eerste instantie in twee elkaar uitsluitende toestanden bevinden. nl. wisselend en niet-wisselend. Een kernpagina is wisselend, wanneer voor deze kernpagina een trommeltransportopdracht gegeven is, van de voltooiing waarvan door de X8 nog geen kennis is genomen; anders is de kernpagina niet-wisselend.

Zolang een kernpagina wisselend is. zal geen X8-programma zijn kernen aanraken, noch voor deze kernpagina vast andere startopdrachten geven. Het aanbod van een startopdracht impliceert dus (ongeacht richting van het transport) altijd de overgang van niet-wisselend naar wisselend; de kennisname van de voltooiing van het transport impliceert dankzij deze zelfde conventie dus altijd de inverse overgang.

(Wij zijn ons er van bewust, dat de regel, dat geen programma de kernen van een wisselende pagina zal selecteren, nodeloos stringent is. Als dit transport een transport van trommel naar kernen is, zou deze selectie onder bijkans alle omstandigheden zinloos zijn; in het inverse geval valt er nog over te denken: als we uit de kernpagina willen lezen, dan kan dit nog, willen we er in schrijven, dan heeft dit alleen maar ten gevolge, dat de copie, die op de trommel komt, tijdens dat proces al obsoleet aan het worden is. Logisch zou dit wel kunnen, we hebben het echter niet toegestaan.

De eerste reden is, dat hierdoor bij inspectie van de toestand van een kernpagina de transportrichting plotseling interessant zou worden, terwijl, zoals we zullen zien, die verder nauwelijks een rol speelt. (Na afloop van een transport, tijdens de uitvoering waarvan we niet geinterfereerd hebben met de kernpagina, kunnen we altijd de kernpagina beschouwen als een copie van de trommelpagina, ongeacht in welke richting het transport heeft plaatsgevonden.) De tweede reden is een kwestie van voorzichtigheid: stel, dat het transport een pariteitsfout detecteert! Om er dan nog uit te komen, als het programma de inhoud van de kernpagina, zoals die bedoeld was, inmiddels heeft mogen veranderen, lijkt een nodeloze verzwaring van deze toch al hachelijke opgave. (Wisselende kernpagina's blijven dus op alle manieren onaangeroerd.)

Wisselende kernpagina's worden onderscheiden in PRAEVRIJE en PRAECOPIEEN.

Een wisselende kernpagina is PRAEVRIJ, wanneer er —met inachtname van de momentane kennis van de getoonde interesse— na afloop van het transport geen interesse zal zijn in de inhoud van de kernen. Dit is dus ten duidelijkste het geval, wanneer de bladzijde, die in het transport betrokken is, tijdens dit transport zal ophouden te bestaan. Ook als een origineel gedumpt wordt om een vrije kernpagina te maken (zie onder), dan zullen we deze kernpagina als PRAEVRIJ beschouwen.

Een wisselende kernpagina is een PRAECOPIE, zodra is vastgesteld, dat er na voltooiing van het transport interesse in de kerneninhoud geacht wordt te ontstaan. Dit is ten duidelijkste het geval bij het aanhalen van een trommelpagina, die een bladzijde bevat, waarmee een van de programma's verder moet werken.

Een niet-wisselende kernpagina is in eerste aanleg een VRIJE, een COPIE of een ORIGINEEL.

Een niet-wisselende kernpagina is een VRIJE, wanneer zijn KIE niet met een BZV geassocieerd is: de inhoud van deze kernen is volledig "afgeschreven", behalve dat we wel correcte pariteit zullen eisen. (Maar dit is alleen een verplichting voor het tandenpoetsen.)

Een kernpagina is een COPIE of een ORIGINEEL, wanneer hij een bladzijde bevat; zijn KIE is dan geassocieerd met de BZV van deze bladzijde. Het verschil tussen een COPIE en een ORIGINEEL is, dat in het geval van de COPIE tevens een trommelpagina geassocieerd is, waar deze zelfde bladzijde te vinden is. Een origineel bestaat slechts enkel op de kernen. Kernpagina's, die programmatext bevatten zullen dus meestal COPIEEN zijn (nodig is dit niet: ze kunnen door de compiler in het kerngeheugen opgebouwd zijn en als er altijd voldoende ruimte in het kerngeheugen is, dan zijn ze nooit gedumpt en dus origineel gebleven.

Er zijn tussen deze vijf toestanden (hoofdtoestanden, we zullen ze later nog wat differentieren) 10 overgangen mogelijk.

1) VRIJE → PRAECOPIE. Dit vindt plaats, als een vrije kernpagina gekozen is om een bladzijde over te nemen, die alleen op dat moment op de trommel staat. Dit is een heel gewone overgang.

2) PRAECOPIE → COPIE. Dit vindt plaats als onderdeel van de kennisname van de voltooiing van een trommeltransport, waarna mogelijk interesse in de getransporteerde bladzijde ontstaat. Dit is de normale overgang, volgens welke de toestand PRAECOPIE verlaten wordt. Wij vestigen er nogmaals de aandacht op, dat deze overgang plaats kan vinden na afloop van een schrijfopdracht naar de trommel (zie overgang 10.)

3) COPIE → ORIGINEEL. Deze overgang vindt plaats, zodra een programma schrijft op een bladzijde, die in een kernpagina geborgen werd, die als COPIE te boek stond; een neveneffect hiervan is, dat de trommelpagina, die in de KIE van de kernpagina vermeld stond, wordt vrijgegeven. Tevens wordt de verwijzing naar deze trommelpagina, ter voorkoming van verwarring, verwijderd. Dit is een normale overgang voor bladzijden van een groot array, met programmabladzijden zal dit niet gebeuren.

4) ORIGINEEL → PRAEVRIJE. Deze overgang vindt plaats, als de keuze van de coordinator om ruimte te scheppen, gevallen is op een "oninteressant" ORIGINEEL. Er wordt, omdat in de kernen een ORIGINEEL staat, voor deze bladzijde een vrije trommelpagina gekozen en er wordt een (dumpende) startopdracht gegeven. Deze overgang (evenals overgang 1) vindt dus altijd plaats op het moment, dat een nieuwe startopdracht aan de trommel aangeboden wordt. Ook deze overgang moet normaal geacht worden.

5) PRAEVRIJE → VRIJE, Deze overgang vindt plaats als onderdeel van de kennisname van de voltooiing van een trommeltransport. Als de KIE van deze kernpagina nog aan een BZV gekoppeld was (en als aan het einde van het transport de bladzijde nog bestond, dan zal dit het geval zijn) dan wordt op dit moment de koppeling losgemaakt. Op dit moment is deze kerninhoud niet meer via deze BZV te bereiken.

6) COPIE → VRIJE. Deze overgang vindt plaats als de keuze van de coordinator om ruimte te scheppen gevallen is op een "oninteressante" COPIE; de overgang vindt tevens plaats, als een bladzijde ophoudt te bestaan (bv. een bladzijde van een groot array bij verlating van het blok), die als COPIE in de kernen aanwezig is, een geval, waarin tevens een trommelpagina wordt vrijgegeven.

7) ORIGINEEL → VRIJE. Deze overgang vindt plaats als een bladzijde, die als origineel in de kernen aanwezig is, ophoudt te bestaan, bv. een arraybladzijde bij blokverlating. Dit zal ook gebeuren bij inkrimping van een programmastapel, omdat, zoals we later zullen zien, de stapelbladzijden altijd als origineel in de kernen zullen staan.

8) VRIJE → ORIGINEEL. Deze overgang vindt plaats bij stapelgroei en eveneens bij eerste referentie naar een van de elementen van een nieuw geintroduceerde arraybladzijde. Bij de arraydeclaratie wordt voor een groot array nog geen bladzijde geintroduceerd, alleen maar een stel "potentiele" bladzijden, dwz. een stel BZV's, die geinitialiseerd worden met de speciale indicatie "nuchter". Pas bij de eerste referentie naar een dergelijke bladzijde wordt er aan deze BZV een feitelijke (dwz. niet-lage) bladzijde toegevoegd. Deze introductie vindt uit de aard der zaak plaats in de kernen en geeft dus aanleiding tot een ORIGINEEL.

9) PRAECOPIE → PRAEVRIJE. Deze overgang zal, zo hij voorkomt, exceptioneel zijn. Het betekent, dat we, van de veronderstelling, dat een bladzijde interessant was, afstappen en beslissen, dat we deze bladzijde toch maar niet in de kernen willen hebben. Dit zou zich voor kunnen doen, doordat het programma, dat deze bladzijde heeft aangevraagd, in die tussentijd ten gevolge van een foutmelding beeindigd wordt. Het meest waarschijnlijke geval, dat dit optreedt, is ten gevolge van de hint (zie onder); hier kan een programma van te voren vast aankondigen, dat het hoogstwaarschijnlijk straks interesse gaat tonen in een zekere bladzijde. Nodig is die interesse niet, de hint kan een arraybladzijde betreffen, die tijdens transport ophoudt te bestaan.

10) PRAEVRIJE → PRAECOPIE. Deze overgang is weer veel waarschijnlijker dan de vorige. Hij treedt op, wanneer interesse getoond wordt in een bladzijde, die op dat moment net in een transport betrokken was, waaraan op dat moment de bedoeling gehecht was, dat het een VRIJE kernpagina zou opleveren; in alle waarschijnlijkheid was dit dus een dumpend transport voor een arraybladzijde.

2. Onderverdeling van de hoofdclassificaties voor kernpagina's.

De VRIJE kernpagina's zijn het gemekkeijkste, omdat daarover het minste te vertellen valt. De belangrijkste eis is, dat we, op het moment, dat we een vrije kernpagina nodig hebben en er inderdaad eentje is, zo snel mogelijk ontdekken, welke kernpagina we daarvoor kunnen nemen. Voor dit doel zijn de VRIJEN in een ketting geregen, het schakelelement waarvan in de KIE van de kernpagina is opgenomen. Men kan voor dit doel volstaan met een enkelvoudige ketting, omdat je de VRIJEN —ze zijn je allemaal even lief— gebruiken kunt volgens het principe van een stapel. Een nieuwe vrije schakel je voor aan de ketting erbij, als je een vrije nodig hebt, neem je de eerste van de ketting. Het is de plicht van de tandenpoetserij om aan het begin alle kernpagina's, die dan vrij zijn, in deze ketting op te nemen.

Voor de wisselende kernpagina's moeten we onderscheiden, of het transport betrekking heeft op een nog bestaande bladzijde of niet. Bij de PRAECOPIEEN is dit in elk geval zo, voor de PRAEVRIJEN hoeft de bladzijde echter niet meer te bestaan, In geval van wisselende kernpagina ten bate van een bestaande bladzijde zal tijdens het transport de koppeling van BZV en KIE bestaan. Wie vraaagt naar een bladzijde, die in wisseling betrokken is, wordt via de BZV van die bladzijde (deze BZV is uniek voor die bladzijde en de enige manier om contact met die bladzijde op te nemen) naar de KIE verwezen worden, daar echter zal genoteerd staan, dat de kernpagina wisselend is. Tot deze "vroegtijdige" koppeling worden we gedwongen door het bestaan van de bibliotheek: het is immers heel goed mogelijk, dat een programma niet verder kan, vanwege de afwezigheid van een bibliotheekbladzijde; in de wachttijd van dit programma kan heel wel een ander programma, dat wel door kan gaan, ontdekken, dat het deze zelfde bibliotheekbladzijde nodig heeft. Zijn het de PRAEVRIJEN, waarbij we het onderscheid moeten aangeven of de bladzijde nog bestaat, bij de PRAECOPIEEN kunnen we reeds betekenis toekennen aan de zg. heiligheidsteller. (Zie onder.)

De COPIEEN en de ORIGINELEN worden onderverdeeld in "profanen" en "heiligen". Een heilige kernpagina is een kernpagina, waaraan de coordinator niet op eigen gezag een andere rol mag toebedelen. Het is nl. wel mogelijk, de programma's zo te schrijven, dat een bladzijde niet noodzakelijkerwijs ergens in de kernen is, het is echter niet toegestaan, dat een bladzijde, waarvan de aanwezigheid door een programma geconstateerd is, op elk willekeurig microscopisch moment verdwijnt. Allicht niet: als een programma een element van een groot array wil selecteren en heeft vastgesteld, op welk kernadres dit element zich bevindt, dan mag tussen deze vaststelling en de feitelijke selectie dit element niet uit de kernen verdwijnen. Wel, heel vaak zal dit gegarandeerd worden, door het stuk van verificatie van de aanwezigheid en de feitelijk selectie in doofheid te doen uitvoeren. Als X8 aan zo'n stukje begonnen is, dan wordt dat dus eerst afgemaakt, voordat mogelijk de coordinator over de rol van deze kernpagina anders beslist. (In multiprogrammering moet je zoals bekend voorzichtig zijn met doofheid: in de individuele programma's zal de X8 heel vaak eventjes doof zijn.)

Het mechanisme van de tijdelijke doofheid is echter niet machtig genoeg: de efficiency van de individuele programma's wordt aanzienlijk verhoogd, indien zij de mogelijkheid hebben om ten aanzien van een beperkt aantal pagina's over een beperkt verloop van tijd er op te mogen rekenen, dat deze bladzijden niet onder hun vingers worden weggegrist. Aan elk programma is de plicht, zo geconstrueerd te zijn, dat het op geen moment te veel kernpagina's simultaan heiligt, want anders zouden ze het kerngeheugen blokkeren. (Hiermee hebben we bij de behandeling van de stapel een beetje de hand gelicht, zie later.)

Heiliging van een kernpagina is iets, dat geschiedt op instignatie van een van de individuele programma's; dit programma laadt daarmee de plicht op zich om deze heiliging mettertijd weer ongedaan te maken, deze pagina weer te "profaneren". In het bijzonder zal een programma, dat een niet in de kernen aanwezige bladzijde voor zijn voortzetting nodig heeft, vragen om deze bladzijde in een geheiligde kernpagina. Met deze heiliging is gekoppeld, dat dit programma straks via een P-operatie op deze bladzijde kan gaan staan wachten. Hiermee voorkomen we de volgende situatie: een programma heeft een bladzijde, die enkel op de trommel staat, beslist nodig, er wordt een trommeltransport voor dit programma gestart en het programma komt op de wachtlijst der geblokkeerde programma's. Na voltooiing van dit transport wordt het wachtende programma overgeheveld naar de uitvoerbaren, maar het komt niet direct aan bod. Als we nu de pagina in kwestie niet geheiligd hadden, dan zou de mogelijkheid bestaan, dat dit programma, tegen de tijd, dat het wel aan bod komt, tot de ontdekking moet komen, dat de bladzijde, waarop het had staan wachten, inmiddels weer verdwenen is. Dit nu hebben we met behulp van de heiligheid uitgesloten: deze bladzijde blijft onaantastbaar in het kerngeheugen staan, totdat het programma, dat om deze bladzijde gevraagd heeft, de inhoud van deze bladzijde inderdaad heeft kunnen selecteren.

De bibliotheek schept hier een kleine complicatie. Uit het voorafgaande zou men kunnen concluderen, dat het voldoende is, om elke kernpagina een heiligheidsbit te geven, maar terwijl het ene programma staat te wachten op de aankomst van een bibliotheekbladzijde, kan een ander programma doorgaan en ook op deze bladzijde komen te wachten. En niet alleen in deze toestand. We zullen in elk programma de bladzijde, waaruit op dat moment de opdrachten gehoorzaamd worden, heiligen, dwz. heiligen bij binnenkomst en profaneren bij verlating. Twee verschillende programma's kunnen dus beide dezelfde bladzijde willen heiligen. Om die reden bevat de KIE niet een heiligheidsbit, maar een heiligheidsteller; heiliging betekent verhoging met 1, profanatie betekent aflaging met 1 en een kernpagina geldt als profaan, wanneer zijn heiligheidsteller = 0 is. (Naast de individuele programma's zullen wij in het "Hotel voor abstracte machines" ook stamgasten hebben ter verzorging van input-output bv.; dankzij de heiligheidsteller is het nu ook mogelijk, meer dan 1 kleine stamgast in dezelfde programmapagina te definieren, c.q. een stamgast parallel te activeren.)

Het heilig aanvragen van een bladzijde legt een onontkoombare verplichting op aan de coordinator (nl. om vroeg of laat met die bladzijde voor de draad te komen), het programma, op welks instignatie dit gebeurd is, heeft hiermee de verplichting op zich genomen, om vroeg of laat deze pagina weer te profaneren. Dit is het ene mechanisme, dit lijkt ons absoluut noodzakelijk. Daarnaast willen we in dit stadium niet de mogelijkheid uitsluiten, dat een (waarschijnlijk handgemaakt) programma de coordinator een zg. "hint" geeft, behelsend "Straks ben ik waarschijnlijk in die en die bladzijde geinteresseerd". Dit is het soort opmerkingen, dat de coordinator als niet opportuun naast zich neer kan leggen; het programma, dat de hint gegeven heeft, zal voor de feitelijk selectie toch de aanwezigheidstest weer uitvoeren. Als de coordinator de hint krijgt en de bladzijde is aanwezig, dan kan hij het interessegetal van deze kernpagina verhogen. daarmee de kans verkleinend, dat deze bladzijde er in de naaste toekomst uitgekeild wordt; als de bladzijde er niet is, dan mag een trommeltransport gegeven worden (als er een geschikte vrije is, als er ruimte in het startmagazijn is etc.) Deze verwerking van de hint gaat niet met heiliging gepaard. Hier zien we dus, dat ook bij de PRAECOPIEEN de heiligheidsteller van deze kernpagina zal moeten gegeven worden, als de PRAECOPIE in COPIE overgaat. In de volgende sectie zullen we zien, dat dit gegeven op de keeper beschouwd, overbodig zal zijn.

3. Terugmelding van aankomst van bladzijden.

Als een programma vraagt om een heilige kernpagina, dan betekent dit vanuit de kant van het programma, dat het via de voltooiing van een P-operatie van de aanwezigheid van deze bladzijde in kennis gesteld wil worden, het betekent voor het systeem, dat de aankomst van deze bladzijde in een kernpagina door de ophoging van een seinpaal bezegeld moet worden. De eersste vraag is, waar we deze seinpaal onderbrengen. Een eerste suggestie is, om deze seinpaal onder te brengen bij de startopdracht; we zouden dan vier van deze trommelseinpalen hebben, voor elke mogelijke startopdracht in het magazijn eentje. Bij aanbod van een nieuwe startopdracht zouden we deze seinpaal = 0 moeten zetten, in de heiligheidsteller van de PRAECOPIE houden we bij, hoeveel programma's echt op deze bladzijde staan te wachten en als reactie op de ingreep voeren we de V-operatie op deze seinpaal uit met de dan heersende waarde van de heiligheidsteller als increment. Een programma, dat op een bladzijde moet wachten, doet dit, omdat de bladzijde al wisselend is of omdat er een startopdracht voor die bladzijde gegeven moet worden, in beide gevallen is bekend, welke van de vier startopdrachten voor het transport zorgt en het programma weet op dat moment, op welke seinpaal het straks moet gaan wachten. Dit arrangement is echter uiterst link. Dit werkt nl. goed, wanneer het aanvragende programma al op deze seinpaal staat te wachten, tegen de tijd, dat de bladzijde arriveert; in dat geval bewerkstelligt de V-operatie nl. dat het programma van de lijst der geblokkeerden naar die van de uitvoerbaren wordt overgeheveld. Dit gaat echter scheef, wanneer op het moment van aankomst van de bladzijde het programma nog niet zover is, nog niet aan de P-operatie toe is. Tegen de tijd, dat dat nl. het geval is, kan diezelfde plaats in het startmagazijn (en daarmee diezelfde seinpaal) allang voor een ander transport gebruikt zijn.

Een weg uit deze moeilijkheden is, om de seinpaal niet onder te brengen bij de startopdracht maar bij de KIE van de kernpagina in kwestie. Dit is dan wel safe, maar heeft het nadeel, dat dit wel de nodige geheugenplaatsen zal gaan kosten. Een geprogrammeerde seinpaal kost nl. drie geheugenplaatsen en dit zou dus ongeveer een verdubbeling van de pakweg vijftig KIE's betekenen; dit is des te triester, omdat er normaliter niet meer dan vier van deze seinpalen gebruikt worden.

Wij hebben besloten kool en geit te sparen en de seinpalen, die deze blokkade beheersen, bij de individuele programma's onder te brengen (er op gokkend, dat we aanmerkelijk minder programma's in actie zullen hebben dan kernpagina's). Ieder programma heeft dus zijn eigen seinpaal. Het aanvragen van een heilige kernpagina betekent nu (het = 0 zetten van de eigen seinpaal, als daar het tandenpoetsen niet al voor gezorgd heeft) en het melden aan de trommelorganisatie, via welke seinpaal dit keer de voltooiing van het transport teruggeseind moet worden. Bij de kennisname van de voltooiing van het transport moet nu de "trommelingreepreactor" niet een vaste seinpaal met mogelijk meer dan 1 ophogen, maar een lijstje van seinpalen (mogelijk leeg, mogelijk langer dan 1) moeten allemaal met 1 opgehoogd worden. Daartoe bevatten de eigen seinpalen een schakelelement en bij de startopdracht bevindt zich de beginschakel. En nu zien we, dat de heiligheidsteller bij de PRAECOPIE eigenlijk overbodig is; deze is nl. gelijk aan de lengte van de seinpalenketting, die hangt aan de betrokken startopdracht.

Een wezenlijke veronderstelling, die aan dit laatste arrangement ten grondslag ligt, is, dat per programma de aanvraag van een heilige pagina en de P-operatie op de eigen seinpaal in de tijd elkaar afwisselen: Als je immers eerst de ene bladzijde heilig aanvraagt en dan de ander, en vervolgens 1 P-operatie doet, dan weet je niet meer, welke bladzijde nu gearriveerd is. Dit staat ons tevens toe, de eigen seinpaal slechts in 1 enkele ketting opneembaar te kunnen maken. Mocht, wat ik niet veronderstel, dit "ongenest" gebruik van de eigen seinpaal tot ongewenste restricties leiden, dan kunnen we altijd nog over gaan tot een seinpaal per kernpagina.

4. Coderingen.

Om der wille van de volledigheid zal ik nu beschrijven, welke afspraken wij gemaakt hebben over de wijze, waarop de BZV's en de KIE's de informatie, die ze moeten herbergen, vasthouden. Dit is helemaal niet spannend, de genomen beslissingen zijn soms volslagen willekeurig en als ze dat niet zijn, zijn ze ingegeven door het reinste opportunisme.

4.1. De structuur van de BZV.

Een BZV beslaat 1 woord (met zorg niet meer, omdat we er voor elke bestaande bladzijde eentje in het kerngeheugen moeten invoeren). Een BZV kent vier toestanden.

1) BZVi = -0

Dit zal betekenen "De bladzijde in kwestie is nuchter".

2) BZVi = | d[26] = 1

| d[25] = 0

| d[24] . . . d[0] = beginadres trommelpagina, links aangevuld met nullen

Dit zal betekenen, dat de bladzijde in kwestie niet nuchter is, te vinden is op de aangegeven plaats op de trommel en nergens anders; de bladzijde is tevens niet in een transport betrokken.

3) BZVi =:BKPk (dwz. Basis Kern Pagina nr. k), links aangevuld met nullen.

Dit zal betekenen, dat de bladzijde niet nuchter is, niet in een transport betrokken is, wel op de kernen aanwezig is en wel als

BKPk[0] t/m BKPk[511]

Opm.1.:BKPk zal een 512-voud zijn.

Opm.2. Het basisadres van de bijbehorende KIEk wordt gevonden door

:KIEk =:BPKk / 128 + KPC

(KPC = KIEtabel Positionerings Constante); een KIE beslaat dus vier woorden. Opm.3. Toestand 3 zal gekarakteriseerd zijn door BZVi groter dan 0.

4) BZVi =:BPKk, links aangevuld met nullen)

Dit zal betekenen, dat de bladzijde in een transport betrokken is met betrekking tot kernpagina k.

Opm. Deze toestand onderscheidt zich van 1) doordat BZVi ≠ 0 is en van 2) doordat

d[25] = 1 is.

Het gevolg van deze conventie's is, dat de vaste aanwezigheid van een bladzijde volledig uit de BZV volgt, zonder raadpleging van de KIE; bij het schrijven op zo'n bladzijde moet wel de KIE geraadpleegd worden om te kijken, of dit schrijven wellicht de overgang van COPIE tot ORIGINEEL ten gevolge heeft. Wij kunnen overwegen, om dit bij toestand 3 in bit d[25] weer te geven; ik voel hier op het ogenblik eigenlijk weinig voor, selectie van een arrayelement is anyhow een aanzienlijk proces en raadplegen van de KIE is nu ook weer niet onoverkomelijk (het testen van zo'n hoger bit kost ook wel wat).

4.2. De structuur van de KIE.

KIEk[0] =
indien gekoppeld aan een BZV, dan
= :BZVi, links aangevuld met nullen

anders
= - 0

KIEk[1] = indien trommelverwijzing van toepassing (impliceert onvrij)

d[26] = 1.

d[25] = 0

d[24] . . . d[0] = beginadres trommelpagina, links aangevuld met nullen

anders (dwz. trommelverwijzing niet van toepassing)

indien onvrij, dan = -0

anders (dwz. indien VRIJ)

indien laatste pagina uit vrije ketting, dan = +0

anders =:KIEj (≠ 0), verwijzend naar de KIE van de volgende pagina uit de vrije ketting

KIEk[2] =
als COPIE of ORIGINEEL, dan HHTk (< 128)

(= 0, 1, 2, 3, . . . .) HHT betekent "Heiligheidsteller".

als PRAECOPIE, dan 128 + HHTk + 1024 * startopdrachtnummer

als PRAEVRIJ, dan 256 + 1024 * startopdrachtnummer

als VRIJ, dan 512.

KIEk[3] = strategische informatie (ook wel interessegetal genoemd).

4.3. Opmerkingen.

Tijdens transport van een bestaande pagina zijn KIEk en BZVi dus aan elkaar gekoppeld, ook als de kernpagina PRAEVRIJ is. Tijdens transport van een niet meer bestaande bladzijde is de kernpagina uit de aard der zaak PRAEVRIJ, KIEk[0] = -0 en ook KIEk[1] = -0. Het vermoorden van een bladzijde, die in een transport betrokken is, heeft nl. toch onmiddellijke vrijgave van de trommelpagina ten gevolge. Dit kan, omdat de trommelorganisatie de transportopdrachten verwerkt in de volgorde van aanbod, elke latere bestemming van deze trommelpagina wordt dus pas effectief, als de eerste transporteerderij volledig afgelopen is.

Wij hebben de kernpagina's niet geinterspatieerd met de bijbehorende KIE's, hoewel dit ons de schuifopdracht voor de herleiding van KIEk tot BKPk of omgekeerd zou besparen. De redenen waren:

- mocht ooit een systeem van memory protection zijn intrede doen, dan is het waarschijnlijk, dat een techniek, die kernpagina's op hoge tweemachten laat beginnen, meer aansluit bij wat gebruikelijk wordt
- het zou voor testdoeleinden (misschien bedrijven wij aanvankelijk nog wel inspectie) wel eens makkelijk kunnen zijn
- de herleiding van physisch adres tot kernpagina en regelnummer in deze pagina wordt aanmerkelijk vergemakkelijkt.

Van het feit, dat de schone schuifopdrachten in een enkel register snel zullen zijn, zullen wij dus dankbaar gebruik maken.

transcribed by Sam Samshuijzen

revised Tue, 18 Jul 2006

---

## English translation

### On page administration

EWD 84

15 May 1964

On page administration.

0. Some terminology.

In order to avoid confusion of speech, we introduce the notions "page frame" and "page". A page is a quantity of information of 512 words; a page frame is a portion of one of the memories, intended, if used, to house a page. A page frame therefore occupies 512 words. The page is thus the quantity of information that "fits" into a page frame.

In the programs it is the pages that have a clear identity. The text of the object program is subdivided into pages, pages whose existence begins in the course of the compilation process and ends after completion of the program. Large arrays will likewise have their elements arranged into pages; in principle these pages begin their existence upon the execution of the array declaration, and they end their existence upon the definitive departure from the block in question.

The stack is treated somewhat differently; it will indeed always consist of a number of pages, but in order to address the elements of these pages a mechanism other than the one needed to select an element from a general page (program text or large array) will be created. The elements in the stack, namely, will during their lifetime be able to reside only at a fixed place in the cores, so that we can characterize these elements throughout their existence by their physical core address.

Every page (at least the non-stack pages) derives its identity from a quantity attached to it, a so-called BZV (BladZijde Variabele, Page Variable). Usually the BZV's are situated in the stack of the program in which these pages occur. (This holds most clearly for the pages that make up the program text; the corresponding BZV's may be regarded as global variables of this program. It also holds for the BZV's of array pages; these are local quantities of the block in which the array is declared.) There also occur, in the interplay, other pages, e.g. the pages that the library of standard procedures, which is deemed to be permanently present on the drum; these evidently cannot be accommodated exclusively in the stack of 1 program. For the library is a part of the universe in which the various programs are jointly embedded; the library surpasses, encompasses, in lifetime also the individual programs. This finds its reflection in the fact that, beside what the programs need privately, at a fixed place in the core memory a little row of BZV's will be accommodated, the "library BZV's", to which every individual program, by virtue of the fact that they will permanently be found at a fixed place, has access via direct addressing. A similar arrangement will be made to make it possible to transfer pages from one process to another process; here I have in mind input-output buffers.

Page frames we have, in the nature of things, in two kinds, namely "core page frames" and "Drum page frames". In both media they occupy 512 consecutive memory words. The BZV's are called "variables" because among other things they record in which page frame(s) the page in question is to be found. We shall presently describe extensively and in detail which information is recorded in the BZV's and how. In this convention two things have served as a guideline: the most frequently occurring case must be recognizable as quickly as possible (a consideration of time, then) and the BZV must take up as little space as possible (a consideration of memory space, then). The latter is important, because the BZV's —as we shall see later— will reside permanently in the core memory and to every occurring page a BZV is attached.

The page frames too are described more closely by additional "variables". For the drum page frames - of which there are very many - this is only a single bit, which indicates whether this drum page frame is free or occupied. For the core page frame this status information is considerably more extensive, but there this matters less, because the maximum number of core page frames is so much smaller. The information that describes the status of a core page frame is called a KIE (Kern Inhoud Element, Core Content Element). A KIE occupies, up to now, four words. (We note in passing that this is almost one percent of the memory space that is taken up by the core page frame itself; with much smaller core page frames the memory space required for the KIE's would begin to take its revenge.) Precisely what a KIE indicates in full and how, we shall likewise discuss in detail.

1. Classification of core page frames.

In the following a classification will be given of the principal states that we distinguish in a core page frame. This classification indicates (in every case) which role this core page frame is playing at this moment. In the individual programs this role is fairly clear; we must, however, realize that a core page frame can also be involved in a drum transport order, either to dump onto the drum the page that it housed, or to take over the content of a drum page frame. These actions are undertaken not so much by the individual programs as by the coordinator. The duration of these transports is by no means negligible, and we must therefore explicitly introduce the status of a core page frame "involved in a transport". How many distinctions we wish to make within this, which facets are of importance during such a transport, depends on the coordinator. That the partition of core page frames to be described below is meaningful is therefore actually not demonstrable without also describing in detail that part of the coordinator that is relevant here. We shall nonetheless attempt this; this report may then serve as "underlying information" for the description of the coordinator mechanisms concerned. (As they are described in the block diagrams of the document "bladzorg" [page care] dated 1–3 May 1964.)

I am aware that the partition to be given below is not compelling; it is the reflection of well-considered (if one wishes to be pejorative, "arbitrary") decisions. The best that I can therefore strive for is to make it probable that the decisions taken pave the way to a manageable organization!

By way of introduction to the kind of difficulties, the following "PTT-parable" may serve. The PTT [the postal service] has taken upon itself to deliver letters, does not deliver these instantaneously but knows perfectly well that these letters have a finite travel time. If all addressees had eternal life and never moved house, the PTT would have no problem. Given, however, is the fact that addressees spend only a finite residence time per address. The problem of letter delivery would take on quite different forms if the average residence time at one and the same address were once small relative to the average travel time of a letter! (Sometimes I wonder whether the PTT is really aware how very much they have, in this matter, crawled through the eye of the needle.)

Similar difficulties we too have, owing to the fact that drum transports do not take place instantaneously. During the period in which a page is being transported between slow and fast memory, all sorts of things can happen to it. The page can cease to exist. It can also happen that while we are busy dumping this page onto the drum, renewed interest in it arises. In other words: a drum transport is undertaken on the grounds of some decision or other (that a page is, it seems, not very interesting, for instance), but during the execution of such a transport circumstances may arise on account of which one actually wishes to revoke this decision, or rather to give the transport a different meaning. The first decision that we have therefore taken is that, for the role assignment of a core page frame that is going to be involved in a transport, we shall take no decision that reaches further into the future than the completion of this transport.

It comes down to the fact that we have not followed the following arrangement. If one of the programs shows a lively interest in a page, say page A, which at that moment is not in the cores, then one finds out which page that is in the cores seems the least interesting. Let this be page B. One now removes page B from the cores (if it does not also stand on the drum, this implies a dumping transport to the drum) and subsequently issues a transport order to take over page A onto the freed core page frame. This arrangement therefore forces us, with respect to the core page frame in question, to look two transports into the future, and this we have not chosen.

Instead we make the system such that there is, in principle, a free core page frame. If a need for page A now arises, a transport order to this free core page frame is issued; subsequently it is analysed whether, through this occupation, the number of free core page frames has become alarmingly low; if so, then one looks which is the least interesting and this one is removed from the cores; the latter therefore possibly entails a dumping transport.

The reasons that we have chosen the second method are the following. Once more, we are aware that they are not compelling. In the first case you must first choose the least interesting, thereby making room for what you need. This seems unfavourable in two respects. The choice process, which is the least interesting, could well be a bit time-consuming, and moreover (possibly) the room-making transport comes first, before the transport from drum to cores can take place. The program that needed page A is thereby guaranteed to be held up for quite a long time. That is all fine in multiprogramming, if you have many other programs, but that is of course not something to build on. In the chosen arrangement the transport for page A can be issued right away, so that the program that needs page A can resume as soon as possible. The question now is whether the machine has yet another useful program in the pipeline, and this is therefore the perfect moment to pick out the "least interesting" and just go ahead and make room.

Another consequence is purely organizational: in the start store for the drum there will at every moment stand at most 1 start order for a given core page frame; in transport every core page frame is involved with at most 1 page. This means that we can keep track, per core page frame, of the information about what is on its way, what is being transported away, etc., i.e. that we can keep the size of a KIE small. Although we have not worked out the alternatives to the very end (we made attempts, but got stuck, possibly through lack of experience), we have the feeling that the system to be proposed here derives its simplicity, among other things, from the fact that a core page frame, during the time that it is being freed, has not already been given another destination.

Now we shall give the classification of the core page frames. A core page frame can in the first instance be in two mutually exclusive states, namely changing and non-changing. A core page frame is changing when a drum transport order has been issued for this core page frame, of the completion of which the X8 has as yet taken no cognizance; otherwise the core page frame is non-changing.

As long as a core page frame is changing, no X8 program will touch its cores, nor will it issue, for this core page frame, any further start orders. The offering of a start order therefore always implies (regardless of the direction of the transport) the transition from non-changing to changing; the taking cognizance of the completion of the transport therefore, thanks to this same convention, always implies the inverse transition.

(We are aware that the rule that no program shall select the cores of a changing page frame is needlessly stringent. If this transport is a transport from drum to cores, this selection would under nearly all circumstances be meaningless; in the inverse case there is still something to consider: if we want to read from the core page frame, this is still possible; if we want to write into it, this has only the consequence that the copy that comes onto the drum is, during that process, already becoming obsolete. Logically this would be possible; we have, however, not permitted it.

The first reason is that hereby, upon inspection of the state of a core page frame, the transport direction would suddenly become interesting, whereas, as we shall see, it otherwise hardly plays a role. (After the conclusion of a transport during the execution of which we have not interfered with the core page frame, we can always regard the core page frame as a copy of the drum page frame, regardless of in which direction the transport has taken place.) The second reason is a matter of caution: suppose that the transport detects a parity error! To then still extricate oneself, if the program has meanwhile been allowed to change the content of the core page frame as it was intended, seems a needless aggravation of this already precarious task. (Changing core page frames thus remain in every way untouched.)

Changing core page frames are distinguished into PREFREE and PRECOPIES.

A changing core page frame is PREFREE when —taking into account the momentary knowledge of the interest shown— after the conclusion of the transport there will be no interest in the content of the cores. This is therefore most clearly the case when the page involved in the transport will cease to exist during this transport. Also when an original is dumped in order to make a free core page frame (see below), we shall regard this core page frame as PREFREE.

A changing core page frame is a PRECOPY as soon as it has been established that, after completion of the transport, interest in the core content is deemed to arise. This is most clearly the case in the fetching of a drum page frame that contains a page with which one of the programs has to work on.

A non-changing core page frame is in the first instance a FREE one, a COPY or an ORIGINAL.

A non-changing core page frame is a FREE one when its KIE is not associated with a BZV: the content of these cores has been completely "written off", except that we shall indeed require correct parity. (But this is only an obligation for the tooth-brushing.)

A core page frame is a COPY or an ORIGINAL when it contains a page; its KIE is then associated with the BZV of this page. The difference between a COPY and an ORIGINAL is that in the case of the COPY a drum page frame is also associated, where this same page is to be found. An original exists solely on the cores alone. Core page frames that contain program text will therefore usually be COPIES (this is not necessary: they can have been built up by the compiler in the core memory, and if there is always sufficient room in the core memory, then they have never been dumped and have thus remained original.

Between these five states (principal states; we shall differentiate them somewhat further later) 10 transitions are possible.

1) FREE → PRECOPY. This takes place when a free core page frame has been chosen to take over a page that at that moment stands only on the drum. This is a very ordinary transition.

2) PRECOPY → COPY. This takes place as part of the taking cognizance of the completion of a drum transport, after which interest in the transported page possibly arises. This is the normal transition by which the state PRECOPY is left. We once more draw attention to the fact that this transition can take place after the conclusion of a write order to the drum (see transition 10.)

3) COPY → ORIGINAL. This transition takes place as soon as a program writes onto a page that was stored in a core page frame which was on the books as a COPY; a side effect of this is that the drum page frame that was recorded in the KIE of the core page frame is released. At the same time the reference to this drum page frame is, in order to prevent confusion, removed. This is a normal transition for pages of a large array; with program pages this will not happen.

4) ORIGINAL → PREFREE. This transition takes place when the coordinator's choice to create room has fallen on an "uninteresting" ORIGINAL. Because an ORIGINAL stands in the cores, a free drum page frame is chosen for this page and a (dumping) start order is issued. This transition (just like transition 1) therefore always takes place at the moment that a new start order is offered to the drum. This transition too must be deemed normal.

5) PREFREE → FREE. This transition takes place as part of the taking cognizance of the completion of a drum transport. If the KIE of this core page frame was still coupled to a BZV (and if at the end of the transport the page still existed, then this will be the case) then at this moment the coupling is undone. At this moment this core content is no longer reachable via this BZV.

6) COPY → FREE. This transition takes place when the coordinator's choice to create room has fallen on an "uninteresting" COPY; the transition also takes place when a page ceases to exist (e.g. a page of a large array upon departure from the block) that is present in the cores as a COPY, a case in which a drum page frame is also released.

7) ORIGINAL → FREE. This transition takes place when a page that is present in the cores as an original ceases to exist, e.g. an array page upon block departure. This will also happen upon shrinkage of a program stack, because, as we shall see later, the stack pages will always stand in the cores as originals.

8) FREE → ORIGINAL. This transition takes place upon stack growth and likewise upon the first reference to one of the elements of a newly introduced array page. Upon the array declaration no page is yet introduced for a large array, only a set of "potential" pages, i.e. a set of BZV's, which are initialized with the special indication "sober". Only upon the first reference to such a page is an actual (i.e. non-low) page attached to this BZV. This introduction takes place, in the nature of things, in the cores and therefore gives rise to an ORIGINAL.

9) PRECOPY → PREFREE. This transition will, if it occurs, be exceptional. It means that we abandon the supposition that a page was interesting and decide that we do not want to have this page in the cores after all. This could occur because the program that requested this page is, in the meantime, terminated as a result of an error report. The most probable case in which this occurs is as a consequence of the hint (see below); here a program can announce in advance that it will most probably show interest, presently, in a certain page. That interest is not necessary; the hint may concern an array page that ceases to exist during transport.

10) PREFREE → PRECOPY. This transition is again much more probable than the previous one. It occurs when interest is shown in a page that at that moment was just involved in a transport to which, at that moment, the intention had been attached that it would yield a FREE core page frame; in all probability this was therefore a dumping transport for an array page.

2. Subdivision of the principal classifications for core page frames.

The FREE core page frames are the easiest, because there is least to tell about them. The most important requirement is that, at the moment when we need a free core page frame and there is indeed one, we discover as quickly as possible which core page frame we can take for it. For this purpose the FREE ones are strung in a chain, the link element of which is included in the KIE of the core page frame. For this purpose one can make do with a single chain, because you can use the FREE ones —they are all equally dear to you— according to the principle of a stack. A new free one you link on at the front of the chain; if you need a free one, you take the first of the chain. It is the duty of the tooth-brushing to include at the beginning all core page frames that are then free into this chain.

For the changing core page frames we must distinguish whether the transport concerns a still-existing page or not. With the PRECOPIES this is in any case so; for the PREFREE ones, however, the page need no longer exist. In the case of a changing core page frame for the benefit of an existing page, the coupling of BZV and KIE will exist during the transport. Whoever asks for a page that is involved in changing will, via the BZV of that page (this BZV is unique for that page and the only way to make contact with that page), be referred to the KIE, where, however, it will be noted that the core page frame is changing. To this "premature" coupling we are forced by the existence of the library: it is, after all, very well possible that a program cannot proceed because of the absence of a library page; during the waiting time of this program another program, which can indeed proceed, may very well discover that it needs this same library page. It is for the PREFREE ones that we must indicate the distinction whether the page still exists; for the PRECOPIES we can already assign meaning to the so-called sanctity counter. (See below.)

The COPIES and the ORIGINALS are subdivided into "profane ones" and "sacred ones". A sacred core page frame is a core page frame to which the coordinator may not, on its own authority, assign another role. It is, namely, indeed possible to write the programs in such a way that a page is not necessarily somewhere in the cores; it is, however, not permitted that a page whose presence has been ascertained by a program disappears at any arbitrary microscopic moment. Naturally not: if a program wishes to select an element of a large array and has ascertained at which core address this element resides, then between this ascertainment and the actual selection this element may not disappear from the cores. Well, very often this will be guaranteed by having the piece of verification of the presence and the actual selection executed in deafness. If the X8 has begun on such a little piece, then that is first finished before the coordinator possibly decides otherwise about the role of this core page frame. (In multiprogramming one must, as is known, be careful with deafness: in the individual programs the X8 will very often be momentarily deaf.)

The mechanism of temporary deafness is, however, not powerful enough: the efficiency of the individual programs is considerably increased if they have the possibility of being able to count, with respect to a limited number of pages over a limited course of time, on these pages not being snatched away from under their fingers. Upon each program lies the duty to be so constructed that it at no moment sanctifies too many core page frames simultaneously, for otherwise they would block the core memory. (In this respect we have, in the treatment of the stack, cut a few corners; see later.)

Sanctification of a core page frame is something that happens at the instigation of one of the individual programs; this program thereby takes upon itself the duty to undo this sanctification in due time, to "profane" this page frame again. In particular, a program that needs, for its continuation, a page not present in the cores will ask for this page in a sanctified core page frame. Coupled with this sanctification is that this program can presently, via a P-operation, go and wait on this page. Hereby we prevent the following situation: a program definitely needs a page that stands only on the drum, a drum transport for this program is started and the program comes onto the waiting list of the blocked programs. After completion of this transport the waiting program is transferred to the executable ones, but it does not come up immediately. If we had not sanctified the page in question, then the possibility would exist that this program, by the time that it does come up, must come to the discovery that the page on which it had been waiting has in the meantime disappeared again. This, then, we have ruled out with the help of sanctity: this page remains inviolable in the core memory until the program that requested this page has indeed been able to select the content of this page.

The library creates here a small complication. From the foregoing one might conclude that it is sufficient to give every core page frame a sanctity bit, but while the one program is waiting for the arrival of a library page, another program can proceed and also come to wait on this page. And not only in this state. In each program we shall sanctify the page from which at that moment the orders are being obeyed, i.e. sanctify upon entry and profane upon departure. Two different programs can thus both wish to sanctify the same page. For that reason the KIE contains not a sanctity bit, but a sanctity counter; sanctification means increase by 1, profanation means decrease by 1, and a core page frame counts as profane when its sanctity counter = 0. (Beside the individual programs we shall, in the "Hotel for abstract machines", also have regulars, for the servicing of input-output, for example; thanks to the sanctity counter it is now also possible to define more than 1 small regular in the same program page, respectively to activate a regular in parallel.)

The sacred requesting of a page imposes an inescapable obligation upon the coordinator (namely, to come forward with that page sooner or later); the program at whose instigation this has happened has thereby taken upon itself the obligation to profane this page again sooner or later. This is the one mechanism; this seems to us absolutely necessary. Besides this we do not, at this stage, want to rule out the possibility that a (probably handmade) program gives the coordinator a so-called "hint", to the effect "Presently I shall probably be interested in such-and-such a page". This is the kind of remark that the coordinator may set aside as not opportune; the program that has given the hint will, for the actual selection, nonetheless execute the presence test again. If the coordinator gets the hint and the page is present, then it can increase the interest number of this core page frame, thereby diminishing the chance that this page is kicked out in the near future; if the page is not there, then a drum transport may be issued (if there is a suitable free one, if there is room in the start store, etc.). This processing of the hint is not accompanied by sanctification. Here we therefore see that with the PRECOPIES too the sanctity counter of this core page frame will have to be given when the PRECOPY passes into a COPY. In the following section we shall see that this datum, when all is said and done, will be superfluous.

3. Report-back of the arrival of pages.

When a program asks for a sacred core page frame, then this means, from the side of the program, that it wishes to be informed, via the completion of a P-operation, of the presence of this page; it means, for the system, that the arrival of this page in a core page frame must be sealed by the raising of a semaphore. The first question is where we accommodate this semaphore. A first suggestion is to accommodate this semaphore with the start order; we would then have four of these drum semaphores, one for every possible start order in the store. Upon the offering of a new start order we would have to set this semaphore = 0; in the sanctity counter of the PRECOPY we keep track of how many programs are really waiting on this page, and as reaction to the intervention we execute the V-operation on this semaphore with the then-prevailing value of the sanctity counter as increment. A program that has to wait on a page does this because the page is already changing or because a start order for that page must be issued; in both cases it is known which of the four start orders takes care of the transport, and the program knows at that moment on which semaphore it must presently go and wait. This arrangement is, however, extremely tricky. It works well, namely, when the requesting program is already waiting on this semaphore by the time the page arrives; in that case the V-operation namely brings about that the program is transferred from the list of the blocked ones to that of the executable ones. This goes awry, however, when at the moment of arrival of the page the program is not yet that far, is not yet up to the P-operation. By the time that this is the case, namely, that same place in the start store (and thereby that same semaphore) may long since have been used for another transport.

A way out of these difficulties is to accommodate the semaphore not with the start order but with the KIE of the core page frame in question. This is then indeed safe, but has the disadvantage that this will start to cost the necessary memory places. A programmed semaphore costs, namely, three memory places, and this would therefore mean roughly a doubling of the some fifty KIE's; this is all the sadder because normally no more than four of these semaphores are used.

We have decided to spare both cabbage and goat [to have it both ways] and to accommodate the semaphores that govern this blockade with the individual programs (gambling on the fact that we shall have considerably fewer programs in action than core page frames). Every program therefore has its own semaphore. The requesting of a sacred core page frame now means (the setting = 0 of the own semaphore, if the tooth-brushing has not already taken care of that) and the notifying of the drum organization via which semaphore the completion of the transport must this time be signalled back. Upon taking cognizance of the completion of the transport, the "drum-intervention reactor" must now raise not a fixed semaphore by possibly more than 1, but a little list of semaphores (possibly empty, possibly longer than 1) must all be raised by 1. To this end the own semaphores contain a link element, and with the start order is located the initial link. And now we see that the sanctity counter with the PRECOPY is actually superfluous; it is, namely, equal to the length of the semaphore chain that hangs on the start order concerned.

An essential supposition that underlies this last arrangement is that, per program, the request of a sacred page and the P-operation on the own semaphore alternate with one another in time: for if you first sacredly request the one page and then the other, and subsequently do 1 P-operation, then you no longer know which page has now arrived. This also permits us to make the own semaphore includable in only 1 single chain. Should, which I do not suppose, this "un-nested" use of the own semaphore lead to undesirable restrictions, then we can always still go over to a semaphore per core page frame.

4. Codings.

For the sake of completeness I shall now describe which conventions we have made about the manner in which the BZV's and the KIE's hold the information that they must house. This is not exciting at all; the decisions taken are sometimes utterly arbitrary, and when they are not, they are dictated by the purest opportunism.

4.1. The structure of the BZV.

A BZV occupies 1 word (with care no more, because we must enter one into the core memory for every existing page). A BZV knows four states.

1) BZVi = -0

This will mean "The page in question is sober".

2) BZVi = | d[26] = 1

| d[25] = 0

| d[24] . . . d[0] = start address drum page frame, padded on the left with zeros

This will mean that the page in question is not sober, is to be found at the indicated place on the drum and nowhere else; the page is moreover not involved in a transport.

3) BZVi =:BKPk (i.e. Base Core Page frame no. k), padded on the left with zeros.

This will mean that the page is not sober, is not involved in a transport, is indeed present on the cores, and indeed as

BKPk[0] through BKPk[511]

Note 1.: BKPk will be a multiple of 512.

Note 2. The base address of the corresponding KIEk is found by

:KIEk =:BPKk / 128 + KPC

(KPC = KIEtable Positioning Constant); a KIE therefore occupies four words. Note 3. State 3 will be characterized by BZVi greater than 0.

4) BZVi =:BPKk, padded on the left with zeros)

This will mean that the page is involved in a transport with respect to core page frame k.

Note. This state distinguishes itself from 1) by the fact that BZVi ≠ 0 and from 2) by the fact that

d[25] = 1.

The consequence of these conventions is that the fixed presence of a page follows entirely from the BZV, without consultation of the KIE; upon writing onto such a page the KIE must indeed be consulted to see whether this writing perhaps entails the transition from COPY to ORIGINAL. We can consider rendering this, in state 3, in bit d[25]; I actually feel little for this at the moment; selection of an array element is anyhow a considerable process and consulting the KIE is, again, not exactly insurmountable (the testing of such a higher bit also costs something).

4.2. The structure of the KIE.

KIEk[0] =
if coupled to a BZV, then
= :BZVi, padded on the left with zeros

else
= - 0

KIEk[1] = if drum reference applicable (implies non-free)

d[26] = 1.

d[25] = 0

d[24] . . . d[0] = start address drum page frame, padded on the left with zeros

else (i.e. drum reference not applicable)

if non-free, then = -0

else (i.e. if FREE)

if last page frame from free chain, then = +0

else =:KIEj (≠ 0), referring to the KIE of the next page frame from the free chain

KIEk[2] =
if COPY or ORIGINAL, then HHTk (< 128)

(= 0, 1, 2, 3, . . . .) HHT means "Sanctity counter".

if PRECOPY, then 128 + HHTk + 1024 * start order number

if PREFREE, then 256 + 1024 * start order number

if FREE, then 512.

KIEk[3] = strategic information (also called interest number).

4.3. Remarks.

During transport of an existing page frame KIEk and BZVi are therefore coupled to one another, also when the core page frame is PREFREE. During transport of a no-longer-existing page the core page frame is, in the nature of things, PREFREE, KIEk[0] = -0 and also KIEk[1] = -0. The murdering of a page that is involved in a transport entails, namely, immediate release of the drum page frame anyway. This is possible because the drum organization processes the transport orders in the order of offering; every later destination of this drum page frame therefore only becomes effective once the first transporting has fully finished.

We have not interspersed the core page frames with the corresponding KIE's, although this would save us the shift order for the reduction of KIEk to BKPk or vice versa. The reasons were:

- should ever a system of memory protection make its entry, then it is probable that a technique that lets core page frames begin on high powers of two will accord better with what becomes customary
- it could well be convenient for test purposes (perhaps initially we shall still conduct inspection)
- the reduction of physical address to core page frame and line number within this page frame is considerably facilitated.

Of the fact that the clean shift orders in a single register will be fast, we shall therefore make grateful use.

transcribed by Sam Samshuijzen

revised Tue, 18 Jul 2006
