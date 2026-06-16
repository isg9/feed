---
title: "Ruwe schets vertaalproces (EWD112)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD112.html
published: "1964-12-06T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD112.PDF
---

# Ruwe schets vertaalproces

EWD112 - 0

Ruwe schets vertaalproce

We zullen proberen om de vertaler in twee left to right scans in elkaar te draaien. In de eerste scan gebeurt onder meer het volgende.

De heptaden van de invoerstroom worden tot basic characters geassembleerd en in deze vorm wordt de text van het ALGOL programma ten bate van de tweede scan opgeborgen. In deze representatie komen enige non-ALGOL characters voor, zoals "Nieuwe Regel" en "illigitiem", bv. een rare onderlijning. We zullen nl. proberen om bij check alarm de pass in questie af te maken; het is niet gezegd, dat dat kan: immers, als je de executie van een programma beschouwt als "de laatste pass", wat het toch eigenlijk is, dan hebben we inmiddels al besloten, dat we in geval van alarm zullen stoppen.

Behalve, dat de text in basic characters voor later gebruik wordt opgebouwd, wordt hij in de eerste scan aan een aantal analyses onderworpen. Ten eerste zullen we de delimiter structure zorgvuldig checken; in deze phase zullen we nooit gebruik maken van "de bijbehorende declaratie" omdat die in principe nog ongezien kan wezen.

Verder is de belangrijkste taak van de eerste scan om de volledige naamlijst op te bouwen. Wij stellen ons voor, dat dit gaat via een stapel, die als transition mechanism gebruikt wordt: hierin worden alle gedeclareerde namen plus de uit deze declaratie volgende en voor de tweede pass relevante gegevens opgenomen. Zodra een blok einde gedetecteerd is, wordt de top van deze naamstapel naar de definitieve naamlijst overgeheveld. Wel moeten we door enige ketening zorgen, dat de boomstructuur van deze naamlijst correct tot uiting gebracht wordt. In deze definitieve naamlijst fokken we dus de namen bloksgewijs in volgorde van blokeinde. Hiermee hebben we bereikt, dat voor dit naamsorteerproces elke naam precies een keer verhuisd wordt. Dit is mooi.

Neventaak van de eerste scan is om de niet lege regels van de aangeboden text te nummeren en om via de output op herkenbare plaatsen het toegekende regelnummer uit te printen. (Bv. in regels, waarin labels voorkomen en regels, waarin procedures gedeclareerd worden. De output mag zich hier bedienen van een enkel alphabet.) In deze output mag zelfs wel iets van de blokstructuur tot uiting komen; als in de eerste scan al fouten ontdekt worden, komen deze foutmeldingen door dit lijstje heen, maar dit achten we geen bezwaar (we = Cor en ik).

In de tweede scan wordt het complete ALGOL-programma definitief gemaakt en op segementen ingedeeld. Dit lijkt een beetje moeilijk, maar moet wel meevallen, want we hebben drie machtige kanonnen tot onze beschikking. Ten eerste hebben we de volledige naamlijst met alle declaratiegegevens, die we maar wensen (daar had de eerste pass dan maar voor moeten zorgen). Ten tweede is dit essentieel een vertaler, die het objectprogramma in het geheugen opbouwt, ten derde hebben we de volledige text van het objectprogramma ook al in het geheugen staan en kunnen we de vertaler tamelijk straffeloos een stukje in de toekomst laten kijken.

Een en ander impliceert wel, dat de routine "read next character" een soortement van juweeltje dient te worden. Ten eerste kent hij twee toestanden: in de eerste fase moet hij het volgende basic character uit de heptaden opbouwen en dumpen, in de tweede fase moet hij "the next character" uit de gedumpte informatiestroom halen. Ter verhoging van de feestvreugde zal hij wel twee pointers moeten hebben, een testpointer en een gehadpointer.

Dit lijkt in deze staat van "ruwheid" een redelijk overzicht van de taken.

8 december 1964

Transcribed by Frank Steggink

Revised Tue, 6 Jan 2004.

---

## English translation

### Rough sketch of the translation process

EWD112 - 0

Rough sketch of the translation proces

We shall try to fold the translator into two left to right scans. In the first scan, among other things, the following happens.

The heptads of the input stream are assembled into basic characters, and in this form the text of the ALGOL program is stored away for the benefit of the second scan. In this representation a number of non-ALGOL characters occur, such as "New Line" and "illegitimate", e.g. a strange underlining. We shall namely try, in the case of a check alarm, to finish the pass in question; it is not said that this can be done: after all, if one regards the execution of a program as "the last pass", which it really is, then we have meanwhile already decided that in the case of an alarm we shall halt.

Apart from the fact that the text in basic characters is built up for later use, in the first scan it is subjected to a number of analyses. Firstly we shall carefully check the delimiter structure; in this phase we shall never make use of "the corresponding declaration", because in principle that may still be unseen.

Furthermore the most important task of the first scan is to build up the complete name list. We picture this as proceeding by means of a stack which is used as a transition mechanism: in it are recorded all declared names plus the data that follow from this declaration and are relevant for the second pass. As soon as a block end has been detected, the top of this name stack is transferred to the definitive name list. We must, however, by means of some chaining, see to it that the tree structure of this name list is correctly brought to expression. In this definitive name list we thus breed the names block by block in order of block end. With this we have achieved that, for this name sorting process, each name is moved exactly once. This is nice.

A side task of the first scan is to number the non-empty lines of the submitted text and, via the output, to print out the assigned line number at recognizable places. (E.g. in lines in which labels occur and lines in which procedures are declared. The output may here make use of a single alphabet.) In this output even something of the block structure may be brought to expression; if errors are already discovered in the first scan, these error messages come through this little list, but this we deem no objection (we = Cor and I).

In the second scan the complete ALGOL program is made definitive and divided into segments. This seems a bit difficult, but it must turn out to be manageable, for we have three powerful cannon at our disposal. Firstly we have the complete name list with all the declaration data that we could wish (the first pass should just have taken care of that). Secondly this is essentially a translator that builds up the object program in the store, thirdly we already have the complete text of the object program standing in the store as well, and we can let the translator look a little way into the future fairly unpunished.

All this does imply that the routine "read next character" ought to become a sort of little jewel. Firstly it knows two states: in the first phase it must build up and dump the next basic character out of the heptads, in the second phase it must fetch "the next character" out of the dumped information stream. To heighten the festive mood it will indeed have to have two pointers, a test pointer and a had pointer.

In this state of "roughness" this seems a reasonable overview of the tasks.

8 December 1964

Transcribed by Frank Steggink

Revised Tue, 6 Jan 2004.
