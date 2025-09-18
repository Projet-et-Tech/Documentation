# Simulation & Caméra

Ce répertoire centralise la partie simulation et vision du robot. L'objectif est de modéliser l'environnement dans lequel le robot évoluera (table, éléments de jeu, robot adverse) afin d'adapter les déplacements de nos différents robots et ainsi maximiser le nombre de points collectés. Nous utilisons Python et la bibliothèque Pybullet pour effectuer cette simulation.
Le répertoire contient également la partie traitement d'images, car les algorithme utilisé sont en grande partie identique. Bien que pour des raisons d'atomicité et clarté, ces deux parties pourraient être divisées.

La simulation a pour vocation de mettre en place des routines ou tests, en parallèle de la construction du robot ou en attente de matériel.

## L'application

Le code relatif et regroupé dans le répertoire `Simulation` .
Actuellement la branche d'avancement la plus récente est `update/image`.
Elle contient une simulation de la table de jeu, des obstacles, du robot et de la grille de pathfinding. Le robot est capable de se déplacer sur la table en évitant les obstacles et en suivant le chemin optimal calculé par l'algorithme A*.
Elle contient également la simulation de la détection caméra et de la calibration par Aruco dans l'environnement de simulation. Le code est très similaire à celui utilisé en conditions réelles, et permet de tester les algorithmes de vision sans avoir besoin de matériel.

## Organisation du code

- `src/`: Contient le code source principal.
  - `simulation/`: Gestion de la simulation PyBullet, du robot, de la grille et du pathfinding.
  - `utils/`: Fonctions utilitaires pour la vision, la transformation d'image, la segmentation, la détection Aruco, etc.
  - `urdf_models/`: Modèles URDF des objets, obstacles, table et robots utilisés dans la simulation.
  - `config.py`: Constantes globales de la simulation (dimensions, positions, IDs, ...).


## Principe

### Principe général

L'objectif est de maximiser le nombre de points collectés. Le point de vue que nous avons sur un match est donc global. Ainsi, il n'est pas nécessaire de modéliser le robot de façon détaillée ou ses actionneurs de manière précise. En revanche les comportements, vitesses et temps d'exécutions doivent être réalistes.

Pour simuler les actions du robot, on peut imaginer un timer qui représente le temps pris par le robot pour effectuer l'action de jeu. Le robot s'immobilisera lorsqu'il effectue cette action de manière à correspondre à ce que serait la réalité. De même, le robot sera modélisé par un cube dont les côtés correspondent à son envergure réelle.

Aussi, la manière dont le robot interagit avec les objets de la table est la suivante : lorsque le robot prend un objet, celui-ci disparaît de la simulation et entre dans "l'inventaire" du robot. Lorsque le robot pose un objet à un endroit, celui-ci réapparaît à l'endroit posé.

Aujourd'hui le robot n'est pas capable d'attraper des objets, en revanche la recherche de chemin optimal et effective (algo A*).

### Imagerie

Pour repérer le robot, nous utilisons 2 caméras, placées à 1m de hauteur (conformément aux règles de la coupe (jusqu'à 4 autorisées)). 

Nos exigences caméras sont les suivantes :
- une qualité d'image de 1080 x 720p (vidéo)
- une ouverture horizontale de 43°, 50° en verticale

Ces caméras nous permettront, grâce aux 4 codes Aruco placés sur la table et de celui placé sur le robot, de récupérer les coordonnées du robot dans le référentiel de la table en temps réel (et donc de pouvoir déterminer précisément sa position). Cette opération est réalisée une unique fois au lancement de l'application pour ne pas alourdir le traitement, et peut en cas de problème de détection, être réalisée manuellement par un opérateur lors de l'installation.

Cette imagerie est réalisée grâce à la bibliothèque OpenCV sur python.

### Structure des fichiers importants

- `main.py` : Point d'entrée principal pour la simulation pathfinding.
- `mainCameraSimu.py` : Simulation de la détection caméra et de la calibration par Aruco dans l'environnement de simulation.
- `mainCameraReel.py` : Utilisation de la détection caméra et de la calibration par Aruco en vrai.

## Dépendances

- Python 3.8+
- pybullet
- numpy
- opencv-python
- matplotlib

## Auteurs

N'hésitez à compléter les différentes documentations au fur et à mesure de vos découvertes.

- Ruben (septembre 2025)
