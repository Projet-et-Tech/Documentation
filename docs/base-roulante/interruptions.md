# Interruptions

---

- *Fichiers : `main.c` et `*.ioc`*

Bienvenue dans ce chapitre dédié aux **interruptions**, un mécanisme simple en apparence, mais **essentiel** à maîtriser dans la programmation des systèmes embarqués.

> [!NOTE]
> Bien que ce chapitre mette particulièrement en avant leur utilité dans les systèmes embarqués, les interruptions sont également présentes dans les appareils électroniques du quotidien, comme les ordinateurs et les téléphones. Elles y jouent un également un rôle clé, notamment dans la **communication avec les périphériques externes**.
>
> Tout ça pour dire que, en soit, ça n'a rien de spécifique aux systèmes embarqués, et que c'est en fait un mécanisme essentiel à la plupart de nos sytèmes numériques.

## Qu’est-ce qu’une interruption ?

Une **interruption** est un mécanisme permettant à un microprocesseur (comme celui que nous utilisons) de **réagir instantanément** à un événement, qu'il soit interne ou externe, **sans avoir à le surveiller en continu**.

Comme son nom l'indique, ce mécanisme fonctionne en **interrompant temporairement** le programme actuellement en cours d'exécution.

### Pourquoi utiliser des interruptions ?

1. **Efficacité** :
      - Sans interruption, le microprocesseur doit **vérifier en boucle** si un événement s’est produit (ex. : un bouton pressé, une donnée reçue).
      - Avec une interruption, le microprocesseur est **alerté automatiquement** quand l’événement se produit, ce qui économise du temps et de l’énergie.

2. **Réactivité** :
      - Certaines actions nécessitent une réponse **immédiate** (ex. : un capteur détecte un obstacle, un encodeur envoie une impulsion).
      - Les interruptions permettent une **réaction en temps réel**, cruciale pour des systèmes comme une carte moteur.

3. **Simplification du code** :
      - Au lieu d’écrire des boucles pour surveiller des événements, on **délègue cette tâche au matériel**, ce qui rend le code plus simple et plus lisible.

### Fonctionnement technique

1. Un **événement** se produit (ex. : impulsion d’un encodeur).
2. Le **STM32 déclenche une interruption** et met en pause le programme en cours.
3. Le microprocesseur exécute une **routine d’interruption (ISR)** pour traiter l’événement.
4. Une fois l’ISR terminée, le programme **reprends là où il s’était arrêté**.

## Implémentation

### Routine d'interruption dans `main.c`

Dans le fichier **`main.c`**, on trouve une **routine d'interruption (ISR)** nommée **`HAL_TIM_PeriodElapsedCallback`** (nom par défaut). Cette fonction est appelée automatiquement **lorsque le timer atteint la fin de sa période**.
> [!TIP] Noms des routines d'interruption
> Pour connaître les **noms par défaut** des routines d'interruption associées à des **événements spécifiques**, consultez :
>
> - Le fichier **`stm32f4xx_it.c`** dans votre projet,
> - Ou la **documentation officielle** de votre microcontrôleur STM32.

Dans notre projet, ce timer est utilisé pour temporiser l’asservissement.

> [!NOTE]
> Dans notre cas, la période du timer est configurée à **10 ms**. Cela signifie qu’une interruption est déclenchée **toutes les 10 ms**, appelant ainsi notre fonction `HAL_TIM_PeriodElapsedCallback` à intervals réguliers.

### Description de la routine d'interruption

```c
volatile uint8_t flag_interrupt = 0; // Variable globale

void HAL_TIM_PeriodElapsedCallback(TIM_HandleTypeDef *htim)
{
  // Interruption déclenchée toutes les DELTA_T ms
  if (htim->Instance == TIM5) // 1.
  {
    flag_interrupt = 1; // 2.
    // [...]
  }
}
```

1. **Vérification du timer** :

      - Lors de l’appel de cette fonction, on vérifie d’abord que c’est bien le **bon timer (TIM5)** qui a déclenché l’interruption. Cette vérification est nécessaire car **plusieurs timers** peuvent partager la même routine d’interruption par défaut.

      - Ici, **l’asservissement est synchronisé avec l’interruption de notre timer principal (TIM5)**.

1. **Modification du drapeau d’interruption** :

      - On modifie la variable globale **`flag_interrupt`** pour indiquer qu’une interruption vient de se produire.

      - Cette variable sera ensuite lue par la **boucle principale** pour exécuter les actions nécessaires.

> [!CAUTION] À éviter
> En règle générale, **on ne place jamais** (sauf pour du débogage ou des tests ponctuels) le code principal que l'on souhaite éxecuter, directement dans la routine d’interruption.

#### Pourquoi ?

> [!NOTE] À retenir
> Une interruption **interrompt la boucle principale** du programme. Pour minimiser l’impact sur les performances, il est crucial que **le temps passé dans l’interruption soit le plus court possible**.
> On limite donc au maximum les opérations dans l’ISR.
>
> **Solution courante** :
>
> Plutôt que d’exécuter des tâches longues dans l’interruption, on se contente de **modifier une variable globale** (comme `flag_interrupt`). La boucle principale du programme **vérifie régulièrement cette variable** et exécute les actions nécessaires **en dehors de l’interruption**.
>
> Cela permet de **garder l’ISR rapide** tout en assurant que les tâches critiques (comme l’asservissement) sont exécutées **au bon moment**.

### Configuration des interruptions dans STM32CubeIDE

Pour configurer les **interruptions** :

1. Ouvrez le fichier **.ioc** dans STM32CubeIDE.
2. Dans l'onglet **"Pinout & Configuration"**, accédez au sous-menu **"System Core"**.
3. Sélectionnez **"NVIC"** (Nested Vectored Interrupt Controller).
4. Vous y trouverez :
      - La liste des **périphériques et interruptions** associés.
      - La possibilité d'**activer/désactiver** chaque interruption.
      - Le réglage des **niveaux de priorité**.

> [!IMPORTANT]
> Concernant les **niveaux de priorité** dans NVIC :
>
> - **0** correspond à la **priorité la plus élevée** (la plus prioritaire).
> - Plus le chiffre **augmente**, plus la priorité **diminue**.

## Cas d'utilisation pour la carte moteur

Voici quelques cas d’utilisation des interruptions sur une carte moteur :

| **Événement**               | **Utilisation de l’interruption**                                                                        |
| --------------------------- | -------------------------------------------------------------------------------------------------------- |
| **Encodeurs de moteur**     | Compter les impulsions pour calculer la vitesse ou la position **sans ralentir le programme principal**. |
| **Temporisateurs (Timers)** | Mettre à jour la vitesse des moteurs **toutes les 1 ms** pour un contrôle précis.                        |
| **Communication (UART)**    | Traiter immédiatement les données reçues **sans attendre une vérification manuelle**.                    |
| **Boutons ou capteurs**     | Arrêter les moteurs **instantanément** en cas d’urgence (Ex: Bouton d'arrêt d'urgence).                  |

## Ressources et Références

- [STM32 Interrupts Tutorial](https://deepbluembedded.com/stm32-interrupts-tutorial-nvic-exti/) (le tutoriel en question)
- [Getting started with TIM](https://wiki.st.com/stm32mcu/wiki/Getting_started_with_TIM)
