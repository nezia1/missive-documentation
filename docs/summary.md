---
hide: 
    - navigation
    - toc
---
## Bilan personnel

### Introduction
Mon travail de diplôme sur la création de Missive a été une expérience de développement intense et enrichissante. J'ai abordé des aspects variés du développement logiciel, depuis la conception initiale de l'architecture de l'API et de la base de données jusqu'au déploiement final sur le cloud Infomaniak. Voici une rétrospective détaillée de ce que j'ai réalisé, des défis rencontrés et des apprentissages tirés de ce projet.

### Réalisations clés
1. Conception et architecture
    - Conception et mise en place d'une API robuste en utilisant la spécification OpenAPI 3.0.
    - Conception d'une base de données relationnelle performante, modélisée avec Prisma. 

2. Implémentation technique
    - Développement et test des routes critiques pour la gestion des utilisateurs, des clés publiques, et des messages.
    - Implémentation du protocole Signal pour le chiffrement de bout-en-bout des messages, avec gestion du stockage sécurisé.
    - Intégration et gestion des communications en temps réel via WebSocket.

3. Déploiement et DevOps
    - Création d'environnements de développement et de production robustes en utilisant Docker et Docker Swarm.
    - Migration et déploiement de l'application sur Jelastic Cloud, garantissant la disponibilité et la résilience.

4. Interface utilisateur
    - Développement de maquettes intuitives et de l'interface utilisateur en utilisant Figma et Flutter, avec une attention particulière à l'expérience utilisateur.
    - Implémentation des notifications push pour Android et iOS via Firebase Cloud Messaging.

5. Tests et documentation
    - Création de tests unitaires et d'intégration pour assurer la fiabilité du code.
    - Maintien d'une documentation détaillée et accessible pour tous les aspects du projet.

### Défis rencontrés
1. Protocole Signal
    - La complexité de l'implémentation du protocole Signal et la gestion des clés publiques ont nécessité une recherche approfondie et une rigueur dans la mise en œuvre.
    - Le manque de documentation sur l'implémentation en Dart a rendu la compréhension de la bibliothèque plus complexe, ce qui a nécéssité plus de temps passé dessus. L'avantage principal étant que cela m'a permis de comprendre beaucoup plus en profondeurs comment le tout fonctionnait ensemble.

2. Gestion des connexions
    - La gestion des connexions WebSocket a prouvé être un défi intéressant, notamment au niveau de la gestion de son cycle de vie au niveau du client. Ce problème a été réglé rapidement, en me penchant d'avantage sur le cycle de vie des providers et des composants avec Flutter.

4. Gestion des notifications
    - La gestion des notifications push sur différentes plateformes a été particulièrement délicate, notamment en raison des restrictions spécifiques à iOS. Ce défi a été surmonté en prenant le temps de bien créer les bons certificats et clés, ainsi qu'en passant du temps sur XCode afin de comprendre les différentes parties.

5. Déploiement en production
    - L'intégration et le déploiement de l'application sur l'environnement Infomaniak avec Docker Swarm a posé des défis en termes de configuration et de gestion des secrets. Cela m'a poussé à réadapter mes fichiers Docker, ainsi que de passer sur une gestion des secrets différentes.

6. Tests unitaires 
    - La création de tests unitaires et la gestion des mocks pour Prisma et les WebSockets ont demandé une compréhension approfondie des outils de test. J'ai rencontré pas mal de difficultés avec les mocks, en particulier avec Typescript qui m'a poussé à changer plusieurs fois de librairies de test, mais j'ai finalement fini par trouvé une solution solide et maintenable.

### Ce que le projet m'a appris
1. Compétences Techniques
    - J'ai amélioré mes compétences en conception et en architecture d'API et de bases de données.
    - Une compréhension beaucoup plus en profondeur des protocoles de chiffrement, en particulier le protocole Signal, et comment les implémenter dans une application.
    - Des compétences renforcées en déploiement et gestion d'environnements de développement et de production avec Docker et Docker Swarm.
    - Apprentissage du Dart avec Flutter, que je voulais découvrir et approfondir depuis un bon moment.

2. Gestion de projet
    - Utilisation efficace d'outils comme Trello pour la gestion des tâches et la planification du projet.
    - Importance d'une documentation claire et complète pour la pérennité et la maintenabilité du projet, pour moi-même ainsi que pour les autres.

3. Résolution de Problèmes
    - Capacité à s'adapter et à trouver des solutions alternatives face aux défis techniques.
    - Apprentissage de la persévérance et de la patience dans la résolution de problèmes complexes.

### Conclusion
Ce projet a été une expérience inestimable qui m'a permis de développer une gamme complète de compétences en développement logiciel, de la conception à la mise en production. Les défis rencontrés ont renforcé ma capacité à résoudre des problèmes complexes et à m'adapter avec des contraintes temporelles. Je suis fier de la robustesse et de la sécurité de l'application Missive, ainsi que de la qualité de la documentation et des tests. Je suis convaincu que les compétences acquises lors de ce projet seront extrêmement bénéfiques pour mes futurs projets professionnels.

Au-delà des compétences techniques, ce projet m'a également permis de développer une certaine éthique et méthodologie de travail. J'ai appris à gérer mon temps de manière efficace, à prioriser les tâches les plus critiques, et à maintenir un niveau de qualité élevé tout au long du développement. Cette expérience m'a aussi montré l'importance de la collaboration et de la communication, des qualités essentielles pour réussir dans n'importe quel projet technologique.
