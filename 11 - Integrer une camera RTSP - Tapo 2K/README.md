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

## 2. Configuration dans l'app Tapo (RTSP / ONVIF)

### 2.1 Mise en service
1. Installer l'app **Tapo**, créer / se connecter au compte TP-Link.
2. Ajouter la caméra (`+` → Caméras → modèle) et la connecter au Wi-Fi **2,4 GHz**.
3. Mettre à jour le firmware : *Caméra → ⚙️ → Mise à jour du firmware*.

### 2.2 Créer le compte caméra (RTSP / ONVIF)
*Caméra → ⚙️ Paramètres → Paramètres avancés → **Compte de la caméra***

- Définir un **nom d'utilisateur** et un **mot de passe** dédiés (6–32 caractères).
- ⚠️ Ce compte est **différent** du compte TP-Link. C'est lui qu'utilisent RTSP, ONVIF et Frigate.
- Éviter les caractères spéciaux type `@ : / ?` (ils cassent les URL RTSP).

### 2.3 Compatibilité tierce (firmwares récents)
*Onglet **Moi** → **Tapo Lab** → **Compatibilité tierce** → Activer*

Sans cette option, certains firmwares refusent RTSP / ONVIF ou l'authentification locale de Home Assistant.

### 2.4 Détection
*Caméra → ⚙️ → Détection / Paramètres de détection*

- Activer la **détection de mouvement** et la **détection de personne** (IA).
- Régler la sensibilité et, si disponible, les **zones d'activité**.
- Facultatif : désactiver les notifications de l'app (Home Assistant les remplacera).

### 2.5 Informations utiles

| Paramètre | Valeur |
|---|---|
| Flux principal (2K) | `rtsp://USER:PASS@IP_CAMERA:554/stream1` |
| Flux secondaire (basse définition) | `rtsp://USER:PASS@IP_CAMERA:554/stream2` |
| Port ONVIF | `2020` |
| Codec | H.264 (en général) |

## 3. IP fixe (réservation DHCP)

Méthode recommandée : **réservation DHCP sur la box**, plutôt qu'une IP statique dans la caméra.

1. Récupérer l'adresse **MAC** de la caméra : *app Tapo → Caméra → ⚙️ → Informations sur l'appareil*.
2. Dans l'interface de la box / du routeur, menu *Réseau → DHCP → Baux statiques / Réservations*.
3. Associer la MAC à une IP, par ex. `192.168.1.50`.
4. Redémarrer la caméra et vérifier l'IP dans l'app.

| Box | Emplacement indicatif |
|---|---|
| Freebox | Paramètres de la Freebox → DHCP → Baux statiques |
| Livebox | Configuration avancée → Réseau → DHCP → Adresses IP statiques |
| SFR Box | Réseau → DHCP → Adresses IP fixes |
| Bbox | Services de la Box → DHCP → Adresses IP statiques |

## 4. Tester les flux RTSP

Avant de couper internet, valider le flux avec **VLC** : *Média → Ouvrir un flux réseau* :

```
rtsp://USER:PASS@192.168.1.50:554/stream1
```

Si l'image s'affiche, RTSP est opérationnel.

## 5. Bloquer l'accès internet de la caméra

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

## 6. Intégration native dans Home Assistant

### 6.1 Intégration TP-Link Smart Home (native)
*Paramètres → Appareils et services → Ajouter une intégration → **TP-Link Smart Home***

- La caméra est souvent **découverte automatiquement**.
- Identifiants : compte **TP-Link** (pour le contrôle local), puis **compte caméra** pour le flux vidéo.
- Entités typiques : caméra, mode privé, détection de mouvement / personne (interrupteurs), pan/tilt selon le modèle.

### 6.2 Flux RTSP générique (alternative simple)
*Ajouter une intégration → **Generic Camera***

- URL du flux : `rtsp://USER:PASS@192.168.1.50:554/stream1`
- Protocole de transport : TCP
- Vidéo uniquement (pas de capteurs).

### 6.3 ONVIF (événements de détection)
*Ajouter une intégration → **ONVIF***

- Hôte : `192.168.1.50`, port : `2020`, identifiants : **compte caméra**.
- Crée des `binary_sensor` de mouvement (et de personne selon firmware).

> On peut combiner plusieurs méthodes, mais la caméra limite le nombre de flux simultanés (environ 2–3). Utiliser **go2rtc** (inclus dans HA et Frigate) pour mutualiser.

## 7. Installation de HACS Tapo Control

Intégration **Tapo: Cameras Control** (dépôt `JurajNyiri/HomeAssistant-Tapo-Control`).

### 7.1 Installation
1. **HACS** → rechercher **Tapo: Cameras Control** → *Télécharger*.
2. **Redémarrer Home Assistant**.
3. *Paramètres → Appareils et services → Ajouter une intégration → **Tapo: Cameras Control***.

### 7.2 Configuration
| Champ | Valeur |
|---|---|
| IP | `192.168.1.50` |
| Utilisateur / mot de passe (caméra) | **Compte caméra** créé à l'étape 2.2 |
| Mot de passe cloud | Mot de passe du **compte TP-Link** (demandé pour certaines fonctions ; la communication reste locale) |
| Contrôle / flux | Activer le flux HD et/ou SD selon les besoins |

### 7.3 Fonctions disponibles (selon le modèle)
- **Pan / tilt** : boutons de déplacement (haut / bas / gauche / droite), positions prédéfinies, calibrage.
- **Détection** : `binary_sensor` mouvement et **personne** (via événements ONVIF).
- **Réglages** : mode privé, vision nocturne, LED, alarme, suivi automatique, sensibilité, retournement d'image…
- **Flux** HD / SD, mise à jour du firmware.

### 7.4 Exemple : carte Lovelace avec pan/tilt
```yaml
type: vertical-stack
cards:
  - type: picture-entity
    entity: camera.tapo_hd_stream
    camera_view: live
  - type: horizontal-stack
    cards:
      - type: button
        icon: mdi:arrow-left
        tap_action: { action: call-service, service: button.press, target: { entity_id: button.tapo_move_left } }
      - type: button
        icon: mdi:arrow-up
        tap_action: { action: call-service, service: button.press, target: { entity_id: button.tapo_move_up } }
      - type: button
        icon: mdi:arrow-down
        tap_action: { action: call-service, service: button.press, target: { entity_id: button.tapo_move_down } }
      - type: button
        icon: mdi:arrow-right
        tap_action: { action: call-service, service: button.press, target: { entity_id: button.tapo_move_right } }
```
> Adapter les `entity_id` à ceux créés par l'intégration (*Paramètres → Entités*).

## 8. Notifications « personne détectée »

Prérequis : app **Home Assistant Companion** installée sur le téléphone (crée `notify.mobile_app_<nom>`).

```yaml
alias: Caméra - personne détectée
mode: single
trigger:
  - platform: state
    entity_id: binary_sensor.tapo_person_detection
    to: "on"
action:
  - service: camera.snapshot
    target:
      entity_id: camera.tapo_hd_stream
    data:
      filename: /config/www/tapo_last.jpg
  - service: notify.mobile_app_mon_telephone
    data:
      title: "👤 Personne détectée"
      message: "Quelqu'un a été détecté par la caméra."
      data:
        image: /local/tapo_last.jpg
  - delay: "00:01:00"   # anti-spam : 1 notification max par minute
```

Variantes utiles : condition « personne absente de la maison », plage horaire, mode alarme activé.

## 9. Option Frigate : détection IA locale

### 9.1 Qu'est-ce que Frigate ?
**Frigate** est un **NVR open source** qui analyse les flux RTSP avec de l'**IA locale** (sans cloud, sans abonnement) pour détecter des objets : personne, voiture, animal…

### 9.2 Avantages
- ✅ Détection **bien plus fiable** que celle de la caméra (moins de fausses alertes : ombres, pluie, branches).
- ✅ **100 % local**, respect de la vie privée.
- ✅ **Enregistrement** (continu ou sur événement), clips et captures.
- ✅ **Zones et masques** (ex. alerter seulement dans l'allée, ignorer la rue).
- ✅ Intégration HA : `binary_sensor` par objet/zone, compteur de personnes, navigation dans les clips.
- ✅ Multi-marques (Tapo, Reolink, Hikvision, Dahua…).

### 9.3 Inconvénients
- ❌ **Gourmand en ressources** (décodage vidéo + inférence IA).
- ❌ Configuration en **YAML**, courbe d'apprentissage.
- ❌ Nécessite souvent du **matériel supplémentaire** (accélérateur, SSD).
- ❌ Stockage important si enregistrement.
- ❌ Ne gère pas le pan/tilt (garder Tapo Control pour cela).

### 9.4 Limites et règles avec un Raspberry Pi 4

| Point | Limite / règle |
|---|---|
| CPU seul | Inférence lente (~100 ms+/image), CPU saturé, HA ralenti → **déconseillé** |
| Accélérateur | **Google Coral USB quasi indispensable**, sur port **USB 3** (bleu) |
| Stockage | **Pas de carte SD** pour les enregistrements (usure, lenteur) → **SSD USB**, idéalement boot HA sur SSD |
| RAM | 4 Go minimum, **8 Go conseillé** |
| Nb de caméras | **1 à 2** caméras raisonnablement |
| Détection | Utiliser **`stream2`** (basse déf.) à **5 fps**, résolution 640×360 |
| Enregistrement | **Sur événement uniquement**, `stream1` réservé à l'enregistrement |
| Décodage | Activer `preset-rpi-64-h264` (HA OS 64 bits requis) |
| Alimentation | Alimentation officielle 3 A ; hub USB alimenté si Coral + SSD posent problème |
| Refroidissement | Boîtier ventilé / dissipateur recommandé |

### 9.5 Choix du matériel

| Solution | Coût indicatif | Performance détection | Caméras | Commentaire |
|---|---|---|---|---|
| **Raspberry Pi 4, CPU seul** | 0 € (existant) | ❌ Faible (~100 ms+) | 1 (limite) | Test uniquement |
| **Raspberry Pi 4 + Google Coral USB** | ~60–80 € | ✅ Très bonne (~10 ms) | 1–3 | Bon compromis si on garde le Pi ; SSD conseillé |
| **Raspberry Pi 5 + Coral / Hailo** | ~100–150 € | ✅ Très bonne | 2–4 | Plus de marge CPU ; vérifier le support Frigate de l'accélérateur |
| **Mini-PC Intel N100** (8–16 Go, SSD) | ~150–200 € | ✅ Bonne via **OpenVINO** (iGPU), sans Coral | 4–8+ | ⭐ **Recommandé** : HA + Frigate confortables, décodage matériel QuickSync, faible conso (~6–10 W) |
| **Mini-PC N100 + Coral** | ~220–280 € | ✅✅ Excellente | 8+ | Pour beaucoup de caméras |
| **PC avec GPU Nvidia** | 300 € + | ✅✅ Excellente (TensorRT) | 10+ | Surdimensionné pour un usage domestique, conso élevée |

> **Google Coral** : existe en **USB** (simple, universel) et en **M.2 / Mini PCIe** (plus stable, pour mini-PC compatible). La disponibilité varie : vérifier avant achat.

**Recommandation :**
- 1 caméra, pas d'enregistrement → **Tapo Control seul** (détection de la caméra), sans Frigate.
- Garder le Pi 4 → **Coral USB + SSD**, 1–2 caméras.
- Projet évolutif → **Mini-PC Intel N100** avec OpenVINO.

### 9.6 Installation (HA OS)
1. *Paramètres → Modules complémentaires → Boutique* → ⋮ → **Dépôts** → ajouter :
   ```
   https://github.com/blakeblackshear/frigate-hass-addons
   ```
2. Installer **Frigate** (version standard ; « Full Access » seulement si nécessaire pour certains accélérateurs).
3. Recommandé : installer l'add-on **Mosquitto broker** et l'intégration **MQTT**.
4. Créer la configuration (`/config/frigate.yml` ou selon la version de l'add-on), puis démarrer l'add-on.
5. **HACS** → installer l'intégration **Frigate** → redémarrer HA → *Ajouter une intégration → Frigate*.

### 9.7 Exemple de configuration (Pi 4 + Coral USB, 1 Tapo)
```yaml
mqtt:
  enabled: true
  host: core-mosquitto
  user: mqtt_user
  password: mqtt_password

detectors:
  coral:
    type: edgetpu
    device: usb

go2rtc:
  streams:
    tapo_hd: rtsp://USER:PASS@192.168.1.50:554/stream1
    tapo_sd: rtsp://USER:PASS@192.168.1.50:554/stream2

cameras:
  tapo:
    ffmpeg:
      hwaccel_args: preset-rpi-64-h264
      inputs:
        - path: rtsp://127.0.0.1:8554/tapo_sd
          input_args: preset-rtsp-restream
          roles: [detect]
        - path: rtsp://127.0.0.1:8554/tapo_hd
          input_args: preset-rtsp-restream
          roles: [record]
    detect:
      width: 640
      height: 360
      fps: 5
    objects:
      track: [person]
      filters:
        person:
          min_score: 0.5
          threshold: 0.7
    record:
      enabled: true
      retain:
        days: 0          # pas d'enregistrement continu
      alerts:
        retain:
          days: 7        # clips d'événements conservés 7 jours
    snapshots:
      enabled: true
      retain:
        default: 7
```

### 9.8 Variante Mini-PC Intel N100 (OpenVINO)
```yaml
detectors:
  ov:
    type: openvino
    device: GPU

model:
  width: 300
  height: 300
  input_tensor: nhwc
  input_pixel_format: bgr
  path: /openvino-model/ssdlite_mobilenet_v2.xml
  labelmap_path: /openvino-model/coco_91cl_bkgr.txt

cameras:
  tapo:
    ffmpeg:
      hwaccel_args: preset-vaapi
      # ... inputs identiques
```

> La syntaxe (`record`, `alerts`, modèles) dépend de la version de Frigate : se référer à https://docs.frigate.video

### 9.9 Notification avec Frigate
```yaml
alias: Frigate - personne détectée
trigger:
  - platform: state
    entity_id: binary_sensor.tapo_person_occupancy
    to: "on"
action:
  - service: notify.mobile_app_mon_telephone
    data:
      title: "👤 Personne détectée (Frigate)"
      message: "Caméra Tapo"
      data:
        image: /api/frigate/notifications/latest/tapo/person/snapshot.jpg
  - delay: "00:01:00"
```
> Des **blueprints** communautaires (« Frigate Notifications ») offrent des notifications riches (clip, zones, anti-spam).

## 10. Dépannage

| Symptôme | Piste |
|---|---|
| RTSP refusé / authentification échouée | Vérifier le **compte caméra** (pas le compte TP-Link), activer **Compatibilité tierce**, éviter les caractères spéciaux |
| Flux qui coupe | Utiliser **go2rtc** / restream, réduire le nombre de clients simultanés, vérifier le signal Wi-Fi |
| ONVIF introuvable | Port **2020**, IP fixe, firmware à jour |
| Pas de capteur « personne » | Détection de personne activée dans l'app ; dépend du modèle / firmware → sinon Frigate |
| Tapo Control : erreur d'authentification | Renseigner le mot de passe **cloud TP-Link** ; firmware récent → Compatibilité tierce |
| Heure incorrecte après blocage internet | Autoriser le NTP (UDP 123) |
| Frigate lent sur Pi | Vérifier le Coral (USB 3), `stream2` en détection, 5 fps, SSD |

*Document de référence, à adapter selon votre modèle de caméra, votre box et vos versions de logiciels.*
