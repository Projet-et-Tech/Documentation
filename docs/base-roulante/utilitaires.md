# Utilitaires

---

Bienvenue dans ce **chapitre** — si l’on peut dire — dédié aux **utilitaires**. Il s’agit d’un ensemble de **fonctions et de morceaux de code** conçus pour accomplir des **tâches spécifiques**, utiles à plusieurs endroits du projet ou présentant un **intérêt particulier**.

## Redirection de `printf` vers l'UART

- *Fichier: `main.c`*

La fonction suivante permet de **rediriger la sortie standard** (utilisée par `printf`) vers les **périphériques UART** du microcontrôleur :

```c
int __io_putchar(int ch)
{
  HAL_UART_Transmit(&huart1, (uint8_t *)&ch, 1, 10);
  HAL_UART_Transmit(&huart6, (uint8_t *)&ch, 1, 10);
  return ch;
}
```

### Pourquoi cette redirection ?

Dans un **système embarqué** (comme un STM32), il n’y a pas de **console système** comme sur un PC ou un OS tel que Linux. La fonction `printf` ne peut donc pas afficher directement du texte à l’écran.

Cependant, comme `printf` est **pratique et couramment utilisé** en programmation C, on redirige simplement son flux de sortie vers les **connexions UART** (ici, `huart1` et `huart6`). Cela permet d’utiliser `printf` pour :

- **Afficher des logs** (messages de débogage, états du système, etc.).
- **Communiquer avec un terminal série** (via un câble USB-UART, par exemple).

> [!NOTE]
> Cette méthode est particulièrement utile pour obtenir des **logs système via UART**, comme c’est le cas dans le fichier **`log.c`**, que je détaille dans un autre chapitre.

## Conversion `string` en `ENUM`

- *Fichier : `uart.c`*

Voici une **petite fonction utilitaire** en C permettant de convertir une **chaîne de caractères** en sa valeur associée dans une **énumération** :

```c
// [...]

char *lut[255] = {
  "HELP",
  "MOVE_F",
  "MOVE_B",
  "MOVE_L",
  "MOVE_R",
  ...
};

// [...]

uint8_t keyfromstring(const char *str)
{
  uint8_t i;
  for (i = 0; i < NBKEYS; i++)
    if (strcmp(str, lut[i]) == 0)
      break;
  return i;
}

// [...]
```

### Fonctionnement

La fonction parcourt un tableau de correspondance (`lut`) contenant les chaînes de caractères dans le même ordre que leur énumération associée. Dès qu’une correspondance est trouvée, elle retourne l’index correspondant.

### Contrainte

Les chaînes de caractères doivent être déclarées dans le même ordre que les valeurs de l’énumération pour garantir une correspondance correcte.

## Ressources et Références

- [How to Use printf and scanf on STM32 using UART](https://embedthreads.com/how-to-use-printf-and-scanf-on-stm32-using-uart/)
