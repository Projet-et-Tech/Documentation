# Odométrie

---

_Fichiers: `odometrie.c` et `odometrie.h`_

## C'est quoi ?

L'**odométrie** pour un robot est une technique qui permet d'estimer la position et l'orientation d'un robot mobile en mesurant les déplacements de ses roues (ou d'autres éléments mécaniques en contact avec le sol). Elle repose sur l'utilisation de capteurs, généralement des codeurs incrémentaux ou absolus, qui mesurent la rotation des roues.

### Pourquoi est-elle indispensable ?

La simple **commande envoyée aux moteurs** ne suffit pas pour connaître avec précision la **vitesse réelle** ou la **position actuelle** du robot.

Plusieurs phénomènes physiques peuvent fausser le mouvement attendu, même si chaque moteur reçoit une consigne de déplacement identique. Ces phénomènes se divisent en **deux grandes catégories** :

1. **Les frottements** :
      - Au **démarrage** (inertie à vaincre).
      - Pendant le **freinage** (résistance au ralentissement).
      - Lors des **accélérations** (variation de la force nécessaire).

2. **Les glissements** :
      - Liés à la **surface** (adherence variable : sol glissant, irrégulier, etc.).
      - Causés par un **obstacle** bloquant le robot (roues tournant dans le vide).

**Résultat** : Sans odométrie, le robot ne peut pas **corriger ces écarts** et risque de dévier de sa trajectoire prévue.

### Fonctionnement de base

1. **Mesure des rotations** : Les capteurs (codeurs) comptent le nombre de tours ou d'incréments effectués par chaque roue.
2. **Calcul des distances** : En connaissant le diamètre des roues, le robot convertit le nombre de tours en distance parcourue.
3. **Estimation de la position** : En combinant les distances parcourues par chaque roue (et en tenant compte de la géométrie du robot, comme l'écartement des roues), le robot calcule sa nouvelle position et son orientation par rapport à sa position initiale.

### Avantages

- **Simplicité** : Pas besoin d'infrastructure externe (comme des balises ou des caméras).
- **Coût réduit** : Utilise des capteurs peu coûteux et intégrés au robot.

### Limites

- **Erreurs cumulatives** : Les petites erreurs de mesure (glissement des roues, surface irrégulière, etc.) s'accumulent au fil du temps, ce qui peut rendre l'estimation de la position imprécise sur de longues distances.
- **Dérive** : Sans recalibration (par exemple avec un capteur externe comme un GPS ou un lidar), l'odométrie seule peut devenir inexacte.

## Pour une base roulante holonome à 4 roues

Une **base roulante holonome à 4 roues** (réparties à **90° les unes des autres**) permet un **déplacement dans n'importe quelle direction** sans modifier son **orientation**. Pour réaliser correctement son **odométrie pendant l'asservissement**, voici les **étapes clés** à suivre :

### Configuration mécanique

- Les roues choisit pour notre base roulante sont des roues **suédoises** car permettant un mouvement latéral et un peu moins sujettes au glissement que les roues **mecannum**.

  _(Le glissement reste présent, mais il peut être **quasi négligeable** sur la durée d'un match, à condition de cumuler des codeurs de qualité, une fusion de capteurs efficace et, éventuellement, un recalage pendant la partie.)_

- Chaque roue est motorisée par un moteur à courant continue équipée d'un codeur incrémental pour mesurer sa vitesse de rotation _(je vous renvoi au chapitre sur les codeurs)_.

- Les roues sont placées aux coins d'un châssis carré, à 90° les unes des autres. Le choix d'un **châssis carré** a été privilégié pour sa **symétrie** qui simplifie les calculs

### Modèle cinématique

Le **modèle cinématique** de notre base roulante permet d'établir les équations reliant la **vitesse des 4 roues individuelles** à la **vitesse du robot** selon ses trois axes ($X$, $Y$ et $\theta$).

#### Vitesse des roues

Soit :

- $v_i$ : vitesse linéaire de la roue $i$ (en m/s).
- $\omega_i$ : vitesse angulaire de la roue $i$ (en rad/s).
- $r$ : rayon des roues (en m).
- $L$ : demi-diagonale du châssis (distance du centre à une roue, en m).

La vitesse linéaire de chaque roue est donnée par :

$$
v_i = r \cdot \omega_i
$$

#### Vitesse du robot

La vitesse du robot dans le repère global ($V_x, V_y, \Omega$) est calculée à partir des vitesses des roues. Pour une base holonome à 4 roues, la matrice de transformation est :

$$
\begin{bmatrix}
V_x \\
V_y \\
\Omega
\end{bmatrix}
=
\frac{r}{4}
\begin{bmatrix}
-1 & 1 & 1 & -1 \\
1 & 1 & -1 & -1 \\
-\frac{1}{L} & \frac{1}{L} & -\frac{1}{L} & \frac{1}{L}
\end{bmatrix}
\begin{bmatrix}
\omega_1 \\
\omega_2 \\
\omega_3 \\
\omega_4
\end{bmatrix}
$$

- $V_x$ : vitesse du robot selon l'axe X.
- $V_y$ : vitesse du robot selon l'axe Y.
- $\Omega$ : vitesse angulaire du robot (en rad/s).

> [!WARNING]
> Avec une base **carrée**, la distance $L$ est **constante et identique** pour tous les moteurs.
> Pour une base **non carrée ou asymétrique**, cette valeur **varie pour chaque moteur** dans la matrice de conversion, ce qui complexifie les calculs.
>
> _Ne pas oublier de le prendre en compte_.

### Calcul de l'odométrie

#### Intégration des vitesses

Pour estimer la position ($x, y$) et l'orientation ($\theta$) du robot :

- Intégrez $V_x$ et $V_y$ pour obtenir les déplacements en $x$ et $y$.
- Intégrez $\Omega$ pour obtenir l'orientation $\theta$.

Les équations discrètes (pour une période d'échantillonnage $\Delta t$) sont :

$$
x_{k+1} = x_k + (V_x \cdot \cos(\theta_k) - V_y \cdot \sin(\theta_k)) \cdot \Delta t
$$

$$
y_{k+1} = y_k + (V_x \cdot \sin(\theta_k) + V_y \cdot \cos(\theta_k)) \cdot \Delta t
$$

$$
\theta_{k+1} = \theta_k + \Omega \cdot \Delta t
$$

> [!NOTE]
> Dans notre cas, l'odométrie étant calculée **pendant l'asservissement à intervals réguliers**, il n'est pas nécessaire de multiplier par $\Delta t$. En effet, celui-ci est **constant** et déjà intégré dans la conversion du nombre de ticks des encodeurs en $mm/s$ via la constante `TICK_TO_MM_S` _(je vous renvoi au chapitre correspondant)_.

#### Prise en compte de l'orientation

Les vitesses $V_x$ et $V_y$ doivent être projetées dans le repère global en fonction de l'orientation actuelle $\theta$, d'où les équations précédentes.

### Implémentation pratique

La fonction `localisation_update()` est appelée dans la boucle principal lors de la mise à jour de la position du robot, comme son nom l'indique.

```c
// fichier: odometrie.c
// [...]

void localisation_update(Position *position, Encoder encodeurs[4])
{
    update_encoder(&encodeurs[0], &htim3); // M1
    update_encoder(&encodeurs[1], &htim2); // M2
    update_encoder(&encodeurs[2], &htim4); // M3
    update_encoder(&encodeurs[3], &htim1); // M4

    // Calcule de l'odométrie
    compute_odometry(position, encodeurs);
}

// [...]

```

#### Mesure des vitesses des roues

- Utilisez les codeurs pour mesurer $\omega_i$ pour chaque roue.
- Convertissez $\omega_i$ en $v_i$ avec $v_i = r \cdot \omega_i$

> [!INFO]
> Cette étape est réaliser en amont de l'appel à la fonction `compute_odomtrie()`

#### Calcul des vitesses du robot

Appliquez la matrice de transformation pour obtenir $V_x$, $V_y$, et $\Omega$.

#### Mise à jour de la position

Mettez à jour $x$, $y$, et $\theta$ à chaque itération en utilisant les équations d'intégration.

> [!CAUTION]
> Toujours **vérifier**, après la mise à jour de la valeur d'orientation, que celle-ci reste dans l'intervalle défini pour nos calculs ([$-\pi$, $\pi$] dans notre cas). Cela permet d'éviter des **mouvements imprévisibles** du robot.

```c
/**
 * Calcule l'odométrie (position et orientation) d'un robot à partir des ticks des codeurs.
 *
 * @param position  Pointeur vers la structure Position à mettre à jour (x, y, theta).
 * @param encodeurs  Tableau de 4 structures Encoder, chacune contenant delta_tick (variation des ticks du codeur).
 */
void compute_odometry(Position *position, Encoder encodeurs[4])
{
    // Sortie anticipée si aucun mouvement n'est détecté (tous les deltas des codeurs sont nuls)
    if (encodeurs[0].delta_tick == 0 && encodeurs[1].delta_tick == 0 && encodeurs[2].delta_tick == 0 && encodeurs[3].delta_tick == 0)
        return;

    // Modèle cinématique : calcul des déplacements linéaires et angulaires
    // dx et dy sont les déplacements linéaires dans le repère local du robot
    // dtheta est le déplacement angulaire
    float dx = ODOMETRY_DXY_FACTOR * (-encodeurs[0].delta_tick - encodeurs[1].delta_tick + encodeurs[2].delta_tick + encodeurs[3].delta_tick);
    float dy = ODOMETRY_DXY_FACTOR * (+encodeurs[0].delta_tick - encodeurs[1].delta_tick - encodeurs[2].delta_tick + encodeurs[3].delta_tick);
    float dtheta = ODOMETRY_DTHETA_FACTOR * (encodeurs[0].delta_tick + encodeurs[1].delta_tick + encodeurs[2].delta_tick + encodeurs[3].delta_tick);

    // Mise à jour de l'orientation globale (theta) du robot
    position->theta += dtheta;

    // Mise à jour de la position globale (x, y) uniquement si le mouvement est significatif
    if (fabs(dx) > 0.00001 || fabs(dy) > 0.00001)
    {
        // Conversion des déplacements locaux (dx, dy) en coordonnées globales en utilisant l'orientation actuelle (theta)
        float cosTheta = cosf(position->theta);
        float sinTheta = sinf(position->theta);
        position->x += cosTheta * dx - sinTheta * dy;
        position->y += cosTheta * dy + sinTheta * dx;
    }

    // Normalisation de l'angle pour qu'il reste dans l'intervalle [-PI, PI]
    if (position->theta > M_PI)
        position->theta -= M_TWOPI;
    else if (position->theta < -M_PI)
        position->theta += M_TWOPI;
}
```

### Erreurs courantes et solutions

Le **glissement des roues**, notamment avec des roues suédoises, peut entraîner des **imprécisions dans l'odométrie** en raison de leur tendance à glisser latéralement. Pour atténuer ce problème, il est conseillé de vérifier régulièrement l'état des roues et d'optimiser leur adhérence.

Une **calibration rigoureuse** des paramètres mécaniques est également essentielle. Les valeurs du **rayon des roues ($r$)** et de **l'empattement ($L$)** doivent être mesurées avec précision, car toute erreur sur ces dimensions affecte directement la justesse des calculs de position et de vitesse.

Enfin, pour compenser la **dérive inhérente à l'odométrie**, il est recommandé d'utiliser une **approche de fusion de capteurs**. En combinant les données des codeurs avec des informations provenant de capteurs externes tels qu'une **IMU**, un **lidar** ou une **caméra**, on obtient une estimation plus robuste et précise de la position du robot.

## Ressources et Références

- [Ressources Odométrie](https://github.com/VRAC-team/la-maxi-liste-ressources-eurobot?tab=readme-ov-file#22-odom%C3%A9trie)

- Papiers de recherche sur les base holonomes à 4 roues :
    - [Development of Omni-Wheeled Mobile Robot Based-on Inverse Kinematics and Odometry](https://eprints.undip.ac.id/79371/1/C14-Development_of_Omni-Wheeled_Mobile_Robot_Based-on_Inverse_Kinematics_and_Odometry.pdf)
    - [Implementation Kinematics Modeling and Odometry of Four Omni Wheel Mobile Robot on The Trajectory Planning and Motion Control Based Microcontroller](https://journal.umy.ac.id/index.php/jrc/article/view/10664/5908)
