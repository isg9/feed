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


---

## English translation

### Examination Problem “Cooperating Sequential Processes”

EWD 150

Examination problem "Cooperating Sequential Processes."

The customers of a barber are numbered from 1 to N inclusive; the integer "N" (as well as the integer "M" to be mentioned later) is to be given in the universe in which this program is to be understood.

The structure of the society is as follows:

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

In (1) the further required semaphores and variables must be declared. Subsequently the necessary initializations must be carried out.

In (2) the barber must decide whether he is going to cut a next customer; if not, then he may be held up in "P(kappersem)". During the cutting no activity is demanded of the customer.

In (3) the barber must report the completion of the haircut to the customer in question, so that the latter can resume everyday life.

In (4) the customer reports to the barber; he may only complete "P(klantsem[i])" after the barber has cut him.

The customers with number from 1 to M (< N) inclusive have black hair, those with number from M + 1 to N inclusive have blond hair.

The barber has a great aversion to switching over from cutting blond customers to cutting black-haired customers. He has therefore adopted the following line of conduct:

As long as he is set on blond customers, he calmly lets the black-haired ones wait; he sets himself to cutting black-haired customers as soon as

1) no more blond customers are waiting and moreover

2) at least 3 black-haired customers are waiting.

As long as he is set on black-haired customers, he calmly lets the blond ones wait; he sets himself to cutting blond customers only as soon as

1) no more black-haired customers are waiting and moreover

2) at least 1 blond customer is waiting.

The examinees are requested to fill in the parts (1), (2), (3) and (4) missing from the program; for section (4) a blond and a black version must be made.

Customers of the same hair colour are helped in order of arrival; begin the program with the barber set on blond customers.

The variables which have already been included in the declaration at the beginning of the program have been introduced there in order to facilitate the marking of the programs, and may furthermore be regarded as a hint towards the solution.

The variables "rangnummer[i]" are intended to indicate whether the i-th customer must be cut; if not, then let "rangnummer[i] = 0", and if so, then let "rangnummer[i]" equal the arrival number of customers with the same hair colour. Incoming black-haired and blond customers respectively are counted, by hair colour, by the quantities "lopend zwart" and "lopend blond" respectively, both counts beginning with 1.

The variable "aantal zwarte wachters" is to equal the number of black-haired customers with "rangnummer[i] > 0".

In order to determine, from a hair group, the longest-waiting customer, one chooses from that group the customer with minimal positive rank number; the examinee need make no extra effort to speed up this search process — we may, so to speak, assume that this search process pales into insignificance beside the cutting itself.

transcribed by Sam Samshuijzen

revised Wed, 19 Jul 2006
