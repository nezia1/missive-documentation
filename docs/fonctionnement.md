# Fonctionnement

Le fonctionnement de Missive repose sur l'efficacité du protocole Signal, qui est décrit [sur cette page](https://signal.org/docs/). Vous trouverez ci-dessous un schéma d'utilisation classique, de la création du compte utilisateur à l'envoi et la réception d'un message.

## Architecture

L'application comporte trois parties distinctes : le client, qui est l'application mobile réalisée en Flutter, l'API, qui est une API REST en TypeScript, ainsi qu'un serveur de WebSocket, qui est lui aussi en TypeScript. Le client peut communiquer avec l'API, pour la partie autorisation (gestion de la connexion à l'application, de l'authentification en 2 étapes...), ainsi que la réception des messages depuis le serveur si l'on était hors-ligne, et avec le serveur de WebSocket, pour la partie communication en temps réel. Vous trouverez ci-dessous un schéma de l'architecture de l'application.
<figure markdown>
![Schéma de l'architecture de l'application](assets/diagrams/Missive.svg)
<figcaption>Schéma de l'architecture de l'application</figcaption>
</figure>

Le fonctionnement des différentes parties est détaillé plus bas.

### Client

Le client est l'application mobile, réalisée en Flutter. Il est le point d'entrée de l'utilisateur·rice, et permet de communiquer avec l'API et le serveur de WebSocket. Il permet aussi de gérer le chiffrement, le déchiffrement, et la signature des messages. Flutter a été retenu pour sa rapidité de développement, son expérience développeur, sa facilité de déploiement, et sa capacité à gérer les différentes plateformes (iOS, Android, Web, Desktop). Il permet également de gérer les mises à jour de manière efficace, et de gérer les différentes versions de l'application.

### API

L'API est une API REST en TypeScript, qui permet de gérer l'authentification, le stockage des différentes clés publiques, ainsi que les utilisateurs. Le framework utilisé pour cette dernière sera Fastify, qui est un framework back-end rapide, efficace, qui permet de gérer les routes de manière efficace. Beaucoup de fonctionnalités sont déjà implémentées, comme la gestion des erreurs, la gestion des paramètres, etc. Cela me permettra de me concentrer au maximum sur le développement de l'application et de ses fonctionnalités.

La base de données est gérée par Prisma, qui est un ORM (Object-Relational Mapping) permettant de gérer les différentes tables de manière efficace, et de gérer les relations entre les différentes tables. Il permet également de gérer les migrations de manière efficace, et de gérer les différentes versions de la base de données (ce qui sera crucial durant le processus de développement).

#### Authentification

Le système d'authentification de l'API repose sur un système de jetons JWT. Ces derniers permettent une gestion de l'authentification complètement découplée du serveur (ce qui est important pour une API REST, qui doit rester *stateless*).

Après la connexion à l'application (route `POST /users`), l'utilisateur·rice reçoit :

- Jeton d'accès : permet de s'authentifier sur les différentes routes de l'API. Il est valide pendant 15 minutes, et contient les différentes permissions de l'utilisateur·rice. Il est envoyé dans le corps de la réponse, et doit être stocké localement de manière sécurisée.
- Jeton de rafraîchissement : permet de rafraîchir le jeton d'accès. Il est valide pendant 30 jours, et permet de générer un nouveau jeton d'accès sans avoir à se reconnecter. Il permet également de révoquer la connexion en cas de perte ou de vol du compte. Il est envoyé dans un cookie sécurisé HTTPOnly, et doit être stocké localement de manière sécurisée.

Vous pouvez voir [sur ce lien](https://jwt.io/#debugger-io?token=eyJhbGciOiJSUzI1NiJ9.eyJzY29wZSI6WyJwcm9maWxlOnJlYWQiLCJwcm9maWxlOndyaXRlIiwia2V5czpyZWFkIiwibWVzc2FnZXM6cmVhZCJdLCJpYXQiOjE3MTI2NjA5NzEsInN1YiI6ImFlZDA2MGFjLTc1ZDUtNDRkZi1hODJlLTc2MGIzZDFkNjYzNiIsImV4cCI6MTcxMjY2MTg3MX0.heFr_h9vINm4qUAI55alSFL52CVBz26IbaxyEmqH8rz-m7vHhV1ccolj_OogP4lMYeyfE7jINioLEUR6JkBePPkTMoU2XYXg1rc2yaXSYfti8zyQYEPF8PzN-Njz4GbszzrriKqNxsV8JjKLdTWEcYRvDXmBGLNJJar3bZWyNWwrc6gpfOfa0tX5CN66Riu5tK2kirEmY1xaPpNtEukyUXUQANvY75RYmJUBSyOGuHrp0uGdsNT5GQ7JVgQtWuVw34fHVi72psfHsRTsh_KOhpbXkUndWDtogwKR6Pyli8acwlacEGuuCbz5pLQ9G-nXuTqPEgkIvi2h7M0g9PHRsg) à quoi ressemble le contenu d'un jeton d'accès.

Le jeton de rafraîchissement contient uniquement l'identifiant unique de l'utilisateur, ainsi que sa date d'expiration.

Ces derniers étant signés cryptographiquement via une clé privée, il est impossible de les modifier sans la clé privée correspondante. Cela permet de garantir l'intégrité des jetons, et de garantir que l'utilisateur·rice est bien celui·celle qu'il·elle prétend être. Cela permet également d'avoir à gérer des sessions côté serveur, et laisse au client la responsabilité de gérer son propre état.

### Serveur WebSocket

Le serveur WebSocket est un serveur en TypeScript, qui permet de gérer la communication en temps réel entre les utilisateur·rice·s. Il permet de gérer l'envoi des messages, ainsi que leur stockage si nécéssaire. La bibliothèque utilisée pour ce dernier est un plugin Fastify, [@fastify/websocket](https://www.npmjs.com/package/@fastify/websocket), qui encapsule le protocole WebSocket et permet d'utiliser les fonctionnalités de Fastify.

Ce dernier utilise le même système d'authentification que l'API (jeton d'accès) à la création de la connexion. Il peut également accéder à la base de données, ce qui lui permet de stocker les messages temporairement dans le cas où l'utilisateur·rice n'est pas connecté·e.

Il permet d'envoyer des messages à un utilisateur•trice en utilisant son identifiant, ainsi que d'en recevoir et de gérer les statuts de lecture et de réception.

### Base de données

La base de données est une base de données PostgreSQL, qui permet de stocker les utilisateurs, les messages non envoyés et les clés publiques. PostgreSQL a été retenu pour sa robustesse, sa fiabilité, et sa capacité à gérer de gros volumes de données. Il permet également de gérer les transactions, les clés étrangères, et les index de manière efficace. Un diagramme de la base de données est disponible ci-dessous :
<figure markdown>
![Schéma de la base de données](assets/diagrams/database.svg)
<figcaption>Schéma de la base de données</figcaption>
</figure>

## Cas d'utilisation

### Premier lancement

Au premier lancement de l'application, l'utilisateur·rice devra créer un compte. Pour cela, il lui suffira de choisir un nom d'utilisateur·rice, et de saisir un mot de passe. L'application générera ensuite les différentes clés, et enverra également les clés publiques au serveur (requête HTTPS classique). Ces dernières sont cruciales pour le fonctionnement de l'application, car elles permettent la communication initiale des utilisateur·rice·s entre eux·lles.

La clé privée sera stockée localement sur l'appareil, protégée par les mécanismes de sécurité du système d'exploitation (ie. Keychain sur iOS, KeyStore sur Android, etc.).

<figure markdown>
![Diagramme de séquence de la connexion à l'application](assets/diagrams/out/login-sequence.svg){ width=800 loading=lazy }
<figcaption>Diagramme de séquence de la création d'un compte utilisateur</figcaption>
</figure>
