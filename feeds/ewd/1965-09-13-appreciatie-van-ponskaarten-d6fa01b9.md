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
