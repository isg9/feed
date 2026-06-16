---
title: "Error checking (EWD111)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD111.html
published: "1964-11-27T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD111.PDF
---

# Error checking

EWD 111

8 december 1964

Error checking

Voornamelijk is vandaag aandacht geschonken aan de wijze van terugmelding.

De vertaler nummert de niet-lege regels van de programmatext. Als bijproduct geeft hij via de output van een aantal labels het regelnummer van de regel, waarin ze voorkomen. Procedure declarations zijn ook mooi.

Opm. 1 De vertaler zal hier identifiers in 1 alphabet uitprinten.

Opm. 2. De output kan aangeven of het een label dan wel een procedure identifier is;

kan zelfs iets van de blokstructuur proberen aan te geven. Als de text zo slechts,

dat we een lower bound voor label aanzien, dan zal deze print wel wat cryptisch worden !

Probleem: in de dynamische plaatsbepaling ontmoeten we ook MCP's. In het minimale geval zeggen we, dat elke bibliotheekprocedure op 1 regel staat. Een alternatief is, om elke MCP moedwillig in een aantal regels onder te verdelen; netjes is om bij de terugmelding ook de naam van de MCP te vermelden. (Dit kon er wel eens toe leiden, dat we elke MCP een naam geven.)

Statische checking.

De vertaling zal bestaan uit een aantal passes (eigenlijk hoop ik van niet te veel). We zullen proberen om bij de detectie van een fout althans de pass, waarin deze fout gedetecteerd wordt, af te maken met de bedoeling meer fouten van hetzelfde allooi te vinden. (Of dat altijd kan, is heel erg de vraag: is niet de executie "de laatste pass"?)

Wat betreft error checking zie ik maar 2 passes noodzakelijk optreden:

1) analyse van delimiter structure en opbouw van de naamlijst

2) type checking etc. dwz. alles, waarvoor je de declaratie gezien moet hebben.

Opm. 3 Als ik in de pass, waarin de regelnummers uitgedeeld worden ook al check, zullen die foutmeldingen er wel doorheen komen; dit hindert niet.

Opm. 4 Een voordeel van Cor's regelnummering is, dat deze uniek is, ook in die gevallen, waarin de systactische structuur macroscopisch niet klopt.!

Ik stel me voor, dat de vertaler in een left to right pass, waarin de regelscheidingen in de source text nog herkenbaar zijn, het uiteindelijke programma definitief over de segmenten verdeelt. Op dat moment kan de vertaler dus een lijst maken van invariante beginadressen (programma adressen), in de volgorde van opklimmend regelnummer.

(Ik neem aan, dat voor de opeenvolgende programma segmenten opeenvolgende SV's gebruikt kunnen worden, zodat we die niet hoeven te ketenen.) Bij de dynamische plaatsmelding kan deze lijst geraadpleegd worden.

Statische errormelding zal bestaan uit:

1) korte beschrijving van de gemaakte fout. (Omdat we een trommel en een high speed printer hebben, moeten we hiermee niet te karig zijn, dus niet "fout 7")

2) regelnummer.

Dynamische errormelding zal bestaan uit:

1) korte beschrijving van de soort fout (zie boven)

2) plaats.

Ruwweg komt deze opgave op het volgende neer:

je geeft een regelnummer (afgeleid uit een invariant adres). Via inspectie van de stapel vindt je de aard van het niveau (te weten binnenblok, parameter of fictitious block). Deze aard meldt je; als het een binnenblok is, ga je de dynamische ketting af, totdat je een fictitious block vindt (of de buitenkant). Dan heb je in de stapel een returnadres en vervolgens ga je dat analyseren. Aangezien parameters geen binnenblokken hebben zijn

er dus drie mogelijkheden:

1) parametersituatie

2) block (dwz. buitenste of binnenblok van een procedure)

3) fictitious block.

Opm. Als we het fictitious block gevonden hebben, staat in de stapel het invariante starting address van de procedure, die we wilden verlaten. Een lijstje is voldoende om de procedure identifier er bij te verschaffen.

Post mortem.

Als we ons op het standpunt stellen, dat een uniforme post mortem reactie (bv. altijd alles) irreeel is, dan laat zich dit als volgt sturen. Vooraan de band (programheading) is een entry, die de aard van de post mortem dump aangeeft. De hier mogelijke notities worden met 1 uitgebreid, die betekent "als een dynamische fout optreedt, vraag dan via toetsenbord". Dit geeft alle mogelijke flexibiliteit zonder last als je er geen behoefte aan hebt. (Je kunt dit uitbreiden met bv. "De lege post mortem notitie in de heading betekent niets".)

Verder hebben we te dumpen anoniemen, scalairen en arrays.

ad1) de anoniemen zijn wat moeilijk, omdat we de interpretatie niet in de stapel vinden. De dumper kan integers uitprinten en zo mogelijk (tekenconsistentie) ten overvloede paren ook als drijvend getal. De interpretatie van deze rommel is in elk geval voorbehouden aan specialisten (men denke bv. aan de verwisseling van primaries !)

ad2) de scalairen hebben twee moeilijkheden

2a) bij de scalairen van een expliciet programma moet je, als je enig fatsoen hebt, de indentifiers —zij het dan gereduceerd tot een alphabet— er bij fokken. Taakje voor de vertaler en de trommel!

2b) bij de handgecodeerde MCP's heb je niet een twee drie een naamlijst. Het alternatief dat we voor de anonymen bedacht hebben, kon hier wel eens aangewezen zijn (format kan hier nl. wisselen.)

ad3) arrays kunnen zo groot zijn. Wat we met arrays in MCP's moeten doen is nog minder duidelijk, maar op dit ogenblik willen we daar niet te veel over denken.

Statische checks zullen omvatten: delimiter structure, identifier matching, correct gebruik (subscripts etc) en type checking. Identifier matching omvat een duidige correspondentie vaststellen tussen gebruik en declaratie, vangen van meervoudige declaratie in een blok, multipele formals, complete specificatie, controle van de value list et.

Dynamische checks omvatten: actual formal correspondence (inclusief tellen aantal parameters) array declaratie (lower bound niet groter dan de upper bound) subscript values within bounds, formal array's test op aantal subscripts, formal left hand side consistency by multiple assignment, complex ~real barrier, sqrt, ln, to the power

integer division, input output control, integer overflow

Wat we niet checken is: delen door 0, feitelijke assignment in type procedure, undefined variable (grote voorkeur, althans in het begin, voor de uniforme reactie) niet eindigende recursie (maar die wordt via de stack control gevangen) en de blinde loop

transcribed by Sam Samshuijzen

revised Sun, 16 Jan 2005

---

## English translation

### Error checking

EWD 111

8 December 1964

Error checking

Today attention has chiefly been devoted to the manner of reporting back.

The translator numbers the non-empty lines of the program text. As a by-product it gives, via the output, the line number of the line in which they occur for a number of labels. Procedure declarations are nice too.

Rem. 1 The translator will here print out identifiers in 1 alphabet.

Rem. 2. The output can indicate whether it is a label or a procedure identifier;

it can even try to indicate something of the block structure. If the text is so bad,

that we take it for a lower bound on labels, then this printout will become rather cryptic!

Problem: in the dynamic location determination we also encounter MCPs. In the minimal case we say that every library procedure stands on 1 line. An alternative is to deliberately divide every MCP across a number of lines; the tidy thing is to mention, in the report-back, the name of the MCP as well. (This could well lead to our giving every MCP a name.)

Static checking.

The translation will consist of a number of passes (in fact I hope of not too many). We shall try, upon the detection of an error, at least to finish the pass in which this error is detected, with the intention of finding more errors of the same alloy. (Whether that is always possible is very much the question: is not execution "the last pass"?)

As regards error checking I see only 2 passes necessarily occurring:

1) analysis of delimiter structure and construction of the name list

2) type checking etc., i.e. everything for which you must have seen the declaration.

Rem. 3 If, in the pass in which the line numbers are handed out, I already check as well, then those error messages will come through all right; this does no harm.

Rem. 4 An advantage of Cor's line numbering is that it is unique, also in those cases in which the syntactic structure macroscopically does not check out.!

I imagine that the translator, in a left to right pass in which the line separations in the source text are still recognizable, definitively distributes the eventual program over the segments. At that moment the translator can therefore make a list of invariant beginning addresses (program addresses), in the order of ascending line number.

(I assume that for the successive program segments successive SVs can be used, so that we need not chain them.) At the dynamic location reporting this list can be consulted.

Static error reporting will consist of:

1) short description of the error made. (Because we have a drum and a high speed printer, we should not be too stingy with this, so not "error 7")

2) line number.

Dynamic error reporting will consist of:

1) short description of the kind of error (see above)

2) location.

Roughly this task comes down to the following:

you give a line number (derived from an invariant address). By inspection of the stack you find the nature of the level (namely inner block, parameter or fictitious block). This nature you report; if it is an inner block, you go down the dynamic chain until you find a fictitious block (or the outside). Then you have in the stack a return address and you then go on to analyse that. Since parameters have no inner blocks there are

therefore three possibilities:

1) parameter situation

2) block (i.e. outermost or inner block of a procedure)

3) fictitious block.

Rem. When we have found the fictitious block, the invariant starting address of the procedure which we wished to leave stands in the stack. A little list is sufficient to supply the procedure identifier along with it.

Post mortem.

If we take the standpoint that a uniform post mortem reaction (e.g. always everything) is unrealistic, then this can be steered as follows. At the front of the tape (program heading) there is an entry which indicates the nature of the post mortem dump. The notations possible here are extended by 1, which means "if a dynamic error occurs, then ask via keyboard". This gives every possible flexibility without burden if you have no need of it. (You can extend this with e.g. "The empty post mortem notation in the heading means nothing".)

Furthermore we have to dump anonymous quantities, scalars and arrays.

re 1) the anonymous quantities are somewhat difficult, because we do not find the interpretation in the stack. The dumper can print out integers and where possible (sign consistency) superfluously print pairs as floating point number too. The interpretation of this rubbish is in any case reserved for specialists (one may think for instance of the interchange of primaries!)

re 2) the scalars have two difficulties

2a) for the scalars of an explicit program you must, if you have any decency, throw in the identifiers along with them —reduced, then, to one alphabet. A little task for the translator and the drum!

2b) for the hand-coded MCPs you do not have a name list just like that. The alternative which we devised for the anonymous quantities could well be indicated here (for the format can vary here.)

re 3) arrays can be so large. What we are to do with arrays in MCPs is even less clear, but at this moment we do not wish to think too much about that.

Static checks will comprise: delimiter structure, identifier matching, correct use (subscripts etc) and type checking. Identifier matching comprises establishing an unambiguous correspondence between use and declaration, catching of multiple declaration within a block, multiple formals, complete specification, control of the value list etc.

Dynamic checks comprise: actual formal correspondence (including counting the number of parameters) array declaration (lower bound not greater than the upper bound) subscript values within bounds, formal arrays test on number of subscripts, formal left hand side consistency by multiple assignment, complex ~real barrier, sqrt, ln, to the power

integer division, input output control, integer overflow

What we do not check is: division by 0, actual assignment in a type procedure, undefined variable (great preference, at least in the beginning, for the uniform reaction) non-terminating recursion (but that is caught via the stack control) and the blind loop

transcribed by Sam Samshuijzen

revised Sun, 16 Jan 2005
