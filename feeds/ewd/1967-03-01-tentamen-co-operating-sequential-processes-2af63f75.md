---
title: "Tentamen “Co-operating Sequential Processes” (EWD190)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD01xx/EWD190.html
published: "1967-03-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd01xx/EWD190.PDF
---

# Tentamen “Co-operating Sequential Processes”

EWD 190

Tentamen "Co-operating Sequential Processes."
Aan een grote ronde tafel zijn de plaatsen cyclisch genummerd van 0 t/m 39; de leden van een gezelschap van 40 personen zijn eveneens genummerd van 0 t/m 39, ieder mag alleen op zijn eigen plaats - dwz. de plaats, die zijn eigen nummer heeft - aan tafel aanzitten.

Hun gedragspatroon bestaat uit een niet-eindigende cyclische opeenvolging van:

leven (buitenskamers);

borrelen (binnenskamers, maar niet aan tafel);

eten (binnenskamers, aan tafel);

en alleen voor de heren

sigaren (binnenskamers, niet aan tafel).

Hun gedragingen moeten zo gesynchroniseerd zijn dat

1) nimmer drie cyclische opeenvolgende plaatsen aan tafel tegelijkertijd bezet zijn, en

2) geen dame van tafel opstaat, tenzij er tenminste 1 andere dame in de kamer is.

Voor de dames (die qualitate qua na tafel geen sigaar roken) is het opstaan van tafel en het
verlaten van de kamer eenzelfde handeling. Eventuele personen, die met aan tafel gaan moeten wachten (wegens regel 1) op het van tafel opstaan van dame A, die (wegens regel 2) met opstaan moet wachten totdat, zeg, dame B binnenkomt, hebben ten aanzien van het aan tafel gaan voorrang boven dame B.

Er zijn tenminste twee dames in het gezelschap, dat met iedereen buitenskamers levend
geinitialiseerd wordt.

De oplossing acht (ik) in het omvattend universum gedeclareerd:

1)
de boolean
procedure dame(u); value u; integer u; deze heeft de waarde "true" als persoon nr. u een dame is, de waarde "false"
als persoon nr. u een heer is.

2)
de integer procedure mod40(u); value u; integer u; deze heeft de waarde van u, gereduceerd modulo 40.

Gegeven is een oplossing van de volgende structuur.

begin semaphore mutex, damutex; semaphore array persem [0:39];
boolean array honger [0:39]; integer array eettal [0:39];
integer damnr, k;
procedure test(u); value u; integer u;
begin if honger[u] then
begin if eettal[mod40(u-1)] < 2 and
eettal[u] < 2 and
eettal[mod40(u+1)] < 2 then
begin eettal[mod40[u-1)] := eettal[mod40(u-1)] + 1;
eettal[u] := eettal[u] + 1;
eettal[mod40(u+1)] := eettal[mod40(u+1)] + 1;
honger[u] := false ; V(persem[u])
end
end
end

procedure staop(u); value u; integer u;
begin eettal[mod40(u-1)] := eettal[mod40(u-1)] - 1;
eettal[u] := eettal[u] - 1;
eettal[mod40(u+1)] := eettal[mod40(u+1)] - 1;
test(mod40(u-2)); test(mod40(u-1));
test(mod40(u+1)); test(mod40(u+2))
end
for k := 0 step 1 until 39 do
begin honger[k] := false; eettal[k] := 0; persem[k] := 0 end;
mutex := 1; damutex := 1; damnr := -1; k := 0;

parbegin
persoon 0 : begin..... ..... ..... .....end;
⋮
persoon39 : begin..... ..... ..... .....end
parend
end

waarbij de structuur van persoon h is als volg:

persoon h:
begin Lh: leven;
if dame(h) then
begin P(damutex); k := k + 1;
if damnr ≥ 0 then
begin P(mutex); staop(damnr); V(mutex);
V(persem[damnr]); damnr := -1
end;
V(damutex)
end;
borrelen;
P(mutex); honger[h] := true; test(h); V(mutex); P(persem[h]);
eten;
if dame(h) then
begin P(damutex); k := k - 1;
if k = 0 then
begin damnr := h; V(damutex); P(persem[h]) end
else
begin P(mutex); staop(h); V(mutex); V(damutex) end
end
else
begin P(mutex); staop(h); V(mutex);
sigaarroken
end;
goto Lh
end

Opgave. Bewijs, dat de gegeven oplossing aan alle specificaties voldoet.

NB. De tentaminandi wordt verzocht hun papier slechts eenzijdig te beschrijven.

transcribed by Sam Samshuijzen

revised Mon, 15 Jan 2007


---

## English translation

### Examination "Co-operating Sequential Processes"

EWD 190

Examination "Co-operating Sequential Processes."
At a large round table the places are numbered cyclically from 0 to 39; the members of a company of 40 persons are likewise numbered from 0 to 39, and each may sit at the table only at his own place — that is, the place that bears his own number.

Their behaviour pattern consists of a non-terminating cyclic succession of:

living (out of doors);

having drinks (indoors, but not at the table);

eating (indoors, at the table);

and, for the gentlemen only,

cigars (indoors, not at the table).

Their behaviours must be synchronized in such a way that

1) never are three cyclically consecutive places at the table occupied simultaneously, and

2) no lady rises from the table unless there is at least 1 other lady in the room.

For the ladies (who, qualitate qua, do not smoke a cigar after dinner) rising from the table and
leaving the room are one and the same action. Any persons who, in order to take their place at the table, must wait (on account of rule 1) for lady A to rise from the table, who (on account of rule 2) must wait with rising until, say, lady B enters, have, with respect to taking their place at the table, priority over lady B.

There are at least two ladies in the company, which is initialized with everyone living out of
doors.

The solution assumes (I) the following to be declared in the enclosing universe:

1)
the boolean
procedure dame(u); value u; integer u; this has the value "true" if person no. u is a lady, the value "false"
if person no. u is a gentleman.

2)
the integer procedure mod40(u); value u; integer u; this has the value of u, reduced modulo 40.

Given is a solution of the following structure.

begin semaphore mutex, damutex; semaphore array persem [0:39];
boolean array honger [0:39]; integer array eettal [0:39];
integer damnr, k;
procedure test(u); value u; integer u;
begin if honger[u] then
begin if eettal[mod40(u-1)] < 2 and
eettal[u] < 2 and
eettal[mod40(u+1)] < 2 then
begin eettal[mod40[u-1)] := eettal[mod40(u-1)] + 1;
eettal[u] := eettal[u] + 1;
eettal[mod40(u+1)] := eettal[mod40(u+1)] + 1;
honger[u] := false ; V(persem[u])
end
end
end

procedure staop(u); value u; integer u;
begin eettal[mod40(u-1)] := eettal[mod40(u-1)] - 1;
eettal[u] := eettal[u] - 1;
eettal[mod40(u+1)] := eettal[mod40(u+1)] - 1;
test(mod40(u-2)); test(mod40(u-1));
test(mod40(u+1)); test(mod40(u+2))
end
for k := 0 step 1 until 39 do
begin honger[k] := false; eettal[k] := 0; persem[k] := 0 end;
mutex := 1; damutex := 1; damnr := -1; k := 0;

parbegin
person 0 : begin..... ..... ..... .....end;
⋮
person39 : begin..... ..... ..... .....end
parend
end

where the structure of person h is as follows:

person h:
begin Lh: living;
if dame(h) then
begin P(damutex); k := k + 1;
if damnr ≥ 0 then
begin P(mutex); staop(damnr); V(mutex);
V(persem[damnr]); damnr := -1
end;
V(damutex)
end;
having drinks;
P(mutex); honger[h] := true; test(h); V(mutex); P(persem[h]);
eating;
if dame(h) then
begin P(damutex); k := k - 1;
if k = 0 then
begin damnr := h; V(damutex); P(persem[h]) end
else
begin P(mutex); staop(h); V(mutex); V(damutex) end
end
else
begin P(mutex); staop(h); V(mutex);
smoking a cigar
end;
goto Lh
end

Problem. Prove that the given solution satisfies all the specifications.

N.B. The examinees are requested to write on one side of their paper only.

transcribed by Sam Samshuijzen

revised Mon, 15 Jan 2007
