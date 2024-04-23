# Fonctionnement

Le fonctionnement de Missive repose sur l'efficacité du protocole Signal, qui est décrit [sur cette page](https://signal.org/docs/). Vous trouverez ci-dessous un schéma d'utilisation classique, de la création du compte utilisateur à l'envoi et la réception d'un message.

## Architecture

L'application comporte trois parties distinctes : le client, qui est l'application mobile réalisée en Flutter, l'API, qui est une API REST en TypeScript, ainsi qu'un serveur de WebSocket, qui est lui aussi en TypeScript. Le client peut communiquer avec l'API, pour la partie autorisation (gestion de la connexion à l'application, de l'authentification en 2 étapes...), ainsi que la réception des messages depuis le serveur si l'on était hors-ligne, et avec le serveur de WebSocket, pour la partie communication en temps réel. Vous trouverez ci-dessous un schéma de l'architecture de l'application.
<figure markdown>
![Schéma de l'architecture de l'application](assets/diagrams/Missive.svg)
<figcaption>Schéma de l'architecture de l'application</figcaption>
</figure>

1. Le client envoie un message à un•e utilisateur•trice via son application. Ce message est chiffré au niveau de l'application grâce au protocole Signal.
2. Le message est envoyé grâce à une connexion WebSocket au serveur.
3. Le serveur vérifie l'authentification et les permissions de l'utilisateur•trice grâce au jeton d'accès, puis regarde si la•le destinataire est connecté•e.

    - Si la•le destinataire est connecté•e, le message est envoyé directement à l'utilisateur•trice.
    - Si la•le destinataire n'est pas connecté•e, le message est stocké dans la base de données, et sera envoyé dès que la•le destinataire se connectera. Une notification est également envoyée.

4. L'utilisateur•trice reçoit le message, qui est déchiffré au niveau de l'application grâce au protocole Signal.
    1. Au démarrage de l'application, l'utilisateur•trice se connecte à l'API pour récupérer les messages en attente.

Le fonctionnement technique de ces différentes parties est détaillé ci-dessous.

### Client

Le client est l'application mobile, réalisée en Flutter. Il est le point d'entrée de l'utilisateur·rice, et permet de communiquer avec l'API et le serveur de WebSocket. Il permet aussi de gérer le chiffrement, le déchiffrement, et la signature des messages. Flutter a été retenu pour sa rapidité de développement, son expérience développeur, sa facilité de déploiement, et sa capacité à gérer les différentes plateformes (iOS, Android, Web, Desktop). Il permet également de gérer les mises à jour de manière efficace, et de gérer les différentes versions de l'application.

#### Concepts Flutter

Avant de commencer à parler du fonctionnement de l'application, il est important de comprendre certains concepts de Flutter, qui est le framework utilisé pour le développement de l'application. Ces concepts vont beaucoup revenir par la suite.

##### Provider

Une application front-end doit gérer un état global, qui permet de gérer les différentes parties de l'application. Par exemple, dans le cas de Missive, il faut un moyen de gérer les différentes clés, les messages, les utilisateurs, etc. Il faut également que l'application puisse réagir à certains de ces évènements (ie. réception d'un message, envoi d'un message, etc.).

Pour palier à ce problème, Flutter propose un système de gestion de l'état global, appelé Provider. Ce dernier permet de gérer l'état global de l'application, et de le partager entre les différentes parties de l'application. Un Provider est une classe, qui étend la classe `ChangeNotifier`, et qui permet de notifier les différentes parties de l'application lorsqu'un changement d'état a lieu.

Pour Missive, deux Provider sont disponibles :

- `AuthProvider` : permet de gérer l'authentification de l'utilisateur·rice, ainsi que son compte utilisateur.
- `SignalProvider` : propose une interface plus haut niveau de mon implémentation du protocole Signal, et permet de chiffrer, déchiffrer, et signer les messages. Il interagit également avec le SecureStorage pour stocker les clés de manière sécurisée (sérialisation / désérialisation).
- `ChatProvider` : permet de gérer la communication en temps réel avec le serveur de WebSocket.

<figure markdown="span">
    ![Mindmap du fonctionnement des Provider](assets/diagrams/out/provider-diagram.svg)
    <figcaption>Mindmap de l'architecture des Provider</figcaption>
</figure>

##### flutter_secure_storage

flutter_secure_storage est une bibliothèque Flutter qui permet de persister des données de manière sécurisée dans le stockage sécurisé du système d'exploitation (Keychain pour iOS, Keystore pour Android, libsecret pour les systèmes Linux). Elle est une partie extrêmement cruciale de Missive, car elle est utilisée dans les stores de mon implémentation du protocole Signal afin de récupérer les clés, sessions et autres données sensibles.

Les données sont stockées en base64 après avoir été sérialisées, car le stockage sécurisé ne permet que le stockage de chaînes de caractères. Il y a donc tout un travail de sérialisation/désérialisation à faire pour stocker des objets plus complexes, sur lequel nous allons revenir plus tard.

Il permet également de stocker les clés de plusieurs utilisateurs, car il préfixe chaque clé avec le nom de l'utilisateur. Cela permet de gérer plusieurs comptes sur la même application, et de les isoler les uns des autres afin d'éviter les problèmes et les conflits.

#### Arborescence

```sh
client/lib
├── common
│   └── http.dart
├── constants
│   └── api.dart
├── features
│   ├── authentication
│   │   ├── landing_screen.dart
│   │   ├── login_screen.dart
│   │   ├── models
│   │   │   └── user.dart
│   │   ├── providers
│   │   │   └── auth_provider.dart
│   │   ├── register_screen.dart
│   │   └── totp_modal.dart
│   ├── chat
│   │   └── providers
│   │       └── chat_provider.dart
│   ├── encryption
│   │   ├── providers
│   │   │   └── signal_provider.dart
│   │   ├── secure_storage_identity_key_store.dart
│   │   ├── secure_storage_pre_key_store.dart
│   │   ├── secure_storage_session_store.dart
│   │   └── secure_storage_signed_pre_key_store.dart
│   └── home
│       └── screens
│           ├── home_screen.dart
│           └── settings_screen.dart
└── main.dart
```

Comme vous pouvez le voir, l'application est divisée en plusieurs parties, en utilisant l'approche *feature-first* : chaque fonctionnalité a son propre dossier, et est organisée de la même manière, avec les écrans séparés des providers. Cela permet de voir rapidement les différentes parties de l'application, de séparer au maximum la logique de l'interface, et m'a beaucoup aidé lors du développement.

Ces différentes parties sont ensuites importées dans le fichier `main.dart`, qui est le point d'entrée de l'application. Il contient le routeur, importe tous les providers afin d'y avoir accès dans toute l'application, et redirige les utilisateurs par rapport à leur état de connexion.

### Serveur

Le serveur de Missive est composé de deux parties distinctes : l'API, et le serveur WebSocket. Ces deux parties permettent de gérer l'authentification, l'autorisation, le stockage des messages, et la communication en temps réel entre les utilisateur·rice·s. Ils sont réalisés à l'aide du framework Fastify, qui est un framework back-end rapide, et qui permet de gérer les différentes routes de manière efficace. Il permet également de gérer les différentes parties de l'application de manière découplée, ce qui est crucial dans le développement de cette application.

#### Concepts Fastify

Avant de commencer à détailler le fonctionnement du serveur, il est important de comprendre les concepts de base de Fastify, qui est le framework utilisé pour l'API et le serveur de WebSocket, car la nomenclature sera utilisée dans la suite de la documentation.

##### Hooks

Les hooks Fastify sont des fonctions qui sont exécutées avant ou après une route. Ils permettent de gérer des opérations comme l'authentification, la validation des données, la vérification des permissions, etc. Ils sont très utiles pour gérer les différentes parties de l'application, et permettent de découpler les différentes parties de l'application. Ce sont l'équivalent des middlewares dans d'autres frameworks.

#### API

L'API est une API REST en TypeScript, qui permet de gérer l'authentification, le stockage des différentes clés publiques, ainsi que les utilisateurs. Le framework utilisé pour cette dernière sera Fastify, qui est un framework back-end rapide, qui permet de gérer les routes de manière efficace. Beaucoup de fonctionnalités sont déjà implémentées, comme la gestion des erreurs, la gestion des paramètres, etc. Cela me permettra de me concentrer au maximum sur le développement de l'application et de ses fonctionnalités.

La base de données est gérée par Prisma, qui est un ORM (Object-Relational Mapper) permettant de gérer les différentes tables de manière efficace, et de gérer les relations entre les différentes tables. Il permet également de gérer les migrations de manière efficace, et de gérer les différentes versions de la base de données (ce qui est crucial durant le processus de développement).

##### Authentification

Le système d'authentification de l'API repose sur un système de jetons JWT. Ces derniers permettent une gestion de l'authentification complètement découplée du serveur (ce qui est important pour une API REST, qui doit rester *stateless*).

Après la connexion à l'application (route `POST /users`), l'utilisateur·rice reçoit :

- Jeton d'accès : permet de s'authentifier sur les différentes routes de l'API. Il est valide pendant 15 minutes, et contient les différentes permissions de l'utilisateur·rice. Il est envoyé dans le corps de la réponse, et doit être stocké localement de manière sécurisée.
- Jeton de rafraîchissement : permet de rafraîchir le jeton d'accès. Il est valide pendant 30 jours, et permet de générer un nouveau jeton d'accès sans avoir à se reconnecter. Il permet également de révoquer la connexion en cas de perte ou de vol du compte. Il est envoyé dans un cookie sécurisé HTTPOnly, et doit être stocké localement de manière sécurisée.

Le contenu d'un jeton d'accès est le suivant :

```json
{
  "scope": [
    "profile:read",
    "profile:write",
    "keys:read",
    "messages:read"
  ],
  "iat": 1712660971,
  "sub": "aed060ac-75d5-44df-a82e-760b3d1d6636",
  "exp": 1712661871
}
```

- `scope` : les différentes permissions de l'utilisateur·rice
- `iat` : l'heure à laquelle le jeton a été émis (issued at)
- `sub` : l'identifiant unique de l'utilisateur·rice
- `exp` : l'heure à laquelle le jeton expire

Le jeton de rafraîchissement contient uniquement l'identifiant unique de l'utilisateur, ainsi que sa date d'expiration.

Ces derniers étant signés cryptographiquement via une clé privée, il est impossible de les modifier sans la clé privée correspondante. Cela permet de garantir l'intégrité des jetons, et de garantir que l'utilisateur·rice est bien celui·celle qu'il·elle prétend être. Cela permet également d'avoir à gérer des sessions côté serveur, et laisse au client la responsabilité de gérer son propre état.

<figure markdown="span">
    ![Diagramme de séquence du processus d'authentification](assets/diagrams/out/authentication.svg)
    <figcaption>Diagramme de séquence du processus d'authentification</figcaption>
</figure>

Le processus d'authentification est géré par le hook `authenticationHook`.

##### Autorisation

Le processus d'autorisation de Missive repose sur un système de permissions. Chaque utilisateur·rice possède un ensemble de permissions, qui lui permettent d'accéder à certaines routes de l'API. Ces permissions sont stockées dans le jeton d'accès, et sont vérifiées à chaque requête. Si l'utilisateur·rice n'a pas les permissions nécessaires pour accéder à une route, il·elle recevra une erreur 403 Forbidden.

Voici le champ en question, qui a déjà été mentionné dans la partie Authentification :

```json
{
    "scope": [
        "profile:read",
        "profile:write",
        "keys:read",
        "messages:read"
    ]
}
```

Le processus d'autorisation est géré par le hook `authorizationHook`, qui prend également en paramètre les permissions nécessaires pour accéder à une route.
<figure markdown="span">
    ![Diagramme de séquence du processus d'autorisation](assets/diagrams/out/authorization.svg)
    <figcaption>Diagramme de séquence du processus d'autorisation</figcaption>
</figure>

#### Serveur WebSocket

Le serveur WebSocket est un serveur en TypeScript, qui permet de gérer la communication en temps réel entre les utilisateur·rice·s. Il permet de gérer l'envoi des messages, ainsi que leur stockage si nécéssaire. La bibliothèque utilisée pour ce dernier est un plugin Fastify, [@fastify/websocket](https://www.npmjs.com/package/@fastify/websocket), qui encapsule le protocole WebSocket et permet d'utiliser les fonctionnalités de Fastify.

Ce dernier utilise le même système d'authentification que l'API (jeton d'accès) à la création de la connexion. Il peut également accéder à la base de données, ce qui lui permet de stocker les messages temporairement dans le cas où l'utilisateur·rice n'est pas connecté·e.

Il permet d'envoyer des messages à un utilisateur•trice en utilisant son identifiant, ainsi que d'en recevoir et de gérer les statuts de lecture et de réception.

##### Authentification / autorisation

Comme expliqué auparavant, le serveur de Websocket emploie le même procédé de vérification d'identité et de permissions que l'API. Les jetons étant signés avec une clé privée propre au serveur, la vérification de l'identité utilise la même clé publique que l'API. Ce dernier a également accès à la même base de données, car l'instance de Prisma est partagée entre les deux parties du serveur.

##### Création du WebSocket

Le WebSocket est créé en effectuant une requête HTTP classique à la route `/`, qui est ensuite transformée en WebSocket. Cette route est protégée par le hook `authenticationHook`, qui vérifie l'authentification de l'utilisateur·rice. Si l'utilisateur·rice est authentifié·e, le serveur accepte la connexion, et permet à l'utilisateur·rice de communiquer en temps réel avec les autres utilisateur·rice·s.

##### Envoi et réception de messages

Une fois la connexion établie, l'utilisateur·rice peut envoyer des messages à un autre utilisateur·rice en utilisant son identifiant unique. Le serveur vérifie que l'utilisateur·rice est bien connecté·e, et que l'utilisateur·rice destinataire est également connecté·e. Le corps du message qui doit être envoyé ressemble à ceci :

```json
{
    "receiverId": "5443f5ef-9b6d-438b-9c0f-e00298725d13",
    "content": "This is an encrypted test message"
}
```

Des informations sont ensuite ajoutées au message :

- L'identifiant de l'expéditeur
- L'heure d'envoi

Si l'utilisateur est connecté, le message est envoyé directement à l'utilisateur·rice destinataire, sans passer par la base de données. Un message de confirmation est ensuite à l'expéditeur pour lui indiquer que le message a bien été remis.

Si ce n'est pas le cas, le message est stocké dans la base de données, et sera envoyé à l'utilisateur·rice dès qu'il·elle se connectera. Après le stockage, un message de confirmation est ensuite envoyé à l'expéditeur pour lui indiquer que le message a bien été envoyé.

La sécurité des messages est garantie par le chiffrement de bout en bout, ainsi que WSS, grâce à TLS, qui permet d'avoir une double sécurité. Les messages sont également supprimés de la base de données une fois qu'ils ont été demandés par le destinataire.

<figure markdown>
![Diagramme du processus d'envoi d'un message](assets/diagrams/out/sending-message.svg)
<figcaption>Diagramme du processus d'envoi d'un message</figcaption>
</figure>

### Base de données

La base de données est une base de données PostgreSQL, qui permet de stocker les utilisateurs, les messages non envoyés et les clés publiques. PostgreSQL a été retenu pour sa robustesse, sa fiabilité, et sa capacité à gérer de gros volumes de données. Il permet également de gérer les transactions, les clés étrangères, et les index de manière efficace. Un diagramme de la base de données est disponible ci-dessous :

<figure markdown>
![Schéma de la base de données](assets/diagrams/database.svg){ width=800 loading=lazy }
<figcaption>Schéma de la base de données</figcaption>
</figure>
