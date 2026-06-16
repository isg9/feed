---
title: "Letter to J. Arsac (EWD644)"
url: https://www.cs.utexas.edu/~EWD/transcriptions/EWD06xx/EWD644.html
published: "1977-10-01T00:00:00Z"
feed: ewd
guid: https://www.cs.utexas.edu/~EWD/ewd06xx/EWD644.PDF
---

# Letter to J. Arsac

EWD 644

UNIVERSITE DE
PARIS VI - U.E.R. 110

INSTITUT DE PROGRAMMATION

TOUR 55065 - 11, QUAI SAINT-BERNARD - 75-PARIS Ve
TEL.336.25.25 OU 326.12.21 - POSTE:

Le 10 Octobre 1977

Cher Professeur Dijkstra

Ceci n'est pas la r�ponse "officielle" � votre note EWD636-0. J'en r�digerai une, mais elle doit �tre elle aussi r�dig�e avec soin. Dans l'imm�diat, je voudrais seulement r�pondre sur deux points;
-1. Vous �tes un homme heureux. Vous avez l'esprit d'une vivacit� sans �gale (ne le niez pas. Je vous ai vu � l'ouvre � Saint Pierre de Chartreuse). Tout le monde n'a pas votre rapidit�, moi en particulier. Et je ne suis pas le pire, je suis redout� en France pour ma vitesse de r�action. En aucun cas, le fait qu'il vous a fallu 15 minutes pour r�soudre un probl�me ne peut �tre pris pour une r�f�rence. Rappelez-vous Saint Pierre de Chartreuse : combien des personnes pr�sentes (et ce ne sont pas n'importe qui. Mettez � part les observateurs: les membres du WG2.3 sont d'�minents programmeurs, du moins je l'esp�re) ont-elles r�solu votre probl�me sur le nombre de facteur k dans n! ?.

D�s lors, le probl�me fondamental pour moi est de savoir comment suppl�er au manque d'imagination, d'esprit cr�atif. Il n'y a pas de voie royale pour le faire. J'ai �mis l'hypoth�se qu'une solide capacit� � faire du calcul, dot�e de moyens appropri�s, pourrait pallier partiellement au manque d'imagination.

C'est chez moi une vieille exp�rience. Dans ma jeunesse, on me faisait faire de la " g�om�trie ". Vous savez bien: construire un carr� dont chaque c�t� ( ou la droite portant chaque c�t�) passe par un point donn�. La solution est triviale si vous avez la bonne id�e. Sur ce probl�me pr�cis, j'ai vu des gens chercher des jours ou des semaines... N'�tant pas tr�s intuitif, j'utilisais quasi syst�matiquement les calculs de la g�om�trie analytique. C'est long, c'est lourd, mais il est exceptionnel que �a �choue.

Cel� fait plus de 4 ans que je pratique quotidiennement les transformations de programme, et je les poss�de maintenant tr�s bien. Elles ne me demandent que peu d'efforts de pens�e, et je dirai m�me peu d'attention tant elles sont devenues pour moi "automatiques". Il n'y a aucun doute pour moi que je peux maintenant r�soudre des probl�mes qui m'auraient autrefois �chapp�. Je me suis entrain� sur votre livre A discipline of programming . Mais j'ai aussi d�velopp� mes propres exemples.

Pour moi, d�finitivement, la question n'est pas : combien de temps a-t-il fallu pour trouver la solution ? Mais bien plut�t : ai-je r�solu le probl�me?

-2. Sur le probl�me particulier des syst�mes " naifs" de transformation de programme, enti�rement d'accord avec vous. M�me si cel� peut vous surprendre, je ne crois pas plus que vous en un quelconque syst�me qui optimiserait " automatiquement" les programmes. C'est la raison pour laquelle je ne suis pas tr�s attir� par la voie prise par Rod Burstall, quelle que soit l'estime que je lui porte et l'admiration que j'aie pour son travail. "Il n'y a pas de voie royale..."

Quel que soit le chemin suivi, la programmation demande imagination et cr�ation, chose dont une machine ou des calculs sont d�pourvus. Il y a des formes d'esprit diverses. Celles qui richement pourvue en intuition vont droit au but n'ont pas � se soucier d'outils de calcul.

Celles qui n'ont pas cette facilit� peuvent d�velopper par l'exercice une aptitude au calcul et combler ainsi leurs lacunes. Les calculs sont fastidieux, et sources d'erreur. On peut les faire faire par une machine. C'est ce que j'ai fait durant le mois d'aout � Rio de Janeiro. Je me suis �crit ( en Snobol) un syst�me interactif de manipulation de programmes, incluant la transformation de proc�dures r�cursives sans param�tres en proc�dures it�ratives. Durant le mois de septembre, j'en ai exp�riment� l'emploi, par moi-m�me, et avec mes �tudiants. L'un d'eux est maintenant en train de poursuivre l'exp�rience, dont j'attends beaucoup d'enseignements.

Il est absolument �vident que si le syst�me est naif ( je l'ai �crit, seul, en un mois), l'utilisateur ne peut �tre naif, ou alors il n'en tirera rien. Je n'aurais probablement rien tir� moi-m�me su syst�me si je ne m'�tais pas entrain� pendant des ann�es � faire des transformations.

Il en va, pour moi, en ce domaine comme dans celui des calculs alg�briques. La premi�re rencontre avec a2 - b2 = (a-b)(a+b) est troublante, d�concertante: " je ne saurais jamais m'en servir". Avec le temps, cel� devient une seconde nature, et des enfants m�me pas tr�s dou�s peuvent devenir habiles en calcul alg�brique. Nul n'a jamais pr�tendu que celui-ci soit naif...

Je pense que l'exc�s de z�le des partisans des manipulations n'ait pu pousser � commettre l'erreur de pr�senter celles-ci comme la voie magique du succ�s. J'ai pu moi-m�me commettre cette erreur, ne fut-ce que sous la pression des referees ou membres des comit�s de programme qui n'accepteraient jamais un papier commen�ant par: voici une fa�on de programmer qui vous demandera des efforts. Il vous faudra d'abord apprendre les transformations de programme et savoir les utiliser. Apr�s quoi, l'�criture d'un programme vous demandera des pages de calcul. Mais cel� en vaut la peine, parce que rien ne vous resistera ( ou du moins bien peu de choses).

C'est le point de vue que j'ai adopt� dans mon livre " la construction de programmes structur�s " ( Navr� du titre. Je n'en ai pas eu le choix. L'�diteur voulait du " structur�" dans le titre parce que �a se vend bien. Voyez ce que vous avez fait...). J'invite le lecteur � l'effort, parce que �a paie. Suivant un proverbe fran�ais: Quand on n'a pas de t�te, il faut avoir des jambes. En un sens, les transformations de programme, ce sont mes b�quilles...

Je vais essayer de mettre ces r�flexions en forme, de les fonder sur des exemples convaincants. Je crois profond�ment � ce que je dis, parce que j'en ai �prouv� les bienfaits moi-m�me. La question ouverte, c'est de savoir si cela peut servir � d'autres. C'est sur ce point que vont maintenant porter mes efforts.

Merci de votre lettre. Cel� m'a �t� d'un grand r�confort que de savoir que j'avais au moins r�ussi � vous faire douter. Respectueusement et cordialement votre

J.ARSAC

EWD 644

Burroughs

PLATAANSTRAAT 5     NEUNEN     THE NETHERLANDS

DR. EDSGER DIJKSTRA

Professor J.Arsac
Institut de Programmation
Tour 55
9 Quai Saint-Bernard
University of Paris
75 PARIS Ve
France

Sunday 16th of October 1977

Dear Colleague,

thank you very much for your nice letter of the 10th of October 1977. It was a clear and very honest letter, which hence was very illuminating and taught me a lot —besides the word "b�quille" that I had never encountered before— .

From your letter it is perfectly clear that the two of us are products of rather different cultures with fairly different conceptions of a person's own responsibility for his own habits. I have been educated to feel myself responsible for the quality of my own handwriting —with the exception of the physically disabled, people write as well as they care— . I have been educated to feel myself responsible for my mastery of the language I use —with the exception of the mentally disabled, people express themselves as well as they care— and for the effectiveness of my thinking habits when solving problems.

From that point of view you are "escaping" from the responsibility to try to teach yourself how to think as effectively as possible, by just postulating that you are suffering from "lack of imagination and lack of enough creativity", and by just postulating that for instance I have such an incredibly quick mind that it is not fair for me to enter the competition or to use my own performances as a reference point for comparison. From my background I must consider those postulates as "easy ways out". The second one —and that is more serious— I consider as even wrong.

You see, I do deny having a quick mind. On the contrary, I have every reason to consider myself slow-witted —"l'esprit d'escalier" is for me a most common experience— . It was the awareness of my mind's slowness that induced me to learn to use what little wit I had as well as possible; it was, again, my awareness of my limited manipulative agility that motivated my training of myself to come away with as little formal manipulation as possible. You seem to discard my performance as "hors concours", as of a quick mind hardly with its equal; I would prefer you to interpret them as examples of what a slow mind can achieve, provided it has been trained well. (Your letter arrived yesterday morning before breakfast. After having read it —it was the only important letter that arrived that morning— I walked with the dogs and thought some about your letter in general and about the reconstruction of the square in particular. After having mentally constructed circles on which its vertices should lie, and having rejected the resulting picture because it became too complicated, I gave my attention to the dogs. After breakfast, during which we talked about other matters, my wife and I went shopping in Eindhoven. I drove; after having parked the car, we walked a few minutes from the parking lot to the "Bijenkorf" —a "Magasin du Nord" on Dutch scale— and my thoughts returned to the problem. On the escalators —which are of the double kind, the one going up perpendicular to the one going down— I said to my wife "I have solved Arsac's problem; it is, indeed, trivial.". And you must believe me when I write you that I solved it without great excitement, without any need for a divine spark, without any posing as a genius or what have you: walking from the parking lot I just did what any well-trained mind should obviously do. So I did it. Just that. And I didn't do it quickly either!)

The first sentence on the second page explains a lot. True that the time needed to find a solution is only of secondary importance —although it is undeniably an indication for the effectiveness with which someone has learned to think— . And then you state as the more important question "Have I solved the problem?", and here, again, I think that an important difference in our cultural backgrounds becomes manifest. Of course it is of some importance whether or not you have solved the problem (although not more important than the problem itself!) If you have, however, the question that is much more important in my scale of values is "Do you have a beautiful solution?". For most mathematical theorems I regard their most beautiful proofs as a greater contribution to our culture than the theorems themselves.

*         *         *

When replying to the second point you state (in the second paragraph) that there are different minds, and you seem to contrast the "calculators" with those "in rich supply of intuition". But I think the distinction a red herring. As Condorcet remarked of Euler's death: "Euler ceased to live and calculate."

Again, the distinction may seem meaningful as long as you think it helpful to postulate something intangible as "mathematical intuition" or whatever name you wish to give to the divine spark that I would rather not invent. For how do we see that wonderful "intuition" or "creativity" at work? The man calculates and calculates and calculates all the time. He has a great "aptitude au calcul" in the sense that by careful exploration of the structure of his formal obligations, he discovers how to avoid avoidable calculations, and by being so conscious of his own methodology, he has a greater likelyhood than most of arriving at the formal argument that is short enough to be convincing. He knows —by bitter experience, usually!— that mathematical elegance is not a dispensable luxury that only geniuses can afford themselves, but a matter of life and death. Avoiding avoidable formal manipulations and thereby keeping the amount of remaining formal labour to be done within the boundaries of the doable, that —and very little else— is precisely what mathematical elegance is all about. And a conscious analysis of the structure of your formal obligations should keep you on the right track. (Usually kept on the right track you don't waste your time on dead alleys, and you find yourself endowed with "a quick mind"!)

I honestly believe that, when all is said and told, the distinction you try to make between the calculators and the people with intuition is a red herring —for one thing: I would not know how to classify myself— . There are people who don't calculate —we can ignore them in this discussion— and those who do. Of the latter kind some do it more ugly and more clumsily than others, who take the responsibility for the beauty of their arguments.

I thank you once more for your thoughtful and illuminating letter and am looking forward to your "official" answer to EWD636, which, by the way, you should not regard as anything "official" itself, being only a part of what I regard as my scientific correspondence.

With warmest regards,

yours ever,

prof.dr.Edsger W.Dijkstra
Burroughs Research Fellow

PS. After rereading this letter I decided that I would like to give it some wider distribution, and hence I included it in the EWD-series. I trust that you don't regard this as a breach of confidence.

transcribed by Corrado Cantelmi

revised 18-Sep-2011
