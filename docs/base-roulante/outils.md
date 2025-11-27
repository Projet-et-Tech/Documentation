# Logiciels et Materiels Recommandés

---

Bienvenus dans ce chapitre qui à pour objectif de présenter une liste des outils et logiciels **indispensables** ou utiles pour travailler efficacement avec la carte moteur et, plus généralement, avec les cartes électroniques telles que les **STM32** ou **Arduino**.

Il détaille également **le matériel indispensable** pour utiliser ces outils ou pour travailler efficacement avec la base roulante.

## Les outils

> [!IMPORTANT]
> Bien qu'il soit **possible** d'utiliser tous ces **outils** sous **Linux**, je recommande principalement **Windows** pour une expérience optimale.
>
> En effet, la **compatibilité avec les pilotes** et les outils d'interaction avec les cartes (comme les programmateurs ST-Link) est généralement **meilleure et plus stable** sous cet OS.

### [STM32CubeIDE](https://www.st.com/en/development-tools/stm32cubeide.html)

Il s'agit de **l'environnement de développement intégré (IDE) officiel** de STM32, conçu pour programmer leurs microcontrôleurs.
Son utilisation est **fortement recommandée** pour :

- La **simplicité** de configuration d'un projet pour une cible spécifique, grâce à son éditeur graphique.
- Ses **fonctionnalités avancées**, telles que :
    - Le débogage en temps réel.
    - La lecture de la mémoire.
    - Le monitoring des variables.

### [STM32 ST-LINK Utility](https://www.st.com/en/development-tools/stsw-link004.html)

Un **outil pratique** pour dépanner une carte qui, soudainement, cesse de répondre ou n'est plus détectée, que ce soit par l'ordinateur ou par STM32CubeIDE.

### [Arduino IDE](https://docs.arduino.cc/software/ide/)

Il s'agit de **l'environnement de développement intégré (IDE) officiel** d'**Arduino**, spécialement conçu pour programmer les **cartes électroniques** Arduino et compatibles.

### [VSCode](https://code.visualstudio.com/) + [PlatformIO](https://platformio.org/platformio-ide)

Il est également possible de programmer les cartes électroniques **STM32** ou **Arduino** avec **Visual Studio Code** et l'extension **PlatformIO**.

Cependant, cette solution n'est **pas particulièrement recommandée**, car elle ne propose pas tous les outils spécialisés disponibles dans les IDE dédiés à ces cartes.

D'autres extensions existent également pour **VSCode**, comme :

- L'extension officielle **Arduino**.
- Une extension dédiée à la **Raspberry Pi Pico** et à **MicroPython**

### [PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html)

**PuTTY** est un outil open source largement utilisé pour établir des connexions **SSH**, **Telnet**, **Serial** et **Rlogin** vers des appareils distants ou des microcontrôleurs. Il est particulièrement utile pour :

- **Déboguer** des cartes électroniques via une liaison série.
- **Configurer** des appareils réseau ou des systèmes embarqués.
- **Accéder** à des terminaux distants de manière sécurisée.

Idéal pour les développeurs et les ingénieurs travaillant sur des projets embarqués ou des systèmes sans interface graphique.

Dans notre cas, **PuTTY** est principalement utilisé pour :

- **Déboguer** nos cartes électroniques via des **lectures et écritures** en communication **UART**.
- **Envoyer des commandes** directement à la carte pour effectuer des **tests de communication**.
- **Lire les logs** en temps réel, ce qui permet de **contrôler le fonctionnement** et les différentes actions exécutées par la carte.

### [PlotJuggler](https://plotjuggler.io/) / [Serial Port Plotter](https://github.com/CieNTi/serial_port_plotter)

Ces outils sont conçus pour **visualiser et analyser des données en temps réel**, notamment celles provenant de systèmes embarqués ou de capteurs.

Ces logiciels sont souvent utilisés et particulièrement utiles pour **valider le comportement des algorithmes** ou **surveiller les performances** des systèmes embarqués.

Dans notre cas, ces outils se sont révélés **particulièrement utiles** pour :

- **Visualiser les réponses indicielle et temporelle** de nos moteurs.
- Analyser le **comportement global de notre asservissement**.
- **Afficher sous forme de graphiques** diverses données collectées **en temps réel** pendant le fonctionnement de la carte.

> [!EXAMPLE] Exemple d'utilisation en temps réel
> Voici comment j'ai procédé pour visualiser les logs en temps réel :
>
> 1. J'ai connecté un **adaptateur "USB to UART"** à l'une des interfaces UART actives de notre base roulante.
> 2. J'ai envoyé des **trames de logs** formatées selon un protocole spécifique, compatible avec l'outil de visualisation utilisé.
> 3. J'ai configuré l'outil pour qu'il **écoute** sur cette interface UART (accessible via le port USB).
> 4. Grâce au **format standardisé** des données, l'outil a pu **détecter automatiquement** les trames, les **décoder**, et les afficher sous forme de **graphes évoluant en temps réel** pendant le fonctionnement de la base roulante.

### [Teleplot](https://marketplace.visualstudio.com/items?itemName=alexnesnes.teleplot)

**Teleplot** est une extension pour **Visual Studio Code** qui permet de **visualiser en temps réel** des données envoyées depuis un microcontrôleur ou un programme. Elle est particulièrement utile pour :

- **Déboguer** des algorithmes de contrôle ou de traitement de données.
- **Surveiller** des variables ou des signaux (capteurs, PID, etc.).
- **Tracer des graphiques** dynamiques directement depuis votre code embarqué.

## Le matériel

### [USB to UART - Adaptateur USB to TTL Serial (3.3V)](https://www.amazon.ca/CP2102-Download-Serial-Module-Upgrade/dp/B0DWKRLQ8Q?s=electronics)

Les PCs modernes n'ont plus de port **RS-232** (DB9). Cet adaptateur convertit le signal **USB** (standard sur les PCs) en signal **TTL** (3.3V ou 5V), compatible avec la plupart des cartes électroniques.

Un adaptateur **USB to TTL Serial** est un outil essentiel pour communiquer avec des microcontrôleurs. Il permet de connecter un **PC** à une carte électronique (comme une STM32, Arduino, ESP32, etc.) via une liaison **série (UART)**. Cela est indispensable pour :

- **Envoyer des commandes** (ex : configuration, tests).
- **Recevoir des logs** (ex : débogage, suivi en temps réel).
- **Visualiser les données** envoyées par le microcontrôleur (ex : valeurs de capteurs, états des moteurs).

> [!WARNING] Pilotes requis
> Pour que votre PC puisse **détecter et utiliser** cet adaptateur, vous devrez peut-être installer les **pilotes VCP (Virtual COM Port)** correspondants.
>
> Téléchargez-les ici : [Pilotes USB-to-UART (Silicon Labs)](https://www.silabs.com/software-and-tools/usb-to-uart-bridge-vcp-drivers?tab=downloads).

Une fois les pilotes installés, l'adaptateur apparaît comme un **port COM virtuel** sur votre PC. Vous pouvez alors utiliser des logiciels comme :

- **PuTTY** / **Tera Term** (pour une communication série basique).
- **Arduino IDE** / **STM32CubeIDE** (pour le téléversement de code).
- **Logiciels de visualisation** (ex : PlotJuggler pour tracer des graphes en temps réel).

> [!TIP] Conseils d'utilisation
>
> - **Branchez** l'adaptateur à un port USB de votre PC.
> - **Connectez** les broches **TX**, **RX**, et **GND** de l'adaptateur aux broches UART correspondantes de votre carte (en respectant les niveaux de tension, ici **3.3V**).
> - Vérifiez que le port COM attribué apparaît dans le **Gestionnaire de périphériques** (Windows) ou via la commande `ls /dev/tty*` (Linux/Mac).
> - Utilisez un terminal série (comme **PuTTY**, **Tera Term**, ou **screen**) pour communiquer avec votre carte, en configurant le bon **baudrate** (ex: 115200 bauds).

---

> [!CAUTION] Attention aux niveaux de tension
> Assurez-vous que votre adaptateur est bien en **3.3V** pour éviter d'endommager votre carte STM32. Certains adaptateurs USB-to-TTL fonctionnent en **5V** par défaut.

### [Carte STM32F411 BlackPill](https://www.adafruit.com/product/4877)

La **STM32F411 BlackPill** est la **carte électronique principale** utilisée pour exécuter le code embarqué de notre base roulante. Elle est basée sur un **microcontrôleur STM32F411CEU6**, une puce **32 bits** de la famille **ARM Cortex-M4**, offrant un bon équilibre entre **performance**, **consommation énergétique** et **coût**.

#### Caractéristiques clés

- **Microcontrôleur** : STM32F411CEU6 (ARM Cortex-M4 à 100 MHz).
- **Mémoire** :
    - 512 Ko de **Flash** pour stocker le programme.
    - 128 Ko de **RAM** pour les données dynamiques.
- **Connectivité** :
    - Plusieurs interfaces **UART**, **SPI**, **I2C** et **USB**.
    - **Pins GPIO** pour contrôler des périphériques externes (moteurs, capteurs, etc.).
- **Alimentation** : 3.3V (compatible avec la plupart des capteurs et modules).
- **Format compact** : Idéal pour les projets embarqués comme les robots.

#### Pourquoi ce choix ?

- **Puissance suffisante** pour gérer les calculs d'asservissement, l'odométrie et la communication en temps réel.
- **Compatibilité** avec les outils de développement comme **STM32CubeIDE** et les bibliothèques HAL/LL.
- **Coût abordable** tout en offrant des performances adaptées aux besoins d'une base roulante.

---
> [!TIP]
> Pour démarrer avec cette carte, vous aurez besoin :
>
> - D'un **câble USB-to-TTL** (pour la communication série).
> - D'un **programmateur** (comme un ST-Link) pour flasher le code.
> - D'une **alimentation stable** (3.3V) pour éviter les instabilités.

### [Moteurs 37DX68L](https://www.pololu.com/file/0J1706/pololu-37d-metal-gearmotors.pdf)

Les **moteurs 37DX68L** de Pololu sont des **moteurs à engrenages métalliques** utilisés pour entraîner les roues de notre base roulante. Ils sont conçus pour offrir un **bon compromis** entre **puissance**, **précision** et **durabilité**, ce qui les rend adaptés aux applications robotiques exigeantes.

#### Caractéristiques principales

- **Type** : Moteur CC à engrenages métalliques.
- **Tension nominale** : **12V** (compatible avec une alimentation standard pour robots).
- **Rapport de réduction** : **68:1**, offrant un **couple élevé** tout en réduisant la vitesse de rotation.
- **Vitesse à vide** : Environ **200 RPM** (tours par minute) à 12V.
- **Couple de sortie** : Jusqu'à **1.5 kgf·cm** (suffisant pour déplacer une base roulante légère à moyenne).
- **Encodeur intégré** : Permet une **mesure précise de la vitesse et de la position** des roues, essentielle pour l'asservissement.
- **Durabilité** : Boîtier métallique et engrenages robustes pour une **longue durée de vie**.

#### Raisons de notre choix ?

- **Couple adapté** : Le rapport de réduction de **50:1** permet de fournir suffisamment de force pour déplacer le robot, même sur des surfaces légèrement irrégulières.
- **Précision** : L'encodeur intégré facilite la mise en place d'un asservissement précis en vitesse et en position.
- **Compatibilité** : Ces moteurs sont faciles à interfacer avec des contrôleurs de moteurs standards (comme les drivers L298N ou DRV8833).

### La carte électronique (PCB)

La **PCB (Printed Circuit Board)** est un élément essentiel de notre base roulante. Elle sert de **plateforme centrale** pour connecter, alimenter et organiser tous les composants électroniques du système.

> [!NOTE]
> La PCB a été **conçue sur mesure** pour répondre aux besoins spécifiques du projet.

## Ressources et Références

- Outils
    - [Outils Développement](https://github.com/VRAC-team/la-maxi-liste-ressources-eurobot?tab=readme-ov-file#34-outils-pour-le-d%C3%A9veloppement)
    - [IDE pour STM32](https://github.com/VRAC-team/la-maxi-liste-ressources-eurobot?tab=readme-ov-file#312-stm32)
- Matériels
    - [What is USB to UART ?](https://trickycircuit.com/components/integrated-circuits/usb-to-uart/usb-to-uart-what-it-is-and-how-it-works/)
    - [STM32F411 Pinout](https://deepbluembedded.com/stm32f411-black-pill-board-pinout-schematic-datasheet/)
