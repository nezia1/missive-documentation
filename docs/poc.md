# POC (Proof Of Concept)

Un POC (Proof Of Concept) est un processus qui permet de commencer le travail de diplôme en testant les technologies qui seront utilisées dans le projet. Il permet également de tester la faisabilité du projet, et de déterminer si les technologies choisies sont adaptées. L'idée est de réaliser un projet simple, mais qui implémente les fonctionnalités du projet qui ont le plus de risque de causer des points de friction.

## Objectifs

Réaliser une application qui permet à l'utilisateur de se connecter à un serveur, et de recevoir des messages de la part d'autres utilisateurs (non chiffrés). Cela permettra de tester la communication entre le client et le serveur, ainsi que la communication en temps réel entre les utilisateurs avec WebSocket.

L'API sera sécurisée avec les mécanismes classiques du web (JWT, HTTPS, CORS). La connexion utilisateur sera également possible avec un jeton TOTP (Time-based One-Time Password), et qui sera possible de relier à une application d'authentification comme Google Authenticator ou Authy.

Les objectifs de ce POC sont donc les suivants:

- Tester Flutter
- Tester Typescript
- Tester la communication entre le client et le serveur
- Tester la communication en temps réel entre les utilisateurs
- Implémenter une API sécurisée
  - JWT
  - CORS
  - Authentification TOTP
