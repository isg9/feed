---
title: "Multiprogrammering en de X8 (EWD51)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD00xx/EWD51.html
published: "1963-05-12T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd00xx/EWD51.PDF
---

# Multiprogrammering en de X8

EWD 51

MULTIPROGAMMERING EN DE X8

DIRECTE TOEGANG TOT DELEN EN SECTIES  -  DIRECT ACCESS TO INSTALMENTS AND SECTIONS

DEEL 1: EWD51

0
1
1.1
1.2
1.3
2
2.1
2.2
2.3

DEEL 2: EWD54

2.4
3
3.1
3.2

DEEL 3: EWD57

3.3
4
4.0
4.1
4.2
4.3
4.4

DUTCH-ENGLISH DICTIONARY OF PRINCIPAL KEYWORDS

Seinpaal
Semaphore

Verhogen (V)
To raise, To increase

Prolagen (P),

a neologism coming from
To try and lower,

Probeer te verlagen
To try and decrease

Ingreep
Interrupt, Interruption

Multiprogrammering en de X8
X8 Nr. 17.

0. Inleiding.

Om te beginnen beschouwen wij een aantal "zwak gekoppelde", onderling
asynchrone, in zich zelf sequenti�le processen. Zonder schade te doen
aan de algemeenheid kunnen we ons tot cyclische processen
beperken.

De zwakke koppeling zal daaruit bestaan dat deze processen informatie
met elkaar kunnen uitwisselen, dwz. toegang hebben tot enig
gemeenschappelijk geheugen. Wanneer deze processen samen van eenzelfde
faciliteit bij toerbeurt gebruik zouden willen maken, of wanneer het
ene proces informatie wil consumeren, die door een ander proces
geproduceerd moet worden, dan volgen hieruit zekere
synchronisatie-eisen. We moeten zorgen, dat de processen elkaar niet
storen of, in het geval van informatieuitwisseling, niet te ver ten
opzichte van elkaar uit de pas geraken.

In het eerste gedeelte wordt in de opzet volledig in het midden
gelaten, door welke apparatuur deze onderscheiden processen
gerealiseerd worden. De uiteindelijke bedoeling is, om in aansluiting
met de gegeven - en ik moet er aan toevoegen: concipi�erbare -
werkelijkheid sommige van deze cyclische processen door een specifiek
stuk hardware te laten uitvoeren, terwijl de overige processen door de
centrale computer verzorgd zullen worden. In het eerste gedeelte is dit
onderscheid niet van belang; wel belangrijk is, dat een "cyclisch
proces" een geprogrammeerde conceptie kan zijn en dat het aantal
cyclische processen, dat op elk gegeven moment in het spel betrokken
is, in de tijd kan vari�ren.

In 1. zal ik in de plaats van "cyclisch proces" spreken van "machine", ter onderstreping van het feit, dat we ons deze processen als grotendeels (of volslagen) autonoom moeten voorstellen. In het licht van het vorige moeten we er ons dus niet over verbazen, als er een paar machines bijgeschapen worden of er een paar machines geannihileerd worden. Later, als we dichter bij de realisatie met behulp van beperkte hardware komen, dan zullen we onderscheid maken tussen "concrete machines" - dwz. cyclische processen, waaraan een specifiek stuk hardware beantwoordt zoals bv. een communicatieapparaat - en "abstracte machines", die tezamen door de centrale computer gesimuleerd worden. Voor deze simulator-generaal zal ik het woord computer reserveren. Deze simulatie roept, met het oog op efficienter gebruik van de totale installatie, strategieproblemen op, waarvan ik me op dit ogenblik met nadruk distanciëer. Voordat ik nl. het strategieprobleem überhaupt wil bekijken, dwz. voordat ik er ook maar over denken wil, hoe de computer zijn vrijheid zou kunnen uitbuiten, wil ik eerst zijn "onvrijheid" geformaliseerd zien, wil ik een passend formalisme hebben, waarin de logisch noodzakelijke "interlocks" tussen de diverse machines geformuleerd kunnen worden.

In 2. zullen we een begin maken met de analyse van de simulatie door één computer van de verzamelde abstracte machines. Hier zien wij overigens heel duidelijk, dat over de onderlinge snelheidsverhoudingen der verschillende machines geen veronderstellingen gemaakt mogen worden: als een van de abstracte machines werkt, staan de andere stil (werken de andere oneindig langzaam).

1. Seinpalen

De onderlinge temporele restricties worden vastgelegd in zg.
"seinpalen". Een seinpaal is een integer, die voor minstens twee
verschillende machines toegankelijk is; de mogelijkheden, waarop iedere
indiviuele machine met de seinpalen kan communiceren, is echter beperkt
tot een tweetal, de operaties P en V.

1.1. De operatie V ("Verhoog").

De operatie V kan betrekking hebben op een willekeurig aantal
verschillende seinpalen, dus bv. "V (S1, S2, S3)". Als deze operatie in
een van de machines voorkomt - in ons voorbeeld moeten dan S1, S2, S3
voor deze machine toegankelijke seinpalen zijn - dan is het effect dat
alle opgegeven seinpalen in ��n ondeelbare handeling met 1 verhoogd worden.

1.2. De operatie P ("Prolaag").

De operatie P kan betrekking hebben op een willekeurig aantal
seinpalen, dus bv. "P (S1, S2, S3)". Als deze operatie in een van de
machines voorkomt - in ons voorbeeld moeten dan S1, S2 en S3 voor deze
machine toegankelijk zijn - dan begint een operatie die slechts
beeindigd kan worden op een moment, dat alle opgegeven seinpalen
positief zijn. Beeindiging van een P-operatie impliceert, dat alle
opgegeven seinpalen met 1 verlaagd worden en deze beeindiging geldt als
��n ondeelbare handeling. Ook voor de operatie P beperken we ons tot het geval, dat de erbij opgegeven seinpalen verschillend zijn.

1.3. Enige voorbeelden van het gebruik van seinpalen.

Opm. Vele seinpalen nemen slechts de waarden 0 en 1 aan. In dat geval
de operatie V als "baanvak vrijgeven"; de operatie P - een tentatieve
passering - kan slechts voltooid worden, als de betrokken seinpaal (of
seinpalen) op veilig staat (staan) en passering impliceert dan een op
onveilig zetten. De terminologie is ten duidelijkste ontleend aan het
spoorwegwezen - althans aan mijn voorstelling daarvan.

1.3.1.
Als wij een klasje machines Xi hebben (X0, X1, X2, ...) en in elke
machine komt een critische sectie TXi voor, critisch in die zin, dat
geen twee critische secties tegelijkertijd onder bewerking mogen zijn,
dan kunnen we dit bereiken met een seinpaal, zeg SX, die in dit simpele
geval tweewaardig zal zijn:

SX = 0 zal betekenen: een van de machines is aan zijn critische sectie bezig.

SX = 1 zal betekenen: geen van de machines is aan zijn critische sectie bezig.

De omschrijving van alle machines is gelijkluidend:

"LXi: P(SX); TXi, V(SX); rest proces Xi; goto LXi".

Als we alle machines op hun gelabelde punt ("aan het begin van de
regel") zouden starten met de beginwaarde SX = 2, dan zouden we
verwezenlijkt hebben, dat ongeacht de omvang van het klasje machines Xi
er nooit meer dan twee simultaan aan hun critische secties bezig zouden
zijn. Dir is kennelijk een generalisatie van de probleemstelling van
wederzijdse uitsluiting. (Het is precies de situatie van bv. n tape
decks aan twee kanalen.)

Wij vestigen er de aandacht op, dat de formulering der individuele
machines Xi onafhankelijk is van de omvang van het klasje machines Xi,
iets wat met het oog op de dynamische variatie van deze omvang wel
hoogst gewenst is.

1.3.2.
Nu beschouwen we een groepje machines Xi en een groepje machines Yi,
elk met hun critische sectie TXi resp. TYj. Uitvoering van een der
critische secties dient uitvoering van alle andere critische secties
uit te sluiten, maar tevens eisen we, dat de uitvoering van een
TX-sectie en een TY-sectie alternerende gebeurtenissen zijn.

We kunnen dit bereiken met twee tweewaardige seinpalen, zeg SX en Sy.

SX = 1 betekent dat nu eerst een TX-sectie aan de beurt is.

SY = 1 betekent dat nu eerst een TY-sectie aan de beurt is.

De programma's voor de machines luiden:

"LXi: P(SX); TXi; V(SY); proces Xi; goto LXi" en

"LYj: P(SY); TYj; V(SX); proces Yj; goto LYj".

Als de machines alle op het begin van de regel gestart worden moet SX = 1 en SY = 0 zijn of andersom.

Opm.
Hadden we ook nog een groepje machines Zk gehad en was de opgave
geweest, dat de uitvoering van een TX-sectie, een TY-sectie en een
TZ-sectie elkaar in die volgorde moesten opvolgen, dan haden de
programma's geluid:

"LXi: P(SX); TXi; V(SY); proces Xi; goto LXi"

"LYj: P(SY); TYj; V(SZ); proces Yj; goto LYj"

"LZk: P(SZ); TZk; V(SX); proces Zk; goto LZk".

Wij vestigen er de aandacht op, dat in deze uitbreiding de tekst van de
machines Xi niet is veranderd.

1.3.3.
Nu beschouwen we een klasje machines Xi, die informatieeenheden willen
lozen in een cyclische buffer meet een capaciteit van N
informatieeenheden; voorst een klasje machines Yj, die
informatieeenheden uit deze buffer willen verwerken.

Omdat vullen van de buffer administratieve maatregelen met de
"vulwijzer" impliceert - en voor legen mutatis mutandis hetzelfde geldt
- eisen we bovendien, dat vullen slechts door 1 machine Xi
tegelijkertijd kan geschieden en evenzo, dat legen slechts door ��n
machine Yj tegelijkertijd kan geschieden.

We voeren hiervoor in vier seinpalen:

SX1 = aantal vrije plaatsen in de buffer (aanvankelijk = N)

SX2 = 0 als ��n van de machines Xi vult, anders 1

SY1 = aantal gevulde plaatsen in de buffer (aanvankelijk = 0)

SY2 = 0 als ��n van de machines Yj leegt, anders 1.

Er zal gelden N - 1 <= SX1 + SY1 <= N. De machines zijn nu:

"LXi: P (SX1, SX2); vul volgende plaats van de buffer; V (SY1, SX2); proces Xi; goto LXi"

"LYj: P (SY1, SY2); leeg volgende plaats van de buffer; V (SX1, SY2); proces Yj; goto Yj"

In het voorafgaande is het verschil tussen abstracte en concrete
machines helemaal op de achtergrond gebleven, juist om te laten zien,
dat ze, van een zeker macroscopisch standpunt bezien, volslagen
gelijkwaardig zijn. Wij gaan dit onderscheid nu wel maken om te
analyseren met wat voor soort hardware dit gerealiseerd zou kunnen
worden.

2. Onderscheid tussen abstracte en concrete machines.

Een abstracte machine is een of ander geprogrammeerd sequenti�el
proces, een concrete machine is een transputapparaat. (Transput is
input of output.)

Wij interesseren ons vanzelfsprekend speciaal voor het bespelen van
seinpalen die de synchronisatie tussen de computer - dwz. het aggregaat
van abstracte machines - en een transputmachine anderzijds
regelen.

Voor elke seinpaal van dit type is een vaste geheugenplaats
gereserveerd. Deze toevoeging komt op twee manieren tot uiting:
enerzijds in de manier, waarop het transputapparaat is aangesloten,
anderzijds in het programma, dat de samenwerking met dit
transputapparaat verzorgt. Beide processen moeten immers tot deze
gemeenschappelijke seinpaal toegang hebben.

Wij beperken ons nu tot twee soorten seinpalen, nl. de seinpaal, waarop
de P-operatie door de computer en de V-operatie door de transputmachine
wordt uitgevoerd en de seinpaal, waarbij dit juist andersom is.

De afbeelding van de seinpaal in een geheugenplaats ligt heel erg voor
de hand: in het algemeen is het een integer, die onthouden moet worden.

Als
de seinpaal in het geheugen staat, zijn er vanzelfsprekende opdrachten,
waarmee de computer toegang tot de seinpaal kan krijgen. Bovendien zou
de transputmachine - als alle - in elk geval al autonoom toegang hebben
tot het geheugen. Het basismechanisme voor de seinpaalbespeling is er
dus helemaal. Een verder voordeel van een geheugenplaats is de
vanzelfsprekende mogelijkheid tot tot uitbreiding van het aantal
transputmachines (en bijbehorende seinpalen) gegeven. Er is echter 1
complicatie, waarbij we wel even stil mogen staan: van de waarde van
een seinpaal - preciezer: van het positief zijn of niet - moet een
sturende invloed uitgaan. We kunnen dus niet volstaan met de seinpaal
op kernen op te bergen [en] verder niets meer. Aan al deze seinpalen
wordt daarom een flip-flop toegevoegd, die een derivaat van de seinpaal
zal zijn, nl. = 1 als de seinpaal positief is en anders = 0. (Zolang we
ons beperken tot de non-negatieve seinpalen is de enige andere
mogelijkheid = 0. We zullen omdat het hier hardware faciliteiten
betreft ons geen nodeloze beperkingen opleggen en in onze definities
steeds ook negatieve seinpalen toelaten, dwz. we voeren teken-tests uit
en geen nul-tests.)

Een dergelijke toegevoegde flip-flop lost dan wel het probleem op hoe
we aan een seinpaal op kernen een sturende invloed kunnen geven, het
confronteert ons echter wel met de taak te zorgen, dat als de seinpaal
in het geheugen gewijzigd wordt, deze flip-flop zonodig aan de nieuwe
toestand wordt aangepast.

Hiervoor zijn twee mogelijkheden

(a) we bevorderen de combinatie van seinaalwijziging ene flip-flop-aanpassing tot de status van ondeelbare handeling.

(b)
we gaan bij de separate aanpassing van de flip-flop aan de nieuwe
seinpaalwaarde zo omzichtig te werk, dat tijdelijke discrepantie geen
brokken kan opleveren.

Waar het transputapparaat een seinpaal wijzigt kiezen we de eerste
methode, waar de computer een seinpaal wijzigt kiezen we de tweede
methode. De eerste methode is voor de transputmachine niet moeilijk te
verwezenlijken, want de transputmachine heeft immers de mogelijkheid om
het geheugen op te eisen - althans met een grotere prioriteit dan de
computer. Voor de machine is de eerste oplossing minder leuk, want de
computer kon al bij de seinpalen via normale opdrachten. De automatisch
aanpassing van de flip-flop zou dus betekenen, dat er op de adresbits
een extra uitcodering zou moeten komen; dit si niet zoerg aantrekkelijk
als we bedenken, dat dit aantal seinpalen toeneemt als er meer
apparatuur wordt aangesloten. Verkozen is daarom de methode, waarbijde
machine met aparte - matig geadresseerde - opdrachten op elk van de
individuele toegevoegde flip-flops kan opereren.

2.1. De P-operatie door de transput-machine.

Zij t de integer in het kerngeheugen, die de seinpaal representeert,
Zij T de bijbehorende flip-flop, die 1 is, als 0 < t is. T fungeert
dan als "Actie-flip-flop" voor de communicatiemachine, die op een
passend ogenblik in zijn cyclus zal uitvoeren:

"if T = 1 then begin t := t - 1; if t <= 0 then T := 0 end".

De
P-operatie door de transputmachine impliceert bij voltooiing het
accepteren van een startopdracht. (Over het wachten als T = 0 laat ik
me hier niet uit, bv. of het apparaat werkelijk stilstaat of een dode
slag maakt, want dat is in dit verband onbelangrijk. Essenti�el is, dat
de transputmachine bij T = 0 de seinpaal t en de flip-flop T ongemoeid
zal laten.)

Deze P-operatie - onderdeel van het consumeren van de volgende
startopdracht - heeft als mogelijk gevolg, dat het opnemen van de
volgende startopdracht via de assignment T := 0 verder wordt
tegengehouden, nl. als de voorraad is uitgeput.

Additionele spelregels zijn:

(a)
dat deze handeling vanuit de computer bezien moet worden als ondeelbare
handeling - dit schept zijn voordelen - maar dat anderzijds de computer
niet over de mogelijkheid zal beschikken om deze handeling over een
aantal opdrachten te verbieden - en dit schept zijn problemen. Dit
laatste vloeit voort uit het feit, dat de transputmachine bij autonoom
geheugencontact absolute prioriteit boven de machine heeft (in
overeenstemming met het feit, dat je op dit microscopische niveau
essenti�le haastsituaties moet kunnen toestaan).

(b)
dat bij beinvloeding van t en T door de computer het pertinent verboden
is als, hoe kort maar ook, ten onrechte T = 1 zou gelden. Op dat moment
zou nl. meteen het transputapparaat ten onrechte gestart kunnen
worden.

Dit betekent, dat de computer de V-operatie niet mag uitvoeren door het programma

"t := t + 1; C := 0 < t;

if C then T := 1"

(Opm.
De algol-programma'tjes, waarmee we de actie van de machine beschrijven
hebben als neven-eigenschap, dat met elke regel 1 X8-opdracht
overeenkomt; dit maakt het duidelijker om te zien, waar het
transputapparaat - tussen de opdrachten nl. - er tussen door kan komen.
Men lette erop, dat de telling aan t met een (conditiezettende)
additieve opdracht wordt uitgevoerd; daarmee is althans de telling een
ondeelbare handeling en kunnen de tellende operaties van computer en
transputmachine niet met elkaar interfereren.)

Als het bovenstaande programma wordt uitgevoerd op een moment, dat T =
1 is, bestaat er immers de kans, dat na de test op het teken van de
nieuwe waarde van t en voor de assignment T := 1 het transputapparaat
zoveel P-operaties uitvoert, dat t tot nul wordt afgelaagd en in
overeenstemming daarmee T := 0 verricht wordt, waarna de computer op
grond van een inmiddels obsoleet gegeven alsnog T := 1 uitvoert.

De weg uit deze ellende is

"t := t + 1;

if T = 0 then

begin if 0 < t

then T := 1 end"

De rechtvaardiging van dit stukje programma is nu als volgt. Eerst
testen we T; als T = 1 is, dan was 0 < t; door de optelling is t
alleen maar groter geworden en het eventueel aanpassen van T ligt nu op
de weg van de transputmachine. Maar als we T = 0 aantreffen, dan kan de
transputmachine de operatie P niet meer uitvoeren, zodat van die kant t
en T ongemoeid gelaten worden en de computer rustig de test kan
uitvoeren.

In de praktijk van de X8 zal het nog een stapje ingewikkelder
uitgevoerd worden. Een dergelijke "Actie-flip-flop" als T zal zich, om
de communicatielijn niet nodeloos te belasten niet in de basismachine,
maar in de transputmachine in kwestie bevinden. Dit maakt de
vraagopdracht "T = 0" een minder aantrekkelijke propositie. Daarom
wordt in het geheugen een copie van T bijgehouden, ST genaamd.

De P-handeling door het transputapparaat wordt nu:

"if T = 1 then begin t := t - 1; if t < 0 then ST := T := 0 end"

en de V-handeling door de computer wordt

"t := t + 1;

if ST = 0 then

begin if 0 < t then

begin ST := 1;

T
:= 1

end

end"

waarbij we erop moeten letten, dat de assignment aan ST voor de assignment aan T wordt uitgevoerd.

Om het aantal geheugencontacten van de transputmachine te beperken is
het gewenst om ST en t in hetzelfde woord te representeren, maar dan
zo, dat de operatie t := t + 1 ST niet wijzigt. Dit kan bv. door het
woord in te delen:

d[26] = ST, d[25]...d[0] = (2^25 - 1) + t

De test "t <= 0" komt dan neer op de test "d[25] = 0".

2.2. De V-operatie door de transputmachine.

Weer
zullen we de seinpaal aanduiden met t en de bijbehorende flip-flop met
T en zal gelden T = 1 als 0 < t. De V-opdracht is nu eenvoudig

"t := t + 1; if 0 < t then T := 1"

door het transput-mechanisme uit te voeren als een ondeelbare, door de computer niet tegen te houden handeling.

De P-operatie zal de computer alleen uitvoeren als T = 1; de computer
kan zich, bij gratie van doofheid, echter permitteren, om tijdelijk T =
0 te zetten.

De voltooiing van de P-operatie door de computer bestaat nu uit het volgende stukje programma

"t := t - 1;

T := 0;

if 0 < t then

T := 1"

Essenti�el is hierbij aangenomen

(a) dat het geen kwaad kan, als T ten onrechte een tijdje = 0 is;

(b)
dat als transputmachine en computer "tegelijkertijd" proberen om de
assignment "T := 1" uit te voeren, dat dan na afloop zeker "T = 1" zal
gelden.

Om aan te tonen dat bovenstaand stukje programma na afloop onder alle
omstandigheden de waarde T in overeenstemming met t achterlaat, hoeven
we niet te veronderstellen dat T aanvankelijk positief was. (Als we de
eerste twee statements omdraaien, dan moet ik dat wel aannemen; een
helderder inzicht in dit soort spelletjes is nog steeds zeer gewenst!)
We kunnen desgewenst dit programma ook gebruiken om via "t := t - n"
een zg. negatieve voorgift te geven.

De redenering is als volgt. Elke V-operatie voor de assignment T := 0
uit uit bovenstaand programma'tje doet niet ter zake, omdat daarna in
elk geval T := 0 uitgevoerd wordt. Als tijdens de assignment T := 0 t
positief is, dan zal de computer uiteindelijk de statement T := 1
uitvoeren. Door tussengeschoven operaties V kan t immers dan alleen
maar nog groter worden en T wordt dus correct achtergelaten. Als
tijdens de assignment T := 0 daarentegen t non-positief is, dan moeten
wij twee gevallen onderscheiden. Als ondanks eventuele tussengeschoven
V-operaties t non-positief blijft, dan zullen zowel transputmachine als
computer T verder ongemoeid laten en dus zal T correct = 0
achtergelaten worden.

Als
door ondergeschoven V-operaties daarentegen t positief wordt, dan zal
in elk geval door te transputmachine op het moment van omslag T := 1
uitgevoerd worden (en mogelijk ten overvloede ook nog een keer door de
computer, nl. als t ten tijde van de test al positief is). In dit
laatste geval wordt T dus gegarandeerd correct = 1 achtergelaten.

Na deze analyses van V- resp. P-operatie door de computer vertrouw ik

(a) dat de lezer zich van de waterdichtheid overtuigd heeft

(b) dat de lezer mij de uitvoerigheid, waarmee ik dit heb uitgesponnen, niet kwalijk neemt. Integendeel zelfs.

Als de P-operatie door de machine uitgevoerd moet worden, dan is de bijbehorende flip-flop T een zg. ingreep-flip-flop. Een ingreep-flip-flop zit in de basismachine, kan door de computer in beide richtingen gezet worden, kan door de transputmachine op 1 gezet worden. (Dit in tegenstelling tot een actie-flip-flop, zie 2.1.)

2.3. Hardware ten behoeve van de ingreep-controle.

Ingreep-flip-flops kunnen (zie 2.2.) individueel in beide richtingen gezet worden.

Aan elke ingreep-flip-flop is een luisterbit toegevoegd. Ook deze is
als flip-flop uitgevoerd; ook de luisterbits kunnen individueel in
beide richtingen gezet worden door de computer.

Ingreep-flip-flops, zoewl als luisterbits kunnen woordsgewijs worden
uitgelezen via een communicatie-opdracht. (Zolang het er minder dan 27
per type zijn, dan doen we dit met een woord voor elk type.)
Overeenkomstige flip-flops hebben in deze woorden dezelfde
positie.

Ten slotte is er een doof-horendbit in de computer, die tevens beschikt
over de opdrachten "maak doof" en "maak horend".

Als voldaan is aan de volgende twee voorwaarden:

(a) de machine is horend

(b) het collatie-resultaat van ingreepwoord(en) en (overeenkomstige) luisterwoord(en)  is ≠ 0

dan vindt een ingreep plaats, dwz.

inplaats
van normaal de volgende opdracht aan te halen, wordt een
subroutinesprong in het opdrachtregister geplaatst, die met
onderdrukking van de ophoging van de opdrachtteller wordt uitgevoerd;
tevens wordt de machine door deze actie automatisch doof.

Opm.
Hieruit zou volgen, dat een ingreep alleen maar plaats kan vinden op
een plek in het programma, waar de machine horend is. Als tijdens
de laatste opdracht voor de ingreep al besloten is, dat deze ingreep
plaats zal vinden, dan is het mogelijk moeilijk om op deze decisie
terug te komen, als de laatste opdracht nu juist de opdracht "maak
doof" was. De oplossing is om bij het wegbergen van de link ook de
"doof-horendheid" te redden en bij de voortzetting van het onderbroken
programma "herstellend" terug te springen. Bij de X1 is gebleken, dat
"vertraagde actie" van de opdrachten "maak doof" en "maak horend" mits
deze vertraging zich slechts over 1 opdracht uitstrekt, geen enkele
moeilijkheid oplevert.

Het programma'tje voor de aflaging van T in 2.2. zal in doofheid geschieden; vandaar dat T vrijelijk even een verkeerde waarde mag hebben.

De luisterbits stellen ons in staat ingreep-flip-flops - eventueel
tijdelijk - te negeren, wanneer er nl. - nog - geen belangstelling voor
bestaat.

transcribed by Johan E.

Mebius

revised Thu, 29 Mar 2007

---

## English translation

### Multiprogramming and the X8

EWD 51

MULTIPROGRAMMING AND THE X8

DIRECTE TOEGANG TOT DELEN EN SECTIES  -  DIRECT ACCESS TO INSTALMENTS AND SECTIONS

PART 1: EWD51

0
1
1.1
1.2
1.3
2
2.1
2.2
2.3

PART 2: EWD54

2.4
3
3.1
3.2

PART 3: EWD57

3.3
4
4.0
4.1
4.2
4.3
4.4

DUTCH-ENGLISH DICTIONARY OF PRINCIPAL KEYWORDS

Seinpaal
Semaphore

Verhogen (V)
To raise, To increase

Prolagen (P),

a neologism coming from
To try and lower,

Probeer te verlagen
To try and decrease

Ingreep
Interrupt, Interruption

Multiprogramming and the X8
X8 Nr. 17.

0. Introduction.

To begin with, we consider a number of "weakly coupled", mutually
asynchronous, intrinsically sequential processes. Without doing harm
to the generality, we may restrict ourselves to cyclic processes.

The weak coupling will consist in the fact that these processes can
exchange information with one another, i.e. have access to some
common store. When these processes would together want to make use of
one and the same facility in turn, or when the one process wants to
consume information that has to be produced by another process, then
certain synchronization requirements follow from this. We must see to
it that the processes do not disturb one another or, in the case of
information exchange, do not get too far out of step with respect to
one another.

In the first part it is, in the conception, left entirely open by what
apparatus these distinct processes are realized. The eventual
intention is, in connection with the given - and I must add: the
conceivable - reality, to have some of these cyclic processes carried
out by a specific piece of hardware, while the remaining processes will
be taken care of by the central computer. In the first part this
distinction is of no importance; what is important, however, is that a
"cyclic process" can be a programmed conception and that the number of
cyclic processes that is involved in the game at each given moment can
vary over time.

In 1. I shall, in place of "cyclic process", speak of "machine", in order to underline the fact that we must conceive of these processes as largely (or wholly) autonomous. In the light of the foregoing we must therefore not be surprised if a couple of machines are created additionally or a couple of machines are annihilated. Later, when we come closer to the realization by means of limited hardware, then we shall distinguish between "concrete machines" - i.e. cyclic processes to which a specific piece of hardware corresponds, such as e.g. a communication apparatus - and "abstract machines", which together are simulated by the central computer. For this simulator-general I shall reserve the word computer. This simulation gives rise, with a view to a more efficient use of the total installation, to strategy problems, from which at this moment I emphatically dissociate myself. For before I want to consider the strategy problem at all, i.e. before I even want to give a thought to how the computer might exploit its freedom, I first want to see its "unfreedom" formalized, I want to have a suitable formalism in which the logically necessary "interlocks" between the various machines can be formulated.

In 2. we shall make a beginning with the analysis of the simulation by one computer of the assembled abstract machines. Here, by the way, we see very clearly that no assumptions may be made about the mutual speed ratios of the various machines: when one of the abstract machines is working, the others stand still (the others work infinitely slowly).

1. Semaphores

The mutual temporal restrictions are laid down in so-called
"semaphores". A semaphore is an integer that is accessible to at least
two different machines; the possibilities by which each individual
machine can communicate with the semaphores are, however, restricted
to a pair, the operations P and V.

1.1. The operation V ("Raise").

The operation V may pertain to an arbitrary number of different
semaphores, thus e.g. "V (S1, S2, S3)". When this operation occurs in
one of the machines - in our example S1, S2, S3 must then be semaphores
accessible to this machine - then the effect is that all specified
semaphores are increased by 1 in one indivisible action.

1.2. The operation P ("Prolow").

The operation P may pertain to an arbitrary number of semaphores, thus
e.g. "P (S1, S2, S3)". When this operation occurs in one of the
machines - in our example S1, S2 and S3 must then be accessible to this
machine - then an operation begins which can only be terminated at a
moment at which all specified semaphores are positive. Termination of a
P-operation implies that all specified semaphores are decreased by 1,
and this termination counts as one indivisible action. For the
operation P too we restrict ourselves to the case in which the
semaphores specified with it are distinct.

1.3. Some examples of the use of semaphores.

Rem. Many semaphores take only the values 0 and 1. In that case
the operation V [acts] as "release block section"; the operation P - a
tentative passage - can only be completed if the semaphore (or
semaphores) concerned stands (stand) at safe, and passage then implies a
setting to unsafe. The terminology is most clearly borrowed from the
railway business - at least from my conception thereof.

1.3.1.
If we have a little class of machines Xi (X0, X1, X2, ...) and in each
machine there occurs a critical section TXi, critical in the sense that
no two critical sections may be under processing simultaneously, then
we can achieve this with a semaphore, say SX, which in this simple
case will be two-valued:

SX = 0 will mean: one of the machines is engaged on its critical section.

SX = 1 will mean: none of the machines is engaged on its critical section.

The description of all machines is identically worded:

"LXi: P(SX); TXi, V(SX); rest of process Xi; goto LXi".

If we were to start all machines at their labelled point ("at the
beginning of the line") with the initial value SX = 2, then we would
have brought about that, regardless of the size of the little class of
machines Xi, there would never be more than two engaged on their
critical sections simultaneously. This is evidently a generalization of
the problem statement of mutual exclusion. (It is precisely the
situation of e.g. n tape decks on two channels.)

We draw attention to the fact that the formulation of the individual
machines Xi is independent of the size of the little class of machines
Xi, something which, with a view to the dynamic variation of this size,
is indeed most highly desirable.

1.3.2.
Now we consider a little group of machines Xi and a little group of
machines Yi, each with their critical section TXi resp. TYj. Execution
of one of the critical sections must exclude execution of all other
critical sections, but at the same time we require that the execution
of a TX-section and a TY-section be alternating events.

We can achieve this with two two-valued semaphores, say SX and Sy.

SX = 1 means that now first a TX-section has its turn.

SY = 1 means that now first a TY-section has its turn.

The programs for the machines read:

"LXi: P(SX); TXi; V(SY); process Xi; goto LXi" and

"LYj: P(SY); TYj; V(SX); process Yj; goto LYj".

If the machines are all started at the beginning of the line, then SX = 1 and SY = 0 must hold, or the other way around.

Rem.
Had we also had a little group of machines Zk and had the task been
that the execution of a TX-section, a TY-section and a TZ-section had
to follow one another in that order, then the programs would have read:

"LXi: P(SX); TXi; V(SY); process Xi; goto LXi"

"LYj: P(SY); TYj; V(SZ); process Yj; goto LYj"

"LZk: P(SZ); TZk; V(SX); process Zk; goto LZk".

We draw attention to the fact that in this extension the text of the
machines Xi has not changed.

1.3.3.
Now we consider a little class of machines Xi, which want to deposit
information units into a cyclic buffer with a capacity of N
information units; furthermore a little class of machines Yj, which
want to process information units out of this buffer.

Because filling the buffer implies administrative measures with the
"fill pointer" - and for emptying the same holds mutatis mutandis -
we additionally require that filling can only take place by 1 machine
Xi at a time, and likewise that emptying can only take place by one
machine Yj at a time.

For this we introduce four semaphores:

SX1 = number of free places in the buffer (initially = N)

SX2 = 0 if one of the machines Xi is filling, otherwise 1

SY1 = number of filled places in the buffer (initially = 0)

SY2 = 0 if one of the machines Yj is emptying, otherwise 1.

It will hold that N - 1 <= SX1 + SY1 <= N. The machines are now:

"LXi: P (SX1, SX2); fill next place of the buffer; V (SY1, SX2); process Xi; goto LXi"

"LYj: P (SY1, SY2); empty next place of the buffer; V (SX1, SY2); process Yj; goto Yj"

In the foregoing the difference between abstract and concrete machines
has remained entirely in the background, precisely in order to show
that they, viewed from a certain macroscopic standpoint, are wholly
equivalent. We are now going to make this distinction in order to
analyse with what kind of hardware this could be realized.

2. Distinction between abstract and concrete machines.

An abstract machine is some programmed sequential process or other, a
concrete machine is a transput apparatus. (Transput is input or
output.)

We are, of course, especially interested in the playing-upon of
semaphores that regulate the synchronization between the computer -
i.e. the aggregate of abstract machines - and a transput machine on the
other hand.

For each semaphore of this type a fixed store location is reserved.
This addition manifests itself in two ways: on the one hand in the
manner in which the transput apparatus is connected, on the other hand
in the program that takes care of the cooperation with this transput
apparatus. After all, both processes must have access to this common
semaphore.

We now restrict ourselves to two sorts of semaphores, namely the
semaphore upon which the P-operation is carried out by the computer and
the V-operation by the transput machine, and the semaphore in which
this is precisely the other way around.

The mapping of the semaphore into a store location is very obvious: in
general it is an integer that has to be remembered.

When
the semaphore stands in the store, there are obvious instructions by
means of which the computer can gain access to the semaphore. Moreover
the transput machine - like all [of them] - would in any case already
have autonomous access to the store. The basic mechanism for the
playing-upon of the semaphore is thus entirely present. A further
advantage of a store location is the obvious possibility for extension
of the number of transput machines (and associated semaphores) given.
There is, however, 1 complication on which we may well dwell for a
moment: from the value of a semaphore - more precisely: from its being
positive or not - a governing influence must emanate. We thus cannot be
content with storing the semaphore on cores [and] nothing further. To
all these semaphores there is therefore added a flip-flop, which will
be a derivative of the semaphore, namely = 1 if the semaphore is
positive and otherwise = 0. (As long as we restrict ourselves to the
non-negative semaphores the only other possibility is = 0. Because what
is here at issue concerns hardware facilities, we shall impose no
needless restrictions on ourselves and in our definitions always also
admit negative semaphores, i.e. we carry out sign-tests and not
zero-tests.)

Such an added flip-flop does indeed solve the problem of how we can
give a governing influence to a semaphore on cores, it does, however,
confront us with the task of seeing to it that, when the semaphore in
the store is changed, this flip-flop is, if necessary, adjusted to the
new state.

For this there are two possibilities

(a) we promote the combination of semaphore change and flip-flop adjustment to the status of indivisible action.

(b)
in the separate adjustment of the flip-flop to the new semaphore value
we proceed so circumspectly that a temporary discrepancy can do no
damage.

Where the transput apparatus changes a semaphore we choose the first
method, where the computer changes a semaphore we choose the second
method. The first method is not difficult for the transput machine to
realize, for the transput machine has, after all, the possibility of
claiming the store - at least with a greater priority than the
computer. For the machine the first solution is less pleasant, for the
computer could already get at the semaphores via normal instructions.
The automatic adjustment of the flip-flop would thus mean that on the
address bits an extra decoding would have to come; this is not so very
attractive when we consider that this number of semaphores increases as
more apparatus is connected. The chosen method is therefore the one in
which the machine can operate, with separate - moderately addressed -
instructions, on each of the individual added flip-flops.

2.1. The P-operation by the transput-machine.

Let t be the integer in the core store that represents the semaphore,
let T be the associated flip-flop, which is 1 if 0 < t. T then
functions as "Action-flip-flop" for the communication machine, which at
a suitable moment in its cycle will carry out:

"if T = 1 then begin t := t - 1; if t <= 0 then T := 0 end".

The
P-operation by the transput machine implies, on completion, the
acceptance of a start instruction. (About the waiting when T = 0 I do
not express myself here, e.g. whether the apparatus really stands still
or makes a dead stroke, for that is in this connection unimportant. What
is essential is that the transput machine, when T = 0, will leave the
semaphore t and the flip-flop T undisturbed.)

This P-operation - part of the consuming of the next start instruction -
has as a possible consequence that the taking-up of the next start
instruction is further held back via the assignment T := 0, namely when
the supply is exhausted.

Additional rules of the game are:

(a)
that this action, viewed from the computer, must be regarded as an
indivisible action - this creates its advantages - but that on the
other hand the computer will not have the possibility of prohibiting
this action over a number of instructions - and this creates its
problems. This latter follows from the fact that the transput machine,
in autonomous store contact, has absolute priority over the machine (in
accordance with the fact that at this microscopic level one must be able
to allow essential rush situations).

(b)
that, when t and T are influenced by the computer, it is pertinently
forbidden that, however briefly, T = 1 should wrongly hold. For at that
moment the transput apparatus could immediately be started wrongly.

This means that the computer may not carry out the V-operation by means of the program

"t := t + 1; C := 0 < t;

if C then T := 1"

(Rem.
The little algol programs by which we describe the action of the
machine have as a side-property that to each line 1 X8-instruction
corresponds; this makes it clearer to see where the transput apparatus -
namely between the instructions - can come in between. One should note
that the counting on t is carried out with a (condition-setting)
additive instruction; thereby the counting at least is an indivisible
action and the counting operations of computer and transput machine
cannot interfere with one another.)

When the above program is carried out at a moment at which T = 1, there
exists, after all, the chance that, after the test on the sign of the
new value of t and before the assignment T := 1, the transput apparatus
carries out so many P-operations that t is lowered down to zero and, in
accordance therewith, T := 0 is performed, after which the computer, on
the basis of a meanwhile obsolete datum, nevertheless carries out T :=
1.

The way out of this misery is

"t := t + 1;

if T = 0 then

begin if 0 < t

then T := 1 end"

The justification of this little piece of program is now as follows.
First we test T; if T = 1, then 0 < t held; through the addition t has
only become larger, and the eventual adjustment of T now lies on the
path of the transput machine. But if we encounter T = 0, then the
transput machine can no longer carry out the operation P, so that from
that side t and T are left undisturbed and the computer can calmly carry
out the test.

In the practice of the X8 it will be carried out a little step more
complicatedly. Such an "Action-flip-flop" as T will, in order not to
load the communication line needlessly, be located not in the basic
machine but in the transput machine in question. This makes the
interrogation instruction "T = 0" a less attractive proposition.
Therefore in the store a copy of T is kept, called ST.

The P-action by the transput apparatus now becomes:

"if T = 1 then begin t := t - 1; if t < 0 then ST := T := 0 end"

and the V-action by the computer becomes

"t := t + 1;

if ST = 0 then

begin if 0 < t then

begin ST := 1;

T
:= 1

end

end"

whereby we must take care that the assignment to ST is carried out before the assignment to T.

In order to limit the number of store contacts of the transput machine,
it is desirable to represent ST and t in the same word, but then in
such a way that the operation t := t + 1 does not change ST. This can be
done, e.g., by partitioning the word:

d[26] = ST, d[25]...d[0] = (2^25 - 1) + t

The test "t <= 0" then amounts to the test "d[25] = 0".

2.2. The V-operation by the transput machine.

Again
we shall denote the semaphore by t and the associated flip-flop by T,
and it will hold that T = 1 if 0 < t. The V-instruction is now simply

"t := t + 1; if 0 < t then T := 1"

to be carried out by the transput mechanism as an indivisible action, not to be held back by the computer.

The P-operation the computer will carry out only when T = 1; the
computer can, however, by grace of deafness, permit itself to set T = 0
temporarily.

The completion of the P-operation by the computer now consists of the following little piece of program

"t := t - 1;

T := 0;

if 0 < t then

T := 1"

Here it is essentially assumed

(a) that it can do no harm if T is wrongly = 0 for a while;

(b)
that, if transput machine and computer "simultaneously" try to carry
out the assignment "T := 1", then afterwards "T = 1" will certainly
hold.

In order to show that the above little piece of program afterwards, under
all circumstances, leaves the value T in accordance with t, we need not
suppose that T was initially positive. (If we reverse the first two
statements, then I do have to assume that; a clearer insight into this
sort of little game is still very much desired!) We can, if desired,
also use this program to give, via "t := t - n", a so-called negative
prescription.

The reasoning is as follows. Every V-operation before the assignment T
:= 0 from the above little program is of no consequence, because after
it T := 0 is in any case carried out. If during the assignment T := 0 t
is positive, then the computer will eventually carry out the statement T
:= 1. Through interposed operations V t can, after all, then only become
still larger and T is thus left correct. If, on the contrary, during the
assignment T := 0 t is non-positive, then we must distinguish two cases.
If, despite any interposed V-operations, t remains non-positive, then
both transput machine and computer will leave T further undisturbed and
thus T will be left correct = 0.

If,
on the contrary, through interposed V-operations t becomes positive,
then in any case by the transput machine, at the moment of changeover, T
:= 1 will be carried out (and possibly, superfluously, once more by the
computer too, namely if t is already positive at the time of the test).
In this latter case T is thus guaranteed to be left correct = 1.

After these analyses of the V- resp. P-operation by the computer I trust

(a) that the reader has convinced himself of the water-tightness

(b) that the reader does not hold against me the extensiveness with which I have spun this out. On the contrary, even.

When the P-operation must be carried out by the machine, then the associated flip-flop T is a so-called interrupt-flip-flop. An interrupt-flip-flop sits in the basic machine, can be set by the computer in both directions, can be set to 1 by the transput machine. (This in contrast to an action-flip-flop, see 2.1.)

2.3. Hardware for the benefit of the interrupt-control.

Interrupt-flip-flops can (see 2.2.) be set individually in both directions.

To each interrupt-flip-flop a listen-bit is added. This too is
implemented as a flip-flop; the listen-bits too can be set individually
in both directions by the computer.

Interrupt-flip-flops, as well as listen-bits, can be read out word-wise
via a communication instruction. (As long as there are fewer than 27
per type, then we do this with one word for each type.) Corresponding
flip-flops have in these words the same position.

Finally there is a deaf-hearing-bit in the computer, which also disposes
of the instructions "make deaf" and "make hearing".

When the following two conditions are satisfied:

(a) the machine is hearing

(b) the collation-result of interrupt-word(s) and (corresponding) listen-word(s) is ≠ 0

then an interrupt takes place, i.e.

instead
of normally fetching the next instruction, a subroutine-jump is placed
in the instruction register, which is carried out with suppression of
the incrementing of the instruction counter; at the same time the
machine is, by this action, automatically made deaf.

Rem.
From this it would follow that an interrupt can take place only at a
spot in the program where the machine is hearing. If during the last
instruction before the interrupt it has already been decided that this
interrupt will take place, then it is possibly difficult to come back on
this decision if the last instruction happened to be precisely the
instruction "make deaf". The solution is, when storing away the link, to
also save the "deaf-hearing-ness" and, on the continuation of the
interrupted program, to jump back "restoringly". With the X1 it has
turned out that "delayed action" of the instructions "make deaf" and
"make hearing", provided this delay extends only over 1 instruction,
gives no difficulty whatsoever.

The little program for the lowering of T in 2.2. will take place in deafness; hence T may freely have a wrong value for a moment.

The listen-bits enable us to ignore interrupt-flip-flops - possibly
temporarily - namely when there is - as yet - no interest in them.

transcribed by Johan E.

Mebius

revised Thu, 29 Mar 2007
