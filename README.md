# Console d'Affaire (CDA)

## 🧱 Description du projet

**CDA** est une application web modulaire et maintenable conçue pour la gestion d’affaires dans le domaine de la construction. Elle permet de centraliser et suivre toutes les données clés d’une affaire : informations générales, parties prenantes, données financières, planning, labels environnementaux, etc.

Développée selon les principes de la **clean architecture**, l'application garantit une séparation claire des responsabilités, une évolutivité du code, et une bonne testabilité de chaque couche (domaine, application, infrastructure, présentation).

## 🔍 Fonctionnalités principales

- Création, consultation, modification et suppression d’affaires
- Gestion complète des typologies et classifications
- Suivi des labels environnementaux et des actions RSE
- Gestion des intervenants et des données contractuelles
- Base de données pensée pour l’extensibilité et la clarté
- Backend développé avec NestJS, PostgreSQL et TypeORM

## 🛠️ Technologies utilisées

- **Backend** : NestJS, TypeORM
- **Base de données** : PostgreSQL
- **Langage** : TypeScript
- **Autres outils** : Prettier, ESLint, Jest

## 📂 Structure du projet

```
.
├── assets
│   └── templates
│       └── mailer
│           ├── account-activated.hbs
│           ├── activation.hbs
│           ├── password-reset-request.hbs
│           ├── password-reset-success.hbs
│           └── welcome.hbs
├── database
│   └── migrations
│       ├── 1648623447697-addUser.d.ts
│       ├── 1648623447697-addUser.js
│       └── 1648623447697-addUser.js.map
├── src
│   ├── app.module.*
│   ├── domain
│   │   ├── adapters
│   │   ├── config
│   │   ├── enums
│   │   ├── exceptions
│   │   ├── logger
│   │   ├── model
│   │   └── repositories
│   ├── infrastructure
│   │   ├── common
│   │   ├── config
│   │   ├── controllers
│   │   ├── entities
│   │   ├── exceptions
│   │   ├── logger
│   │   ├── repositories
│   │   ├── services
│   │   └── usecases-proxy
│   ├── main.*
│   └── usecases
│       └── business-case
└── tsconfig.build.tsbuildinfo
```

## 🚀 Scripts disponibles

Les scripts suivants sont définis dans le fichier `package.json` :

- `start` : Démarre l'application en mode production.
- `start:dev` : Démarre l'application en mode développement avec rechargement à chaud.
- `build` : Compile le projet.
- `test` : Exécute les tests unitaires.
- `lint` : Analyse le code avec ESLint et applique des corrections automatiques.
- `typeorm:*` : Commandes pour gérer les migrations avec TypeORM.

## 📜 Licence

Ce projet est sous licence **UNLICENSED**.
