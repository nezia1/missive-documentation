# Fonctionnement

Le fonctionnement de Missive repose sur l'efficacité du protocole Signal, qui est décrit [sur cette page](protocole-signal.md). Vous trouverez ci-dessous un schéma d'utilisation classique, de la création du compte utilisateur à l'envoi et la réception d'un message.

## Architecture

L'application comporte trois parties distinctes : le client, qui est l'application mobile réalisée en Flutter, l'API, qui est une API REST en TypeScript, ainsi qu'un serveur de WebSocket, qui est lui aussi en TypeScript. Le client peut communiquer avec l'API, pour la partie autorisation (gestion de la connexion à l'application, de l'authentification en 2 étapes...), ainsi que la réception des messages depuis le serveur si l'on était hors-ligne, et avec le serveur de WebSocket, pour la partie communication en temps réel. Vous trouverez ci-dessous un schéma de l'architecture de l'application.
<figure markdown>
![Schéma de l'architecture de l'application](assets/diagrams/architecture.svg)
<figcaption>Schéma de l'architecture de l'application</figcaption>
</figure>

## Premier lancement

Au premier lancement de l'application, l'utilisateur·rice devra créer un compte. Pour cela, il lui suffira de choisir un nom d'utilisateur·rice, et de saisir un mot de passe. L'application générera ensuite les différentes clés, et enverra également les clés publiques au serveur (requête HTTPS classique). Ces dernières sont cruciales pour le fonctionnement de l'application, car elles permettent la communication initiale des utilisateur·rice·s entre eux·lles.

La clé privée sera stockée localement sur l'appareil, protégée par les mécanismes de sécurité du système d'exploitation (ie. Keychain sur iOS, KeyStore sur Android, etc.).

<figure markdown>
![Diagramme de séquence de la connexion à l'application](assets/diagrams/out/login-sequence.svg){ width=800 loading=lazy }
<figcaption>Diagramme de séquence de la création d'un compte utilisateur</figcaption>
</figure>
