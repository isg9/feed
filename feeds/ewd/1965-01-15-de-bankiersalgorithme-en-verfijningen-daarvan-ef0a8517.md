---
title: "De bankiersalgorithme en verfijningen daarvan (EWD116)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD116.html
published: "1965-01-15T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD116.PDF
---

# De bankiersalgorithme en verfijningen daarvan

EWD116 - 0

De bankiersalgorithme en verfijningen daarvan

0. Inleiding

In dit rapport wordt de interferentie besproken tussen simultane programma's, zoals deze voortvloeit uit het gemeenschappelijk gebruik van randapparatuur. Aan de restricties, die voortvloeien uit het feit, dat deze verschillende programma's in eenzelfde (eindig) geheugen moeten worden geherbergd, wordt (voorlopig?) geen aandacht geschonken. Onze primaire zorg ten aanzien van deze interferentie is de beheersing ervan, d.w.z. de vermijding van een dodelijke omarming. Onze eerste beschouwing is van toepassing, als het slechts gaat om een enkel soort communicatieapparaat.

1. De bankier met homogene klanten en één valuta

Een bankier (lees "installatie") beschikt over een eindig kapitaal in guldens (lees "bandlezers"). Hij is bereid geld aan klanten (lees "programma's") te lenen op de volgende voorwaarden:

- de klant verricht de leningen voor een in de tijd gemeten eindige transactie;
- de klant moet van tevoren de maximale omvang van de voor de transactie benodigde lening opgeven;
- de klant hoeft niet meteen dit totale bedrag op te nemen en tot het einde toe vast te houden; integendeel, het wordt aangemoedigd, als hij pas om meer geld komt vragen, als hij het nodig heeft en als hij zo gauw mogelijk weer komt afbetalen;
- de klant mag niet mopperen, wanneer hij om uitbreiding van zijn lening om meer geld komt en te horen krijgt: "Weliswaar overschrijdt U met deze aanvraag Uw opgegeven maximum niet, maar op dit moment kan ik U het gevraagde niet lenen. Ik zal U bericht sturen, wanneer ik U wel de door U gevraagde lening kan verstrekken";
- de garantie, dat dit moment mettertijd zal aanbreken ontleent de klant aan de wijsheid van de bankier en het feit, dat zijn medeklanten aan dezelfde eis gebonden zijn, nl., dat ze, zodra ze over het gevraagde bedrag de beschikking gekregen hebben, met bekwame spoed hun zaken voortzetten, d.w.z. dus in een eindige tijd om meer komen vragen, dan wel terugbetalen, dan wel hun transactie beëindigen;
- beëindiging van een transactie, van de bankier-klant contract impliceert dat het geleende bedrag geheel is terugbetaald. Rente wordt niet berekend.

De primaire vragen zijn

- onder welke voorwaarden kan de bankier met een zich aanmeldende klant een contract aangaan?
- onder welke voorwaarden kan de bankier bij aanvraag van (meer) geld tot uitbetaling overgaan zonder het gevaar te lopen, dat hij wegens dodelijke omarming niet zijn verplichtingen kan nakomen?

Ter verklaring van het verschijnsel van dodelijke omarming het volgende voorbeeld: Een bankier met een kapitaal van 8 gulden heeft twee klanten A en B, die beiden 6 gulden als hun maximale behoefte hebben opgegeven. Als A en B beiden twee gulden hebben opgegeven is er nog geen vuiltje aan de lucht. Wil nu bv. eerst A zijn lening tot 3 uitbreiden, dan mag dat. Komt dan echter B om zijn derde gulden, dan moet uitbetaling aangehouden worden, hoewel er nog 3 gulden in kas is. Zou immers aan B zijn derde gulden uitbetaald worden, dan zou er nog maar 2 gulden in kas overblijven. Als nu A voor zijn voortzetting nog 3 gulden vraagt (waar hij "mettertijd" recht op kan laten gelden) dan kan dit niet gegeven worden en A moet wachten; als echter ook B met zijn zaken in een stadium gekomen is, dat

EWD116 - 1

hij de resterende 3 gulden nodig heeft, voordat hij over terugbetaling kan gaan denken, dan zit de hele troep zo vast als een huis. Dit verschijnsel wordt "de dodelijke omarming" genoemd en het is deze calamiteit, die de bankier in zijn wijsheid dient te voorkomen. Deze alinea ter explicatie.

Vraag a. is gemakkelijk beantwoord: de bankier kan met elke zich aanbiedende klant het contract ondertekenen als de door de klant genoemde behoefte het totale kapitaal van de bankier niet overtreft.

Om vraag b. te beantwoorden voeren wij enige termen in:

De bankier beschikt over een vast "kapitaal". Elke klant biedt zich aan onder opgave van zijn maximale "behoefte".

Er geldt voor elke klant

"behoefte ≤ kapitaal".

De momentane toestand van een klant is gegeven door zijn "lening"; aanvankelijk is "lening ≡ 0", voor elke klant zal steeds gelden

"lening ≤ behoefte".

Afhankelijk van deze twee grootheden voeren wij per klant in de "claim", gegeven door

"claim = behoefte - lening".

De claim is dus het bedrag, dat de klant mogelijk voor de completering van zijn transacties er nog bij wil kunnen eisen.

Tenslotte voeren wij in de zg. "kasvoorraad" gegeven door

"kasvoorraad = kapitaal - som van de leningen".

Uiteraard geldt "0 ≤ kasvoorraad".

Om te kijken of de bankier bij aanvraag om meer tot uitbetaling kan overgaan, onderzoekt hij de toestand die ontstaan zou zijn als hij tot uitbetaling zou overgaan. Is deze toestand "veilig", dan gaat hij tot uitbetaling over; is deze toestand "onveilig", dan wordt de uitbetaling aangehouden en komt de klant op de wachtlijst. Rest ons te beschrijven, hoe hij de veiligheid van een situatie onderzoekt.

In wezen onderzoekt hij, of hij al zijn klanten nog kan bedienen en hij dus "mettertijd" zijn hele kapitaal kan vrijmaken. Hij voert daartoe in de groep van "bedienbare klanten" (aanvankelijk leeg) en de groep van "te dienen klanten" (aanvankelijk alle klanten omvattend). Voorts voert hij in de hulpgrootheid "vrij te maken bedrag", die aanvankelijk gelijk gemaakt wordt aan de kasvoorraad. Nu begint de algorithme:

Zolang de groep der "te dienen klanten" nog niet leeg is, wordt er hieronder gezocht naar een klant, wiens momentane claim de momentane waarde van het vrij te maken bedrag niet overschrijdt. Wordt zo'n te bedienen klant gevonden, dan wordt hij overgeheveld naar de groep "bedienbare klanten", waarbij tegelijkertijd het "vrij te maken bedrag" met zijn momentane lening vermeerderd wordt, waarna het proces herhaald zal worden. Dit proces kan op twee manieren eindigen:

- Doordat op deze wijze de groep "te dienen klanten" leeg wordt, in dit geval is de situatie "veilig";
- Doordat, voordat de groep "te bedienen klanten" leeg is, zich hierin geen klant bevindt met een claim, die de inmiddels bereikte waarde van het vrij te maken bedrag niet overschrijdt; in dit geval is de situatie "onveilig".

EWD116 - 2

Opm. Veiligheid kan reeds geconstateerd worden, zodra "vrij te maken bedrag = kapitaal" is; de resterende te bedienen klanten zijn dan a fertion bedienbaar.

Het is duidelijk, dat de bankier, door de situatie steeds veilig te houden, de dodelijke omarming steeds kan vermijden. Men kan ook laten zien, dat zodra de bankier zich tot een onveilige situatie verleiden laat, het behoefte patroon zich zo kan ontwikkelen, dat de dodelijke omarming niet meer is af te wenden.

2. Uitbreiding tot verscheidene in-converteerbare valuta

Het bovenbeschrevene is van toepassing wanneer de onderlinge interferentie tussen programma's zich beperkt tot "vechten om een soort apparaat", zeg "bandlezers", waarbij de ene bandlezer ons iedere keer even lief is als de andere. In de praktijk werden we geconfronteerd met het feit, dat de programma's bandlezers, bandponsers, kaartlezers en wat niet willen bespelen. Deze zijn inconverteerbaar in de zin, dat het bitter weinig helpt, als je voor een programma, dat eigenlijk een bandlezer nodig heeft, probeert te troosten met de mededeling "Pech, maar ik heb nog wel een ponser voor je".

In termen van de bankier komt dit er op neer dat kapitaal, lening, behoefte, claim en kasvoorraad nu niet enkel in guldens, maar in meer dimensies (franken, ponden en kronen) uitgedrukt worden. De passage in de algorithme "een klant wiens momentane clai de momentane waarde van het vrij te maken bedrag niet overschrijdt" dient vervangen te worden door "een klant, van wie geen claim de momentane waarde van de corresponderende kasvoorraad overschrijdt". Verder is de hele algorithme, inclusief het idee van het kasgeld onveranderd bruikbaar.

Transcribed by Frank Steggink

Revised Wed, 7 Jan 2004.

---

## English translation

### The Banker's Algorithm and Refinements Thereof

EWD116 - 0

The Banker's Algorithm and Refinements Thereof

0. Introduction

In this report the interference between simultaneous programs is discussed, as it arises from the shared use of peripheral equipment. To the restrictions that arise from the fact that these various programs must be housed in one and the same (finite) store, no attention is paid (for the time being?). Our primary concern with respect to this interference is the control of it, i.e. the avoidance of a deadly embrace. Our first consideration applies when it concerns only a single kind of communication device.

1. The banker with homogeneous customers and a single currency

A banker (read "installation") has at his disposal a finite capital in guilders (read "tape readers"). He is willing to lend money to customers (read "programs") under the following conditions:

- the customer makes the loans for a transaction that is finite as measured in time;
- the customer must state in advance the maximum size of the loan required for the transaction;
- the customer need not draw this total amount immediately and hold on to it until the end; on the contrary, it is encouraged that he comes to ask for more money only when he needs it and that he comes to pay it back again as soon as possible;
- the customer may not grumble when he comes to request an extension of his loan, for more money, and is told: "True, with this request you do not exceed your stated maximum, but at this moment I cannot lend you what you ask. I shall send you word when I am indeed able to grant you the loan you have requested";
- the guarantee that this moment will, in time, arrive the customer derives from the wisdom of the banker and from the fact that his fellow customers are bound by the same requirement, namely, that they, as soon as they have obtained the disposal of the requested amount, proceed with their affairs with all due dispatch, that is, therefore, within a finite time come to ask for more, or repay, or terminate their transaction;
- termination of a transaction, of the banker-customer contract, implies that the borrowed amount has been entirely repaid. Interest is not charged.

The primary questions are

- under what conditions can the banker enter into a contract with a customer who presents himself?
- under what conditions can the banker, upon a request for (more) money, proceed to disbursement without running the danger that, on account of a deadly embrace, he cannot meet his obligations?

In explanation of the phenomenon of the deadly embrace, the following example: A banker with a capital of 8 guilders has two customers A and B, who have both stated 6 guilders as their maximum need. If A and B have both stated two guilders, there is as yet not a cloud in the sky. Suppose now that, e.g., A first wishes to extend his loan to 3, then that is permitted. If B then, however, comes for his third guilder, then disbursement must be withheld, although there are still 3 guilders in the cash box. For if B's third guilder were paid out, then only 2 guilders would remain in the cash box. If A now asks for a further 3 guilders for his continuation (to which he can, "in time", assert a right), then this cannot be given and A must wait; if, however, B too has come to a stage in his affairs in which

EWD116 - 1

he needs the remaining 3 guilders before he can begin to think about repayment, then the whole crew is stuck as fast as a house. This phenomenon is called "the deadly embrace" and it is this calamity that the banker, in his wisdom, ought to prevent. This paragraph by way of explication.

Question a. is easily answered: the banker can sign the contract with every customer who presents himself if the need stated by the customer does not exceed the total capital of the banker.

To answer question b. we introduce a few terms:

The banker has at his disposal a fixed "capital". Each customer presents himself stating his maximum "need".

For each customer it holds that

"need ≤ capital".

The momentary state of a customer is given by his "loan"; initially "loan ≡ 0"; for each customer it will always hold that

"loan ≤ need".

Depending on these two quantities we introduce per customer the "claim", given by

"claim = need - loan".

The claim is thus the amount that the customer may possibly still wish to be able to demand in addition for the completion of his transactions.

Finally we introduce the so-called "cash stock" given by

"cash stock = capital - sum of the loans".

Self-evidently "0 ≤ cash stock" holds.

To see whether the banker can, upon a request for more, proceed to disbursement, he examines the state that would have arisen if he were to proceed to disbursement. If this state is "safe", then he proceeds to disbursement; if this state is "unsafe", then the disbursement is withheld and the customer goes onto the waiting list. It remains for us to describe how he examines the safety of a situation.

In essence he examines whether he can still serve all his customers and hence whether he can, "in time", free up his entire capital. To that end he introduces the group of "serviceable customers" (initially empty) and the group of "customers to be served" (initially comprising all customers). Furthermore he introduces the auxiliary quantity "amount to be freed up", which is initially made equal to the cash stock. Now the algorithm begins:

As long as the group of "customers to be served" is not yet empty, there is sought among these a customer whose momentary claim does not exceed the momentary value of the amount to be freed up. If such a customer to be served is found, then he is transferred to the group of "serviceable customers", whereby at the same time the "amount to be freed up" is increased by his momentary loan, after which the process will be repeated. This process can end in two ways:

- In that the group "customers to be served" becomes empty in this manner; in this case the situation is "safe";
- In that, before the group "customers to be served" is empty, there is no customer in it with a claim that does not exceed the value of the amount to be freed up reached by then; in this case the situation is "unsafe".

EWD116 - 2

Remark. Safety can be established as soon as "amount to be freed up = capital"; the remaining customers to be served are then a fortiori serviceable.

It is clear that the banker, by always keeping the situation safe, can always avoid the deadly embrace. One can also show that as soon as the banker lets himself be tempted into an unsafe situation, the pattern of needs can develop in such a way that the deadly embrace can no longer be averted.

2. Extension to several inconvertible currencies

What has been described above applies when the mutual interference between programs is limited to "fighting over one kind of device", say "tape readers", where the one tape reader is each time as dear to us as the other. In practice we were confronted with the fact that the programs want to operate tape readers, tape punches, card readers and what not. These are inconvertible in the sense that it helps bitterly little if, for a program that actually needs a tape reader, you try to console it with the announcement "Bad luck, but I do still have a punch for you".

In the banker's terms this comes down to the fact that capital, loan, need, claim and cash stock are now expressed not merely in guilders, but in more dimensions (francs, pounds and crowns). The passage in the algorithm "a customer whose momentary claim does not exceed the momentary value of the amount to be freed up" should be replaced by "a customer of whom no claim exceeds the momentary value of the corresponding cash stock". For the rest the whole algorithm, including the idea of the cash money, is usable unchanged.

Transcribed by Frank Steggink

Revised Wed, 7 Jan 2004.
