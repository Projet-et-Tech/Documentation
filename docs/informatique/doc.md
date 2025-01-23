# Documentation de la documentation

## Organisation sur [git](https://github.com/Projet-et-Tech/Documentation)

Sur le git, il y a constamment trois branches : `main`, `dev` et `gh-pages`.

### Branche `main`

Elle contient la dernière version de notre documentation, donc est la source de la GitHub Pages actuellement consultable.

Elle n'est pas modifiable directement (même par les admins).

### Branche `dev`

Elle contient la version à venir de la documentation, quand des modifications ont été faites et qu'elles sont attentes de vérification des responsables, **exclusivement pour la syntaxe Markdown**. Le fond est vérifier *avant* de merge ici.

### Branche `gh-pages`

La·e premier·ère qui fait partir une branche ou *touche* à cette branche va se faire déglinguer par le responsable de la documentation.

C'est la branche qui contient le site publié, donc tout le HTML, CSS et JS, elle est géré par la CI.

### Comment j'écris ma documentation ?

Il faut :

1. Créer une nouvelle branche qui part de `dev` (bien vérifier qu'elle ne parte pas de `main` et **surtout pas** de `gh-pages`).

2. Nommer sa branche logiquement, avec la convention `domaine/ajout`.

   Par exemple, je veux ajouter une page de tutoriel Altium dans la section électricité, j'appelle ma branche `elec/altium-tuto`.

3. Je fais toutes les modifications nécessaires dans la branche.

4. Une fois que les modifications sont satisfaisantes, et que la syntaxe Markdown est respectée au possible, je fais une [PR](https://docs.github.com/fr/pull-requests/collaborating-with-pull-requests/proposing-changes-to-your-work-with-pull-requests/about-pull-requests) depuis ma branche vers `dev`, en assignant un autre membre du pôle pour vérifier le fond.

5. Une fois les modifications approuvés, le relecteur peut effectuer la PR vers `dev`.

6. Régulièrement, les responsables de la documentation doivent vérifier les modifications apportées à `dev`, et s'assurer que la syntaxe Markdown est respectée, avant de faire une PR de `dev` dans `main`. C'est seulement une fois celle-ci acceptée que les ajouts seront disponibles sur le site.


*[CI]: Continuous Integration
*[PR]: Pull Request
