---
title: "Plotting a curve with a printer (EWD260)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD02xx/EWD260.html
published: "1969-04-07T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd02xx/EWD260.PDF
---

# Plotting a curve with a printer

EWD 260

Plotting a curve with a printer.

given:  0 ≤ fx(i) < 100   and   0 ≤ fy(i) < 50   for   0 ≤ i < 1000

COMPFIRST

begin

draw: {build; print};

var image;

instr build(image), print(image)

end

CLEARFIRST

begin

build: {clear; setmarks};

instr clear(image), setmarks(image)

end

ISCANNER

begin integer i;

setmarks: {i:= 0; while i < 1000 do {addmarks; i plus 1}};

instr addmark(i,image)

end

COMPPOS

begin integer x, y;

addmark: {x:= fx(i); y:=fy(i); markpos};

instr markpos(x,y,image)

end

LINER

begin integer j;

image: {array line[0:49]};

print: {j:= 49; while j ≥ 0 do {lineprint(line[j]); j minus 1}};

clear: {j:= 49; while j ≥ 0 do {lineclear(line[j]); j minus 1}};

markpos: {linemark(line[y])};

type line;

instr lineprint(line), lineclear(line), linemark(x,line)

end

LONGREP

begin integer k;

line: {integer array sym[0:99]};

lineprint: {k:= 0; while k < 100 do {PRSYM(sym[k]); k plus 1}; NLCR};

lineclear: {k:= 0; while k < 100 do { sym[k]:= space; k plus 1}};

linemark: {sym[x]:= mark}

end

SHORTREP

begin integer k;

line: {integer f; integer array sym[0:99]};

lineprint: {k:= 0; while k < f do {PRSYM(sym[k]); k plus 1}; NLCR};

lineclear: {f:= 0};

linemark: {while f ≤ x do {sym[f]:= space; f plus 1};

sym[x]:= mark}

end

N.V.PHILIPS' COMPUTER INDUSTRIE

APELDOORN

transcribed by Corrado Cantelmi

revised Fri, 2 May 2008
