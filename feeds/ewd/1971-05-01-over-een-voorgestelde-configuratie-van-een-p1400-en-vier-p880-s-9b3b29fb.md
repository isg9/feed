---
title: "(over een voorgestelde configuratie van een P1400 en vier P880’s) (EWD309)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD03xx/EWD309.html
published: "1971-05-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd03xx/EWD309.PDF
---

# (over een voorgestelde configuratie van een P1400 en vier P880’s)

Het idee is zo simpel. Je hebt kleine programma's en grotere programma's.
Plaats daarom een configuratie van wat kleine machines en een grotere, dan
kunnen de kleinere programma's op de kleine en de groter programma's op de
grotere machine. Simple comme Bonjour!

Bij nadere beschouwing blijkt het echter een drogreden, want het is bij rekenmachines niet waar, dat vele kleintjes een grote maken. Het zou het leven voor gebruikers een stuk ingewikkelder maken, zelfs als de machines compatibel zouden zijn (wat in het onderhavige voorstel niet eens het geval is, maar daarover later).

Een gewetensvol programmeur houdt rekening met de beperkingen van de
machine, die zijn programma moet uitvoeren. (De gemaakte opmerkinge, dat lang
niet alle programmeurs in dit opzicht altijd zo gewetensvol zijn is irrelevant:
je intensiefste gebruikers worden het heus wel en we hebben het over hen, niet
over de beginnertjes die ook eens een broddellapje mogen breien.) Het voorstel
zou om te beginnen impliceren, dat de gebruiker de macroscopische karakteristieken
van twee verschillende machines moet kennen. Hinderlijk.

Nog hinderlijker is, dat de gebruiker moet kiezen, welke machine het programa
moet uitvoeren. Vaak moet die keuze gemaakt worden voordat het programma
gemaakt is omdat de keuze de structuur van het te maken programma zal beinvloeden.
Dit betekent dat een programmeur moet uitmaken, wanneer een programma "klein"
en wanneer "groot" is. Het werd voorgesteld of rekentijd hiervoor de maatstaf
was. Dat is al niet aantrekkelijk, want afhankelijk van invoergegevens kan
hetzelfde programma dan soms klein en soms groot zijn. Een ander aspect is
benodigde werkruimte, maar ook dit kan voor het zelfde programma van run tot
run verschillen. Tenslotte is er nog zoiets als lengte van de programmatekst.
Gebruikers zouden zich bv. misschien wel moeten realiseren hoe lang de teksten
van bibliotheekprocedures zijn! Maar het allerhinderlijkste is, dat deze
verschillende maatstaven best tegen elkaar in kunnen wijzen, wat de gebruiker
zou confronteren met het soort conflicten, die wij hem nu juist zo graag wilden
besparen.

Totzover opmerkingen, die (enigszins) betrekking hebben op het programmabestand zoals zich dat tot op heden heeft ontwikkeld. Bij de keuze van een volgende machine hebben wij echter meer verantwoordelijkheden: de machine moet de wijze van werken, waar wij naar streven, bevorderen. Willen meer geavanceerde of in enigerlei opzicht meer gestroomlijnde machinetoepassingen van de grond komen, dan kan dit niet door ieder programma iedere keer weer van begin tot eind opnieuw uit te drukken op hetzelfde standaardniveau van een vaste programmeertaal. Middelen hiertoe zijn de opbouw van grote procedurebibliotheken en (special purpose) language processors. Het gebruik hiervan heeft ongetwijfeld tot gevolg dat meer programma's —in elk geval qua tekstlengte— groot worden. Dit is een gewenste ontwikkeling; door de voorgestelde configuratie van "vele kleintjes die niet een grote maken" zou hij echter tegengewerkt worden. (Tot overmaat van ramp kunnen we hierbij nog aantekenen, dat alle machines uit de configuratie als voorgesteld, met linkage editing werken, waardoor programmalengte een extra penalty veroorzaakt,)

Tenslotte zijn de machines niet compatibel, zo incompatibel, dat zelfs de ALGOL-systemen moeten verschillen. De hoop, dat deze verschillen door software realistisch gemaskeerd zouden kunnen worden, moet als illusoir gekenschetst worden. Zonder dit incompatibiliteitsdefect is de voorgestelde multi-machine-configuratie al hoogst onaantrekkelijk, met dit defect volstrekt onacceptabel.

15 april 1971prof.dr.Edsger W.Dijkstra

transcribed by Hans Terlouw

revised
Sat, 9 Jan 2010

---

## English translation

### (on a proposed configuration of one P1400 and four P880's)

The idea is so simple. You have small programs and larger programs.
Therefore set up a configuration of a few small machines and a larger one, and then
the smaller programs can run on the small ones and the larger programs on the
larger machine. Simple comme Bonjour!

On closer inspection, however, it turns out to be a fallacy, for with computers it is not true that many little ones make a big one. It would make life for users considerably more complicated, even if the machines were compatible (which in the present proposal is not even the case, but more on that later).

A conscientious programmer takes into account the limitations of the
machine that has to execute his program. (The remark that has been made, that by no
means all programmers are always so conscientious in this respect, is irrelevant:
your most intensive users do indeed become so, and it is they we are talking about, not
the little beginners who may also be allowed to knit a botched-up rag now and then.) The proposal
would, to begin with, imply that the user must know the macroscopic characteristics
of two different machines. Annoying.

Even more annoying is that the user must choose which machine is to execute the program.
Often that choice has to be made before the program
has been written, because the choice will influence the structure of the program to be written.
This means that a programmer must determine when a program is "small"
and when "large". It was suggested whether computing time was the criterion for this.
That is already not attractive, for depending on the input data
the same program may then sometimes be small and sometimes large. Another aspect is
the working space required, but this too can differ for the same program from run to
run. Finally there is also such a thing as the length of the program text.
Users would, for instance, perhaps have to realize how long the texts
of library procedures are! But the most annoying thing of all is that these
different criteria may well point against one another, which would confront the user
with the kind of conflicts that we precisely so much wanted
to spare him.

So far for remarks that bear (somewhat) on the body of programs as it has developed up to now. In the choice of a next machine, however, we have more responsibilities: the machine must promote the way of working we are striving for. If more advanced or in some respect more streamlined machine applications are to get off the ground, then this cannot be done by expressing every program over and over again from beginning to end at the same standard level of a fixed programming language. The means to this end are the build-up of large procedure libraries and (special purpose) language processors. The use of these undoubtedly has the consequence that more programs —at any rate in terms of text length— become large. This is a desirable development; by the proposed configuration of "many little ones that do not make a big one", however, it would be counteracted. (To make matters worse, we may note here as well that all machines in the configuration as proposed work with linkage editing, whereby program length causes an extra penalty,)

Finally, the machines are not compatible, so incompatible that even the ALGOL systems must differ. The hope that these differences could be realistically masked by software must be characterized as illusory. Without this incompatibility defect the proposed multi-machine configuration is already highly unattractive; with this defect utterly unacceptable.

15 April 1971prof.dr.Edsger W.Dijkstra

transcribed by Hans Terlouw

revised
Sat, 9 Jan 2010
