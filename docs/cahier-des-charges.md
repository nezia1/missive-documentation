# Cahier des charges

## Contexte

Durant la deuxième et dernière année du diplôme de technicien ES en développement d'applications, il est demandé aux étudiants de réaliser un travail de diplôme. Ce travail consiste en la réalisation d'un projet informatique, qui doit être réalisé seul, et qui doit être présenté à la fin de l'année. Ce projet doit être réalisé en 10 semaines, et doit être accompagné d'un journal de bord, qui décrit les différentes étapes de réalisation du projet, les choix techniques et les difficultés rencontrées, et d'une documentation utilisateur, qui décrit le fonctionnement de l'application et les différentes fonctionnalités dans un langage accessible au type d'utilisateur visé.

## Description du projet

L'objectif principal de ce projet est de développer une application de messagerie sécurisée qui garantit la confidentialité des messages échangés entre les utilisateurs grâce à l'implémentation du protocole de chiffrement Signal. L'application sera conçue pour fonctionner sur la majorité des plateformes (iOS, Android, Windows, macOS, Linux). Elle permettra aux utilisateurs de communiquer en temps réel de manière sécurisée, de synchroniser leurs conversations entre plusieurs appareils, et d'être disponible sur les principaux app stores. De plus, l'application sera hébergée chez Infomaniak pour garantir la disponibilité et la sécurité des données des utilisateurs.

## Spécifications techniques

### Messagerie sécurisée

L'application doit permettre aux utilisateurs d'envoyer des messages chiffrés de bout en bout à d'autres personnes. Ce chiffrement sera assuré par le protocole Signal, qui est décrit dans plusieurs documents de la fondation, et déjà implémenté par des entreprises de sécurité informatique dans une multitude de langages.

#### Le protocole Signal

Ce protocole fonctionne avec ce qu'on appelle un algorithme à *double ratchet* (roue à rochet / crantée) : il s'agit d'un algorithme  qui permet de chiffrer les messages de bout en bout depuis une clé partagée (protocole X3DH), et ensuite en utilisant une partie du message précédemment envoyé afin de s'assurer que si une des clés est compromise, les autres messages restent confidentiels.

##### Diffie-Hellman

Avant d'expliquer ce qu'est X3DH, qui est une partie importante du protocole Signal, il est nécéssaire de bien comprendre comment l'algorithme de Diffie-Hellman fonctionne.
Ce dernier est un algorithme permettant de générer des clés partagées qui sont les mêmes, ce qui permet d'attester de l'identité des utilisateurs qui communiquent. Cette clé partagée est formée en mélangeant la clé publique signée de l'envoyeur (précédemment envoyée sur le serveur), et la clé privée du destinataire. L'opération est ensuite réalisée en sens inverse. Cet algorithme génère une clé qui sera la même pour les deux utilisateurs, et qui permet d'attester de l'identité de l'autre utilisateur. Cette clé partagée sera ensuite affichée sur la conversation pour que les deux utilisateurs puissent attester de l'identité de l'autre si le besoin se présente.
![Diffie Hellman](./img/diffie-hellman.png)

##### X3DH (Extended Triple Diffie-Hellman)

Maintenant que nous avons expliqué Diffie-Hellman, nous allons nous intéresser à XD3H, qui est une partie importante du protocole Signal. Ce dernier reprend les mêmes concepts que Diffie-Hellman, mais génère une série de valeurs au lieu d'une seule clé partagée. Il est plus adapté aux contextes asynchrones, et permet de gérer automatiquement les cas où le destinataire serait hors-ligne, ou aurait son périphérique éteint.

Il fonctionne exactement de la même manière, mais utilise des valeurs différentes dont nous allons parler plus tard. X3DH récupère d'abord un bundle de pré-clés dont le contenu sera expliquée plus bas, et l'utilise pour générer 4 valeurs Diffie-Hellman, qui sont ensuite regroupées pour calculer une clé partagée secrète. Elle a le même but que Diffie-Hellman, à savoir établir un accord entre les deux utilisateurs, mais elle a l'avantage de contenir des clés éphémères (certaines des clés privées sont mêmes supprimées après utilisation), et permet d'encore renforcer la sécurité dans le cas où une des clés serait compromise.

###### Génération des clés

Après avoir installé l'application, la première étape du protocole est de générer une liste de clés, qui seront utilisées pour vérifier l'identité de l'utilisateur, et d'informer les autres utilisateurs de sa présence sur le serveur. Ces clés sont les suivantes :

- Une paire de clés d'identité (privée / publique)
- Une pré-clé signée (publique)
- Plusieurs pré-clés non signées

#### Communication en temps réel

Elle fonctionnera avec la mise en place d’un serveur WebSocket, qui permettra l’échange des messages en temps réel, ainsi qu’une API plus classique, qui permettra de gérer les comptes utilisateurs et les connexions.
Le serveur de WebSocket ainsi que l’API seront protégés par les mécanismes classiques au web (entête CORS, authentification avec jeton, chiffrement SSL).

Le serveur ne stockera jamais d'autres informations que les informations de connexion, et les messages en attente de réception. Les messages seront supprimés dès qu'ils auront été reçus par le destinataire. Cela permettra de garantir la confidentialité des messages, même dans le cas où le serveur serait compromis.

### Multiplateforme

L’application doit être compatible avec les principales plateformes mobiles (iOS / Android), ainsi que les différentes plateformes bureautiques (Windows macOS Linux). Le choix de React Native couplé à Tauri permettra de garder une seule base qui permettra ensuite de build automatiquement l’application, notamment grâce à fastlane.

### Synchronisation entre périphériques

L'application doit pouvoir permettre de synchroniser différents périphériques entre eux. Pour ce faire, un mécanisme de QR code sera mis en place, comme on peut le retrouver sur un grand nombre d'applications de messagerie. L'utilisateur pourra scanner le QR code de son appareil principal avec un autre appareil, et ainsi synchroniser les conversations et les messages entre les deux appareils.

### Hébergement

Les différents serveurs de l’application seront hébergés sur les serveurs / VPS de chez Infomaniak. Cette dernière étant basée sur la confidentialité des données, le choix d’un hébergeur suisse est donc crucial même pour ce projet, où les données ne seront conservées que sur le client.

### Déploiement

Le déploiement de l’application sera complètement automatisé grâce à fastlane, un service acquéri en 2017 par Google. Il permet le build automatique, le déploiement en accès anticipé sur les différentes plateformes mobiles (TestFlight, Play Console), la soumission sur les app stores, et il est complètement intégrable avec GitLab Runner pour autant que le serveur soit sous macOS (afin de pouvoir build pour iOS).

## Budget

### Hébergement Jelastic Cloud Infomaniak

L'application mobile sera hébergée sur la plateforme Jelastic Cloud d'Infomaniak. Les coûts d'hébergement débutent à partir de 20 .- par mois, mais ils seront adaptables en fonction de la consommation de ressources et de la configuration sélectionnée.

Coût estimé : À partir de 20 .- par mois (ajustable selon la consommation)

### Licence Développeur Apple

Une licence de développeur Apple, au coût de 99 $ par an, sera nécessaire pour accéder aux outils de développement, effectuer des tests sur des appareils réels, et soumettre l'application à l'examen d'Apple.

Coût estimé : 99 $ par an

### Licence Développeur Android

Pour la publication de l'application sur le Google Play Store, l'obtention d'une licence de développeur Android est requise. Cette licence est un paiement unique de 25 $.

Coût estimé : 25 $ (unique)

### Runner Gitlab macOS

Un runner Gitlab macOS sera utilisé pour automatiser la construction de l'application iOS. Les coûts associés commencent à partir de 15 .- par mois, mais peuvent varier en fonction des spécifications et de l'utilisation.

Coût estimé : À partir de 15 .- par mois (variable selon les spécifications)
