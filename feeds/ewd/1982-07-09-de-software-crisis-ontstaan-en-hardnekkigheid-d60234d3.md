---
title: "De software crisis, ontstaan en hardnekkigheid (EWD829)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD08xx/EWD829.html
published: "1982-07-09T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd08xx/EWD829.PDF
---

# De software crisis, ontstaan en hardnekkigheid

Titel:
De software crisis, ontstaan en hardnekkigheid.

Spreker:
prof.dr.Edsger W.Dijkstra

Burroughs Corporation/

Technische Hogeschool Eindhoven

Samenvatting:

Het ontstaan van de software crisis is een direct gevolg van
de onderschatting van het revolutionaire karakter van de moderne
computer, welks programmeeropgave ons confronteert met een intellectuele
uitdaging zonder precedent. De ontwikkeling van een
feilloos programma van behoorlijke omvang is een opgave waarvan
de moeilijkheidsgraad aanvankelijk schromelijk is onderschat; het
voortduren van de software crisis is grotendeels te wijten aan het
feit dat deze moeilijkheidsgraad over het algemeen tot op de huidige
dag ontkend wordt. Op de achtergrond van deze ontkenning zal nader
worden ingegaan.

De software crisis, ontstaan en
hardnekkigheid

Edsger W.Dijkstra

Burroughs Corporation/

Technische Hogeschool Eindhoven

Officieel bestaat de software crisis sinds october 1968;
toen werd zijn bestaan namelijk openlijk toegegeven op de NATO
Conferentie over "Software Engineering" die te Garmisch-Partenkirchen
gehouden werd. Maar hij bestond natuurlijk al veel
langer, want anders was die conferentie toen nooit georganiseerd.
En in het jaar onzes Heren 1982 bestaat hij nog steeds,
want anders was deze ASI leergang er niet aan gewijd. En dat
de software crisis nog wel een tijdje zal voortduren is niet
een gewaagde voorspelling. Deze voordracht zal hoofdzakelijk
gewijd zijn aan zijn ontstaan en zijn hardnekkigheid.

Op de vraag "Hoe is de software crisis ontstaan?" is een
heel makkelijk antwoord te geven. Waarom mislukken zoveel
softwareprojecten? Heel eenvoudig: omdat programmeren blijkbaar
iedere keer weer veel moeilijker is dan wij denken. Dit
onontkoombare antwoord is evenwel te makkelijk, want het roept
onmiddellijk de vraag op hoe de blijkbaar schromelijke onderschatting
van de moeilijkheidsgraad van de programmeeropgave
heeft kunnen ontstaan en blijkbaar zo'n taai leven kan hebben.

Voor de aanvankelijke onderschatting van de moeilijkheidsgraad
van de programmeeropgave is een heel duidelijke verklaring;
zij bestaat uit twee gedeelten. Ten eerste eiste in de beginjaren
de constructie van betrouwbare hardware alle aandacht op.
De electronica betrad het onontgonnen terrein van de digitale
techniek, daarbij tegelijkertijd geconfronteerd met een schaalvergroting
zonder precedent en ongehoorde betrouwbaarheidseisen.

Aan hardwareproblemen hadden we toen echt onze handen meer dan
vol en de waarschuwing dat, als de machine eenmaal werkte, het
gebruik niet triviaal zou zijn was het laatste, waar we behoefte
aan hadden. Als de waarschuwing geuit werd —en dat werd hij wel— , werd hij niet gehoord. Dit is maar al te begrijpelijk: het was immers niet de eerste zorg.

Ten tweede werd de waarschuwing slechts schuchter gegeven, niet alleen omdat er veel moed voor nodig is om een impopulaire boodschap van de daken te schreeuwen, maar ook omdat de feitelijke ervaring, die de waarschuwing zou moeten schragen, nog ontbrak. We wisten dat het eerst ontwikkelde programma —voor het genereren van een tabel van quadraten— meteen had gewerkt; we wisten ook dat het tweede programma —voor het genereren van een tabel van priemgetallen— dat tot ontnuchtering van zijn auteurs niet had gedaan. Maar met die ervaring ontmoedig je niet een hoopvolle mensheid. Het was pas hun tweede programma, die kerels hadden toch nog geen enkele ervaring, en, bovendien, wie is er nou in priemgetallen geinteresseerd? Het punt is, dat automatische rekenmachines toen inderdaad artefacten zonder enig precedent waren en dat de programmeeropgave inderdaad eisen aan ons intellect stelt, die er nog nooit aan waren gesteld. Zonder eerder aan een dergelijke discontinuiteit te zijn blootgesteld is het inderdaad heel moeilijk voorstelbaar dat iets, dat toch doenbaar moet zijn, onvoorstelbaar moeilijk uitpakt. We kunnen het naiveteit noemen, onschuld klinkt vriendelijker, maar over de aanvankelijke onderschatting hoeven we ons niet te verbazen.

Boeiender is de vraag, hoe het mogelijk is geweest, dat
deze onderschatting zo'n taai leven leidt. Langzamerhand is er
meer dan een kwart eeuw overweldigende experimentele ervaring
dat programmeren op zijn zachtst gezegd een uiterst verraderlijke
activiteit is, zo verraderlijk dat hij de inzet van ons

beste intellect rechtvaardigt. Maar de onderschatting duurt
voort tot op de huidige dag. Deze ASI leergang is er een symptoom
van: bij de voorbereidende sprekersbijeenkomst legde de
secretaris, Mr. W.L. den Dekker, uit dat het traditioneel
technisch/wetenschappelijke
accent van dit soort leergangen dit maal
"in verband met het onderwerp natuurlijk minder geprononceerd
zou zijn". Een verklaring van het taaie leven van de onderschatting
is ingewikkelder: een aantal oorspronkelijk onafhankelijke
oorzaken blijkt elkaar wederzijds te versterken.

Het staat buiten kijf dat, toen het phenomeen computer
zich kort na de tweede wereldoorlog aandiende, de internationale
wiskundige wereld er niet rijp voor was. Traditioneel bestaat
die wereld uit twee continenten, namelijk het wiskundig
werelddeel dat zich bezighoudt met de problemen van het continuum
en het deel dat zich op de discrete problematiek concentreert.
Het zo eminent toepasbare werk van Newton, nu zo'n 300
jaar oud, is voor de continue wiskunde een enorme stimulans geweest
en bij de stormachtige ontwikkeling van dat deel der wiskunde
is de discrete wiskunde ontegenzeggelijk op het tweede
plan geraakt. Maar de digitale rekenautomaat manipuleert per
definitie nu eenmaal objecten in een discreet universum. Voeg
hieraan toe, dat dit universum, hoewel groot, per definitie eindig
is, en men kan zich misschien voorstellen dat het merendeel
der wiskundigen van die generatie zich gemachtigd voelde het
phenomeen computer te negeren: discreet en eindig moest het immers
triviaal zijn. Men kan hun deze aanvankelijke beroepsblindheid
bijna vergeven. Helaas moeten wij constateren, dat in de
daaropvolgende 35 jaar deze beroepsblindheid nauwelijks is afgenomen
en dat de wiskundige, die de intellectuele uitdaging van
computing science serieus neemt, nog steeds een zeldzaamheid is
en dat de gemiddelde wiskundige in bekrompenheid voor zijn nietwiskundige
medeburgers niet onderdoet. Overheerst bij wiskundigen
eenmaal het gevoel dat computers niet de moeite waard zijn

om je mee bezig te houden, dan werkt dat vervolgens als een z.g.
"self-fulfilling prophecy": als de intellectueel het best geequipeerden
het vak links laten liggen, wordt het gebied bezet
door tweede- en derderangs mensen en na enige tijd kan de echt
knappe jongen zich nog moeilijker voorstellen dat daar een taak
voor hem ligt. Ligt een stuk van de oorzaak van de software
crisis bij het feit dat de wiskundige wereld door de bank genomen
het gebied heeft veronachtzaamd, het taaie leven van de
software crisis is mede het gevolg van het feit dat de wiskundige
wereld door de inmiddels ontstane omstandigheden in deze veronachtzaming
gesterkt wordt.

Naast deze culturele oorzaak is er ook een economische,
die ging spelen op het moment dat Sperry-Rand als eerste rekenmachinefabrikant
de UNIVAC als serieproduct op de markt bracht.
Die markt moest gecreeerd worden, en die markt is gecreeerd
door ondeskundigen, die over voldoende fondsen beschikten, gouden
bergen te beloven. De gewenste ondeskundigheid was rijkelijk
voorhanden in de top van grote bedrijven, die toen nog
over geld beschikten; voor de gouden bergen zorgden de verkoopafdelingen
der computerfabrikanten. Termen als "analytical
engine", "mathematical machine" en "instrumentelle Mathematik"
lieten er geen twijfel over bestaan waar in onze cultuur dit
soort apparatuur was ontstaan en waarschijnlijk thuishoorde.
Hoe verkoop je het dan een een administrateur? Door met een
groots opgezette campagne het wezenlijk wiskundige karakter
van de hele gebruiksproblematiek onder tafel te praten, programmeren
voor te stellen als iets dat iedereen in een drieweekse
cursus kan leren, kortom door zo hardnekking te verklaren
dat het goud der beloofde bergen massief is, dat genoeg
mensen het geloven. In de jaren die daar op volgden is er
voor het vak van programmeur volledig critiekloos geronseld.
Hier in den lande was MULO ruimschoots genoeg, maar in andere
landen was het geen haar beter. Het resultaat laat zich denken:

van de half-millioen professionele programmeurs, die de wereld
inmiddels telt is het merendeel van een incompetentie die elke
beschrijving tart. Maar hier ligt wel een verklaring van het
taaie leven van de software crisis: een incompetent arbeidsleger
van een half millioen ververs je niet een-twee-drie.

Heel pessimistische mensen betogen, dat de teerling is
geworpen en dat we nooit van de horde incompetente programmeurs
verlost zullen worden. Die sombere voorspelling is gebaseerd
op de volgende, ongetwijfeld juiste observatie. Om
met dit soort personeel er toch nog het minst slechte van te
maken heb je veel van dat personeel nodig, dat vervolgens alleen
maar werken kan in een organisatie zo strak, en ook zo
infantiliserend, dat de mensen die je eigenlijk zou moeten hebben
daar niet in passen. De observatie is ongetwijfeld juist;
het is ook duidelijk dat het geobserveerde verschijnsel mede
voor de taaihaid van het leven van de software crisis verantwoordelijk
is. De pessimistische conclusie, dat de software
crisis een eeuwig leven beschoren zal zijn lijkt mij evenwel
een beetje haastig.

De sterkte van dit pessimisme verschilt van mens tot mens;
over het algemeen is het in het bedrijfsleven sterker dan in
het onderwijs. Het verschilt ook van land tot land; over het
algemeen is het in de Verenigde Staten sterker dan in Europa.
Dit laatste is niet verwonderlijk, aangezien de aftakeling van
het onderwijs ginds nog verder is voortgeschreden dan hier.
Het loont de moeite, daar en passant even de aandacht op te
vestigen, omdat onze perceptie van wat de centrale problemen
van computing science zijn de neiging heeft sterk beinvloed te
worden door de Amerikaanse perceptie daarvan. Bij alle verhalen
over de noden van "the casual user" en de intellectuele beperkingen
van "the average programmer" moeten wij nimmer vergeten een
duidelijk onderscheid te maken tussen enerzijds de intrinsieke

problemen van computing science en anderzijds de moeilijkheden
veroorzaakt door het falend Amerikaans onderwijsbestel. Van
een academicus vanzelfsprekend vergen dat hij zijn moedertaal
in woord en geschrift voortreffelijk beheerst en effectief hanteert
geldt hier inmiddels als een beetje overdreven, in de
Verenigde Staten als absurd en irreeel. Alle Amerikaanse nadruk
op "readability", "self-documentation" en soortgelijke,
niet nader gespecificeerde deugden wordt daardoor wel begrijpelijk.

Waar het pessimisme overheerst, heeft dat een ongewenst
neveneffect. Het leidt er toe, dat de software crisis gezien
gaat worden als een ongeneeslijke ziekte, en gelijk wij allen
weten is de ongeneeslijke ziekte de bij uitstek vruchtbare
bodem voor kwakzalverij. En inderdaad: ik ken geen professie
met een zo grote kwakzalverdichtheid als de onze! Dit is een
aspect waar ik het merendeel van mijn academische collegae
in de Onderafdeling der Wiskunde en Informatica van de Technische
Hogeschool te Eindhoven geregeld aan moet herinneren.
Het is een verschijnsel, dat zij in hun eigen professie niet
meemaken en waarmee zij alleen geconfronteerd worden, wanneer
een onderwijskundige hun pad kruist. Maar wie met pek omgaat,
wordt er door besmet: je kunt van buitenstaanders nauwelijks
verwachten dat zij kaf en koren scheiden, en de hoge kwakzalverdichtheid
in onze professie maakt het voor de academische
wereld eens zo moeilijk het vak serieus te nemen.

Ligt de schuld van de charlatannerie bij de kwakzalver
of bij het publiek dat hij bedot? Wel, als dat publiek bedot
wil worden, ligt de schuld dunkt me daar. In het bedrijfsleven
is zulks dunkt me het geval. Met heel serieuze bedoelingen
karakteriseerde ik enige manieren die bij het orde scheppen in
wat anders onbeheerste chaos wordt veelbelovend leken en lanceerde
ik termen als "structured programming", "layers of

abstraction" en "separation of concerns". In de kortst mogelijke
tijd werden niet de bedoelingen, maar wel de kreten gemeengoed
en bevond ik me tegen wil en dank ingedeeld bij de goeroes.
Als het zelfs de serieuze wetenschapsman zo vergaat, kunt U
nagaan hoe moeilijk het is geen kwakzalver te worden.

Het bedrijfsleven is zo wanhopig, dat elke heilsboodschap er in gaat als koek. Aan heilsboodschappen is dan ook geen gebrek. Voor de wanhopige manager is het bijvoorbeeld een ontstellend geruststellende gedachte, dat de gebezigde programmeertaal de bron van al zijn ellende is. De stakker klampt zich vast aan de droom van de programmeertaal, waarin programmeren zo makkelijk is, dat het allemaal vanzelf goedkomt. De nieuwe programmeertalen als panacee gaan in de kwakzalverskraam over de toonbank als verse broodjes bij de bakker. In dit verband berucht is de IBM-advertentie in Datamation, 1968, waarin een stralende Susie Meyer —in kleuren!— verklaart, dat de bekering tot PL/I het einde van al haar programmeerproblemen was. Hoe de arme Susie Meyer er een paar jaar later uitzag, vermeldt de historie helaas niet, maar het laat zich raden, want het wondermiddel heeft natuurlijk niet gewerkt. Wie de propagandaliteratuur voor Ada —de programmeertaal, die door het Amerikaanse ministerie van defensie wordt gepousseerd— leest, moet constateren, dat de wereld in 14 jaar weinig is veranderd. (In Polen wordt Ada, terecht en onthullend, als "PL/II" aangeduid.)

Naast de potjes met programmeertalen hebben we de flesjes
met nieuwe operating systems: op een regime van driemaal daags
een druppel UNIX wordt U het eeuwige leven beloofd. En als
dat niet werkt, dan is er nog het onfeilbare poeiertje van de
"personal computer". Dat is helemaal heerlijk, want zonder dat
U zich iets van al die andere patienten hoeft aan te trekken,
kan dit poeiertje gemengd worden helemaal naar Uw persoonlijke
behoeften.

Het laatste en gevaarlijkste luchtkasteel in deze windhandel
is het "intelligent terminal" waartegen je "gewoon" kunt
zeggen, wat je nodig hebt. Hier ligt de fraude er duimen dik
boven op: de meeste mensen weten niet wat ze nodig hebben, en
als ze het al zouden weten, kunnen ze het niet zeggen, noch "gewoon",
noch "ongewoon". Desalniettemin schijnt de Japanse industrie
te staan te trappelen om het te leveren, en wij zullen
wel stom genoeg zijn om het te kopen. Ik denk, dat het daarmee
net zo gaat als met homeopathische geneesmiddelen: je weet nooit
of het helpt en, "Baat het niet, dan schaadt het niet.".

Deze laatste geruststellende overweging is overigens een
ernstige misvatting. De z.g. "word processor" wordt aangeprezen
met de overweging, dat het zo makkelijk is om iets in je tekst
te verbeteren. Maar elke stroomlijning van het correctieproces
is een staande invitatie voor de productie van dingen, die deze
correctie nodighebben. Het is een volslagen illusie, dat de
"word processor" de auteur helpt. Weinigen immers kunnen de
verleiding weerstand bieden, maar iets te produceren, dat dan
later wel gerepareerd of geperfectioneerd zal worden. Maar uit
een rammelend concept is door bijvijlen nog nooit iets goeds ontstaan.
Met een tekst die niet deugt, kun je maar een ding doen:
naar de prullemand verwijzen. Als ik de Nederlandse overheid
was, zou ik het gebruik van dit soort apparatuur niet verbieden;
ik zou het zwaar belasten, net als alcohol en tabak.

De hoge kwakzalverdichtheid is mede verantwoordelijk voor
de hardnekkigheid van de software crisis. Elke keer dat de patient
bij een volgend lapmiddel geen baat heeft gevonden, wordt
hij argwanender jegens de medische wetenschap. De onvervulde
beloftes van volledige genezing maken hem doof voor wat de wetenschap
verantwoord geven kan: suggesties ter verbetering van
de toestand.

Laat mij evenwel niet zo in mineur eindigen. De toestand
is inderdaad hopeloos als het wetenschappelijk onderwijs zich
laat koejeneren door de eis van "maatschappelijke relevantie".
Die leidt er namelijk toe dat de maatschappij krijgt waar hij
om vraagt en dat, wanneer om lapmiddelen gevraagd wordt, er
lapmiddelen worden geboden. Het "Mundus vult decipi; decipiatur
ergo." is dan onverdund van toepassing. De toestand is evenwel
niet hopeloos als het wetenschappelijk onderwijs niet voor zijn
leidinggevende taak terugdeinst en de moed heeft de wereld te
bieden, niet waar de wereld om vraagt, maar wat de wereld nodig
heeft.

Over de hele wereld is de toestand aan de universiteiten
tegenwoordig zo benard, dat op hen je hoop vestigen onverantwoord
optimisme moet lijken. De wortels van onze universitaire traditie
gaan echter zo diep, dat ik er van overtuigd ben dat die traditie
een taaier leven beschoren zal zijn dan de software crisis van
vandaag.

Nuenen, 4 juli 1982

transcribed by Hans Terlouw

revised
Sat, 9 Jan 2010

---

## English translation

### The software crisis, origin and tenacity

Title:
The software crisis, origin and tenacity.

Speaker:
prof.dr. Edsger W. Dijkstra

Burroughs Corporation/

Eindhoven University of Technology

Summary:

The origin of the software crisis is a direct consequence of the underestimation of the revolutionary character of the modern computer, whose programming task confronts us with an intellectual challenge without precedent. The development of a flawless program of reasonable size is a task whose degree of difficulty was at first grievously underestimated; the persistence of the software crisis is largely to be ascribed to the fact that this degree of difficulty is, generally, denied to the present day. The background of this denial will be gone into further.

### The software crisis, origin and tenacity

Edsger W. Dijkstra

Burroughs Corporation/

Eindhoven University of Technology

Officially the software crisis has existed since October 1968; for it was then that its existence was openly admitted at the NATO Conference on "Software Engineering" held at Garmisch-Partenkirchen. But it of course existed much longer, for otherwise that conference would never have been organized then. And in the year of Our Lord 1982 it still exists, for otherwise this ASI course would not have been devoted to it. And that the software crisis will persist for a while yet is no daring prediction. This lecture will be devoted chiefly to its origin and its tenacity.

To the question "How did the software crisis arise?" a very easy answer can be given. Why do so many software projects fail? Quite simply: because programming is, apparently, every single time much harder than we think. This unavoidable answer is, however, too easy, for it immediately raises the question how the apparently grievous underestimation of the degree of difficulty of the programming task can have arisen and can apparently have such a tough life.

For the initial underestimation of the degree of difficulty of the programming task there is a very clear explanation; it consists of two parts. In the first place, in the early years the construction of reliable hardware demanded all the attention. Electronics entered the unbroken terrain of digital technique, thereby at the same time confronted with a scaling-up without precedent and with unheard-of reliability requirements.

With hardware problems we then really had our hands more than full, and the warning that, once the machine worked, its use would not be trivial was the last thing we had any need of. If the warning was uttered —and it was— it was not heard. This is only too understandable: it was, after all, not the first concern.

In the second place, the warning was given only timidly, not only because much courage is needed to shout an unpopular message from the rooftops, but also because the actual experience that should support the warning was still lacking. We knew that the first program developed —for the generation of a table of squares— had worked straight away; we also knew that the second program —for the generation of a table of prime numbers— had, to the disillusionment of its authors, not done so. But with that experience you do not discourage a hopeful mankind. It was only their second program, those fellows had after all no experience whatsoever, and, besides, who is now interested in prime numbers? The point is that automatic computers were indeed, at the time, artefacts without any precedent, and that the programming task does indeed make demands on our intellect such as had never been made on it. Without having earlier been exposed to such a discontinuity, it is indeed very hard to imagine that something which after all must be doable turns out to be unimaginably difficult. We can call it naivety; innocence sounds friendlier; but at the initial underestimation we need not be amazed.

More fascinating is the question how it has been possible that this underestimation leads such a tough life. By now there is more than a quarter-century of overwhelming experimental experience that programming is, to put it mildly, an extremely treacherous activity, so treacherous that it justifies the deployment of our best intellect. But the underestimation persists to the present day. This ASI course is a symptom of it: at the preparatory speakers' meeting the secretary, Mr. W.L. den Dekker, explained that the traditional technical/scientific emphasis of this kind of course would this time "of course be less pronounced, in connection with the subject". An explanation of the tough life of the underestimation is more complicated: a number of originally independent causes turn out mutually to reinforce one another.

It is beyond dispute that, when the phenomenon of the computer presented itself shortly after the Second World War, the international mathematical world was not ripe for it. Traditionally that world consists of two continents, namely the mathematical part that concerns itself with the problems of the continuum and the part that concentrates on the discrete problematic. The so eminently applicable work of Newton, now some 300 years old, has been an enormous stimulus to continuous mathematics, and in the stormy development of that part of mathematics, discrete mathematics has unquestionably ended up in the second rank. But the digital computer manipulates, by definition, objects in a discrete universe. Add to this that this universe, though large, is by definition finite, and one can perhaps imagine that the majority of the mathematicians of that generation felt entitled to ignore the phenomenon of the computer: discrete and finite, it surely had to be trivial. One can almost forgive them this initial professional blindness. Alas, we must observe that in the following 35 years this professional blindness has scarcely diminished, and that the mathematician who takes the intellectual challenge of computing science seriously is still a rarity, and that the average mathematician yields nothing in narrow-mindedness to his non-mathematical fellow citizens. Once the feeling predominates among mathematicians that computers are not worth occupying oneself with, that then works as a so-called "self-fulfilling prophecy": if the intellectually best-equipped leave the field alone, the area is occupied by second- and third-rate people, and after some time the really clever lad can still less easily imagine that there a task lies for him. If a part of the cause of the software crisis lies in the fact that the mathematical world has by and large neglected the area, the tough life of the software crisis is partly the consequence of the fact that the mathematical world is, by the circumstances since arisen, strengthened in this neglect.

Besides this cultural cause there is also an economic one, which came into play at the moment when Sperry-Rand, as the first computer manufacturer, brought the UNIVAC onto the market as a series product. That market had to be created, and that market was created by the incompetent, who had sufficient funds at their disposal, by promising mountains of gold. The desired incompetence was abundantly available at the top of large companies, which then still had money at their disposal; for the mountains of gold the sales departments of the computer manufacturers took care. Terms such as "analytical engine", "mathematical machine" and "instrumentelle Mathematik" left no doubt as to where in our culture this kind of apparatus had arisen and probably belonged. How, then, do you sell it to an administrator? By, with a grandly mounted campaign, talking the essentially mathematical character of the whole usage problematic under the table, presenting programming as something that everyone can learn in a three-week course, in short by so persistently declaring that the gold of the promised mountains is solid that enough people believe it. In the years that followed there was, for the trade of programmer, completely uncritical recruitment. Here in this country the MULO was amply sufficient, but in other countries it was no whit better. The result may be imagined: of the half-million professional programmers the world by now counts, the majority are of an incompetence that defies all description. But here lies an explanation of the tough life of the software crisis: an incompetent labour army of half a million you do not refresh in a twinkling.

Very pessimistic people argue that the die is cast and that we shall never be rid of the horde of incompetent programmers. That gloomy prediction is based on the following, undoubtedly correct, observation. To make, with this kind of personnel, the least bad of it after all, you need a lot of that personnel, which can then only work in an organization so rigid, and also so infantilizing, that the people you really ought to have do not fit into it. The observation is undoubtedly correct; it is also clear that the observed phenomenon is partly responsible for the toughness of the life of the software crisis. The pessimistic conclusion, however, that the software crisis is destined for an eternal life seems to me a bit hasty.

The strength of this pessimism differs from man to man; in general it is stronger in business than in education. It also differs from country to country; in general it is stronger in the United States than in Europe. The latter is not surprising, since the deterioration of education over there has progressed even further than here. It is worth the trouble to draw attention to it in passing, because our perception of what the central problems of computing science are has a tendency to be strongly influenced by the American perception of them. With all the stories about the needs of "the casual user" and the intellectual limitations of "the average programmer", we must never forget to make a clear distinction between, on the one hand, the intrinsic problems of computing science and, on the other hand, the difficulties caused by the failing American educational system. To require of an academic, as a matter of course, that he have an excellent command of his mother tongue in speech and writing and handle it effectively counts here by now as a little exaggerated, in the United States as absurd and unrealistic. All the American emphasis on "readability", "self-documentation" and similar, unspecified, virtues thereby becomes understandable.

Where pessimism predominates, that has an undesirable side effect. It leads to the software crisis coming to be seen as an incurable disease, and, as we all know, the incurable disease is the pre-eminently fertile soil for quackery. And indeed: I know no profession with so great a quack-density as ours! This is an aspect of which I have regularly to remind the majority of my academic colleagues in the Sub-department of Mathematics and Informatics of the Eindhoven University of Technology. It is a phenomenon they do not experience in their own profession and with which they are confronted only when an educationalist crosses their path. But whoever deals with pitch is defiled by it: you can scarcely expect outsiders to separate the chaff from the wheat, and the high quack-density in our profession makes it doubly hard for the academic world to take the subject seriously.

Does the guilt of the charlatanry lie with the quack or with the public he dupes? Well, if that public wants to be duped, the guilt lies, it seems to me, there. In business that, it seems to me, is the case. With very serious intentions I characterized some ways that seemed promising in the creation of order in what is otherwise unbridled chaos, and I launched terms such as "structured programming", "layers of abstraction" and "separation of concerns". In the shortest possible time not the intentions but the slogans became common property, and I found myself, willy-nilly, classed among the gurus. If even the serious man of science fares thus, you can imagine how hard it is not to become a quack.

Business is so desperate that every gospel of salvation goes down with it like cake. Of gospels of salvation there is then also no shortage. For the desperate manager it is, for instance, a startlingly reassuring thought that the programming language used is the source of all his misery. The poor wretch clings to the dream of the programming language in which programming is so easy that it all comes right of its own accord. The new programming languages as panacea go over the counter in the quack's booth like fresh bread rolls at the baker's. In this connection notorious is the IBM advertisement in Datamation, 1968, in which a beaming Susie Meyer —in colour!— declares that conversion to PL/I was the end of all her programming problems. How poor Susie Meyer looked a couple of years later history, alas, does not record, but it may be guessed, for the miracle cure of course did not work. Whoever reads the propaganda literature for Ada —the programming language pushed by the American department of defence— must observe that the world has changed little in 14 years. (In Poland Ada is, rightly and revealingly, designated "PL/II".)

Besides the little pots of programming languages we have the little bottles of new operating systems: on a regimen of three times a day a drop of UNIX you are promised eternal life. And if that does not work, then there is still the infallible little powder of the "personal computer". That is altogether delightful, for without your having to bother about all those other patients, this little powder can be mixed entirely to your personal needs.

The last and most dangerous castle in the air in this swindle is the "intelligent terminal" to which you can "just" say what you need. Here the fraud lies inches thick on top: most people do not know what they need, and even if they did know it, they cannot say it, neither "just" nor "unjust". Nonetheless the Japanese industry seems to be champing at the bit to deliver it, and we shall probably be stupid enough to buy it. I think it goes with it just as with homeopathic remedies: you never know whether it helps and, "If it does no good, it does no harm.".

This last reassuring consideration is, by the way, a serious misconception. The so-called "word processor" is praised with the consideration that it is so easy to correct something in your text. But every streamlining of the correction process is a standing invitation for the production of things that need this correction. It is a complete illusion that the "word processor" helps the author. For few can resist the temptation to produce just something that will later be repaired or perfected. But out of a rickety draft nothing good has ever yet arisen through touching up. With a text that is no good you can do only one thing: refer it to the wastepaper basket. If I were the Dutch government, I should not forbid the use of this kind of apparatus; I should tax it heavily, just like alcohol and tobacco.

The high quack-density is partly responsible for the tenacity of the software crisis. Each time the patient has found no benefit from a next palliative, he becomes more suspicious of medical science. The unfulfilled promises of complete cure make him deaf to what science can responsibly give: suggestions for the improvement of the condition.

Let me, however, not end so in a minor key. The condition is indeed hopeless if scientific education lets itself be bullied by the demand of "social relevance". For that leads to society getting what it asks for, and that, when palliatives are asked for, palliatives are offered. The "Mundus vult decipi; decipiatur ergo." then applies undiluted. The condition is, however, not hopeless if scientific education does not shrink from its leading task and has the courage to offer the world, not what the world asks for, but what the world needs.

All over the world the condition at the universities is nowadays so dire that to set one's hope on them must seem irresponsible optimism. The roots of our university tradition, however, go so deep that I am convinced that that tradition is destined for a tougher life than the software crisis of today.

Nuenen, 4 July 1982
