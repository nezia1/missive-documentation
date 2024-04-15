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
| POST /users              | POST    | Créer un compte utilisateur                        | Vérifier la création de compte                     | 204 No Content | X      | Élevé  |
| GET /users/{id}          | GET     | Obtenir les données d'un utilisateur               | Vérifier la récupération des données utilisateur   | 200 OK         | X      | Élevé  |
| DELETE /users/{id}       | DELETE  | Supprimer le profil d'un utilisateur               | Vérifier la suppression du profil utilisateur      | 204 No Content | X      | Élevé  |
| POST /users/{id}/keys    | POST    | Stocker un lot de pré-clés                         | Vérifier le stockage des pré-clés                  | 204 No Content | X      | Moyen  |
| GET /users/{id}/keys     | GET     | Récupérer les pré-clés d'un utilisateur            | Vérifier la récupération des pré-clés              | 200 OK         | X      | Moyen  |
| PATCH /users/{id}/keys   | PATCH   | Mettre à jour la pré-clé signée d'un utilisateur   | Vérifier la mise à jour de la pré-clé signée       | 204 No Content | X      | Moyen  |
| GET /users/{id}/messages | GET     | Récupérer les messages en attente d'un utilisateur | Vérifier la récupération des messages en attente   | 200 OK         | X      | Moyen  |

#### Notes supplémentaires

- Le statut "X" indique que le cas de test doit encore être exécuté ou que le résultat est en attente de vérification.
- Ces tests doivent être effectués dans un environnement de test avant le déploiement en production.

### Tests de Sécurité et de Performance

- Des tests de charge et de sécurité devraient également être effectués en utilisant des outils comme OWASP ZAP pour identifier toute vulnérabilité potentielle.

### Automatisation des tests

- Envisager l'automatisation de ces tests à l'aide de Postman ou d'autres frameworks d'automatisation pour intégrer dans un pipeline CI/CD.
