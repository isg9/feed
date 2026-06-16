---
title: "Over stapeladministratie (EWD69)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD69.html
published: "1963-11-10T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD69.PDF
---

# Over stapeladministratie

EWD 69

4 november 1963

Over stapeladministratie

1. Binnenblokken, procedureblokken en blokhoogte.

Het begrip blok is louter lexicographisch. Het hele programma is per definitie een blok, binnen in een blok kunnen een of meer andere blokken gedefinieerd worden. Behalve de ALGOL-blokken voeren wij nog in:

a) een procedure-declaratie is per definitie een blok

b) een for-statement is per definitie een blok.

Wij onderscheiden nu twee soorten blokken:

a) procedureblokken

b) alle overige blokken; deze noemen wij "binnenblokken".

Opm. Wij laten voorlopig even buiten beschouwing of we de procedure body als binnenblok van het procedureblok beschouwen.

De aldus ingevoerde blokken hebben de eigenschap, dat elk blok met uitzondering van het programma-blok in een ander blok gedefinieerd is. We voeren nu in het begrip "blokhoogte", dwz. het programma-blok krijgt blokhoogte 0. Elk ander blok krijgt een blokhoogte 1 hoger dan die van het lexicographisch onmiddellijk omvattende blok.

2. Binnentophoogte en display.

Aan elk procedureblok kennen we toe een zg. "binnentophoogte". Ook de binnentophoogte is een zuiver lexicographisch begrip: de binnentophoogte is 1 meer dan de maximum blokhoogte in het inwendige van het procedureblok zonder dat het in een daarin inwendig procedureblok zit. (Andere definitie: het is 1 meer dan de maximum inwendig voorkomende blokhoogte na verwijdering van alle erin voorkomende proceduredeclaraties.)

Binnenblokken hebben de eigenschap, dat men ze slechts vanuit het onmiddellijk omvattende blok kan binnengaan, dit in tegenstelling tot procedureblokken.

Bij binnenkomst van een procedureblok, en alleen dan, zullen we een nieuwe display invoeren. De lengte van deze display kiezen we gelijk aan de binnentophoogte, dwz. groot genoeg om te garanderen, dat we erin de ruimte hebben om (zie onder) de binnenblokken van deze procedure te kunnen behandelen.

De opmerking is nl. dat de techniek van de MC1-vertaler voor ingaan en uitgaan van een binnenblok echt wel efficient is. De MC1-vertaler werkt met een vaste display. Bij binnenkomst hoef je alleen een nieuw element in de display in te vullen en de stapelwijzer op te hogen, bij verlating hoef je alleen de stapelwijzer af te lagen (en displayelement te negeren, maar dat kost niet zo'n moeite.)

Hebben we echter, als in de MC1-vertaler een enkele display, dan hebben we om de haverklap de plicht tot de operatie UDD (Up Date Display). De bewering is, dat slechts een nieuwe display invoeren bij binnenkomst van het procedureblok voldoende is om alle UDD-operaties kwijt te zijn - dwz. te kunnen condenseren tot wisseling van L.

3. De werkruimtewijzer WW.

Tussen statements keert de stapelwijzer SW in een blok steeds terug tot hetzelfde rustpunt; we noemen dit de werkruimtewijzer WW: hij wijst naar de eerste stapelplaats voor de anonyme tussenresultaten.

Normaliter hoeft men zich over de WW-waarde van een blok niet te bekommeren. Tijdens de uitvoering van een statement wordt immers de SW evenveel opgehoogd als afgelaagd en de SW keert automatisch tot zijn rustpunt terug. We hebben echter ook de zg. "plotselinge blokverlating": in een blok kan een "goto L" staan, waarbij "L" een globale label is. Als we hierin terugkeren in een blok, dat we via een type procedure, dwz. midden in een statement, tijdelijk verlaten hebben, dan hebben we in het bestemmingsblok als regel nog een paar anonyme tussenresultaten te vergeten, maw: SW moet teruggezet worden op de waarde van WW, die bij de activering van het bestemmingsblok behoort.

Onze gedachten bepalen we even tot binnenblokken. Als we zitten in een binnenblok B van hoogte n, dan zijn we daarin gekomen via het nauwst lexicographisch omvattende procedureblok op hoogte m ( m < n ). Bij de binnenkomst in dat procedureblok A hebben we in de stapel een nieuwe display ingevoerd, waarvan de onderste m elementen uit een andere display gecopieerd zijn (zie onder), waar het m + 1ste element uit de SW is afgeleid en waarvan de overige elementen beantwoorden aan de binnenblokken van A. Het idee is, om tijdens de uitvoering van binnenblok B de locale grootheden van dit blok te localiseren ten opzichte van element n van deze display en element n + 1 gelijk te laten zijn aan de bijbehorende waarde van WW. Gaan we een binnenblok van B in, dan fungeert element n + 1 onveranderd als referentiepunt voor de locale grootheden van dit blok en element n + 2 bergt dan de (binnen-)WW. Dit is de reden, waarom de binnentophoogte 1 hoger gekozen is dan de maximum voorkomende binnenblokhoogte.

4. Ingaan en uitgaan van binnenblokken.

Bij het ingaan van een binnenblok op hoogte n en bij het uitgaan moeten we de waarde van n weten: bij het binnengaan om te weten waar (nl. op plaats n + 1 van de display) we de nieuwe WW invullen, bij het verlaten omdat we dan de SW terug moeten zetten, dwz. gelijk moeten maken aan element n van de heersende display.

In de programmatekst is de heersende blokhoogte, wat een lexicographisch bepaalde grootheid is, desgewenst statisch bekend en we kunnen de mechanismen "binnenblok in" en "binnenblok uit" creeren, zodat ze van het programma een parameter meekrijgen, dwz. "binnenblok in van hoogte n" en "binnenblok uit van hoogte n".

Een alternatieve oplossing is, om "ergens" —waarover dadelijk meer— het systeem de heersende blokhoogte bij te laten houden: "Binnenblok in" en "Binnenblok uit" krijgen dan geen parameter van het programma mee, maar raadplegen en wijzigen zelf de "ergens" te vinden heersende blokhoogte. Dit laatste is een techniek, die mij iets meer aanspreekt, hoewel ik er op het moment slechts vage argumenten ten gunste van weet te bedenken. Het is het algemene argument, dat je in je programma niet hoeft te vermelden, wat het verwerkende systeem uit de structuur kan afleiden. (Op precies dezelfde manier als we de waarde van de stapelwijzer niet expliciet aangeven, maar door de stapelende opdrachten laten meesjouwen.) De tekst van het programma wordt door deze technieken korter. een nevenvoordeel is, dat de inhoud van de stapel, door de heersende blokhoogte daarin onder te brengen, sprekender wordt. (Dit is voorlopig ook nog vaag: het zou wel eens gewenst kunnen zijn, dat we in de stapel een zodanige administratie bijhouden, dat de stapelinhoud op elk gewenst moment "extern interpretabel" is. Ik denk aan stapelverschuivingen door de coordinator en post mortem dumps.)

Als we de heersende blokhoogte willen onthouden, dan rijst de vraag: waar?

In de MC1-vertaler deden we dit op een vaste plaats: daar was de heersende blokhoogte een in enkelvoud aanwezige, plaatsgebonden staturvariabele. Dit zou voor ons onpractisch zijn in verband met multiprogrammering, omdat we dan bij wisseling van programma de heersende blokhoogte zouden moeten redden. Als we de heersende blokhoogte als globale variabele van een programma invoeren, dwz. onder in de stapel van dat programma, dan hebben we de multiprogrammeringsmoeilijkheid ondervangen, maar zijn we wat betreft de uni-programmering nog net in de toestand die we bij de MC1-vertaler hebben: redden en herstellen bij procedure aanroepen en returns. Mijn voorlopige suggestie is, om de heersende blokhoogte op te bergen bij de heersende display, dwz. als ML[–1]. Deze suggestie is voorlopig, omdat ik nog moet nakijken, of we door de nog niet in het beeld betrokken parametersituatie niet in moeilijkheden komen.

De heersende blokhoogte kan dan bij ingaan en uitgaan van binnenblokken bijgehouden worden; bij binnenkomst in een procedure, waar we een nieuw display introduceren, moet de blokhoogte van de procedure bekend zijn (nl. om te weten, hoeveel plaatsen in de nieuwe display gecopieerd moeten worden); bij die gelegenheid kan ook de initialisering van het nieuwe heersende bloknummer verzorgd worden.

Ik eindig deze korte notitie met het signaleren van de volgende mogelijke complicaties:

1) bij blokverlating moeten we, als er in dit blok "grote arrays" geintroduceerd waren, de door deze arrays bezette trommelpagina's weer vrijgegeven worden.

2) bij "plotseling blokverlating" kunnen een heleboel blokken uit de stapel verdwijnen; wederom geldt, dat aflagen van SW niet voldoende is, omdat er ook grote arrays hierdoor afgevoerd kunnen moeten worden.

3) in de parameter-situatie zitten we nu heel anders: we werken dan onder controle van een heersende L, maar het daarbij geborgen blokhoogte-gegeven ML[–1] heeft op dat ogenblik geen betekenis. Dit zou zich kunnen wreken, als we er over denken, wat we moeten doen, als de stapel verschoven moet worden.

E.W.Dijkstra

transcribed by Sam Samshuijzen

revised Fri, 6 May 2005

---

## English translation

### On stack administration

EWD 69

4 November 1963

On stack administration

1. Inner blocks, procedure blocks and block height.

The concept of a block is purely lexicographic. The entire program is by definition a block; within a block one or more other blocks can be defined. Besides the ALGOL blocks we further introduce:

a) a procedure declaration is by definition a block

b) a for-statement is by definition a block.

We now distinguish two kinds of blocks:

a) procedure blocks

b) all remaining blocks; these we call "inner blocks".

Note. For the time being we leave aside whether we regard the procedure body as an inner block of the procedure block.

The blocks thus introduced have the property that every block, with the exception of the program block, is defined within another block. We now introduce the concept "block height", i.e. the program block is given block height 0. Every other block is given a block height 1 greater than that of the lexicographically immediately enclosing block.

2. Inner-top height and display.

To every procedure block we assign a so-called "inner-top height". The inner-top height too is a purely lexicographic concept: the inner-top height is 1 more than the maximum block height in the interior of the procedure block without it lying within an inner procedure block contained therein. (Alternative definition: it is 1 more than the maximum block height occurring in the interior after removal of all procedure declarations occurring within it.)

Inner blocks have the property that one can enter them only from the immediately enclosing block, this in contrast to procedure blocks.

Upon entry of a procedure block, and only then, we shall introduce a new display. We choose the length of this display equal to the inner-top height, i.e. large enough to guarantee that we have within it the room to be able to handle (see below) the inner blocks of this procedure.

The remark, namely, is that the technique of the MC1 translator for entering and leaving an inner block really is quite efficient. The MC1 translator works with a fixed display. Upon entry you need only fill in a new element in the display and increment the stack pointer; upon departure you need only decrement the stack pointer (and ignore the display element, but that does not cost much effort.)

If, however, as in the MC1 translator, we have a single display, then we are obliged at every turn to perform the operation UDD (Up Date Display). The claim is that merely introducing a new display upon entry of the procedure block is sufficient to be rid of all UDD operations - i.e. to be able to condense them into a switching of L.

3. The working-space pointer WW.

Between statements the stack pointer SW within a block always returns to the same resting point; we call this the working-space pointer WW: it points to the first stack location for the anonymous intermediate results.

Normally one need not concern oneself with the WW value of a block. During the execution of a statement the SW is, after all, incremented by as much as it is decremented, and the SW automatically returns to its resting point. We have, however, also the so-called "sudden block exit": within a block there may stand a "goto L", where "L" is a global label. If by this we return into a block which we have temporarily left via a procedure of type, i.e. in the middle of a statement, then in the destination block we have as a rule still a few anonymous intermediate results to forget, in other words: SW must be reset to the value of WW belonging to the activation of the destination block.

Let us confine our thoughts for a moment to inner blocks. If we are in an inner block B of height n, then we have come into it via the most closely lexicographically enclosing procedure block at height m ( m < n ). Upon entry into that procedure block A we have introduced a new display in the stack, of which the lowest m elements have been copied from another display (see below), where the m + 1st element has been derived from the SW and of which the remaining elements correspond to the inner blocks of A. The idea is, during the execution of inner block B, to localize the local quantities of this block with respect to element n of this display and to let element n + 1 be equal to the corresponding value of WW. If we enter an inner block of B, then element n + 1 functions unchanged as the reference point for the local quantities of this block and element n + 2 then holds the (inner) WW. This is the reason why the inner-top height has been chosen 1 greater than the maximum occurring inner-block height.

4. Entering and leaving inner blocks.

Upon entering an inner block at height n and upon leaving we must know the value of n: upon entry in order to know where (namely at location n + 1 of the display) we fill in the new WW, upon departure because we must then reset the SW, i.e. must make it equal to element n of the prevailing display.

In the program text the prevailing block height, which is a lexicographically determined quantity, is if desired statically known, and we can create the mechanisms "inner block in" and "inner block out" in such a way that they receive a parameter from the program, i.e. "inner block in of height n" and "inner block out of height n".

An alternative solution is to have the system keep track of the prevailing block height "somewhere" —more on which presently—: "Inner block in" and "Inner block out" then receive no parameter from the program, but themselves consult and modify the prevailing block height to be found "somewhere". This latter is a technique which appeals to me somewhat more, although at the moment I can think of only vague arguments in its favour. It is the general argument that in your program you need not state what the processing system can derive from the structure. (In precisely the same way as we do not indicate the value of the stack pointer explicitly, but let it be carried along by the stacking instructions.) The text of the program is shortened by these techniques. a collateral advantage is that the contents of the stack, by accommodating the prevailing block height in it, become more expressive. (This is for the time being still vague too: it might well be desirable that we keep in the stack an administration of such a kind that the stack contents are at every desired moment "externally interpretable". I am thinking of stack shifts by the coordinator and post mortem dumps.)

If we wish to remember the prevailing block height, then the question arises: where?

In the MC1 translator we did this in a fixed location: there the prevailing block height was a state variable present in a single copy and bound to a location. This would be impractical for us in connection with multiprogramming, because we would then have to save the prevailing block height upon switching of program. If we introduce the prevailing block height as a global variable of a program, i.e. at the bottom of the stack of that program, then we have circumvented the multiprogramming difficulty, but with respect to uniprogramming we are still exactly in the situation that we have with the MC1 translator: saving and restoring upon procedure calls and returns. My provisional suggestion is to store the prevailing block height with the prevailing display, i.e. as ML[–1]. This suggestion is provisional, because I must still check whether we do not run into difficulties on account of the parameter situation not yet brought into the picture.

The prevailing block height can then be kept track of upon entering and leaving inner blocks; upon entry into a procedure, where we introduce a new display, the block height of the procedure must be known (namely in order to know how many locations in the new display must be copied); on that occasion the initialization of the new prevailing block number can also be taken care of.

I conclude this short note by signalling the following possible complications:

1) upon block exit we must, if "large arrays" had been introduced in this block, again release the drum pages occupied by these arrays.

2) upon "sudden block exit" a whole lot of blocks can disappear from the stack; again it holds that decrementing SW is not sufficient, because by this large arrays too may have to be disposed of.

3) in the parameter situation we are now in a quite different position: we then work under control of a prevailing L, but the block-height datum ML[–1] stored therewith has at that moment no meaning. This might come back to bite us when we consider what we must do when the stack has to be shifted.

E.W.Dijkstra

transcribed by Sam Samshuijzen

revised Fri, 6 May 2005
