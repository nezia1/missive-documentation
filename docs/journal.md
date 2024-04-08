# Journal de bord

## 2024-03-27

Aujourd'hui, je commence mon travail de diplôme. Après un briefing avec M. Garcia¸ j'ai décidé de commencer à mettre en place l'environnement de développement de mon projet : j'aimerais m'assurer que je peux travailler sur mon projet depuis n'importe quel ordinateur, ainsi que de ne pas avoir de soucis au niveau des outils que j'utilise.

Je suis également en train de créer les différentes tâches, sur mon Trello, synchronisé aux *issues* Gitlab, qui permettront de suivre l'avancement de mon projet. J'essaie au maximum de découper mon projet en tâches logiques, ainsi de les ordonner de telle manière à ce que je puisse travailler de manière efficace. Je m'occupe également de faire un diagramme de Gantt grâce à un *power up* (extension) Trello.

Je vais commencer par architecturer mon API ainsi que ma base de données : j'utiliserais la spécification OpenAPI 3.0 pour décrire mon API, ce qui me fera gagner du temps car le format est bien documenté, et je pourrais générer de la documentation à partir de cette spécification grâce à Postman.

Pour la base de données, je vais tout d'abord créer un diagramme grâce à PlantUML, qui me permettra de visualiser les différentes tables et relations entre celles-ci. Ensuite, je vais implémenter tout ça grâce à un schéma Prisma, qui me permettra de générer du code TypeScript pour interagir avec ma base de données au niveau de l'API ainsi que de créer toutes les migrations.

En fin d'après-midi, j'ai commencé à travailler sur la spécification API, vu qu'il me restait un peu de temps. J'ai terminé de définir mes routes ``/tokens``, et commence à travailler sur ``/users``.

## 2024-03-28

Aujourd'hui, je continue de travailler sur le design de mon API. Je suis en train de terminer de définir les routes ``/users``, et je vais ensuite m'attaquer à la route ``/users/{id}/keys``, qui permettra la gestion des clés publiques au niveau du serveur.

Les routes ``/users/{id}`` étant terminées, je me renseigne un peu plus sur le protocole Signal. Je vais devoir implémenter un mécanisme qui permettra d'ajouter et retirer des clés publiques à un utilisateur de manière périodique (les pre keys sont à usage unique, et doivent être régénérées régulièrement quand le serveur n'en possède plus beaucoup). Il faut également idéalement que la clé publique signée change, de manière moins fréquente. Il faudra creuser ces détails pour voir comment je vais implémenter tout ça. Il sera important de stocker un timestamp de création pour toutes les clés, ainsi que de mise à jour pour la clé signée, pour pouvoir les régénérer au bon moment.

Je viens de réfléchir au mécanisme de mise à jour de mes clés : est-ce qu'il ne serait pas plus pratique de mettre à jour la clé publique non signée automatiquement après qu'elle ait été lue pour la première fois ? Cela permettrait de ne pas avoir à gérer de mécanisme de mise à jour côté client. Le faire du côté de l'API me semble plus simple. Il faudrait réfléchir à ça.

Au niveau des clés primaires de mes tables, je pense que des UUIDv4 seraient idéal pour les utilisateurs. Cela permettrait de pouvoir empêcher un potentiel attaquant de deviner les identifiants des ressources pouvant être accédées par identifiant, comme les utilisateurs.

Je viens de terminer le design de mon API. Je suis plutôt satisfait du résultat, et je pense que le modèle que j'ai décidé d'utiliser est plutôt robuste. Bien évidemment, des révisions seront possibles si jamais je me rends compte que quelque chose ne va pas. Les routes sont pour l'instant disponibles sur [ce lien](./api.md), mais à terme, elles seront disponibles directement sur une documentation Postman. Je m'attaque maintenant au design de la base de données, que je réalise avec PlantUML.

Pour la base de données, j'ai décidé de partir sur un schéma qui sépare les

 pre-keys signées des non signées : le nombre de clés signées étant bien inférieures au nombre de clés non signées (ces dernières étant générées en masse), cela permettra de rendre l'une des deux requêtes plus rapides.

J'ai mis à jour le schéma de base de données afin d'inclure ces nouvelles tables et relations.

Voici le résultat :
<figure markdown="span">
    ![Schéma initial de la base de données](./assets/diagrams/out/database.svg)
    <figcaption>Schéma initial de la base de données</figcaption>
</figure>

Je me suis aussi occupé de rajouter des exemples de tokens dans la spécification OpenAPI, afin de simplifier la compréhension globale du projet. Journée finalement très productive, je suis content de mon avancement et me sens prêt à attaquer la suite, qui sera l'implémentation de l'API. Cependant, avant de m'y mettre, je vais encore peaufiner la documentation, et m'assurer que tout soit bien clair et défini.

## 2024-03-29

Aujourd'hui, ayant terminé le design de mon API et de ma base de données, je m'occupe de copier ce que j'avais pour le POC, qui va me servir de base, et de l'adapter un peu pour coller à ce que j'ai défini dans mes spécifications. Je me charge maintenant de modifier la base de données afin de coller à ce que j'ai défini dans mon diagramme.

## 2024-05-02

Aujourd'hui, je m'attaque à la programmation des routes. Ayant déjà une route fonctionnelle pour les tokens, je vais juste m'assurer que tout soit bien en ordre, et je vais ensuite m'attaquer aux routes ``/users``.

Les routes ``/tokens`` sont maintenant en ordre. J'ai dû changer quelques détails, par rapport à la structure qui a été définie dans ma spécification. Je m'attaque maintenant aux routes ``/users``, plus précisément à ``/users/{id}/keys``. J'aimerais pouvoir mettre en place la gestion de clés publiques pour les utilisateurs, et je vais commencer par implémenter la route ``GET /users/{id}/keys``, qui permettra de récupérer les clés publiques d'un utilisateur.

J'ai réussi à implémenter la route ``GET /users/{id}/keys``, qui récupère la première clé publique non signée et la supprime après l'avoir récupérée, et la clé signée d'un utilisateur. J'ai rajouté un sous plugin Fastify dans mon plugin utilisateurs, qui me permet de séparer tout ça proprement. J'ai dû également rajouter une permission KEYS:READ, qui donne le droit à un utilisateur de lire les clés publiques d'un autre utilisateur.

Il va me falloir mettre à jour ma spécification, car je me suis rendu compte que la structure n'était pas tout à fait juste par rapport à mon implémentation. Je pense surtout à ma route ``GET /users/{id}/keys``, qui ne renvoie pas un tableau de clés, mais bien un objet contenant une clé signée et une clé non signée. Je vais également mettre à jour les exemples pour cette route.

J'ai malheureusement rencontré des problèmes lors de l'utilisation de ma pipeline Gitlab: en effet, vu que j'utilise un submodule, la pipeline n'est pas lancée automatiquement quand des changements de documentation sont effectués. J'ai dû donc lancer la pipeline manuellement, ce qui n'est pas idéal. Je vais devoir trouver une solution pour automatiser tout ça.

J'ai réussi à faire fonctionner la pipeline correctement ! Il suffisait juste d'utiliser le nom du submodule directement dans changes, au lieu du dossier (Gitlab a l'air de vérifier le pointeur vers le commit, ce qui simplifie grandement la tâche). J'ai également mis à jour la spécification API afin d'inclure le bon corps de réponse pour ``GET /users/{id}/keys`` (à savoir preKey et signedPreKey).

## 2024-05-03

Aujourd'hui, je décide de m'intéresser de plus près à l'implémentation en elle-même du protocole Signal. Je n'étais pas sûr de comment nommer certaines choses, je suis donc allé voir un exemple que les développeurs de la librairie que je vais utiliser pour Flutter ont réalisé en Typescript, qui est disponible [ici](https://signal-demo.privacyresearch.io/). Leur interface est extrêmement intuitive, et explique bien comment fonctionne le protocole, ainsi de comment l'implémenter au niveau du client. Voici ci-dessous une capture d'écran de leur interface :
<figure markdown="span">
  ![Capture d'écran de l'interface](./assets/img/signal-web-interface.png)
  <figcaption>Image caption</figcaption>
</figure>

Je met donc à jour ma spécification API, afin de coller à ce que j'ai vu dans l'exemple. La première chose que je vais faire est de renommer mon champ preKey en oneTimePreKey afin d'éviter les ambiguïtés.

## 2024-05-04

Aujourd'hui, je me suis occupé d'écrire la spécification pour la route `POST /users/{id}/messages`, qui permettra au serveur WebSocket de stocker les messages chiffrés pour un utilisateur, si son périphérique est déconnecté.

Je viens de me rendre compte qu'il sera nécéssaire de partager une clé entre l'API et le serveur WebSocket, afin de pouvoir vérifier les jetons JWT. Je vais donc passer sur un système de signature asymétrique, qui me permettra de signer les jetons JWT côté API avec une clé privée, et de les vérifier côté serveur WebSocket avec la clé publique. Je vais donc générer une paire de clés, et stocker la clé publique côté serveur WebSocket, et la clé privée côté API. Je dois également modifier le code afin de prendre en compte ce changement.

J'ai réussi à effectuer le changement. J'ai généré une paire de clés RSA256 avec OpenSSL, et j'ai stocké la clé privée et publique à la racine de `api`, bien évidemment ignorés avec .gitignore. J'ai rajouté la procédure pour générer ces dernières dans mon README. Maintenant, les JWT sont signés côté API grâce à la clé privée, ce qui améliore encore la sécurité de mon application, et il suffira juste d'utiliser la clé publique côté serveur WebSocket pour vérifier les jetons, ce qui évite de devoir partager une clé secrète entre les deux services. Je pense que ce changement est une bonne idée, car il permet de ne pas avoir deux potentiels points d'attaque, et de simplifier la gestion des clés.

Je devrais également regarder si il ne serait pas plus judicieux de faire la même chose pour le secret des cookies.

J'ai également créé une fonction dans le fichier utils afin de charger les clés publiques et privées, pour éviter de le faire sur chaque fichier. Je suis vraiment satisfait de ma progression aujourd'hui, et je m'attaquerais au serveur WebSocket demain.

Finalement, j'ai réglé un souci avec la version de Node, qui ne supportait pas le *globbing* afin de trouver les tests unitaires. J'ai donc dû mettre à jour la version de Node, et tout fonctionne correctement maintenant. J'ai également réglé quelques soucis dans les tests car je préfixe maintenant les routes avec `/v1` (afin de garder une version stable), ce qui n'était pas le cas avant.

## 2024-05-05

Aujourd'hui, je vais préparer le projet afin de pouvoir implémenter les WebSocket. Je me suis rendu compte que Fastify avait des fonctionnalités qui permettent d'avoir un serveur WebSocket sur la même application. Je vais donc restructurer le projet, afin de pouvoir bien séparer les différentes parties. Il serait je pense plus pratique d'avoir un dossier `api` et un dossier `websocket`, qui contiendront respectivement l'API et le serveur WebSocket.

Je vais maintenant implémenter les routes pour les messages, qui permettront de stocker les messages chiffrés pour un utilisateur, si son périphérique est déconnecté. Il faudra également implémenter une stratégie d'authentification différente, car les messages seront envoyés par le serveur WebSocket, et non par l'utilisateur. Comme défini dans ma spécification, une simple clé d'API au format UUID sera utilisée. Il me faut donc étendre mon plugin d'authentification pour supporter cette stratégie.

J'ai étendu mon plugin d'authentification afin de supporter différentes stratégies d'authentification. Je suis encore en train de réfléchir à comment stocker la clé d'API, que je pense simplement stocker dans une variable d'environnement (elle a seulement besoin d'être vérifiée).

Mes routes `GET users/{id}/messages` et `POST users/{id}/messages` fonctionnent désormais. J'ai dû effectuer des petits changements dans la base de données, notamment pour rajouter le contenu du message ainsi que changer le nom des champs afin qu'ils soient moins ambigüs. J'ai aussi reflété ces changements de base de données dans ma spécification OpenAPI. Il faudrait également donner un moyen au serveur de rajouter l'identifiant de l'envoyeur du message, ce qui n'est pas encore fait.

## 2024-05-08

Aujourd'hui, je m'attaque à la partie WebSocket. Fastify ayant son propre plugin WebSocket, j'ai décidé de l'utiliser pour plusieurs raisons pratiques:

- Il est intégré directement dans Fastify, ce qui me permet de ne pas avoir à gérer un autre serveur
- Il me permet de réutiliser ma stratégie d'authentification, ce qui me facilite énormément la tâche
- Il est bien documenté, ce qui me permet de ne pas perdre de temps à chercher comment l'utiliser

Je viens également de me rendre compte que si j'utilise le plugin WebSocket de Fastify, je n'aurais pas besoin d'avoir une route pour créer les messages temporaires: je pourrais simplement utiliser Prisma, mon ORM, afin de stocker les messages directement dans la base de données. Cela me permet également d'éviter d'utiliser une clé d'API, qui n'est plus nécéssaire. Le code devra donc être adapté afin de retirer cette dernière. J'aurais beaucoup aimé avoir trouvé ce plugin plus tôt, car il m'aurait évité de perdre du temps à implémenter des routes que je ne vais pas utiliser.

J'ai réussi à implémenter tout ça. J'ai été agréablement surpris de la facilité d'implémentation, ayant fait tout le code pour gérer les autorisations. C'est un plaisir de pouvoir tout réutiliser !

Pour l'instant, j'ai seulement implémenté une route exemple. Je m'occuperais de commencer à implémenter la logique de traitement des messages demain.

Il faudra également implémenter une validation au niveau du corps de la requête, pour s'assurer que les différentes requêtes envoyées par l'utilisateur•trice sont bien formées. Fastify intègre ça de manière assez élégante et native en Typescript, en utilisant des schémas JSON qui peuvent aussi servir de types. Voici [le lien vers la documentation](https://fastify.dev/docs/latest/Reference/TypeScript/) .

Apparemment, il est possible de générer un schéma JSON directement depuis Prisma, qui prend le schéma Prisma et créé un schéma JSON qui peut être utilisé pour valider les requêtes. Cela me semble être une bonne idée, car cela me permettra de ne pas avoir à réécrire les schémas de validation, et de m'assurer que les données envoyées par l'utilisateur•trice sont bien formées. Le souci étant que les requêtes, typiquement celle de connexion, n'est pas la même que le schéma de la base de données (il suffit de donner un nom et un mot de passe). Il faudra donc que je réfléchisse à comment je vais gérer ça. Voici [le lien vers la documentation](https://github.com/valentinpalkovic/prisma-json-schema-generator).

J'ai également refactor un peu mon application, et transformé mon client Prisma en plugin Fastify. Cela me permet de gérer l'instance de Prisma de manière beaucoup plus propre, ainsi que d'alléger les routes (mon client est maintenant importé une seule fois, et peut être utilisé partout dans l'application).

Finalement, j'ai aussi créé un Dockerfile de base, qui me permet de servir mon application facilement. Il y a encore quelques points de friction, comme un message `FATAL: database "prisma" does not exist". Il faudra que je m'en occupe plus tard. Également éviter de stocker le mot de passe de la base de données en clair dans le Dockerfile.
