# Communications

---

- *Fichiers : `uart.c` `uart.h`*

Bienvenue dans ce chapitre dédié à la **communication**, où nous allons expliquer comment le code utilise les différents protocoles de communication pour interagir avec les capteurs et les autres éléments du robot.

Vous découvrirez :

- **À quoi elle sert** dans le contexte de notre système.
- **Comment nous l'utilisons** pour interagir avec les autres composants du robot et les capteurs associés.

## Les différents protocoles

Il existe plusieurs **protocoles de communication série point à point**, chacun présentant ses propres avantages, inconvénients et cas d'utilisation.

Voici un **tableau récapitulatif** des plus répandus, ainsi que ceux que nous utilisons pour les différentes communications au sein de notre robot :

| Protocole | Description                                                          | Avantages                        | Inconvénients                                        | Exemples d’utilisation                           |
| --------- | -------------------------------------------------------------------- | -------------------------------- | ---------------------------------------------------- | ------------------------------------------------ |
| UART      | Communication asynchrone, full-duplex, sans horloge partagée.        | Simple, peu coûteux, flexible.   | Pas de détection d’erreur intégrée, vitesse limitée. | Debugging, communication entre microcontrôleurs. |
| I2C       | Bus série synchrone, multi-maître, multi-esclave, 2 fils (SDA, SCL). | Peu de fils, adressage multiple. | Vitesse limitée, complexité en multi-maître.         | Capteurs, EEPROM, écrans LCD.                    |
| SPI       | Bus série synchrone, full-duplex, 4 fils (MOSI, MISO, SCK, SS).      | Rapide, simple.                  | Pas d’adressage natif, nécessite plus de fils.       | Mémoires flash, capteurs, cartes SD.             |

## Configuration UART sur STM32

1. **Ouvrir le fichier `.ioc`**
2. **Activer l'UART**
      - Allez dans l'onglet **"Pinout & Configuration"** → **"Connectivité"** → Sélectionnez **USART** (exemple : USART1, USART2).

3. **Configurer les broches**
      - Choisissez le mode **"Asynchrone"**.
      - Sélectionnez les broches **TX** et **RX** (exemple : PA9 et PA10 pour USART1).
      - Assurez-vous que les broches sont configurées en mode **"Fonction alternative"**.

4. **Paramétrer l'UART**
      - **Débit en bauds** : 9600, 115200, etc.
      - **Longueur de mot** : **8 bits**.
      - **Bits de stop** : **1 bit**.
      - **Parité** : **Aucune** (0 bit de parité).
      - **Contrôle de flux matériel** : Désactivé (sauf besoin spécifique).

5. **Activer les interruptions (optionnel)**
      - Allez dans l'onglet **"Paramètres NVIC"** → Activez **Interruption globale USART** si nécessaire.

6. **Générer le code**

> [!NOTE]
> Pour plus de détails ou fonctionnalités avancées, consultez les tutoriels sur le wiki officiel de STM32.

### Choix des paramètres / valeurs

#### Choix du Baudrate

Le **baudrate** (ou débit en bauds) définit la **vitesse de transmission des données** sur une liaison série comme l'UART. Il correspond au **nombre de symboles transmis par seconde**. Voici les critères à prendre en compte pour le choix de celui-ci :

1. **La Compatibilité** :
      - Le baudrate doit être **identique** entre l'émetteur et le récepteur.
      - Exemple : Si votre STM32 utilise **115200 bauds**, configurez votre terminal série (PuTTY, Tera Term, etc.) avec la même valeur.

2. **Les Valeurs standard** :
      - **9600 bauds** : Pour des communications simples ou des périphériques lents.
      - **115200 bauds** : Standard pour les applications embarquées (équilibre entre vitesse et stabilité).
      - **1 Mbps ou plus** : Pour des besoins en haut débit (ex : transmission vidéo), mais nécessite un matériel adapté.

> [!TIP] Choix du baudrate
> Dans notre projet, nous privilégions le **baudrate de 115200**, une valeur standard largement adoptée. Ce choix garantit :
>
> - Une **excellente compatibilité** avec la plupart des périphériques.
> - Un **équilibre optimal** entre rapidité et stabilité de la transmission.
>
> De plus, il s'agit du **baudrate maximal** supporté par notre **Raspberry Pi** pour communiquer avec nos cartes STM32.

#### Choix des autres paramètres

Pour les autres paramètres de configuration, nous optons pour **la configuration la plus simple** :

- **1 bit de stop**,
- **0 bit de parité**,
- des **mots de 8 bits**.

Cette configuration permet une **transmission octet par octet**, sans surcharge inutile. Le **bit de stop** et la **parité** ne sont pas nécessaires dans notre cas, car les risques d'erreur sont **négligeables** : la communication s'effectue via des **câbles courts** entre notre carte et les autres périphériques, ce qui limite considérablement les perturbations.

### Fonctions de formattage

> [!BUG]
> TODO ...

## Configuration I2C sur STM32

> [!NOTE]
> Pour plus de détails ou fonctionnalités avancées, consultez les tutoriels sur le wiki officiel de STM32.

## Configuration SPI sur STM32

> [!NOTE]
> Pour plus de détails ou fonctionnalités avancées, consultez les tutoriels sur le wiki officiel de STM32.

## Implémentation

### Initialisation

Pour activer notre système de réception, nous utilisons la fonction **`HAL_UART_Receive_IT()`**. Celle-ci permet de **créer un "hook"** (ou point d'accroche) sur l'événement d'interruption lié à la réception de données. Grâce à notre buffer **`data_recvd`**, cette fonction déclenche notre **routine d'interruption** pour traiter les données reçues.

> [!TIP] Remarque
> Vous remarquerez que cette fonction est **rappelée dans la routine d'interruption** elle-même. Cela permet de **réactiver le "hook"** et de préparer le système à recevoir de nouvelles données dès que possible.

```c
void init_uart()
{
  // Initialisation de la réception UART en mode interruption
  HAL_UART_Receive_IT(&huart1, &data_recvd, 1);
  HAL_UART_Receive_IT(&huart6, &data_recvd, 1);

  log_debug("OK - (UART_INIT)");
}
```

### Réception

```c
void HAL_UART_RxCpltCallback(UART_HandleTypeDef *huart)
{
  if (huart->Instance == USART1)
  {
    if (data_recvd == '\r')
    {
      RX_buff_1[counter++] = '\0';
      counter = 0;
      flag_uart = 1; // Set flag to indicate data received
    }
    else
    {
      RX_buff_1[counter++] = data_recvd;
    }

    HAL_UART_Receive_IT(&huart1, &data_recvd, 1);
  }

  // [...]
}
```

Pour recevoir les données **sans impacter la boucle principale du programme**, une **interruption est déclenchée à chaque réception d’un caractère**. Le message n’est traité qu’une fois **complètement reçu**, c’est-à-dire après la détection du **caractère de fin de transmission** (octet `0`).

En attendant, **tous les octets du message sont stockés dans une mémoire tampon**. Cela permet d’accéder au message **dans son intégralité** et de le décoder, puis le traiter en conséquences, une fois la réception terminée.

### Transmission

```c

void HAL_UART_TxCpltCallback(UART_HandleTypeDef *huart)
{
  // Clear buffer when transmission is complete
  if (huart->Instance == USART1)
  {
    memset(TX_buff_1, 0, strlen(TX_buff_1));
  }
  else if (huart->Instance == USART6)
  {
    memset(TX_buff_2, 0, strlen(TX_buff_2));
  }
}
```

Concernant la **transmission**, celle-ci peut être effectuée de manière **bloquante**, car le programme ne sera pas occupé par une autre tâche critique pendant cette période.

Il suffit d’implémenter une **interruption à la fin de la transmission** pour **vider la mémoire tampon** contenant les données transmises. Cela garantit qu’**aucun artefact** ne persiste pour la prochaine transmission.

## Ressources et Références

- [STM32 Official Wiki](https://wiki.st.com/stm32mcu/)
    - [Getting started with I2C](https://wiki.st.com/stm32mcu/wiki/Getting_started_with_I2C)
    - [Getting started with UART](https://wiki.st.com/stm32mcu/wiki/Getting_started_with_UART)
    - [Getting started with SPI](https://wiki.st.com/stm32mcu/wiki/Getting_started_with_SPI)
- [STM32 UART (USART) Tutorial + Examples (DMA, Interrupt)](https://deepbluembedded.com/stm32-usart-uart-tutorial/)
- [STM32 I2C Tutorial](https://deepbluembedded.com/stm32-i2c-tutorial-hal-examples-slave-dma/)
