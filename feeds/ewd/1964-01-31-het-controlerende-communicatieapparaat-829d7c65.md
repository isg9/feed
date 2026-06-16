---
title: "Het controlerende communicatieapparaat (EWD77)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD77.html
published: "1964-01-31T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD77.PDF
---

# Het controlerende communicatieapparaat

EWD 77

Het controlerende communicatieapparaat

Het volgende is een studie over het bespelen van een controlerend communicatieapparaat dat bediend wordt via een beperkt startmagazijn. Ala dit startmagazijn bv. 4 startopdrachten kan bevatten, dan hebben X8 en communicatieapparaat toegang tot

"array startopdracht,slotwoord [0:3]"; in de praktijk zullen deze twee array's in het geheugen dooreen gevlochten staan, maar dat doet nu niet ter zake.

Het autonoom transport leest de startopdracht en vult het slotwoord ; omdat het lezen-gehoorzamen- van de startopdracht als primair ervaren wordt, noemen wij de wijzer, onder controle waarvan dit gebeurt, de "integer leegwijzer".

Twee seinpalen, AFT en IFT regelen de synchronisatie: AFT is het aantal startopdrachten in voorraad, IFT het aantal "terugmeldingen", waarvan de X8 nog geen nota genomen heeft.

In dit geval kan de handeling van het autonoom transport in eerste aanleg beschreven worden door zoiets als:

"L: P(AFT);

leegwijzer:= REM(leegwijzer +1,4);

gehoorzaam(startopdracht[leegwijzer]);

meld terug in (slotwoord[leegwijzer]);

V(IFT);

goto L"

Met "zoiets als" wordt bedoeld

a)
dat het hier irrelevant is, of leegwijzer voor of na gebruik wordt opgehoogd; in deze beschrijving hebben we "voor" gekozen.

b)
- dat hier over de non-OK-toestand nog niet gesproken is; we beperken ons voorlopig tot het geval, dat het allemaal goed is gegaan.

In het volgende wil ik de consequenties nagaan van het feit, dat, voorzover de X8 betreft, de "V(IFT)" een dubbele betekenis kan hebben. Dit immers meldt een gebeuren, dat in twee richtingen betekenis heeft. Enerzijds is het gericht op het verleden, en wordt hier dus gemeld, hoe iets dat vroeger gestart is, is afgelopen, anderzijds heeft het betekenis gericht op de toekomst, omdat er nu, desgewenst, weer plaats in het startmagazijn vrijkomt. Tengevolge hiervan kan het zijn, dat er nu twee processen voortgang kunnen vinden, nl. het proces, voor welks voortzetting de voltooiing der autonome handeling essentieel is en het proces, dat graag van het apparaat gebruik zou willen maken.

We voeren hiertoe een derde, geprogrammeerde seinpaal in nl. SRT = startruimtetelling. SRT geeft aan het aantal startopdrachten, waarvoor in het startmagazijn ruimte is.

Het startmagazijn wordt nu bespeeld door twee onderling asynchrone processen, viz. Starter en Controleur. De zojuist ingevoerde seinpaal dient voor de onderlinge synchronisatie van deze twee ; deze twee processen hebben, zoals wij straks zullen zien, nog wel meer verknopingen.

De Controleur raadpleegt het slotwoord onder controle van een "integer controlewijzer" in ruwste vorm:

"Controleur: P(IFT);

controlewijzer:= REM(controlewijzer + 1,4):

if slotwoord[controlewijzer] ≠ correct then actie fout else actie goed;

V(SRT); goto Controleur"

Over "actie goed" en "actie fout" straks meer.

De Starter neemt met het startmagazijn contact op onder controle van een vulwijzer, altijd via de sequens:

".....; P(SRT);

vulwijzer:= REM (vulwijzer + 1,4);

startopdracht[vulwijzer]:= volgende startopdracht;

V(AFT); ....."

Ook over "volgende startopdracht" straks meer ; deze moet er nl. zijn.

De geprogrammeerde seinpaal SRT is ingevoerd om controle op de voltooide operatie en aanbieden van de volgende, waarvan nu immers ruimte in het startmagazijn is, in de tijd te kunnen splitsen.

Nu moeten wij in de organisatie verdisconteren:

a)
dat het gegeven van een startmagazijn van 4 startopdrachten een physisch gegeven is, maar dat deze omvang niets met de logische eenheden te maken heeft, "logische eenheden" ten aanzien van controle (en overdoen) of van interesse in voltooiing.

b)
dat activiteiten van Starter en Controleur op elkaar afgestemd dienen te worden: de ene zal dingen in gang zetten, de ander zal van de voltooiing kennis nemen. De laatste dient dan te weten van de voltooiing waarvan hij kennis neemt, waar dit goed voor was en welke consequenties hieraan verbonden dienen te worden.

Wij kunnen deze beide dingen bereiken, door een (eventueel groter) magazijn in te voeren, waartoe zowel Starter als Controleur toegang hebben. De elementen van dit magazijn —laten we het even actiemagazijn noemen— bevatten:

a)
ten bate van de Starter startopdrachten

b)
ten bate van de Controleur nadere specificaties van de handelingen "actie goed" resp."actie fout" na voltooiing van de in dit element gegeven startopdracht.

Beide processen zullen dit actiemagazijn aftasten via een cyclisch werkende leegwijzer. Voeren wij de twee gebruikelijke seinpalen voor dit actiemagazijn in, via "actiemagazijn vol" en "actiemagazijn leeg" , dan wordt in dit algemene schema de Starter

"Starter: P(SRT, actiemagazijn vol);

starterleegwijzer:= REM(starterleegwijzer + 1,N);

vulwijzer:= REM(vulwijzer + 1,4);

startopdracht[vulwijzer]:= actiemagazijn[starterleegwijzer];

V(AFT); goto Starter"

opm.
Met "actiemagazijn[ ]" wordt hier het startopdrachtgedeelte van een element bedoeld ; N = omvang van achtiemagazijn.

De Controleur begint met

"Controleur: P(IFT);

controleurleegwijzer:= REM(controleurleegwijzer + 1,N);

controlewijzer:= etc...."

"actie goed" wordt nu iets, dat het controleurgedeelte van "actiemagazijn [controleurleegwijzer]" verwerkt.

Hieronder kan vallen:

a)
een aantal malen V(actiemagazijn leeg)

b)
de V-operatie op een in het actiemagazijn opgegeven seinpaal.

Het gecontroleerd transport kan nl. een reeks afsluiten ; hierdoor komt een aantal plaatsen in het actiemagazijn vrij ; verder kan bij een bepaald programma hierop hebben staan wachten.

Een programma, dat een serie elementen van het actiemagazijn wil opgeven, moet de mogelijkheid hebben, deze serie "ongespatieerd" door ander aanbod te kunnen opgeven. Dit kan met een binaire seinpaal AV (aanbod vrij):

"P(AV);

L: P(actiemagazijn leeg);

vulwijzer:= REM(vulwijzer + 1,N);

actiemagazijn[vulwijzer]:= volgend element;

V(actiemagazijn vol);

if B then goto L;

V(AV)"

Opm. De elementen bevatten, hoeveel plaatsen in het actiemagazijn vrijgegeven zullen worden na controle. Het is wel vaak niet meer dan N niet vrijgevende elementen in successie aan te bieden. De veilige manier om dit te garanderen is om bij laatst aangeboden element alle elementen, die tussen P(AV) en V(AV) zijn aangeboden, vrij te doen geven. Als je niet oppast, zou het actiemagazijn "combinatorisch" dicht kunnen groeien!

Als het aantal elementen van het actiemagazijn, dat "per keer" gevuld wordt een redelijke bovengrens heeft, dan lenen deze programma's zich tot een vereenvoudiging, die nog wel illustratief is. Je kunt dan zeggen: kom, wees niet kinderachtig, stel in het actiemagazijn ruimte ter beschikking in wat grotere porties ; we noemen deze porties nu "transportschema's": een transportschema neemt de rol van een "aantal bij elkaar behorende elementen uit het actiemagazijn" over.

Het actiemagazijn wordt nu vervangen door een "transportschemabuffer" ; de seinpalen "actiemagazijn leeg, resp. -vol" vervallen en worden vervangen door "transportschemabuffer leeg, resp. vol" .

Het laatste programma wordt nu:

"P(AV, transportschemabuffer leeg);

vulwijzer:= REM(vulwijzer +1,N1);

transportschema[vulwijzer]:= .....;

V(AV,transportschemabuffer vol)"

De Controleur bevat dan als onderdeel van "actie goed" de operatie "V(transportschemabuffer leeg)", de Starter krijgt de volgende constructie:

Starter: P(transportschemabuffer vol);

starterleegwijzer:= REM(starterleegwijzer +1,N1) ;

L: P(SRT);

vulwijzer:= REM(vulwijzer +1,4);

startopdracht[vulwijzer]:= .....;

V(AFT): if transportschema nog niet leeg then goto L;

goto Starter"

Hiermee zijn enige "hoogfrequente" seinpalen geruild tegen evenveel "laagfrequente": ten koste van wat bufferruimte hebben wat werk uitgespaard. Het aantal onderling asynchrone processen is niet geslonken!

De trommel

Voor de trommel hadden we een iets afwijkend schema bedacht, dat tot op heden —12 februari 1964— nog niet verworpen is. Hierbij komt nl. nog de complicatie, dat een programma zelf geen "transportschema" opstelt; het kan slechts zijn behoefte uiten. Welke transport of transporten dit impliceert, hangt af van de momentane geheugenbezetting. M.a.w.: de "vertaling" van behoefte naar transportschema is een handeling, en het moment, waarop deze handeling plaats zal vinden moet gekozen worden. Wij hebben ons op het standpunt gesteld dat het gewenst is, deze handeling zo laat mogelijk te laten plaats vinden: het verlaten van behoefte in transportschema kan dan nl. van de meest recente gegevens uitgaan. Dit wordt "tegengewerkt" door het streven, de trommeltransport, als dit de bottleneck wordt, zo min mogelijk ledigheid te gunnen. D.w.z. de bufferingsmogelijkheden zijn hier niet bedoeld om grote asynchroniteit op te kunnen vangen, integendeel: tussen het uiten van behoefte en bevrediging daarvan zal het programma in kwestie als regel helemaal niet door kunnen werken! We hebben dus meer te maken met wat inmiddels een "morele haastsituatie" is gedoopt: buffering van startopdrachten is slechts een middel, om de morele haastsituatie te mitigeren.

Op deze gronden kwamen we tot de volgende constructie. We hebben een transportschemabuffer, dat cyclisch wordt afgewerkt. Over het aantal transportschema's, dat in deze buffer kan, zullen we het later hebben.

Elk programma communiceert met de trommeltransport via een zg. "behoeftetoonbank"; hier is gedacht aan een toonbank met de capaciteit van 1 behoefte.

Het aanbod van een behoefte vindt plaats via

".....P(behoeftetoonbank leeg);

aanbod behoefte;

V(behoeftetoonbank vol);

P(zekere seinpaal)....."

Over de "zekere seinpaal" zullen we het straks nog wat uitgebreider hebben.

Het vertalen van behoefte vindt plaats gesynchroniseerd ten opzichte van het vullen van het physisch startmagazijn van de trommel, nl:

VERZORGER: P(toonbank vol, transportschemabuffer leeg);

vertaal behoefte in transportschema;

L: P(SRT);

zet volgende startopdracht trommel;

V(AFT);

if heersend transportschema nog niet leeg then goto L;

goto VERZORGER..."

De controleur bevat als onderdeel van "actie goed" na controle van een heel transportschema

"V(transportschemabuffer leeg, zekere seinpaal)".

Voor de "zekere seinpaal" kan men kiezen:

a)
elk programma zijn eigen seinpaal ; deze moet dan (als onderdeel van de behoefte) in het transportschema worden opgenomen.

b)
anderzijds weet men, dat van deze seinpalen er maar een paar tegelijkertijd actueel kunnen zijn (ik geloof 1 meer dan er transportschema's in de transportschemabuffer kunnen). Men kan dit N-tal cyclisch afwerken bij het toonbank vullen en bij het vrijgeven door de controleur. Dit is een klein detail, dat niet belangrijk is.

Uit de tekst van de VERZORGER blijkt, dat als elk transportschema minstens 1 transport vergt, het zinloos is, om meer dan vijf transportschema's te kunnen bufferen. Twee is een absoluut minimum: als je maar 1 transportschema hebt, is bij overgang van het ene transportschema naar het volgende de trommeltransport beslist even ledig. Gezien de tijd van transporten (tientallen millisecunden) dachten we niet, dat een transportschemabuffer van meer dan twee erg verdedigbaar is.

transcribed by Sam Samshuijzen

revised Mon, 10 Jan 2005

---

## English translation

### The controlling communication apparatus

EWD 77

The controlling communication apparatus

The following is a study on the operation of a controlling communication apparatus that is served via a limited start magazine. If this start magazine can contain, say, 4 start commands, then X8 and the communication apparatus have access to

"array startcommand,closeword [0:3]"; in practice these two arrays will be interleaved with one another in the store, but that is now beside the point.

The autonomous transport reads the start command and fills in the close word; because the reading—obeying—of the start command is experienced as primary, we call the pointer under whose control this happens the "integer empty pointer".

Two semaphores, AFT and IFT, regulate the synchronisation: AFT is the number of start commands in stock, IFT the number of "feedbacks" of which the X8 has not yet taken notice.

In this case the action of the autonomous transport can in first instance be described by something like:

"L: P(AFT);

emptypointer:= REM(emptypointer +1,4);

obey(startcommand[emptypointer]);

feed back into (closeword[emptypointer]);

V(IFT);

goto L"

By "something like" is meant

a)
that it is irrelevant here whether the empty pointer is incremented before or after use; in this description we have chosen "before".

b)
- that the non-OK state has not yet been spoken of here; we restrict ourselves for the time being to the case in which everything has gone well.

In what follows I wish to examine the consequences of the fact that, as far as the X8 is concerned, the "V(IFT)" can have a double meaning. For it announces an event that has significance in two directions. On the one hand it is directed at the past, and thus announces here how something that was started earlier has terminated; on the other hand it has significance directed at the future, because now, if desired, room becomes free again in the start magazine. As a consequence of this it may be that two processes can now make progress, namely the process for whose continuation the completion of the autonomous action is essential, and the process that would like to make use of the apparatus.

To this end we introduce a third, programmed semaphore, namely SRT = start-room-count. SRT indicates the number of start commands for which there is room in the start magazine.

The start magazine is now operated by two mutually asynchronous processes, viz. Starter and Controller. The semaphore just introduced serves for the mutual synchronisation of these two; these two processes have, as we shall see presently, still more interconnections.

The Controller consults the close word under the control of an "integer control pointer", in its crudest form:

"Controller: P(IFT);

controlpointer:= REM(controlpointer + 1,4):

if closeword[controlpointer] ≠ correct then action wrong else action right;

V(SRT); goto Controller"

More about "action right" and "action wrong" presently.

The Starter makes contact with the start magazine under the control of a fill pointer, always via the sequence:

".....; P(SRT);

fillpointer:= REM (fillpointer + 1,4);

startcommand[fillpointer]:= next startcommand;

V(AFT); ....."

More about "next start command" too presently; for it must be there.

The programmed semaphore SRT has been introduced in order to be able to split, in time, the checking of the completed operation and the offering of the next one, for which there is now indeed room in the start magazine.

Now we must take into account in the organisation:

a)
that the fact of a start magazine of 4 start commands is a physical fact, but that this size has nothing to do with the logical units, "logical units" with respect to checking (and redoing) or to interest in completion.

b)
that the activities of Starter and Controller must be attuned to one another: the one will set things in motion, the other will take notice of the completion. The latter must then know of the completion of which he takes notice, what it was good for and what consequences must be attached to it.

We can achieve both of these things by introducing a (possibly larger) magazine to which both Starter and Controller have access. The elements of this magazine —let us call it the action magazine for the moment— contain:

a)
for the benefit of the Starter, start commands

b)
for the benefit of the Controller, further specifications of the actions "action right" resp. "action wrong" after completion of the start command given in this element.

Both processes will scan this action magazine via a cyclically working empty pointer. If we introduce the two usual semaphores for this action magazine, via "action magazine full" and "action magazine empty", then in this general scheme the Starter becomes

"Starter: P(SRT, action magazine full);

starteremptypointer:= REM(starteremptypointer + 1,N);

fillpointer:= REM(fillpointer + 1,4);

startcommand[fillpointer]:= actionmagazine[starteremptypointer];

V(AFT); goto Starter"

note.
By "actionmagazine[ ]" is meant here the start-command portion of an element; N = size of action magazine.

The Controller begins with

"Controller: P(IFT);

controlleremptypointer:= REM(controlleremptypointer + 1,N);

controlpointer:= etc...."

"action right" now becomes something that processes the controller portion of "actionmagazine [controlleremptypointer]".

Under this may fall:

a)
a number of times V(action magazine empty)

b)
the V-operation on a semaphore specified in the action magazine.

For the controlled transport may close off a sequence; through this a number of places in the action magazine become free; furthermore, in a particular program, something may have been waiting for this.

A program that wants to specify a series of elements of the action magazine must have the possibility of specifying this series "unspaced" by other offerings. This can be done with a binary semaphore AV (offering free):

"P(AV);

L: P(action magazine empty);

fillpointer:= REM(fillpointer + 1,N);

actionmagazine[fillpointer]:= next element;

V(action magazine full);

if B then goto L;

V(AV)"

Note. The elements contain how many places in the action magazine will be released after checking. It is often only possible to offer no more than N non-releasing elements in succession. The safe way to guarantee this is, with the last element offered, to have all elements that were offered between P(AV) and V(AV) released. If you are not careful, the action magazine could grow "combinatorially" clogged!

If the number of elements of the action magazine that is filled "at a time" has a reasonable upper bound, then these programs lend themselves to a simplification that is still illustrative. You can then say: come now, do not be childish, make room available in the action magazine in somewhat larger portions; we now call these portions "transport schemes": a transport scheme takes over the role of "a number of mutually belonging elements from the action magazine".

The action magazine is now replaced by a "transport-scheme buffer"; the semaphores "action magazine empty, resp. -full" lapse and are replaced by "transport-scheme buffer empty, resp. full".

The last program now becomes:

"P(AV, transport-scheme buffer empty);

fillpointer:= REM(fillpointer +1,N1);

transportscheme[fillpointer]:= .....;

V(AV,transport-scheme buffer full)"

The Controller then contains as part of "action right" the operation "V(transport-scheme buffer empty)", the Starter gets the following construction:

Starter: P(transport-scheme buffer full);

starteremptypointer:= REM(starteremptypointer +1,N1) ;

L: P(SRT);

fillpointer:= REM(fillpointer +1,4);

startcommand[fillpointer]:= .....;

V(AFT): if transportscheme not yet empty then goto L;

goto Starter"

With this, some "high-frequency" semaphores have been exchanged for an equal number of "low-frequency" ones: at the cost of some buffer room we have saved some work. The number of mutually asynchronous processes has not shrunk!

The drum

For the drum we had devised a somewhat divergent scheme, which up to the present —12 February 1964— has not yet been rejected. Here there is, namely, the additional complication that a program does not itself draw up a "transport scheme"; it can only express its need. Which transport or transports this implies depends on the momentary store occupancy. In other words: the "translation" of need into transport scheme is an action, and the moment at which this action is to take place must be chosen. We have taken the standpoint that it is desirable to let this action take place as late as possible: the translating of need into transport scheme can then proceed from the most recent data. This is "counteracted" by the endeavour to grant the drum transport, if this becomes the bottleneck, as little idleness as possible. That is to say, the buffering possibilities here are not intended to absorb great asynchrony; on the contrary: between the expressing of a need and its satisfaction the program in question will as a rule not be able to proceed at all! We are thus dealing rather with what has meanwhile been dubbed a "moral hurry situation": the buffering of start commands is merely a means of mitigating the moral hurry situation.

On these grounds we arrived at the following construction. We have a transport-scheme buffer that is processed cyclically. About the number of transport schemes that can fit in this buffer we shall speak later.

Each program communicates with the drum transport via a so-called "need counter"; here a counter with the capacity of 1 need has been thought of.

The offering of a need takes place via

".....P(need counter empty);

offer need;

V(need counter full);

P(certain semaphore)....."

About the "certain semaphore" we shall speak somewhat more extensively presently.

The translating of need takes place synchronised with respect to the filling of the physical start magazine of the drum, namely:

PROVIDER: P(counter full, transport-scheme buffer empty);

translate need into transport scheme;

L: P(SRT);

place next start command drum;

V(AFT);

if prevailing transport scheme not yet empty then goto L;

goto PROVIDER..."

The controller contains as part of "action right", after checking a whole transport scheme,

"V(transport-scheme buffer empty, certain semaphore)".

For the "certain semaphore" one can choose:

a)
each program its own semaphore; this must then (as part of the need) be incorporated in the transport scheme.

b)
on the other hand, one knows that of these semaphores only a few can be actual at the same time (I believe 1 more than the number of transport schemes that can fit in the transport-scheme buffer). One can process this N-tuple cyclically at the filling of the counter and at the release by the controller. This is a small detail that is not important.

From the text of the PROVIDER it appears that if each transport scheme requires at least 1 transport, it is pointless to be able to buffer more than five transport schemes. Two is an absolute minimum: if you have only 1 transport scheme, then at the transition from one transport scheme to the next the drum transport is definitely idle for a moment. Given the time of transports (tens of milliseconds), we did not think that a transport-scheme buffer of more than two is very defensible.

transcribed by Sam Samshuijzen

revised Mon, 10 Jan 2005
