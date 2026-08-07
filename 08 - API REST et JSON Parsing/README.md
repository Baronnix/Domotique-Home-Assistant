# 🏠 Tutoriel pour utiliser des API REST dans Home Assistant

Ce guide explique comment utiliser des API REST de façon dynamique permettant par exemple de créer des capteurs (sensor) ou de d’exécuter des actions sur des composants connectes au Wifi local.

# 🎯 Objectif du tutoriel

Ce guide explique comment :
* Créer un sensor Home Assistant qui appelle une API Internet gratuite (TimeAPI.io), puis parser le JSON pour extraire l’heure.
* Appeler une API REST locale pour envoyer des notes MIDI avec des paramètres dynamiques (note + channel), et créer un dashboard avec un sélecteur de channel et deux boutons (Scene A & Scene B).

Tu obtiendras un dashboard fonctionnel permettant de piloter ton serveur MIDI et d’afficher l’heure Internet.

# 📺 Vidéo

Lien Youtube: [https://www.youtube.com/@Baronnix/playlists](https://www.youtube.com/@Baronnix/playlists)

# 📦 Prérequis

* Une installation fonctionnelle de Home Assistant
* Carte ESP32‑S3 programmée en tant que MIDI contrôleur

Ce tutoriel repart du code final des tutoriels:
* [02 - ESP32 Arduino - USB‑C MIDI Controleur - Ajout Server Web local](https://github.com/Baronnix/ESP32/tree/main/Controleur%20Midi/02%20-%20ESP32%20Arduino%20-%20USB%E2%80%91C%20MIDI%20Controleur%20-%20Ajout%20Server%20Web%20local)
* [07 - HACS et button-card](https://github.com/Baronnix/Domotique-Home-Assistant/tree/main/07%20-%20HACS%20et%20button-card)

Lien Youtube des tutoriels précédent: 
* [02 - ESP32 Arduino - USB‑C MIDI Controleur - Ajout Server Web local](https://www.youtube.com/watch?v=u2ZS3cqmaGg)
* [07 - HACS et button-card](https://www.youtube.com/watch?v=8RZtPTxC_Nk)

# 🏛️ API Internet : Créer un sensor et parser le JSON

On va créer un sensor (capteur) remontant l'heure locale de Paris en utilisant une API disponible sur internet et retournant les informations au format JSON.

Ce sera générique et On pourra l'adapter à de nombreux cas comme par exemple pour récupérer des informations météorologiques. 

## 🕒 1. Desciption de l'API TimeAPI.io

On va utiliser l'API: https://timeapi.io/api/Time/current/zone?timeZone=Europe/Paris

TimeAPI renvoie un JSON contenant :

```json
{
  "hour": 10,
  "minute": 32,
  "seconds": 10,
  "dateTime": "2026-07-29T10:32:10.0000000"
}
```

## 🧩 2. Création d’un sensor REST appelant l’API TimeAPI.io et parsant le JSON pour extraire l’heure

1. Ouvrir Studio Code Server
2. Ajoute dans configuration.yaml :
```yaml
rest:
  - resource: https://timeapi.io/api/Time/current/zone?timeZone=Europe/Paris
    scan_interval: 15
    sensor:
      - name: "heure_paris"
        value_template: "{{ value_json['hour'] }}"
      - name: "minute_paris"
        value_template: "{{ value_json['minute'] }}"
      - name: "hhmm_paris"
        value_template: >
          {{ "%02d:%02d" | format(
              value_json['hour'],
              value_json['minute']
          ) }}
```
3. Vérifie la configuration: Paramètres -> Outils de développement -> Vérifier la configuration
4. Corrige si nécessaire. Ne redémarre pas Home Assistant tant que le message suivant n'apparait pas lors de la vérification:
  * La configuration n'empêchera pas Home Assistant de démarrer !
5. Redémarre Home Assistant: Paramètres -> Outils de développement -> Redémarrer -> Redémarre Home Assistant
6. Vérifie que les entités existent:
  * Va dans  Paramètres -> Appareils et services -> Entités
  * Cherche "_paris"
  * Les entités suivantes doivent apparaitre:
    * heure_paris
    * minute_paris
    * hhmm_paris

## 📟 3. Affichage de l’heure dans le dashboard

1. Ouvrir un dashboard existant ou créer un nouveau Dashboard
2. Passer en mode "Modifier le tableau de bord"
3. Cliquer sur "Ajouter une carte"
4. Choisir la carte et l'entité à afficher, par exemple hhmm_paris

* Exemple carte simple
```yaml
type: entity
entity: sensor.hhmm_paris
name: Heure Internet
icon: mdi:clock
```
* Exemple Version Custom Button Card
```yaml
type: custom:button-card
entity: sensor.hhmm_paris
name: Heure Internet
icon: mdi:clock-outline
show_state: true
style:
  - font-size: 22px
  - padding: 14px
```

# 🎹 API MIDI locale : Appels REST avec paramètres dynamiques

## 🎛️ 1. Desciption de l'API du contrôleur MIDI

Dans le tutoriel suivant nous avons crée un contrôleur MIDI: [ESP32/Controleur Midi/02 - ESP32 Arduino - USB‑C MIDI Controleur - Ajout Server Web local](https://github.com/Baronnix/ESP32/tree/main/Controleur%20Midi/02%20-%20ESP32%20Arduino%20-%20USB%E2%80%91C%20MIDI%20Controleur%20-%20Ajout%20Server%20Web%20local)

Ce contrôleur MIDI expose l'API suivante: http://midiserver.local/playNote?note={note_number}&channel={channel_number}
L’API accepte deux paramètres :
* note → numéro de note MIDI (0–127)
* channel → canal MIDI (1–16)

## 🎛️ 1. Déclarer l’appel API REST paramétrable

1. Ouvrir Studio Code Server
2. Ajoute dans configuration.yaml :
```yaml
rest_command:
  play_note:
    url: "http://midiserver.local/playNote?note={{ note }}&channel={{ channel }}"
    method: get
```
3. Vérifie la configuration: Paramètres -> Outils de développement -> Vérifier la configuration
4. Corrige si nécessaire. Ne redémarre pas Home Assistant tant que le message suivant n'apparait pas lors de la vérification:
  * La configuration n'empêchera pas Home Assistant de démarrer !
5. Redémarre Home Assistant: Paramètres -> Outils de développement -> Redémarrer -> Redémarre Home Assistant

## 🎚️ 2. Créer un sélecteur de channel (1–16)

Dans Home Assistant :
1. Aller dans Paramètres → Appareils et services  → Entrées
2. Cliquer sur Créer une entrée
3. Choisir Liste déroulante
4. Entrer les information suivantes:
  * Nom : MIDI Channel
  * Icone: mdi:midi
  * Ajouter les options: 
    * 1
    * 2
    * 3 
    ...
    * 16

Autre solution:
1. Ouvre l'éditeur de configuration de ton choix (ex: Studio Code Server)
2. Ajoute l' input_select suivant:
```yaml
input_select:
  midi_channel:
    name: MIDI Channel
    options:
      - "1"
      - "2"
      - "3"
      - "4"
      - "5"
      - "6"
      - "7"
      - "8"
      - "9"
      - "10"
      - "11"
      - "12"
      - "13"
      - "14"
      - "15"
      - "16"
    initial: "1"
    icon: mdi:midi
```
3. Ouvre Paramètres → Outils de développement dans Home Assistant.
4. Vérifier la configuration
5. Si la configuration est valide, Redémarrer avec un rechargement rapide

Remarque: si des input_select existent déjà dans configuration.yaml, il ne faut pas répliquer "input_select:" mais juste ajouter midi_channel à la suite des input_select existants. YAML et donc Home assistant n'acceptent pas de clés dupliquées.

## 🎼 3. Créer un script à appeler

On va créer un script à appeler par les boutons du dashboard. On pourrait appeler le service directement mais cela permettra de
* centraliser les appels
* réduire la duplication de code pour chaque bouton pour récupérer la valeur du channel
* permettre de faciliter les futures évolutions

1. Aller dans Paramètres → Automatisations et scènes → Scripts
2. Créer un script
3. Modifier en YAML
4. Coller la configuratin suivantes
```yaml
play_note_script:
  sequence:
  - data:
      channel: "{{ states('input_select.midi_channel') | string }}"
      note: "{{ note }}"
    action: rest_command.play_note
  alias: play_note_script
  description: ''
```
5. Enregistrer
6. Donner le nom "play_note_script"
7. Enregistrer

## 🖥️ 4. Créer le dashboard MIDI

1. Ouvrir un dashboard existant ou créer un nouveau Dashboard
2. Passer en mode "Modifier le tableau de bord"
3. Cliquer sur "Ajouter une carte"
4. Ajouter une carte Button-Card
  * Pour plus de détailss voir le tutoriel [Domotique-Home-Assistant/07 - HACS et button-card](https://github.com/Baronnix/Domotique-Home-Assistant/tree/main/07%20-%20HACS%20et%20button-card)
5. Modifier en YAML
6. Coller le code suivant
```yaml
type: vertical-stack
title: Dashboard MIDI
cards:
  - type: horizontal-stack
    cards:
      - type: entities
        entities:
          - input_select.midi_channel
  - type: horizontal-stack
    cards:
      - show_name: true
        show_icon: true
        type: custom:button-card
        show_state: false
        icon: mdi:dice-1
        name: Scene A
        color: blue
        tap_action:
          action: call-service
          service: script.play_note_script
          service_data:
            note: "60"
      - show_name: true
        show_icon: true
        type: custom:button-card
        show_state: false
        icon: mdi:dice-2
        name: Scene B
        color: purple
        tap_action:
          action: call-service
          service: script.play_note_script
          service_data:
            note: "62"
```
7. Enregistrer

## 🔢 5. Créer une automatisation appelant le script quand on appui sur un bouton Zigbee

Vous pouvez utiliser les boutons physique Zigbee du tutoriel [05 - Buzzer de Salon en Zigbee](https://github.com/Baronnix/Domotique-Home-Assistant/tree/main/05%20-%20Buzzer%20de%20Salon%20en%20Zigbee) pour appeler le script play_note_script. La [vidéo est disponible sur youtube](https://www.youtube.com/watch?v=9MrslnI1DvQ).

Cela permet de coupler les buzzers à des scènes audio ou vidéo.

1. Aller dans Paramètres → Automatisations et scènes → Automatisations
2. Créer une automatisaton qui se lance sur l'appui du bouton et qui appelle le script play_note_script en passant la valeur de la note en paramètre

🔵 Automatisation pour le Bouton 1
```yaml
alias: bouton 1 - Note 60
description: ""
triggers:
  - domain: mqtt
    device_id: 8cdfe9e99a8e42c1b8e69414a201cf66
    type: action
    subtype: single
    metadata: {}
    trigger: device
conditions: []
actions:
  - action: script.play_note_script
    metadata: {}
    data:
      note: "60"
mode: single
```

🔴 Automatisation pour le Bouton 2
```yaml
alias: bouton 2 - note 62
description: ""
triggers:
  - domain: mqtt
    device_id: c889dbd51df213df3aae9f45a9130197
    type: action
    subtype: single
    trigger: device
conditions: []
actions:
  - action: script.play_note_script
    metadata: {}
    data:
      note: "62"
mode: single
```

## 🧪 6. Tester

1. Branche l'ESP32S3 devkit sur le port USB-OTG
2. Ouvre Midi View
3. Joue avec le channel et appui sur les boutons virtuels et réels
4. Tu devrais voir les notes s'afficher dans Midi View
