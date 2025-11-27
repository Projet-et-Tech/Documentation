# Système de Logs

---

- *Fichiers : `log.c` et `log.h`*

Pour **faciliter le débogage** et **comprendre le comportement du système** sans recourir à un débogueur externe (comme STM32CubeIDE), j’ai développé une petite bibliothèque de logs.

Celle-ci utilise les **interfaces UART** pour afficher des messages de différents niveaux (**INFO**, **DEBUG**, **DANGER**, etc.), accompagnés d’un **timestamp**, du **nom du fichier** et de la **ligne de code** à l’origine du log.

> [!NOTE]
> Les **couleurs** dans les logs peuvent être désactivées, car :
>
> - Certaines consoles ne les affichent pas correctement.
> - Elles ne sont pas utiles lors d’une communication entre deux cartes, où seules les **informations textuelles** comptent.

## Fonctionnement

### Dans `log.c`

- Les tableaux `level_strings[6]` et `level_colors[6]` stockent :
    - Les **libellés** des niveaux de logs (ex. : "INFO", "DEBUG").
    - Les **codes de couleur** associés (pour une meilleure lisibilité dans les terminaux compatibles).
- La fonction **`log_log(...)`** est la fonction principale :
    - Elle **formate le message** (niveau, timestamp, fichier, ligne, contenu).
    - Elle **envoie le log** via l’interface UART.

### Dans `log.h`

- Utilisation de **macros** pour :

    - Définir des fonctions de log **selon l’interface UART souhaitée**.
    - Activer ou désactiver le système de logs via la variable globale **`DEBUG_LOG`** (définie dans `defines.h`).

- Utilisation de **fonctions variadiques** (comme `printf`) pour :

    - Permettre un appel flexible à `log_log()` avec un nombre variable d’arguments.
    - Faciliter l’utilisation avec une syntaxe similaire à `printf()`.

## Déclaration et utilisation

### Déclaration

```c
// Fichier: log.h
// [...]

#define log_trace(...) log_log(&huart1, LOG_TRACE, __FILE__, __LINE__, __VA_ARGS__)
#define log_debug(...) log_log(&huart1, LOG_DEBUG, __FILE__, __LINE__, __VA_ARGS__)
#define log_info(...)  log_log(&huart1, LOG_INFO,  __FILE__, __LINE__, __VA_ARGS__)
#define log_warn(...)  log_log(&huart1, LOG_WARN,  __FILE__, __LINE__, __VA_ARGS__)
#define log_error(...) log_log(&huart1, LOG_ERROR, __FILE__, __LINE__, __VA_ARGS__)
#define log_fatal(...) log_log(&huart1, LOG_FATAL, __FILE__, __LINE__, __VA_ARGS__)

// [...]
```

### Utilisation

```c
// Exemple pour un log de niveau INFO
log_info("Position actuelle : (%.1f, %.1f, %.1f)", position->x, position->y, position->theta);
```

Cette approche permet d’avoir un **système de logs flexible, efficace et facile à utiliser**, tout en restant compatible avec les contraintes d’un système embarqué.

> [!NOTE]
> Les fonctions `log_xxx(...)` s'utilisent **de manière identique** à la fonction `printf`.
>
> Cela est possible car `log_log()` est une **fonction variadique**, et les différentes déclinaisons des fonctions `log_xxx()` ne sont que des **macros** qui appellent cette fonction sous-jacente avec des **paramètres prédéfinis**.

#### Example de logs

##### Code

```c
log_info("Hello, World");
log_error("Two variables: x = %d, y = %d", 10, 8);
```

##### Sortie

```text
[00:02] INFO  main.c:4: Hello, World
[00:02] ERROR main.c:5: Two variables: x = 10, y = 8
```

## Ressources et Références

- [C-logger](https://github.com/yksz/c-logger) - A simple logging library for C/C++
- [Variadic Functions In C](https://www.geeksforgeeks.org/c/variadic-functions-in-c/) - Tutorial about variadic functions
