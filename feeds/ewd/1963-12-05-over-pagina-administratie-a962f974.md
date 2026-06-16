---
title: "Over pagina-administratie (EWD71)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD71.html
published: "1963-12-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD71.PDF
---

# Over pagina-administratie

EWD 71

Over pagina-administratie

Het idee is, om in elk programma voor elke in dit programma voorkomende trommelpagina eem zg. TPV (= trommelpagina variabele) in te voeren. De TPV's komen in de stapel van het bijbehorende programma. Ik zie minstens twee klassen:

a) opdrachten-pagina's

b) array-pagina's

De TPV's voor de opdrachten-pagina's van een programma zijn locale variabelen van het buitenste blok; zij worden geïnitialiseerd door de vertaler/inlezer.

De TPV's van grote array's worden, hoger in de stapel, geïnitialiseerd door de array-declaratie en afgevoerd door de blok-verlating.

De bedoeling van de TPV's is, om snel antwoord te kunnen krijgen op de vraag "Kaatje ben je boven?". Wij zouden dit kunnen bewerkstelligen door in de kernen een volledige —gestrekte— inhoudsopgave van de trommel(s) bij te houden, maar met duizend pagina's per trommel kost dit ons, als we niet in een onbarmhartig geschuif willen vervallen, per trommel minstens 3% van de kernen per trommel, ook al gebruik je deze trommelruimte niet.

Nu we de TPV's onderbrengen bij de locale variabelen van de programma's hebben we bereikt, dat er in de kernen niet meer TPV's onthouden hoeven te worden, dan trommelpagina's, waarin mogelijk interesse kan bestaan.

Een gevolg hiervan is dat de interne formulering van het objectprogramma er vrij ongevoelig voor is, welke trommelpagina's nu eigenlijk door dit programma bezet worden: dit staat nl. niet meer door het hele programma verspreid, maar (éénmalig) in de TPV's. Voor de huishouding op de trommel neem ik aan, dat dit voordelen biedt; het betekent wel, dat de wijze, waarop bibliotheekprocedures naar elkaar moeten kunnen refereren me met enige zorg vervult (met "later zorg", om precies te wezen).

Naast de, in de programma's ondergebrachte, TPV's hebben wij ook nog de "inhoudsopgave van de kernen". Deze bevat een element per kernpagina, de omvang van de kernpaginatabel kunnen we meteen instellen op maximaal gebruik van de kernen. (Gebruiken we een stuk van die inhoudsopgave niet, dan betekent dit nl. dat we een stuk van het kerngeheugen niet nodig blijken te hebben, maar dan hadden we het kennelijk breed en konden we het ook breed laten hangen.) De relatieve prijs van deze tabel is de omvang van een element/omvang van een kernpagina. Dit is weer een argument voor niet te kleine pagina's. Elementen van deze inhoudsopgave noem ik KIE (Kernen Inhoudsopgave Elementen).

We gaan nu onderzoeken, wat alzo in een TPV en een KIE onthouden moet worden. De tendens zal zijn, om de omvang van een TPV klein te houden —hiervan kunnen we er immers zoveel hebben. In een TPV moeten we in de eerste plaats onthouden, of zich een copie in de kernen bevindt. Ik stel voor:

TPV ≥ +0 betekent: pagina in kernen aanwezig

TPV ≤ –0 betekent: pagina niet in kernen aanwezig.

In het geval TPV ≥ +0 moeten we dus aangeven, waar deze pagina zich dan wel bevindt. We kunnen opgeven: het beginadres in de kernen of het adres van de overeenkomstige KIE (deze zijn uit elkaar af te leiden). Ik denk dat het KIE-adres het handigste is, de KIE hebben we nl. waarschijnlijk tòch nodig. (Ik laat nog in het midden, of we in de KIE wellicht het kernenadres van de pagina opnemen; dit is dan abundant, maar kon ons wel eens op een welkome wijze geschuif besparen. Zodra we dit doen, zijn we bovendien van tweemachten af als opgelegde lengte van een KIE.)

In het geval TPV ≤ –0 moeten we nog onderscheid maken tussen "trommelpagina gereserveerd" en "nog geen trommelpagina gereserveerd". Dit laatste kunnen we nl. tegenkomen bij een TPV, die hoort bij een groot array. In het eerste geval kunnen we de TPV het beginadres op de trommel laten bevatten, het tweede geval kunnen we b.v. karakteriseren door TPV = –0.

Referentie naar een trommelpagina met TPV ≤ –0 loopt nu verschillend. In beide gevallen moet er een kernpagina gekozen worden. Als TPV ≠ –0, dan is er een —nu een wel-omschreven— transportplicht van trommel naar kernen; als TPV = –0, dan moeten we uit de "vrije trommelpaginalijst" een nieuwe trommelpagina aanvragen, van deze "bezetting" nota nemen en niet feitelijk transporteren. (Deze transport is overbodig en misschien zelfs schadelijk: waarom zou je b.v. nodeloos eisen, dat bij lang niet gebruikte trommelpagina's de pariteit zou deugen?)

Opm. De toekenning van trommelpagina kan ook uitgesteld worden, totdat werkelijk gedumpt moet worden.

Nu gaan wij over tot de KIE. Als wij voor een TPV slechts één woord ter beschikking stellen, dan betekent dit, dat we in de TPV geen ruimte meer hebben om het trommeladres op te bergen, zodra de pagina zich op de kernen bevindt. M.a.w.: dan hebben we de plicht om in de bijbehorende KIE het trommeladres bij te houden.

Wat we in een KIE allemaal gaan bijhouden, weet ik nog niet zo precies; de KIE's zullen nl. ook uitgebreid door de coordinator bespeeld worden. Ik zie zo met het blote oog drie toestanden voor een kernpagina, toegekend, wisselend en vogelvrij.

Een kernpagina wordt vogelvrij, zodra er in de inhoud geen interesse meer is. Het treedt op bij blokverlating, als dit blok grote arrays bevat. Bij deze blokverlating moeten de TPV's, die hierdoor boven de stapeltop komen, gescand worden. Vinden we een TPV = –0, dan hoeven we niets te doen, vinden we een TPV < 0, dan moeten we de in de TPV vermelde trommelpagina aan de lijst van vrijen toevoegen; vinden we een TPV > 0, dan is dat een KIE-adres. De in deze KIE vermelde trommelpagina worde aan de lijst van vrijen toegevoegd en in de bijbehorende KIE moeten we nu "vogelvrij" noteren. Voor de opdrachtenpagina's treedt dit op bij definitieve beÎindiging van het programma.

Het proces "wisselend" moeten wij waarschijnlijk separaat in de KIE aangeven: zodra een kernpagina een nieuwe rol heeft toebedeeld gekregen en deze toekenning impliceert trommel-transporten, dan mag de coordinator niet aan deze kernpagina komen: de coordinator mag aan deze kernpagina pas een andere rol toekennen als het transport voltooid is. (Hetzelfde respect moet de coordinator hebben voor pagina's, die als bufferruimte voor autonome transporten van en naar de communicatie-apparatuur fungeren.)

Wat zich in de KIE van een toegekende kernpagina moet bevinden, is wat beter te overzien.

Ten eerste moet in de KIE onthouden worden het bijbehorende beginadres op de trommel— we hebben immers aangenomen, dat er in de TPV van een aanwezige pagina daarvoor geen ruimte meer zou zijn. Ten tweede moet de KIE een verwijzing naar de TPV bevatten zoals de TPV een verwijzing naar de KIE bevat. (We zijn immers bezig met een soort dubbele boekhouding.)

Als de coordinator een vrije kernpagina zoekt (of moet maken), dan neem ik aan, dat hij het kerngeheugen in zijn totaliteit beschouwt d.w.z. inspectie pleegt in de KIE-tabel (zie ook onder, bij de "look back" informatie). Als de coordinator dan gekozen heeft, welke kernpagina nu van rol zal veranderen, dan moet bij de er aanvankelijk bijbehorende TPV de verwijzing naar deze KIE weggehaald worden. Op grond van de afspraak in de vorige alinea doet de coordinator dit, door aldaar, behalve TPV negatief te maken, ook het trommeladres weer in te vullen. (Dit suggereert, om als KIE-onderdeel, dat in de vorige alinea genoemd is, zonder meer te kiezen: de TPV-waarde bij afwezigheid.)

Het is even de vraag, hoe we deze verwijzing naar TPV denken te doen. Een mogelijkheid is "physisch adres". Dit geeft ons dan wel de plicht om bij stapel-verschuiving de KIE-lijst af te lopen, om te kijken welke adressen daardoor bijgewerkt moeten worden. Een andere mogelijkheid is, om in de KIE de TPV te localiseren ten opzichte van de stapelbodem. Tegen dit laatste lijkt me geen bezwaar. (Wijziging van KIE's komt toch alleen maar voor bij pagina-wisseling.) Wel betekent dit, dat we in de KIE moeten vermelden, bij welk programma de KIE hoort.

(We stappen hier even buiten de gezichtskring van elk individueel programma. Ik neem aan, dat de coordinator alle onder behandeling zijnde programma's kent onder een nummer, dat aan dit programma blijft voorbehouden, totdat het de machine uitgaat. Ik stel tevens dat hij —ook om andere redenen— bij een nummer de stapelbodem zal kunnen vinden.)

Verder zullen we in een KIE, die beantwoordt aan een toegekende kernpagina, moeten noteren, dat deze kernpagina niet tot een stapel behoort. (Stapelpagina's zijn onze vierde groep: ze moeten ook met respect behandeld worden, omdat ze in principe op de kernen blijven staan en slechts met zorg verschoven kunnen worden.)

Verder zullen we in een KIE moeten noteren, of de inhoud van de kernpagina soms identiek is met die van de overeenkomstige trommelpagina. Dit geldt nl. voor alle opdrachten-pagina's; het kan tevens gelden voor getallen-pagina's. (Een betrekkelijk lang constant array hoeft nooit teruggeschreven te worden.) Dit compliceert de assignment aan het element van een groot array. Dat wordt, vrees ik, toch niet zo'n triviale handeling en het zetten van een bit "beschreven" kan er, denk ik, nog wel bij.

Tenslotte is KIE de aangewezen plaats om "look back" informatie in op te slaan, d.w.z. notitie bij te houden van getoonde interesse. Mochten we ooit er toe overgaan een begrip als "starheid" te introduceren dan is ook hiervoor de KIE de aangewezen plaats. Bij gebrek aan beter stel ik me voorlopig maar voor dat we een Naur-achtig interesse-getal in de KIE zullen noteren. (Deze vorm van "look back" volgt de geschiedenis van kernpagina's; zodra een pagina teruggaat naar de trommel verliezen we hem geheel uit het oog. We zouden getempteerd kunnen zijn, om dan in de TPV nog een of ander interesse-getal bij te houden. Experimenten hebben aangetoond —voorzover experimenten dit kunnen aantonen— dat dit niet profijtelijk is: de strategie wordt dan nl. beïnvloed door een gebeuren dat te lang geleden is om nog actueel te zijn.)

transcribed by Sam Samshuijzen

revised Thu, 3 Mar 2005

---

## English translation

### On page administration

EWD 71

On page administration

The idea is to introduce in every program, for every drum page occurring in this program, a so-called TPV (= drum page variable). The TPV's go into the stack of the corresponding program. I see at least two classes:

a) instruction pages

b) array pages

The TPV's for the instruction pages of a program are local variables of the outermost block; they are initialized by the translator/reader.

The TPV's of large arrays are, higher up in the stack, initialized by the array declaration and removed by block exit.

The purpose of the TPV's is to be able to obtain a quick answer to the question "Kaatje, are you upstairs?". We could accomplish this by maintaining in core a complete —fully laid-out— table of contents of the drum(s), but with a thousand pages per drum this costs us, if we do not wish to lapse into a merciless shuffling, at least 3% of core per drum, even if you do not use this drum space.

Now that we house the TPV's among the local variables of the programs, we have achieved that in core no more TPV's need be retained than there are drum pages in which interest may possibly exist.

A consequence of this is that the internal formulation of the object program is fairly insensitive to which drum pages are actually occupied by this program: for this is no longer spread throughout the entire program, but (one single time) in the TPV's. For the housekeeping on the drum I assume that this offers advantages; it does mean, however, that the manner in which library procedures must be able to refer to one another fills me with some concern (with "later concern", to be precise).

Besides the TPV's, housed in the programs, we also have the "table of contents of core". This contains one element per core page; we can set the size of the core-page table right away to maximal use of core. (If we do not use part of that table of contents, then this means that we turn out not to need part of core store, but in that case we evidently had it to spare and could let it hang loose.) The relative price of this table is the size of an element/size of a core page. This is again an argument for pages that are not too small. Elements of this table of contents I call KIE (Core Table-of-contents Elements).

We will now investigate what must be retained in a TPV and a KIE. The tendency will be to keep the size of a TPV small —for of these we can after all have so many. In a TPV we must in the first place retain whether a copy resides in core. I propose:

TPV ≥ +0 means: page present in core

TPV ≤ –0 means: page not present in core.

In the case TPV ≥ +0 we must therefore indicate where this page does in fact reside. We can specify: the start address in core or the address of the corresponding KIE (these are derivable from one another). I think that the KIE address is the most convenient, for we probably need the KIE anyway. (I leave open for now whether we should perhaps include in the KIE the core address of the page; this would then be redundant, but might well save us shuffling in a welcome manner. As soon as we do this, we are moreover freed from powers of two as the imposed length of a KIE.)

In the case TPV ≤ –0 we must further distinguish between "drum page reserved" and "no drum page yet reserved". The latter we can after all encounter with a TPV belonging to a large array. In the first case we can let the TPV contain the start address on the drum; the second case we can e.g. characterize by TPV = –0.

A reference to a drum page with TPV ≤ –0 now proceeds differently. In both cases a core page must be chosen. If TPV ≠ –0, then there is a —now well-defined— transport obligation from drum to core; if TPV = –0, then we must request a new drum page from the "free drum-page list", take note of this "occupation" and not actually transport. (This transport is superfluous and perhaps even harmful: why would one e.g. needlessly require that the parity of long-unused drum pages be sound?)

Note. The assignment of the drum page can also be postponed until a dump must actually take place.

We now turn to the KIE. If we make available only one word for a TPV, then this means that in the TPV we have no more room to store the drum address once the page resides in core. In other words: then we have the obligation to maintain the drum address in the corresponding KIE.

What exactly we shall maintain in a KIE I do not yet know quite precisely; for the KIE's will also be extensively played upon by the coordinator. With the naked eye I see three states for a core page: assigned, switching, and outlawed.

A core page becomes outlawed as soon as there is no longer any interest in its contents. This occurs at block exit, if this block contains large arrays. At this block exit the TPV's that thereby come above the stack top must be scanned. If we find a TPV = –0, then we need do nothing; if we find a TPV < 0, then we must add the drum page mentioned in the TPV to the list of free ones; if we find a TPV > 0, then that is a KIE address. The drum page mentioned in this KIE shall be added to the list of free ones, and in the corresponding KIE we must now note "outlawed". For the instruction pages this occurs upon definitive termination of the program.

The "switching" process we must probably indicate separately in the KIE: as soon as a core page has been allotted a new role and this assignment implies drum transports, the coordinator may not touch this core page: the coordinator may assign this core page another role only when the transport is completed. (The coordinator must have the same respect for pages serving as buffer space for autonomous transports to and from the communication apparatus.)

What must reside in the KIE of an assigned core page is somewhat more surveyable.

Firstly, the corresponding start address on the drum must be retained in the KIE— for we have after all assumed that in the TPV of a present page there would no longer be room for that. Secondly, the KIE must contain a reference to the TPV just as the TPV contains a reference to the KIE. (For we are engaged in a kind of double-entry bookkeeping.)

When the coordinator seeks (or must make) a free core page, then I assume that it considers core store in its totality, i.e. carries out an inspection in the KIE table (see also below, under the "look back" information). When the coordinator has then chosen which core page is now to change role, then the reference to this KIE must be removed from the TPV initially belonging to it. On the basis of the convention in the previous paragraph the coordinator does this by, besides making the TPV negative, also filling in the drum address there again. (This suggests choosing, as the KIE component mentioned in the previous paragraph, simply: the TPV value in the absence [of the page].)

It is for a moment a question how we intend to make this reference to the TPV. One possibility is "physical address". This then gives us the obligation, upon stack displacement, to run through the KIE list to see which addresses must thereby be updated. Another possibility is to localize the TPV in the KIE with respect to the stack bottom. Against the latter I see no objection. (Modification of KIE's only occurs upon page switching anyway.) It does mean, however, that in the KIE we must state to which program the KIE belongs.

(We step here for a moment outside the field of view of each individual program. I assume that the coordinator knows all programs under treatment by a number that remains reserved for this program until it leaves the machine. I posit moreover that it —also for other reasons— will be able to find the stack bottom for a given number.)

Further we shall, in a KIE corresponding to an assigned core page, have to note that this core page does not belong to a stack. (Stack pages are our fourth group: they too must be treated with respect, because in principle they remain in core and can be displaced only with care.)

Further we shall have to note in a KIE whether the contents of the core page happen to be identical with those of the corresponding drum page. For this holds for all instruction pages; it can also hold for number pages. (A relatively long constant array never needs to be written back.) This complicates the assignment to the element of a large array. That, I fear, will not be such a trivial operation anyway, and the setting of a "written" bit can, I think, be thrown in as well.

Finally, the KIE is the designated place to store "look back" information, i.e. to keep a record of interest shown. Should we ever proceed to introduce a concept such as "rigidity", then the KIE is also the designated place for this. For want of anything better I imagine, provisionally, that we shall note a Naur-like interest number in the KIE. (This form of "look back" follows the history of core pages; as soon as a page goes back to the drum we lose sight of it entirely. We might be tempted then to maintain some interest number or other in the TPV. Experiments have shown —insofar as experiments can show this— that this is not profitable: for the strategy then becomes influenced by an event too long ago to still be relevant.)

transcribed by Sam Samshuijzen

revised Thu, 3 Mar 2005
