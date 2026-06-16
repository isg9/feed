---
title: "Contractie en expansie (EWD235)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD02xx/EWD235.html
published: "1968-05-12T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd02xx/EWD235.PDF
---

# Contractie en expansie

Samenvatting. Twee operaties, “contractie” resp. “expansie” gedoopt, worden onderkend. Zij worden bestudeerd in hun samenhang met assemblage, adresarithmetiek, geheugenindeling en protectie. Een en ander lijkt er op te wijzen, dat het begrip “adresruimte” en deszelfs zinvolheid voor een hernieuwd critisch onderzoek in aanmerking komen. NB. Als associatieve geheugens op grote schaal ter beschikking komen, verliest dit onderzoek een aanzienlijk stuk van zijn relevantie.

1. Open en gesloten adressering.

Ik spreek van een gesloten adressering, wanneer de objecten van een verzameling geidentificeerd zijn door consecutief oplopende nummers, zeg bij nul te beginnen. Een gesloten adressering laat zich implementeren door een op consecutieve geheugenplaatsen opgeborgen tabelletje: de adresarithmetiek —dwz. de overgang van identificerend nummer naar ~physisch adresimpliceert optelling van identificerend nummer en ~physisch beginadres van het tabelletje.

Ik spreek van een open adressering wanneer de objecten van een verzameling geidentificeerd zijn door (nonnegatieve) nummers, waarbij het maximale nummer (veel) groter is dan het aantal objecten in de verzameling. In principe is een open adressering op dezelfde wijze te implementeren als een gesloten, als we nl. de gaten in de tabel voor lief nemen. Ik ga echter van een open adressering spreken als we ons dit niet meer kunnen veroorloven en we naar andere technieken moeten omzien (hashed adressing, associatieve geheugens, naamlijsten die gescand moeten worden etc.) Opm. De openheid van een adressering is dus een gradueel begrip: hij kan zo'n klein beetje open zijn, dat we hem nog als gesloten kunnen beschouwen.

Uitgangspunt van dit onderzoek is de observatie dat een beroep op open adressering vaak zo kostbaar is, dat men zoekt naar wegen om het aantal malen dat dit moet gebeuren, te reduceren. Het is hier, dat de operatie's contractie en expansie een rol spelen.

2. De operaties “contractie” en “expansie”.

De operaties “contractie” en “expansie” zijn elkaars inverse.

Gegeven een grote verzameling objecten, waarvan ik eenvoudshalve aanneem, dat zij hierin door een gesloten adressering geidentificeerd zijn. We noemen dit even “de globale adressering”. Indien in een zekere context slechts naar een relatief kleine keuze van deze objecten gerefereerd wordt, dan is de globale adressering betrokken op deze keuze een open adressering. We spreken van contractie wanneer de gekozen objecten ten bate van de genoemde context aansluitend hernummerd worden; deze aldus ingevoerde “locale adressering” is betrokken op deze keuze dan gesloten. De locale adressering is een exploitatie van de wetenschap dat de context in kwestie zich afspeelt in een gecontraheerd universum.

Gegeven een context die via een gesloten adressering elementen van een verzameling bespeelt, dan spreken we van expansie als we genoemde verzameling gaan beschouwen als deel van een grotere verzameling en daarmee de elementen als elementen uit een grotere verzameling. De overgang van de locale adressering naar een globale is veronachtzaming van de wetenschap, dat de genoemde context zich in het gecontraheerde universum afspeelt.

(Ik zal in nog iets ruimer verband van expansie spreken, nl. als onze objecten informatiehoeveelheden zijn, voor elk waarvan een aansluitend stuk geheugen ter beschikking moet worden gesteld. De overgang van identificeren door nummer naar identificeren door beginadres zal ik ook expansie noemen.)

3. De organisatie van een subroutinebibliotheek.

In deze sectie beschrijf ik een subroutineorganisatie die de kenners zal treffen als een verder doorgevoerde automatisering van de subroutineorganisatie van de EDSAC 1 (1949). De keuze is hierop gevallen omdat dit het eenvoudigste voorbeeld lijkt waarmee we kunnen illustreren hoe expansie en contractie functioneren.

We beschouwen een subroutinebibliotheek met een zeer groot aantal subroutines. Ik neem aan, dat de routines in de bibliotheek geidentificeerd zijn door catalogusnummer CN en dat catalogusnummers in de totale bibliotheek een gesloten adressering vormen.

Subroutineteksten verwijzen als regel naar zichzelf en naar enkele andere subroutines in de bibliotheek. Men zou dit in termen van catalogusnummers kunnen doen; in plaats daarvan wordt ten bate van de formulering van de routine contractie gepleegd en verwijst de subroutinetekst in een gesloten locale terminologie naar zichzelf en die paar anderen. (In de EDSAC-organisatie heetten de locale namen “code letters”, bij de ARMAC en de X1 hebben ze als “sluitletters” nog jaren doorgeleefd.) Bij elke subroutine hoort een zg. “expansietabel”, die de locale terminologie vertaalt in catalogusnummers; de expansietabel bevat dus een rijtje CN-waarden.

Zodoende kan een subroutinetekst zelf geformuleerd worden op een wijze die onafhankelijk is van de bibliotheek waarvan hij deel uitmaakt (dwz. onafhankelijk van de omvang van de bibliotheek en de hierin gebruikte catalogusnummers). De locale terminologie heeft geen groter onderscheidend vermogen nodig dan vereist voor het onderscheid tussen de subroutines waarnaar hij werkelijk verwijst. Als om een of andere reden herindeling van de bibliotheek met verandering van de catalogusnummers gewenst is, dan beperkt de hieruit voortvloeiende bijwerkplicht zich tot de goed geisoleerde expansietabellen. Tenslotte biedt de expansietabel de uitwendige referenties op een presenteerblaadje.

Nu het probleem van assemblage; opdat dit een probleem zij neem ik aan dat de bibliotheek veel te groot is om permanent in het werkgeheugen te bivakkeren; voorts neem ik aan dat slechte een kleine fractie van de bibliotheek in een programma wordt opgenomen.

Als nu benodigde teksten en bijbehorende expansietabellen onveranderd in het geheugen zijn gekomen, welke mechanismen hebben we dan nog nodig?

Om steeds de heersende locale terminologie te kunnen interpreteren veronderstel ik aanwezig een toegewijd register, bevattende het beginadres van de vigerende expansietabel. Daar vinden we echter in eerste instantie het catalogusnummer terwijl men, om een subroutine aan te kunnen roepen, beginadressen van tekst en expansietabel moet weten. Sinds invoer zijn deze gegevens bekend; doordat echter maar een klein gedeelte van de bibliotheek in het werkgeheugen is opgenomen, vormt net catalogusnummer hiervoor een uiterst open adressering.

Het antwoord hierop is contractie. Voor de in het programma opgenomen routines voert men een gesloten adressering in; laten we deze de “selectienummers” SN noemen. De taak van het assemblageprogramma omvat nu

1) het toekennen van selectienummers aan op te nemen routines

2) het bijwerken van op te nemen expansietabellen, zodat elke entry (ook) het toegekende selectienummer bevat

3) het invullen van de zg. “selectietabel”; dit is een expansietabel voor het hele programma voor de overgang van selectienummer naar de beginadressen.

Assemblage presenteert zich hier dus als contractie; omdat dat een vervelende operatie is loont het de moeite dit eenmalig per programma te doen. Subroutineteksten zijn ongewijzigd gebleven, de expansietabellen zijn per entry uitgebreid.

4. Vermindering van diepte van indirecte adressering.

De overgang van de locale terminologie van een subroutinetekst naar physisch adres kost nu tweemaal het raadplegen van een expansietabel

a) het raadplegen van de expansietabel van de heersende subroutine geeft een selectienummer SN

b) met deze SN-waarde raadplegen van de selectietabel van het programma geeft een physisch adres.

Als deze tweevoudige expansie te tijdrovend geacht wordt staan ons ter bespoediging twee wegen open, nl. het kortsluiten van de eerste, resp. de tweede expansie!

Methode A. Men laat het assemblageprogramma de subroutineteksten herschrijven door de locale terminologie te vervangen door SN’s. De expansietabellen hoeven dan tijdens uitvoering niet meer geraadpleegd te worden: onder controle van de door de programmatekst geleverde SN-waarde kan in de selectietabel meteen het adres gevonden worden.

Methode B. De expansietabellen van de subroutines worden uitgebreid, zodat elke entry nu ook de beginadressen bevat, waardoor het frequent raadplegen van de selectietabel vervalt.

Methode A, die het meest de kant van klassieke assemblage uitgaat, heeft als grootste voordelen

1) het toegewijde register dat verwijst naar het begin van de heersende expansietabel kan vervallen (inclusief de hiermee geassocieerde reden herstelprocedures).

2) van elke subroutine komt het beginadres slechts 1 keer in de administratie voor (nl. in de selectietabel) wat herziening van een dergelijk adres wel vergemakkelijkt.

Het nadeel van methode A is echter, dat de programmatekst niet meer onveranderd in het geheugen staat maar in een vorm, die afhangt van het grotere geheel, waar deze tekst nu deel van uitmaakt. En hoe langer ik erover nadenk, hoe zwaarder ik die prijs ga vinden.

Het punt is, dat expansie structuur verduistert en daarom zou men liefst het resultaat van de expansieoperatie zo volatiel mogelijk houden. Voelt men zich desondanks toch geroepen om een resultaat van expansie te laten beklijven, dan is dit minder schadelijk wanneer dit beklijven tot de tabellen beperkt blijft dan wanneer het expansieresultaat tot je laagste niveau, de programmatekst, diffundeert.

Ik moge hiervan een paar voorbeelden noemen..

Methode A vraagt om iets als een “linkage editor”, die bij methode B practisch vervalt.

Methode A maakt het onmogelijk om verschillende programma’s van de zelfde subroutinetekst gebruik te laten maken.

Als men de ene overblijvende expansie versnellen wil, dan vraagt dit bij methode A om een stukje heel snel geheugen zolang als de selectietabel (en als dat er niet is —de selectietabel groeit door combinatie!— om een klein associatief geheugen, dat de relevantste beginadressen bijhoudt); bij methode B kan volstaan worden met een snel geheugentje groot genoeg voor een expansietabel.

20 juni 1968
E.W.Dijkstra

N.V. PHILIPS’ COMPUTER INDUSTRIE

APELDOORN

transcribed by Bart Vreugdenhil

revised
29-Dec-2011

---

## English translation

### Contraction and expansion

Summary. Two operations, baptized “contraction” and “expansion” respectively, are distinguished. They are studied in their connection with assembly, address arithmetic, store layout and protection. All of this seems to indicate that the notion of “address space” and its meaningfulness are candidates for renewed critical examination. NB. If associative stores become available on a large scale, this investigation loses a considerable part of its relevance.

1. Open and closed addressing.

I speak of a closed addressing when the objects of a set are identified by consecutively ascending numbers, say beginning at zero. A closed addressing lends itself to implementation by a little table stored at consecutive store locations: the address arithmetic —i.e. the transition from identifying number to ~physical address— implies addition of identifying number and ~physical starting address of the little table.

I speak of an open addressing when the objects of a set are identified by (nonnegative) numbers, where the maximal number is (much) larger than the number of objects in the set. In principle an open addressing can be implemented in the same way as a closed one, namely if we accept the holes in the table for what they are. I shall, however, begin to speak of an open addressing when we can no longer afford this and we must look around for other techniques (hashed addressing, associative stores, name lists that have to be scanned etc.) Note. The openness of an addressing is thus a gradual notion: it can be just so slightly open that we can still regard it as closed.

The point of departure of this investigation is the observation that an appeal to open addressing is often so costly that one looks for ways to reduce the number of times this has to happen. It is here that the operations contraction and expansion play a role.

2. The operations “contraction” and “expansion”.

The operations “contraction” and “expansion” are each other's inverse.

Given a large set of objects, of which I assume for the sake of simplicity that they are identified within it by a closed addressing. Let us call this for the moment “the global addressing”. If within a certain context only a relatively small selection of these objects is referred to, then the global addressing as applied to this selection is an open addressing. We speak of contraction when the selected objects are renumbered contiguously for the benefit of the said context; this “local addressing” thus introduced is then closed as applied to this selection. The local addressing is an exploitation of the knowledge that the context in question takes place within a contracted universe.

Given a context that operates upon elements of a set via a closed addressing, we then speak of expansion when we come to regard the said set as part of a larger set and thereby the elements as elements of a larger set. The transition from the local addressing to a global one is a disregard of the knowledge that the said context takes place within the contracted universe.

(I shall, in a somewhat wider connection still, speak of expansion, namely when our objects are quantities of information, for each of which a contiguous piece of store must be made available. The transition from identifying by number to identifying by starting address I shall likewise call expansion.)

3. The organization of a subroutine library.

In this section I describe a subroutine organization which will strike the connoisseurs as a further automation of the subroutine organization of the EDSAC 1 (1949). The choice has fallen on this because it seems the simplest example with which we can illustrate how expansion and contraction function.

We consider a subroutine library with a very large number of subroutines. I assume that the routines in the library are identified by catalogue number CN and that catalogue numbers in the total library form a closed addressing.

Subroutine texts as a rule refer to themselves and to a few other subroutines in the library. One could do this in terms of catalogue numbers; instead, for the benefit of the formulation of the routine, contraction is performed and the subroutine text refers, in a closed local terminology, to itself and to those few others. (In the EDSAC organization the local names were called “code letters”; with the ARMAC and the X1 they lived on for years as “sluitletters” [closing letters].) With each subroutine there belongs a so-called “expansion table”, which translates the local terminology into catalogue numbers; the expansion table thus contains a little row of CN-values.

In this way a subroutine text can itself be formulated in a manner that is independent of the library of which it forms a part (i.e. independent of the size of the library and the catalogue numbers used within it). The local terminology needs no greater discriminating power than is required for the distinction between the subroutines to which it actually refers. If for one reason or another a reorganization of the library, with a change of the catalogue numbers, is desired, then the resulting obligation to update is confined to the well-isolated expansion tables. Finally, the expansion table presents the external references on a little serving tray.

Now the problem of assembly; for this to be a problem I assume that the library is far too large to bivouac permanently in the working store; furthermore I assume that only a small fraction of the library is incorporated into a program.

Now if the needed texts and the corresponding expansion tables have entered the store unchanged, what mechanisms do we then still need?

In order to be able to interpret the prevailing local terminology at all times, I assume there is present a dedicated register, containing the starting address of the current expansion table. There, however, we find in the first instance the catalogue number, whereas one must, in order to be able to call a subroutine, know the starting addresses of text and expansion table. Since input these data are known; because, however, only a small portion of the library has been incorporated into the working store, precisely the catalogue number forms for this an extremely open addressing.

The answer to this is contraction. For the routines incorporated into the program one introduces a closed addressing; let us call these the “selection numbers” SN. The task of the assembly program now comprises

1) the assigning of selection numbers to routines to be incorporated

2) the updating of expansion tables to be incorporated, so that each entry (also) contains the assigned selection number

3) the filling in of the so-called “selection table”; this is an expansion table for the whole program for the transition from selection number to the starting addresses.

Assembly thus presents itself here as contraction; because that is a tedious operation it is worth the trouble to do this once per program. Subroutine texts have remained unchanged, the expansion tables have been extended per entry.

4. Reduction of depth of indirect addressing.

The transition from the local terminology of a subroutine text to physical address now costs the consulting of an expansion table twice

a) the consulting of the expansion table of the prevailing subroutine yields a selection number SN

b) consulting, with this SN-value, the selection table of the program yields a physical address.

If this twofold expansion is deemed too time-consuming, two ways stand open to us for expediting it, namely the short-circuiting of the first, respectively the second expansion!

Method A. One has the assembly program rewrite the subroutine texts by replacing the local terminology with SN's. The expansion tables then no longer have to be consulted during execution: under control of the SN-value supplied by the program text the address can be found at once in the selection table.

Method B. The expansion tables of the subroutines are extended, so that each entry now also contains the starting addresses, whereby the frequent consulting of the selection table lapses.

Method A, which goes most in the direction of classical assembly, has as its greatest advantages

1) the dedicated register that refers to the beginning of the prevailing expansion table can lapse (including the recovery procedures associated with it).

2) of each subroutine the starting address occurs only once in the administration (namely in the selection table), which does facilitate revision of such an address.

The disadvantage of method A, however, is that the program text no longer stands in the store unchanged but in a form which depends on the larger whole of which this text now forms a part. And the longer I think about it, the heavier I come to find that price.

The point is that expansion obscures structure and therefore one would preferably keep the result of the expansion operation as volatile as possible. If one nevertheless feels called upon to let a result of expansion persist, then this is less harmful when this persistence remains confined to the tables than when the expansion result diffuses down to your lowest level, the program text.

Let me name a couple of examples of this..

Method A calls for something like a “linkage editor”, which with method B practically lapses.

Method A makes it impossible to let different programs make use of the same subroutine text.

If one wants to speed up the one remaining expansion, then with method A this calls for a piece of very fast store as long as the selection table (and if there is none —the selection table grows through combination!— for a small associative store that keeps the most relevant starting addresses); with method B a little fast store large enough for one expansion table will suffice.

20 June 1968
E.W.Dijkstra

N.V. PHILIPS’ COMPUTER INDUSTRIE

APELDOORN

transcribed by Bart Vreugdenhil

revised
29-Dec-2011
