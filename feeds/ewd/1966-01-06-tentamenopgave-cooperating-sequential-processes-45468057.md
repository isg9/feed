---
title: "Tentamenopgave “Cooperating Sequential Processes” (EWD150)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD150.html
published: "1966-01-06T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD150.PDF
---

# Tentamenopgave “Cooperating Sequential Processes”

EWD 150

Tentamenopgave "Cooperating Sequential Processes."

De klanten van een kapper zijn genummerd van 1 t/m N; de integer "N" (evenals de later te noemen integer "M") zij gegeven in het universum, waarin dit programma begrepen moet worden.

De structuur van de samenleving is als volgt:

begin integer array klantsem, rangnummer [1 : N];
integer kappersem, klantnummer, lopend zwart, lopend blond,
aantal zwarte wachters, aantal blonde wachters;
⋮
⋮ (1)
⋮
parbegin
kapper: begin
wie volgt:
⋮
⋮ (2)
⋮
⋮
P(kappersem);
kapper knipt haren van de klant aangewezen door de waarde
van de variabele genaamd klantnummer;
⋮
⋮ (3)
⋮
⋮
goto wie volgt
end;
klant 1: begin ....................end;
⋮
⋮
⋮
klant i: begin
naar de kapper:
⋮ (4)
⋮
P(klantsem[i]);
leven van alledag en laat de haren groeien;
goto naar de kapper
end;
⋮
⋮
⋮
klant N: begin....................end
parend
end

In (1) moeten de verder benodigde seinpalen en variabelen gedeclareerd worden. Vervolgens moeten de nodige initialisaties worden verricht.

In (2) moet de kapper uitmaken, of hij een volgende klant gaat knippen; zo nee, dan kan hij in "P(kappersem)" opgehouden worden. Gedurende het geknipt worden wordt van de klant geen activiteit gevergd.

In (3) moet de kapper het voltooien van het haarknippen aan de klant in kwestie melden, opdat deze het leven van alledag kan voortzetten.

In (4) meldt de klant zich bij de kapper, hij mag "P(klantsem[i])" pas voltooien, nadat de kapper hem geknipt heeft.

De klanten met nummer van 1 t/m M (< N) hebben zwart haar, zij met nummer van M + 1 t/m N hebben blond haar.

De kapper heeft een grote weerzin regen het overschakelen van het knippen van blonde klanten naar het knippen van zwarte klanten. Hij heeft zich daarom de volgende gedragslijn aangemeten:

Zolang hij ingesteld is op blonde klanten, laat hij de zwarte rustig wachten; hij stelt zich in op het knippen van zwarte klanten zodra

1) er geen blonde klanten meer wachten en bovendien

2) er ten minste 3 zwarte klanten wachten.

Zolang hij is ingesteld op zwarte klanten, laat hij de blonde rustig wachten; hij stelt zich pas in op het knippen van blonde klanten zodra

1) er geen zwarte klanten meer wachten en bovendien

2) er ten minste 1 blonde klant wacht.

De tentaminandi wordt verzocht de in het programma ontbrekende stukken (1), (2), (3) en (4) in te vullen; voor sectie (4) moet een blonde en een zwarte versie gemaakt worden.

Klanten van dezelfde haarkleur worden geholpen in volgorde van binnenkomst, het programma beginnen met de kapper ingesteld op blonde klanten.

De variabelen, die reeds in de declaratie aan het begin van het programma zijn opgenomen, zijn daar ingevoerd om het nakijken van de programma's te vergemakkelijken en kunnen verder als hint voor de oplossing beschouwd worden.

De variabelen "rangnummer[i]" zijn bedoeld om aan te geven, of de i-de klant geknipt moet worden; zo nee, dan zij "rangnummer[i] = 0", zo ja, dan zij "rangnummer[i]" gelijk aan het nummer van binnenkomst van klanten met dezelfde haarkleur. Binnenkomende zwartharige respectievelijk blonde klanten worden haarkleursgewijs geteld door de grootheden "lopend zwart" respectievelijk "lopend blond", beide tellingen te beginnen met 1.

De variabelen "aantal zwarte wachters" zij gelijk aan het aantal zwartharige klanten met "rangnummer[i] > 0".

Om uit een haargroep de langst wachtende klant te bepalen, kiest men uit die groep de klant met minimaal positief rangnummer; de tentaminandus hoeft geen extra moeite te doen om dit zoekproces te versnellen, we mogen bij wijze van spreken aannemen, dat dit zoekproces in het niet valt bij het knippen zelf.

transcribed by Sam Samshuijzen

revised Wed, 19 Jul 2006
