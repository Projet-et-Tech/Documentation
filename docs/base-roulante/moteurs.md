# Moteurs

---

- *Fichiers : `moteurs.c` et `moteurs.h`*

Bienvenue dans ce chapitre qui a pour objectif d'expliquer comment le code de ce projet pilote les moteurs utilisés pour la base roulante.

> [!WARNING]
> Nous n'aborderons pas en détail le fonctionnement interne des moteurs ni les raisons de leur choix, ces aspects étant traités dans la section **Électronique** de cette documentation.
>
> C'est également dans cette section que nous aborderons de manière plus approfondie le **contrôle des moteurs** ainsi que le **fonctionnement des ponts en H**.

## Quels moteurs ?

Nous avons opté pour des **moteurs à courant continu** (MCC) pour les raisons suivantes :

- **Précision suffisante** : Avec un asservisement bien calibré, ils répondent parfaitement à nos besoins.
- **Bon couple** : Ils offrent un couple suffisant pour nos applications.
- **Vitesse élevée** : Capables d’atteindre de fortes vitesses.
- **Consommation modérée** : Ils restent peu énergivores en courant, ce qui rend leur usage plus facile.

## Contrôle des Moteurs via PWM

### Initilisation

Nous commençons par initialiser les **TIMs en mode PWM** au début du programme, avant la boucle principale.

```c
void init_moteurs()
{
    // Initialisation des PWMs
    HAL_TIM_PWM_Start(&htim9, TIM_CHANNEL_1);
    HAL_TIM_PWM_Start(&htim9, TIM_CHANNEL_2);
    HAL_TIM_PWM_Start(&htim10, TIM_CHANNEL_1);
    HAL_TIM_PWM_Start(&htim11, TIM_CHANNEL_1);
}
```

### Contrôle individuel

Pour contrôler les moteurs, nous utilisons des des Ponts en H avec des **PWM** (Modulation de Largeur d’Impulsion). Cette méthode permet de :

- **Régler la vitesse** de chaque moteur de manière précise.
- **Contrôler le sens de rotation** (avant/arrière).

La fonction **`controle_moteurs(...)`** met à jour la commande de **tous les moteurs** en appelant les fonctions spécifiques **`controle_moteur_MX(...)`** pour chaque moteur individuel.

> [!NOTE]
> Dans notre projet, nous avons **limité l’intervalle des valeurs PWM** à des entiers compris entre **0 et 255**. Ce choix présente deux avantages majeurs :
>
> - **Optimisation de la taille des données** : La vitesse est codée sur **1 octet** (8 bits), ce qui simplifie les **manipulations** et les **communications** (ex. : envoi via UART).
> - **Précision suffisante** : L’intervalle reste **assez large** pour contrôler la vitesse des moteurs avec une très bonne précision.

Chaque moteurs à aussi deux pins connecté au pont en H pour permettre de communiquer à celui-ci le sens dans le quel on souhaite que notre moteur tourne.

#### Implémentation contrôle moteur individuel

```c
/*
 * Contrôle du moteur M3
 *
 * @param direction : SENS_AVANT ou SENS_ARRIERE
 * @param vitesse   : Valeur de vitesse (0 à 255, 0 = arrêt, 255 = vitesse maximale)
 *                   Les vitesses négatives sont gérées en amont par la fonction `controle_moteurs(...)`.
 */
void controle_moteur_M3(uint8_t direction, uint8_t vitesse)
{
    // Configuration des pins du pont en H selon la direction
    if (direction == SENS_AVANT)
    {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_14, GPIO_PIN_RESET);  // Pin 14 à 0
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_15, GPIO_PIN_SET);    // Pin 15 à 1
        __HAL_TIM_SET_COMPARE(&htim11, TIM_CHANNEL_1, vitesse); // Mise à jour du PWM
    }
    else if (direction == SENS_ARRIERE)
    {
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_14, GPIO_PIN_SET);    // Pin 14 à 1
        HAL_GPIO_WritePin(GPIOB, GPIO_PIN_15, GPIO_PIN_RESET);  // Pin 15 à 0
        __HAL_TIM_SET_COMPARE(&htim11, TIM_CHANNEL_1, vitesse); // Mise à jour du PWM
    }
    else // Arrêt du moteur
    {
        // Arrêt rapide si activé
        if (FAST_STOP_ENABLE)
        {
            HAL_GPIO_WritePin(GPIOB, GPIO_PIN_14, GPIO_PIN_SET);  // Pin 14 à 1
            HAL_GPIO_WritePin(GPIOB, GPIO_PIN_15, GPIO_PIN_SET);  // Pin 15 à 1
            __HAL_TIM_SET_COMPARE(&htim11, TIM_CHANNEL_1, 256); // PWM à 256 (arrêt forcé)
        }
        else // Arrêt normal
        {
            HAL_GPIO_WritePin(GPIOB, GPIO_PIN_14, GPIO_PIN_RESET); // Pin 14 à 0
            HAL_GPIO_WritePin(GPIOB, GPIO_PIN_15, GPIO_PIN_RESET); // Pin 15 à 0
            __HAL_TIM_SET_COMPARE(&htim11, TIM_CHANNEL_1, 0);     // PWM à 0
        }
    }
}
```

> [!NOTE]
>
> Pour déterminer quels pins mettre à 0 ou à 1 en fonction du sens de rotation du moteur souhaité, il vous fatu consulter 3 ressources, j'ai noté :
>
> - La PCB de la carte moteur pour les branchements.
> - Le fichier IOC pour la configuration des pins.
> - La datasheet du Pont en H

### Contrôle de groupe

```c
/**
 * Contrôle groupé des moteurs
 *
 * @param commande : Tableau de 4 valeurs (vitesse signée pour chaque moteur)
 *                  - Valeur positive : sens avant, valeur négative : sens arrière
 *                  - 0 : arrêt du moteur
 */
void controle_moteurs(const float commande[4])
{
    // [...]

    // Vérification si la commande pour le moteur M1 a changé
    if (lastCommande[0] != (int16_t)commande[0])
    {
        // Détermination du sens et de la vitesse pour le moteur M1
        if (commande[0] > 0)
            controle_moteur_M1(SENS_AVANT, (uint8_t)commande[0]);  // Sens avant
        else if (commande[0] < 0)
            controle_moteur_M1(SENS_ARRIERE, (uint8_t)(-commande[0])); // Sens arrière
        else
            controle_moteur_M1(STOP, 0); // Arrêt du moteur

        // Mise à jour de la dernière commande pour M1
        lastCommande[0] = (int16_t)commande[0];
    }

    // [...]
    // Répéter pour les autres moteurs (M2, M3, M4) avec leurs index respectifs
}
```

> [!NOTE]
> Pour chaque moteur, le sens de rotation est déterminé par le signe de la vitesse. La valeur absolue de la vitesse est utilisée pour le contrôle individuel du moteur. Si la vitesse est nulle, le moteur est arrêté.

## Configuration des PWM (STM32)

Pour configurer les **signaux PWM** :

1. Ouvrez votre projet dans **STM32CubeIDE** et accédez au fichier **.ioc**.
2. Dans l'onglet **"Pinout & Configuration"**, sélectionnez le **périphérique de timer** que vous souhaitez utiliser (ex: TIM1, TIM2, etc.).
3. Cliquez sur le timer et choisissez **"PWM Generation"** dans le menu déroulant.
4. Configurez les **canaux** (CH1, CH2, etc.) en fonction des broches que vous utilisez.
5. Réglez les paramètres :
      - **Période** (en fonction de la fréquence souhaitée).
      - **Résolution** (ex: 16 bits).
      - **Mode** (PWM1 ou PWM2 selon vos besoins).
6. Validez et générez le code.

> [!TIP] Astuce
> Assurez-vous que les broches sélectionnées sont compatibles avec la fonction PWM sur votre microcontrôleur. POur ce faire, vous pouvez consulter le **Pinout** de la carte sur internet.

### Configuration des paramètres

> [!WARNING]
> Pour cette partie, je vous invite à consulter cette [ressource](https://deepbluembedded.com/stm32-pwm-example-timer-pwm-mode-tutorial).

#### Choix de l'auto-reload

> [!TIP] En résumé
> Il est possible d'ajuster la valeur du **Prescaler (PSC)** et de l'**auto-reload (ARR)** pour influencer la fréquence. L'auto-reload a un impact direct sur les valeurs que peut prendre votre signal PWM. L'objectif est donc de régler ces deux paramètres pour obtenir :
>
> - Un intervalle de valeurs PWM souhaité (dans notre cas, **[0, 255]**).
> - Une fréquence PWM adaptée à vos besoins.

#### Choix de la fréquence / période

La **fréquence** du signal PWM est déterminée par la formule suivante :

$$
\text{Fréquence (Hz)} = \frac{\text{Fréquence du timer (Hz)}}{\text{(PSC + 1) (ARR + 1)}}
$$

où :

- **PSC** est la valeur du Prescaler.
- **ARR** est la valeur de l'auto-reload.

Pour choisir la fréquence, il est important de prendre en compte :

- Les **contraintes matérielles** (par exemple, la fréquence maximale supportée par vos moteurs ou capteurs).
- La **résolution** souhaitée pour le signal PWM (par exemple, un intervalle de **[0, 255]** nécessite un ARR d'au moins **255**).
- Les **performances** globales du système (une fréquence trop élevée peut surcharger le microcontrôleur).

Dans notre cas, nous visons une fréquence de **20 kHz** (courante pour le contrôle des moteurs), tout en garantissant une résolution suffisante pour couvrir l'intervalle **[0, 255]**. Vous devrez donc ajuster **PSC** et **ARR** en conséquence.

> [!WARNING] Attention au "+1" dans les formules
> N'oubliez pas de **prendre en compte le "$+1$"** dans les formules de calcul pour **PSC** (Prescaler) et **ARR** (Auto-Reload Register). Ce détail est crucial pour obtenir la **fréquence souhaitée**.

## Déplacements de base

Dans le fichier `moteurs.c`, vous trouverez une partie du code dédiée aux **fonctions de base** permettant de réaliser les mouvements fondamentaux, tels que les **rotations** et les **translations**, pour une base roulante holonome à 4 roues.

> [!CAUTION] Attention !
> Ces fonctions sont **codées en dur** : l'ordre des commandes dépend directement de la **numérotation des moteurs** que vous avez choisie, ainsi que de la définition de l'**avant**, de l'**arrière**, de la **droite** et de la **gauche** sur votre robot.
>
> Ces fonctions éxistent principalement pour des raisons de test

## Ressources et Références

- STM32CubeIDE
    - [STM32 PWM Generation Tutorial](https://deepbluembedded.com/stm32-pwm-example-timer-pwm-mode-tutorial/)
- Pont en H
    - [Eurobot - Pont en H](https://www.eurobot.org/wiki/fr/electronics/pont_en_h)
    - [Fribotte - DrivesPontH](http://fribotte.free.fr/bdtech/Drivers/DrivesPontH.html)
    - [Wikipedia - Pont en H](https://fr.wikipedia.org/wiki/Pont_en_H)
    - [Fonctionnement Pont En H](https://ressources.univ-lemans.fr/AccesLibre/UM/Pedago/physique/02/electro/pontenH.html)
