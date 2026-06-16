---
title: "Over trommelpaginatransporten (EWD72)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD72.html
published: "1963-12-18T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD72.PDF
---

# Over trommelpaginatransporten

EWD 72

Over trommelpaginatransporten.

1. Om het niet te moeilijk te maken beperken we ons voorlopig tot array-pagina's.

1.1. De noodzaak tot transport wordt door het behoeftige programma zelf gedetecteerd en wel in doofheid. Als het nl. niet in doofheid gebeuren zou, zou, na de constatering, dat de pagina er wel is, deze nog onder je vingers weggeratst kunnen worden.

1.2. Als transport noodzakelijk blijkt, dan gaan we naar de coordinator, dwz. het programma komt op de wachtlijst. We keren pas in het programma terug, als de paginatransport voltooid is; we keren dan in doofheid in het programma terug en de aangevraagde arraypagina is geheiligd.

1.2.1. De pagina is geheiligd met de volgende bedoeling: als het transport voltooid is, kan dit programma weer verder; het is niet gezegd, dat het ook onmiddellijk weer verder gaat. In die tussentijd mag de pagina niet verdwijnen!

1.2.2. Het terugkeren in doofheid is niet strikt noodzakelijk, wanneer het programma maar eerst het array-element selecteert en daarna pas de pagina ontheiligt. Keren we terug in doofheid —wat we voor niks krijgen— dan zijn we vrij in de volgorde van elementselectie en ontheiliging. Het programma heeft de plicht de arraypagina te ontheiligen, het element te selecteren (volgorde vrij) en weer horend te worden. Ook dit krijgen we cadeau via de herstellende terugsprong. Een nevenvoordeel is, dat wanneer dit programma verder gaat in nog even doofheid, de ontheiliging gegarandeerd binnen afzienbare tijd na voortzetting plaatsvindt.

1.3. Ik neem aan, dat in geval van noodzakelijk arraypaginatransport de heersende programmapagina ook geheiligd wordt. Dit vergemakkelijkt de terugkeerprocedure aanmerkelijk. Als we dit doen, moet naar ik aanneem ook de programmapagina in doofheid ontheiligd worden (zie 1.1., 1.9.2. e.v.).

1.4. Onbewezen is, dat door de heiligingsprocedure de boel niet kan vast lopen. Dit bewijs kunnen we pas proberen te leveren, als we een volledig overzicht hebben over de aanvraagprocedure bij afwezige (array)pagina.

1.6. Ik beperk me voorlopig tot dat stuk van de coordinator, dat permanent op de kernen aanwezig zal zijn, op een verrekt vaste plaats. De coordinator zal een aantal zg "coordinatorroutines" omvatten; als regel zullen deze met een doofmakende subroutinesprong worden aangeroepen. Het aanroepen van een coordinatorsubroutine op zich zelf betekent niet noodzakelijkerwijze "verlating van het heersende ALGOL-programma". Als regel kan het zo zijn, dat de coordinatorroutine vaststelt, of het heersende programma ongehinderd door kan werken of niet. Zo ja, dan gaat het ALGOL-programma weldra ongehinderd verder — de machine wordt horend bij de herstellende terugsprong uit de coordinatorroutine. De andere mogelijkheid is, dat de coordinatorroutine beslist, dat dit (ALGOL-)programma niet ongehinderd door kan gaan. In dat geval wordt het programma verlaten, dwz. het is niet langer "het heersende programma", het wordt als non-actief op de wachtlijst van de coordinator geplaatst en de coordinator kiest een ander programma en maakt dit heersend.

Ik zoek hier misschien wat spijkers op laag water; wat ik ook probeer is het vinden van een terminologie. In de text van het ALGOL-programma fungeert menig aanroep van een coordinatorroutine als een "mogelijke verlating". Binnenin de coordinatorroutine is de al of niet verlating van het programma een bewuste zaak en kunnen we voor de verlating in doofheid, en na de terugkomst, doelbewuste maatregelen treffen (heiligen en ontheiligen b.v.)

1.7. Coordinatorroutines mogen "over hun break point heen" geen vaste geheugenplaatsen met variabele informatie bezetten: ze kunnen immers parallel geactiveerd worden. Functioneel blijven de betroffen acties dus tot het aanroepende programma behoren. Uit ruimtelijke overwegingen —ook al omdat ze geacht worden zich permanent op de kernen te bevinden— staan ze ook in het geval van multiprogrammering slechts in enkelvoud op de kernen.

1.8. Een gevolg van de graad van bewustheid van een mogelijke onderbreking is de volgende: een gewone interruptie impliceert, dat bij stapelschuiven het B-register aangepast moet worden en dat we heel voorzichtig moeten zijn met absolute verwijzingen naar stapelplaatsen. In een coordinatorroutine kunnen we veel vrijer zijn: na terugkeer onder het break point kunnen we immers testen of de stapel verschoven is en zo ja, wat adressen aanpassen! (dit moet om practische redenen liever niet de spuigaten uitlopen!)

1.9. Voorbeelden van coordinatorroutines zijn de P-operatie, de V-operatie en de indiceeroperaties (schrijvend en lezend). Als het een kleine array blijkt te zijn loopt het helemaal met een sisser af!

1.9.1. Opm. De vorige zin sterkt mij, door de moeilijke scheidbaarheid van "runtime system" en coordinator in de overtuiging, dat we over multiprogrammering in machinecode voorlopig niet hoeven te denken.

1.9.2. In hoeverre de aanroep van een coordinatorroutine in het geval van break point "heiliging" van de aanroepende pagina impliceert zal er van geval to geval van afhangen. Iemand zou kunnen stellen, dat in elk programma de heersende pagina altijd heilig is, maar ik voel daar niet zoveel voor, als het betekent, dat elk programma, ongeacht hoelang het op non-actief staat, altijd minstens 1 pagina blijft bezetten. Je kunt b.v. kijken, of je ermee volstaan kunt heiliging alleen te introduceren in die gevallen, waarvan je er op aan kunt, dat de inhibitie op korte termijn opgeheven zal zijn. Het wordt nogmaals duidelijk, dat we de "deugdelijkheid" van het concept heiligheid pas kunnen bekijken, als een concreet voorstel op tafel ligt. De rol van Advocatus Diaboli lijkt me in dezen uiterst belangrijk! Het zou me niet verbazen, als ons idee over heiligheid in de loop van deze onderzoekingen drastisch zouden veranderen.

1.10. De coordinator-wachtlijst bevat alle programma's in de machine, onderverdeeld naar twee categorieen, de geblokkeerden en de uitvoerbaren.

Geblokkeerde programma's zijn programma's, die niet verder kunnen, doordat ze op een seinpaal staan te wachten, op de voltooiing van een voor hun voortzetting noodzakelijk gebeurtenis.

Uitvoerbare programma's zijn programma's, die wel verder kunnen, ware het niet, dat de X8 maar 1 van hen verder kan helpen. (Ik neem aan, dat het programma, dat door de X8 uitgevoerd wordt, onder de uitvoerbare gerangschikt blijft; de term "wachtlijst" is dan wat merkwaardig, maar laten we daar maar in berusten. Als de besturing echt in de coordinator zit, dan is de term in orde.)

Het effect van de P-operatie kan zijn, dat een programma van uitvoerbaar nu geblokkeerd wordt, het effect van de V-operatie kan zijn, dat een (ander) programma uit de groep der geblokkeerden naar de uitvoerbaren wordt overgeheveld. Een programma, dat in actie door een interruptie onderbroken wordt, blijft gerangschikt onder de uitvoerbare !

We kunnen het ook zo zien: als in een programma een P-operatie geinitieerd wordt, dan was op dat moment het programma in actie, dus uitvoerbaar. Nu zijn er twee gevallen. Of de heersende waarden van de betroffen seinpalen vormen geen beletsel, of zij doen dit wel. In het eerste geval worden zij afgelaagd, de P-operatie is daarmee voltooid en het programma blijft uitvoerbaar. (Of het ook in actie blijft is een heel ander chapiter!) In het tweede geval wordt het programma uit de groep der uitvoerbaren gehaald. De groep der geblokkeerden bevat dus alleen alle programma's, die in een geinitialiseerde, maar niet voltooide P-operatie zijn blijven hangen.

1.11. Ten aanzien van de uitvoerbare programma's kunnen we twee wegen inslaan inzake heiliging. We beschouwen een programma, dat in actie was, door een interruptie is onderbroken en tengevolge daarvan op het uitvoerbare gedeelte van de wachtlijst is gekomen.

1.11.1. In de ene techniek staan we toe, dat de programma-pagina, waarmee we op het moment van interruptie mee bezig waren tijdens het wachten verdwijnt. Dit impliceert: a) dat we in de wachtlijst bij de status quo gegevens het terugkeeradres invariant moeten opbergen. b) dat als de coordinator er toe besluit, dat deze uitvoerbare er nu aan toe is, om voortgezet te worden, er getest moet worden, of de programmapagina, waar we vandaan kwamen en waarheen we weer terugwillen, nog op dezelfde plaats in de kernen zit, maw. de voortzetting moet gespeeld worden als een goto-statement naar een andere, dwz. mogelijk niet aanwezige pagina. c) dat dan in het geval van afwezigheid door de nu aangevraagde trommeltransport het programma weer op de lijst van de geblokkeerde programma's voorkomt.

Als nu na voltooid transport dit programma weer bij de uitvoerbare terecht komt, maar niet direct aan bod komt en dus de pagina voor feitelijk gebruik alweer verdwenen kan zijn, dan is dit niet alleen triest, maar ook gevaarlijk: dit zou nl. betekenen, dat we de mogelijkheid van "effectloze" trommeltransporten geintroduceerd hebben. Zijn we er dan nog zeker van, dat we niet het "Na U" effect geintroduceerd hebben? Dit zou te ondervangen zijn door de kernpagina op het moment van beslissing, dat het transport zal plaatsvinden te heiligen en de ontheiliging pas plaats te laten vinden door de feitelijke voortzetting van dit programma.

1.11.2. In de andere techniek gaan we er van uit, dat het terugkeeradres van een door interruptie onderbroken programma als physisch kernadres opgeborgen wordt, maar dat tijdens uitvoerbaar wachten de pagina in kwestie altijd heilig zal zijn. De andere manier, waarop een programma bij de uitvoerbaren kan komen is via de geblokkeerden. Als blokkade altijd vanuit een coordinatorroutine plaats vindt, dan mogen we ook met een physisch adres volstaan. De coordinatorroutine is immers permanent op de kernen; de coordinatorroutine krijgt dan de plicht om er in geval van gebruikt break point voor te zorgen, dat hij zijn schepen niet achter zich verbrandt. (Door hetzij het terugkeeradres invariant te bergen, dan wel over het break point heen de aanroepende pagina te heiligen.)

Deze techniek kan in eerste instantie op twee wijzen gespeeld worden. We kunnen de heersende pagina altijd heiligen, dit vereist maatregelen bij elke sprong van de ene pagina naar de andere. Een coordinatorroutine heeft dan mogelijk de plicht om bij gebruikt break point —als het te lang kan duren— de heiliging op te heffen en na terugkeer testend terug te springen.

De andere manier is, dat we bij goto statements naar andere pagina's helemaal niets doen, maar in het geval van een interruptie de verlaten pagina heiligen en bij voortzetting ontheiligen. In dit geval moeten we in een coordinatorroutine mogelijk —als de break gegarandeerd kort duurt— deze pagina heiligen voor de duur van de break.

1.11.3. Door de bibliotheek te behandelen zoals Voorhoeve gesuggereerd heeft, kunnen we niet volstaan met in de KIE een heiligbit op te nemen. Immers: vanuit een bibliotheekpagina wordt een coordinatorroutine aangeroepen en de bibliotheekpagina wordt heilig achtergelaten. In de tussentijd komt een ander programma aan bod, dat eveneens deze bibliotheekpagina via een dergelijke coordinatorroutine verlaat. Een van de beide programma's wordt het eerste weer voortgezet: dir programma mag dan de bibliotheekpagina niet ontheiligen! Toen ik dit addertje vanonder het gras had losgewoeld, was ik wel een beetje tevreden met mezelf!

De tweede is duidelijk: de Kie heeft inplaats van een heiligbit een heiligtelling. Telling = 0 betekent profaan, telling positief betekent heilig. Heiligen betekent telling met 1 verhogen, ontheiligen betekent telling met 1 verlagen. Het welbekende "Heilig, heilig, driemaal heilig" begint een bijzonder relief te krijgen.

Opm. Moeten we per programma een "ontheiligingsplicht" bijhouden voor het geval het programma voortijdig beeindigd wordt? Ik denk van wel.

18 December 1963
E. W. Dijkstra

transcribed by Sam Samshuijzen

revised Sun, 13 Feb 2005

---

## English translation

### On drum page transports

EWD 72

On drum page transports.

1. So as not to make it too difficult, we restrict ourselves for the time being to array pages.

1.1. The need for a transport is detected by the needy program itself, and indeed while deaf. For if it were not to happen while deaf, then, after establishing that the page is indeed present, it could still be snatched away from under your fingers.

1.2. If a transport proves necessary, then we go to the coordinator, i.e. the program is placed on the waiting list. We only return into the program once the page transport has been completed; we then return into the program while deaf, and the requested array page has been sanctified.

1.2.1. The page has been sanctified with the following intention: once the transport has been completed, this program can proceed again; it is not said that it will also resume immediately. In the intervening time the page must not disappear!

1.2.2. Returning while deaf is not strictly necessary, provided that the program first selects the array element and only afterwards desanctifies the page. If we return while deaf —which we get for nothing— then we are free in the order of element selection and desanctification. The program has the duty to desanctify the array page, to select the element (order free) and to become hearing again. This too we get as a gift via the restoring return jump. A side advantage is that, when this program proceeds while still briefly deaf, the desanctification is guaranteed to take place within a foreseeable time after continuation.

1.3. I assume that in the case of a necessary array page transport the reigning program page is also sanctified. This considerably eases the return procedure. If we do this, then, I assume, the program page must also be desanctified while deaf (see 1.1., 1.9.2. et seq.).

1.4. It is unproven that the sanctification procedure cannot bring the whole thing to a standstill. We can only attempt to furnish this proof once we have a complete survey of the request procedure in the case of an absent (array) page.

1.6. I restrict myself for the time being to that part of the coordinator which will be permanently present in core, at a damned fixed location. The coordinator will comprise a number of so-called "coordinator routines"; as a rule these will be called with a deaf-making subroutine jump. Calling a coordinator subroutine does not in itself necessarily mean "abandonment of the reigning ALGOL program". As a rule it may be the case that the coordinator routine establishes whether or not the reigning program can continue to work unhindered. If so, then the ALGOL program presently continues unhindered — the machine becomes hearing upon the restoring return jump out of the coordinator routine. The other possibility is that the coordinator routine decides that this (ALGOL) program cannot continue unhindered. In that case the program is abandoned, i.e. it is no longer "the reigning program", it is placed as non-active on the coordinator's waiting list, and the coordinator chooses another program and makes that one reigning.

I am perhaps splitting hairs here; what I am also trying to do is to find a terminology. In the text of the ALGOL program many a call of a coordinator routine functions as a "possible abandonment". Within the coordinator routine the abandonment or non-abandonment of the program is a conscious matter, and we can, prior to the abandonment while deaf, and after the return, take deliberate measures (sanctifying and desanctifying, for instance).

1.7. Coordinator routines may not, "across their break point", occupy fixed memory locations holding variable information: after all, they can be activated in parallel. Functionally, the actions in question therefore continue to belong to the calling program. For reasons of space —also because they are supposed to reside permanently in core— they too, even in the case of multiprogramming, reside in core in only a single copy.

1.8. A consequence of the degree of awareness of a possible interruption is the following: an ordinary interrupt implies that, upon stack shifting, the B register must be adjusted and that we must be very careful with absolute references to stack locations. In a coordinator routine we can be much freer: after all, upon returning below the break point we can test whether the stack has shifted and, if so, adjust some addresses! (for practical reasons this had better not get out of hand!)

1.9. Examples of coordinator routines are the P operation, the V operation, and the indexing operations (writing and reading). If it turns out to be a small array, the whole thing fizzles out!

1.9.1. Note. The previous sentence strengthens me, on account of the difficult separability of "runtime system" and coordinator, in the conviction that for the time being we need not think about multiprogramming in machine code.

1.9.2. To what extent the call of a coordinator routine in the case of a break point implies "sanctification" of the calling page will depend from case to case. One might maintain that in every program the reigning page is always sacred, but I am not much taken with that, if it means that every program, regardless of how long it remains non-active, always keeps occupying at least 1 page. One can, for instance, look into whether one can make do with introducing sanctification only in those cases in which one can rely on the inhibition being lifted in the short term. It becomes clear once again that we can only assess the "soundness" of the concept of sanctity once a concrete proposal is on the table. The role of Advocatus Diaboli seems to me extremely important in this matter! It would not surprise me if our idea about sanctity were to change drastically in the course of these investigations.

1.10. The coordinator waiting list contains all the programs in the machine, subdivided into two categories, the blocked ones and the executable ones.

Blocked programs are programs that cannot proceed because they are waiting at a semaphore, for the completion of an event necessary for their continuation.

Executable programs are programs that could proceed, were it not that the X8 can help only 1 of them along. (I assume that the program being executed by the X8 continues to be ranked among the executable ones; the term "waiting list" is then somewhat curious, but let us simply resign ourselves to that. If the control really resides in the coordinator, then the term is in order.)

The effect of the P operation can be that a program now becomes blocked from executable; the effect of the V operation can be that a (different) program is transferred from the group of blocked ones to the executable ones. A program that, while in action, is interrupted by an interrupt remains ranked among the executable ones!

We can also view it thus: if in a program a P operation is initiated, then at that moment the program was in action, hence executable. Now there are two cases. Either the reigning values of the semaphores in question form no obstacle, or they do. In the first case they are decremented, the P operation is thereby completed and the program remains executable. (Whether it also remains in action is quite another chapter!) In the second case the program is taken out of the group of executable ones. The group of blocked ones therefore contains only all the programs that have got stuck in an initiated but not completed P operation.

1.11. With regard to the executable programs we can take two roads as concerns sanctification. We consider a program that was in action, has been interrupted by an interrupt, and as a result has come onto the executable part of the waiting list.

1.11.1. In the one technique we allow the program page on which we were engaged at the moment of the interrupt to disappear during the waiting. This implies: a) that in the waiting list, among the status quo data, we must store the return address in an invariant form. b) that if the coordinator decides that this executable one is now due to be continued, it must be tested whether the program page from which we came and to which we want to return again is still in the same location in core, in other words the continuation must be played as a goto statement to another, i.e. possibly absent, page. c) that then, in the case of absence, the program appears once more on the list of blocked programs by virtue of the now requested drum transport.

If now, after a completed transport, this program ends up again among the executable ones, but does not immediately get its turn and hence the page may have disappeared again before actual use, then this is not only sad, but also dangerous: for it would mean that we have introduced the possibility of "effectless" drum transports. Are we then still certain that we have not introduced the "After You" effect? This could be obviated by sanctifying the core page at the moment of deciding that the transport will take place and letting the desanctification take place only through the actual continuation of this program.

1.11.2. In the other technique we proceed on the assumption that the return address of a program interrupted by an interrupt is stored as a physical core address, but that during executable waiting the page in question will always be sacred. The other way in which a program can come among the executable ones is via the blocked ones. If blocking always takes place from within a coordinator routine, then we may again make do with a physical address. After all, the coordinator routine is permanently in core; the coordinator routine then gets the duty, in the case of a used break point, to make sure that it does not burn its bridges behind it. (Either by storing the return address in an invariant form, or by sanctifying the calling page across the break point.)

This technique can, in the first instance, be played in two ways. We can always sanctify the reigning page; this requires measures at every jump from one page to another. A coordinator routine then possibly has the duty, in the case of a used break point —if it may take too long— to lift the sanctification and, after the return, to jump back testing.

The other way is that at goto statements to other pages we do nothing at all, but in the case of an interrupt we sanctify the abandoned page and desanctify it upon continuation. In this case we must possibly, in a coordinator routine —if the break is guaranteed to be short— sanctify this page for the duration of the break.

1.11.3. By treating the library as Voorhoeve has suggested, we cannot make do with including a sanctity bit in the KIE. For: from a library page a coordinator routine is called, and the library page is left behind sacred. In the meantime another program gets its turn, which likewise abandons this library page via such a coordinator routine. One of the two programs is the first to be continued again: that program must then not desanctify the library page! When I had dug this snake out from under the grass, I was rather pleased with myself!

The second is clear: the KIE has, instead of a sanctity bit, a sanctity count. Count = 0 means profane, count positive means sacred. Sanctifying means incrementing the count by 1, desanctifying means decrementing the count by 1. The well-known "Holy, holy, thrice holy" begins to take on a special relief.

Note. Must we keep, per program, a "desanctification obligation" for the case in which the program is terminated prematurely? I think so.

18 December 1963
E. W. Dijkstra

transcribed by Sam Samshuijzen

revised Sun, 13 Feb 2005
