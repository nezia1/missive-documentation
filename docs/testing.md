---
hide: navigation
---
# Plan de tests


## API

### Introduction

Ce plan détaille les tests fonctionnels prévus pour valider les fonctionnalités de l'API Missive, qui gère les tokens d'accès et de rafraîchissement ainsi que les informations utilisateur.

### Lien de la production

L'API est hébergée sous ce lien : `https://missive.nezia.dev/api/v1`

### Authentification

Toutes les requêtes nécessitent une authentification par token Bearer, sauf la route `POST /tokens` et `PUT /tokens` qui est utilisée pour l'authentification.

### Scénarios de Test

| Endpoint                 | Méthode | Description                                        | Objectif                                           | Code Attendu   | Statut | Impact |
| ------------------------ | ------- | -------------------------------------------------- | -------------------------------------------------- | -------------- | ------ | ------ |
| POST /tokens             | POST    | Authentifier l'utilisateur et fournir tokens       | Vérifier l'authentification et la remise de tokens | 200 OK         | OK     | Élevé  |
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

### Rapport de tests
#### POST /tokens
##### L'utilisateur peut s'authentifier et reçoit des tokens valides

Une des grandes difficultés avec ce schéma d'authentification a été principalement de comprendre comment fonctionnaient les JWT. Je suis parti sur un schéma de token avec un `scope`, qui permet d'attribuer des permissions à l'utilisateur. J'ai utilisé la bibliothèque `jose` afin de générer ces différents jetons, ainsi que Prisma pour récupérer les informations de l'utilisateur et comparer le mot de passe haché avec `argon2`.

J'ai du étendre une des classes de `jose`, car j'avais besoin de rajouter un `scope` sur mes jetons, et vu que j'utilise Typescript, la gestion des types a été assez complexe. Après avoir approfondi, j'ai réussi à créer ma propre classe dérivée, et tout fonctionne correctement.

Ensuite, pour implémenter une des fonctionnalités les plus essentielles du JWT, la signature, j'ai tout d'abord opté pour une simple chaîne de caractères générée aléatoirement. Voulant encore plus renforcer ma sécurité, j'ai au final opté pour une paire de clés RSA, qui me permettront d'augmenter encore l'entropie et la complexité de la signature, rendant le risque de trouver cette fameuse clé privée encore plus minime, et également de standardiser encore plus la sécurité, une clé asymmétrique étant de manière générale plus standarde qu'une chaîne de caractères dans la cryptographie moderne.

Une fois cette signature implémentée, elle peut être vérifée au niveau du serveur, et une exception sera générée si il s'avère que le jeton a été modifié. Je pourrais donc refuser l'accès aux routes protégées en conséquence.

#### PUT /tokens
##### Mise à jour du token d'accès

En ce qui concerne la mise à jour du token d'accès, j'ai dû créer une table dans ma base de données, avec Prisma et rajouter une migration afin de pouvoir avoir une table qui stocke un token de rafraîchissement avec un ID utilisateur. De ce fait, je garde mon modèle d'API au maximum *stateless*, ce qui est un des points principaux d'une API REST. Ensuite, avec le jeton de rafraîchissement, je viens vérifier sa validité ainsi que son appartenance, et je génère un nouveau token pour l'utilisateur. Toute cette logique fonctionne parfaitement.

#### DELETE /tokens
##### Révocation du refresh token

Je n'ai pas eu le temps d'implémenter la révocation du refresh token. Il est par contre correctement supprimé du côté du client, ce qui ne poserait pas de problèmes, à moins que la base de données devienne vraiment grande.

#### POST /users
##### Création du compte utilisateur

Pour la création du compte utilisateur, j'ai récupéré les informations de l'utilisateur via le corps de la requête, et utilisé Prisma afin de créer un compte. J'ai également haché le mot de passe en utilisant argon2, afin de ne pas le stocker en clair.

#### GET /users/{id}
##### Obtenir les données d'un utilisateur

En ce qui concerne la récupération des données, je voulais faire en sorte que cette route retourne le minimum d'informations possibles : elle est utilisée afin d'effectuer une recherche des utilisateurs par tout le monde, il était donc important que le moins d'informations soient disponibles afin de respecter la confidentialité des utilisateurs.

J'ai donc dû développer une fonction qui me permet d'exclure des clés d'un type d'objet. Ensuite, après avoir récupéré les données avec Prisma, j'utilise cette fonction `exclude` afin d'envoyer le strict nécéssair au fonctionnement de l'application.

#### POST /users/{id}/keys
##### Stocker un bundle de pré-clés

Cette partie m'a posé un sacré défi : en effet, ayant réalisé mon serveur avant de faire l'application, je n'avais aucune idée d'à quoi allait ressembler un bundle de pré-clés de manière exacte. Ayant lu le *whitepaper* du protocole, j'ai essayé au mieux de créer une table qui me semblait correcte. Cependant, l'implémentation en Dart utilisait une structure assez différente, en rajoutant des champs auxquels je n'avais pas du tout pensé auparavant, comme le `registrationId`, ou même la gestion des différents périphériques. 

J'ai donc dû passer beaucoup de temps sur la modélisation de mes données, et y revenir un bon nombre de fois. Je pense avoir fini sur une itération qui est complètement fonctionnelle. J'ai également fait attention à stocker uniquement la partie publique de ces clés, en les sérialisant en base64 dans la base de données afin de simplifier au maximum le stockage.

#### GET /users/{id}/keys
##### Récupérer un bundle de pré-clés

La récupération des pré-clés avait une petite subtilité : il faut supprimer les *one time pre keys* une fois utilisées. J'ai donc implémenté une logique similaire à cette pour récupérer un utilisateur, sauf que cette fois, je n'ai pas exclu de champ, et j'ai simplement supprimé la pré-clé correspondante après l'avoir récupérée, en me servant de son ID.

#### PATCH /users/{id}/keys
##### Mise à jour de la pré-clé signée
Idéalement, une application implémentant le protocole Signal devrait se charger de pouvoir effectuer la rotation de la pré-clé signée, afin d'accroître encore plus la confidentialité. Missive implémente cette dernière, en utilisant encore une fois Prisma afin de pouvoir mettre à jour le bon champ. La route est fonctionnelle, malheureusement, il n'y a pas eu le temps de l'implémenter dans le client.

#### GET /users/{id}/messages
##### Récupération des messages en attente

Cette fonction est une partie extrêmement importante de l'application, car elle permet à chaque lancement de l'application de récupérer les messages envoyés quand le destinataire était hors-ligne (application fermée ou en arrière-plan). J'ai donc passé un bon moment dessus, afin de m'assurer qu'elle fonctionne, et surtout qu'elle supprime bien les messages après avoir accédé à la route. 

#### DELETE /users/{id}
##### Suppression du profil utilisateur
Le profil utilisateur peut être supprimé de manière sécurisée. 

## Serveur WebSocket

### Introduction

Ce plan détaille les tests fonctionnels prévus pour valider les fonctionnalités du serveur WebSocket de Missive, qui gère l'envoi et la réception des messages en temps réel ainsi que la gestion des statuts des messages.

Les tests unitaires du serveur de Missive sont réalisés avec Vitest, un framework de test moderne et rapide, simple à utiliser et extrêmement bien documenté et maintenu. Il est développé par Evan You, le créateur de Vue.JS. Il permet également de réaliser des mocks de manière simple, ce qui a été crucial étant donné que le serveur de Missive dépend d'un bon nombre de composants externes, comme Prisma pour la base de données.

Ils sont organisés en ajoutant .test.ts après le nom du fichier, ce qui permet d'avoir les tests directement à côté du fichier concerné dans l'arborescence.

Ils également mis en place de manière à générer un rapport JUnit, ce qui permet d'avoir une vue dans la pipeline Gitlab des différents tests.

### Lien de la production

Le serveur WebSocket est hébergé sous ce lien : `wss://missive.nezia.dev`

### Scénarios de Test

| Description                                                 | Objectif                                             | Statut | Impact |
| ----------------------------------------------------------- | ---------------------------------------------------- | ------ | ------ |
| Établir une connexion WebSocket                             | Vérifier l'établissement de la connexion             | OK     | Élevé  |
| Envoyer un message à un utilisateur spécifique              | Vérifier l'envoi du message                          | OK     | Élevé  |
| Recevoir un message de l'utilisateur                        | Vérifier la réception du message                     | OK     | Élevé  |
| Stocker le message si le récepteur est hors ligne           | Vérifier le stockage temporaire des messages         | OK     | Élevé  |
| Envoyer le message directement si le récepteur est en ligne | Vérifier l'envoi en temps réel                       | OK     | Élevé  |
| Mettre à jour le statut du message à 'envoyé'               | Vérifier la mise à jour du statut après l'envoi      | OK     | Moyen  |
| Mettre à jour le statut du message à 'reçu'                 | Vérifier la mise à jour du statut après la réception | OK     | Moyen  |
| Mettre à jour le statut du message à 'lu'                   | Vérifier la mise à jour du statut après lecture      | OK     | Moyen  |

#### Historique

- 2024-03-29 : Les routes `POST /tokens`, `PUT /tokens` sont opérationnelles et validées.
- 2024-05-02 : Validation des routes `POST /users/{id}/keys`, `GET /users/{id}/keys`.
- 2024-05-16 : Amélioration de la gestion des WebSocket et validation des tests d'envoi et de réception de messages.
- 2024-06-03 : Refactoring et validation des tests de mise à jour des statuts des messages.
- 2024-06-05 : Documentation du code avec docstrings.
- 2024-06-08 : Implémentation de la gestion des WebSocket 
- 2024-06-13 : Implémentation des tests de mise à jour des statuts des messages à "lu".
- 2024-06-15 : Validation de la reconnexion automatique et stockage temporaire des messages.

### Rapport de tests

##### Établissement d'une connexion WebSocket

L'établissement d'une connexion WebSocket depuis le serveur étant essentielle pour Missive, j'ai décidé de me concentrer sur cet élément dès les premiers stades de mon projet. J'ai pu trouver une bibliothèque, `@fastify/websocket`, qui m'a permis d'intégrer des routes WebSocket à mon serveur Fastify existant. Cela a été primordial pour plusieurs raisons : 

- Avoir tout le code au même endroit m'a grandement simplifié la tâche
- Avoir le WebSocket au même endroit que mon API m'a permis de réutiliser la même authentification (le hook `authenticationHook`), ce qui est très pratique car il fallait pouvoir réutiliser la même clé publique afin de vérifier la signature du JWT

Je me suis donc attaqué à la création d'une route WebSocket à la racine de mon projet. La documentation de `@fastify/websocket` étant très bonne, j'ai pu réussir à l'implémenter  très rapidement, et même pu réutiliser mon authentification avec la même syntaxe que pour mon API.

##### Envoi d'un message à un utilisateur / réception

Pour envoyer un message à un utilisateur, j'ai tout d'abord commencé par réfléchir à comment j'allais faire pour garder en mémoire les connexions WebSocket : pour envoyer un message à un utilisateur spécifique, il faut utiliser la méthode `send()` de sa connexion WebSocket. J'ai décidé de partir sur une liste de connexions stockée au dessus de mes routes, dans la mémoire du serveur, car ce ne sont pas des données persistantes, et ces connexions se déconnectant / reconnectant constamment, le choix de tout stocker dans une variable était je pense judicieux dans le cas de Missive.

J'ai ensuite réfléchi à une structure en JSON afin de pouvoir envoyer ce message : j'ai tout d'abord essayé d'envoyer le message via l'ID de l'utilisateur, ce qui fonctionnait au départ, mais malheureusement, je me suis rendu compte en réalisant le client que ce n'était pas une information à laquelle on pouvait avoir accès, la seule information connue par l'utilisateur étant le nom d'utilisateur. J'ai donc décidé d'utiliser ces derniers, qui sont uniques, et je stocke toutes les connexions WebSocket dans ma liste de connexions sous une clé, la clé étant leur nom.

A chaque nouveau message, je regarde à quel destinataire il appartient, je retrouve la connexion WebSocket correspondante, et j'envoie le message. Ça fonctionne !

##### Stockage du message si l'utilisateur est hors-ligne

Un des premiers défis que j'ai rencontré en programmant mon serveur WebSocket était le souci d'un des deux utilisateurs étant hors-ligne : si le destinataire est hors-ligne, comment j'allais pouvoir gérer tout ça ? C'était essentiel dans le cas de Missive, car l'utilisation d'une application de messagerie instantanée sur téléphone se fait généralement en grande majorité en dehors de l'application, l'application étant seulement ouverte pour répondre à un message ou à l'envoyer (elle est très souvent fermée).

Ayant implémenté une route pour récupérer les messages temporaires au niveau de mon API, je me suis dit que j'allais stocker le message dans la base de données, en utilisant le client Prisma défini dans l'API (encore un autre avantage du partage de code). Je suis passé par de multiples itérations de ma logique, afin de garantir une expérience responsive pour l'envoyeur et le destinataire. La logique repose principalement sur la vérification de l'existence d'une connexion WebSocket dans la liste des connexions. Si elle n'existe pas, le message sera stocké temporairement en base de données (ce qui n'est pas un problème de sécurité dans le cadre de Missive, les messages étant chiffrés de bout en bout, et supprimés à la récupération.

J'envoie un message de statut au destinataire après le stockage de son message, pour lui confirmer qu'il a bien été reçu (ce message de statut sera ensuite traité dans le client).

##### Mise à jour des statuts de messages

J'aimerais revenir plus en détails sur la mise à jour des statuts de messages, qui est une partie importante d'une application de messagerie. Cette dernière fonctionne exactement comme l'envoi des messages, avec la différence que la structure d'un message de statut est différente de celle d'un message classique (elle ne possède pas de contenu, avec une propriété `state` à la place). 

## Client

### Introduction

Ce plan détaille les tests fonctionnels prévus pour valider les fonctionnalités du client Missive, développé en Flutter, qui gère l'authentification des utilisateurs, la communication sécurisée via le protocole Signal, et l'envoi/réception de messages en temps réel.

Les tests unitaires du client de Missive sont réalisés avec flutter_test, une librairie native à Flutter. Ils utilisent aussi mockito afin de pouvoir créer les différents mocks, qui dans le cas du client est crucial (le client dépend d'un bon nombre d'éléments externes, l'exemple le plus évident étant l'API).

Ils sont organisés dans le répertoire test, ce qui est la convention pour les tests en Flutter. Un rapport JUnit est également généré afin d'avoir une visualisation claire directement dans la pipeline Gitlab.

### Scénarios de Test

| Fonctionnalité                                 | Description                                                           | Objectif                                                       | Statut | Impact |
| ---------------------------------------------- | --------------------------------------------------------------------- | -------------------------------------------------------------- | ------ | ------ |
| Connexion utilisateur                          | Authentifier un utilisateur avec ses identifiants                     | Vérifier l'authentification et la remise de tokens             | OK     | Élevé  |
| Création de compte utilisateur                 | Créer un nouveau compte utilisateur                                   | Vérifier la création de compte et l'initialisation des clés    | OK     | Élevé  |
| Récupération des données utilisateur           | Obtenir les informations de l'utilisateur connecté                    | Vérifier la récupération des données utilisateur               | OK     | Élevé  |
| Déconnexion utilisateur                        | Déconnecter l'utilisateur et révoquer les tokens                      | Vérifier la déconnexion et la révocation des tokens            | OK     | Élevé  |
| Stockage et récupération des messages en local | Stocker et récupérer les messages en local                            | Vérifier la persistance des messages en local                  | OK     | Élevé  |
| Envoi de message                               | Envoyer un message chiffré à un autre utilisateur                     | Vérifier l'envoi et le chiffrement des messages                | OK     | Élevé  |
| Implémentation du protocole Signal             | Implémenter le protocole Signal en Dart, avec libsignal_protocol_dart | Vérifier le bon fonctionnement du protocole de chiffrement     | OK     | Élevé  |
| Réception de message                           | Recevoir un message chiffré d'un autre utilisateur                    | Vérifier la réception et le déchiffrement des messages         | OK     | Élevé  |
| Reconnexion automatique                        | Reconnecter automatiquement en cas de perte de connexion              | Vérifier la reconnexion automatique et la reprise des messages | OK     | Moyen  |
| Gestion des notifications                      | Recevoir des notifications push en arrière-plan                       | Vérifier la réception des notifications                        | OK     | Moyen  |
| Statut des messages                            | Mettre à jour le statut des messages (envoyé, reçu, lu)               | Vérifier la mise à jour des statuts des messages               | OK     | Moyen  |

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
- 2024-06-17 : Validation des notifications sur iOS et Android.
- 2024-06-28 : Déploiement sur Jelastic Cloud.

### Rapport de tests
#### Création de compte

J'ai tout d'abord commencé par implémenter un système d'authentification avec le client. Ayant un système d'API fonctionnelle, j'ai tout d'abord commencé par créer un écran de démarrage, qui permet d'accéder à l'écran de création de compte, et à l'écran de connexion. Ensuite, j'ai implémenté mon `AuthProvider`, qui gère l'intégralité du statut de création de compte de l'utilisateur, en utilisant l'API, et gère également les jetons, en les stockant dans le stockage sécurisé du téléphone. Le `AuthProvider` gère également l'upload des clés publiques sur le serveur, afin que les autres utilisateurs puissent les trouver.

J'ai ensuite implémenté un routeur, avec `go_router`, qui me permet d'avoir des routes nommées, et d'également de permettre une redirection automatique avec la propriété `refreshListenable`, qui écoute les changements sur mon `AuthProvider`, et dès qu'il y en a, la fonction `redirect` est appelée, qui contient la logique de redirection par rapport au statut de connexion de l'utilisateur.

#### Authentifier un utilisateur

J'ai rajouté une fonction de connexion dans mon `AuthProvider`, qui va venir faire un appel à l'API, et vérifier les informations de l'utilisateur. Quand la connexion réussit, les jetons sont stockés comme pour la création de compte, et l'utilisateur est renvoyé sur la page d'accueil (la page des conversations).

#### Récupération des données utilisateur

J'ai implémenté un getter dans mon `AuthProvider`, qui permet de récupérer les informations utilisateur si elles ne sont pas disponibles. Cela permet notamment d'afficher le nom de l'utilisateur dans la barre à gauche.

#### Déconnexion utilisateur

La déconnexion de l'utilisateur a été implémentée en ajoutant une fonction `logout` dans mon `AuthProvider`, qui se charge de supprimer tous les jetons, remettre le statut `isLoggedIn` à `false`, et de supprimer toutes les données de connexion. La redirection au *landing screen* est ensuite effectuée automatiquement, grâce au routeur, qui écoute les changements sur `AuthProvider`.

#### Envoi des messages

Une fois tout ça implémenté, car j'avais besoin des jetons pour tester la connexion en temps réel, j'ai commencé à réfléchir à comment j'allais gérer les messages en temps réel. C'est une étape qui m'a pris du temps, car je n'avais jamais travaillé avec des WebSocket auparavant, je n'étais donc pas sûr de comment le faire. Je n'avais en plus jamais fait de Flutter de toute ma vie, donc j'ai dû me pencher sur la manière dont on stocke l'état global de l'application en Flutter de manière générale. L'architecture de l'application est d'ailleurs une des choses qui m'a pris le plus de temps, car les frameworks à composants non dogmatiques ont tendance à devenir extrêmement désorganisés si on ne réfléchit pas à son architecture.  

J'ai donc décidé de programmer toute la logique dans un Provider, `ChatProvider`, afin de pouvoir séparer la logique le plus possible de l'interface. Le système de widgets de Flutter pouvant devenir assez complexe à suivre, en raison de l'imbrication des différentes parties, j'ai donc implémenté pour commencer une fonction `connect()`, qui permet de connecter l'application au serveur WebSocket. Une fois que tout ça a été réalisé, j'ai implémenté une fonction `sendMessage`, qui permet d'envoyer un message en clair pour commencer à tester le WebSocket. Une fois que la bonne structure JSON a été implémentée, tout a fonctionné !

#### Réception des messages

Une fois les messages envoyés, il était temps de programmer la manière dont j'allais les afficher. J'ai décidé, de manière temporaire, de les afficher de manière désorganisée sur la page d'accueil afin de juste pouvoir tester si j'arrivais à avoir une application réactive. Vu que ma connexion WebSocket a été enveloppée dans un Provider, ces derniers donnent accès à une fonction, `notifyListeners`, qui permet de notifier l'application et les parties de l'interface qui ont besoin d'être rechargées. Cela veut dire qu'à chaque message, on peut utiliser `notifyListeners` afin de notifier l'interface que les messages ont besoin d'être mis à jour. Il m'a ensuite fallu utiliser un widget `ListView`, qui prend une liste quelconque, en l'occurence les messages, et permet d'afficher des widgets en itérant sur la liste automatiquement. Cela permet également d'ajouter des séparateurs, et de contrôler le comportement de visualisation, ce qui sera très utile par la suite.

#### Implémentation du protocole Signal

Maintenant que la connexion au WebSocket fonctionne, il est temps de s'attaquer au gros morceau du projet, à savoir implémenter le protocole Signal. Une des raisons principales pour lesquelles j'ai choisi Flutter pour mon projet, est l'existence d'une implémentation de ce protocole en Dart. Après avoir lu comment fonctionnait le protocole Signal, avec le *whitepaper* que la fondation a publiée sur son site, j'ai commencé à me pencher sur comment utiliser l'implémentation en Dart.

Pour faire simple, la librairie implémente toutes les fonctions mathématiques et la logique dont une application Signal a besoin. La manière dont la librairie fonctionne est très intéressante, car elle nous laisse libre arbitre sur la manière dont on souhaite implémenter le stockage des différentes clés privées et publiques. L'idée est que il est nécéssaire d'implémenter des classes, appelées des stores, qui vont gérer les interactions avec la manière de stocker de votre choix. Vous trouverez ci-dessous un exemple d'une de ces interfaces :
```dart
abstract mixin class PreKeyStore {
  Future<PreKeyRecord> loadPreKey(
      int preKeyId); //  throws InvalidKeyIdException;

  Future<void> storePreKey(int preKeyId, PreKeyRecord record);

  Future<bool> containsPreKey(int preKeyId);

  Future<void> removePreKey(int preKeyId);
}
```
Classe abstraite de PreKeyStore

Ces différentes méthodes doivent être implémentées de la manière dont l'on souhaite. Le seul souci de ce procédé était malheureusement le fait que la librairie était assez mal documentée, j'ai donc dû passer beaucoup de temps à déchiffrer les implémentations et à essayer de comprendre la logique derrière.

Une fois tout ça terminé, il suffit simplement d'utiliser les classes qu'ils ont implémenté, comme `SessionCipher`, qui prend toutes nos classes et utilise leurs fonctions internes, et le chiffrement fonctionne ! J'ai fait en sorte de tester ces classes unitairement de manière assez extensive, afin de m'assurer du bon fonctionnement de ces dernières. J'ai également à terme créé un provider, `SignalProvider`, qui me permet d'envelopper la logique un peu plus bas niveau de ces derniers dans des fonctions simples, comme `encrypt`, `buildSession` et `decrypt`. Cela permet de rendre la logique beaucoup plus lisible à terme, et beaucoup plus facilement maintenable.

#### Reconnexion automatique

La reconnexion automatique est extrêmement importante pour Missive, car étant une application téléphone, la qualité de la connexion de l'appareil peut changer constamment car l'utilisateur est en mouvement.

J'ai donc rajouté de la logique dans ma fonction `connect()`, qui me permet de gérer tout ça. Voici à quoi la logique ressemble :
```dart

      Future<void> connect() async {
      ...
      // If reconnection attempts have been made, fetch pending messages as there could have been messages sent while disconnected, and reset the attempts
      try {
      if (_reconnectionAttempts > 0) {
        fetchPendingMessages();
        fetchMessageStatuses();
        _reconnectionAttempts = 0;
        notifyListeners();
      }

      _channel!.stream.listen(
        (message) async {
          await _handleMessage(message);
        },
        onDone: _handleConnectionClosed,
        onError: (error) =>
            _logger.log(Level.SEVERE, 'WebSocket Error: $error'),
      );
    } catch (e) {
      _logger.log(Level.SEVERE, 'Failed to connect: $e');
      _connected = false;
      _scheduleReconnection();
    } finally {
      _isConnecting = false;
    }
    ...
  }

  void _handleConnectionClosed() {
    _logger.log(Level.WARNING, 'WebSocket connection closed.');
    _channel = null;
    _scheduleReconnection();
  }

  void _scheduleReconnection() {
    if (_disposed) return;
    if (_reconnectionTimer != null && _reconnectionTimer!.isActive) return;

    _reconnectionAttempts++;
    final delay = min(_reconnectionAttempts * 2,
        30); // take more and more time to reconnect after each failed attempt (max 30s)
    _reconnectionTimer = Timer(Duration(seconds: delay), () {
      _logger.log(Level.INFO,
          'Attempting to reconnect... (Attempt $_reconnectionAttempts)');
      connect();
    });
  }
```
Logique de reconnexion automatique

Une fois cette partie implémentée, il a fallu trouver un moyen de gérer le cas où un message est envoyée si la connection se ferme, afin d'éviter de perdre un message. 

Il a fallu créer une base de données locale Realm, `PendingMessages`, qui fonctionne de la même manière que le stockage des conversations. J'ai rajouté un bout de code dans `sendMessage` qui vérifie si la connexion WebSocket est établie, et si elle ne l'est pas, le message sera stocké avec `_storePendingMessage`.

Ensuite, une fois la reconnexion effectuée au WebSocket, une fonction est appelée, `_sendPendingMessages`, qui s'occupe d'envoyer l'intégralité des messages en attente. L'interface est également modifiée, au niveau de `conversation_screen.dart`, pour faire en sorte d'afficher un *spinner* de chargement si le message est en attente.

#### Gestion des notifications

Pour la gestion des notifications, comme dit précédemment, Firebase Cloud Messaging a été utilisé. Comme le serveur s'occupe d'envoyer les notifications, il a fallu également installer l'impléemntation de Firebase Cloud Messaging sur le client.

J'ai simplement suivi le tutoriel sur le site de Firebase, qui explique en détails comment relier son application à son projet Firebase.

Un des soucis principaux que j'ai rencontré était avec la mise en place des notifications iOS, qui a été plus complexe que la gestion sur Android. Il a fallu ajouter un bon nombre de profils et de clés dans ma console développeur Apple, ainsi que de les relier via XCode. 

Une fois toutes ces opérations effectuées, il a simplement fallu instancier Firebase Cloud Messaging, ainsi que de générer un jeton qui permet de relier le périphérique de l'utilisateur à la base de données des connexions. Voici à quoi ressemble le code : 

```dart
      if (Platform.isAndroid || Platform.isIOS) {
        FirebaseMessaging messaging = FirebaseMessaging.instance;
        notificationId = await messaging.getToken();
      } else {
        notificationId = null;
      }
```
Génération de l'ID de notification

Ce code est utilisé dans la fonction `login` et `register`, car il est nécéssaire d'éviter d'envoyer des notifications si l'utilisateur est déconnecté, ou de recevoir des notifications qui ne concernent pas la connexion actuelle.

#### Statut des messages

La gestion des statuts de messages étant prise en compte au niveau du serveur, il a fallu également l'implémenter côté client. Cette gestion est prise en charge dans ma fonction `_handleMessage` : si une propriété `state` est présente (mise à jour de statut), et le message stocké en local sera modifié.

Ensuite, une logique a été implémentée au niveau de `conversation_state.dart`, qui permet d'afficher les petites coches, ou le spinner si le message est en attente, afin d'informer l'utilisateur du statut de lecture / envoi de leur message.
