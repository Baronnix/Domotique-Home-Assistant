# 🏠 Tutoriel pour installer des composants de la communauté (HACS)

Ce guide explique comment ajouter HACS, installer des features communautaires (dont button-card) et créer une vue animée pour une ampoule Zigbee

# 📌 Objectifs du tutoriel

* Installer HACS (Home Assistant Community Store)
* Ajouter des intégrations communautaires utiles, notamment button-card
* Créer une vue personnalisée dans Lovelace
* Afficher les informations d’une ampoule Zigbee
* Utiliser button-card pour créer une carte avec icônes animées (état ON/OFF, transition, luminosité)

# 🚀 Prérequis

* Une installation fonctionnelle de Home Assistant
* Une ampoule Zigbee intégrée via Zigbee2MQTT
* Un compte GitHub (obligatoire pour HACS)
* Un navigateur web moderne

# 🧩 Installation de Terminal & SSH et HACS (Home Assistant Community Store)

HACS permet d’ajouter des intégrations et extensions communautaires.

La procédure d’installation HACS est compatible avec les versions de Home Assistant supérieures ou égales à 2026.

## Installation de Terminal & SSH

Étapes d’installation:
1. Aller dans Paramètres → Apps
2. Cliquer sur Installer l'appplication
3. Rechercher Terminal & SSH
4. Cliquer sur Installer
5. Démarrer Terminal & SSH

## Installation de HACS

Étapes d’installation:
1. Ouvrir Terminal & SSH dans Home Assistant
2. Exécuter la commande :
```bash
wget -O - https://get.hacs.xyz | bash -
```
3. Attendre la fin de l’installation
4. Redémarrer Home Assistant

## Activer l’intégration HACS dans Home Assistant

Une fois Home Assistant redémarré:
1. Aller dans Paramètres → Appareils & Services
2. Cliquer sur Ajouter une intégration
3. Recherche HACS
4. Sélectionne HACS – Home Assistant Community Store
5. Connecte-toi avec ton compte GitHub
6. Autorise l’accès

HACS apparaît maintenant dans la barre latérale.

# Installer des intégrations utiles via HACS

⭐ Intégrations recommandées
* button-card — Carte ultra personnalisable pour Lovelace
* card-mod — Permet de styliser les cartes avec CSS
* stack-in-card — Pour regrouper plusieurs cartes dans une seule
* layout-card — Mise en page avancée

# 📥 Installation de button-card

1. Ouvrir HACS → Frontend
2. Cliquer sur Explorer & télécharger des dépôts
3. Rechercher button-card
4. Installer
5. Redémarrer Home Assistant

# Créer une vue dédiée à l’ampoule Zigbee

🗂️ Exemple de vue Lovelace (YAML)
```yaml
title: Lumière Salon
path: lumiere-salon
icon: mdi:lightbulb-group
cards:
  - type: custom:button-card
    entity: light.salon
    name: Ampoule Salon
    icon: mdi:lightbulb
    layout: icon_name
    tap_action:
      action: toggle
    styles:
      icon:
        - color: >
            [[[ return entity.state === 'on' ? 'gold' : 'grey'; ]]]
      card:
        - border-radius: 12px
        - padding: 12px
```

# Ajouter des animations d’icônes avec button-card

Voici un exemple avancé avec animation de l’icône, variation de luminosité, et effet pulsation lorsque la lumière est allumée.

✨ Carte animée pour ampoule Zigbee
```yaml
type: custom:button-card
entity: light.salon
name: Ampoule Salon
icon: mdi:lightbulb
tap_action:
  action: toggle

state:
  - value: 'on'
    icon: mdi:lightbulb-on
    styles:
      icon:
        - animation: blink 1.5s ease-in-out infinite
      card:
        - background: 'rgba(255, 215, 0, 0.2)'
  - value: 'off'
    icon: mdi:lightbulb-off
    styles:
      icon:
        - filter: opacity(40%)

styles:
  card:
    - border-radius: 16px
    - padding: 16px
    - transition: 0.3s
  icon:
    - width: 40px
    - height: 40px

custom_fields:
  brightness: >
    [[[ return entity.state === 'on' ? entity.attributes.brightness + '%' : '' ]]]

extra_styles: |
  @keyframes blink {
    0% { opacity: 0.4; }
    50% { opacity: 1; }
    100% { opacity: 0.4; }
  }
```  

# Résultat final

Vous obtenez :
* Une vue dédiée à votre ampoule Zigbee
* Une carte interactive
* Une icône animée lorsque la lumière est allumée
* Une interface moderne et personnalisée