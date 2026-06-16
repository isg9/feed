---
title: "Over de bewijsbaarheid van programmacorrectheid (EWD301)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD03xx/EWD301.html
published: "1971-01-05T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd03xx/EWD301.pdf
---

# Over de bewijsbaarheid van programmacorrectheid

1)     Omdat "Verdeel en Heers" in eerste instantie het enige principe is, waarmee
we iets groots aankunnen, zal een programma naarmate het groter is uit meer
componenten opgebouwd moeten worden. Willen we voor het totale programma een
zeker betrouwbaarheidsniveau bereiken, dan dient het betrouwbaarheidsniveau van
de individuele programmacomponenten hoger te zijn naarmate het aantal componenten
groter is.

2)     Testen van programma's kan een zeer overtuigende methode zijn om de aanwezigheid
van fouten aan te tonen, maar is hopeloos inadequaat om hun afwezigheid
aan te tonen. De enige mogelijkheid die ons rest is de correctheid van het programma
te bewijzen.

3)     Aangezien de (bv. intellectuele) inspanning nodig voor de bewijsvoering
critisch afhangt van de structuur van het programma, dient de programmeur zijn
programma zo te structureren, dat bewijsvoering nog wel mogelijk is. Ik ga er
hierbij van uit dat programma's behalve voor de machine uitvoerbaar voor de mens
leesbaar moeten zijn. In het bijzonder dient de programmeur er voor te waken, dat
het aantal gevallen, dat onderscheiden moet worden, additief en vooral niet mul-
tiplicatief combineert.

4)     In dit licht dient het gebruik van de goto-statement ten sterkste afgeraden
te worden: de goto-statement is als combinatorische complexiteitsgenerator
ontmaskerd.

5)     De (bekende) sequentieringsclausules belichamen een aspect van programmeringsdiscipline,
waardoor bewijsvoering profiteren kan van de postulaten van
C.A.R.Hoare. Van het gebruik van deze postulaten zal een voorbeeld gegeven worden.
Aan de hand hiervan zal het begrip "operationele abstractie" als middel ter beknotting
van de hoeveelheid bewijswerk worden toegelicht.

6)     Aan de hand van een volgend voorbeeld zal het begrip "representationele
abstractie" worden toegelicht. Ook dit dient ter beknotting van de hoeveelheid
bewijswerk.

7)     Betoogd zal worden dat wij om de correctheid van programma's bewijsbaar te
houden, programma's moeten leren zien niet als op zichzelf staande objecten, maar
als leden van een familie aanverwante programma's: aequivalente programma's kunnen
zich hier presenteren als alternatieve verfijningen van eenzelfde abstract
programma.

8)     Tenslotte zal de hoop uitgesproken (en naar vermogen gerechtvaardigd worden)
dat de bewijslast geen taakverzwaring voor de programmeur hoeft in te houden.
Integendeel: als hij de bewijsvoering hand in hand met het programma laat groeien
lijkt hij een waardevol r�chtsnoer bij de opbouw van zijn programma gevonden te
hebben.

Edsger W.Dijkstra

---

## English translation

### On the provability of program correctness

1)     Because "Divide and Rule" is in the first instance the only principle with which
we can cope with something large, a program will, the larger it is, have to be built
up out of more components. If we wish to attain for the total program a
certain level of reliability, then the level of reliability of
the individual program components must be the higher the larger the number of components
is.

2)     Testing of programs can be a very convincing method of demonstrating the presence
of errors, but is hopelessly inadequate for demonstrating their absence.
The only possibility that remains to us is to prove the correctness of the program.

3)     Since the (e.g. intellectual) effort needed for the demonstration of the proof
depends critically on the structure of the program, the programmer should structure his
program in such a way that the demonstration of the proof is still possible at all. I assume
here that programs, besides being executable for the machine, must also be readable for man.
In particular the programmer should guard against
the number of cases that must be distinguished combining additively and above all not multiplicatively.

4)     In this light the use of the goto-statement should be most strongly discouraged:
the goto-statement has been unmasked as a generator of combinatorial complexity.

5)     The (well-known) sequencing clauses embody an aspect of programming discipline,
by virtue of which the demonstration of proofs can profit from the postulates of
C.A.R.Hoare. An example of the use of these postulates will be given.
On the basis of this the notion "operational abstraction" will be elucidated as a means for curtailing
the amount of proof work.

6)     On the basis of a following example the notion "representational
abstraction" will be elucidated. This too serves to curtail the amount of
proof work.

7)     It will be argued that, in order to keep the correctness of programs provable,
we must learn to see programs not as objects standing on their own, but
as members of a family of kindred programs: equivalent programs can
here present themselves as alternative refinements of one and the same abstract
program.

8)     Finally the hope will be expressed (and justified as far as is within our power)
that the burden of proof need not entail any increase in the programmer's task.
On the contrary: if he lets the demonstration of the proof grow hand in hand with the program,
he seems to have found a valuable guideline in the construction of his program.

Edsger W.Dijkstra
