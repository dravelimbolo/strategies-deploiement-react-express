# 9. Structures recommandées

Ce chapitre regroupe les arborescences de référence utilisées dans tout le cours.

## 9.1 Monorepo

```text
project/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── deploy.yml
│   └── dependabot.yml
│
├── frontend/
│   ├── src/
│   ├── public/
│   ├── index.html
│   ├── vite.config.js
│   ├── eslint.config.js
│   ├── .env.example
│   ├── package.json
│   └── package-lock.json
│
├── backend/
│   ├── src/
│   │   ├── app.js
│   │   └── server.js
│   ├── tests/
│   ├── scripts/
│   │   └── activate-release.sh
│   ├── ecosystem.config.js
│   ├── eslint.config.js
│   ├── .env.example
│   ├── package.json
│   └── package-lock.json
│
├── docs/
│   ├── architecture.md
│   ├── deployment.md
│   ├── security.md
│   └── api.md
│
├── .nvmrc
├── .gitignore
└── README.md
```

Séparer `app.js` (création de l'application Express, routes, middlewares) et `server.js` (appel à `app.listen`) permet de tester l'application avec `supertest` sans ouvrir de port.

## 9.2 Repository frontend séparé

```text
frontend/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── deploy.yml
│   └── dependabot.yml
├── src/
├── public/
├── index.html
├── vite.config.js
├── eslint.config.js
├── .env.example
├── .nvmrc
├── .gitignore
├── package.json
├── package-lock.json
└── README.md
```

## 9.3 Repository backend séparé

```text
backend/
├── .github/
│   ├── workflows/
│   │   ├── ci.yml
│   │   └── deploy.yml
│   └── dependabot.yml
├── src/
│   ├── app.js
│   └── server.js
├── tests/
├── scripts/
│   └── activate-release.sh
├── ecosystem.config.js
├── eslint.config.js
├── .env.example
├── .nvmrc
├── .gitignore
├── package.json
├── package-lock.json
└── README.md
```

## 9.4 Serveur

```text
/var/www/myapp/
├── frontend/
│   ├── releases/
│   │   ├── 20261001-120000-a1b2c3d/      fichiers de dist/
│   │   └── 20261002-090000-e4f5a6b/
│   └── current -> releases/20261002-090000-e4f5a6b
│
├── backend/
│   ├── releases/
│   │   ├── 20261001-120000-a1b2c3d/      code + node_modules + lien .env
│   │   └── 20261002-090000-e4f5a6b/
│   └── current -> releases/20261002-090000-e4f5a6b
│
├── shared/
│   ├── backend.env                        secrets du backend (droits 600)
│   └── logs/                              logs PM2
│
└── backups/                               sauvegardes de la base
```

Principes :

- chaque déploiement crée une nouvelle release, jamais modifiée ensuite ;
- le nom d'une release contient la date UTC et le SHA court du commit ;
- `current` est un lien symbolique vers la release active ;
- frontend et backend ont leurs propres releases, donc peuvent être déployés et restaurés séparément ;
- ce qui doit survivre aux déploiements (configuration, logs, sauvegardes) est en dehors des releases ;
- seules les 5 dernières releases sont conservées (script du chapitre 7).

Le dossier `backups/` est pratique pour démarrer, mais les sauvegardes doivent aussi être copiées hors du serveur (chapitre 8).
