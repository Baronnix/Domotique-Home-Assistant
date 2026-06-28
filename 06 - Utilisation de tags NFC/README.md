# Automatisation Home Assistant : Contrôler un prise ou ampoule Zigbee avec un tag NFC

Ce tutoriel explique comment appairer une prise ou ampoule Zigbee, créer une automatisation dans Home Assistant et associer un tag NFC pour changer l’état de la prise ou ampoule (ON/OFF) en la scannant avec un smartphone compatible NFC.

# 📺 Vidéo

Lien Youtube: [https://www.youtube.com/watch?v=lWhhbuQt8qc](https://www.youtube.com/watch?v=lWhhbuQt8qc)

# ✨ Objectif

Scanner un tag NFC → Home Assistant reçoit l’événement → l’automatisation bascule l’état d’une prise ou ampoule Zigbee (allumer si éteinte, éteindre si allumée).

# 🧩 Prérequis

 * Une installation fonctionnelle de Home Assistant
 * Une prise ou ampoule Zigbee compatible (via Zigbee2MQTT)
 * Un coordinator Zigbee (clé USB ou hub)
 * Zigbee2MQTT et Mosquitto Broker installés et configurés
 * Un smartphone NFC
 * Un tag NFC vierge

# 📱 1. Installation de Home Assistant sur smartphone

 * Pour Android: 
    1. Ouvrir le Google Play Store
    2. Rechercher Home Assistant
    3. Installer l’application officielle
    4. Ouvrir l’application et se connecter à votre instance Home Assistant

 * Pour iOS (iPhone)
    1. Ouvrir l’App Store
    2. Rechercher Home Assistant
    3. Installer l’application officielle
    4. Lancer l’application et se connecter à votre instance

Pourquoi l’application est indispensable ?
 * Elle permet de scanner et enregistrer des tags NFC
 * Elle envoie les événements NFC directement à Home Assistant
 * Elle permet d’utiliser les capteurs du téléphone (batterie, localisation, etc.)

# 🔌 2. Appairage de la prise ou ampoule Zigbee

1. Mettre la prise ou ampoule Zigbee en mode appairage (souvent un bouton à maintenir 5–10 secondes).
2. Dans Home Assistant :
    1. Aller dans Zigbee2MQTT
    2. Autoriser l'appairage
    3. Attendre que la prise ou ampoule apparaisse dans la liste
    4. Vérifier qu’elle expose bien une entité de type switch.
    5. Renommer la prise ou ampoule(ex: prise_salon)
    6. Desactiver le mode appairage dans Zigbee2MQTT

# ⚙️ 3. Création de l’automatisation

Nous allons créer une automatisation qui :
 * se déclenche lorsqu’un tag NFC spécifique est scanné
 * bascule l’état de la prise ou ampoule (toggle)

Étapes: 
1. Aller dans Paramètres → Automatisations & Scènes
2. Cliquer sur Créer une automatisation
3. Choisir Démarrer avec une automatisation vide
     * Déclencheur: 
         * Type : Tag
         * Tag : sera ajouté plus tard (Home Assistant le remplira automatiquement après l’enregistrement du tag)
     * Action
         * Type : Appel de service
         * Service : switch.toggle
         * Cible : votre prise ou ampoule Zigbee (ex: switch.prise_salon ou light.ampoule_ikea_1)

Remplacer switch.toggle en light.toggle pour une ampoule

# 🏷️ 4. Enregistrement du tag NFC

1. Ouvrir l’application mobile Home Assistant
2. Aller dans Paramètres de l’application → Tags NFC
3. Appuyer sur Créer un tag
4. Approcher le smartphone du tag NFC
5. Donner un nom au tag (ex : toggle_prise_salon ou ampoule_ikea_1)

Home Assistant génère un Tag ID unique

# 🔗 5. Associer le tag NFC à l’automatisation

1. Retourner dans votre automatisation
2. Dans le déclencheur Tag, sélectionner le tag créé
3. Sauvegarder l’automatisation

# 🧪 6. Test

Approcher votre smartphone du tag NFC, les acions suivantes devraient se passer:
1. L’application Home Assistant détecte le tag
2. L’automatisation s’exécute
3. La prise ou ampoule Zigbee change d’état (ON ↔ OFF)

# 📁 Exemple de configuration YAML pour prise (optionnel)

```yaml
alias: tag auto prise
description: ""
triggers:
  - trigger: tag
    tag_id: 94ba2ff1-6f26-4286-8569-0ca4c01b6df6
conditions: []
actions:
  - action: switch.toggle
    metadata: {}
    target:
      entity_id: switch.prise_salon
    data: {}
mode: single
```

# 📁 Exemple de configuration YAML pour ampoule (optionnel)

```yaml
alias: tag auto ampoule
description: ""
triggers:
  - trigger: tag
    tag_id: 94ba2ff1-6f26-4286-8569-0ca4c01b6df6
conditions: []
actions:
  - action: light.toggle
    metadata: {}
    target:
      entity_id: light.ampoule_ikea_1
    data: {}
mode: single
```

# 🎯 Conclusion

Vous avez maintenant un système simple et efficace pour contrôler une prise Zigbee ou un autre appareil en scannant un tag NFC.

C’est idéal pour :
 * allumer une lampe en passant son téléphone
 * activer une machine à café
 * couper l’alimentation d’un appareil en quittant une pièce
 * automatiser des actions sans interface visible
 * ...