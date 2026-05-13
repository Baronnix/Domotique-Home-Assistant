# 🏠 Installation de Home Assistant OS sur un Raspberry Pi 4

Ce document présente les étapes pour installer et configurer un serveur Home Assistant autonome sur un raspberry pi 4.
Ce guide explique comment installer Home Assistant OS sur un Raspberry Pi 4.

## 📺 Vidéo

Lien Youtube: [https://www.youtube.com/@Baronnix/playlists](https://www.youtube.com/@Baronnix/playlists)

## Comparatif des méthodes d’installation de Home Assistant

Home Assistant peut être installé de plusieurs façons, chacune ayant ses avantages et limites.
Voici un tableau comparatif des méthodes principales sur Raspberry Pi 4.

 | Méthode | Supervisor | Add-ons | Maintenance | Liberté système | Support officiel | Difficulté | Support |
 | --- | --- | --- | --- | --- | --- | --- | --- |
 | Home Assistant OS | ✔️ | ✔️ | Très faible | ❌ | ⭐⭐⭐⭐⭐ | ⭐ | Support Complet |
 | Home Assistant Supervised | ✔️ | ✔️ | Moyenne | ✔️ (Debian uniquement) | ⭐⭐⭐⭐ | ⭐⭐⭐ | Abandonné |
 | Home Assistant Container | ❌ | ❌ | Moyenne/Élevée | ✔️ | ⭐⭐⭐ | ⭐⭐⭐ | Supporté |

Nous allons installer "Home Assistant OS" en suivant [ce tutoriel](https://www.home-assistant.io/installation/raspberrypi/)

## 1. 📦 Prérequis

Matériel
 - Raspberry Pi 4 (4 Go ou 8 Go recommandé)
 - Carte micro‑SD (32 Go minimum, classe A2 recommandée)
 - Alimentation officielle
 - Connexion réseau stable (Ethernet recommandé)
 - Un PC Windows, macOS ou Linux 

Plus de details voir l' episode 01 - Materiel Home Assistant:
 - Lien Youtube: [https://www.youtube.com/watch?v=PQOpOTrsC44](https://www.youtube.com/watch?v=PQOpOTrsC44)
 - Lien GitHub: [https://github.com/Baronnix/Domotique-Home-Assistant/tree/main/01%20-%20Materiel%20Home%20Assistant](https://github.com/Baronnix/Domotique-Home-Assistant/tree/main/01%20-%20Materiel%20Home%20Assistant)

## 2. 🧰 Télécharger l’image Home Assistant OS

1. Aller sur : https://www.home-assistant.io/installation/raspberrypi

2. Choisir ton modèle de Raspberry Pi

3. Télécharger l’image .img.xz

## 3. ⚙️ Flasher la carte SD

1. Télécharger Raspberry PI Imager from [https://www.raspberrypi.com/software](https://www.raspberrypi.com/software)

2. Installer et ouvrir Raspberry Pi Imager

3. Cliquer sur Choose OS → Use custom

4. Sélectionner l’image Home Assistant téléchargée

5. Cliquer sur Choose Storage et sélectionner la carte SD

6. Cliquer sur Write

## 4. 🚀 Premier démarrage

1. Insérer la carte SD dans le Raspberry Pi

2. Brancher le câble réseau (recommandé pour la première installation)

3. Alimenter le Raspberry

4. Attendre 5 à 10 minutes pendant l’installation automatique

## 5. 🌐 Accéder à Home Assistant

Une fois le Raspberry démarré, ouvrir un navigateur et aller sur :
* http://homeassistant.local:8123
Si cela ne fonctionne pas, utiliser l’adresse IP du Raspberry :
* http://X.X.X.X:8123

## 6. 🛠️ Configuration initiale

1. Créer un compte administrateur

2. Définir la localisation, le fuseau horaire et les unités

3. Ajouter les premiers appareils (si détectés automatiquement)

## 7. 📡 Activer le Wi‑Fi (optionnel)

Si tu veux utiliser le Wi‑Fi au lieu du câble Ethernet, il te faut créer une clé USB CONFIG séparée.

1. Préparer une clé USB:

    1. Formate-la en FAT32

    2. Renomme-la exactement : CONFIG (en majuscules)

1. Insérer la clé USB dans ton ordinateur

2. Créer un dossier network à la racine :
    
3. Créer un fichier my-network dans le dossier network :
    ``` 
    network/my-network
    ```

4. Y mettre ce contenu (adapter SSID et mot de passe) :
    ``` 
    [connection]
    id=my-network
    uuid=12345678-1234-5678-1234-567812345678
    type=wifi

    [wifi]
    mode=infrastructure
    ssid="NomDuWiFi"

    [wifi-security]
    auth-alg=open
    key-mgmt=wpa-psk
    psk="MotDePasseWiFi"

    [ipv4]
    method=auto

    [ipv6]
    method=auto
    ```
5. Sauvegarder, éjecter la clé USB, la brancher au Raspberry et redémarrer le Raspberry.

## 8. 🔧 Mise à jour
Dans Home Assistant :
* Settings → System → Updates

## 9. 📦 Add-ons utiles

* File Editor : éditer les fichiers YAML
* Studio Code Server : éditer les fichiers YAML
* Samba Share : accéder aux fichiers depuis Windows
* Mosquitto MQTT Broker : pour les appareils MQTT


# 🏠 Installation d’un OS sur Raspberry Pi 4 + Home Assistant Supervised

Cette section présente les étapes pour installer et configurer un serveur Home Assistant autonome sur un raspberry pi 4. Ce guide explique comment :
* Installer un système d’exploitation compatible sur un Raspberry Pi 4
* Préparer l’environnement pour Home Assistant Supervised
* Installer Home Assistant Supervised proprement

## ⚠️ 1. Notes importantes

Home Assistant Supervised est sensible aux modifications système et n'est plus maintenu.

Ce type d'installation permet d'utiliser le Raspberry Pi pour d'autres projets tout en gardant home assistant intact, mais nous savons déjà que Debian 12 est en fin de vie, pour plus de détails cliquez [ici](https://www.debian.org/releases/bookworm/).

Éviter d’installer des services qui pourraient entrer en conflit (Docker Swarm, Kubernetes, etc.).

Debian 12 est la seule distribution officiellement supportée pour Supervised.

La procédure ci-dessous ne fera pas l'objet d'un tutoriel et peut comporter des erreurs.

## 🧰 2. Choisir et installer l’OS

Home Assistant Supervised n’est officiellement supporté que sur Debian 12 (Bookworm).

Étapes

1. Télécharger Raspberry PI Imager from [https://www.raspberrypi.com/software](https://www.raspberrypi.com/software)

2. Installer et lancer Raspberry PI Imager

3. Télécharger l’image Debian 12 Desktop pour Raspberry Pi depuis un des liens suivants
 - [https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit](https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-64-bit)
 - [https://raspi.debian.net/tested-images/](https://raspi.debian.net/tested-images/)

4. Flasher l’image sur le disque SSD USB (ou la carte SD ou la clé USB) avec Raspberry Pi Imager

    1. Selectionner le modele du raspberry PI

    2. Selectionner "custom image" et choisir celle téléchargée

5. Connecter le disque SSD USB (ou la clé USB) au raspberry pi ou mettre la carte SD dans le raspberry pi
    
6. Démarrer le raspberry pi

7. Finit la configuration via l'interface graphique

8. Lancer le Terminal

9. Mettre a jour
    ```
    > sudo apt update && sudo apt upgrade -y
    ```

## ⚙️ 3. Préparation du système pour Home Assistant Supervised

1. Installer les dépendances nécessaires :

    ```
    > sudo apt install apparmor apparmor-utils avahi-daemon ca-certificates cifs-utils curl dbus jq libglib2.0-bin lsb-release network-manager nfs-common systemd-journal-remote socat software-properties-common systemd-resolved udisks2 wget -y
    ```

2. Désactiver cloud-init si présent (souvent inutile sur Pi) :
    ```
    > sudo systemctl disable --now cloud-init
    ```

3. S’assurer que NetworkManager gère le réseau :

    1.  Désactiver dhcpcd
        ``` 
        > sudo systemctl disable --now dhcpcd 
        ```

    2. Activer NetworkManager
        ``` 
        > sudo systemctl enable --now NetworkManager 
        ```

## 🐳 4. Installer Docker

Home Assistant Supervised nécessite Docker.

1. Installer docker

    ``` 
    > curl -fsSL https://get.docker.com | sh
    ```

2. Donner les droits nécessaires
    ``` 
    > sudo usermod -aG docker $USER
    ```

3. Redémarrer le Raspberry Pi

## 🏠 5. Installer Home Assistant Supervised

1. Télécharger le script officiel :
    ``` 
    > wget https://github.com/home-assistant/supervised-installer/releases/latest/download/homeassistant-supervised.deb
    ```

2. Installer
    ``` 
    > wget sudo dpkg -i homeassistant-supervised.deb
    ```

3. Si des dépendances manquent :
    ``` 
    > sudo apt --fix-broken install -y
    ```

## 🚀 6. Accéder à Home Assistant

Après quelques minutes, Home Assistant sera disponible à l’adresse : http://ip_du_raspberry:8123

La première initialisation peut prendre 5 à 20 minutes.

## 🔧 7. Maintenance & mises à jour

 - Mettre à jour Debian :
    ``` 
    > sudo apt update && sudo apt upgrade -y
    ```

 - Mettre à jour Home Assistant depuis l’interface web.

# 📚 Ressources utiles

 - Documentation officielle Home Assistant : https://www.home-assistant.io

 - Dépôt Supervised Installer : https://github.com/home-assistant/supervised-installer

 - Images Debian pour Raspberry Pi : https://raspi.debian.net