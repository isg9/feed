---
title: "Keuze tussen symcharf en symchart (EWD164)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD164.html
published: "1966-06-02T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD164.PDF
---

# Keuze tussen symcharf en symchart

Keuzemechanisme tussen symcharf en symchart. In het volgende beperk ik me tot het vaststellen, of een band als flexowriterband, dan wel als Telexband geinterpreteerd gaat worden: ik sta toe, dat dit proces tot een decisie komt, zonder dat alle statusvariabelen van CHARF resp. CHART vanuit de band gedefinieerd zijn. De ontbrekenden zullen volgens conventie geinitialiseerd moeten worden. Evenzo valt de detectie van een loslopende telexwagenterug buiten het bestek van de keuze.

Voor de beschrijving zal ik (voor de duidelijkheid, dan wel als mystificatieoefening) van BNF gebruik maken; de syntactische dubbelzinnigheden zijn dan voor de fijnproevers. Heptaden zal ik aangeven door hun decimaal equivalent tussen aanhalingstekens. Relevante waarden zijn voor de teleprinter: 2(wagenterug), 8(nieuweregel), 27(cijfershift) en 31(lettershift); voor de flexowriter: 11(stopcode), 26(carriagereturn), 122(lowercase) en 124(uppercase).

<vijferase>::= "31" | <vijferase> "31"

<flexonoise>::= "0" | "11" | "127"

<flexoskip>::= <flexoskip> <flexonoise> | <flexonoise>

<flexoleader>::= <empty> | <vijferase> | <vijferase> <flexoskip> | <flexoskip>

<flexCR>::= <flexoleader> "26"

<flexoLO>::= <flexoleader> "122"

<flexoUP>::= <flexoleader> "124"

<telcar>::= "2" | "8"

<telcarLE>::= "31" <telcar> | "31" <telcarLE>

<telCY>::= "27" | "31" <telCY>

<teleleader>::= <empty> | <flexoskip> | <vijferase> "0" | teleleader> "0"

<telxCY>::= <teleleader> <telCY>

<telxCARun>::= <teleleader> <telcar>

<telxCARLE>::= <teleleader> <telcarLE>

Codekeuze en initialisatie vindt plaats bij detectie van

<flexCR> Flexowriter, wagenstand = 0, case volgend conventie

<flexoLO> Flexowriter, wagenstand volgens conventie, lowercase

<flexoUP> Flexowriter, wagenstand volgens conventie, uppercase

<telxCY> Teleprinter, wagenstand volgens conventie, cijfershift

<telxCARun> Teleprinter, wagenstand = 0, shift volgens conventie

<telxCARLE> Teleprinter, wagenstand = 0, lettershift.

transcribed by Carl Ludwigson
revised Mon, 25 Oct 2010

---

## English translation

### Choice between symcharf and symchart

Choice mechanism between symcharf and symchart. In what follows I restrict myself to determining whether a tape is going to be interpreted as a flexowriter tape or as a Telex tape: I permit this process to reach a decision without all the status variables of CHARF resp. CHART being defined from the tape. The missing ones will have to be initialized according to convention. Likewise the detection of a runaway telex carriage return falls outside the scope of the choice.

For the description I shall (for the sake of clarity, or else as an exercise in mystification) make use of BNF; the syntactic ambiguities are then for the connoisseurs. Heptads I shall denote by their decimal equivalent between quotation marks. Relevant values are, for the teleprinter: 2(carriage return), 8(new line), 27(figure shift) and 31(letter shift); for the flexowriter: 11(stop code), 26(carriage return), 122(lowercase) and 124(uppercase).

<vijferase>::= "31" | <vijferase> "31"

<flexonoise>::= "0" | "11" | "127"

<flexoskip>::= <flexoskip> <flexonoise> | <flexonoise>

<flexoleader>::= <empty> | <vijferase> | <vijferase> <flexoskip> | <flexoskip>

<flexCR>::= <flexoleader> "26"

<flexoLO>::= <flexoleader> "122"

<flexoUP>::= <flexoleader> "124"

<telcar>::= "2" | "8"

<telcarLE>::= "31" <telcar> | "31" <telcarLE>

<telCY>::= "27" | "31" <telCY>

<teleleader>::= <empty> | <flexoskip> | <vijferase> "0" | teleleader> "0"

<telxCY>::= <teleleader> <telCY>

<telxCARun>::= <teleleader> <telcar>

<telxCARLE>::= <teleleader> <telcarLE>

Code choice and initialization take place upon detection of

<flexCR> Flexowriter, carriage position = 0, case according to convention

<flexoLO> Flexowriter, carriage position according to convention, lowercase

<flexoUP> Flexowriter, carriage position according to convention, uppercase

<telxCY> Teleprinter, carriage position according to convention, figure shift

<telxCARun> Teleprinter, carriage position = 0, shift according to convention

<telxCARLE> Teleprinter, carriage position = 0, letter shift.

transcribed by Carl Ludwigson
revised Mon, 25 Oct 2010
