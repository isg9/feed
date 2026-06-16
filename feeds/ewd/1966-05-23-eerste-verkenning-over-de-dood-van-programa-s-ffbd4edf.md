---
title: "Eerste verkenning over de dood van programa’s (EWD163)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD163.html
published: "1966-05-23T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD163.PDF
---

# Eerste verkenning over de dood van programa’s

EWD 163

Eerste verkenning over de dood van programma's.

1. De processen, die door de CM's worden uitgevoerd, zijn onsterfelijk. Dit rapportje handelt dus duidelijk over een PM-aangelegenheid.

2. Een PM heeft een onsterfelijk buitenste blok, het PM-blok. Dit wordt nooit verlaten, het is in dit blok, dat de in- en uitvoerstromen thuishoren; evenzo hoort hier thuis de statusinteger "runnumber". De PM bevindt zich in dit blok, wanneer op standaardband gewacht wordt. (Deze toestand, ook in een statusvariabele voor de bandlezerselectie vastgelegd, zou ook in "runnumber = 0" kunnen worden vastgelegd.) Zodra een vermeende standardband wordt gevonden, wordt het runnumber ingevuld; als het een algolprogramma blijkt wordt op grond van de buiten het programma vermelde pluncherbehoefte de acceptabiliteit getest; zo ja, dan gaat de PM een binnenblok in, waarin de globalen voor de vertaler gedeclareerd worden, wordt de vertaler als procedure aangeroepen en als we daaruit terugkomen (en het programma was goed vertaald) dan roepen we het programma aan. Komen we hier helemaal legitiem uit terug, dan doet de standaardtekst van de PM de blokexit van het ingegane blok, we zijn terug op het niveau van het PM-blok. De eigen informatie (segmenten, heiligingen etc.) van het algolprogramma zijn hiermee afgehandeld, de PM hoeft niet meer te doen, dan de in- en uitvoerstromen tot hun nuchtere staat terug te brengen. Als er nog iets in een invoerstroom zit (of zit te komen), dan moet "skip until end of tape" gedaan worden (inclusief "geen interesse meer", zie de tape reader CM), als in een pluncherstroom een unfinished document zit, moet dit worden voltooid en aangehaakt, als in de printerstroom nog een unfinished form zit (hier zit een WC-rol conventie: ik neem aan, dat deze erin zit, zo er ueberhaupt van de printer gebruik gemaakt is) wordt deze als de laatste gemarkeerd en "number of final documents nfd" wordt met 1tje opgehoogd. (Dit even met Coen opnemen.)

Nadat alle stromen zijn afgemaakt, zet de PM zijn behoeften op 0 (inclusief repercussies voor verdachte plunchers), zet het runnumber op 0 en gaat op standaardband staan wachten.

Dit is, zoals ik me de natuurlijke dood van een programma voorstelde. In verband met schijndood en de gewelddadige dood (zie onder) komt er nog wel wat bij.

3. De operateur heeft de mogelijkheid (waarvoor moge de hemel weten) om een programma schijndood te verklaren. Het moet dan op een zacht pitje zitten wachten, totdat het weer, via de console, wordt gewekt.

Het schijndood maken is een bevel, gericht tot een programma in een PM; als er geen 1tje in zit (runnumber = 0) dan lijkt het commando me Non Applicable. Ook als het programma, dat er in deze PM zit, al schijndood is. Waarschijnlijk zullen we voor een aanwezig programma ook de boolean "op sterven" hebben; dan moet er ook maar gestorven worden en ik dacht, dat het bevel tot schijndood ook dan NA moest wezen.
De complicatie bij schijndood is, dat het programma dat zachte pitje bij voorkeur wel plaatst op een onschuldig punt. Wij hadden zo gedacht "niet in een rode sectie". Als een programma wit en uitvoerbaar is, dan kan het commando tot schijndood meteen verwezenlijkt worden: men boekt het als wachtend op een voor dit doel in elke PM voorkomende seinpaal. Anders moeten we alleen het verzoek tot schijndood boeken (weer een toestand, waarin het commando "val schijndood" als NA behandeld zou kunnen worden).

Deze blokkering van schijndood moet achterwege blijven, als de PM rood is (hij moet nog doorwerken) en is niet lekker codeerbaar, als de PM al op een andere seinpaal staat te wachten. De uitgestelde schijndoodheid moet effectief worden bij de witmakende V-operatie en de voltooide P-operatie. (Opmerking van Piet: de schijndood hoeft pas effectief te worden als je dreigt de controle aan een wit proces metterdaad te gunnen. Dit geldt ongeacht de reden tot uitstel en zou tot compactere codering aanleiding kunnen geven.)

De wekoperatie is nu makkelijk. Als de schijndood gevraagd, maar nog niet gerealiseerd is, kan worden volstaan de notitie weg te nemen. Als de aanvraag niet hangt, moet de schijndood gerealiseerd zijn (anders NA) en de message interpreter kan op de daartoe strekkende seinpaal een V-operatie uitvoeren. (Let wel, dat het veilig lijkt om voor de schijndood een specifieke seinpaal per PM in te voeren !)

4. De gewelddadige dood komt vanuit een andere AM (hetzij van een CM, die ontdekt, dat een ponser kapot is, hetzij via de message interpreter van de operateur, misschien ook van de PM zelf, die ontdekt, dat hij "onmogelijk verder kan". Ik zie dit nog niet zo direct, zo mogelijk openhouden.)
Het grote probleem van de gewelddadige dood is, dat we dit programma moeten beÎindigen, dwz. er een punt aan moeten draaien, zodat het toch nog well-behaved blijft. Een eerste vereiste is, dat het in elk geval zover doorgaat (zo nodig), dat ook de toestand "schijndood" zou mogen ingaan. Dit geeft ons een nieuw criterium om vast te stellen, waar roodheid begint en waar roodheid moet ophouden: buiten roodheid moet de PM in een hanteerbare toestand achterblijven. Nadat we nu straks hebben laten zien, wat we ons bij dat hanteren voorstellen, moeten we de stukken PM-programma, waarin roodheid voorkomt en die we inmiddels gemaakt hebben, in dit licht nog eens duchtig nakijken.

Er is een communicatieapparaat, dat hier wat roet in het eten doet, en dat is de trommel. Ik wil een programma liever niet vermoorden, als het nog op een seinpaal staat te wachten, in het bijzonder niet, wanneer het aan de segment controller een segment gevraagd heeft. Als we af kunnen spreken, dat sterven niet begint, zolang de PM nog op een seinpaal wacht, dan kunnen we dit als volgt spelen (NB. Het aanvragen van een segment aan de segment controller gaat buiten alle roodheid om: het kan in roodheid, het kan in witheid). Bij de opdracht aan de segment controller kan de PM een boolean "Aanvraag Segment" true zetten.

De structuur in de PM wordt dan

AS := true

aanvrage van segment

V(SCS)

AS := false; P(eigen segment seinpaal)

De laatste twee statements staan op een regel, om heel nadrukkelijk af te spreken, dat ze in doofheid na elkaar gebeuren. Nu spreken we af, dat de initialisering van de zelfmoordriten niet plaats kan vinden, als AS. Hiermee bereiken we, dat de aangevraagde pagina's er zijn, voordat ze wellicht vernietigd zijn. (Ik schreef pagina's, ik bedoelde segmenten.)

Zo te zien, kunnen we met 1 variabele "killvar" al een heleboel doen. Tentatief zou ik deze de volgende waarden met de volgende betekenis willen geven (De op pag.0 gesuggereerde codering van X "runnummer = 0" kan dan vervallen.)

killvar = 0
De PM is leeg

killvar = 1
De PM is bezig met een programma, dat volslagen levend is

killvar = 2
Er is aanvrage tot schijndood

killvar = 3
Het programma is schijndood (hangt op de seinpaal killsem)

killvar = 4
Er is aanvrage tot zelfmoord

killvar = 5
Het programma is tot de zelfmoordriten overgegaan.

(Of de boolean AS hier ook in gecodeerd moet worden, bv. door killvar = 8 is voor mij nog een open vraag.)

Ik stel me nu op het standpunt, dat het agens, dat de zelfmoordopdracht geeft, hiervan via de consoleteleprinter melding maakt, maar dat we de PM (of liever: het er in zittende programma) niet de gelegenheid geven, nog een brief aan vrienden en bekenden ten afscheid te schrijven, dus geen rapportering over de status, warin het programma zich bevond, toen tot de zelfmoordriten werd overgegaan. (Hoogstens voeren we nog in

killvar = 6
Het programma is op eigen initiatief tot de zelfmoordriten overgegaan, in tegenstelling tot killvar = 5. Op de laatste printerform kan dan nog een kreet komen te staan "ik heb mezelf beeindigd", dan wel "ik ben er afgetrapt".)

Ik hoopte, dat de overgang tot de zelfmoordriten geeffectueerd zou kunnen worden door in de status van het onderbroken programma zo te knoeien, dat zo ongeveer met het mechanisme van de General Goto de besturing springt naar het punt in het PM-blok, waar de PM de stromen gaat afmaken. Dit is niet genoeg, want er kunnen vanwege de SIS twee programmasegmenten heilig zijn - en de general goto ontheiligt er maar 1tje, als de moord plaatsvindt wanneer een arraysegment heilig is aangevraagd, dan zitten we daar ook nog mee. (De moord wordt wel uitgesteld, totdat dit segment gearriveerd is). Ik kan me voorstellen, dat een en ander bij het heilig aanvragen van arraysegmenten een extra notitie vergt en dat de overgang tot de zelfmoordriten zo min mogelijk door de coordinator en zoveel mogelijk door de PM zelf gebeurt. Het wegwerken van een stukje SIS (ontheiliging en zo) is nl. een handeling, die bij de dynamische fout onder auspicien van de PM zelf zal gebeuren.

5. De dynamische fout.

Na ampele overwegingen zijn we tot de conclusie gekomen, dat de reactie op een dynamische fout een zuivere PM-aangelegenheid is. Ik stel me zo voor, dat deze alleen tijdens killvar = 1 zal kunnen voorkomen (in rode stukken wil ik helemaal niet zulke griezelige dingen kunnen doen). De PM zal dan de stapel moeten afbreken totaan de eerste Working Space Pointer; als de fout in een SIS optreedt, dan moet hij dit door stapelinspectie ontdekken, als er nog een arraysegment heilig is, dan weet het foutendetecterende mechanisme dat, en kan dat arraysegment ontheiligd worden. Is de stapel teruggebracht tot het eerste niveau, waarop SE1 aangeroepen kan worden, dan zal dat moeten gebeuren om een aanroep van de procedure ALARM te kunnen simuleren. Met de sprong daarnaar toe wordt de laatste programmapagina ontheiligd, ALARM kan onder zich de stapel zoveel inspecteren als hij wil en kan tenslotte springen naar het punt in het PM-blok, waar we na programmabeÎindiging terecht komen.

De microscopie van het mieren aan de status onder in de HAM-kamer en het toespelen naar een tussengeschoven systementry is een spelletje, dat me wel boeit; ik beschik zelf niet over de kennis om direct te zien, hoe dit moet, ik dacht wel, dat het kon en wil er graag over helpen denken.

EWD.

transcribed by Sam Samshuijzen

revised Sun, 14 Jan 2007

---

## English translation

### First exploration concerning the death of programs

EWD 163

First exploration concerning the death of programs.

1. The processes that are executed by the CM's are immortal. This little report therefore clearly deals with a PM-matter.

2. A PM has an immortal outermost block, the PM-block. This is never left; it is in this block that the input and output streams belong; likewise the status integer "runnumber" belongs here. The PM resides in this block when standard tape is being awaited. (This state, also recorded in a status variable for the tape reader selection, could also be recorded in "runnumber = 0".) As soon as a presumed standard tape is found, the runnumber is filled in; if it turns out to be an algol program then, on the basis of the punch requirement stated outside the program, acceptability is tested; if so, then the PM enters an inner block, in which the globals for the translator are declared, the translator is called as a procedure, and when we return from it (and the program was correctly translated) then we call the program. If we return from this entirely legitimately, then the standard text of the PM performs the block exit of the entered block, and we are back at the level of the PM-block. The program's own information (segments, sanctifications etc.) of the algol program has hereby been disposed of; the PM now need do no more than restore the input and output streams to their sober state. If something is still in an input stream (or is still coming in), then "skip until end of tape" must be done (including "no further interest", see the tape reader CM); if an unfinished document is in a punch stream, this must be completed and hooked on; if an unfinished form is still in the printer stream (here there is a toilet-roll convention: I assume that one is in it, if the printer has been used at all) it is marked as the last one and "number of final documents nfd" is incremented by one. (To be taken up with Coen shortly.)

After all streams have been finished off, the PM sets its requirements to 0 (including repercussions for suspect punches), sets the runnumber to 0 and goes and waits for standard tape.

This is how I imagined the natural death of a program. In connection with suspended animation and the violent death (see below) there is still rather more to come.

3. The operator has the possibility (for what purpose heaven only knows) of declaring a program to be in suspended animation. It must then sit waiting on a low flame, until it is awakened again, via the console.

The making suspended-animate is a command, directed at a program in a PM; if there is no one in it (runnumber = 0) then the command seems to me Non Applicable. Likewise if the program that is in this PM is already in suspended animation. Probably for a program present we shall also have the boolean "dying"; in that case dying may as well take place too, and I thought that the command to suspended animation should then also be NA.
The complication with suspended animation is that the program preferably places that low flame at an innocent point. We had thought along the lines of "not in a red section". If a program is white and executable, then the command to suspended animation can be realized immediately: one books it as waiting on a semaphore occurring in every PM for this purpose. Otherwise we must only book the request for suspended animation (again a state in which the command "fall into suspended animation" could be treated as NA).

This blocking of suspended animation must be omitted if the PM is red (it must work on further) and is not nicely codable if the PM is already waiting on another semaphore. The deferred suspended-animateness must become effective at the whitening V-operation and the completed P-operation. (Remark by Piet: the suspended animation need only become effective when you are about to actually grant control to a white process. This holds regardless of the reason for deferral and could give occasion for more compact coding.)

The wake operation is now easy. If the suspended animation has been requested but not yet realized, it suffices to remove the note. If the request is not pending, then the suspended animation must have been realized (otherwise NA) and the message interpreter can perform a V-operation on the semaphore serving this purpose. (Note well that it seems safe to introduce, for the suspended animation, a specific semaphore per PM!)

4. The violent death comes from another AM (either from a CM, which discovers that a puncher is broken, or via the message interpreter of the operator, perhaps also from the PM itself, which discovers that it "cannot possibly continue". I do not yet see this so directly; keep it open if possible.)
The great problem of the violent death is that we must terminate this program, i.e. must put an end to it, so that it nevertheless remains well-behaved. A first requirement is that it in any case proceeds far enough (if necessary) that the state "suspended animation" too may be entered. This gives us a new criterion for establishing where redness begins and where redness must cease: outside redness the PM must be left in a manageable state. After we shall presently have shown what we envisage by that handling, we must, in this light, once more thoroughly inspect the pieces of PM-program in which redness occurs and which we have meanwhile made.

There is a communication device that throws a bit of a spanner in the works here, and that is the drum. I would rather not murder a program while it is still waiting on a semaphore, in particular not when it has requested a segment from the segment controller. If we can agree that dying does not begin as long as the PM is still waiting on a semaphore, then we can play this as follows (N.B. The requesting of a segment from the segment controller goes outside all redness: it can be done in redness, it can be done in whiteness). At the order to the segment controller the PM can set a boolean "Request Segment" true.

The structure in the PM then becomes

AS := true

request of segment

V(SCS)

AS := false; P(own segment semaphore)

The last two statements stand on one line, in order to agree very emphatically that they happen one after the other in deafness. Now we agree that the initialization of the suicide rites cannot take place if AS. Herewith we achieve that the requested pages are there before they are perhaps destroyed. (I wrote pages, I meant segments.)

By the look of it, we can already do a great deal with 1 variable "killvar". Tentatively I would like to give it the following values with the following meanings (The coding of X "runnumber = 0" suggested on page 0 can then lapse.)

killvar = 0
The PM is empty

killvar = 1
The PM is busy with a program that is completely alive

killvar = 2
There is a request for suspended animation

killvar = 3
The program is in suspended animation (hangs on the semaphore killsem)

killvar = 4
There is a request for suicide

killvar = 5
The program has proceeded to the suicide rites.

(Whether the boolean AS should also be coded into this here, e.g. by killvar = 8, is still an open question for me.)

I now take the standpoint that the agent which gives the suicide command reports this via the console teleprinter, but that we do not give the PM (or rather: the program contained in it) the opportunity to write yet a farewell letter to friends and acquaintances, hence no reporting about the status in which the program found itself when the suicide rites were proceeded to. (At most we still introduce

killvar = 6
The program has proceeded to the suicide rites on its own initiative, as opposed to killvar = 5. On the last printer form a cry can then still appear "I have terminated myself", or else "I have been kicked off".)

I hoped that the transition to the suicide rites could be effectuated by tampering with the status of the interrupted program in such a way that, roughly with the mechanism of the General Goto, control jumps to the point in the PM-block where the PM goes to finish off the streams. This is not enough, for on account of the SIS two program segments can be sanctified - and the general goto desanctifies only one of them; if the murder takes place when an array segment has been sanctifyingly requested, then we are saddled with that as well. (The murder is indeed deferred until this segment has arrived.) I can imagine that all this requires, at the sanctifying request of array segments, an extra note, and that the transition to the suicide rites happens as little as possible through the coordinator and as much as possible through the PM itself. The clearing away of a piece of SIS (desanctification and so on) is namely an action that, in the case of the dynamic error, will happen under the auspices of the PM itself.

5. The dynamic error.

After ample considerations we have come to the conclusion that the reaction to a dynamic error is a pure PM-matter. I imagine that this will only be able to occur during killvar = 1 (in red pieces I do not at all want to be able to do such gruesome things). The PM will then have to break down the stack as far as the first Working Space Pointer; if the error occurs in an SIS, then it must discover this by stack inspection; if an array segment is still sanctified, then the error-detecting mechanism knows this, and that array segment can be desanctified. Once the stack has been reduced to the first level at which SE1 can be called, then that will have to happen in order to be able to simulate a call of the procedure ALARM. With the jump towards it the last program page is desanctified, ALARM can inspect the stack below it as much as it wants and can finally jump to the point in the PM-block where we end up after program termination.

The microscopy of the fiddling with the status down in the HAM-room and the steering towards an interposed system entry is a little game that does rather captivate me; I myself do not possess the knowledge to see directly how this is to be done; I did think that it was possible and should be glad to help think about it.

EWD.

transcribed by Sam Samshuijzen

revised Sun, 14 Jan 2007
