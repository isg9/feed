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

---

## English translation

### Examination problem "Cooperating Sequential Processes" (April 1966)

Examination problem "Cooperating Sequential Processes"

April 1966

In each of five cyclic processes (named, respectively, "A", "B", "C", "D" and "E") there occurs a critical section (named, respectively, "TA", "TB", "TC", "TD" and "TE"). They are critical in the sense that, for their execution times, five mutual exclusion relations must be guaranteed:

1) the simultaneous execution of TA and TB is forbidden

2) the simultaneous execution of TB and TC is forbidden

3) the simultaneous execution of TC and TD is forbidden

4) the simultaneous execution of TD and TE is forbidden

5) the simultaneous execution of TE and TA is forbidden.

The cyclic processes are started outside their critical section; as long as a process must wait at the entrance of its critical section, this may not, for any of the remaining processes, constitute a permanent obstacle to entering their critical section.

The structure of process A is as follows:

A: begin

start: non critical section A;

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

The structure of the remaining processes is analogous.

The pieces 1A, 2A, 1B, 2B, 1C, 2C, 1D, 2D, 1E and 2E, together with the enclosing block, must be filled in.

The examinees are requested to hand in only paper written on one side.

transcribed by Carl Ludwigson

revised
12-Nov-2014
