---
title: "A bug in my book! (EWD598)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD05xx/EWD598.html
published: "1977-01-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd05xx/EWD598.PDF
---

# A bug in my book!

On page 191, at the end of Chapter 24 of A Discipline of Programming (Prentice-Hall, 1976) the following note should be added:

Note 3. In the program as published above, Mark Bebie has found an error. In order to maintain the convention:

"set(i) = 2 and set(−i) edge i is a processed edge of the clockwise boundary of the light cap"

the values of set(i) and set(-i) have to be adjusted when edge i is processed, i.e. b:hiext(i) takes places. In "extend b and refresh xx" this has been done (in the third line), in "inspect face along −xx", however, it has erroneously been omitted. Its tenth line:

do b.dom = 0 → b:hiext(lumen * yy) od

should therefore be replaced by

do b.dom = 0 →

od b:hiext(lumen * yy);
set:(b.high)= 2; set:(−b.high)= 0

(End of note 3.)

*                   *
*

Two further corrections.

On page 159, line 21 from above

begin glocon N, to, lcr; glovar from, uvl, suv, min, h; privar len;

should be replaced by

begin glocon to, lcr; glovar from, uvl, suv, min, h; privar len;

On page 141, line 11 from above

x2 + y2 ≤ r and x2 + (y + 1)2 < r

should be replaced by

x2 + y2 ≤ r and x2 + (y + 1)2 > r

Edsger W. Dijkstra

transcribed by Alejandro Ruiz

revised Wed, 29 Apr 2009
