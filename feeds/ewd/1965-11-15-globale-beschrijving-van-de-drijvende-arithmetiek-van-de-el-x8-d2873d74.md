---
title: "Globale beschrijving van de drijvende arithmetiek van de EL X8 (EWD145)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD145.html
published: "1965-11-15T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD145.PDF
---

# Globale beschrijving van de drijvende arithmetiek van de EL X8

EWD 145

Globale beschrijving van de drijvende arithmetiek van de EL X8.

Met "globaal" wordt bedoeld, dat deze beschrijving de lezer in staat stelle voor zichzelf uit te maken, wat het antwoord van een zekere operatie is, als hij de operanden kent. Het is niet gezegd, dat deze beschrijving ook de beschrijving is van de manier, waarop de arithmetische circuits van de EL X8 dit antwoord construeren.

Voorstelbare getallen.

De drijvende arithmetiek opereert op getallen van de vorm

m × 2↑e

waarbij m (van mantisse) voldoet aan 0 ≤ abs (m) ≤ 2↑40 − 1

en e (van exponent) voldoet aan 0 ≤ abs (e) ≤ 2↑11 − 1 = 2047.

Opm. 1. Voor e zijn 12 bits ter beschikking, voor m 41; in beide gevallen kan de waarde 0 op twee wijzen worden voorgesteld, nl. +0 en −0.

Opm. 2. Niet alleen e maar ook m neemt dus louter gehele waarden aan.

Afronding.

Optelling, aftrekking, vermenigvuldiging en deling kunnen alle een resultaat afleveren, dat, ongeacht de bovengrens van abs(e) niet exact voorstelbaar is, omdat de afstand van meest en minst significante bit van abs(m) meer dan 39 zou moeten zijn.

(Bij optelling en aftrekking kan dit ontstaan doordat de mantissen onder elkaar geschoven moeten worden, bij de vermenigvuldiging is de afstand van meest en minst significante bit van het execte product in het algemeen 79 of 78, bij de doorgaans niet opgaande deling is deze afstand oneindig).

In al deze gevallen wordt exact afgerond, en in het twijfelgeval van de nul af.

Het twijfelgeval treedt op, wanneer de afstand van meest en minst significante bit van het exacte antwoord 40 is; m.a.w. als de exacte m = 2↑40 + 1 wordt dit tot 2↑40 + 2 afgerond, zodat halvering tot 2↑39 + 1 mogelijk is.

Aldus is de eventueel afgeronde m en exacte e (waarin netjes van alle halveringen is boekgehouden) bepaald. Van deze exacte e nemen we voorshands niet aan, dat hij aan abs(e) ≤ 2047 voldoet.

Opm. 3. Door de conventie, dat het twijfelgeval van de nul af wordt afgerond is gegarandeerd, dat (−a)× b = − (a × b) is.

Vervolgens worden exacte e en (eventueel afgeronde) m aan de normering onderworpen.

De normering probeert de absolute waarde van e zo klein mogelijk te krijgen als zonder cijferverlies mogelijk is. (Elke verhoging van e moet gecompenseerd worden door halvering van m,

wat alleen zonder cijferverlies mogelijk is, als m eerst even is; elke verlaging van e moet gecompenseerd worden door verdubbeling van m, wat zonder "cijferverlies aan kop" alleen mogelijk is als abs(m) < 2↑39).

Opm. 4. Dit proces kan eindigen doordat abs(e) = 0 wordt; in dat geval wordt e = +0 afgeleverd.

Als m = ± 0, kan onbeperkt zonder cijferverlies geschoven worden: in de hardware zit een m = ± 0 detectie, die dit geval kortsluit en onmiddelijk e = + 0 maakt.

Nadat aldus gepoogd is abs(e) zo klein mogelijk te maken, wordt gekeken, of abs(e) klein genoeg is.

Als e ≥ 2048 is, dan wordt als antwoord afgeleverd e = 2047 en m = ± (2↑40 − 1), en wel met het teken van de oorspronkelijk m.

Als e ≤ − 2048 wordt (bij herhaling)
1) e met 1 opgehoogd
2) m afkappend gehalveerd, als abs(m) > 1 is;

als abs(m) = 1 is blijft m ongewijzigd.

Met andere woorden: in geval van overflow wordt het grootste getal met het goede teken afgeleverd, in het geval van underflow wordt door uitschuiven gepoogd de exponent alsnog binnen de perken te brengen. Het minste, wat overblijft is daarbij &plusmn; 2↑(−2047).

Underflow kan dus niet m = 0 genereren.

Opm. 5. Aftrekking van een getal "m × 2↑e" wordt gerealiseerd door "−m × 2↑e" op te tellen.

Opm. 6. Overflow kan optreden bij +, −, × en ∕ .
Underflow kan niet bij + en × optreden.

Opm. 7. Additieve bewerkingen leveren alleen nul op als het antwoord exact = 0 is; dit antwoord is alleen m = +0 wanneer de operanden van de effectieve optelling (zie opm. 5) beide = +0 zijn.

Opm. 8. De vermenigvuldiging levert alleen exact nul als antwoord, wanneer ten minste een factor = 0 is. Het teken van de productnul volgt de normale tekenregel voor producten.

Opm. 9. Voor de deling geldt:
deeltal ≠ 0, deler ≠ 0→ quotiÎnt ≠ 0
deeltal = ± 0, deler is ≠ 0 → quotiÎnt = ± 0 (met in achtname van de normale tekenregels).

De situatie voor deler = ± 0 is momenteel het voorwerp van ernstige discussie.

Verwacht wordt:
deeltal ≠ 0, deler ± 0 → ± (2↑40 −1) × 2↑2047 met in achtname van de normale tekenregel.
deeltal = ± deler = ± 0 → een welgedefinieerd antwoord met in achtname van de tekenregel.

Eindhoven, 6 december 1965.

transcribed by Sam Samshuijzen

revised Wed, 19 Jul 2006

---

## English translation

### Global description of the floating-point arithmetic of the EL X8

EWD 145

Global description of the floating-point arithmetic of the EL X8.

By "global" it is meant that this description shall enable the reader to determine for himself what the result of a certain operation is, if he knows the operands. It is not asserted that this description is also the description of the manner in which the arithmetic circuits of the EL X8 construct this result.

Representable numbers.

The floating-point arithmetic operates on numbers of the form

m × 2↑e

where m (for mantissa) satisfies 0 ≤ abs (m) ≤ 2↑40 − 1

and e (for exponent) satisfies 0 ≤ abs (e) ≤ 2↑11 − 1 = 2047.

Remark 1. For e, 12 bits are available, for m, 41; in both cases the value 0 can be represented in two ways, namely +0 and −0.

Remark 2. So not only e but also m takes on purely integer values.

Rounding.

Addition, subtraction, multiplication and division can all deliver a result that, regardless of the upper bound of abs(e), is not exactly representable, because the distance between the most and least significant bit of abs(m) would have to be more than 39.

(With addition and subtraction this can arise because the mantissas must be shifted relative to one another; with multiplication the distance between the most and least significant bit of the exact product is in general 79 or 78; with division, which usually does not come out even, this distance is infinite.)

In all these cases exact rounding is applied, and in the doubtful case, away from zero.

The doubtful case occurs when the distance between the most and least significant bit of the exact result is 40; in other words, if the exact m = 2↑40 + 1, this is rounded to 2↑40 + 2, so that halving to 2↑39 + 1 is possible.

In this way the possibly rounded m and the exact e (in which all halvings have been neatly accounted for) are determined. Of this exact e we do not, for the time being, assume that it satisfies abs(e) ≤ 2047.

Remark 3. By the convention that the doubtful case is rounded away from zero, it is guaranteed that (−a)× b = − (a × b).

Subsequently the exact e and the (possibly rounded) m are subjected to normalization.

Normalization tries to make the absolute value of e as small as is possible without loss of digits. (Every increase of e must be compensated by a halving of m,

which is only possible without loss of digits if m is first even; every decrease of e must be compensated by a doubling of m, which is only possible without "loss of digits at the head" if abs(m) < 2↑39.)

Remark 4. This process can terminate because abs(e) = 0; in that case e = +0 is delivered.

If m = ± 0, shifting can be done without limit and without loss of digits: in the hardware there is an m = ± 0 detection, which short-circuits this case and immediately sets e = + 0.

After it has thus been attempted to make abs(e) as small as possible, it is checked whether abs(e) is small enough.

If e ≥ 2048, then the answer delivered is e = 2047 and m = ± (2↑40 − 1), and indeed with the sign of the original m.

If e ≤ − 2048, then (repeatedly)
1) e is increased by 1
2) m is halved with truncation, if abs(m) > 1;

if abs(m) = 1, m remains unchanged.

In other words: in case of overflow the largest number with the correct sign is delivered; in case of underflow an attempt is made, by shifting out, to bring the exponent within bounds after all. The least that thereby remains is &plusmn; 2↑(−2047).

Underflow therefore cannot generate m = 0.

Remark 5. Subtraction of a number "m × 2↑e" is realized by adding "−m × 2↑e".

Remark 6. Overflow can occur with +, −, × and ∕ .
Underflow cannot occur with + and ×.

Remark 7. Additive operations yield zero only if the answer is exactly = 0; this answer is m = +0 only when the operands of the effective addition (see remark 5) are both = +0.

Remark 8. Multiplication yields exactly zero as answer only when at least one factor = 0. The sign of the product zero follows the normal sign rule for products.

Remark 9. For division the following holds:
dividend ≠ 0, divisor ≠ 0→ quotient ≠ 0
dividend = ± 0, divisor is ≠ 0 → quotient = ± 0 (with observance of the normal sign rules).

The situation for divisor = ± 0 is currently the subject of serious discussion.

It is expected:
dividend ≠ 0, divisor ± 0 → ± (2↑40 −1) × 2↑2047 with observance of the normal sign rule.
dividend = ± divisor = ± 0 → a well-defined answer with observance of the sign rule.

Eindhoven, 6 December 1965.

transcribed by Sam Samshuijzen

revised Wed, 19 Jul 2006
