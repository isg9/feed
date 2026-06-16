---
title: "Communicatiebuffering voor de EL-X8 - T.H.E. (EWD118)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD118.html
published: "1965-02-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD118.PDF
---

# Communicatiebuffering voor de EL-X8 - T.H.E.

EWD118 - 0

3 Februari 1965

Communicatiebuffering voor de EL-X8 - T.H.E.

Algemeen

De mogelijkheid communicatie-informatie te bufferen is een faciliteit van de installatie als geheel: de bufferruimte wordt dan ook niet bij speciale programma's ondergebracht, maar "universaliter", zoals de tabel van de SV's van de bibliotheek.

Dit doet ons leed, want het impliceert een permanente kerngeheugenbezetting voor de SV-lijst van de buffersegmenten. Toch moet het maar; wij denken aan, zeg 256 SV's (d.w.z. maximaal &Mac185; trommel vol gebufferde transput informatie).

Naast de 256 SV's zullen wij extra ruimte ter beschikking stellen om hierin kettingen van segmenten te rijgen.

Analyse van de Segment Controller heeft aangetoond, dat het niet ondoenlijk is om SV's te verhuizen; dit maakt het mogelijk, dat een producent een segment als privé segment vult, en als het vol is de informatie naar de bufferruimte overhevelt, zonder de informatie "vleselijk" te transporteren. Er hoeft slechts een SV verhuisd te worden

Invoer via bandlezers

De invoer via bandlezers behandelen we apart, omdat we deze tamelijk afwijkend bedienen.

De PM's nemen niet rechtstreeks contact met de bandlezers op, maar doen dit via een informatiestroom, waarvan de opbouw ver op de consumptie vóór mag zijn. Men kan denken dat de bandlezer zo snel is, dat dit nauwelijks de moeite waard is; wij moeten dit echter wel doen, omdat de buffering de enige manier is om de bandlezer zo gauw mogelijk vrij te maken.

Elke PM krijgt slechts de beschikking over een enkele papierband-informatiestroom; het feit dat er (wellicht) meer bandlezers aan de installatie zitten komt niet de individuele programma's, maar het samenspel ten goede.

Om deze impliciete invoerstroom hoeft een programma niet te vragen: hij was er kennelijk al, toen het programma ingevoerd ging worden.

Wij zagen twee wegen: òf de bandlezers in de bankiersalgorithme betrekken en van de programmeur verlangen, dat hij de interesse "afmeldt" wanneer hij zijn laatste gegeven heeft ingelezen òf het er maar op wagen, d.w.z. niet de programmeur laten verklaren dat hij niet meer lezen wil, en, omgekeerd, de bandlezers buiten het bankiersalgorithme te houden. Wij hebben - tegen alle regels in! - tot het laatste besloten. Om het ruimtebeslag in de buffer te mitigeren hebben we wel besloten, om in de informatiestroom van de lezers het bandbeeld gepakt te bergen. (Een pakking van 2000 karakters per segment geeft bij volle band van 300 meter à 400 kar. per meter 60 segmenten. Dat is natuurlijk niet mals, maar het zij zo.)

Wij hebben dit onder anderen gekozen, omdat wij niet veel vertrouwen hebben in de zo moeilijk afdwingbare plicht tot afmelden. (Wij doen ons best onze programmeurs niet te hoog aan te slaan!)

EWD118 - 1

Reservering van een bandlezer voor de opbouw van een bepaalde informatiestroom valt onherroepelijk af bij de detectie van "einde band".

Deze informatiestromen zijn objecten, die elk behoren bij een bepaalde PM. We kunnen ons het "handvat" onderin de stapel van de PM voorstellen. (Geheugenprotectie schept wel problemen, want de producent moet bij het handvat kunnen, maar de producent is tenslotte een stamgast, die moet weten wat hij doet.)

Over de gedetailleerde layout van het handvat doen we nu geen voorstel. Wij merken op, dat er onderscheid gemaakt moet worden tussen "informatiestroom nuchter" - d.w.z. niet eens in opbouw - en "informatiestroom leeg", d.w.z. het komt wel, maar er zit geen enkel segment in de bufferruimte. Het handvat bevat de seinpaal, via welke de producent toevoeging van één nieuw segment aan de consument doormeldt. Het handvat bevat tevens een identificatie van de producent - om vrij te kunnen maken in geval van desaster, b.v.

Plotter, printer & ponser

Wij nemen aan, dat een plotter, die aan een bepaald plaatje begonnen is, dit plaatje moet kunnen afmaken, voordat hij aan een ander plaatje begint. Voorts stellen wij, dat een plotterprent potentieel zo uitgebreid kan zijn, dat wachten met output, het programma "einde plaatje" heeft gemeld, prohibitief zou kunnen zijn. Hieruit volgt, dat met 1 physische plotter, geen PM via twee "abstracte" kanalen, twee - willekeurig uitgebreide - plaatjes kan gaan tekenen. Wij voorkomen dit door geen PM het uitdrukkingsvermogen te geven om aan meer dan een plaatje bezig te zijn. Bij voorkeur zal de plotter (zie onder) slechts aan plaatjes beginnen, waarvan de specificatie volledig voltooid is.

Ook voor bandponsers geldt, dat we bij voorkeur slechts een ponser aan het ponsen van een bandje zullen laten beginnen, als het bandbeeld geheel is opgebouwd. Als dit nl. zo is, kan de machine controleren, of er nog genoeg papier op de rol is. Moet een ponser aan een onaf bandbeeld beginnen, dan zal hij vragen om een nieuwe band in de ponser.

Tenslotte: de highspeed printer verwerkt van de programma's de informatie "formuliersgewijs". Als echter twee programma's met dezelfde snelheid formulieren aanbieden is het voor de receptie hoogst onaangenaam, als de formulieren om de ander komen. Ook hier hebben we dus zoiets, dat de formulier printer - zo mogelijk - affe teksten produceren.

Ondanks de verschillen zullen wij proberen, deze drie apparaten zo gelijk mogelijk te bespreken.

Het volgende beeld werd in eerste instantie voor de plotter(s) ontwikkeld. Wij zullen straks bekijken, in hoeverre het ook voor de andere gereedschap van toepassing is. Wij praten dus eerst even over de plotter(s).

Kwantitatief spelen twee aantallen een grote rol: het aantal physische apparaten en het aantal "documenten in opbouw". (Een "document" is voor een plotter een plaatje, voor een bandponser een noodzakelijkerwijze onverknipte band.)

EWD118 - 2

Wij stellen ons voor, dat een physisch apparaat niet aan het volgende document begint, voordat het vorige is voltooid. (Bij de plotter is dat niet strikt noodzakelijk!) Wij gaan er verder van uit, dat een document zo omvangrijk kan zijn, dat het onredelijk is om a priori te eisen, dat het in zijn totaliteit gebufferd moet kunnen worden, voordat aan de feitelijke uitvoer begonnen wordt. (Dit is voor de ponser weer wat restrictief, omdat de lengte van een physische band wel aan een bovengrens gebonden is.)

Voor een bepaald soort physisch apparaat voeren wij in "de uitvoerders" - in evenveelvoud als er physische apparaten zijn - en "de prae-uitvoerders". Elke prae-uitvoerder is de mogelijkheid om een eventueel "onbegrensd" document te gaan opbouwen. Prae-uitvoerders zijn geprogrammeerde instituten, hun aantal kan in overigens identieke installaties verschillen.

De programmeur kent het verschil tussen de prae-uitvoerder en de uitvoerder niet. Als hij als zijn behoefte opgeeft b.v. "1 plotter", dan wordt deze behoefte ten bate van de bankier vertaald in "1 praeplotter + 1 plotter". Als hij aangeeft, dat hij een document wil gaan opbouwen, dan wordt dit vertaald in de poging een prae-uitvoerder te lenen. Nu krijgt de PM een prae-uitvoerder toegewezen: hij kan nu het document opbouwen - mogelijk hierin geremd door een P-operatie op vrijkomende bufferruimte en tenslotte wordt van hem verwacht, dat hij het document "voltooid" meldt. Op dit moment komt de prae-uitvoerder vrij en kan een andere - of dezelfde - PM aan documentopbouw beginnen.

Als er n prae-uitvoerders en m echte uitvoerders zijn, spelen achter de schermen n + m overigens equivalente informatiestromen een rol: hiervan zijn op elk moment een ten hoogste n-tal "de prae-uitvoerders". (Deze som komt voort uit het volgende: elke physische uitvoerder kan bezig zijn met de consumptie van een inmiddels af document; daarnaast zijn n documenten in opbouw.)

Als een PM een document voltooid meldt, dan moeten wij twee getallen onderscheiden: of een feitelijke uitvoerder was reeds aan de consumptie van deze stroom begonnen of niet. Zo nee, dan wordt het voltooide, onbegonnen beeld aan de z.g. "documentenketting" gehangen. Naast genoemde informatiestromen die hoogstens één document in opbouw, in afbraak of beide bevatten, is er nl. nog een ketting van voltooide documenten. In deze ketting worden klare documenten opgenomen in volgorde van voltooiïng; in deze zelfde volgorde zal later aan hun feitelijke uitvoer begonnen worden. (N.b.: bij meer uitvoerders betekent dit niet, dat ze de een na de ander in volgorde uitgevoerd hoeven te worden!) Verder wordt een prae-uitvoerder vrijgegeven.

Als bij voltooiïng van het documentbeeld wel reeds een physische uitvoerder aan de feitelijke consumptie begonnen was, wordt de documentenketting met rust gelaten, maar wordt aan de bankier beëindiging van lening gemeld van niet alleen een prae-uitvoerder, amar ook van een uitvoerder. (Zie onder.)

Een vrij ledige uitvoerder handelt als volgt:

Als de documentenketting niet leeg is, zoekt hij uit de m + n informatiestromen een ongebruikte (die is er!) en initieert deze stroom met het kopdocument uit de ketting. Vervolgens gaat hij uit deze stroom consumeren.

EWD118 - 3

Als de documentenketting leeg is, kan hij, wanneer er wel documenten in opbouw zijn, proberen, of hij al vast aan de uitvoer van een onaf document beginnen kan. Het beginnen aan een onaf document betekent echter dat het opbouwende programma vanaf dat moment geacht wordt behalve een prae-uitvoerder ook een uitvoerder geleend te hebben. De uitvoerder mag zich dus slechts onder controle van de bankier aan een onaf document beginnen.

Als een uitvoerder zichzelf uit de documentenketting initieert, is dit een aflopende taak: de uitvoerder kan zich op daggeldbasis ter beschikking stellen en omarmingsrestricties worden hierdoor aanmerkelijk milder.

Het idee van de prae-uitvoerders is dus om zo mogelijk op daggeldbasis van de physische uitvoerders gebruik te maken; dit is des te interessanter naarmate de beschikbare bufferruimte groot is ten opzichte van voorkomende document omvangen.

Een en ander werpt wel strategische problemen op, speciaal: hoeveel prae-uitvoerders kiezen we en wanneer beslissen we, dat een uitvoerder begint aan de uitvoer van een onaf document. (De mogelijkehid zulks te doen wordt door de bankier afgeperkt: als het mogelijk is, hoeft de uitvoerder het echt nog niet te doen.)

Een enkel voorbeeld moge de strategische problemen nader illustreren. Stel, dat we in een installatie van een bepaald type 1 uitvoerder hebben, maar meer prae-uitvoerders en stel, dat de installatie geladen is met een programma, dat een niet ontiegelijk omvangrijk document heel traag vult (een z.g. "druppelaar"). Zolang de omvang van de gebufferde informatiestroom nog geen onbehoorlijke proporties heeft aangenomen, is het zaak, de uitvoerder nog niet aan de uitvoer te laten beginnen: dit kan de physische uitvoerder nl. nodeloos lang bezig houden. En inderdaad: terwijl de druppelaar moeizaam zijn document opbouwt, komt een extra programma, een "vluggertje", die eventjes een snel produceerbaar document wil uitvoeren; van de kant van de machine is dit een ideale "mix'. Het tweede programma gaat document produceren: als nu dit tweede document - b.v. omvangrijk - niet eerst in zijn geheel gebufferd kan worden, moet het physisch apparaat wegens gebrek aan bufferruimte op een gegeven moment aan een van beide documenten beginnen. Hoe besturen we, dat hij dan aan het document van het vluggertje begint?

Er zijn twee oplossingen: als een uitvoerder aan een onaf physisch document moet beginnen - wegens uitputting van de bufferruimte - laat hem dan in geval van keuze de langste informatiestroom kiezen: dit helpt het meeste. In boven beschreven situatie moet je dus hopen dat op het moment dat tot uitvoer besloten wordt, de producenten van het vluggertje de productie van de druppelaar inmiddels heeft ingehaald. (Je moet dus met het vluggertje wel "op tijd" komen: dat moet toch, want als de druppelaar definitief aan de uitvoerder gekoppeld is, dan heb je het gehad en komt het vluggertje toch al niet meer aan bod. Deze inhaaloverweging stelt de grens van "op tijd zijn" alleen wat scherper.)

Er is een andere techniek, nl. die der vrijheidsklassen. Hierin kent men aan elk programma een vrijheidsnummer toe: de bankier hanteert nu als nevenvoorwaarde, dat een programma met hoger vrijheidsnummer voor zijn voortzetting niet op de voortzetting van - d.w.z. vrijgave door - een programma uit een lagere vrijheidsklasse afhankelijk mag zijn. Door

EWD118 - 4

de druppelaar dan een lager vrijheidsnummer te geven dan het vluggertje forceert men dat

a) het vluggertje alleen geaccepteerd wrodt als de uitvoerder nog niet aan het document van de druppelaar begonnen is,

b) daarna de feitelijke uitvoer van het document van het vluggertje de voorrang krijgt.

Of de invoering van het vrijheidsgetal geen nadere complicaties met zich meebrengt - afgezien ervan, dat men het moet definiëren - moet een nadere analyse nu uitmaken.

Zonder invoering van het vrijheidsgetal kan men altijd (logisch) straffeloos een vrije uitvoerder aan minstens een van de onaffe documenten laten beginnen. Immers: dit geldt als een lening van de uitvoerder. Voordat deze koppeling tot stand gekomen is, was de toestand veilig, d.w.z. de bankier zag al zijn programma's op den duur eindigen, d.w.z. kon dit programma in een zekere volgorde verifiëren: het eerste prae-ponsende programma, waarvan hij de doorgang garanderen kan, mag dus ook de feitelijke uitvoerder op dit moment willen lenen. Houdt ook na introducite van het vrijheidsgetal deze redenering stand?

In dat geval onderzoekt de bankier eerst of de programma's uit de hoogste vrijheidsklasse door kunnen gaan. Hieronder kunnen er zijn, die wel prae-uitvoerder en uitvoerder geclaimd, maar nog niet geleend hebben. Vervolgens worden de programma's uit de daaronder liggende klasse onderzocht: hier kan het eerste prae-uitvoerende programma ontmoet worden, dat evenwel de uitvoerder zelf niet lenen mag, omdat dan een programma uit hogere vrijheidsklasse in de knoei zou kunnen komen.

Moraal is: zonder vrijheidsklasse kan altijd de uitvoerder aan minstens 1 niet leeg onaf document beginnen. Met vrijheidsklasse - en vrije prae-uitvoerders - is het mogelijk de uitvoerder voor een nog lege prae-uitvorder te reserveren!

Het voorgaande dekt vrij aardig de bandponser en de plotter: de highspeed printer is een ander chapiter.

Om te beginnen: het lijkt irreëel om voor de printer het aantal prae-uitvoerders lager te kiezen dan het aantal PM's ("een programma, dat niet print is nauwelijks een programma").

Als wij ons op het standpunt stellen, dat de printer snel is, dat de programmeur snel zijn gegevens naar buiten wil hebben, dan zou men zeggen: beschouw een formulier van de printer als "document", zodra dit klaar is, neem het dan in de ketting op. Anderzijds is dit onacceptabel: het scheur- en sorteerwerk, dat hierdoor voor de expeditie zou ontstaan is gewoon te erg. En dan ga je dus zeggen: beschouw aan elke prae-uitvoerder "N" formulieren als document. Maar hoe groot kies je "N"?

Het is duidelijk, dat beëindiging van het programma de prae-uitvoerder vrijgeeft en het staartje formulieren als document naar de ketting kan overhevelen. Dan is er ten duidelijkste werk voor de printer. Als de documentenketting leeg is, begint de printer (beginnen de printers) pas actief te worden als de totale omvang van de gebufferde informatie daartoe aanleiding geeft. Het ligt voor de hand, om dan uit een informatiestroom, die begint met het oudste document of de informatiestroom, die het grootste aantal formulieren bevat? Mijn neiging is om de tijdsfactor te verwaarlozen - er komen geen programmeurs in de machinekamer - en de langste informatiestroom te pakken.

Transcribed by Frank Steggink

Revised Tue, 6 Jan 2004.


---

## English translation

### Communication Buffering for the EL-X8 - T.H.E.

EWD118 - 0

3 February 1965

Communication Buffering for the EL-X8 - T.H.E.

General

The possibility of buffering communication information is a facility of the installation as a whole: the buffer space is therefore not assigned to special programs, but "universally", like the table of the library's SV's.

This grieves us, for it implies a permanent core-store occupation for the SV-list of the buffer segments. Yet so it must be; we have in mind, say, 256 SV's (i.e. at most &Mac185; drum full of buffered transput information).

Besides the 256 SV's we shall make extra space available in which to string chains of segments.

Analysis of the Segment Controller has shown that it is not impracticable to relocate SV's; this makes it possible for a producer to fill a segment as a private segment, and, when it is full, to transfer the information to the buffer space without transporting the information "in the flesh". Only an SV need be relocated

Input via tape readers

The input via tape readers we treat separately, because we serve it in a rather deviant manner.

The PM's do not make direct contact with the tape readers, but do so via an information stream whose build-up may run far ahead of consumption. One might think that the tape reader is so fast that this is hardly worth the trouble; we must, however, do this, because the buffering is the only way to free the tape reader as soon as possible.

Each PM is granted the disposal of but a single paper-tape information stream; the fact that there are (perhaps) more tape readers on the installation benefits not the individual programs, but the interplay.

A program need not ask for this implicit input stream: it was evidently already there when the program was about to be read in.

We saw two roads: either to involve the tape readers in the banker's algorithm and to require of the programmer that he "deregisters" his interest when he has read in his last datum, or simply to risk it, i.e. not to let the programmer declare that he no longer wishes to read, and, conversely, to keep the tape readers outside the banker's algorithm. We have - against all the rules! - decided on the latter. To mitigate the space claim in the buffer, we have indeed decided to store the tape image in packed form in the readers' information stream. (A packing of 2000 characters per segment gives, for a full tape of 300 metres at 400 char. per metre, 60 segments. That is of course not modest, but so be it.)

We have chosen this, among other reasons, because we have little confidence in the so hard-to-enforce duty to deregister. (We do our best not to rate our programmers too highly!)

EWD118 - 1

Reservation of a tape reader for the build-up of a particular information stream lapses irrevocably upon detection of "end of tape".

These information streams are objects, each belonging to a particular PM. We can picture the "handle" at the bottom of the PM's stack. (Memory protection does create problems, for the producer must be able to reach the handle, but the producer is after all a regular, who ought to know what he is doing.)

Concerning the detailed layout of the handle we make no proposal now. We remark that a distinction must be made between "information stream sober" - i.e. not even under build-up - and "information stream empty", i.e. it is coming, but there is not a single segment in the buffer space. The handle contains the semaphore by which the producer signals to the consumer the addition of one new segment. The handle also contains an identification of the producer - in order to be able to free it in case of disaster, for example.

Plotter, printer & punch

We assume that a plotter that has begun a particular picture must be able to finish this picture before it begins another picture. Furthermore we posit that a plotter print can potentially be so extensive that waiting with output until the program has reported "end of picture" could be prohibitive. From this it follows that with 1 physical plotter, no PM may draw two - arbitrarily extensive - pictures via two "abstract" channels. We prevent this by giving no PM the expressive power to be engaged on more than one picture. By preference the plotter (see below) will begin only on pictures whose specification has been fully completed.

For tape punches too it holds that we shall by preference let a punch begin punching a tape only when the tape image has been fully built up. For if this is so, the machine can check whether there is still enough paper on the roll. Should a punch have to begin on an unfinished tape image, then it will ask for a new tape in the punch.

Finally: the highspeed printer processes the programs' information "form by form". If, however, two programs offer forms at the same speed, it is most unpleasant for the reception if the forms come alternately. So here too we have something of the sort, that the form printer - if possible - produce finished texts.

In spite of the differences we shall try to discuss these three devices as uniformly as possible.

The following picture was in the first instance developed for the plotter(s). We shall presently consider to what extent it also applies to the other apparatus. So we shall first talk for a moment about the plotter(s).

Quantitatively two numbers play a large role: the number of physical devices and the number of "documents under build-up". (A "document" is, for a plotter, a picture; for a tape punch, a necessarily uncut tape.)

EWD118 - 2

We picture it thus, that a physical device does not begin on the next document before the previous one is completed. (For the plotter this is not strictly necessary!) We further proceed from the assumption that a document can be so voluminous that it is unreasonable to demand a priori that it must be capable of being buffered in its totality before the actual output is begun. (This is again somewhat restrictive for the punch, since the length of a physical tape is indeed bound to an upper limit.)

For a particular kind of physical device we introduce "the executors" - in as many copies as there are physical devices - and "the pre-executors". Each pre-executor is the possibility of building up a possibly "unbounded" document. Pre-executors are programmed institutions, their number may differ in otherwise identical installations.

The programmer does not know the difference between the pre-executor and the executor. If he states as his requirement e.g. "1 plotter", then this requirement is, for the banker's benefit, translated into "1 pre-plotter + 1 plotter". If he indicates that he wishes to build up a document, then this is translated into the attempt to borrow a pre-executor. Now the PM is assigned a pre-executor: he can now build up the document - possibly restrained in this by a P-operation on buffer space becoming free - and finally he is expected to report the document "completed". At this moment the pre-executor becomes free and another - or the same - PM can begin document build-up.

If there are n pre-executors and m real executors, then behind the scenes n + m otherwise equivalent information streams play a role: of these at every moment at most n are "the pre-executors". (This sum arises from the following: every physical executor can be engaged in the consumption of a meanwhile finished document; in addition there are n documents under build-up.)

When a PM reports a document completed, then we must distinguish two cases: whether an actual executor had already begun the consumption of this stream or not. If not, then the completed, unbegun image is hung onto the so-called "document chain". For besides the named information streams, which contain at most one document under build-up, under breakdown, or both, there is namely yet another chain of completed documents. Into this chain ready documents are taken up in order of completion; in this same order their actual output will later be begun. (N.B.: with more executors this does not mean that they must be executed one after the other in order!) Furthermore a pre-executor is released.

If, upon completion of the document image, a physical executor had already begun the actual consumption, then the document chain is left in peace, but to the banker the termination of the loan is reported of not only a pre-executor, but also of an executor. (See below.)

A free idle executor acts as follows:

If the document chain is not empty, it seeks out of the m + n information streams an unused one (there is one!) and initiates this stream with the head document from the chain. Thereupon it sets about consuming from this stream.

EWD118 - 3

If the document chain is empty, it can, when there are documents under build-up, try whether it can already begin on the output of an unfinished document. Beginning on an unfinished document, however, means that the building program is from that moment on deemed to have borrowed, besides a pre-executor, also an executor. The executor may therefore begin on an unfinished document only under control of the banker.

If an executor initiates itself out of the document chain, this is a terminating task: the executor can make itself available on a day-loan basis, and embracing restrictions are thereby considerably mitigated.

The idea of the pre-executors is thus to make use of the physical executors, where possible, on a day-loan basis; this is the more interesting the larger the available buffer space is in relation to occurring document sizes.

All this does raise strategic problems, especially: how many pre-executors do we choose, and when do we decide that an executor begins on the output of an unfinished document. (The possibility of doing so is fenced off by the banker: if it is possible, the executor really still need not do so.)

A single example may further illustrate the strategic problems. Suppose that in an installation of a particular type we have 1 executor, but more pre-executors, and suppose that the installation is loaded with a program that fills a not unreasonably voluminous document very slowly (a so-called "dripper"). As long as the size of the buffered information stream has not yet assumed improper proportions, it is advisable not yet to let the executor begin on the output: for this could keep the physical executor needlessly long occupied. And indeed: while the dripper laboriously builds up its document, an extra program comes along, a "quickie", which wants to output, just briefly, a quickly producible document; from the machine's side this is an ideal "mix". The second program sets about producing a document: if now this second document - e.g. voluminous - cannot first be buffered in its entirety, the physical apparatus must, for lack of buffer space, at a given moment begin on one of the two documents. How do we control matters so that it then begins on the quickie's document?

There are two solutions: if an executor must begin on an unfinished physical document - because of exhaustion of the buffer space - then let it, in case of choice, choose the longest information stream: this helps the most. In the situation described above one must thus hope that, at the moment the decision to output is made, the producers of the quickie have meanwhile caught up with the production of the dripper. (One must thus arrive "in time" with the quickie: that is necessary anyway, for once the dripper is definitively coupled to the executor, you have had it and the quickie does not get a turn anyway. This catching-up consideration only sets the boundary of "being in time" somewhat more sharply.)

There is another technique, namely that of freedom classes. In this one assigns to each program a freedom number: the banker now applies, as a side condition, that a program with a higher freedom number may not, for its continuation, depend on the continuation of - i.e. release by - a program from a lower freedom class. By

EWD118 - 4

then giving the dripper a lower freedom number than the quickie, one forces that

a) the quickie is accepted only if the executor has not yet begun on the dripper's document,

b) thereafter the actual output of the quickie's document gets priority.

Whether the introduction of the freedom number brings no further complications with it - apart from the fact that one must define it - a further analysis must now decide.

Without introduction of the freedom number one can always (logically) with impunity let a free executor begin on at least one of the unfinished documents. For: this counts as a loan of the executor. Before this coupling was established, the state was safe, i.e. the banker saw all his programs eventually terminate, i.e. could verify this in a certain order: the first pre-punching program whose passage he can guarantee may therefore also wish to borrow the actual executor at this moment. Does this reasoning hold up also after the introduction of the freedom number?

In that case the banker first examines whether the programs from the highest freedom class can proceed. Among these there may be some that have indeed claimed a pre-executor and an executor, but not yet borrowed them. Thereupon the programs from the class lying beneath are examined: here the first pre-executing program may be encountered, which may nevertheless not borrow the executor itself, because then a program from a higher freedom class might get into trouble.

The moral is: without a freedom class the executor can always begin on at least 1 non-empty unfinished document. With a freedom class - and free pre-executors - it is possible to reserve the executor for a still-empty pre-executor!

The foregoing covers the tape punch and the plotter rather nicely: the highspeed printer is another chapter.

To begin with: it seems unrealistic to choose, for the printer, the number of pre-executors lower than the number of PM's ("a program that does not print is hardly a program").

If we take the standpoint that the printer is fast, that the programmer wants his data out quickly, then one would say: regard a form of the printer as a "document", and, as soon as it is ready, take it up into the chain. On the other hand this is unacceptable: the tearing and sorting work that would thereby arise for the dispatch department is simply too dreadful. And so one comes to say: regard, for each pre-executor, "N" forms as a document. But how large do you choose "N"?

It is clear that termination of the program releases the pre-executor and can transfer the tail end of forms as a document to the chain. Then there is, most clearly, work for the printer. If the document chain is empty, the printer (the printers) only begins to become active when the total size of the buffered information gives cause for it. It is obvious, then, to take from an information stream - that which begins with the oldest document, or the information stream that contains the largest number of forms? My inclination is to neglect the time factor - no programmers come into the machine room - and to take the longest information stream.

Transcribed by Frank Steggink

Revised Tue, 6 Jan 2004.
