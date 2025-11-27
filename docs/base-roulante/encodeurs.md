# Codeur Incrémental

---

- *Fichiers : `encoders.c` et `encoders.h`*

Bienvenue dans ce chapitre dédié au fonctionnement des **codeurs incrémentaux** (ou codeuses). Dans cette partie, je commencerai par expliquer brièvement le principe de fonctionnement de ces capteurs, ainsi que la manière dont nous les utilisons.

Ensuite, j'aborderai comment nous interagissons avec eux via notre code pour mesurer la **vitesse de rotation réelle** de nos moteurs.

> [!TIP] Rappel
> Dans notre configuration, les codeurs sont directement intégrés aux moteurs, montés à l'arrière de ceux-ci.

## Définition

Un **codeur incrémental** est un capteur électromécanique qui convertit un mouvement de rotation (ou linéaire) en une série d'impulsions électriques. Il permet de mesurer des **déplacements relatifs** (incréments) plutôt que des positions absolues.

## À quoi ça sert ?

- **Mesure de position/vitesse** : Détermine la vitesse de rotation, la distance parcourue ou l'angle de rotation.
- **Contrôle de mouvement** : Utilisé dans les systèmes de commande de moteurs (ex : robots, machines CNC, imprimantes 3D).
- **Rétroaction (feedback)** : Fournit des informations en temps réel pour ajuster la position ou la vitesse d'un système.
- **Applications courantes** :
    - Automobile (capteurs de position de pédales, volants).
    - Industrie (automatisation, convoyeurs).
    - Électronique grand public (souris optiques, encodeurs rotatifs).

## Fonctionnement

### 1. Structure de base

- Un disque perforé ou strié (avec des fentes ou des motifs optiques/magnétiques).
- Un émetteur (LED, aimant) et un récepteur (photodiode, capteur à effet Hall).
- Deux signaux de sortie en quadrature (déphasés de 90°) : **A** et **B**.
- Un signal optionnel **Z** (index) pour marquer une position de référence.

> [!NOTE]
> Le signal **$Z$** est disponible sur certaines codeuses. Bien qu’il ne soit **pas indispensable**, il offre un **signal utile** : il passe à un état haut **lorsque la codeuse a effectué un tour complet**. Cela peut faciliter certaines applications, comme la détection de position absolue ou la synchronisation.

### 2. Principe de détection

- Le disque tourne entre l'émetteur et le récepteur.
- Les fentes laissent passer (ou bloquent) la lumière/champ magnétique, générant des impulsions carrées sur les sorties **A** et **B**.
- Le déphasage entre **A** et **B** permet de déterminer le **sens de rotation** (horaire ou anti-horaire).

### 3. Traitement des signaux

- **Comptage des impulsions** : Chaque impulsion correspond à un incrément de mouvement (ex : 1 impulsion = 1° de rotation).
- **Résolution** : Nombre d'impulsions par tour (ex : 1000 impulsions/tour = résolution de 0.36°).
- **Vitesse** : Calculée en mesurant la fréquence des impulsions sur une période donnée.

### 4. Limites

- **Pas de position absolue** : Nécessite une initialisation (référencement) au démarrage.
- **Sensible aux perturbations** : Bruit électrique ou mécanique peut fausser le comptage.

## Schéma simplifié

```txt
   Disque rotatif
  ---------------
 |               |
 |   A   B       |  → Signaux en quadrature (A et B)
 |  █ █ █ █      |
  ---------------
       |
  Emetteur/Recepteur
```

> [!NOTE]
> Un circuit électronique (ex : microcontrôleur) compte et interprète les impulsions pour en déduire la position ou la vitesse.

## Implémentation

Il est important de noter que la carte que nous utilisons — et plus précisément le microcontrôleur (STM32F411CEU6) — intègre un module dédié aux codeurs incrémentaux.

Ce module se charge directement du traitement des signaux et de l’incrémentation ou décrémentation du compteur en conséquence.

### Initialisation

Les quatre modules STM32 dédiés aux codeurs sont initialisés et démarrés durant la phase d’initialisation du programme, avant l’entrée dans la boucle principale.

Pour cela, la fonction `HAL_TIM_Encoder_Start(...)` est utilisée afin d’activer chaque timer configuré en mode encodeur.

```c
HAL_TIM_Encoder_Start(&htim1, TIM_CHANNEL_ALL);
HAL_TIM_Encoder_Start(&htim2, TIM_CHANNEL_ALL);
HAL_TIM_Encoder_Start(&htim3, TIM_CHANNEL_ALL);
HAL_TIM_Encoder_Start(&htim4, TIM_CHANNEL_ALL);
```

### Mise à jour de la vitesse

La fonction suivante a donc pour objectif principal de déterminer, à partir de l’évolution du compteur par rapport à sa valeur précédente, la vitesse de la roue ainsi que son sens de rotation — indiqué par le signe de la vitesse.

Elle assure également la gestion des dépassements de valeur maximale et minimale du compteur.

```c
void update_encoder(Encoder *encodeur, TIM_HandleTypeDef *htim)
{
  // Lire une seule fois la valeur du compteur et l'autoreload
  uint32_t temp_counter = __HAL_TIM_GET_COUNTER(htim);
  uint32_t auto_reload = __HAL_TIM_GET_AUTORELOAD(htim);

    // Gestion du cas où le compteur n'a pas changé
    if (temp_counter == encodeur->valeur_precedente)
    {
        encodeur->delta_tick = 0;
        encodeur->vitesse_mms = 0;
    }
    else
    {
      // Vérifier si le timer est en mode décompte ou incrément
      uint8_t counting_down = __HAL_TIM_IS_TIM_COUNTING_DOWN(htim);

        if (temp_counter > encodeur->valeur_precedente)
        {
            // En mode décompte avec dépassement
            if (counting_down) {
              encodeur->delta_tick = -encodeur->valeur_precedente - (auto_reload - temp_counter);
            // En mode incrémentation simple
            } else {
                encodeur->delta_tick = temp_counter - encodeur->valeur_precedente;
            }
        }
        else
        {
            // En mode décompte avec sous-passement
            if (counting_down) {
              encodeur->delta_tick = temp_counter - encodeur->valeur_precedente;
            // En mode incrémentation avec sous-passement
            } else {
              encodeur->delta_tick = temp_counter + (auto_reload - encodeur->valeur_precedente);
            }
        }

        encodeur->valeur_precedente = temp_counter;
        encodeur->vitesse_mms = encodeur->delta_tick * TICK_TO_MM_S;
    }
}
```

Dans l'odre, voici que réalise cette fonction qui a pour vocation d'être appelée à interval régulier dans la routine d'asservissement :

1. Lit la valeur actuelle du compteur du timer et la valeur d’auto-reload.
2. Vérifie si le compteur n’a pas changé :
      1. Si oui → delta_tick = 0 et vitesse = 0.
3. Sinon (le compteur a changé) :
      1. Détermine si le timer compte à rebours ou en incrémentation.
      2. Si la valeur du compteur est supérieure à la précédente :
            1. Si le timer compte à rebours → calcul du dépassement en mode décompte.
            2. Sinon → simple incrémentation.
      3. Si la valeur du compteur est inférieure à la précédente :
            1. Si le timer compte à rebours → simple décompte.
            2. Sinon → sous-passement en mode incrémentation.
      4. Met à jour la valeur précédente du compteur.
4. Calcule la vitesse en mm/s : `vitesse_mms = delta_tick * TICK_TO_MM_S`.

> [!NOTE]
> La constante **`TICK_TO_MM_S`**, utilisée pour la conversion, repose sur la **résolution du codeur incrémental**. Cette information est disponible dans la *datasheet* du capteur et correspond au nombre de ticks (impulsions) équivalant à **un tour complet de l'encodeur**.
>
> Dans notre cas, la résolution est de **3200 ticks par tour**, ce qui signifie que **3200 impulsions correspondent à une rotation complète** de l'encodeur.

### Structure de donnée

Les variables d’entrée et de sortie de cette fonction sont regroupées au sein de la structure `Encoder` :

```c
typedef struct {
    uint16_t valeur_precedente;  // Valeur précédente du compteur
    int16_t  delta_tick;         // Variation du compteur entre deux mesures
    int16_t  vitesse_pwm;        // Vitesse exprimée en valeur PWM (pas utilisée)
    float    vitesse_mms;        // Vitesse en mm/s
} Encoder;
```

## Ressources et Références

- [Lire les codeurs - Wiki Eurobot](https://www.eurobot.org/wiki/fr/informatics/lire_les_codeurs)
- [Les Codeurs](https://si.blaisepascal.fr/1t-codeurs/) - *Codeur incrémental double voie*
- [Getting started with TIM](https://wiki.st.com/stm32mcu/wiki/Getting_started_with_TIM)
- [STM32 Tutorials](https://wiki.st.com/stm32mcu)
    - [STM32 Rotary Encoder](https://stm32world.com/wiki/STM32_Rotary_Encoder)
    - [STM32 Timer Encoder Mode](https://deepbluembedded.com/stm32-timer-encoder-mode-stm32-rotary-encoder-interfacing/)
