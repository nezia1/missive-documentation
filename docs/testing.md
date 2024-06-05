---
hide: navigation
---
# Plan de tests


## API

### Introduction

Ce plan détaille les tests fonctionnels prévus pour valider les fonctionnalités de l'API Missive, qui gère les tokens d'accès et de rafraîchissement ainsi que les informations utilisateur.

### URL

**Production** : `https://missive.nezia.dev/api/v1`

### Authentification

Toutes les requêtes nécessitent une authentification par token Bearer, sauf la route `POST /tokens` et `PUT /tokens` qui est utilisée pour l'authentification.

### Scénarios de Test

| Endpoint                 | Méthode | Description                                        | Objectif                                           | Code Attendu   | Statut | Impact |
| ------------------------ | ------- | -------------------------------------------------- | -------------------------------------------------- | -------------- | ------ | ------ |
| POST /tokens             | POST    | Authentifier l'utilisateur et fournir tokens       | Vérifier l'authentification et la remise de tokens | 200 OK         | OK     | Élevé  |
| POST /tokens             | POST    | Authentifier l'utilisateur et fournir tokens       | Gérer l'authentification avec 2FA                  | 202 Accepted   | OK     | Élevé  |
| PUT /tokens              | PUT     | Rafraîchir le token d'accès                        | Vérifier la mise à jour du token d'accès           | 200 OK         | OK     | Élevé  |
| DELETE /tokens           | DELETE  | Révoquer un refresh token                          | Vérifier la révocation du refresh token            | 204 No Content | X      | Élevé  |
| POST /users              | POST    | Créer un compte utilisateur                        | Vérifier la création de compte                     | 204 No Content | OK     | Élevé  |
| GET /users/{id}          | GET     | Obtenir les données d'un utilisateur               | Vérifier la récupération des données utilisateur   | 200 OK         | OK     | Élevé  |
| DELETE /users/{id}       | DELETE  | Supprimer le profil d'un utilisateur               | Vérifier la suppression du profil utilisateur      | 204 No Content | OK     | Élevé  |
| POST /users/{id}/keys    | POST    | Stocker un lot de pré-clés                         | Vérifier le stockage des pré-clés                  | 204 No Content | OK     | Moyen  |
| GET /users/{id}/keys     | GET     | Récupérer les pré-clés d'un utilisateur            | Vérifier la récupération des pré-clés              | 200 OK         | OK     | Moyen  |
| PATCH /users/{id}/keys   | PATCH   | Mettre à jour la pré-clé signée d'un utilisateur   | Vérifier la mise à jour de la pré-clé signée       | 204 No Content | OK     | Moyen  |
| GET /users/{id}/messages | GET     | Récupérer les messages en attente d'un utilisateur | Vérifier la récupération des messages en attente   | 200 OK         | OK     | Moyen  |
| GET /users/{id}/messages | GET     | Récupérer les messages en attente d'un utilisateur | Vérifier la suppression après récupération         | 200 OK         | OK     | Moyen  |

#### Notes supplémentaires

- Le statut "X" indique que le cas de test doit encore être exécuté ou que le résultat est en attente de vérification.
- Ces tests doivent être effectués dans un environnement de test avant le déploiement en production.

### Tests de Sécurité et de Performance

- Des tests de charge et de sécurité devraient également être effectués en utilisant des outils comme OWASP ZAP pour identifier toute vulnérabilité potentielle.

### Automatisation des tests

- Envisager l'automatisation de ces tests à l'aide de Postman ou d'autres frameworks d'automatisation pour intégrer dans un pipeline CI/CD.

## Serveur WebSocket

### Introduction

Ce plan détaille les tests fonctionnels prévus pour valider les fonctionnalités du serveur WebSocket de Missive, qui gère l'envoi et la réception des messages en temps réel ainsi que la gestion des statuts des messages.

Les tests unitaires du serveur de Missive sont réalisés avec Vitest, un framework de test moderne et rapide, simple à utiliser et extrêmement bien documenté et maintenu. Il est développé par Evan You, le créateur de Vue.JS. Il permet également de réaliser des mocks de manière simple, ce qui a été crucial étant donné que le serveur de Missive dépend d'un bon nombre de composants externes, comme Prisma pour la base de données.

Ils sont organisés en ajoutant .test.ts après le nom du fichier, ce qui permet d'avoir les tests directement à côté du fichier concerné dans l'arborescence.

Ils également mis en place de manière à générer un rapport JUnit, ce qui permet d'avoir une vue dans la pipeline Gitlab des différents tests.

### URL

**WebSocket Server** : `wss://missive.nezia.dev`

### Scénarios de Test

| Description                                                 | Objectif                                             | Code Attendu            | Statut | Impact |
| ----------------------------------------------------------- | ---------------------------------------------------- | ----------------------- | ------ | ------ |
| Établir une connexion WebSocket                             | Vérifier l'établissement de la connexion             | 101 Switching Protocols | OK     | Élevé  |
| Envoyer un message à un utilisateur spécifique              | Vérifier l'envoi du message                          | 200 OK                  | OK     | Élevé  |
| Recevoir un message de l'utilisateur                        | Vérifier la réception du message                     | 200 OK                  | OK     | Élevé  |
| Stocker le message si le récepteur est hors ligne           | Vérifier le stockage temporaire des messages         | 200 OK                  | OK     | Élevé  |
| Envoyer le message directement si le récepteur est en ligne | Vérifier l'envoi en temps réel                       | 200 OK                  | OK     | Élevé  |
| Mettre à jour le statut du message à 'envoyé'               | Vérifier la mise à jour du statut après l'envoi      | 200 OK                  | OK     | Moyen  |
| Mettre à jour le statut du message à 'reçu'                 | Vérifier la mise à jour du statut après la réception | 200 OK                  | OK     | Moyen  |
| Mettre à jour le statut du message à 'lu'                   | Vérifier la mise à jour du statut après lecture      | 200 OK                  | OK     | Moyen  |


#### Historique

- 2024-03-29 : Les routes `POST /tokens`, `PUT /tokens` sont opérationnelles et validées.
- 2024-05-02 : Validation des routes `POST /users/{id}/keys`, `GET /users/{id}/keys`.
- 2024-05-16 : Amélioration de la gestion des WebSocket et validation des tests d'envoi et de réception de messages.
- 2024-06-03 : Refactoring et validation des tests de mise à jour des statuts des messages.
- 2024-06-05 : Documentation du code avec docstrings.
- 2024-06-08 : Implémentation de la gestion des WebSocket 
- 2024-06-13 : Implémentation des tests de mise à jour des statuts des messages à "lu".
- 2024-06-15 : Validation de la reconnexion automatique et stockage temporaire des messages.

### Plan de tests

## Client

### Introduction

Ce plan détaille les tests fonctionnels prévus pour valider les fonctionnalités du client Missive, développé en Flutter, qui gère l'authentification des utilisateurs, la communication sécurisée via le protocole Signal, et l'envoi/réception de messages en temps réel.

Les tests unitaires du client de Missive sont réalisés avec flutter_test, une librairie native à Flutter. Ils utilisent aussi mockito afin de pouvoir créer les différents mocks, qui dans le cas du client est crucial (le client dépend d'un bon nombre d'éléments externes, l'exemple le plus évident étant l'API).

Ils sont organisés dans le répertoire test, ce qui est la convention pour les tests en Flutter. Un rapport JUnit est également généré afin d'avoir une visualisation claire directement dans la pipeline Gitlab.

### URL
**Production** : `wss://missive.nezia.dev`
### Scénarios de Test

| Fonctionnalité                                 | Description                                              | Objectif                                                       | Statut | Impact |
| ---------------------------------------------- | -------------------------------------------------------- | -------------------------------------------------------------- | ------ | ------ |
| Connexion utilisateur                          | Authentifier un utilisateur avec ses identifiants        | Vérifier l'authentification et la remise de tokens             | OK     | Élevé  |
| Création de compte utilisateur                 | Créer un nouveau compte utilisateur                      | Vérifier la création de compte et l'initialisation des clés    | OK     | Élevé  |
| Récupération des données utilisateur           | Obtenir les informations de l'utilisateur connecté       | Vérifier la récupération des données utilisateur               | OK     | Élevé  |
| Déconnexion utilisateur                        | Déconnecter l'utilisateur et révoquer les tokens         | Vérifier la déconnexion et la révocation des tokens            | OK     | Élevé  |
| Envoi de message                               | Envoyer un message chiffré à un autre utilisateur        | Vérifier l'envoi et le chiffrement des messages                | OK     | Élevé  |
| Réception de message                           | Recevoir un message chiffré d'un autre utilisateur       | Vérifier la réception et le déchiffrement des messages         | OK     | Élevé  |
| Gestion des clés publiques                     | Synchroniser et mettre à jour les clés publiques         | Vérifier la synchronisation et la mise à jour des clés         | OK     | Moyen  |
| Stockage et récupération des messages en local | Stocker et récupérer les messages en local               | Vérifier la persistance des messages en local                  | OK     | Moyen  |
| Reconnexion automatique                        | Reconnecter automatiquement en cas de perte de connexion | Vérifier la reconnexion automatique et la reprise des messages | OK     | Moyen  |
| Gestion des notifications                      | Recevoir des notifications push en arrière-plan          | Vérifier la réception des notifications                        | OK     | Moyen  |
| Statut des messages                            | Mettre à jour le statut des messages (envoyé, reçu, lu)  | Vérifier la mise à jour des statuts des messages               | OK     | Moyen  |
| Interface utilisateur                          | Afficher les messages et interactions utilisateur        | Vérifier la bonne présentation des messages et des statuts     | OK     | Moyen  |

#### Historique

- 2024-03-27 : Mise en place de l'environnement de développement et configuration initiale.
- 2024-05-17 : Validation de l'interface de connexion et des conversations.
- 2024-05-18 : Implémentation du stockage sécurisé des clés publiques.
- 2024-05-21 : Révision du stockage des clés publiques et validation de la création de sessions chiffrées.
- 2024-05-22 : Validation de la connexion WebSocket et de l'envoi/réception de messages chiffrés.
- 2024-05-23 : Gestion de la persistance des messages locaux avec Hive.
- 2024-05-25 : Validation de l'envoi et de la réception de messages avec mise à jour des statuts.
- 2024-05-26 : Amélioration de l'interface utilisateur.
- 2024-06-01 : Validation de l'écran de conversations.
- 2024-06-02 : Implémentation de la recherche d'utilisateurs.
- 2024-06-03 : Validation de la récupération et du stockage des messages temporaires.
- 2024-06-07 : Validation des notifications push avec Firebase.
- 2024-06-13 : Validation des tests de mise à jour des statuts des messages à "lu".
- 2024-06-15 : Validation de la reconnexion automatique et du stockage temporaire des messages.
- 2024-06-16 : Documentation et plan de tests.
- 2024-06-17 : Validation des notifications sur iOS et Android.
- 2024-06-27 : Déploiement sur Jelastic Cloud.
