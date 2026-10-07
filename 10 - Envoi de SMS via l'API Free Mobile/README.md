# 📱 Home Assistant : Envoi de SMS via l'API Free Mobile

Ce tutoriel explique comment mettre en place un système d'envoi de SMS dans Home Assistant grâce à l'API de Free Mobile, puis comment l'utiliser dans n'importe quelle automatisation de manière générique. L'API permet à un abonné Free Mobile de s'envoyer des notifications SMS sur son propre numéro à l'aide d'une simple requête HTTP.

# 📺 Vidéo

Lien Youtube: [https://www.youtube.com/@Baronnix/playlists](https://www.youtube.com/@Baronnix/playlists)

# 🎯 Objectif

À la fin de ce guide, vous disposerez :

* ✅ D'une méthode unique réutilisable dans toutes vos automatisations
* ✅ D'un interrupteur permettant d'activer ou non les SMS
* ✅ D'exemples d'utilisation avec des capteurs Home Assistant
* ✅ D'une architecture simple à maintenir
* ✅ D'informations sur les alternatives proposées par les autres opérateurs français

# 📋 Prérequis

Vous devez disposer :
* D'un abonnement Free Mobile.
* D'un Home Assistant fonctionnel.
* D'un accès à votre espace abonné Free Mobile.
* D'un moyen d'éditer votre configuration YAML (Studio Code Server ici).

# 1. 🔑 Récupération de la clé API Free Mobile

1. Connectez-vous à votre espace abonné Free Mobile
2. Allez dans Mon Compte → Mes Options → Notifications par SMS
3. Activez l'option puis récupérez :
    * votre identifiant (user)
    * votre clé API (pass)
4. L'API utilise une URL de ce type :
    * https://smsapi.free-mobile.fr/sendmsg?user=12345678&pass=abcdefgh12345678&msg=Bonjour

# 2. 🔘 Création d'un interrupteur global

Nous allons créer un booléen permettant d'activer ou couper tous les SMS sans modifier les automatisations.
1. Ajoutez dans configuration.yaml :
```yaml
input_boolean:
  sms_notifications:
    name: Notifications SMS
    icon: mdi:message-text
```
2. Ouvrez Paramètres → Outils de développement dans Home Assistant.
3. Vérifiez la configuration
4. Corrigez la configuration si nécessaire
5. Si la configuration est valide, Redémarrez avec un rechargement rapide 
6. Après redémarrage de Home Assistant, un interrupteur apparaîtra dans l'interface.
7. Ajoutez un interrupteur dans votre Dashboard
    * ON: Les SMS peuvent être envoyés
    * OFF: Aucun SMS n'est envoyé

# 3. 🌐 Création de la commande REST

1. Ajoutez dans configuration.yaml :
```yaml
rest_command:
  free_mobile_sms:
    url: >
      https://smsapi.free-mobile.fr/sendmsg?user=VOTRE_USER&pass=VOTRE_PASS&msg={{ message | urlencode }}
    method: GET
```
2. Remplacez dans le code précédent, les clés suivantes par les informations récupérées précédemment:
    * VOTRE_USER
    * VOTRE_PASS
3. Ouvrez Paramètres → Outils de développement dans Home Assistant.
4. Vérifiez la configuration
5. Corrigez la configuration si nécessaire
6. Si la configuration est valide, Redémarrez avec un rechargement rapide 
7. Après redémarrage de Home Assistant, un nouveu service apparaîtra dans l'interface.

# 4. 🧩 Création du script générique

L'objectif est que toutes les automatisations appellent le même script.

1. Ajoutez dans scripts.yaml (ou créez un nouveau script et passez en mode d'édition au format yaml):
```yaml
send_sms:
  alias: Envoyer un SMS
  mode: queued

  fields:
    message:
      description: Message à envoyer

  sequence:
    - condition: state
      entity_id: input_boolean.sms_notifications
      state: "on"

    - service: rest_command.free_mobile_sms
      data:
        message: "{{ message }}"
```
2. Ouvrez Paramètres → Outils de développement dans Home Assistant.
3. Vérifiez la configuration
4. Corrigez la configuration si nécessaire
5. Si la configuration est valide, Redémarrez avec un rechargement rapide 
6. Après redémarrage de Home Assistant, un nouveau script apparaîtra dans l'interface.

# 5. ⚙️ Fonctionnement

Lorsqu'une automatisation appelle: 

```yaml
service: script.send_sms
```

Le système :
1. Vérifie l'état du booléen.
2. Si le booléen est activé :
    * le SMS est envoyé.
3. Si le booléen est désactivé :
    * aucun SMS n'est envoyé.

Cette logique est centralisée dans un seul endroit.

## 📨 Exemple 1 : Envoi d'un message fixe

```yaml
action:
  - service: script.send_sms
    data:
      message: "La porte d'entrée est ouverte"
```

SMS reçu: La porte d'entrée est ouverte

## 🌡️ Exemple 2 : Envoi de la valeur d'un capteur

Supposons le capteur: sensor.temperature_salon

Automatisation :
```yaml
automation:
  - alias: Alerte température élevée

    trigger:
      - platform: numeric_state
        entity_id: sensor.temperature_salon
        above: 30

    action:
      - service: script.send_sms
        data:
          message: >
            Alerte température !
            Le salon est actuellement à
            {{ states('sensor.temperature_salon') }} °C.
```

SMS reçu: Alerte température ! Le salon est actuellement à 31.6 °C.

## 🚨 Exemple 3 : Capteur binaire

Supposons le capteur: binary_sensor.alarme_maison

Automatisation :
```yaml
automation:
  - alias: Notification état alarme

    trigger:
      - platform: state
        entity_id: binary_sensor.alarme_maison

    action:
      - service: script.send_sms
        data:
          message: >
            Etat de l'alarme :
            {% if is_state('binary_sensor.alarme_maison', 'on') %}
              ACTIVÉE
            {% else %}
              DÉSACTIVÉE
            {% endif %}
```

SMS reçu (2 alternatives): 
    * Etat de l'alarme : ACTIVÉE
    * Etat de l'alarme : DÉSACTIVÉE

# 🚀 Utilisation dans toutes les automatisations

Une fois le script créé, toutes vos notifications se résument à :
```yaml
action:
  - service: script.send_sms
    data:
      message: "Contenu du SMS"
```

Par exemple :
```yaml
message: "Détection de mouvement dans le garage"
```
```yaml
message: "Lave-vaisselle terminé"
```
```yaml
message: "Présence détectée dans le jardin"
```

# ⭐ Variante recommandée : gestion des alertes critiques

Pour éviter de recevoir trop de SMS, vous pouvez ajouter un second booléen:

```yaml
input_boolean:
  sms_notifications:
    name: Notifications SMS

  sms_alertes_critiques:
    name: Alertes SMS critiques
```

Votre script peut alors gérer :
* les notifications classiques ;
* les alertes importantes ;
* un mode "vacances" ;
* un mode "alarme".

C'est souvent la méthode préférée dans les installations Home Assistant importantes.

# 🇫🇷 Les autres opérateurs proposent-ils la même chose ?

## 🟢 Free Mobile

Free Mobile propose une API SMS simple et gratuite permettant à un abonné de recevoir des notifications sur sa propre ligne via une requête HTTP. C'est aujourd'hui la solution la plus simple à intégrer dans Home Assistant.

## 🟠 Orange

Orange dispose d'API SMS destinées aux développeurs et aux entreprises. Elles nécessitent généralement la création d'une application et une authentification OAuth. Il ne s'agit pas d'une fonctionnalité équivalente intégrée aux forfaits mobiles grand public.

## 🔴 SFR

Aucune API grand public équivalente à celle de Free Mobile n'est proposée pour les particuliers. Les solutions SMS disponibles chez SFR passent principalement par des offres professionnelles dédiées.

## 🔵 Bouygues Telecom

Bouygues Telecom propose diverses API destinées aux entreprises via son portail développeur, mais aucune API SMS grand public comparable à celle de Free Mobile n'a été identifiée.

# 💡 Alternatives populaires sous Home Assistant

Si vous ne disposez pas d'une ligne Free Mobile, vous pouvez utiliser:
* 📲 Application Companion Home Assistant
* ✈️ Telegram
* 🔔 Pushover
* 💬 Discord
* 📡 Signal
* 📞 Modem GSM (SIM800, SIM7600, Huawei 4G, etc.)
* 🟢 WhatsApp via services tiers

# ✅ Conclusion

L'API SMS de Free Mobile constitue aujourd'hui la solution la plus simple pour les utilisateurs Home Assistant souhaitant recevoir des alertes critiques sans dépendre d'une application mobile.

Avec cette architecture :

* ✅ Une seule configuration API
* ✅ Un script centralisé
* ✅ Un interrupteur global d'activation
* ✅ Des messages dynamiques via Jinja2
* ✅ Une intégration simple dans toutes les automatisations Home Assistant

Vous obtenez ainsi un système robuste, maintenable et facilement extensible pour toutes vos notifications domotiques importantes. 🚀