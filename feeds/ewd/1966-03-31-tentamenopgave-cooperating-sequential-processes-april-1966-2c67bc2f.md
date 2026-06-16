---
title: "Tentamenopgave “Cooperating Sequential Processes” (april 1966) (EWD158)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD158.html
published: "1966-03-31T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD158.PDF
---

# Tentamenopgave “Cooperating Sequential Processes” (april 1966)

Tentamenopgave "Cooperating Sequential Processes"

april 1966

In elk van vijf cyclische processen (resp. genaamd "A", "B", "C", "D" en "E") komt een critische sectie voor (resp. genaamd "TA", "TB", "TC", "TD" en "TE"). Zij zijn critisch in de zin, det voor hun uitvoeringstijden vijf wederzijdse uitsluitingarelaties gegarandeerd moeten worden:

1) het gelijktijdig in uitvoering zijn van TA en TB is verboden

2) het gelijktijdig in uitvoering zijn van TB en TC is verboden

3) het gelijktijdig in uitvoering zijn van TC en TD is verboden

4) het gelijktijdig in uitvoering zijn van TD en TE is verboden

5) het gelijktijdig in uitvoering zijn van TE en TA is verboden.

De cyclische processen worden buiten hun critische sectie gestart; zolang een proces bij de ingang van zijn critische sectie moet wachten, mag dit voor geen van de overige processen een blijvend beletsel vormen om hun critische sectie binnen te gaan.

De structuur van proces A is als volgt:

A: begin

start: niet critische sectie A;

.

.

.

. (1A)

.

.

TA;

.

.

.

. (2A)

.

.

goto start

end

De structuur van de overige processen is analoog.

De stukken 1A, 2A, 1B, 2B, 1C, 2C, 1D, 2D, 1E en 2E, benevens het omvattende blok moeten worden ingevuld.

De tentaminandi wordt verzocht slechts eenzijdig beschreven papier in te leveren.

transcribed by Carl Ludwigson

revised
12-Nov-2014
