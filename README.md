# Documentation P&T

Pour ajouter des pages, les créer au format Markdown (`*.md`) sous `./docs/` et le sous-dossier adéquat.

Le site se construit tout seul en utilisant l'intégration continue de GitLab une fois les modifications poussées.

## Outils

Cette documentation utilise [mkdocs](https://www.mkdocs.org/) avec le thème [mkdocs material](https://squidfunk.github.io/mkdocs-material/). En cas de modification de configuration, se référer à la documentation du thème en premier lieu.

### Fonctionnalités actuelles

- Blocs de codes colorées automatiquement
- Rendu d'équations mathématiques grace à KaTeX
- Abbréviations

# Principes de rédaction

- Éviter les images autant que possible, ça prend de la place et ça diminue l'accessibilité (typiquement pas de capture d'écran pour une configuration de logiciel, retranscrire en texte).
- Ne pas créer de page directement dans `./docs/`. Pour rester organisé, la créer dans la catégorie adéquate (élec, info, méca) ou ajouter le contenu voulu dans une nouvelle section de `./docs/index.md`.
- Vérifier l'orthographe, c'est moche une documentation mal écrite.
