---
title: "Keuze tussen symcharf en symchart (EWD164)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD164.html
published: "1966-06-02T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD164.PDF
---

# Keuze tussen symcharf en symchart

Keuzemechanisme tussen symcharf en symchart. In het volgende beperk ik me tot het vaststellen, of een band als flexowriterband, dan wel als Telexband geinterpreteerd gaat worden: ik sta toe, dat dit proces tot een decisie komt, zonder dat alle statusvariabelen van CHARF resp. CHART vanuit de band gedefinieerd zijn. De ontbrekenden zullen volgens conventie geinitialiseerd moeten worden. Evenzo valt de detectie van een loslopende telexwagenterug buiten het bestek van de keuze.

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

<flexCR> Flexowriter, wagenstand = 0, case volgend conventie

<flexoLO> Flexowriter, wagenstand volgens conventie, lowercase

<flexoUP> Flexowriter, wagenstand volgens conventie, uppercase

<telxCY> Teleprinter, wagenstand volgens conventie, cijfershift

<telxCARun> Teleprinter, wagenstand = 0, shift volgens conventie

<telxCARLE> Teleprinter, wagenstand = 0, lettershift.

transcribed by Carl Ludwigson
revised Mon, 25 Oct 2010
