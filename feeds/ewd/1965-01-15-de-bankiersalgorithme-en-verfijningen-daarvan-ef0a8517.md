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
