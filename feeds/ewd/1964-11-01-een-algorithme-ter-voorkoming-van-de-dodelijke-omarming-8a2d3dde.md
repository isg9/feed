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

L:  if claim[i] < vrij bedrag then

begin vrij bedrag:= vrij bedrag + lening[i];

if vrij bedrag < A then begin i:= i + 1; goto L end

else veilig:= true

end

else

veilig:= false"

De bankier controleert dus of hij "door verstandig te wachten al zijn geld weer terugziet."

Transcribed by Frank Steggink

Revised Tue, 6 Jan 2004.


---

## English translation

### An algorithm for the prevention of the deadly embrace

EWD108 - 1

An algorithm for the prevention of the deadly embrace.

A banker is willing to lend money to clients -in whole guilders- on the following -benevolent- conditions.

1) The client must be able to guarantee that, if he gains the disposal of the requested money, the project for which the loan is taken out will be completed within finite time, after which the client will return all the borrowed money to the banker.

2) The client must specify in advance an upper bound on what he will ever need to be able to have on loan simultaneously for the benefit of this project.

3) If the client -in observance of 2)- comes to ask for a supplementary loan, he may not complain when he is told by the banker "At the moment I cannot place the requested amount at your disposal, but I guarantee you that this will indeed be the case within finite time."

4) It is permitted -indeed it is even encouraged!- that the client, when he sees during the transaction that the initial (see 2.) respectively the last-mentioned upper bound will no longer be reached, notifies the banker of the reduction of the upper bound.

The questions are: (assuming the banker operates with a capital of A guilders)

a) under what conditions can he accept a client who presents himself?

b) what is the algorithm for deciding to pay out or, respectively, not to pay out, in such a way that the banker on the one hand will never postpone a payment that he could safely have made, and on the other hand never makes a payment when he can no longer guarantee under all circumstances that he will be able to meet his future obligations.

The answer to the first question is that the banker can accept every client whose (initial) upper bound does not exceed his total capital A.

In order to decide whether he can proceed to payment upon a request, he surveys the situation that would arise if he were to proceed to payment. If this situation is "safe", then he pays out; if it is not safe, then he refuses the payment. To describe the algorithm that establishes whether a situation is safe, we introduce the following terms.

For each client the banker keeps track of two amounts:

1) loan (initially = 0)

2) claim (initially = initial upper bound)

At every moment, for each client, "loan + claim = upper bound" holds.

The initial capital A diminished by the sum of the loans we call the "cash reserve".

In order to examine a situation, the banker arranges his clients in order of non-decreasing claim, thus claim[i] < claim[i + 1]. Subsequently he executes:

"vrij bedrag:= kasvoorraad; i:= 1;

L:  if claim[i] < vrij bedrag then

begin vrij bedrag:= vrij bedrag + lening[i];

if vrij bedrag < A then begin i:= i + 1; goto L end

else veilig:= true

end

else

veilig:= false"

The banker thus checks whether he "sees all his money again by waiting prudently."

Transcribed by Frank Steggink

Revised Tue, 6 Jan 2004.
