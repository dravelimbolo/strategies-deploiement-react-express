# 2. Stratégie 1 : monorepo et un serveur

## 2.1 Architecture

```text
                              DÉVELOPPEUR
                                  |
                                  | git push / Pull Request
                                  v
                         +-----------------+
                         |     GITHUB      |
                         |    MONOREPO     |
                         |                 |
                         | frontend/       |
                         | backend/        |
                         | .github/        |
                         | .gitignore      |
                         | README.md       |
                         +--------+--------+
                                  |
                                  v
                         +-----------------+
                         | GITHUB ACTIONS  |
                         |      CI/CD      |
                         +--------+--------+
                                  |
                     +------------+------------+
                     |                         |
                     v                         v
                 React CI                  Express CI
                     |                         |
                 npm ci                    npm ci
                 lint                      lint
                 npm audit                 npm audit
                 npm test                  npm test
                 npm run build                 |
                     |                         |
                     +------------+------------+
                                  |
                                  v
                            SSH + rsync
                                  |
                                  v
                    +--------------------------+
                    |       SERVEUR VPS        |
                    |         NGINX            |
                    |      +----+----+         |
                    |      |         |         |
                    |      v         v         |
                    |    React    Express      |
                    |  statique   (PM2)        |
                    |                |         |
                    |                v         |
                    |            PostgreSQL    |
                    |            / MySQL       |
                    +--------------------------+
```

Le backend Express écrit en JavaScript n'a pas d'étape de build : il est déployé tel quel. Une étape `npm run build` n'est nécessaire côté backend que si le code est écrit en TypeScript ou passe par un outil de compilation.

## 2.2 Fonctionnement du frontend

React est compilé avant le déploiement :

```bash
npm ci
npm run build
```

Le résultat est un répertoire de fichiers statiques (HTML, CSS, JavaScript), `dist/` avec Vite.

> Create React App (qui produisait un dossier `build/`) est officiellement déprécié depuis 2025. Pour un nouveau projet, utilisez Vite.

Nginx sert directement ces fichiers. Aucun processus Node.js ne tourne pour le frontend en production.

## 2.3 Fonctionnement du backend

Express reste un processus Node.js qui doit tourner en permanence.

PM2 permet de :

- démarrer l'application ;
- la maintenir active ;
- la redémarrer après un plantage ;
- consulter les logs ;
- la relancer automatiquement après un redémarrage du serveur.

Exemple :

```bash
pm2 start ecosystem.config.js
pm2 save
```

Attention : `pm2 save` enregistre la liste des processus, mais ne suffit pas à les relancer au démarrage du serveur. Il faut aussi exécuter une fois `pm2 startup`, qui crée un service systemd. Le fichier `ecosystem.config.js` complet et la procédure sont détaillés au chapitre 7.

## 2.4 Nginx

Une séparation typique utilise deux sous-domaines :

```text
https://app.example.com
        |
        v
      Nginx
        |
        v
   React statique
```

et :

```text
https://api.example.com
        |
        v
      Nginx
        |
        v
127.0.0.1:3000
        |
        v
     Express
```

Nginx peut également gérer :

- HTTPS/TLS ;
- redirections HTTP vers HTTPS ;
- compression ;
- cache des fichiers statiques ;
- reverse proxy vers Express ;
- headers de sécurité.

Les fichiers de configuration Nginx complets sont donnés au chapitre 7.

## 2.5 Communication frontend/backend et CORS

Deux organisations sont possibles.

### Option A : deux sous-domaines

```text
https://app.example.com  -> React
https://api.example.com  -> Express
```

Le navigateur considère `app.example.com` et `api.example.com` comme deux **origines différentes**. Par sécurité, il bloque les appels de l'une vers l'autre, sauf si le backend l'autorise explicitement via **CORS** (Cross-Origin Resource Sharing).

Sans configuration CORS, l'apprenant verra dans la console du navigateur une erreur du type :

```text
Access to fetch at 'https://api.example.com/...' from origin 'https://app.example.com'
has been blocked by CORS policy
```

Configuration côté Express avec le paquet `cors` :

```js
const cors = require('cors');

app.use(
  cors({
    origin: process.env.CORS_ORIGIN, // par exemple https://app.example.com
    credentials: true, // seulement si des cookies sont utilisés
  })
);
```

Règles :

- autoriser uniquement les origines connues, jamais `*` en production si des cookies ou des tokens sont utilisés ;
- mettre l'origine autorisée dans une variable d'environnement, car elle diffère entre staging et production.

### Option B : un seul domaine avec un préfixe `/api`

```text
https://app.example.com/        -> React
https://app.example.com/api/... -> Express
```

Nginx redirige `/api/` vers Express. Le navigateur ne voit qu'une seule origine : **aucune configuration CORS n'est nécessaire**. C'est souvent l'option la plus simple pour la stratégie 1.

Bloc Nginx correspondant, à ajouter dans le `server` du frontend :

```nginx
location /api/ {
    proxy_pass http://127.0.0.1:3000;
    proxy_set_header Host $host;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header X-Forwarded-Proto $scheme;
}
```

Dans ce cas, les routes Express doivent commencer par `/api`.

## 2.6 Base de données

La base de données ne doit pas être exposée publiquement.

Architecture attendue :

```text
Internet
   |
 Nginx
   |
Express
   |
Database (écoute uniquement sur 127.0.0.1)
```

et non :

```text
Internet
   |
Database:5432/3306
```

La configuration correspondante est détaillée au chapitre 8.

## 2.7 Structure serveur

Le serveur est organisé en **releases** : chaque déploiement crée un nouveau dossier, et un lien symbolique `current` pointe vers la version active. Frontend et backend ont chacun leurs releases, ce qui permet de les déployer et de les restaurer séparément.

```text
/var/www/myapp/
├── frontend/
│   ├── releases/
│   │   ├── 20261001-120000-a1b2c3d/
│   │   └── 20261002-090000-e4f5a6b/
│   └── current -> releases/20261002-090000-e4f5a6b
├── backend/
│   ├── releases/
│   │   ├── 20261001-120000-a1b2c3d/
│   │   └── 20261002-090000-e4f5a6b/
│   └── current -> releases/20261002-090000-e4f5a6b
├── shared/
│   ├── backend.env
│   └── logs/
└── backups/
```

Cette structure est la structure de référence du cours. Elle est expliquée au chapitre 9 et utilisée pour le rollback au chapitre 10.

## 2.8 Pipeline

```text
git push
   |
   v
GitHub
   |
   v
GitHub Actions
   |
   +--> lint
   +--> audit des dépendances
   +--> tests frontend
   +--> build frontend
   +--> tests backend
   |
   v
SSH
   |
   v
VPS
   |
   +--> Express / PM2 (déployé en premier)
   |
   +--> React / Nginx
```

Le backend est déployé avant le frontend : un nouveau frontend peut avoir besoin d'une nouvelle route d'API, l'inverse est plus rare.

## 2.9 Organisation Git

Le flux de branches (`feature/*`, `develop`, `main`) et le lien avec les environnements staging et production sont décrits au chapitre 10.

## Avantages

- une seule base de code ;
- configuration centralisée ;
- CI/CD centralisée ;
- partage facile de documentation ;
- coordination frontend/backend simplifiée (une même pull request peut modifier les deux) ;
- infrastructure initiale simple.

## Limites

- frontend et backend sont plus couplés ;
- les déploiements sont moins indépendants ;
- un seul serveur est un point de défaillance unique ;
- une montée en charge séparée du frontend et du backend est moins naturelle.
