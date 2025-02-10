# Projet
Ce projet vise à créer deux énigmes sur un cube. L'un comportera un message à déchiffrer un code et l'écrire sur un codex et le second consiste à aligner des miroirs pour diriger des faisceaux lasers sur des capteurs.  

Intro:
L’idée est de créer un cube comportant 4 énigmes permettant d’obtenir 3 clefs pour activer un mécanisme à la fin.
La Team A s’occupe de la face du haut, la Team B s’occupe d’une face latérale et la Team C nous nous occupons de deux face latérales et de la face ou l’on insère les clefs.
Notre première face comporte un codex et est relié à une seconde face qui est un parcours de lasers.

IMG CODEX

La partie avec le codex reprend des mécaniques simples mais efficaces des jeux d’énigmes. Cette partie du cube fera tout d’abord clignoter une LED. Et, un message codé se cache dans les clignotements de la LED. L’utilisateur devra donc décoder le message grâce à un tableau de correspondance inscrit quelque part sur le cube. Une fois le message décodé, le joueur n’a plus qu’à le reporter sur le codex de la face. Et si le message est correct, un mécanisme s’enclenche et débloque la seconde face de l’énigme avec les lasers. 

Techniquement, cette face comporte 2 éléments propres : une LED et un codex. La LED est directement reliée à une stm qui fera constamment clignoter le message. Et, le codex est constitué de plusieurs roues. Chacune des roues comprend une lettre du message. Si toutes les roues sont tournées dans le bon sens, un faisceau laser passe à travers ce dernier et est redirigé vers la seconde face de l’énigme. 

IMG LASER

La face avec les lasers comporte 4 lasers qui clignotent à différente fréquence invisible à l'œil nu afin de les différencier. Nous avons besoin de 4 capteurs capables d'identifier différents niveaux de clignotement.
Le but comme vous pouvez le voir sur l’image ci-dessus est de créer 4 parcours différents plus ou moins simple avec des obstacles pour ajouter de la difficulté aux parcours. 
Afin d’orienter les lasers nous utilisons des miroires, les tige qui s’enfonce dans le quadrillage seront crantée afin d'éviter une rotation sous le poids.(cela sera etudiee pour etre sur de l’utilité)

L’énigme une fois réussie ouvre une trappe pour avoir la clef, elle peut être résolution si et seulement si les lasers sont en face des bon capteurs.
