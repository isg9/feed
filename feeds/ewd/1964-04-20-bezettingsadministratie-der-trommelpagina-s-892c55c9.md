---
title: "Bezettingsadministratie der trommelpagina’s (EWD86)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD86.html
published: "1964-04-20T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD86.PDF
---

# Bezettingsadministratie der trommelpagina’s

20 mei 1964

Bezettingsadministratie der trommelpagina's.

Aan elke trommelpagina kennen we twee constanten toe, nl. a en s: dit zijn één-éénduidige functies. De constante a "nummert" de trommelpagina's in volgorde van opklimmend beginadres, de constante s "nummert" de trommelpagina's in de sequens, waarin de beginwoorden van de pagina's de koppen passeren.

Voorlopig stellen we ons op het standpunt, dat de hele trommel voor pagina's ter beschikking staat. (Dit zal niet het geval zijn, maar zulks kunnen we opvangen door een aantal trommelpagina's permanent bezet te verklaren, waarna we ze rustig voor andere doeleinden kunnen gebruiken.)

Als functie van a is het beginadres gegeven door

beginadres(a) = 513 * a ;

tussen opeenvolgende trommelpagina's zit dus een ongebruikt woord. Verder geldt

0 ≤ a ≤ 1021,

voor a = 1022 en a = 1023 is geen ruimte meer op de trommel. De laatste drie woorden op de trommel worden eveneens niet gebruikt.

Nu merken we op, dat 513 * 513 = 1 mod(1024); alle adressen = mod(1024) zijn tegelijkertijd accessibel. Hieruit volgt, dat verhoging van s met 1 neer komt op verhoging van a met 513, gereduceerd mod(1024).

Voegen we aan de pagina met a = 0 code s = 0 toe, dan vinden (we)

a(s) = rem(513 * s, 1024).

Omgekeerd vinden we, dat verhoging van a met 1 de s met 513 verhoogt (modulo 1024)
dus geldt

s(a) = rem(513 * a, 1024).

Ook s voldoet primair aan de ongelijkheid 0 ≤ s ≤ 1023; we moeten ons even afvragen welke s-waarden niet voorkomen; dit zijn

s(1022) = 1022           en

s(1023) = 511   .

Deze nonexistente bladzijden laten we vrolijk meehobbelen, we zullen ze (zijnde nondisponibel) als permanent bezet noteren.

Voorts is s als functie van het beginadres ten duidelijkste gegeven door

s(beginadres) = rem(beginadres, 1024);

alle tot dusverre bekeken functies zijn dus door schoon schuiven over 9 plaatsen, een enkele additie en een collatie met 1023 onmiddellijk uit te rekenen.

Aan deze pagina's gaan we bits toekennen uit een boolean array TBB[0:39, 0:25]; (de afkorting TBB komt van Trommel Bezettings Bits). De bits van dit array gaan we afbeelden op de bits d[25] t/m d[0] van de woorden ven het woordarray TBW[0:39], 40 woorden, in het geheugen te bergen op 40 consecutieve geheugenplaatsen in volgorde van oplopend subscript. Van deze 40 woorden is het tekenbit d[26] permanent gelijk aan 1.

Voor de bits van TBB geldt. dat

a) TBB[i,j] afgebeeld wordt op bit d(25 - j) uit woord TBW[i]

b) TBB[i,j] toegevoegd is aan de trommelpagina met s = 26 * i + j

c) TBB[i,j] = 1 betekent, dat de pagina bezet is
= 0 betekent, dat de pagina vrij is.

De bits TBB[19,17] (dwz. s = 511), TBB[39,8] (dwz. s: 1022) en TBB[39, 10], TBB[39, 11]...TBB[39, 25] (dwz. s = 1024, 1025, ..., 1039) zijn wegens nonexistentie permanent = 1, evenals de TBB's, die beantwoorden aan stukken van de trommel, die voor andere doeleinden bestemd zijn.

(In het volgende is de constante B25 = 2↑25 = 33554432.)

We zullen de TBB's op drie manieren gebruiken.

a) Om een bezette trommelpagina met s = 26 * i + j vrij te geven.

In een register plaatsen we B25 * 2↑(-j) en met een subtractieve uitopdracht voeren we uit

TBW[i]:= TBW[i] - B25 * 2↑(-j).

b) Om een vrije trommelpagina met s = 26 * i + j te bezetten; dit gaat analoog met een additieve uitopdracht:

TBW[i]:= TBW[i] + B25 * 2↑(-j).

De weg van beginadres naar s gaat met een simpele collatie, om uit s de i en de j te isoleren, kost ons wel een deling. De tekencijfers van de woorden TBW zijn met zorg =1 gekozen, opdat de preferentie voor de min nul ons geen parten speelt. Als wij deze vrijgave en bezetting additief spelen, den moeten we er wel zeker van zijn, dat de pagina van te voren als bezet, resp. vrij genoteerd stond: anders zou overloop de boel immers hopeloos bederven!

c) Om een geschikte vrije trommelpagina te zoeken. Als we nl. dumpen, dan zijn we vrij in de keuze van de trommelpagina. Als zich in het startmagazijn van de trommeltransporten nog een startopdracht bevindt, dan kunnen we onderzoeken, bij welke waarde waarde van s deze is afgelopen. Vermeerderen we (gereduceerd modulo 1024) deze s met de "safety marge", die Electrologica ons op moet geven, opdat de volgende startopdracht nog zonder verlies van een omwenteling gehaald wordt, dan hebben we de ideale s, ware het niet, dat we nog moeten onderzoeken, of de bijbehorende trommelpagina wel vrij is. (Als er in het startmagazijn van de trommel nog maar 1 startopdracht staat, kunnen we in verband met de minimum tijdsduur van het nu volgende onderzoekprogramma uit voorzichtigheid s nog wel met één of andere constante verhogen, extra wel te verstaan. Als het startmagzijn leeg is, is elke s even goed, want dan hebben we stand van de trommel volledig uit het oog verloren.) Het array TBB gaan we nu gebruiken om de eerste vrije trommelpagina (met "s ≥ s ideaal", alles modulo 1024) te zoeken. Daartoe splitsen we s ideaal in

s ideaal = 26 * i + j

en voeren met deze waarden van i en j als beginwaarden het volgende programma uit.

A:= TBW[i];
schuif A schoon over j plaatsen naar links;
if A ≠ 0 then
begin normeer A (het aantal slagen komt in B);
j:= j + B
end
else
begin i0:= i;
volgend woord: modhoog(i, 40);
A:= TBW[i];
if A = 0 then begin if i = 10 then goto trommel vol else goto volgend woord
end;
normeer A (het aantal slagen komt in B);
j := B
end

De gekozen pagina is nu gegeven door de eindwaarden van i en j (door s = 26 * i + j), waarmee het gestelde doel bereikt is.

transcribed by Gerrit Jan Veltink
revised Sun, 28 Mar 2010

---

## English translation

### Occupancy administration of the drum pages

20 May 1964

Occupancy administration of the drum pages.

To each drum page we assign two constants, namely a and s: these are one-to-one functions. The constant a "numbers" the drum pages in order of ascending start address, the constant s "numbers" the drum pages in the sequence in which the start words of the pages pass the heads.

For the time being we take the position that the entire drum is available for pages. (This will not be the case, but we can accommodate that by declaring a number of drum pages permanently occupied, after which we can calmly use them for other purposes.)

As a function of a, the start address is given by

beginadres(a) = 513 * a ;

between successive drum pages there is thus an unused word. Furthermore

0 ≤ a ≤ 1021,

for a = 1022 and a = 1023 there is no more room on the drum. The last three words on the drum are likewise not used.

Now we note that 513 * 513 = 1 mod(1024); all addresses = mod(1024) are accessible at the same time. From this it follows that increasing s by 1 amounts to increasing a by 513, reduced mod(1024).

If we assign to the page with a = 0 the code s = 0, then we find

a(s) = rem(513 * s, 1024).

Conversely we find that increasing a by 1 increases s by 513 (modulo 1024),
so

s(a) = rem(513 * a, 1024).

Here too s primarily satisfies the inequality 0 ≤ s ≤ 1023; we must briefly ask ourselves which s-values do not occur; these are

s(1022) = 1022           and

s(1023) = 511   .

These nonexistent pages we cheerfully let bounce along; we shall note them (being nondisposable) as permanently occupied.

Furthermore, s as a function of the start address is most clearly given by

s(beginadres) = rem(beginadres, 1024);

all functions considered so far can therefore be computed immediately by shifting clean over 9 places, a single addition and a collation with 1023.

To these pages we shall assign bits from a boolean array TBB[0:39, 0:25]; (the abbreviation TBB comes from Trommel Bezettings Bits [Drum Occupancy Bits]). The bits of this array we shall map onto the bits d[25] up to and including d[0] of the words of the word array TBW[0:39], 40 words, to be stored in memory in 40 consecutive memory locations in order of increasing subscript. Of these 40 words the sign bit d[26] is permanently equal to 1.

For the bits of TBB it holds that

a) TBB[i,j] is mapped onto bit d(25 - j) of word TBW[i]

b) TBB[i,j] is assigned to the drum page with s = 26 * i + j

c) TBB[i,j] = 1 means that the page is occupied
= 0 means that the page is free.

The bits TBB[19,17] (i.e. s = 511), TBB[39,8] (i.e. s: 1022) and TBB[39, 10], TBB[39, 11]...TBB[39, 25] (i.e. s = 1024, 1025, ..., 1039) are, on account of nonexistence, permanently = 1, as are the TBB's that correspond to portions of the drum which are destined for other purposes.

(In the following, the constant B25 = 2↑25 = 33554432.)

We shall use the TBB's in three ways.

a) To release an occupied drum page with s = 26 * i + j.

In a register we place B25 * 2↑(-j) and with a subtractive output instruction we execute

TBW[i]:= TBW[i] - B25 * 2↑(-j).

b) To occupy a free drum page with s = 26 * i + j; this proceeds analogously with an additive output instruction:

TBW[i]:= TBW[i] + B25 * 2↑(-j).

The path from start address to s goes by a simple collation; to isolate the i and the j from s does cost us a division. The sign digits of the words TBW have been carefully chosen = 1, so that the preference for the minus zero does not play tricks on us. If we carry out this release and occupation additively, then we must indeed be certain that the page was beforehand noted as occupied, respectively free: otherwise overflow would after all spoil the whole thing hopelessly!

c) To search for a suitable free drum page. For when we dump, we are free in the choice of the drum page. If there is still a start instruction present in the start magazine of the drum transports, then we can investigate at which value value of s it has run out. If we increase (reduced modulo 1024) this s by the "safety margin" that Electrologica must supply us, so that the next start instruction is still fetched without loss of a revolution, then we have the ideal s, were it not that we still have to investigate whether the corresponding drum page is indeed free. (If there is only 1 start instruction left in the start magazine of the drum, then in connection with the minimum duration of the search program that now follows we may, out of caution, still increase s by some constant or other, extra that is to say. If the start magazine is empty, then every s is equally good, for then we have completely lost sight of the position of the drum.) We now use the array TBB to search for the first free drum page (with "s ≥ s ideal", everything modulo 1024). To this end we split s ideal into

s ideaal = 26 * i + j

and, with these values of i and j as initial values, execute the following program.

A:= TBW[i];
shift A clean over j places to the left;
if A ≠ 0 then
begin normalize A (the number of strokes ends up in B);
j:= j + B
end
else
begin i0:= i;
next word: modhoog(i, 40);
A:= TBW[i];
if A = 0 then begin if i = 10 then goto drum full else goto next word
end;
normalize A (the number of strokes ends up in B);
j := B
end

The chosen page is now given by the final values of i and j (by s = 26 * i + j), whereby the stated goal is achieved.

transcribed by Gerrit Jan Veltink
revised Sun, 28 Mar 2010
