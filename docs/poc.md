# Proof of concept

Un *Proof Of Concept* (POC) est un processus qui permet de commencer le travail de diplôme en testant les technologies qui seront utilisées dans le projet. Il permet également de tester la faisabilité du projet, et de déterminer si les technologies choisies sont adaptées. L'idée est de réaliser un projet simple, mais qui implémente les fonctionnalités du projet qui ont le plus de risque de causer des points de friction.

## Objectifs

Réaliser une application qui permet à l'utilisateur de se connecter à un serveur, et de recevoir des messages de la part d'autres utilisateurs (non chiffrés). Cela permettra de tester la communication entre le client et le serveur, ainsi que la communication en temps réel entre les utilisateurs avec WebSocket.

L'API sera sécurisée avec les mécanismes classiques du web (JWT, HTTPS, CORS). La connexion utilisateur sera également possible avec un jeton TOTP (*Time-based One-Time Password*), et qui sera possible de relier à une application d'authentification comme Google Authenticator ou Authy.

Les objectifs de ce POC sont donc les suivants:

- Tester Flutter
- Tester Typescript
- Tester la communication entre le client et le serveur
- Tester la communication en temps réel entre les utilisateurs
- Implémenter une API sécurisée
  - JWT
  - CORS
  - Authentification TOTP

## Technologies

Les technologies utilisées seront les suivantes :

- [Flutter](https://flutter.dev/) pour la partie client
- [Typescript](https://www.typescriptlang.org/) pour la partie serveur
- [Bun](https://bun.sh) En tant que remplacement plus rapide de Node.JS
- [Fastify](https://fastify.dev/) pour le serveur HTTP et la gestion des routes
- [Prisma](https://www.prisma.io/) pour la gestion de la base de données
  
## Journal de bord

### Jeudi 30 novembre 2023

Aujourd'hui, je commence à planifier le POC de mon travail de diplôme. J'ai décidé de partir sur une application mobile, car cela me permettra de tester les technologies que j'ai choisies, et de voir si elles sont adaptées à mon projet.
Ces quelques semaines vont également me permettre de découvrir Flutter, et de confirmer mes compétences en TypeScript : ayant déjà réalisé une API en cours de PDA dans les technologies que je voulais utiliser, je vais pouvoir creuser le framework de routes que j'ai décidé d'utiliser.

J'ai décidé d'utiliser Flutter pour le client. Je voulais tout d'abord utiliser React Native car je voulais m'entraîner pour mon travail de diplôme, mais je me suis rendu compte que la bibliothèque de chiffrement que je voulais utiliser pour le projet ne fonctionnait pas sur ce dernier, la bibliothèque Signal utilisant WebCrypto qui est uniquement disponible sur navigateur ou NodeJS. Mes multiples tentatives de polyfill et d'installation de dépendances externes n'ayant menées à rien, j'ai décidé de choisir Flutter, pour son expérience développeur, son support, son développement *cross-platform* et sa bibliothèque Signal native.

J'ai également décidé d'utiliser TypeScript pour le serveur. Ayant déjà utilisé ce dernier auparavant (notamment pour l'API du cours de PDA, ainsi que quelques projets personnels), je suis confiant avec ce langage et pense qu'il est adapté à mon projet. L'écosystème de TypeScript étant également très développé, je pense que cela me permettra de gagner du temps afin de me concentrer sur le développement des fonctionnalités de l'application.

Prisma sera mon ORM de choix pour ce projet. J'ai aussi utilisé ce dernier pendant le développement de l'API du cours de PDA, et j'ai été très satisfait de son utilisation. Il permet de gérer la base de données de manière très simple, il gère les migrations, les seeds ainsi que les relations entre les tables avec un seul format de fichier (le schéma de la base de données). Les différentes parties de la bibliothèque sont même générées avec ce dernier, ce qui permet d'avoir une API typée et qui est toujours à jour avec le schéma de la base de données.

Je vais finalement utiliser Bun pour le développement ainsi que l'hébergement de l'application : ce dernier est récemment sorti en version 1.0.0, et est donc stable. Il est également très rapide, offrant des performances supérieures à NodeJS et permet de développer des applications en TypeScript sans avoir à passer par le transpileur officiel. De plus, le *runtime* lui-même offre une API permettant la réalisation d'un serveur de WebSocket, ce qui me permettra de ne pas avoir à utiliser une bibliothèque externe quand je réaliserais mon travail de diplôme (il sera également utilisé dans le projet de PDA), afin de pouvoir gérer la communication en temps réel. Ce dernier offre également des performances supérieures à Socket.io ou autres bibliothèques similaires.

Après avoir choisi les différentes technologies, je décide de m'attaquer à la planification des routes : idéalement,

### Jeudi 7 décembre 2023
