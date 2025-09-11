# Projet&Tech
Considérations des aspects liés au travail en groupe, et à la présence de différents pôle.
## Communication
Même si le pôle mécanique est celui qui a le plus d'impact sur le design visuel du robot, la communication entre les différents pôles est très importante. 

Il faut absolument que la conception du robot prenne en-compte les contraintes éléctronique, lié au choix des actionneurs. Le choix des actionneurs à un impact significatif sur le nombre de pins utilisé, et donc sur l'architecture éléctronique du robot  
(nombre de carte, type de protocole de communication, architecture des cartes, modèle de micro-controlleur, etc...).

Le choix des actionneurs, et la faisabilité des actions a aussi un impact sur la partie informatique. Il faut limiter le nombre de motorisation différentes utilisé pour l'ensemble des actionneurs, et s'assurer que les mouvements de chaque actionneur soit cohérent. Avoir un actionneur qui demande trop d'attention ou de temporisation particulière est généralement un problème.

Le pôle mécanique ne doit donc pas décider des actionneurs tout seul, c'est une décision qui doit inclure l'avis de tout le monde.

Après chaque étape (modélisation d'un pièce, définition d'un cahier des charges), il faut faire valider la pièce par une personne investie dans le(s) pôle(s) concerné(s). De même, il faut faire les sessions de brainstorming avec le plus de personnes possible.

Il faut aussi récupérer du feedback, savoir si les pièces ont besoins d'être réajuster, même si elles fonctionnent quand même. Il peut aussi être intéressant d'avoir du feedback venant d'autres personnes du pôle mécanique, lors de la conception.

## Cahier des charges
La partie la plus difficile lorsque l'on débute, c'est de trouver et garder les solutions qui sont à la fois faisable, et fiable.
Il est difficile de définir une règle générale permettant de bien choisir une solution. Il faut réussir à se projeter, et faire un choix réflechi. Si le mécanisme est suffisament simple, il peut être intéressant de faire/modéliser un prototype rapide.

Parfois il n'y a pas besoin de modéliser de pièce, et un bricolage plus ou moins complexe peut être un bon compromis entre qualité et temps.

Après avoir choisi une solution, il faut définir des contraintes. Il est important d'effectuer ce travail avant de modéliser la pièce. En effet, cela permet de choisir quels paramètres seront fixé lors de la conception (même arbitrairement), et lesquels seront résolus/calculé automatiquement par le logiciel. C'est quelque chose qui permet de retoucher les pièces plus facilement et plus rapidement.

C'est un travail qui peut être plus difficile pour certaines personnes que pour d'autres, car cela peut faire appel à des compétences de visualisation mentale. Cependant, si on y consacre assez de temps, et qu'il y a suffisament de personnes, se n'est généralement pas un problème.

Lorsque l'on imagine une pièce, il faut prendre en compte le matériel présent à Projet&Tech. Le choix des vis et des actionneurs est important. Il peut être difficile de choisir un actionneur à la volée. Il faut avoir une idée de la visserie disponible, et prévoir si il faut en racheter.

Généralement, pour les actionneurs regarder la tension, et le couple maximum (et donc avoir une idée de la consommation éléctrique) suffit à faire un choix optimisé.
Pour un servomoteur, il faut aussi regarder l'amplitude de mouvement possible et l'amplitude de mesure possible.  (Par exemple, sur un potentiomètre, il n'y a pas forcément de bloquage mécanique, mais on ne dépasse généralement pas 300° d'amplitude éléctronique)

Lorsque l'on modélise une pièce, il faut aussi choisir un matériau et une méthode de fabrication. Si c'est de l'impression 3D, ça sera probablement du PLA, mais si on veut du bois ou du métal, il faudra le prendre en compte. Les matériaux ont différentes caractéristiques : Densité, Rigidité, Solidité à différents efforts, Coefficient de frottement, plasticité, etc...  Ils sont usiné par des machines différentes, et donc il y des contraintes d'assemblage et d'usinage différentes.  
Comparatif de différents matériau:  \[[matériau](#matériau)\] 

Il faut aussi prendre en compte l'importance et la taille de la pièce. Une pièce trop grande ne pourra pas forcément être usiné en 1 morceau. Cela peut aussi avoir un impact sur le choix de la machine utilisé pour l'usinage. De plus, une pièce importante doit probablement être fiable et résister à l'usure. Sinon il faut en faire plus, ce qui peut être un problème de temps et/ou d'argent.

## Gestion des tâches
Il faut faire très attention à toutes les idées qui peuvent passer par le pôle méca. Il faut noter chaque choses qui semble importante, ou qui est à faire.

Généralement, l'impression des pièces en 3D se fait par groupe de pièce, ce qui fait que vous pouvez être ammené à recevoir beaucoup de feedback par moments.

Cette tâche est importante car permet de voir la quantité de travail à réaliser avant de pouvoir passer à autre chose. De façon générale, voir les avancements et le travail effectué sur le robot permet aussi de se motivé, et de motiver le reste du groupe.

Le strict minimum est d'avoir une checklist avec indententation si-possible, pour catégoriser les tâches/pièces, et garder une trace du travail effectué / à effectué.

Il est aussi important de mettre en place un système de versionnage des pièces. Que ce soit localement en gardant un historique des pièces, ou sur un drive/git.

Pour donner un ordre de grandeur, (en 2024-25) il y a 72 pièces et 17 assemblages qui ont été modéliser pour le PAMI. La version finale n'utilise que 7 pièces et 1 assemblage.