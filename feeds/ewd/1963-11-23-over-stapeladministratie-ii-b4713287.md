---
title: "Over stapeladministratie. II (EWD70)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD70.html
published: "1963-11-23T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD70.PDF
---

# Over stapeladministratie. II

EWD 70

Over stapeladministratie. II

0. Inleiding.

Ik had aanvankelijk gehoopt, dat we een min of meer volledig overzicht zouden krijgen over de in aanmerking komende mogelijkheden en dat we tevens de criteria zouden krijgen met behulp waarvan we tussen deze mogelijkheden zouden kunnen kiezen. Dit vlot echter niet zo erg, mijn geduld raakt wat op en ik wil wel eens een voorstel zien, dat in elk geval werkt en niet te gek is. Ik hoop dus nu de verdieping van ons inzicht op een andere manier te bereiken, nl. door critische beschouwing van een concreet voorstel. De rest van deze notitie is aan zulk een voorstel gewijd.

Mijn eerste zorg zijn drie fundamentele problemen, te weten:

a) abrupte blokverschuiving

b) stapelverschuiving

c) paginavrijgave voor zg. grote arrays.

Daarbij komt dan nog een practisch probleem aan de orde, nl. optimalisering van de red - en herstelprocedure in het geval van de zg. "simpele impliciete subroutine".

Over de stapelverschuiving het volgende: behalve de lijstjes van de referentiepunten (de displays) hoeven we in de stapel geen absolute adressen op te nemen, die verwijzen naar punten in de stapel; deze laatste kunnen we altijd opbergen relatief t.o.v. het begin van de stapelbodem. (Of we het altijd zullen doen is een tweede: aangenomen dat stapelverschuivingen dingen zijn, die tot de zeldzaamheden behoren, krijgen we wellicht een sneller systeem, als we waar mogelijk met physische adressen werken.)

Uit het bovenstaande volgt, dat het belangrijk is, dat in de stapel de referentielijstjes gevonden kunnen worden; het is hier, dat we onderscheid maken tussen een procedure (dwz. nieuw lijstje) en ingewikkelde impliciete subroutine (dwz. terugvallen op een oud lijstje). De simpele impliciete subroutine kunnen we hier dacht ik buiten beschouwing laten, omdat deze immers in doofheid uitgevoerd wordt.

Ik ga nu stapelbeelden voorstellen om toe te voegen bij activering van een procedure, resp. ingewikkelde parameter.

1. Aanroep van een procedure.

Het is de bedoeling dat het volgende mechanisme gebruikt wordt zowel voor de formele als voor de nonformele procedure, zowel voor de type procedure als de non-type procedure. Het is de bedoeling dit mechanisme blindelings te volgen in het geval van de type procedure call als zelfstandig statement. (De type procedure zal zijn waarde in het F-register afleveren; bij terugkeer zal SW wijzen naar de stapeltop in de zin van Voorhoeve, dwz. naar de eerste vrije plaats in kerngeheugen. Negeren van dit resultaat is dan al heel eenvoudig!)

Ik veronderstel, dat we voor de formele plaatsen aan twee woorden genoeg hebben en dat ook de invariante representatie van het terugkeeradres aan twee woorden genoeg heeft. (Deze veronderstellingen zijn waarschijnlijk niet juist, maar dat is in dit verband niet essentieel.) Op de volgende pagina is het voorgesteld stapelbeeld gegeven.

Opm.1 Het "centerpunt" wordt dus door een statusvariabele "d" vastgehouden, die in het kader van deze introductie de waarde d = d[i] krijgt; de waarde aan call-kant - hier aangeduid

M[d – 3 – 2 * nf]
M[d – 3 – 2 * nf]
┑

│ formele plaatsen van de 1ste parameter

┙

⋮

⋮

M[d – 5]
M[d – 4]
┑

│ formele plaatsen van nf-de parameter

┙

M[d – 3]
M[d – 2]
┑

│ invariant terugkeeradres

┙

M[d – 1]
= nf

M[d]
= d[i – 1]

M[d + 1]
= L[i] (=d + 3)

M[d + 2]
= bn[i]

M[d + 3]
= 0-de referentiepunt

⋮

⋮

met d[i – 1] - wordt hier gered. De index i onderscheidt in deze beschrijving de vigerende activeringen van procedures en ingewikkelde parameters. NB. De index i speelt alleen een rol in deze beschrijving, in het werkende systeem wordt deze niet bijgehouden, we voeren dus geen dynamische diepteteller in. Met "bn[i]" is gemeend de heersende blokhoogte als geintroduceerd in EWD69; L[i] is de alvast in de stapel neergezette heersende waarde van L.

Opm.2 Het stapelen van "nf" is strikt genomen overbodig, maar ik wou het toch maar doen:

a)
het maakt een dynamische controle op de juistheid van het aantal meegegeven parameters mogelijk

b)
het maakt het - desgewenst - mogelijk om (in de bibliotheek) procedures op te nemen met een variabel aantal parameters.

c)
het terugkeermechanisme hoeft van de procedure niet meer het gegeven mee te krijgen, hoeveel formele plaatsen vergeten kunnen worden.

2. Aanroep van een gecompliceerde parameter.

Dit genereert aan de top van de stapel

M[d – 3]
M[d – 2]
┑

│ invariant terugkeeradres

┙

M[d – 1]
= negatieve value-address indicatie

M[d]
= d[i – 1]

M[d + 1]
= L[i] (= L[j] met j < i)

Dit stapelbeeld beantwoordt aan de aanroep van de impliciete subroutine. De impliciete subroutine is eenvoudiger dan de volslagen procedure in zoverre er geen parameters aan worden meegegeven; hij is ingewikkelder omdat (nl. als de actuele parameter een geindiceerde variabele is) zijn aanroep ook als "left-hand side" van de assignment statement voor kan komen.

Voor deze laatste complicatie zijn ettelijke oplossingen. Het boven gegeven stapelbeeld beantwoordt aan de techniek, waarin het objectprogramma van de body zijn behoefte kenbaar maakt met behulp van "Take formal address" of "Take formal value". Welke van deze twee aanroepen gegeven wordt, moet dan in de stapel opgeslagen worden.

Een alternatief is, dat het objectprogramma slechts zegt "Go formal" en dat de impliciete subroutine behalve zijn primitieve resultaat er bij aflevert de indicatie "address" resp. "value". Het objectprogramma kan dan aan de call kant van de impliciete subroutine inlassen

a) als linkerkant "if value alarm"

b) als rechterkant "if address, take value"

Een derde mogelijkheid is, om altijd "value" en "address" af te leveren en aan call kant het gewenste te selecteren. Hierbij doe je echter altijd te veel. (Dit is bovendien naar als de waarde nog ongedefinieerd is; ik zou niet graag door een parity fout gestraft worden.)

De bovengenoemde beelden zijn gesuggereerd in de veronderstelling dat het teken van M[d – 1] steeds descriptief zou zijn voor de situatie procedure resp. impliciete subroutine. (Dit impliceert, dat een procedure zonder parameters van de call kant nf = +0 mee moet krijgen; het onderscheid tussen "value" resp. "address" kan dan door –0 en –1 gemaakt worden.)

In beide gevallen zouden we zonder de indicatie in M[d – 1] kunnen; het onderscheid tussen de twee stapelbeelden kunnen we ook maken door de testen of er een nieuwe referentielijst is ingewoerd, dwz. of M[d + 1] = d + 3 is of niet.

3. Simpele impliciete subroutine.

In dit geval hoeft alleen het terugkeeradres in de stapel opgenomen te worden. Het onderscheid tussen value en address bestaat niet, het is nl. altijd value. (De enkele variabele identifier als actuele parameter geeft nl. geen aanleiding tot een impliciete subroutine; het moet een echte expressie zijm. Hier is het feit, dat de indicering in de actuele parameter de impliciete subroutine per definitie ingewikkeld maakt een meevaller!)

4. Stapelverschuiving.

Tijdens de simpele impliciete subroutine kan geen interruptie optreden en kan dus ook geen behoefte aan verschuiving optreden. In alle andere gevallen geeft de heersende waarde van de statusvariabele "d" ons de clou, hoe we verder moeten gaan. De referentielijsten kunnen we feilloos vinden en dus kunnen we de verschuiving bijwerken. De gestapelde L-waarden en de geketende d-waarden kunnen we eveneens feilloos vinden en ook hier kunnen we ons de luxe van physische adressen veroorloven. Dankzij de conventie van de gestapelde nf kunnen we ook alle formele plaatsen eenduidig vinden. Als we hier een goede layout voor kiezen, kunnen we ook hier misschien ons de luxe van physische adressen veroorloven. (Het is questieus of ons dat geen verdere ellende oplevert. We moeten niet de vergissing van de B5000 maken!)

5. Paginavrijgave.

Zoals d inhaakt op de jongste schakel van de dynamische ketting, zo kunnen we een statusgrootheid e invoeren (eveneens per programma), die inhaakt op de jongste storage function van een groot array; storage functions van grote arrays ketenen we op de gebruikelijke manier.

In geval van blokverlating onderzoekt men, of hierdoor SW onder de heersende waarde van e gezakt is: zo ja, dan zakt men de ketting af, daarbij alle gepasseerde storage functions scannend voor vrij te geven pagina's. Hiermee gaat men door totdat e onder SW gezakt is.

transcribed by Sam Samshuijzen

revised Fri, 6 May 2005

---

## English translation

### On stack administration. II

EWD 70

On stack administration. II

0. Introduction.

I had initially hoped that we would obtain a more or less complete survey of the possibilities that come into consideration and that we would at the same time obtain the criteria with the help of which we could choose between these possibilities. This, however, is not going so smoothly, my patience is running somewhat thin and I should rather like to see, for once, a proposal that at any rate works and is not too crazy. I therefore hope now to attain the deepening of our insight in another way, namely by critical consideration of a concrete proposal. The remainder of this note is devoted to such a proposal.

My first concern is three fundamental problems, to wit:

a) abrupt block shift

b) stack shift

c) page release for so-called large arrays.

In addition to these, a practical problem then also comes up for discussion, namely the optimization of the save and restore procedure in the case of the so-called "simple implicit subroutine".

Concerning the stack shift, the following: apart from the little lists of the reference points (the displays) we need not include in the stack any absolute addresses that refer to points in the stack; these latter we can always store relative to the beginning of the stack bottom. (Whether we shall always do so is another matter: assuming that stack shifts are things that belong to the rarities, we shall perhaps obtain a faster system if, where possible, we work with physical addresses.)

From the above it follows that it is important that in the stack the reference lists can be found; it is here that we make the distinction between a procedure (i.e. a new list) and a complicated implicit subroutine (i.e. falling back on an old list). The simple implicit subroutine we can here, I think, leave out of consideration, since it is after all executed deaf.

I shall now propose stack images to be added upon activation of a procedure, respectively of a complicated parameter.

1. Call of a procedure.

The intention is that the following mechanism is used both for the formal and for the nonformal procedure, both for the type procedure and the non-type procedure. The intention is to follow this mechanism blindly in the case of the type procedure call as a self-contained statement. (The type procedure will deliver its value in the F-register; on return SW will point to the stack top in the sense of Voorhoeve, i.e. to the first free place in core store. Ignoring this result is then quite simple indeed!)

I assume that for the formal places two words suffice to us and that the invariant representation of the return address likewise suffices with two words. (These assumptions are probably not correct, but that is not essential in this connection.) On the following page the proposed stack image is given.

Note 1. The "center point" is thus held by a status variable "d", which in the framework of this introduction gets the value d = d[i]; the value on the call side - here denoted

M[d – 3 – 2 * nf]
M[d – 3 – 2 * nf]
┑

│ formal places of the 1st parameter

┙

⋮

⋮

M[d – 5]
M[d – 4]
┑

│ formal places of the nf-th parameter

┙

M[d – 3]
M[d – 2]
┑

│ invariant return address

┙

M[d – 1]
= nf

M[d]
= d[i – 1]

M[d + 1]
= L[i] (=d + 3)

M[d + 2]
= bn[i]

M[d + 3]
= 0-th reference point

⋮

⋮

with d[i – 1] - is saved here. The index i distinguishes in this description the prevailing activations of procedures and complicated parameters. NB. The index i plays a role only in this description; in the working system it is not kept track of, we thus do not introduce a dynamic depth counter. By "bn[i]" is meant the prevailing block height as introduced in EWD69; L[i] is the prevailing value of L already laid down in the stack.

Note 2. The stacking of "nf" is, strictly speaking, superfluous, but I nevertheless wanted to do it:

a)
it makes a dynamic check on the correctness of the number of parameters passed possible

b)
it makes it - if desired - possible to include (in the library) procedures with a variable number of parameters.

c)
the return mechanism no longer needs to be given by the procedure the datum of how many formal places may be forgotten.

2. Call of a complicated parameter.

This generates at the top of the stack

M[d – 3]
M[d – 2]
┑

│ invariant return address

┙

M[d – 1]
= negative value-address indication

M[d]
= d[i – 1]

M[d + 1]
= L[i] (= L[j] with j < i)

This stack image corresponds to the call of the implicit subroutine. The implicit subroutine is simpler than the full-fledged procedure inasmuch as no parameters are passed to it; it is more complicated because (namely if the actual parameter is an indexed variable) its call can also occur as the "left-hand side" of the assignment statement.

For this last complication there are several solutions. The stack image given above corresponds to the technique in which the object program of the body makes its need known with the help of "Take formal address" or "Take formal value". Which of these two calls is given must then be stored in the stack.

An alternative is that the object program merely says "Go formal" and that the implicit subroutine delivers, besides its primitive result, the indication "address" respectively "value". The object program can then insert on the call side of the implicit subroutine

a) as left-hand side "if value alarm"

b) as right-hand side "if address, take value"

A third possibility is to always deliver "value" and "address" and to select the desired one on the call side. Here, however, you always do too much. (This is moreover nasty if the value is still undefined; I should not like to be punished by a parity error.)

The images mentioned above are suggested on the assumption that the sign of M[d – 1] would always be descriptive of the situation procedure respectively implicit subroutine. (This implies that a procedure without parameters must be given nf = +0 from the call side; the distinction between "value" respectively "address" can then be made by –0 and –1.)

In both cases we could do without the indication in M[d – 1]; the distinction between the two stack images we can also make by testing whether a new reference list has been introduced, i.e. whether M[d + 1] = d + 3 or not.

3. Simple implicit subroutine.

In this case only the return address need be included in the stack. The distinction between value and address does not exist, it is namely always value. (The single variable identifier as actual parameter gives, namely, no occasion to an implicit subroutine; it must be a genuine expression. Here the fact that the indexing in the actual parameter makes the implicit subroutine complicated by definition is a piece of luck!)

4. Stack shift.

During the simple implicit subroutine no interruption can occur and thus also no need for shift can arise. In all other cases the prevailing value of the status variable "d" gives us the clue how we are to proceed further. The reference lists we can find unfailingly and thus we can update the shift. The stacked L-values and the chained d-values we can likewise find unfailingly and here too we can afford ourselves the luxury of physical addresses. Thanks to the convention of the stacked nf we can also find all formal places unambiguously. If we choose a good layout for this, we can perhaps here too afford ourselves the luxury of physical addresses. (It is questionable whether that does not yield us further misery. We must not make the mistake of the B5000!)

5. Page release.

Just as d hooks onto the youngest link of the dynamic chain, so we can introduce a status quantity e (likewise per program) that hooks onto the youngest storage function of a large array; storage functions of large arrays we chain in the usual manner.

In the case of block exit one examines whether SW has hereby sunk below the prevailing value of e: if so, then one descends the chain, scanning thereby all passed storage functions for pages to be released. With this one continues until e has sunk below SW.

transcribed by Sam Samshuijzen

revised Fri, 6 May 2005
