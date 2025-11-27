# Correcteur PID

---

- *Fichiers : `pid.c` et `pid.h`*

Bienvenue dans ce chapitre consacré au **correcteur PID**, utilisé pour l'asservissement des déplacements de notre robot.

Mon objectif, à travers les paragraphes qui suivent, est de vous transmettre **l'essentiel** à connaître sur ce sujet et de vous expliquer pourquoi nous l'utilisons.

> [!WARNING]
> N'oubliez pas que le **PID** — et l'**asservissement** de manière générale — est un **domaine extrêmement vaste**. Pour ma part, je suis encore **loin d'en maîtriser tous les aspects** et vous encourage à vous renseignez d'avantage sur le sujet si vous voulez en savoir plus.

## Qu’est-ce qu’un correcteur PID ?

Un **correcteur PID** (Proportionnel-Intégral-Dérivé) est un algorithme utilisé pour **contrôler automatiquement un système** (ex. : température, vitesse, position) en ajustant en temps réel une **action correctrice** (ex. : chauffage, freinage, accélération).

Il est largement utilisé dans l’**asservissement** : une boucle de régulation où le système est **mesuré en continu** et **corrigé** pour atteindre une **consigne** (valeur souhaitée).

> [!INFO] Vocabulaire
> **Petit lexique d'asservissement** :
>
> - **Consigne** : Valeur **souhaitée** (objectif à atteindre).
> - **Mesure** : Valeur **actuelle** (captée par les capteurs).
> - **Commande** : Valeur **calculée en sortie du PID** (action corrective).
> - **Erreur** : Différence **signée** entre la **consigne** et la **mesure** (*Erreur = Consigne - Mesure*).

## À quoi sert-il ?

Le PID permet de :

- **Stabiliser un système** (éviter les oscillations ou les dépassements).
- **Réduire l’erreur** entre la consigne et la mesure réelle.
- **Réagir rapidement** aux perturbations (ex. : vent pour un drone, charge pour un moteur).
- **Optimiser la précision et la rapidité** de la réponse.

**Exemples d’applications** :

- Régulation de température (four, climatisation).
- Contrôle de vitesse (voiture autonome, ascenseur).
- Positionnement (bras robotique, imprimante 3D).

## Comment fonctionne-t-il ?

Le PID combine **trois actions** pour calculer la correction à appliquer :

| Action                | Rôle                                                                        | Exemple (régulation de température)                                                    |
| --------------------- | --------------------------------------------------------------------------- | -------------------------------------------------------------------------------------- |
| **P (Proportionnel)** | Agit **proportionnellement à l’erreur actuelle** (consigne – mesure).       | Si la température est trop basse, chauffer plus fort.                                  |
| **I (Intégral)**      | Corrige **l’erreur accumulée dans le temps** (élimine l’erreur résiduelle). | Si le système reste froid trop longtemps, augmenter progressivement la puissance.      |
| **D (Dérivé)**        | Anticipe **la tendance future de l’erreur** (freine les dépassements).      | Si la température monte trop vite, réduire le chauffage avant de dépasser la consigne. |

**Formule du PID** :

$$
\text{Action} = K_p \cdot \text{Erreur} + K_i \cdot \int \text{Erreur} \, dt + K_d \cdot \frac{d(\text{Erreur})}{dt}
$$

Avec

$$
Erreur = Consigne - Mesure
$$

*($K_p$, $K_i$, $K_d$ sont des coefficients à régler selon le système.)*

## Réglage des Coefficients PID

Le **réglage des coefficients** $K_p$ (proportionnel), $K_i$ (intégral) et $K_d$ (dérivé) est **l’étape la plus cruciale** pour un asservissement PID efficace.

**Pourquoi ?**

- Ces coefficients déterminent la **stabilité**, la **réactivité** et la **précision** de votre système.
- Un **mauvais choix** peut entraîner des oscillations, une réponse trop lente, ou même une instabilité totale.
- Un **réglage optimal** garantit un comportement **rapide, précis et stable** de votre asservissement.

### Méthode manuelle

> [!TIP]
> Commencez par ajuster $K_p$ pour obtenir une réponse rapide, puis affinez avec $K_i$ et $K_d$ pour éliminer les erreurs résiduelles et les dépassements.

### Méthode automatique

> [!INFO]
>
> Il existe des **méthodes mathématiques** (comme la méthode de Ziegler-Nichols, la réponse indicielle, ou l'optimisation par placement de pôles) qui permettent de **calculer des valeurs initiales** pour les coefficients $K_p$, $K_i$ et $K_d$.
>
> Ces méthodes fournissent un **point de départ fiable**, censé offrir des **résultats acceptables** dès le premier essai.
> Elles sont particulièrement utiles pour **gagner du temps** et éviter un réglage manuel long et fastidieux.

## Implémentation

> [!NOTE]
> L'**implémentation pratique** du PID dans le code diffère de la **théorie pure** présentée dans la plupart des ressources en ligne et ici même.
> En effet, elle intègre :
>
> - Des **optimisations** pour améliorer les performances.
> - Des **modifications** dans les calculs pour éviter les **effets indésirables** liés à la nature **discrète** (et non continue) et **numérique** des systèmes embarqués.
>
> Ces adaptations permettent de contourner des problèmes comme :
>
> - Les **dépassements numériques** (overflow).
> - Les **pic de dérivée** (Derivative Kick).
> - … ainsi que d’autres éléments plus spécifiques qui ne nous concernent pas nécessairement.
>
> Pour approfondir ce sujet, je vous recommande cette **[excellente ressource](https://www.pm-robotix.eu/2022/01/19/ameliorer-vos-regulateurs-pid/)** qui détaille ces modifications.

La fonction de calcul du PID est **générique** : elle peut s'appliquer à **n'importe quel type de consigne** et avec **n'importe quels paramètres**. Cela permet une **réutilisation flexible** du code pour différents cas d'usage.

Dans l'ordre, voici ce que fait la fonction `compute_pid(...)` :

1. Vérifie si le PID est en mode manuel; si oui, quitte la fonction.
2. Calcule l’erreur : consigne − mesure d’entrée.
3. Si $K_i = 0$ et $K_d = 0$, applique uniquement le correcteur proportionnel.
4. Sinon (mode PID complet) :
      1. Calcule l’erreur dérivée.
      2. Met à jour l’erreur précédente.
      3. Intègre l’erreur pondérée (terme intégral).
      4. Applique l’anti-windup sur l’intégrale.
      5. Calcule la commande PID : $P + I − D$.
5. Si $K_c ≠ 0$, applique le correcteur QSM supplémentaire.
6. Limite la sortie finale entre $−output_{Limit}$ et $+output_{Limit}$

## Structure des Paramètres PID

Les **paramètres du PID** sont initialisés **au début du programme** pour tous les cas d'utilisation.

**Initialisation** : `mainprog.c`

```c
// [...]

// Initialisation des PIDs
init_pid(&pid_moteurs[0], 1, 0.15, 0.4, 0);
init_pid(&pid_moteurs[1], 1, 0.15, 0.4, 0);
init_pid(&pid_moteurs[2], 1, 0.15, 0.4, 0);
init_pid(&pid_moteurs[3], 1, 0.15, 0.4, 0);
init_pid(&pid_distance, 2, 0, 0, 0);
init_pid(&pid_theta, 4, 0, 0, 0);

// [...]
```

Ils sont ensuite passés en argument à la fonction via une **structure `PID`**, définie dans le fichier `pid.h` :

```c
typedef struct {
  float Ki;           // Coefficient intégral
  float Kp;           // Coefficient proportionnel
  float Kd;           // Coefficient dérivé
  float Kc;           // Coefficient pour le QSM
  float Ep;           // Erreur proportionnelle
  float Ei;           // Erreur intégrale
  float Ed;           // Erreur dérivée
  float output;       // Dernière commande calculée
  float error;        // Valeur présente de la mesure
  float maxIntegral;  // Borne supérieure pour l'erreur intégral
  float maxOutput;    // Borne supérieure pour la commande
  uint8_t mode;       // Mode : Automatique (1) ou Manuel (0)
  uint16_t sampleTime;// Pas de temps pour le calcul (en ms)
} PID;
```

### Points clés

Dans notre cas, l'asservissement est **appelé à intervalles réguliers** (par exemple, toutes les 10 ms).
Cela implique que la composante temporelle peut être **intégrée directement** dans les coefficients $K_p$, $K_i$ et $K_d$.

- **Avantage** : réduction du nombre de calculs et d'opérations nécessaires.
- **Contrainte** : si le pas de temps varie, les coefficients doivent être **recalculés** pour conserver la même dynamique de contrôle (via la fonction `set_tuning`).

## Pour aller plus loin

Le sujet du **PID**, de son **implémentation** et du **choix des coefficients** est **vaste**, avec une multitude d’informations techniques et théoriques.

J’ai tenté d’aborder **l’essentiel** ci-dessus, de manière claire et concise, en me concentrant sur les aspects pratiques pour les systèmes embarqués.

Si vous souhaitez **approfondir vos connaissances**, je vous encourage vivement à :

- **Faire vos propres recherches** pour explorer des cas d’usage spécifiques.
- **Consulter des articles scientifiques** et des publications techniques sur le sujet.

Voici quelques **sources fiables** pour commencer vos lectures :

- [Comparison of PID Controller Tuning Methods](https://www.scribd.com/document/348248436/Zomorrodi-Shahrokhi-PID-Tunning-Comparison-pdf)
- [Implémenter un PID sans faire de calculs !](https://www.ferdinandpiette.com/blog/2011/08/implementer-un-pid-sans-faire-de-calculs/)

## Ressources et Références

- [PID Explained](https://pidexplained.com/)
- [Wikipedia - PID](https://en.wikipedia.org/wiki/Proportional%E2%80%93integral%E2%80%93derivative_controller)
