# 🧩 Buzzer de salon en Zigbee avec Home Assistant

Ce guide explique comment créer un système de buzzers Zigbee dans Home Assistant, utilisant 2 boutons Zigbee et 1 ampoule Zigbee comme indicateur lumineux.

# 📺 Vidéo

Lien Youtube: [https://www.youtube.com/watch?v=PQOpOTrsC44](https://www.youtube.com/watch?v=PQOpOTrsC44)

# 🎯 Objectif

Créer un système où :
* Chaque bouton Zigbee agit comme un buzzer.
* L’ampoule Zigbee sert de signal visuel (ex : rouge pour le joueur 1, bleu pour le joueur 2).
* Home Assistant gère la logique (premier à appuyer, reset, etc.).

# 🧩 Matériel nécessaire

* Home Assistant avec une passerelle Zigbee
* Accès à l’interface Home Assistant
* Une passerelle Zigbee configurée avec Home Assistant (Zigbee2MQTT) (Voir tutoriel 04 - Zigbee Installation et Utilisation)
* 2 boutons Zigbee identiques (éviter 2 boutons de types différents car les temps de réponses peuvent grandement varier et fausser le jeu)

  ![bouton_zigbee](images/bouton_zigbee.jpg)

* 1 ampoule Zigbee RGB

  ![ampoule_zigbee_rgb](images/ampoule_zigbee_rgb.jpg)

# 🛠️ 1. Appairer les appareils Zigbee

1. Ouvre Zigbee2MQTT dans Home Assistant.
2. Autoriser l'appairage
3. Mets ton bouton Zigbee en mode appairage.

Répète pour le second bouton et l’ampoule.

Tu devrais voir apparaître trois nouveaux appareils.

# 🎛️ 2. Nommer clairement les appareils

Dans Zigbee2MQTT renomme les appareils :
* Bouton 1 → bouton_joueur_1
* Bouton 2 → bouton_joueur_2
* Ampoule → lampe_buzzer

Cela simplifie les automatisations.

# 🎨 3. Création des input_select pour les couleurs

On crée deux sélecteurs : un pour le joueur 1, un pour le joueur 2.
1. Ouvre l'éditeur de configuration de ton choix (ex: Studio Code Server)
2. Ajoute les 2 input_select suivants:
```yaml
input_select:
  couleur_joueur_1:
    name: Couleur Joueur 1
    options:
      - dodgerblue
      - fuchsia
      - turquoise
      - red
      - salmon
      - gold
      - yellow
      - springgreen
      - seagreen
      - pink
      - plum
      - sandybrown
      - purple
      - orchid
      - orange
      - magenta
      - lime
      - hotpink
    initial: dodgerblue
    icon: mdi:palette
  couleur_joueur_2:
    name: Couleur Joueur 2
    options:
      - dodgerblue
      - fuchsia
      - turquoise
      - red
      - salmon
      - gold
      - yellow
      - springgreen
      - seagreen
      - pink
      - plum
      - sandybrown
      - purple
      - orchid
      - orange
      - magenta
      - lime
      - hotpink
    initial: hotpink
    icon: mdi:palette
```
3. Ouvre Paramètres → Outils de développement dans Home Assistant.
4. Vérifier la configuration
5. Si la configuration est valide, Redémarrer avec un rechargement rapide

# 🔢 4. Création de 2 input_number pour mémoriser la victoire et un compteur de réinitialisation

1. Ouvre l'éditeur de configuration de ton choix (ex: Studio Code Server)
2. Ajoute l'input_number suivant:
```yaml
input_number:
  joueur_gagnant:
    name: joueur gagnant
    initial: 0
    min: 0
    max: 2
    step: 1
    icon: mdi:trophy
  tempo_joueur_gagnant:
    name: Tempo joueur gagnant
    initial: 5
    min: 1
    max: 10
    step: 1
    icon: mdi:timer-sand
```
3. Ouvre Paramètres → Outils de développement dans Home Assistant.
4. Vérifier la configuration
5. Si la configuration est valide, Redémarrer avec un rechargement rapide

# ⚙️ 6. Créer les automatisations pour chaque bouton

Le principe :
* Quand un bouton est pressé, si il y a un joueur gagnant,on ignore (donc seul le premier appui compte).
* Quand un bouton est pressé, si il n'y a pas de joueur gagnant, alors le joueur est déclaré gagnant et les actions suivantes sont realisées:
    1. Définir la variable "joueur gagnant" à la valeur du joueur
    2. Allumer l'ampoule de la couleur du joueur gagnant
    3. Attendre le nombre de seconde définit sur "Tempo joueur gagnant" 
    4. Définir la variable "joueur gagnant" à la valeur 0
    5. Remettre la lumière dans l'état normale (lumière naturelle)

🔵 Automatisation pour le Joueur 1

```yaml
alias: Buzzer Joueur 1
trigger:
  - platform: device
    device_id: buzzer_joueur_1
    domain: zha
    type: remote_button_short_press
condition:
  - condition: state
    entity_id: light.lampe_buzzer
    state: "off"
action:
  - service: light.turn_on
    target:
      entity_id: light.lampe_buzzer
    data:
      color_name: "{{ states('input_select.couleur_joueur_1') }}"
      brightness: 255
  - service: input_number.set_value
    target:
      entity_id: input_number.score_joueur
    data:
      value: 1
mode: single
```

🔴 Automatisation pour le Joueur 2
```yaml
alias: Buzzer Joueur 2
trigger:
  - platform: device
    device_id: buzzer_joueur_2
    domain: zha
    type: remote_button_short_press
condition:
  - condition: state
    entity_id: light.lampe_buzzer
    state: "off"
action:
  - service: light.turn_on
    target:
      entity_id: light.lampe_buzzer
    data:
      color_name: "{{ states('input_select.couleur_joueur_2') }}"
      brightness: 255
  - service: input_number.set_value
    target:
      entity_id: input_number.score_joueur
    data:
      value: 2
mode: single
```

# 🔄 7. Ajouter un bouton “Reset”

On va créer un bouton pour remettre a zéro en cas de besoin.
On peut:
* utiliser un troisième bouton Zigbee, ou
* créer un bouton virtuel dans Home Assistant

Pour cela on va créer une automatisation qui, lorsqu'elle est déclenchée va:
1. Définir la variable "joueur gagnant" à la valeur 0
2. Remettre la lumière dans l'état normale (lumière naturelle)

```yaml
alias: Reset Buzzer
trigger:
  - platform: device
    device_id: bouton_reset
    domain: zha
    type: remote_button_short_press
action:
  - service: light.turn_off
    target:
      entity_id: light.lampe_buzzer
mode: single
```

# 🖼️ 8. Aperçu visuel du tableau de bord

On va créer un tableau de bord pour le jeu.
1. Dans ton tableau de bord principal
2. Passe en mode "Modfier le tableau de bord"
3. Ajoute une vue
4. Créé la vue souhaitée en insérant
     * Le bouton "Reset"
     * Les sélecteurs de couleurs des 2 joueurs
     * La variable "Joueur gagnant"
     * La variable ""Tempo joueur gagnant" "

```yaml
title: Buzzer Quiz
views:
  - title: Quiz
    path: quiz
    cards:

      - type: entities
        title: Paramètres des joueurs
        entities:
          - entity: input_select.couleur_joueur_1
            name: Couleur Joueur 1
          - entity: input_select.couleur_joueur_2
            name: Couleur Joueur 2
          - entity: input_number.score_joueur
            name: Joueur gagnant

      - type: light
        entity: light.lampe_buzzer
        name: Lampe Buzzer

      - type: button
        name: Reset Buzzer
        icon: mdi:restart
        tap_action:
          action: call-service
          service: input_number.set_value
          data:
            value: 0
          target:
            entity_id: input_number.score_joueur
```

# 🧪 9. Tester le système

1. Appuie sur le bouton “Reset”
2. Appuie sur un des deux boutons.
3. Vérifie que
    * La lampe s’allume dans la couleur du joueur
    * La variable "joueur gagnant" est définie à la valeur du joueur
    * La lampe s’éteint après le nombre de seconde définit sur "Tempo joueur gagnant" 
    * La variable "joueur gagnant" repasse à 0 après le nombre de seconde définit sur "Tempo joueur gagnant" 
4. Recommence l'opération avec le second bouton
5. Recommence l'opération avec l'un des 2 boutons et avant la fin du nombre de seconde définit sur "Tempo joueur gagnant", appui sur le bouton “Reset” et vérifie que
    * La lampe s’éteint après le nombre de seconde définit sur "Tempo joueur gagnant" 
    * La variable "joueur gagnant" repasse à 0 après le nombre de seconde définit sur "Tempo joueur gagnant" 

Le jeu peut recommencer.
