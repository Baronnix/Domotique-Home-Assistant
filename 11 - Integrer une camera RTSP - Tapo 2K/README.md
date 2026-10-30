# Intégration d'une caméra Tapo 2K dans Home Assistant

Dans ce guide complet, nous allons configurer pas à pas une caméra Tapo pour obtenir une intégration locale, fiable et performante. De l'activation des flux RTSP et ONVIF à la mise en place d'une adresse IP fixe, en passant par l'isolation d'Internet, l'intégration dans Home Assistant avec HACS et Tapo Control, jusqu'à l'ajout de Frigate pour la détection intelligente par IA en local, vous découvrirez comment exploiter tout le potentiel de votre caméra tout en gardant le contrôle de vos données.

⚠️ Les menus, noms d'options et versions évoluent selon le modèle (C200/C210/C220/C225…), le firmware Tapo et la version de Home Assistant / Frigate. Vérifiez la documentation officielle en cas de doute.

# 📺 Vidéo

Lien Youtube: [https://www.youtube.com/@Baronnix/playlists](https://www.youtube.com/@Baronnix/playlists)

# 🎯 Objectif

* ✅ Configurer correctement une caméra Tapo dans l'application mobile
* ✅ Activer les protocoles RTSP et ONVIF pour une utilisation locale
* ✅ Attribuer une adresse IP fixe à la caméra pour garantir sa stabilité
* ✅ Isoler la caméra d'Internet tout en conservant son fonctionnement local
* ✅ Ajouter la caméra dans Home Assistant
* ✅ Installer l'intégration Tapo Control
* ✅ Utiliser les fonctions Pan/Tilt directement depuis Home Assistant
* ✅ Récupérer les événements et détections de mouvement de la caméra
* ✅ Comprendre les avantages et les limites de l'intégration native
* ✅ Découvrir l'installation de Frigate pour la détection IA locale
* ✅ Améliorer la reconnaissance des personnes, véhicules et animaux sans dépendre du cloud
* ✅ Construire une solution de vidéosurveillance 100 % locale, sécurisée et respectueuse de la vie privée

## 1. 📋 Prérequis

Vous devez disposer :
* D'une Caméra Tapo 2K (ex. C200 / C210 / C220 / C225) avec firmware à jour
* D'un Smartphone: obligatoire pour la mise en service avec l'application **Tapo** + compte **TP-Link**
* D'un Home Assistant fonctionnel.
* D'un réseau avec accès à l'interface de la box / du routeur (réservation DHCP, pare-feu)
* De HACS installé sur Home Assitant (Voir tutoriel [07 - HACS et button-card](https://www.youtube.com/watch?v=8RZtPTxC_Nk))


## 2. Mise en service de la Tapo 2K

### Étape 1 — Installer la caméra
1. Installer l’app **Tapo**, ajouter la caméra et la connecter au Wi-Fi (ou Ethernet).
2. Mettre à jour le **firmware**.

### Étape 2 — Créer le compte caméra (RTSP/ONVIF)
1. App Tapo → caméra → ⚙️ **Paramètres** → **Paramètres avancés** → **Compte de la caméra**.
2. Définir un **nom d’utilisateur** et un **mot de passe** dédiés (différents du compte TP-Link).

> Ce compte sert à la fois pour **RTSP** et **ONVIF**.

### Étape 3 — Fixer l’adresse IP
1. App Tapo → **Infos sur l’appareil** → noter l’**adresse IP**.
2. Dans le routeur, créer une **réservation DHCP** pour cette IP.
3. Associer la MAC à une IP, par ex. `192.168.1.50`.
4. Redémarrer la caméra et vérifier l'IP dans l'app.

| Box | Emplacement indicatif |
|---|---|
| Freebox | Paramètres de la Freebox → DHCP → Baux statiques |
| Livebox | Configuration avancée → Réseau → DHCP → Adresses IP statiques |
| SFR Box | Réseau → DHCP → Adresses IP fixes |
| Bbox | Services de la Box → DHCP → Adresses IP statiques |

### Étape 4 — Tester le flux dans VLC
**Média → Ouvrir un flux réseau** :

```text
rtsp://UTILISATEUR:MOTDEPASSE@192.168.1.50:554/stream1   # haute qualité (2K)
rtsp://UTILISATEUR:MOTDEPASSE@192.168.1.50:554/stream2   # basse qualité (360p/720p)
```

Si l’image s’affiche, le RTSP fonctionne.

## 3. Intégration dans Home Assistant

### 3.1 Informations utiles

Il existe 4 méthodes d'intégration:
* L'application Tapo de TP-Link (non testée car échec à la connection)
* ONVIF
* RTSP (Caméra Générique)
* Tapo Control (HACS)

Voici un résumé des capacités de chaque intégration après tests.

| Fonction | ONVIF | RTSP | Tapo Control (HACS) |
|-----------|-------|-------|---------------------|
| Vidéo + Audio  | ✅ | ✅ | ✅ |
| Contrôle PTZ | ❌ | ❌ | ✅ |
| Détection de mouvement | ✅ | ❌ | ✅ |
| Détection de personnes | Selon caméra | ❌ | ✅ |
| Détection d'animaux | Selon caméra (C520WS: ✅) | ❌ | Selon caméra (C520WS: ❌) |
| Détection de véhicules | Selon caméra (C520WS: ✅) | ❌ | Selon caméra (C520WS: ❌) |
| Contrôle des réglages | Limité | ❌ | ✅ |

Pour une intégration complète et suivant les besoins on choisira une ou plusieurs intégrations.

On peut combiner plusieurs méthodes, mais la caméra limite le nombre de flux simultanés (environ 2–3). 

| Paramètre | Valeur |
|---|---|
| Flux principal (2K) | `rtsp://USER:PASS@IP_CAMERA:554/stream1` |
| Flux secondaire (basse définition) | `rtsp://USER:PASS@IP_CAMERA:554/stream2` |
| Port ONVIF | `2020` |
| Codec | H.264 (en général) |

### 3.2 Intégration TP-Link Smart Home (native)

*Paramètres → Appareils et services → Ajouter une intégration → **TP-Link Smart Home***

- La caméra est souvent **découverte automatiquement**.
- Identifiants : compte **TP-Link** (pour le contrôle local), puis **compte caméra** pour le flux vidéo.
- Entités typiques : caméra, mode privé, détection de mouvement / personne (interrupteurs), pan/tilt selon le modèle.

### 3.3 Flux RTSP générique (alternative simple)

*Ajouter une intégration → **Generic Camera***

- URL du flux : `rtsp://USER:PASS@192.168.1.50:554/stream1`
- Protocole de transport : TCP
- Vidéo uniquement (pas de capteurs).

### 3.4 ONVIF (événements de détection)

*Ajouter une intégration → **ONVIF***

- Hôte : `192.168.1.50`, port : `2020`, identifiants : **compte caméra**.
- Crée des `binary_sensor` de mouvement (et de personne selon firmware).

### 3.5 Installation de HACS Tapo Control

Intégration **Tapo: Cameras Control** (dépôt `JurajNyiri/HomeAssistant-Tapo-Control`).

1. **HACS** → rechercher **Tapo: Cameras Control** → *Télécharger*.
2. **Redémarrer Home Assistant**.
3. *Paramètres → Appareils et services → Ajouter une intégration → **Tapo: Cameras Control***.
4. Configurer:
  * IP: `192.168.1.50` 
  * Utilisateur / mot de passe (caméra): **Compte caméra** créé à l'étape 2.2
  * Mot de passe cloud | Mot de passe du **compte TP-Link** (demandé pour certaines fonctions ; la communication reste locale)
5. Explorer les fonctions disponibles (selon le modèle)
- **Pan / tilt** : boutons de déplacement (haut / bas / gauche / droite), positions prédéfinies, calibrage.
- **Détection** : `binary_sensor` mouvement et **personne** (via événements ONVIF).
- **Réglages** : mode privé, vision nocturne, LED, alarme, suivi automatique, sensibilité, retournement d'image…
- **Flux** HD / SD, mise à jour du firmware.

Exemple de configuration pour une carte button-card (voir [07 - HACS et button-card](https://www.youtube.com/watch?v=8RZtPTxC_Nk))
```yaml
type: vertical-stack
cards:
  - type: grid
    columns: 3
    square: true
    cards:
      - type: button
        entity: button.tapo_2k_01_move_left
        name: ←
        tap_action:
          action: toggle
      - type: button
        entity: button.tapo_2k_01_move_up
        name: ↑
        tap_action:
          action: toggle
      - type: button
        entity: button.tapo_2k_01_move_right
        name: →
        tap_action:
          action: toggle
      - type: button
        entity: button.tapo_2k_01_move_left
        name: ←
        tap_action:
          action: toggle
      - type: button
        entity: button.tapo_2k_01_move_down
        name: ↓
        tap_action:
          action: toggle
      - type: button
        entity: button.tapo_2k_01_move_right
        name: →
        tap_action:
          action: toggle
  - type: entities
    title: Positions
    entities:
      - select.tapo_2k_01_move_to_preset
  - type: entities
    title: Détections
    entities:
      - select.tapo_2k_01_person_detection
      - select.tapo_2k_01_pet_detection
      - select.tapo_2k_01_vehicle_detection
      - select.tapo_2k_01_tamper_detection
      - binary_sensor.piece_a_vivre_tapo_2k_01_person_detection
      - select.tapo_2k_01_pet_detection
      - select.tapo_2k_01_vehicle_detection
      - select.tapo_2k_01_tamper_detection
  - type: entities
    title: Smart Track
    entities:
      - switch.tapo_2k_01_smart_track_people
      - switch.tapo_2k_01_smart_track_pet
      - switch.tapo_2k_01_smart_track_vehicle
      - switch.tapo_2k_01_smart_track_baby
  - type: entities
    title: Vision & Éclairage
    entities:
      - select.tapo_2k_01_night_vision
      - select.tapo_2k_01_night_vision_switching
      - select.tapo_2k_01_spotlight_intensity
      - select.tapo_2k_01_spotlight_on_off_for
  - type: entities
    title: Audio
    entities:
      - switch.tapo_2k_01_record_audio
      - number.tapo_2k_01_speaker_volume
  - type: entities
    title: Sirène
    entities:
      - siren.tapo_2k_01_siren
      - number.tapo_2k_01_siren_volume
      - number.tapo_2k_01_siren_duration
      - select.tapo_2k_01_siren_type
  - type: entities
    title: Enregistrement
    entities:
      - switch.tapo_2k_01_record_to_sd_card
      - camera.tapo_2k_01_sd_stream
      - sensor.tapo_2k_01_recordings_synchronization
  - type: entities
    title: Notifications
    entities:
      - switch.tapo_2k_01_notifications
      - switch.tapo_2k_01_rich_notifications
  - type: entities
    title: Patrouille
    entities:
      - select.tapo_2k_01_patrol_mode
      - switch.tapo_2k_01_preset_patrol_mode
  - type: entities
    title: Alarmes
    entities:
      - switch.tapo_2k_01_trigger_alarm_on_motion_detection
      - switch.tapo_2k_01_trigger_alarm_on_people_detection
      - switch.tapo_2k_01_trigger_alarm_on_pet_detection
      - switch.tapo_2k_01_trigger_alarm_on_vehicle_detection
      - switch.tapo_2k_01_trigger_alarm_on_tamper_detection
      - switch.tapo_2k_01_trigger_alarm_on_linecrossing_detection
  - type: entities
    title: Confidentialité
    entities:
      - switch.tapo_2k_01_privacy
      - switch.tapo_2k_01_privacy_zones
  - type: entities
    title: Système
    entities:
      - sensor.tapo_2k_01_rssi
      - sensor.tapo_2k_01_network_ssid
      - select.tapo_2k_01_timezone
      - number.tapo_2k_01_movement_angle
      - button.tapo_2k_01_sync_time
      - button.tapo_2k_01_reboot
```

## 4. Bloquer l'accès internet de la caméra (Optionnel)

Objectif : la caméra fonctionne **uniquement en local** (Home Assistant), sans envoyer de flux vers le cloud.

> Faites-le **après** avoir tout configuré dans l'app (compte caméra, détection, mises à jour).

### Options (de la plus simple à la plus robuste)
1. **Contrôle parental / blocage d'appareil** de la box : bloquer l'accès internet pour la MAC de la caméra en conservant le réseau local.
2. **Pare-feu du routeur** (OpenWrt, pfSense, OPNsense, UniFi…) : règle `DROP` du trafic sortant WAN pour l'IP `192.168.1.50`.
3. **VLAN IoT dédié** : caméras isolées, internet bloqué, seul Home Assistant peut joindre la caméra (ports 554 et 2020, plus 443 pour Tapo Control).

### Conseils
- **Autoriser le NTP** (UDP 123) si possible, sinon l'horloge de la caméra peut dériver (horodatage incorrect).
- Réautoriser ponctuellement internet pour les **mises à jour de firmware**.
- Tester plusieurs jours la stabilité hors ligne.

### Ce que l'on perd
- Vue à distance et notifications via l'app Tapo (remplacées par Home Assistant).
- Mises à jour automatiques.

