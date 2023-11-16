# Cahier des charges

## Contexte

Durant la deuxième et dernière année du diplôme de technicien ES en développement d'applications, il est demandé aux étudiants de réaliser un travail de diplôme. Ce travail consiste en la réalisation d'un projet informatique, qui doit être réalisé seul, et qui doit être présenté à la fin de l'année. Ce projet doit être réalisé en 10 semaines, et doit être accompagné d'un journal de bord, qui décrit les différentes étapes de réalisation du projet, les choix techniques et les difficultés rencontrées, et d'une documentation utilisateur·rice, qui décrit le fonctionnement de l'application et les différentes fonctionnalités dans un langage accessible au type d'utilisateur·rice visé·e.

## Description du projet

L'objectif principal de ce projet est de développer une application de messagerie sécurisée qui garantit la confidentialité des messages échangés entre les utilisateur·rice·s grâce à l'implémentation du protocole de chiffrement Signal. L'application sera conçue pour fonctionner sur la majorité des plateformes (iOS, Android, Windows, macOS, Linux). Elle permettra aux utilisateur·rice·s de communiquer en temps réel de manière sécurisée, de synchroniser leurs conversations entre plusieurs appareils, et d'être disponible sur les principaux app stores. De plus, l'application sera hébergée chez Infomaniak pour garantir la disponibilité et la sécurité des données des utilisateur·rice·s.

## Spécifications techniques

### Messagerie sécurisée

L'application doit permettre aux utilisateur·rice·s d'envoyer des messages chiffrés de bout en bout à d'autres personnes. Ce chiffrement sera assuré par le protocole Signal. Ce dernier a été retenu car il est tout d'abord open-source (il est très facile de trouver des implémentations dans différents langages), et la documentation du protocole est extrêmement bien fournie. Une explication de ce protocole est disponible [sur ce document](./protocole-signal.md).

#### Communication en temps réel

Elle fonctionnera avec la mise en place d’un serveur WebSocket, qui permettra l’échange des messages en temps réel, ainsi qu’une API plus classique, qui permettra de gérer les comptes utilisateur·rice·s, les connexions, ainsi que les messages stockés temporairement en attente de réception.
Ils fonctionneront en parallèle : le serveur de WebSocket utilisera également l'API afin de pouvoir gérer les messages en attente de réception.
Le serveur de WebSocket ainsi que l’API seront protégés par les mécanismes classiques au web (entête CORS afin d'éviter les CSRF / requêtes depuis des périphériques non autorisés, authentification avec jeton pour s'assurer de l'identité des utilisateurs, et chiffrement SSL avec HTTPS).

Le serveur ne stockera jamais d'autres informations que les informations de connexion, et les messages en attente de réception. Les messages seront supprimés dès qu'ils auront été reçus par le destinataire. Cela permettra de garantir la confidentialité des messages, même dans le cas où le serveur serait compromis.

### Multiplateforme

L’application doit être compatible avec les principales plateformes mobiles (iOS / Android), ainsi que les différentes plateformes bureautiques (Windows macOS Linux). Le choix de Flutter permettra de garder une seule base qui permettra ensuite de build automatiquement l’application, notamment grâce à fastlane.

### Hébergement

Les différents serveurs de l’application seront hébergés sur les serveurs / VPS de chez Infomaniak. Cette dernière étant basée sur la confidentialité des données, le choix d’un hébergeur suisse est donc crucial même pour ce projet, où les données ne seront conservées que sur le client.

### Déploiement

Le déploiement de l’application sera complètement automatisé grâce à fastlane, un service acquéri en 2017 par Google. Il permet le build automatique, le déploiement en accès anticipé sur les différentes plateformes mobiles (TestFlight, Play Console), la soumission sur les app stores, et il est complètement intégrable avec GitLab Runner pour autant que le serveur soit sous macOS (afin de pouvoir build pour iOS).

### Tests et validation

Une batterie de tests unitaires et E2E sera déployée grâce aux outils de test de Flutter pour le client, et un framework plus classique pour la partie serveur. Cela permettra de garantir le bon fonctionnement du projet, et de pouvoir détecter les éventuels bugs et problèmes de sécurité avant même de déployer l'application. Ces tests seront automatiquement exécutés avec la pipeline CI/CD, afin de garantir que chaque commit est fonctionnel.

### Maintenance

La maintenance de l'application sera assurée grâce à des mises à jour automatiquement publiées sur les app stores grâce à fastlane. Cela permettra de garantir la sécurité des utilisateur·rice·s, et de corriger les éventuels bugs qui pourraient être découverts.

## Technologies utilisées

### Flutter

Le framework Flutter sera utilisé afin de réaliser le client. Ce dernier permettra de pouvoir réaliser une application multi-plateformes, qui fonctionne sur iOS, Android, Windows, macOS et Linux. L'utilisation de React Native a été considérée, mais Flutter a été retenu car la bibliothèque du protocole Signal ne fonctionnait pas (WebCrypto, l'API de chiffrement, n'est malheureusement pas disponible sur React Native, malgré les tentatives de polyfills et d'installation de différentes solutions alternatives).

### Typescript

Le back-end de l'application sera réalisé en Typescript, couplé à Bun, nouveau runtime qui offre des performances nettement supérieures à Node.JS, et qui est disponible en version 1.x depuis maintenant quelques mois. Typescript a été retenu pour sa facilité d'utilisation, sa solidité, et sa compatibilité avec les différentes bibliothèques utilisées.

### Docker

Docker sera utilisé afin de pouvoir déployer le back-end de manière simple et reproductible. Un docker-compose sera réalisé, qui permettra de déployer l'application en une seule commande, et également de la mettre à jour avec les différents tags et versions proposés.

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
