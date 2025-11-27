# Trajet et trajectoires

---

_Fichier: `trajet.c`, `trajet.h`, `trajectoires.c` et `trajectoires.h`_

> [!TIP]
> Une grande partie du contenu qui suit s’inspire **fortement** de [ce wiki Eurobot](https://www.eurobot.org/wiki/fr/informatics/de_l_art_de_deplacer_son_robot_avec_classe), que vous retrouverez également dans le sous-chapitre [Ressources et références](#ressources-et-références), ainsi que du code associé.

Bienvenue dans ce chapitre dédiée au trajet et aux trajectoires.

Il est important de préciser que, **à partir de ce stade**, bien que du code soit présent dans le projet, **celui-ci n’a ni été testé en développement, ni validé en conditions pratiques**.

Ce code a été conçu dans l’hypothèse où la base roulante, **de manière autonome et sans communication avec la carte mère**, devrait exécuter :

- des **mouvements point à point plus complexes** qu’une simple ligne droite (arcs, courbes de Bézier, etc.) ;
- des **trajets multi-points** intégrant divers points de contrôle pour atteindre un objectif spécifique.

**Ce qui suit relève donc uniquement de la théorie**, issue de mes lectures et de ma compréhension du sujet.

---

> [!IMPORTANT] Point important
> Il est tout à fait possible d’obtenir une **excellente routine de match** en utilisant uniquement des **mouvements simples en ligne droite**, à condition que ceux-ci soient bien optimisés.
>
> Cependant, l’utilisation de **chemins complexes** permet d’adapter les mouvements à des **situations plus spécifiques**, offrant ainsi une **flexibilité et une précision accrues**.

**Ceci étant dit, plongeons maintenant dans le vif du sujet.**

## Trajet

> [!BUG]
> TODO

## Trajectoires

> [!BUG]
> TODO

## Ressources et Références

- [Trajectoires courbes](https://techthetroll.wordpress.com/wp-content/uploads/2016/07/chemin_courbe.pdf)
- [Wiki Eurobot](https://www.eurobot.org/wiki/fr/informatics/de_l_art_de_deplacer_son_robot_avec_classe)
    - [Le trajet et les chemins](https://www.eurobot.org/wiki/fr/informatics/le_trajet_et_les_chemins)
    - [Trajectoires complexes](https://www.eurobot.org/wiki/fr/informatics/chemins_complexes)
