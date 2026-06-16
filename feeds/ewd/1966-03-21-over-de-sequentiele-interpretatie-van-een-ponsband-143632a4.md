---
title: "Over de sequentiele interpretatie van een ponsband (EWD157)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD157.html
published: "1966-03-21T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD157.PDF
---

# Over de sequentiele interpretatie van een ponsband

Over de sequenti‘le interpretatie van een ponsband.

HEP

Als fundamentele bouwsteen beschouw ik de integer procedure, die ik HEP noem. Bij het lezen van n-gats band levert hij in eerste instantie een van 21n mogelijke waarden af, nl van en met 0 (voor blank tape) tot en met 21n-1 (voor all holes) en wel steeds de waarde die overeenkomt met de volgende n-ade.

Hoe we aan de band komen is een onderdeel van de initialisering van HEP, waarover ik me liever niet wilde uitlaten. Ik neem dus aan, dat we weten, over welke band we het hebben. Die kunnen we dan n-ade voor n÷ade aftasten: die band kan dan wel opraken!

De vraag rijst direct, hoe HEP moet reageren, als "einde band" gedetecteerd wordt.

Mijn voorstel is om het arsenaal van mogelijke, door HEP af te leveren waarden uit te breiden met een speciale waarde (ik denk aan 511 of -1), die "EINDE BAND" zal betekenen. Het is dan aan het programma om vast te stellen, of het nu genoeg gelezen heeft; zo nee, dat zal HEP opnieuw geinitialiseerd moeten worden (band aanvragen).

Onmiddellijk rijst de vraag, hoe HEP reageert, als nadat "EINDE BAND" als waarde is afgeleverd, HEP weer wordt aangeroepen, voordat de volgende initialisatie heeft plaatsgehad.

Nu kunnen we drie kanten uit.

- we kunnen dit interpreteren als letaal deraillement van het programma - we kunnen dit interpreteren als de indicatie, dat er dus blijkbaar meer informatie gevraagd wordt en als onderdeel van HEP impliciet de initieringsriten plegen (om nieuwe band vragen etc.) - we kunnen weer "EINDE BAND" afleveren.  Omdat ik HEP bechouw als mijn meest fundamentele bouwsteen prefereer ik het laatste; het maakt het gebruik van HEP mogelijk zonder na elke aanroep te inspecteren, of "EINDE BAND" soms voorkwam "X:= HEP * HEP" kan nu. Een band wordt nu een halfrechte, oneindig lang in de toekomst, vanaf een zeker moment komen er alleen maar "EINDE BAND"'s op voor.

De routine HEP is van toepassing, als we als band willen verwerken, bv. de output van een registrerend meetapparaat. De rest van deze notitie beschouwt de inhoud van de ponsband als derivaat van een paginabeeld: hier ligt een code aan ten grondslag, bv. telexcode of FLEXOWRITER code.

CHAR

In het volgende zal ik een en ander toelichten aan de hand van Flexowriter code; voor telexcode is een analoge beschouwing te houden, zolang er geen losse carriage returns zonder line feed gegeven worden.

En ik moet me zelfs beperken tot een vrij speciale Flexowriter: een Flexowriter met vaste spatiering, zonder backspace waarbij de tabulatorstoppen op vaste posities staan. (En dit apparaat zullen wij zelfs nog iets moetfin idealiseren: over de minimum afstand, waarin de tabulatie "het nog haalt", bv. 1 niet, twee wel, moetem we een vaste afspraak maken.)

CHAR nu is een routine, die zich interesseert voor het paginabeeld: het is de functie van CHAR om bij het aftasten van een band een verslaglegging te doen, die een÷eenduidig is toegevoegd aan het met de band corresponderende paginabeeld.

Ik stel me bij de definitie van het paginabeeld op het standpunt, dat

- meervoudige underlining niet van enkelvoudige underlining is te onderscheiden - UHX meervoudige "stroking" niet van enkelvoudige "stroking" is te onderscheiden - dat de op de wagen ingestelde afstand van de regelopvoer niet tot het paqinabeeld behoort, een extra regel blank wel - dat op het papier de plaats van de linker kantlijn bekend is (wie elke regel begint met een extra spatie fokt een ander paginabeeld)  Ik stel me voor, dat CHAR bij herhaald aanroepen dit verslag zal afleggen, daarbij het paginabeeld regel voor regel verslaand (in volgorde van boven naar beneden) en elke regel op zichzelf van links naar rechts.

Over de wijze van verslaglegging door CHAR zuJlen we enige conventies moeten treffen.

Ik stel me voor, dat CHAR niet elke regel met spaties tot een vaste maximum lengte hoeft aan te vullen, maar in plaats daarvan NLCR wel tot de output van CHAR te laten behoren; CHAR heeft dan wel de plicht om eventuele TABS en 5PATIES aan het einde van de regel te onderdrukken.

Voor de verwerking van de TAB's stel ik voor ze te vertalen in een aequivalent aantal spaties: op deze wijze geeft een reeks CHAR-waarden zonder NLCR een beeld, dat invariant is voor zijn plaats op de reqel. Wel zal CHAR voor het bepalen van deze aequivalentie de wagenpositie moeten bijhouden.

Wat betreft de underlining en de stroking kunnen we verschillende wegen bewandelen.

- elke "zichtbare, transporterende ponsing" bestaat in vier versies, al af niet met underlining, al of niet met stroking - een aanroep van CHAR kan wel "underlining" en "stroking" separaat doormelden. In dit geval moet, om der wille van de eenduidigheid een vaste afspraak gemaakt worden over de volgorde, waarin deze door CHAR worden doorgemeld (bv, eerst de underlining, dan de stroke en dan het transporterende karakter).  Omdat CHAR dan toch verder moet kijken dan zijn neus lang is, prefereer ik de eerste mogelijkheid. Deze heeft de voordelen, dat

- we van de willekeurige afspraak over de volgorde verlost zijn - nu met elke aanroep van CHAR een positieverplaatsing van de wagen correspondeert.  CHAR heeft de eigenschap, dat mist de code bekend is, elke produceerbare band gelezen kan worden en over elke produceerbare pagina een uniek verslag wordt afgelegd.

Een willekeurige Flexowriterpaqina is niet als willekeurige Telexpagina te interpreteren, noch omgekeerd. Ik stel daarom voor, om de codeonafhankelijke CHAR af te schaffen en zoveel CHAR's in te voeren, als we codes (i.e. apparaten) hebben. Dus CHARF voor de verslaglegging over een Flexowriterpagina en CHART over de verslaglegging over een Telexpagina. (En. Wie naast de beschreven Flexowriter ook een Flexowriter met variabele spatiering etc. wil kunnen bespelen, zal daarvoor een andere CHARF, zeg CHARFV, moeten maken.)

De toevoeging van de integerreeks, die CHARF aan een Flexowriterpagina toekent heeft niets te maken met de integerreeks, die CHART aan een telexpagina toekent, Ik beschouw op dit niveau Flexowriterpagina en Telexpaginas nl. als onvergelijkbare voorwerpen.

Kijk, CHARF mag twee Flexowriters, die alleen daarin verschillen, dat de ene een recbt lettertype en de ander een cursief lettertype heeft, als aenquivalent beschouwen (mits natuurlijk beide lettertypes het zelfde onderscheidende vermogen -zoals verschil tussen de letter O en het cijfer 0 hebben).

CHART mag twee teleprinters met de ALCOR symbolen als aequivalent beschouwen, ook al heeft de ene alphabet van kleine letters en de andere een alpnabet wan hoofdletters .

CHARF en CHART leggen verslag af over het paginabeeld en moeten elke paqina, die op het betrokken apparaat vervvaardigbaar is, uniek verslaan; dit heeft nog niets te maken met ALGOL basic symbols! Er is een aparte beslissing nodig om de Flexowriter "A" resp. "a" op papier te gebruiken als afbeelding van het basic symbol "A" resp. "a". Evenzo zal apart beslist moeten worden, dat het telexcharacter, dat verdacht veel lijkt op een van de gebruikelijke afbeeldingen ven de eerste letter van het alphabet, ge•dentificeerd kan worden met hetzij het ALGOL basic symbol "A", dan wel het ALGOL basic symbol "a". Hoe deze beslissing uitvalt is een kwestie, die helemaal los staat van het feit, of teleprinters meestal met kleine, dan wel meestal met hoofdletters zijn uitgerust!

Ik zou mij kunnen voorstellen, dat het practisch zal blijken om het bereik van CHARF en dat van CHART een lege doorsnede te geven!

HEP is het aangewezen inspectiemiddel, als de programmeur in het bandbeeld ge•nteresseerd is, CHARF en CHART zijn de aangewezen inspectiemiddelen, als dp programmeur in het paginabeeld ge•nteresseerd is. Een programmeur, die, bij wijze van spreken in manuscript een ALGOL÷programma aanbiedt, is niet in het bandbeeld geinteresseerd, noch in de verwerking van het paginabeeld: zijn manuscript heeft in zijn prive-hardware representatie (nl. zijn handschrift en notatieconventies) een reeks basic symbols van ALGOL 60 en hij verwacht van de ponskamer (ik zou hier nog liever willen "typekamer"), dat deze reeks basic symbols op een van de voorhanden zijnde paginabeeldmakers (Flexowriters of teleprinters) wordt afgebeeld in overeenstemming met de specifieke regels, die voor elk van deze apparaten met in acht name van hun beeldend vermogen gekozen zijn. Wie aldus een text heeft aangeboden, mag niet protesteren als waar hij "Boolean" heeft geschreven, hij "boolean" on zijn papier vindt, tenminste niet, wanneer het laatste een van de volgens afspraak toelaatbare representaties is van het basic symbol, dat vermoedelijk bedoeld is. Was het niet zijn bedoeling om een ALGOL programma aan te bieden, maar een op een Flexowriter produceerbare text, dan is een dergelijke vrijheid natuurlijk niet veroorloofd (en waarschijnlijk doet hij er dan beter aan om zelf zijn tekst te Flexowriteren).

Fungeert het paginabeeld slechts als "carrier" voor een reeks ALGOL basic symbols, dan is het aangewezen inspectiemiddel een integer procedure "SYM". Het waardebereik van SYM heeft niets te maken met dat van CHARF en CHART, het onderscheidt waarschijnlijk alle basic symbols van ALGOL 60 (met uitzondering van het overbodige wordtteken?) en waarschijnlijk nog wel wat meer, NLCR bv.

SYM is een integer procedure, die heel nadrukkelijk slechts in enkelvoud staat; dus geen 5YMF of 5YMT.

Het afzondelijke bestaan van CHARF en CHART is gerechtvaardigd omdat wij hiermee paginabeelden kunnen aftasten, vervaardigd op apparaten met heel verschillende (Onvergelijkbare) beeldende vermogens. CHARF en CHART fungeren als optische aftasters; dat de facto de machine via een bandbeeld toegang tot het paginabeeld krijgt, is een omstandigheid, waarvan bij het gebruik van CHARF en CHART geheel geabstraheerd is, evenals van het feit, dat het feitelijke bandbeeld, dat slechts als "carrier" voor het paginabeeld fungeert, door het laatste niet eenduidig bepaald is. Aan deze abstractie ontlenen CHARF en CHART juist hun bruikbaarheid. De toevoeging van paginabeeld aan bandbeeld is bepaald door de eigenschappen van de respectievelijke apparaten.

Bij het gebruik van SYM fungeert, zoals gezegd, het paginabeeld op zijn beurt als carrier voor een reeks ALGOL basic symbols; de toevoeging van deze reeks aan een paginabeeld is nu bepaald door de conventies van de typekamer. Zoals CHARF en CHART abstraheren van het niet eenduidiq bepaalde bandbeeld, Zo abstraheert SYM van het niet eenduidig bepaalde paqinabeeld. Het is deze abstractie, waaraan SYM zijn enorme bruikbaarheid ontleent; het is deze abstractie, die de programmeur ontheft van de plicht het paqinabeeld aanslag voor aanslag, (spatie voor spatie!) te preciseren, het is tevens deze abstractie, die de ponskamer de vrijheid geeft in de keuze van het ponsapparaat en de layout.

Alle genoemde routines hebben hun stilzwijgend veronderstelde "own variables". Voor HEP is dit, waar welke band wordt afgetast, voor CHARF en CHAHT zijn dit de heersende case shift, de wagenpositie etc, voor SYM is dit onder meer de heersende code, dwz. of hij via CHARF, dan wel CHART moet werken. Omdat de afbeeldingsregels voor basic symbols op een pagina apparaatafhankelijk zijn, in deze heersende code de statusvariabele van SYM; dit in dan ook de reden, waarom ik CHAR heb afgeschaft.

Boven gegeven uitweidingen zijn de vrucht van een mislukking. Toen ik HEP, CHARF en CHART qeconcipieerd had, ben ik gaan denken over wat men doen moet, als een programma, na via, zeg, CHARF een stuk band gelezen te hebben, via HEP een stukje doorleest, om daarna desnoods via CHARF weer door te kunnen lezen. Een vraag, die opkomt, zolang men CHARF beschouwt als een speciale manier van bandlezen, een vraag, die zinloos wordt, zodra men CHARF beschouwt als een mechanisme tot paginaaftasting. Zolang men CHARF beschouwt als een speciale manier van bandlezen en dus probeert om aan de alternering van CHARF en HEP een betekenis toe te kennen, duiken er allerlei vieze problemen op, waarvoor ik geen overtuigende oplossing heb kunnen vinden. Ik heb nog geprobeerd om naast HEP een "HEPF" en een "HFPT" in te voeren, die de plicht zouden hebben, voor CHARF, resp CHART de passerende heptade aan te kijken en de relevante updating van de statusinformatie van CHARF resp. CHART te verrichten. Nou, dit is mislukt, speciaal omdat er geen duidelijke manier is om de beide sequenti‘le processen -het aftasten van een band en het aftasten van een uit deze band afgeleid paginabeeld-, hoewel ze macroscopisch gezien wel synchroon lopen (het begin van de band komt overeen met het begin van het paginabeeld, microscopisch door een zinvolle (dwz. hanteerbare) conventie te synchroniseren. Ik kom dan ook tot de conclusie, dat het gemengde gebruik van HEP en CHAR een hinken op twee gedachten is, waarin je met de ene hand probeert terug te nemen, wat je met de andere gegeven hebt (nl. dat het preciese bandbeeld er niet toe doet). Ten aanzien van dit gemengde gebruik zou ik dan ook het standpunt willen innemen: of verbieden (en de implementatie hierop laten testen) of er bij zeggen, dat het undefined is, en dat de gebruiker wilde reacties mag verwachten, onder het motto "What else do you expect?".

transcribed by Jan Berend Dusink
revised Mon, 25 Oct 2010

---

## English translation

### On the sequential interpretation of a punched tape

On the sequential interpretation of a punched tape.

HEP

As the fundamental building block I take the integer procedure that I call HEP. On reading n-hole tape it delivers, in the first instance, one of 2↑n possible values, namely from 0 inclusive (for blank tape) up to and including 2↑n-1 (for all holes), and always the value that corresponds to the next n-ad.

How we come by the tape is part of the initialisation of HEP, on which I would prefer not to commit myself. I therefore assume that we know which tape we are talking about. We can then scan it n-ad by n-ad: that tape may well run out!

The question immediately arises as to how HEP must react when "end of tape" is detected.

My proposal is to extend the arsenal of possible values to be delivered by HEP with a special value (I am thinking of 511 or -1), which will mean "END OF TAPE". It is then up to the program to establish whether it has now read enough; if not, that HEP will have to be re-initialised (request tape).

The question immediately arises as to how HEP reacts when, after "END OF TAPE" has been delivered as a value, HEP is called again before the next initialisation has taken place.

Now we can go three ways.

- we can interpret this as lethal derailment of the program - we can interpret this as the indication that evidently more information is therefore being requested, and as part of HEP implicitly perform the rites of initiation (request new tape, etc.) - we can deliver "END OF TAPE" again.  Because I regard HEP as my most fundamental building block, I prefer the last; it makes the use of HEP possible without inspecting after every call whether "END OF TAPE" perhaps occurred "X:= HEP * HEP" is now possible. A tape now becomes a half-line, infinitely long into the future; from a certain moment onward only "END OF TAPE"'s occur on it.

The routine HEP is applicable when we wish to process as a tape, e.g., the output of a recording measuring instrument. The remainder of this note regards the contents of the punched tape as a derivative of a page image: there is a code underlying it, e.g. telex code or FLEXOWRITER code.

CHAR

In what follows I shall elucidate a number of matters by reference to Flexowriter code; for telex code an analogous treatment can be held, as long as no loose carriage returns without line feed are given.

And I must even restrict myself to a rather special Flexowriter: a Flexowriter with fixed spacing, without backspace, in which the tabulator stops are at fixed positions. (And this device we shall even have to idealise somewhat further: concerning the minimum distance over which the tabulation "still makes it", e.g. 1 does not, two does, we have to make a fixed agreement.)

CHAR now is a routine that concerns itself with the page image: it is the function of CHAR, on scanning a tape, to produce a report that is one-to-one assigned to the page image corresponding to the tape.

In defining the page image I take the position that

- multiple underlining cannot be distinguished from single underlining - UHX multiple "stroking" cannot be distinguished from single "stroking" - that the line-feed distance set on the carriage does not belong to the page image, but an extra blank line does - that on the paper the position of the left margin is known (whoever begins every line with an extra space breeds a different page image)  I imagine that CHAR, on repeated calls, will render this report, reporting the page image line by line (in order from top to bottom) and each line in itself from left to right.

Concerning the manner of reporting by CHAR we shall have to make some conventions.

I imagine that CHAR need not fill out every line with spaces to a fixed maximum length, but instead lets NLCR belong to the output of CHAR; CHAR then does have the duty to suppress any TABS and SPACES at the end of the line.

For the processing of the TAB's I propose to translate them into an equivalent number of spaces: in this way a sequence of CHAR-values without NLCR gives an image that is invariant with respect to its place on the line. CHAR will, however, have to keep track of the carriage position in order to determine this equivalence.

As regards the underlining and the stroking we can travel various roads.

- every "visible, transporting punching" exists in four versions, with or without underlining, with or without stroking - a call of CHAR can also report "underlining" and "stroking" separately. In this case, for the sake of unambiguity, a fixed agreement must be made about the order in which these are reported by CHAR (e.g., first the underlining, then the stroke and then the transporting character).  Because CHAR then has to look further than its nose is long anyway, I prefer the first possibility. This has the advantages that

- we are rid of the arbitrary agreement about the order - now with every call of CHAR a positional displacement of the carriage corresponds.  CHAR has the property that, provided the code is known, every producible tape can be read and over every producible page a unique report is rendered.

An arbitrary Flexowriter page cannot be interpreted as an arbitrary Telex page, nor conversely. I therefore propose to abolish the code-independent CHAR and to introduce as many CHAR's as we have codes (i.e. devices). Thus CHARF for the reporting on a Flexowriter page and CHART on the reporting on a Telex page. (And. Whoever, besides the described Flexowriter, also wishes to be able to play a Flexowriter with variable spacing etc., will have to make for that another CHARF, say CHARFV.)

The assignment of the integer sequence that CHARF allots to a Flexowriter page has nothing to do with the integer sequence that CHART allots to a telex page. At this level I regard Flexowriter page and Telex pages, namely, as incomparable objects.

Look, CHARF may regard two Flexowriters which differ only in that the one has an upright typeface and the other an italic typeface as equivalent (provided, of course, that both typefaces have the same distinguishing power -such as the difference between the letter O and the digit 0).

CHART may regard two teleprinters with the ALCOR symbols as equivalent, even if the one has an alphabet of small letters and the other an alphabet of capital letters.

CHARF and CHART render a report on the page image and must report uniquely on every page that is producible on the device concerned; this has as yet nothing to do with ALGOL basic symbols! A separate decision is needed to use the Flexowriter "A" resp. "a" on paper as the image of the basic symbol "A" resp. "a". Likewise it will have to be separately decided that the telex character, which looks suspiciously much like one of the usual images of the first letter of the alphabet, can be identified with either the ALGOL basic symbol "A" or the ALGOL basic symbol "a". How this decision turns out is a matter that stands entirely apart from the fact whether teleprinters are usually equipped with small, or usually with capital letters!

I could well imagine that it will prove practical to give the range of CHARF and that of CHART an empty intersection!

HEP is the appropriate means of inspection when the programmer is interested in the tape image; CHARF and CHART are the appropriate means of inspection when the programmer is interested in the page image. A programmer who, so to speak, submits in manuscript an ALGOL program, is interested neither in the tape image nor in the processing of the page image: his manuscript has, in his private hardware representation (namely, his handwriting and notational conventions), a sequence of basic symbols of ALGOL 60, and he expects of the punching room (here I would still rather have "typing room") that this sequence of basic symbols be imaged on one of the page-image makers at hand (Flexowriters or teleprinters) in accordance with the specific rules that have been chosen for each of these devices with due regard to their imaging power. Whoever has thus submitted a text may not protest if, where he has written "Boolean", he finds "boolean" on his paper, at least not when the latter is one of the agreed admissible representations of the basic symbol that is presumably intended. Were it not his intention to submit an ALGOL program, but a text producible on a Flexowriter, then such a freedom is of course not permissible (and he probably does better then to Flexowrite his text himself).

If the page image serves merely as "carrier" for a sequence of ALGOL basic symbols, then the appropriate means of inspection is an integer procedure "SYM". The value range of SYM has nothing to do with that of CHARF and CHART; it probably distinguishes all basic symbols of ALGOL 60 (with the exception of the superfluous word-marker?) and probably some more besides, NLCR for instance.

SYM is an integer procedure that very emphatically stands only in the singular; thus no SYMF or SYMT.

The separate existence of CHARF and CHART is justified because by these we can scan page images produced on devices with very different (incomparable) imaging powers. CHARF and CHART function as optical scanners; that de facto the machine obtains access to the page image via a tape image is a circumstance from which, in the use of CHARF and CHART, complete abstraction is made, as it is from the fact that the actual tape image, which serves merely as "carrier" for the page image, is not uniquely determined by the latter. It is precisely from this abstraction that CHARF and CHART derive their usefulness. The assignment of page image to tape image is determined by the properties of the respective devices.

In the use of SYM the page image, as has been said, functions in its turn as carrier for a sequence of ALGOL basic symbols; the assignment of this sequence to a page image is now determined by the conventions of the typing room. As CHARF and CHART abstract from the not-uniquely-determined tape image, so SYM abstracts from the not-uniquely-determined page image. It is this abstraction from which SYM derives its enormous usefulness; it is this abstraction that relieves the programmer of the duty to specify the page image keystroke by keystroke (space by space!); it is at the same time this abstraction that gives the punching room the freedom in the choice of the punching device and the layout.

All the routines mentioned have their tacitly assumed "own variables". For HEP this is which tape is being scanned where; for CHARF and CHART these are the prevailing case shift, the carriage position, etc.; for SYM this is among other things the prevailing code, that is, whether it must work via CHARF or via CHART. Because the imaging rules for basic symbols on a page are device-dependent, this prevailing code is the status variable of SYM; this is then also the reason why I have abolished CHAR.

The digressions given above are the fruit of a failure. When I had conceived HEP, CHARF and CHART, I began to think about what one must do if a program, after having read a piece of tape via, say, CHARF, reads on a little via HEP, in order afterwards, if need be, to be able to read on again via CHARF. A question that arises as long as one regards CHARF as a special manner of tape reading, a question that becomes meaningless as soon as one regards CHARF as a mechanism for page scanning. As long as one regards CHARF as a special manner of tape reading and thus tries to assign a meaning to the alternation of CHARF and HEP, all sorts of nasty problems crop up, for which I have not been able to find a convincing solution. I even tried to introduce, alongside HEP, a "HEPF" and a "HEPT", which would have the duty, for CHARF resp. CHART, to look at the passing heptad and to perform the relevant updating of the status information of CHARF resp. CHART. Well, this failed, especially because there is no clear manner to synchronise the two sequential processes -the scanning of a tape and the scanning of a page image derived from this tape-, although macroscopically they do run in step (the beginning of the tape corresponds to the beginning of the page image), microscopically by means of a meaningful (that is, manageable) convention. I therefore come to the conclusion that the mixed use of HEP and CHAR is a limping on two thoughts, in which with the one hand you try to take back what with the other you have given (namely, that the precise tape image does not matter). With respect to this mixed use I would therefore wish to take the position: either forbid it (and have the implementation tested for this) or add that it is undefined, and that the user may expect wild reactions, under the motto "What else do you expect?".

transcribed by Jan Berend Dusink
revised Mon, 25 Oct 2010
