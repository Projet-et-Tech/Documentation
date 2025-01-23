---
title: Raspberry Pi
---

Le [Raspberry Pi](https://www.raspberrypi.com/) est un nano-ordinateur à processeur ARM, et disposant notamment de ports d'entrée/sortie (GPIO).

## Intérêt

Nous l'utilisons pour du traitement d'image, pour pouvoir détecter l'environnement du robot et pallier aux écarts de routine.

## Système d'exploitation

On ne considère évidemment que des systèmes GNU/Linux (éventuellement \*BSD). Plusieurs aspects sont à prendre en compte lors du choix du système d'exploitation :

- **La simplicité d'installation**

    Assez parlant, on veut pas trop se prendre la tête à l'installation, et pas installer trop de paquets pour pouvoir utiliser les capacités de la carte.

- **Les performances**

    Même si le matériel joue beaucoup, le logiciel n'est pas moins essentiel du point de vue des performances. Évidemment, et peu importe le système, on ne s'encombre pas d'une quelconque interface graphique, mais on peut faire d'autre choses, notamment en évitant de passer par des environnements isolés.

- **La reproductibilité**

    Imaginons que l'association décide de changer de matériel dans les prochaines années, pour par exemple passer sur une [ArmSoM](https://www.armsom.org/) (plus puissante, presque overkill par rapport à nos besoins, mais sait-on jamais), il faut que le système construit précédemment soit facilement portable sur n'importe quel support.


| Système d'exploitation | Simple ? | Reproductible ? | Performant ? |
|:----------------------:|:--------:|:---------------:|:------------:|
| [Raspberry Pi OS](https://www.raspberrypi.com/software/) | Très simple d'installation et d'utilisation, fonctionne sans configuration particulière. | Pas vraiment. | Plutôt. |
| [Gentoo](https://www.gentoo.org/) | Simple d'utilisation, configuration et installation potentiellement fastidieuses. | Pas trop reproductible car « sur mesure » pour la machine, compile tout en prenant en compte les préférences et la machine. Néanmoins, le système peut se recréer simplement en sauvegardant quelques fichiers. | Très performant, comme tout est compilé *pour* le Raspberry Pi. Néanmoins, les temps de compilation peuvent sembler interminables (salut OpenCV). |
| [NixOS](https://nixos.org/) | Pas instinctif ni bien documenté. La configuration peut-être longue. | Conçu pour la reproductibilité (entre autre), idéal de ce point de vue. | À voir, probablement le pire des trois. |

### Installation de Raspberry Pi OS

#### En graphique

On utilise [Raspberry Pi Imager](https://www.raspberrypi.com/software/), on peut choisir l'OS et les différents paramètres qu'on y applique (activation du SSH et Wi-Fi notamment).

Une fois installé, on le démarre en insérant une carte SD dans l'ordinateur, on sélectionne Raspberry Pi OS Lite 64 bits, on modifie les paramètres puis on installe.

Plus qu'à insérer la carte SD dans le RPi et à trouver son adresse IP pour s'y connecter

#### En ligne de commande (systèmes GNU/Linux)

On télécharge le bon système d'exploitation depuis le site officiel ([ici](https://downloads.raspberrypi.com/raspios_lite_arm64/images/raspios_lite_arm64-2024-07-04/2024-07-04-raspios-bookworm-arm64-lite.img.xz) au moment de l'écriture de l'article). La décompresser s'il s'agit d'une archive.

Ensuite, en ayant inséré le carte SD dans l'ordinateur (s'assurer qu'elle est bien démontée), et en supposant qu'elle est détectée sur `/dev/sdb`, on fait avec les droits adéquats (i.e. en root) :

```bash
dd if=/path/to/*-raspios-*.img of=/dev/sdb bs=10M
```

## Utilisation

On se connecte en SSH à la carte, et on interagit avec comme on le ferait sur une distribution GNU/Linux classique. Raspberry Pi OS est basé sur Debian, et utilise le même gestionnaire de paquets, `apt`.
