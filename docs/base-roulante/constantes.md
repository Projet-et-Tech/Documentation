# Constantes globales

---

- *Fichiers : `defines.h`*

Le fichier **`defines.h`** agit comme un **fichier de configuration centralisé** pour le projet. Il regroupe **l'ensemble des constantes et variables globales** utilisées dans le système.

Ces éléments sont **organisés par catégories**, en fonction de la partie du programme à laquelle ils se rapportent. Ils peuvent servir à :

- **Activer ou désactiver** certaines fonctionnalités.
- **Modifier un comportement spécifique** du programme.
- **Définir des valeurs de référence** pour des calculs mathématiques.

Les variables et constantes sont définies sous forme de **macros**. Cela signifie qu’elles sont **remplacées directement dans le code** lors de la **compilation**, ce qui optimise les performances.

*(Voir cette [page](https://c.developpez.com/cours/bernard-cassagne/node104.php) pour plus d'explications)*

> [!NOTE]
> Les macros qui **stockent le résultat d’une opération** (par exemple, un calcul mathématique) sont **entourées de parenthèses**. Cela garantit que le calcul est effectué **comme prévu** lors de la pré-compilation.

## Explications des constantes

### **UART**

Constantes liées à la communication série (UART).

| Constante        | Valeur | Description                                 |
| ---------------- | ------ | ------------------------------------------- |
| `RX_BUFFER_SIZE` | 50     | Taille du buffer de réception.              |
| `TX_BUFFER_SIZE` | 50     | Taille du buffer d'émission.                |
| `UART1`          | 1      | Activation de l'interface UART1             |
| `LOG_COLORS`     | 1      | Activation des couleurs dans les logs.      |
| `DEBUG_LOG`      | 1      | Activation des logs de débogage (commenté). |

---

### **ENCODERS**

Constantes liées aux encodeurs des moteurs.

| Constante               | Valeur                                    | Description                                                            |
| ----------------------- | ----------------------------------------- | ---------------------------------------------------------------------- |
| `DELTA_T`               | 0.005                                     | Temps entre chaque interruption (en secondes).                         |
| `RAYON_ROUE`            | 0.03                                      | Rayon de la roue (en mètres).                                          |
| `RAYON_ROUE_MM`         | 30                                        | Rayon de la roue (en millimètres).                                     |
| `DISTANCE_ROUES_CENTRE` | 0.185                                     | Distance entre les roues et le centre du châssis du robot (en mètres). |
| `RESOLUTION_CODEUSES`   | 3200                                      | Nombre de ticks par révolution des encodeurs.                          |
| `DIFF_DELTA_T`          | (0.01 / DELTA_T)                          | ...                                                                    |
| `INV_RAYON_ROUE`        | (1 / RAYON_ROUE)                          | Inverse du rayon de la roue.                                           |
| `PERIMETRE_ROUE_MM`     | (M_TWOPI * RAYON_ROUE_MM)                 | Périmètre de la roue (en millimètres).                                 |
| `PERIMETRE_ROUE_M`      | (PERIMETRE_ROUE_MM / 1000)                | Périmètre de la roue (en mètres).                                      |
| `TICK_TO_RAD`           | (M_TWOPI / RESOLUTION_CODEUSES)           | Conversion des ticks en radians.                                       |
| `TICK_TO_MM`            | (PERIMETRE_ROUE_MM / RESOLUTION_CODEUSES) | Conversion des ticks en millimètres.                                   |
| `TICK_TO_M`             | (PERIMETRE_ROUE_M / RESOLUTION_CODEUSES)  | Conversion des ticks en mètres.                                        |
| `TICK_TO_M_S`           | (TICK_TO_M / DELTA_T)                     | Conversion des ticks en mètres par seconde.                            |
| `TICK_TO_MM_S`          | (TICK_TO_MM / DELTA_T)                    | Conversion des ticks en millimètres par seconde.                       |

> [!TIP] Rappel
> Un **"tick"** correspond à une **impulsion** générée par un **encodeur incrémental**. Il s'agit d'une valeur **sans unité**, représentant une fraction d'un **tour complet** du moteur autour de son axe.
>
> *Pour plus de détails, reportez-vous au chapitre dédiée aux encodeurs.*

---

### MOTORS

Constantes liées aux moteurs.

| Constante          | Valeur | Description                               |
| ------------------ | ------ | ----------------------------------------- |
| `STOP`             | 0      | Arrêt du moteur.                          |
| `SENS_AVANT`       | 1      | Sens de rotation vers l'avant.            |
| `SENS_ARRIERE`     | 2      | Sens de rotation vers l'arrière.          |
| `FAST_STOP_ENABLE` | 0      | Activation de l'arrêt rapide (désactivé). |

> [!IMPORTANT]
> Bien que nous ne l'ayons pas mis en œuvre, la **fiche technique du pont en H** que nous utilisons semble indiquer (à revérifier) qu'il est possible de réaliser un **"arrêt rapide"** en bloquant les moteurs. Cela peut être réalisé en configurant **les deux mêmes broches** (celles qui déterminent le sens de rotation) d'une manière spécifique.

---

### RAMPE

Constantes liées à la gestion des rampes de vitesse et d'accélération.

> [!NOTE]
> Cette partie **n'est pas utilisée** dans le code actuel, car le **PID de position** a remplacé la **rampe de vitesse** pour la régulation des vitesses et accélérations.
>
> *Remarque :*
>
> Il me semble que ces deux approches soient **interchangeables** dans notre contexte, selon l'usage qu'on en fait. Il serait même possible de les **combiner**, mais cela reste à vérifier.

| Constante  | Valeur | Description                                 |
| ---------- | ------ | ------------------------------------------- |
| `VMAX_PWM` | 200.0  | Vitesse maximale des moteurs (en PWM).      |
| `VMIN_PWM` | 0      | Vitesse minimale des moteurs (en PWM).      |
| `AMAX_LIN` | 200.0  | Accélération linéaire maximale.             |
| `VMAX_LIN` | 200.0  | Vitesse linéaire maximale.                  |
| `AMAX_ANG` | 2.0    | Accélération angulaire maximale.            |
| `VMAX_ANG` | M_PI_2 | Vitesse angulaire maximale (π/2 radians/s). |

---

### PID

Constantes liées au contrôleur PID.

| Constante            | Valeur   | Description                            |
| -------------------- | -------- | -------------------------------------- |
| `QSM_EPSILON`        | 0.001    | Valeur par défaut du coefficient $K_c$ |
| `PID_MANUAL`         | 0        | Mode manuel du PID.                    |
| `PID_AUTO`           | 1        | Mode automatique du PID.               |
| `MAX_INTEGRAL_ERROR` | VMAX_PWM | Erreur intégrale maximale.             |
| `MAX_OUTPUT`         | VMAX_PWM | Sortie maximale du PID.                |

---

### MPU-6050

> [!NOTE]
> En prévision de l'**implémentation de la fusion de capteurs**, j'ai développé et testé ce code pour **communiquer avec ce capteur** (l'un de ceux disponibles à P&T), qui intègre :
>
> - Un **accéléromètre 3 axes**.
> - Un **gyroscope 3 axes**.

Constantes liées au capteur MPU-6050.

| Constante          | Valeur | Description                                   |
| ------------------ | ------ | --------------------------------------------- |
| `MPU6050_ADDR`     | 0xD0   | Adresse I2C du MPU-6050.                      |
| `ACCEL_CONFIG_REG` | 0x1C   | Registre de configuration de l'accéléromètre. |
| `GYRO_CONFIG_REG`  | 0x1B   | Registre de configuration du gyroscope.       |

---

### AltIMU-10 v4

Constantes liées au capteur AltIMU-10 v4.

> [!NOTE]
> En prévision de l'**implémentation de la fusion de capteurs**, j'ai également développé et testé du code pour **communiquer avec ce capteur** (disponible à P&T). Celui-ci intègre :
>
> - Un **accéléromètre 3 axes**.
> - Un **gyroscope 3 axes**.
> - Un **magnétomètre 3 axes**.
> - Un **altimètre**.

| Constante      | Valeur     | Description                                        |
| -------------- | ---------- | -------------------------------------------------- |
| `L3GD20H_ADDR` | 0b11010110 | Adresse I2C du gyroscope L3GD20H.                  |
| `LSM303D_ADDR` | 0b00111010 | Adresse I2C du magnétomètre/accéléromètre LSM303D. |
| `LPS25H_ADDR`  | 0b10111010 | Adresse I2C du capteur de pression LPS25H.         |

> [!TIP]
> *(Idée personnelle — ni testée ni validée :)*
>
> Comme évoqué dans un cours avec Goatailler, il est possible, moyennant une **calibration adaptée**, de **fusionner les données** de ces capteurs (accéléromètre, gyroscope, magnétomètre) pour obtenir une **orientation absolue** du robot (comme une boussole).
>
> Cela pourrait permettre de **corriger la dérive du gyroscope** et d'améliorer la précision de l'odométrie.
>
> *À explorer et valider en pratique !*

---

### TRAJECTOIRES

> [!NOTE]
> Ces éléments **ne sont pas utilisés** dans le code actuel.
>
> Ils étaient initialement **destinés à la gestion de trajectoires**, pour des **mouvements plus complexes** (non implémentés à ce jour).

Constantes liées aux trajectoires.

| Constante            | Valeur | Description                                    |
| -------------------- | ------ | ---------------------------------------------- |
| `BEZIER_NB_PAS`      | 500    | Nombre de pas pour une courbe de Bézier.       |
| `PRECISION_ABSCISSE` | 0.001  | Précision de l'abscisse.                       |
| `NB_POINTS_MAX`      | 20     | Nombre maximal de points pour une trajectoire. |

---

### TRAJET

Constantes liées aux configurations de trajet.

> [!NOTE]
> **Non utilisé** — Similaire aux fonctions de gestion de trajectoires, cet élément n'est pas exploité dans la version actuelle.

| Constante                         | Valeur    | Description                                         |
| --------------------------------- | --------- | --------------------------------------------------- |
| `TRAJECT_CONFIG_AVANCE_DROIT`     | 1000, 500 | Vitesse et accélération pour une translation pure.  |
| `TRAJECT_CONFIG_AVANCE_ET_TOURNE` | 300, 500  | Vitesse et accélération pour un mouvement complexe. |
| `TRAJECT_CONFIG_STD`              | 500, 500  | Vitesse et accélération standard.                   |
| `TRAJECT_CONFIG_ROTATION_PURE`    | 2, 2      | Vitesse et accélération pour une rotation pure.     |
| `DISTANCE_INVALIDE`               | -1        | Valeur pour une distance invalide.                  |

---

### MATH

Constantes mathématiques.

| Constante      | Valeur              | Description                       |
| -------------- | ------------------- | --------------------------------- |
| `DEGRE2RADIAN` | (M_PI / 180.)       | Conversion des degrés en radians. |
| `RADIAN2DEGRE` | (1. / DEGRE2RADIAN) | Conversion des radians en degrés. |
| `DEG2RAD`      | (DEGRE2RADIAN)      | Alias pour `DEGRE2RADIAN`.        |
| `RAD2DEG`      | (RADIAN2DEGRE)      | Alias pour `RADIAN2DEGRE`.        |

---

### ODOMETRIE

Constantes liées à l'odométrie.

| Constante                | Valeur | Description                        |
| ------------------------ | ------ | ---------------------------------- |
| `APPROXIMATION_AVEC_ARC` | 1      | Approximation avec arc (commenté). |
| `APPROXIMATION_SANS_ARC` | 1      | Approximation sans arc (commenté). |

---

### STRATEGIE

> [!IMPORTANT]
> Ces constantes étaient **initialement prévues** pour le cas où la **base roulante** aurait également eu à **gérer la stratégie** du robot.
>
> *(Elles ne sont pas exploitées dans la configuration actuelle.)*

Constantes liées à la stratégie de match.

| Constante        | Valeur | Description                                              |
| ---------------- | ------ | -------------------------------------------------------- |
| `TEMPS_MATCH_S`  | 100    | Durée du match (en secondes).                            |
| `TEMPS_MATCH_MS` | 100000 | Durée du match (en millisecondes).                       |
| `TEMPS_PAMI_S`   | 85     | Durée de la phase de match des PAMIs (en secondes).      |
| `TEMPS_PAMI_MS`  | 85000  | Durée de la phase de match des PAMIs (en millisecondes). |

---

### AUTRES

Autres constantes.

| Constante           | Valeur          | Description                               |
| ------------------- | --------------- | ----------------------------------------- |
| `TARGET_CORRECTION` | (200.0 / 187.0) | Facteur de correction.                    |
| `OPTIMISATION`      | 0               | Activation de l'optimisation (désactivé). |
| `DEBBUGING`         | 1               | Activation du débogage.                   |

> [!BUG]
> Le **facteur de correction** est une valeur déterminée **de manière empirique**. Il vise à compenser au maximum **l'erreur d'odométrie**, qui semble **proportionnelle à la distance parcourue**.
>
> L'origine de cette erreur reste **incertaine** : elle persiste de façon **constante** à travers différents essais, ce qui rend peu probable une cause liée **uniquement au glissement des roues**.

## Ressources et Références

- [Datasheet MPU-6050](https://invensense.tdk.com/wp-content/uploads/2015/02/MPU-6000-Datasheet1.pdf)
