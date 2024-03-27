# Journal de bord

## 2024-03-27

Aujourd'hui, je commence mon travail de diplôme. Après un briefing avec M. Garcia¸ j'ai décidé de commencer à mettre en place l'environnement de développement de mon projet : j'aimereais m'assurer que je peux travailler sur mon projet depuis n'importe quel ordinateur, ainsi que de ne pas avoir de soucis au niveau des outils que j'utilise.

Je suis également en train de créer les différentes tâches, sur le Gitlab à l'aide des *issues*, qui permettront de suivre l'avancement de mon projet. J'essaie au maximum de découper mon projet en tâches logiques, ainsi de les ordonner de telle manière à ce que je puisse travailler de manière efficace.

Je vais commencer par architecturer mon API ainsi que ma base de données : j'utiliserais la spécification OpenAPI 3.0 pour décrire mon API, ce qui me fera gagner du temps car le format est bien documenté, et je pourrais générer de la documentation à partir de cette spécification grâce à Postman.

Pour la base de données, je vais tout d'abord créer un diagramme grâce à PlantUML, qui me permettra de visualiser les différentes tables et relations entre celles-ci. Ensuite, je vais implémenter tout ça grâce à un schéma Prisma, qui me permettra de générer du code TypeScript pour interagir avec ma base de données au niveau de l'API ainsi que de créer toutes les migrations.
