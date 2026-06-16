---
title: "Een algorithme ter voorkoming van de dodelijke omarming (EWD108)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD108.html
published: "1964-11-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD108.PDF
---

# Een algorithme ter voorkoming van de dodelijke omarming

EWD108 - 1

Een algorithme ter voorkoming van de dodelijke omarming.

Een bankier is op de volgende -menslievende- voorwaarden bereid om aan klanten -in hele guldens- geld te lenen.

1) De klant moet kunnen garanderen, dat als hij over het gevraagde geld de beschikking krijgt, het project, waarvoor de lening wordt aangegaan, binnen eindige tijd voltooid zal worden, waarna de klant al het geleende geld aan de bankier zal teruggeven.

2) De klant moet van te voren een bovengrens opgeven van wat hij ooit tegelijkertijd ten bate van dit project in leen zal moeten kunnen hebben.

3) Als de klant -met eerbiediging van 2)- om een aanvullende lening komt vragen, mag hij niet klagen, wanneer hij van de bankier te horen krijgt "Op het moment kan ik U het gevraagde bedrag niet ter beschikking stellen, maar ik garandeer U, dat dit binnen eindige tijd wel het geval zal zijn."

4) Het is toegestaan -zulks wordt zelfs aangemoedigd!- dat de klant, wanneer hij tijdens de transactie ziet, dat de initiele (zie 2.) resp laatst genoemde bovengrens niet meer gehaald zal worden, de bankier van de verlaging van de bovengrens in kennis stelt.

De vragen zijn: (als de bankier met een kapitaal van A gulden werkt)

a) onder welke voorwaarden kan hij een zich aanbiedende klant aannemen?

b) wat is de algorithme om tot wel, resp. niet uitbetaling te beslissen, zo, dat de bankier enerzijds nimmer een uitbetaling zal uitstellen, die hij veilig had kunnen doen, anderzijds nooit een betaling doet, wanneer hij niet meer onder alle omstandigheden garanderen kan, dat hij zijn toekomstige verplichtingen zal kunnen nakomen.

Het antwoord op de eerste vraag is, dat de bankier elke klant kan accepteren, welks (initiele) bovengrens zijn totale kapitaal A niet overschrijdt.

Om te beslissen, of hij bij aanvraag tot betaling kan overgaan, overziet hij de situatie, die zou ontstaan, wanneer hij tot betaling zou overgan. Is deze situatie "veilig", dan betaalt hij uit, is deze niet veilig, dan weigert hij de betaling. Om de algorithme te beschrijven, die vaststelt of een situatie veilig is, voeren we de volgende termen in.

Voor elke klant houdt de bankier twee bedragen bij:

1) lening (aanvankelijk = 0)

2) claim (aanvankelijk = initiele bovengrens)

Op elk moment geldt voor elke klant "lening + claim = bovengrens".

Het aanvangskapitaal A verminderd met de som der leningen noemen we de "kasvoorraad".

Om een situatie te onderzoeken, rangschikt de bankier zijn klanten in volgorde van niet afnemende claim, dus claim[i] < claim[i + 1]. Vervolgens voert hij uit:

"vrij bedrag:= kasvoorraad; i:= 1;

L:  if claim[i] < vrij bedrag then

begin vrij bedrag:= vrij bedrag + lening[i];

if vrij bedrag < A then begin i:= i + 1; goto L end

else veilig:= true

end

else

veilig:= false"

De bankier controleert dus of hij "door verstandig te wachten al zijn geld weer terugziet."

Transcribed by Frank Steggink

Revised Tue, 6 Jan 2004.
