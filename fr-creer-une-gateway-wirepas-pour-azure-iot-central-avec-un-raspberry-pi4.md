# Créer une gateway Wirepas pour Azure IoT Central avec un Raspberry Pi4

Tout au long de ce tutoriel, nous allons voir comment créer une gateway Wirepas pour Azure IoT Central au travers d'un exemple simple.

Il sera question :

* d'envoyer des données de télémétrie à partir de tags Ruuvi vers Azure IoT Central
* d'associer un tag avec une ampoule Philips Hue et d'afficher la couleur correspondant à la température mesurée
* de construire une application Azure IoT Central exploitant ces données et d'envoyer des alertes en cas de dépassement de seuil en faisant clignoter l'ampoule correspondante.

Dans une première partie, nous allons voir les prérequis matériel et logiciel, puis la préparation de la gateway ainsi que des ampoules Philips Hue. Puis, nous verrons la création et la configuration d'une application Azure IoT Central pour notre cas de test. Enfin, nous verrons comment modifier notre gateway pour l'envoi de données et la réception d'alertes et visualiser les données dans l'application Azure IoT Central.

## Prérequis

### Matériel

Pour le matériel, nous allons avoir besoin de :

* 1 Raspberry Pi4 4GB+ avec Raspbian Lite installé : [https://www.raspberrypi.org/products/raspberry-pi-4-model-b/#find-reseller](https://www.raspberrypi.org/products/raspberry-pi-4-model-b/#find-reseller)
* 1 dongle USB u-blox NINA-B1 avec Wirepas 5 : [https://store.clemanis.com/fr/iot/1238-dongle-usb-u-blox-nina-b1-avec-wirepas-40.html#/710-option_protection_usb-oui/727-wirepas-wirepas_50](https://store.clemanis.com/fr/iot/1238-dongle-usb-u-blox-nina-b1-avec-wirepas-40.html#/710-option_protection_usb-oui/727-wirepas-wirepas_50)
* 3 tags Ruuvi avec Wirepas 5 : [https://store.clemanis.com/fr/iot/1202-ruuvi-wirepas.html#/727-wirepas-wirepas_50](https://store.clemanis.com/fr/iot/1202-ruuvi-wirepas.html#/727-wirepas-wirepas_50)
* 1 kit de démarrage Philips Hue avec un pont et 3 ampoules White and Color : [https://www.philips-hue.com/fr-fr/p/hue-white-and-color-ambiance-kit-de-demarrage-e27/8718699695484](https://www.philips-hue.com/fr-fr/p/hue-white-and-color-ambiance-kit-de-demarrage-e27/8718699695484)

### Logiciels

Pour la partie logicielle, nous allons utiliser :

* le sinkService de Wirepas pour collecter les messages envoyés par les capteurs
* le TransportService de Wirepas pour envoyer les messages à un broker MQTT
* Mosquitto faisant office de broker MQTT
* Node-RED pour gérer les flux d'extraction, de transformation et d'envoi des données vers Azure IoT Central ainsi que la récupération des alertes.

## Préparation de la gateway Wirepas

Pour commencer, il suffit de suivre le tutoriel suivant disponible en français : [Prototyper une gateway Wirepas avec un Raspberry Pi4](https://github.com/rackmatrix/wirepas-tutorials/blob/master/fr-prototyper-une-gateway-wirepas-avec-un-raspberry-pi4.md).

Une fois la gateway Wirepas préparée, vous devriez déjà pouvoir constater la bonne réception de données via la console de debug de Node-RED.

## Préparation des ampoules Philips Hue

Pour cette étape, je vous conseille de suivre la documentation officielle de Philips : [Get Started - Philips Hue Developer Program](https://developers.meethue.com/develop/get-started-2/). Pour avoir accès à plus de documentation concernant l'API, il faut créer un compte sur le site [Philips Hue Developer Program](https://developers.meethue.com/).

Cela vous permettra de mettre en route, si ce n'est déjà fait, le kit de démarrage, de créer un utilisateur `developpeur` et de voir comment fonctionne l'API REST.

Cela devrait vous permettre aussi d'identifier les 3 ampoules. Je vous recommande de noter l'identifiant de chaque ampoule afin de pouvoir ensuite l'associer à l'un des tags Ruuvi que l'on va utiliser.

## Azure IoT Central

### Création de l'application Azure IoT Central

Depuis la page d'accueil du portail Azure ([https://portal.azure.com](https://portal.azure.com)), vous cliquez sur `Create a resource`, puis dans `Internet of Things` vous séletionnez `IoT Central application`.

Vous pouvez choisir le nom de la ressource ainsi que son URL. Vous devez l'associer à une souscription Azure ainsi qu'à un groupe de ressource. Enfin, vous devez choisir un `Pricing plan`, une `Template` ainsi qu'une localisation.

Dans notre cas : 

* le `Pricing plan` `Standard 1` est suffisant.
* le `Template`à utiliser est `Custom application`.
* la localisation `Europe`.

Une fois que vous avez cliquer sur `Create`, le deploiement de l'application Azure IoT Central se lance.

### Accès à l'application Azure IoT Central.

Une fois le déploiement terminé, vous pouvez accéder à l'application au moyen d'une URL du type :

    https://nom_de_la_ressource.azureiotcentral.com

La page d'accueil est constitué par un tableau de bord vierge contenant des liens vers diverses documentations.

### Création d'un modèle d'appareil

Dans notre cas, nous souhaitons envoyer des données à notre application en provenance de nos tags Ruuvi.

Pour cela, il nous faut créer un modèle d'appareil correspondant à nos capteurs et aux données envoyées par la gateway Wirepas que nous avon préparé précédemment.

Lorsque vous cliquez sur `Modèles d'appareils` puis sur `Nouveau`, vous pouvez sélectionner `Appareil IoT` dans la section `Créer un modèle d'appareil personnalisé` puis cliquer sur le bouton `Suivnat: Personnaliser`. Il ne vous reste plus qu'à saisir le nom du modèle que vous souhaitez : `RuuviTagWirepas` ou `RuuviTag` par exemple. Pour finir, il faut cliquer sur `Etape suivante : Examiner` puis sur `Créer`.

Le modèle d'appareil est créé mais vide. Il vous d'ailleurs demander si vous souhaitez créer un modèle de capacité vous-même ou bien l'importer. Pour faire simle, vous pouvez importer le modèle de capacité présent de le dépôt : [RuuviTagWirepas.json](RuuviTagWirepas.json).

Une fois importé, le modèle d'appareil contient l'interface définit dna le fichier `json` à l'état de brouillon : il est encore possible de la modifier. Ici, l'interface est suffisante pour notre besoin, vous pouvez la publier.

Vous pouvez aussi configurer des affichages qui seront utilisés pour visualiser les données envoyées par un capteur. Dans mon test, j'ai utilisé l'élément `Générer des vues par défaut` qui suffit amplement pour notre cas.

### Création d'un appareil

Pour la création des appareils, il suffit de choisir le modèle, un nom et un identifiant. Par défaut, il est proposé un nom et un identifiant généré automatiquement.

Dans notre cas, nous allons choisir le nom et l'identifiant :

 * pour le nom, je vous conseille de mettre quelque chose qui vous permette d'identifier facilement l'appareil. Dans mon cas, j'ai choisi de mettre la localisation en plus de l'identifiant du tag.
 * pour l'identifiant, il vaut mieux mettre l'identifiant du tag. Pour la suite du tutoriel, il est préférable d'utiliser `RuuviTag` suivi de l'identifiant du tag présent sur l'étiquette situé en dessous du capteur.

### Création d'un tableau de bord

Une fois les appareils créés, il est possible de créer un tableau de bord ou de modifier celui par défaut.

Dans mon test, j'ai créé un tableau de bord avec une vignette sur le groupe d'appareils par défaut et en sélectionnant tous les appareils du groupe. Puis, j'ai choisi la température et l'humidité dans les données de télémétrie. Une fois la vignette ajoutée sur le tableau de bord, j'ai modifié (icône `roue`) la vignette pour que la plage d'affichage soit sur les dernières 24h (`1 jour avant`) et j'ai choisi une couleur pour chacune des données de télémétrie.

Pour l'instant, le tableau de bord est vide : nous n'envoyons pas encore de données à notre application Azure IoT Central.

### Création d'une règle

Même si nous n'avons pas encore de données, nous allons créer une règle : envoyer une alerte à notre gateway dès que la température dépasse 20°C. Cela se fera en 2 temps : la création de la règle dans Azure IoT Central puis la réception et le traitement de l'alerte dans la gateway.

Ici, nous allons traiter la première partie : la seconde sera traitée dans la section suivante.

Dans la page de création d'une nouvelle règle, il suffit de choisir un modèle d'appareil puis de sélectionner une condition : `Temperature` pour la télémétrie, `Est supérieur ou égal à` pour l'opérateur et `20.0` pour la valeur. Concernant l'action, il faut choisir `Webhook` en mettant dans l'URL, une adresse permettant d'accéder au Node-RED installer sur votre Raspberry Pi.

Etant donné, que ce sera Node-RED qui recevra les alertes, il convient de mettre en place les bonnes ouvertures et redirections de port dans votre routeur/box internet. Le port utilisé par Node-RED est le `1880`.

Le choix du `path` pour l'adresse de l'alerte peut être personnlisé : il sera à configurer dans Node-RED plus tard.

Dans mon cas, j'ai choisi :

    http://IP_OU_DOMAINE_DU_RPI4:1880/alerts/TemperatureUpper20

Maintenant que notre application Azure IoT Central est configurée, il ne reste plus qu'à envoyer des données et configurer la réception des alertes. 

## Envoi de données à Azure IoT Central et réception d'alertes

Avant toute chose, il faut installer une extension pour Node-RED permettant d'envoyer des données var Azure IoT. L'idéal est d'utiliser cette ligne de commande : 

    sudo npm i -g node-red-contrib-azure-iot-hub
    sudo systemctl restart nodered

Cela va nous permettre d'utiliser un noeud `Azure IoT Hub` pour envoyer les données vers le Hub de notre application Azure IoT Central. Ce noeud nécessite de connaitre l'adresse du Hub. Cependant, ce n'est pas une information accessible directement.

Pour cela, il nous faut utiliser le paquet npm `dps-keygen` disponible sur le GitHub Azure : [https://github.com/Azure/dps-keygen](https://github.com/Azure/dps-keygen). Vous pouvez l'installer aussi sur la gateway que votre propre machine au moyen de cette commande :

    sudo npm i -g dps-keygen

Les prochaines sections seront à répéter pour chaque appareil que l'on souhaite connecter à Azure IoT Central.

Si vous souhaitez aller plus vite, vous pouvez consulter la dernière section de ce chapitre intitulée `Notes`.

### Connexion à Azure IoT Central

Pour obtenir l'adresse du Hub, il faut au préalable récupérer le scope (étendue) et la clé d'un des appareils. Vous trouverez ces informations en ouvrant la page de détail d'un appareil puis en cliquant sur `Connexion` en haut à gauche :

 * Clé : champ `Clé primaire`
 * Scope : champ `Etendue de l'ID`

Puis, il faut lancer la commande suivante :
 
     dps-keygen -di:identifiant_d_un_appareil -dk:cle -si:scope

Vous obtiendrez une `connectionString` dans laquelle le champ `HostName` contient l'adresse du Hub de notre application.

### Envoi de données à Azure IoT Central

Maintenant, que nous avons tous les renseignements nécessaires, nous pouvons modifier le flux de données de notre gateway pour qu'il envoi des données vers Azure IoT Central.

Pour commencer, il faut ajouter un noeud `switch` entre le noeud `base64` et les noeud de télémétrie (`Temperature`, `Humidity`, ...) pour sélectionner les données en provenance de l'appareil que l'on est en train de connecter.

La propriété à utiliser est `msg.payload.wirepas.packetReceivedEvent.sourceAddress` en ayant sélectionner le préfixe `msg`, l'opérateur est `==` et la valeur correspond à l'identifiant du capteur présent sur l'étiquette en dessous de celui-ci.

Ensuite, il nous faut créer un noeud `change` pour modifier le format de la propriété `payload` par celui-ci (expression JSONata) :

    {
        "deviceId": "",
        "key": "",
        "protocol": "mqtt",
        "data": msg.payload.data
    }
     
Il s'agit d'une opération `Set` sur la propriété `payload` au moyen d'une expression JSONata. Il ne reste plus qu'à saisir la clé utilisée précédemment et à relier les 4 noeuds `Prepare data object` à ce nouveau noeud que vous pouvez nommer comme vous le souhaitez.

Enfin, il faut ajouter un noeud `Azure IoT Hub` qui sera à relier au noeud précédemment créé. Il faut choisir `mqtt` pour le protocole et utiliser l'adresse obtenue dans `dps-keygen` dans le champ `Hostname`.

### Afficher la couleur sur l'ampoule correspondante

Pour cela, il faut ajouter un noeud de type `switch` après le noeud `Prepare data object` de la température. Ce noeud est à ajouter indépendamment du reste du flux : ce n'est pas un noeud à insérer entre 2 autres.

Pour ce noeud, il faut convertir la température en teinte (`hue`) dans une valeur comprise entre 0 et 65536. L'objectif étant d'afficher une couleur entre le bleu et le violet, j'ai créé une expression JSONata permettant de le faire.

Dans ce noeud, il s'agit d'une opération de type `Set` sur la propriété `payload` (prefixe `msg`) dont l'expression JSONata est la suivante :

    (
        $minHue := 35000;
        $maxHue := 65000;
        $minTemp := -10;
        $maxTemp := 40;
        $temperature := payload.data.temperature < $minTemp
            ? $minTemp
            : payload.data.temperature > $maxTemp
                ? $maxTemp
                : payload.data.temperature
        ;
    
        $hueFactor := $floor(($maxHue - $minHue) / ($maxTemp - $minTemp));
        $hue := $minHue + $floor(($temperature - $minTemp) * $hueFactor);
    
        {
            "on": true,
            "sat": 254,
            "bri": 254,
            "hue": $hue
        }
    )

Enfin, il faut ajouter un noeud de type `http request` avec méthode `PUT` et l'URL correspondant à l'ampoule correspondante :

http://ip_bridge_philips_hue/api/id_developer/lights/id_light_bulb/state

Les informations `ip_bridge_philips_hue`, `id_developer` et `id_light_bulb` sont à remplacer par les informations correspondant à votre installation Philips Hue.

### Déploiement

Il ne vous reste plus qu'à déployer le flux et à passer au capteur suivant en reprenant au debut du chapitre.

### Notes

Pour aller plus vite, vous pouvez importer le fichier json [wirepas-azure-iot-central.json](wirepas-azure-iot-central.json) pour chaque capteur.

Il ne vous restera plus qu'à :

* Saisir les identifiants du broker MQTT Mosquitto dans le noeud `LocalMosquitto` : il faut éditer (icône `crayon`) le serveur au niveau de l'onglet `Security`.
* Saisir l'identifiant du capteur dans le noeud `Select messages from RuuviTag`.
* Saisir la clé et l'identifiant du capteur dans l'expression JSONata du noeud `Prepare Azure IoT Central message`.
* Saisir l'adresse du Hub dans le noeud `Azure IoT Central`.
* Saisir l'URL dans le noeud `Change color light` (voir la section `Afficher la couleur sur l'ampoule correspondante` pour le format de l'URL).

Une fois le déploiement effectué, vous devriez voir passer des messages indiquant l'envoi des données : message du type `Message sent.`

### Réception des alertes

Pour configurer la réception d'alerte, il suffit d'importer le fichier [azure-iot-central-alerts.json](azure-iot-central-alerts.json).

Il suffit de dupliquer, pour chaque capteur, les noeuds `Select Alert for RuuviTag`, `Prepare AlertMessage` et `Send Alert to Group/Light` puis :

* d'indiquer l'identifiant de l'appareil (identifiant Azure IoT Central) dans le noeud `Select Alert for RuuviTag`.
* de renseigner l'URL du groupe correspondant à l'ampoule associée au capteur.

L'URL est de la forme :

    http://ip_bridge_philips_hue/api/id_developer/groups/id_group/action
    
Les informations `ip_bridge_philips_hue`, `id_developer` et `id_group` sont à remplacer par les informations correspondant à votre installation Philips Hue.

Il ne reste plus qu'à déployer le flow.

## Voir les données dans Azure IoT Central

Il faut attendre environ 20 à 30 minutes après le premier `Message sent.` pour que des données apparaissent sur le tableau de bord.

Vous pouvez tout de même observer l'arrivée de données en consultant la page de détail d'un appareil au niveau de l'onglet `Données brutes`.

Maintenant, c'est à vous de jouer pour personnaliser votre application Azure IoT Central, améliorer l'envoi de données, ajouter de nouvelles fonctionnalités.
