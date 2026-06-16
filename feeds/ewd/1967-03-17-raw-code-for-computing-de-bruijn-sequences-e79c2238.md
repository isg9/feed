---
title: "[Raw code for computing De Bruijn-sequences] (EWD221)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD02xx/EWD221.html
published: "1967-03-17T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd02xx/EWD221.PDF
---

# [Raw code for computing De Bruijn-sequences]

EWD221 -
0

array d[0:36];
d[0] := 0; d[1] := 0; d[2] := 0; d[3] := 0; d[4] := 0; d[5] := 0;
k := 5;
4 and k herhaal

begin j := k-1; geluk := true;
j ≥ 4 and geluk herhaal
begin gelijk := true; h := 0;
h ≤ 4 and gelijk herhaal
begin gelijk := (d[k-h] = d[j-h]); h := h+1 end;
gelijk zoja geluk := false; j := j-1
end;
geluk dan
begin k := k+1; k ≤ 31 dan d[k] := 0 anders d[k] := d[k-32] and
anders
begin k > 31 zoja k := 31;
d[k] = 1 zoja k := k-1;
d[k] := 1
end
end

Transcribed by Mikhail Esteves

Last revised on Sat, 19 Jul 2003.
