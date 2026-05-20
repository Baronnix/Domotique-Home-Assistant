# 🏠 Installation de Mosquitto MQTT + Zigbee2MQTT via l’interface Home Assistant

Ce guide explique comment installer Mosquitto MQTT et Zigbee2MQTT depuis l’interface Home Assistant, les configurer, puis appairer un bouton Zigbee.

# 📺 Vidéo

Lien Youtube: [https://www.youtube.com/@Baronnix/playlists](https://www.youtube.com/@Baronnix/playlists)

# 📦 Prérequis

* Home Assistant

* Accès à l’interface Home Assistant

* Des appareils Zigbee (boutons, ampoules, capteurs, ...)

* Un dongle Zigbee compatible (Sonoff ZBDongle‑P/E, ConBee II, CC2652…)

![zigbee_dongle](images/zigbee_dongle.png)

# Installer Mosquitto MQTT via Home Assistant

## 1. 📥 Installation

1. Aller dans Paramètres → Apps

2. Cliquer sur Installer l'appplication

3. Rechercher Mosquitto broker

4. Cliquer sur Installer

## 2. ⚙️ Configuration

1. Ouvrir le module Mosquitto

2. Aller dans Configuration

3. Laisser la configuration par défaut (Home Assistant gère automatiquement les utilisateurs)

4. Activer :

    * Démarrer au démarrage

    * Surveiller

5. Cliquer sur Démarrer

Normalement, Home Assistant détecte automatiquement le broker MQTT.

Si ce n’est pas le cas :

1. Aller dans Paramètres → Appareils & Services

2. Cliquer sur Ajouter une intégration

3. Rechercher MQTT

4. Ajouter l'intégration

Home Assistant détectera automatiquement Zigbee2MQTT.

## 3. 👤 Utilisateur MQTT

Home Assistant utilise ses propres utilisateurs.

Créer un utilisateur dédié :

1. Aller dans Paramètres → Personnes & Utilisateurs → Utilisateurs

2. Ajouter un utilisateur :

    * Nom : mqtt    

# Installer Zigbee2MQTT via Home Assistant

Avant toute chose, brancher le dongle Zigbee

## 1. 📥 Installation

1. Aller dans Paramètres → Apps

2. Cliquer sur Installer l'appplication

3. Cliquer sur les 3 petits points verticaux en haut à droite

4. Cliquer sur dépôt puis Ajouter

5. Renseigner le dépôt: https://github.com/zigbee2mqtt/hassio-zigbee2mqtt

6. Cliquer sur Ajouter

7. Retourner dans Paramètres → Apps

8. Cliquer sur Installer l'appplication

9. Rechercher Zigbee2MQTT (et non Zigbee2MQTT Proxy ni Zigbee2MQTT Edge)

10. Cliquer sur Installer

## 2.🔌 Sélection du dongle Zigbee

1. Aller dans Paramètres → Système →  Matériel

2. Dans la liste rechercher le dongle que vous venez de connecter

3. Copier le devlink reesemblant à l' example suivant: /dev/serial/by-id/usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_502c5c18c278f01195f9fbeba7772636-if00-port0

4. Aller dans Paramètres → Apps → Zigbee2MQTT

5. Aller dans Configuration

6. Renseigner base_topic dans la section mqtt: zigbee2mqtt

7. Renseigner port dans la section serial avec la valeur copiée précédement: (à adapter) /dev/serial/by-id/usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_502c5c18c278f01195f9fbeba7772636-if00-port0

8. Renseigner adapter dans la section serial avec la valeur copiée précédement: ezsp

9. Enregistrer

10. Vérifier la configuration en yaml:

```yaml
data_path: /config/zigbee2mqtt
socat:
  enabled: false
  master: pty,raw,echo=0,link=/tmp/ttyZ2M,mode=777
  slave: tcp-listen:8485,keepalive,nodelay,reuseaddr,keepidle=1,keepintvl=1,keepcnt=5
  options: "-d -d"
  log: false
mqtt:
  base_topic: zigbee2mqtt
serial:
  port: >-
    /dev/serial/by-id/usb-Itead_Sonoff_Zigbee_3.0_USB_Dongle_Plus_V2_502c5c18c278f01195f9fbeba7772636-if00-port0
  adapter: ezsp
```

## 3. ▶️ Lancer Zigbee2MQTT

1. Activer :

    * Démarrer au démarrage

    * Surveiller

2. Cliquer sur Démarrer

3. Ouvrir l’interface Zigbee2MQTT via Ouvrir l’interface Web

4. Sélectionner le dongle sur la page de lancement

5. Cliquer sur submit

# Appairage d’un bouton Zigbee

## 1. 🔄 Mettre Zigbee2MQTT en mode appairage

Dans l’interface Zigbee2MQTT :

1. Aller dans Zigbee2MQTT

2. Activer "Autoriser l'appairage"  

## 2. 🔘 Mettre le bouton en mode appairage

Selon le modèle, généralement :

* Appuyer longuement 5–10 secondes sur le bouton reset

* Une LED clignote → l’appareil cherche le réseau Zigbee

## 3. 🟢 Appareil détecté

Dans Zigbee2MQTT, un nouvel appareil apparaît :

* Nom : 0x00158d000xxxxx

* Type : Wireless switch, Button, etc.

Vous pouvez :

* Renommer l’appareil

* Vérifier les actions (single click, double click, long press…)

* Le retrouver automatiquement dans Home Assistant

# Utiliser le bouton dans Home Assistant

## 1. 🧩 Automatisations

1. Aller dans Paramètres → Automatisations & Scènes

2. Créer une nouvelle automatisation

3. Déclencheur :

    * Appareil

    * Choisir votre bouton Zigbee

    * Sélectionner l’action (ex : single click)

4. Ajouter une action (ex : allumer une lumière)