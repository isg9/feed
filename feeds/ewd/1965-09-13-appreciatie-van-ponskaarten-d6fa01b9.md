---
title: "Appreciatie van ponskaarten (EWD139)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD139.html
published: "1965-09-13T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD139.PDF
---

# Appreciatie van ponskaarten

EWD139 - 0

Appreciatie van ponskaarten

In het volgende hoop ik overtuigend te motiveren waarom ik de vraag, of de EL X8 van de THE met ponskaartenapparatuur uitgerust moet worden, met een volmondig "Neen" beantwoord. De uitgesprokenheid van mijn oordeel is alleen dan gerechtvaardigd, wanneer ik in mijn motivering het medium ponskaart in alle fairheid benader. Het volgende is een proeve van mijn fatsoensvermogen in die richting.

Ik observeer, dat de ponskaart groot geworden is in de tijd van vóór de general purpose rekenmachine. Om dit medium zijn beperkt programmeerbare special purpose machines ontstaan, zoals reproducers, sorteermachines, tabulators, collators, controleponsers etc..

Het is mijn overtuiging dat veel van het mogelijke profijt, van dit medium slechts dan geplukt kan worden, als men metterdaad over voldoende van de genoemde, op dit medium afgestemde apparatuur beschikt: het voordeel, dat ponskaarten goed sorteerbaar zijn is zonder sorteermachine moeilijk realiseerbaar. (Dit neemt niet weg, dat de aanwezigheid van een general purpose automaat wel verschil maakt: het feit, dat een snelle sorteermachine aan de X1 kon worden aangesloten, waarbij de X1 de sortering kon besturen, heeft er vaak toe geleid, dat sorteerprocessen, die nu immers op veel complexere criteria konden worden uitgevoerd, na een X1 plaats vonden, dit met drastische beperking van het aantal doorgangen. Door de beperkte stopmogelijkheden van de sorteermachine betekende dit voor de rekenautomaat helaas wel de introductie van een essentiële haastsituatie.)

De stijl, waarin men met ponskaarten werkt is in het verleden natuurlijk sterk door de eigenschappen van de speciale ponskaartmachines beïnvloed. Het is mijn overtuiging, dat deze invloed zal blijven, met name daar, waar de ponskaart nauw verbonden is met leesbaar schrift. (Het is duidelijk, dat er sinds productie door en voor automaten volledig nieuwe gebruiksvormen ontstaan zijn zoals "a pack of binary cards". Dit treft mij nu juist als het niet typische ponskaartengebruik: dat hier een heleboel informatie over losse kaarten verdeeld wordt, lijkt mij meer vervelend dan prettig en ik dacht, dat dit gebruik nooit ontstaan zou zijn, als de ponskaart er niet al was geweest.)

Deze gebruiksstijl manifesteert zich bv. in de gangbare controletechnieken, kort samen te vatten als "tweemaal is scheepsrecht". Ponskaartlezers en ponskaartponsers zijn voorzien van een extra controlestation, wat verder in de

EWD139 - 1

de baan, met de hand vervaardigde kaarten worden na controleponsing gecontroleerd. (De kaarten, die bij de controleponsing zijn vervaardigd plegen van een speciale markering te zijn voorzien: ervaring heeft geleerd, dat ze suspect zijn.) Naast pariteitschecks -tegen incidenteel slecht functioneren van de apparatuur - gebruikt men tegenwoordig waar mogelijk - bv. bij programmateksten! - liever structurele redundantie, waardoor de controle zich ook over het oorspronkelijk manuscript uitstrekt. (Het feilloos ponsen, waar dan ook in, van monotoon getallenmateriaal, van staten, blijft een probleem, waarvoor ik geen smakelijke oplossing ken. De moraal lijkt me, dat dit vermeden dient te worden.

Erger is wat ik zou willen aanduiden met de traditie der parallelle verwerking en de tyrannie der 80 kolommen.

Parallelle verwerking impliceert een a priori kennis over de omvang van een te verwerken hoeveelheid informatie. Door bv. integers te laten optreden als constituenten in een text met een recursief gedefinieerde syntaxis, is aan deze integers geen bovengrens opgelegd. In de ponskaartenwereld is het echter gebruikelijk de omvang van een informatie-eenheid - bv. een integer - door ad-hoc conventies vast te stellen (bv. het veld van de kolommen 7 t/m 12).

Om te beginnen merken wij op, dat het uitsparen der separatoren geen onverdeelde winst is: hier staat nl. tegenover, dat men altijd het maximum aan posities ter beschikking moet stellen. (Denk aan het volgnummer van programmakaarten, wanneer men bij programmawijziging de mogelijkheid wil hebben, kaarten tussen te voegen!) Erger is, dat men door de veldindeling van de kaart met allerlei bovengrenzen genoegen moet nemen; bovengrenzen die enerzijds de mogelijkheid van de sequentiële automaat miskennen (zie mijn college, sorry), anderzijds gauw hinderlijk zijn, omdat ze wegens de beperkte kaartomvang vrij - soms heel - scherp gesteld moeten worden.

Deze bedenkingen wegen bij mij heel zwaar: ik sluit niet uit, dat de scherpste kantjes er door herscholing vanaf te vijlen zijn; ik weet echter ook, dat ik me niet geroepen voel, om dit te ondernemen en bij gebrek aan nadere gegevens verlaat ik mij dus op observatie elders. Deze observatie omvat:

EWD139 - 2

- de waarneming, dat in alle "card oriented programming systems" bv. het manual, dat verteld, hoe een of andere "control card" geponst moet worden, altijd de noodzakelijke positie op de kaart expliciet vermeldt;
- dat - om der wille van uniforme verwerking in de ponskamer? - alle informatie, die op de kaarten geponst moet worden, positiebewust moet worden opgegeven, ook in die gevallen - programmateksten - waar dit niet de minste informatie draagt. (Zonneveld moet nu zijn programma's opschrijven op papier met voorbedrukte kolommen. Hij beklaagde zich bitter, hij had van zijn leven nog niet zoveel gestuft; en dat alles, omdat de ponskamer geen karakters "achter elkaar" kan ponsen! Ik zie geen reden om te veronderstellen, dat wij deze bezwaren zouden kunnen ondervangen en ril, in bescheidenheid bij voorbaat.) Samenvattend: sinds Alen Turing weten we, dat we ons moeten beperken tot aftelbaar veel mogelijke situaties, maar de ponskaart is een uitnodiging zich tot a priori enumereerbare gevallen te beperken.

Wat betreft de tyrannie hoef ik niet uitgebreid in te gaan op de bochten waarin men uitgenodigd kan worden zich te wringen. ("Als iemand nooit gehuwd geweest is, kan men het veld, normaliter gereserveerd voor kinderaantal (2 decimalen) gebruiken voor nadere polsspecificatie etc.") Ik vermeld slechts, dat deze tyrannie ernstige beperkingen oplegt aan een van de meest effectieve manieren om de leesbaarheid van programma's te verhogen: inspringen ter representatie van de structuur van het programma. (Dit is zo effectief, dat ik de Flexowriter boven de teleprinter verkies, misschien nog wel meer vanwege de mogelijkheid tot tabulatie dan vanwege het tweede alphabet. Het zou mij niet verbazen wanneer wij, zonder spijt van de Flexowriter te hebben, onze gebruikers zouden suggereren zich tot kleine letters te beperken!)

Mijn laatste bezwaar jegens de ponskaart is een van fundamentele onduidelijkheid in de manier, waarop ik geacht word aan de kaart zelf betekenis toe te kennen. Ik geef toe, dat deze overweging een variant lijkt op de vulgaire aanvat. "Als iemand een bak met kaarten laat vallen, etc" (Je kunt je mensen hoop ik leren, daar geen gewoonte van te maken. Bovendien: wie zo stom is om honderd meter paper tape in de knoop te laten raken is ook nog niet jarig!)

EWD139 - 3

Identificatie is, uiteindelijk, altijd een recursief proces. Als wij een willekeurige ponsband tegenkomen, weten we ook niet, wat deze betekent. We kunnen hem door de Flexowriter jagen, en als er dan een programma uitgetikt wordt weten we nog niet of dit soms niet een band met meetgegevens is. Er is maar een oplossing voor dit probleem: voorop elke band - in ponsing of met viltstift - in een metataal, verstaanbaar voor de mensen rond onze installatie, de band identificeren. En wanneer deze band dan via de prullebak en de straatgoot bij de BVD komt, dan blijkt die locaal voldoende, identificatie onvoldoende. Dus moeten we voorop elke band in een meta metataal "THE" zetten, etc. etc..

Een voor locaal gebruik, voldoende identificatie voorop een band fokken is moeilijk, maar, ik hoop, doenlijk. Op elke kaart is dit niet doenlijk. (Op het MC zijn wij genoodzaakt geweest in te stellen, dat iedereen die een onbeschreven band tegen kwam, deze vernietigde. Dit was heel heilzaam. De consequentie is, dat wij uit groepszelfdiscipline onze mensen instrueren elke niet-blanco ponskaart, die ze tegenkwamen, als betekenisloos weg te gooien. Dit is geen goedkoop grapje: het is een op de spits gedreven onlustgevoelen, waarvan ik met hoofd en hart weet, dat het meer dan gerechtvaardigd is.)

Hier wil ik het bij laten. Als mensen mij komen vertellen, dat zij met recht van ponskaarten gebruik gemaakt hebben, dan verandert die mededeling minder aan mijn opinie over ponskaarten, dan over de man.

Ik concludeer dan, dat deze man voldoende inventief of ordelijk geweest is om van een in wezen ongezond middel gezond gebruik te maken. Als pleidooi voor het medium - ik kan niet anders - is zo'n mededeling zonder enig effect.

Transcribed by Frank Steggink

Revised Tue, 6 Jan 2004.

---

## English translation

### An Appreciation of Punched Cards

EWD139 - 0

An Appreciation of Punched Cards

In what follows I hope to give a convincing argument for why I answer the question of whether the THE's EL X8 should be equipped with punched-card apparatus with a wholehearted "No". The outspokenness of my judgement is justified only if, in my argumentation, I approach the medium of the punched card in all fairness. What follows is a test of my capacity for decency in that direction.

I observe that the punched card came of age in the time before the general purpose computing machine. Around this medium there arose special purpose machines of limited programmability, such as reproducers, sorting machines, tabulators, collators, verifying punches, etc..

It is my conviction that much of the possible profit of this medium can be reaped only if one actually has at one's disposal a sufficient quantity of the aforementioned apparatus attuned to this medium: the advantage that punched cards are readily sortable is hard to realize without a sorting machine. (This does not alter the fact that the presence of a general purpose automaton does make a difference: the fact that a fast sorting machine could be connected to the X1, with the X1 controlling the sorting, has often led to sorting processes — which after all could now be carried out on far more complex criteria — taking place after an X1, this with a drastic reduction of the number of passes. Owing to the limited stopping facilities of the sorting machine this unfortunately did, for the computing automaton, mean the introduction of an essential rush situation.)

The style in which one works with punched cards has of course in the past been strongly influenced by the properties of the special punched-card machines. It is my conviction that this influence will remain, particularly there where the punched card is closely bound up with legible script. (It is clear that, since production by and for automata, entirely new modes of use have arisen, such as "a pack of binary cards". This strikes me precisely as the non-typical use of punched cards: that here a great deal of information is distributed over loose cards seems to me more annoying than pleasant, and I should think that this use would never have arisen had the punched card not already been there.)

This style of use manifests itself, e.g., in the customary checking techniques, briefly to be summarized as "twice makes it right" [tweemaal is scheepsrecht]. Punched-card readers and punched-card punches are provided with an extra checking station; what is further down the

EWD139 - 1

line, cards produced by hand are verified after verifying-punching. (The cards produced during verifying-punching are wont to be furnished with a special marking: experience has taught that they are suspect.) Besides parity checks — against incidental malfunctioning of the apparatus — one nowadays prefers, where possible — e.g. with program texts! — structural redundancy, whereby the check extends over the original manuscript as well. (The flawless punching, in whatever, of monotonous numerical material, of tables, remains a problem for which I know no palatable solution. The moral seems to me to be that this ought to be avoided.

Worse is what I should like to designate as the tradition of parallel processing and the tyranny of the 80 columns.

Parallel processing implies an a priori knowledge of the size of a quantity of information to be processed. By letting, e.g., integers occur as constituents in a text with a recursively defined syntax, no upper bound has been imposed on these integers. In the punched-card world, however, it is customary to fix the size of a unit of information — e.g. an integer — by ad-hoc conventions (e.g. the field of columns 7 through 12).

To begin with, we note that the sparing of the separators is no unmixed gain: for there stands over against this that one must always make the maximum number of positions available. (Think of the sequence number of program cards, when, upon program modification, one wishes to have the possibility of interleaving cards!) Worse is that through the field layout of the card one must content oneself with all manner of upper bounds; upper bounds that on the one hand misjudge the capability of the sequential automaton (see my lecture, sorry), and on the other hand soon become a nuisance, because, on account of the limited card size, they have to be set rather — sometimes very — sharply.

These objections weigh very heavily with me: I do not rule out that the sharpest edges can be filed off by re-education; I also know, however, that I do not feel called upon to undertake this, and so, for lack of further data, I rely on observation elsewhere. This observation comprises:

EWD139 - 2

- the observation that in all "card oriented programming systems", e.g., the manual that tells how some or other "control card" must be punched always explicitly states the necessary position on the card;
- that — for the sake of uniform processing in the punch room? — all information that must be punched onto the cards must be specified position-consciously, even in those cases — program texts — where this carries not the least information. (Zonneveld must now write down his programs on paper with pre-printed columns. He complained bitterly; never in his life had he fussed so much; and all that because the punch room cannot punch characters "one after another"! I see no reason to suppose that we could obviate these objections, and shudder, in modesty, in advance.) In summary: since Alan Turing we know that we must restrict ourselves to countably many possible situations, but the punched card is an invitation to restrict oneself to a priori enumerable cases.

As regards the tyranny, I need not enter at length into the contortions into which one can be invited to twist oneself. ("If someone has never been married, one can use the field, normally reserved for number of children (2 decimals), for further wrist specification, etc.") I mention only that this tyranny imposes serious restrictions on one of the most effective ways of increasing the readability of programs: indentation to represent the structure of the program. (This is so effective that I prefer the Flexowriter to the teleprinter, perhaps even more on account of the possibility of tabulation than on account of the second alphabet. It would not surprise me if we, without regretting the Flexowriter, were to suggest to our users that they restrict themselves to lower-case letters!)

My final objection against the punched card is one of fundamental unclarity in the manner in which I am supposed to attach meaning to the card itself. I admit that this consideration looks like a variant of the vulgar opening gambit. "If someone drops a tray of cards, etc." (One can, I hope, teach one's people not to make a habit of that. Besides: whoever is so stupid as to let a hundred metres of paper tape get into a tangle is also not having a happy day!)

EWD139 - 3

Identification is, ultimately, always a recursive process. If we come across an arbitrary punched tape, we likewise do not know what it means. We can run it through the Flexowriter, and if a program is then typed out we still do not know whether it is perhaps a tape with measurement data. There is but one solution to this problem: at the front of every tape — in punching or with felt-tip pen — to identify the tape in a metalanguage intelligible to the people around our installation. And when this tape then, by way of the wastepaper basket and the gutter, ends up with the BVD, then that turns out to be locally sufficient, identification insufficient. So we must put at the front of every tape, in a meta-metalanguage, "THE", etc. etc..

To breed an identification, sufficient for local use, at the front of a tape is difficult, but, I hope, feasible. On every card this is not feasible. (At the MC we were obliged to institute that anyone who came across an unwritten tape destroyed it. This was very salutary. The consequence is that, out of group self-discipline, we instruct our people to throw away as meaningless every non-blank punched card they came across. This is no cheap joke: it is a feeling of distaste driven to its extreme, of which I know with head and heart that it is more than justified.)

Here I will leave it at that. If people come to tell me that they have rightly made use of punched cards, then that communication alters my opinion about punched cards less than my opinion about the man.

I conclude then that this man has been sufficiently inventive or orderly to make healthy use of an essentially unhealthy device. As a plea for the medium — I cannot do otherwise — such a communication is without any effect.

Transcribed by Frank Steggink

Revised Tue, 6 Jan 2004.
