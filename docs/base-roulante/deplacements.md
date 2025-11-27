# Déplacements

---

- *Fichiers : `asserv.c`, `asserv.h`, `moteurs.h`, `moteurs.c`*

Bienvenue dans ce chapitre  dédiée au déplacement du robot.

Ici, nous expliquons comment **combiner les modules précédents** pour permettre au robot de se déplacer **de manière fluide, rapide et précise** entre deux points :

- **Point A** : $(x_1, y_1, \theta_1)$
- **Point B** : $(x_2, y_2, \theta_2)$

**Objectif** : Assurer un **mouvement optimisé** en utilisant les outils d'odométrie et d'asservissement vu précédement.

> [!IMPORTANT]
> Ce chapitre se concentre **uniquement sur l’explication du déplacement point à point**.
> Le **déplacement multi-points** est, quant à lui, traité dans les chapitres dédiées aux **trajectoires** et aux **trajets**.
> Cependant, plusieurs concepts abordés ici y sont **réutilisés**.

## Définition du problème

On souhaite pouvoir se déplacer de notre position actuelle à une position de cible de manière rapide et précise. Le problème peut alors se décomposer de la façon suivante :

- **Position actuelle** : $(x, y, \theta)$
- **Position cible** : $(x_{cible}, y_{cible}, \theta_{cible})$
- **Objectif** : Calculer les vitesses des roues à chaque "instant" pour atteindre la position cible.

## Calcul de l’erreur de position et d’orientation

Pour mettre en œuvre cet asservissement, il est utile de considérer le problème comme **un problème de correction** :

L’objectif est de **corriger l’erreur de position** du robot par rapport à sa **position cible**.

Pour cela, la première étape consiste à **calculer :**

- **l’erreur de position** (en $X$ et $Y$),
- **l’erreur d’orientation** (θ),

du robot **dans l’espace global**, par rapport à la cible visée.

> [!INFO]
> Bien entendu, **le calcul de l’erreur dépend des axes de mouvement disponibles** pour le robot.
>
> Par exemple, pour un **drone**, il faudrait également prendre en compte **tous ses degrés de liberté supplémentaires** (altitude, roulis, tangage, etc.).

**Erreur de position** :

$$
\Delta x = x_{cible} - x
$$

$$
\Delta y = y_{cible} - y
$$

**Erreur d’orientation** :

$$
\Delta \theta = \theta_{cible} - \theta
$$

> [!WARNING]
> N'oubliez pas de normaliser $\Delta \theta$ dans l’intervalle $[-π, π]$ pour éviter les rotations inutilement longues (ou dans le mauvais sens).

Il est également possible de calculer **l’erreur de distance** afin de déterminer notre éloignement par rapport à la cible dans l’espace global. Pour cela, il suffit d’utiliser la **distance euclidienne** :

$$
d = \sqrt{(\Delta x)^2 + (\Delta y)^2}
$$

## Contrôle des vitesses

### Asservissement en position

En ce qui concerne **l’asservissement en position de notre base roulante**, deux approches principales s’offrent à nous :

1. **Asservir la base en distance et en orientation** (approche polaire).
2. **Asservir la base selon tous ses degrés de liberté** (approche cartésienne) : en $X$, $Y$ et $\theta$.

La **seconde option** semble plus flexible, car elle permet un **contrôle indépendant sur chaque axe**, offrant ainsi une **meilleure précision** et une **adaptabilité accrue** selon les besoins du déplacement. Cependant, elle nécessite un asservissement (PID dans notre cas) indépendant pour chacun, ce qui fait d'autant plus de coefficients à calibrer.

> [!NOTE]
> Nous avons opté pour la **première approche**, principalement en raison des **contraintes de temps et de complexité du code**.
>
> Cependant, il est important de souligner que la **seconde méthode** (asservissement en $X$, $Y$ et $\theta$) serait, selon moi, plus adaptée

Utilisez un contrôleur proportionnel (P) ou PID pour générer les vitesses $V_x$, $V_y$, et $\Omega$ en fonction des erreurs :

$$
V_x = K_p \cdot \Delta x \cdot \cos(\theta) + K_p \cdot \Delta y \cdot \sin(\theta)
$$

$$
V_y = -K_p \cdot \Delta x \cdot \sin(\theta) + K_p \cdot \Delta y \cdot \cos(\theta)
$$

$$
\Omega = K_{\theta} \cdot \Delta \theta
$$

- $K_p$ : Gain proportionnel pour la position.
- $K_{\theta}$ : Gain proportionnel pour l’orientation.

> [!NOTE] Remarque
> Pour un mouvement purement holonome, vous pouvez aussi utiliser un contrôleur indépendant pour $V_x$, $V_y$, et $\Omega$.

## Transformation en vitesses des roues

Utilisez la matrice inverse de la cinématique pour convertir $V_x$, $V_y$, et $\Omega$ en vitesses des roues $\omega_i$ :

$$
\begin{bmatrix}
\omega_1 \\
\omega_2 \\
\omega_3 \\
\omega_4
\end{bmatrix}
=
\frac{1}{r}
\begin{bmatrix}
-\frac{1}{\sqrt{2}} & \frac{1}{\sqrt{2}} & -L \\
\frac{1}{\sqrt{2}} & \frac{1}{\sqrt{2}} & L \\
\frac{1}{\sqrt{2}} & -\frac{1}{\sqrt{2}} & -L \\
-\frac{1}{\sqrt{2}} & -\frac{1}{\sqrt{2}} & L
\end{bmatrix}
\begin{bmatrix}
V_x \\
V_y \\
\Omega
\end{bmatrix}
$$

> [!EXAMPLE] Implémentation en C - Cinématique Inverse (`moteurs.c`)
> Cette fonction convertit les vitesses globales du robot (vx, vy, ω) en vitesses individuelles pour chaque moteur.
>
> Les coefficients `0.35355` correspondent à une approximation de `1/√2`, typiquement utilisée pour les robots à roues omnidirectionnelles (ex: configuration Mecanum).
>
> ```c
> /**
>  * Calcule la cinématique inverse pour convertir les vitesses du robot (vx, vy, omega)
>  * en vitesses individuelles pour chaque moteur.
>  *
>  * @param robot_speed  Tableau contenant les vitesses du robot :
>  *                      [0] = vx (vitesse linéaire selon l'axe x),
>  *                      [1] = vy (vitesse linéaire selon l'axe y),
>  *                      [2] = omega (vitesse angulaire).
>  * @param motor_speed  Tableau de sortie contenant les vitesses des 4 moteurs.
>  */
> void inverse_kinematic(float *robot_speed, float *motor_speed)
> {
>     // Variables statiques pour les calculs intermédiaires
>     static float diff_vxy, plus_vxy, thetaScaled;
>
>     // Calcul de la différence et de la somme des vitesses linéaires
>     diff_vxy = robot_speed[0] - robot_speed[1];  // vx - vy
>     plus_vxy = robot_speed[0] + robot_speed[1];  // vx + vy
>
>     // Mise à l'échelle de la vitesse angulaire
>     thetaScaled = robot_speed[2] * DISTANCE_ROUES_CENTRE;
>
>     // Application des formules de cinématique inverse
>     motor_speed[0] = INV_RAYON_ROUE * ((-0.35355 * diff_vxy) + thetaScaled);
>     motor_speed[1] = INV_RAYON_ROUE * ((-0.35355 * plus_vxy) + thetaScaled);
>     motor_speed[2] = INV_RAYON_ROUE * ((+0.35355 * diff_vxy) + thetaScaled);
>     motor_speed[3] = INV_RAYON_ROUE * ((+0.35355 * plus_vxy) + thetaScaled);
> }
> ```
>
> **Remarques :**
>
> - `INV_RAYON_ROUE` = 1/rayon des roues (pour convertir en vitesse de rotation).
> - `DISTANCE_ROUES_CENTRE` = distance entre les roues et le centre du robot.
> - Les variables `diff_vxy` et `plus_vxy` sont déclarées `static` pour optimiser les calculs répétés.

## Implémentation pratique

### Boucle de contrôle

1. **Mesurer** la position et l’orientation actuelles $(x, y, \theta)$.
2. **Calculer** les erreurs $\Delta x$, $\Delta y$, et $\Delta \theta$.
3. **Générer** les vitesses $V_x$, $V_y$, et $\Omega$ avec le contrôleur.
4. **Convertir** $V_x$, $V_y$, et $\Omega$ en vitesses des roues $\omega_i$.
5. **Envoyer** les consignes de vitesse aux moteurs.
6. **Répéter** jusqu’à ce que les erreurs soient inférieures à un seuil défini.

### Asservissement en Position

Cette fonction gère l'asservissement en position d'un robot pour atteindre une cible ($x$, $y$, $\theta$). Elle utilise des correcteurs PID pour la distance et l'angle, et applique une cinématique inverse pour contrôler les moteurs.

> [!EXAMPLE] **Implémentation en C - Asservissement en Position 🎯**
>
> ```c
> /**
>  * Asservit le robot pour atteindre une cible (x, y, θ) en utilisant des PID.
>  *
>  * @param robot       Structure contenant les paramètres du robot.
>  * @param target       Structure décrivant la cible (position et état).
>  * @param position     Position actuelle du robot.
>  * @param pid_distance Correcteur PID pour la distance.
>  * @param pid_theta    Correcteur PID pour l'angle.
>  * @param pid_moteurs  Tableau de correcteurs PID pour chaque moteur.
>  * @param encodeurs    Tableau des encodeurs pour mesurer les déplacements.
>  */
> void Asserv_Position(Robot *robot, Target *target, Position *position, PID *pid_distance, PID *pid_theta, PID pid_moteurs[4], Encoder encodeurs[4])
> {
>     // Sortie anticipée en cas d'arrêt d'urgence ou si la cible est déjà atteinte
>     if (target->state == TARGET_EMERGENCY_STOP || target->state == TARGET_REACHED)
>         return;
>
>     // Variables statiques pour les calculs intermédiaires
>     static float Consigne[4];      // Consignes de vitesse pour les moteurs
>     static float robot_speed[3];   // Vitesse du robot (vx, vy, ω)
>     static float motor_speed[4];   // Vitesse des moteurs
>     static float cosTheta, sinTheta, angle_diff, max_v, vx, vy;
>
>     // --- Calcul des erreurs ---
>     vx = target->x - position->x;  // Erreur en X
>     vy = target->y - position->y;  // Erreur en Y
>     position->distance = hypotf(vx, vy);  // Distance euclidienne à la cible
>
>     // Calcul de l'erreur angulaire (normalisée dans [-π, π])
>     angle_diff = target->theta - position->theta;
>     angle_diff += (angle_diff > M_PI) ? -M_TWOPI : (angle_diff < -M_PI) ? M_TWOPI : 0;
>
>     // --- Calcul des correcteurs PID ---
>     compute_pid(pid_distance, 0.0, -position->distance, VMAX_PWM);  // PID pour la distance
>     compute_pid(pid_theta, 0.0, -angle_diff, 1);  // PID pour l'angle
>
>     // --- Mode "Atteinte de la cible" ---
>     if (fabs(pid_distance->output) > 10 && target->state != TARGET_ADJUST_ANGLE)
>     {
>         if (target->state != TARGET_REACHING)
>         {
>             target->state = TARGET_REACHING;
>             target->previous_state = TARGET_REACHING;
>             log_info("Position actuelle : (%.1f, %.1f, %.1f)", position->x, position->y, position->theta);
>             log_info("En route vers : (%.1f, %.1f, %.1f)", target->x, target->y, target->theta);
>         }
>
>         // Normalisation des vitesses pour éviter les dépassements
>         max_v = fmaxf(fabs(vx), fabs(vy));
>         cosTheta = cosf(position->theta);
>         sinTheta = sinf(position->theta);
>
>         // Conversion des erreurs (x, y) en vitesses locales (vx, vy)
>         robot_speed[0] = (cosTheta * vx + sinTheta * vy) / max_v;
>         robot_speed[1] = (cosTheta * vy - sinTheta * vx) / max_v;
>         robot_speed[2] = pid_theta->output;  // Vitesse angulaire
>
>         // Cinématique inverse pour obtenir les vitesses des moteurs
>         inverse_kinematic(robot_speed, motor_speed);
>         normalize_motor_speeds(motor_speed);  // Normalisation des vitesses
>
>         // Application de la sortie du PID de distance
>         Consigne[0] = motor_speed[0] * pid_distance->output;
>         Consigne[1] = motor_speed[1] * pid_distance->output;
>         Consigne[2] = motor_speed[2] * pid_distance->output;
>         Consigne[3] = motor_speed[3] * pid_distance->output;
>     }
>     // --- Mode "Ajustement de l'angle" ---
>     else if (fabs(pid_theta->output) > 0.01)
>     {
>         if (target->state != TARGET_ADJUST_ANGLE)
>         {
>             target->state = TARGET_ADJUST_ANGLE;
>             target->previous_state = TARGET_ADJUST_ANGLE;
>             log_info("Ajustement de l'angle");
>             log_info("Position actuelle : (%.1f, %.1f, %.1f)", position->x, position->y, position->theta);
>         }
>         // Tous les moteurs tournent à la même vitesse pour corriger l'angle
>         Consigne[0] = pid_theta->output * VMAX_PWM;
>         Consigne[1] = pid_theta->output * VMAX_PWM;
>         Consigne[2] = pid_theta->output * VMAX_PWM;
>         Consigne[3] = pid_theta->output * VMAX_PWM;
>     }
>     // --- Cible atteinte ---
>     else
>     {
>         if (target->state == TARGET_REACHED || flag_ack == 1)
>             return;
>
>         target->state = TARGET_REACHED;
>         log_info("Cible atteinte !");
>         log_info("Position actuelle : (%.1f, %.1f, %.1f)", position->x, position->y, position->theta);
>
>         // Arrêt des moteurs
>         Consigne[0] = 0;
>         Consigne[1] = 0;
>         Consigne[2] = 0;
>         Consigne[3] = 0;
>         controle_moteurs(Consigne);
>         flag_ack = 1;  // Drapeau de confirmation
>         return;
>     }
>
>     // Asservissement en vitesse des moteurs
>     Asserv_Vitesse(target, Consigne, pid_moteurs, encodeurs, robot);
> }
> ```
>
> **Points clés :**
>
> - **Gestion des états** : Le robot peut être en mode "atteinte de la cible" ou "ajustement de l'angle".
> - **Normalisation** : Les vitesses sont normalisées pour éviter les dépassements.
> - **Cinématique inverse** : Convertit les vitesses globales (vx, vy, ω) en consignes pour chaque moteur.
> - **PID** : Utilisation de correcteurs PID pour la distance et l'angle.
> - **Logs** : Affichage des informations pour le débogage.
> - **Sortie anticipée** : Si la cible est atteinte ou en cas d'urgence.

### Asservissement en Vitesse

Cette fonction gère l'asservissement en vitesse des moteurs du robot en utilisant des correcteurs PID. Elle convertit les vitesses mesurées (en ticks) en valeurs PWM, calcule les commandes PID pour chaque moteur, et applique ces commandes aux moteurs.

> [!EXAMPLE] Implémentation en C - Asservissement en Vitesse ⚡
>
> ```c
> /**
>  * Asservit la vitesse des moteurs du robot en utilisant des correcteurs PID.
>  *
>  * @param target       Structure décrivant la cible et l'état actuel du robot.
>  * @param Consigne      Tableau des consignes de vitesse pour chaque moteur.
>  * @param pid_moteurs   Tableau de correcteurs PID pour chaque moteur.
>  * @param encodeurs     Tableau des encodeurs pour mesurer les vitesses réelles.
>  * @param robot         Structure contenant les paramètres du robot.
>  */
> void Asserv_Vitesse(Target *target, float Consigne[4], PID pid_moteurs[4], Encoder encodeurs[4], Robot *robot)
> {
>     float Commande[4];  // Tableau pour stocker les commandes finales des moteurs
>
>     // --- Conversion des vitesses (ticks → PWM) ---
>     // Conversion des variations de ticks en valeurs PWM pour chaque moteur
>     encodeurs[0].vitesse_pwm = M1_Ticks2PWM(encodeurs[0].delta_tick);
>     encodeurs[1].vitesse_pwm = M2_Ticks2PWM(encodeurs[1].delta_tick);
>     encodeurs[2].vitesse_pwm = M3_Ticks2PWM(encodeurs[2].delta_tick);
>     encodeurs[3].vitesse_pwm = M4_Ticks2PWM(encodeurs[3].delta_tick);
>
>     // --- Calcul des correcteurs PID pour chaque moteur ---
>     // Les PID comparent la consigne de vitesse à la vitesse réelle (PWM)
>     compute_pid(&pid_moteurs[0], Consigne[0], (float)encodeurs[0].vitesse_pwm, VMAX_PWM);
>     compute_pid(&pid_moteurs[1], Consigne[1], (float)encodeurs[1].vitesse_pwm, VMAX_PWM);
>     compute_pid(&pid_moteurs[2], Consigne[2], (float)encodeurs[2].vitesse_pwm, VMAX_PWM);
>     compute_pid(&pid_moteurs[3], Consigne[3], (float)encodeurs[3].vitesse_pwm, VMAX_PWM);
>
>     // --- Ajustement des commandes en fonction de l'état du robot ---
>     if (target->state == TARGET_ADJUST_ANGLE)
>     {
>         // Réduction des commandes à 65% pour un ajustement précis de l'angle
>         Commande[0] = pid_moteurs[0].output * 0.65;
>         Commande[1] = pid_moteurs[1].output * 0.65;
>         Commande[2] = pid_moteurs[2].output * 0.65;
>         Commande[3] = pid_moteurs[3].output * 0.65;
>     }
>     else
>     {
>         // Application directe des sorties PID pour les autres états
>         Commande[0] = pid_moteurs[0].output;
>         Commande[1] = pid_moteurs[1].output;
>         Commande[2] = pid_moteurs[2].output;
>         Commande[3] = pid_moteurs[3].output;
>     }
>
>     // --- Application des commandes aux moteurs ---
>     controle_moteurs(Commande);  // Envoie les commandes aux moteurs
> }
> ```
>
> **Points clés :**
>
> - **Conversion ticks → PWM** : Les encodeurs mesurent les déplacements en ticks, qui sont convertis en valeurs PWM pour le contrôle des moteurs.
> - **Correcteurs PID** : Chaque moteur a son propre PID pour réguler sa vitesse.
> - **Réduction des commandes** : En mode `TARGET_ADJUST_ANGLE`, les commandes sont réduites à 65% pour un ajustement plus précis.
> - **Application des commandes** : La fonction `controle_moteurs` envoie les valeurs finales aux moteurs.

## Optimisations et précautions

Pour améliorer le code, voici quelques pistes à explorer :

- **Limitation des vitesses** : Assurez-vous que les vitesses des roues ne dépassent pas les limites mécaniques du robot pour éviter les glissements ou les dommages aux moteurs.

- **Filtrage des consignes** : Appliquez un **filtre passe-bas** sur les vitesses $V_x$, $V_y$, et $\Omega$ pour lisser les mouvements et éviter les à-coups, surtout à haute vitesse.

- **Arrêt progressif** : Réduisez progressivement les gains $K_p$ et $K_{\theta}$ lorsque le robot approche de la cible pour éviter les dépassements et les oscillations.

## Cas particuliers et extensions

### Contrôle de l’orientation seule

Si seul l’angle $\theta_{cible}$ est prioritaire (par exemple, pour aligner le robot avant un mouvement) :

- Annulez $V_x$ et $V_y$.
- Contrôlez uniquement $\Omega$ avec un asservissement en orientation : $\Omega = K_{\theta} \cdot \Delta \theta$

### Trajectoire courbe ou complexe

Il est possible de réaliser des **mouvements plus complexes** en utilisant la gestion des trajectoires. Pour plus de détails sur ce sujet, je vous invite à consulter le **chapitre suivant**, qui y est entièrement dédié.

## Ressources et Références

- [Asservissement - Eurobot](https://github.com/VRAC-team/la-maxi-liste-ressources-eurobot?tab=readme-ov-file#25-asservissement)
- [Asservissement et pilotage de robot autonome](https://www.pm-robotix.eu/2022/02/02/asservissement-et-pilotage-de-robot-autonome/)
