# 3. Stratégie 2 : deux repositories et déploiements indépendants

## 3.1 Architecture

```text
                       GITHUB
                         |
             +-----------+-----------+
             |                       |
             v                       v
       FRONTEND REPO            BACKEND REPO
             |                       |
           React                   Express
             |                       |
             v                       v
      GitHub Actions          GitHub Actions
             |                       |
             v                       v
   Lint / Audit / Tests     Lint / Audit / Tests
        Build React                  |
             |                       |
             v                       v
   Hébergement statique           VPS API
       ou serveur Nginx              |
             |                       |
             v                       v
      app.example.com          api.example.com
                                     |
                                     v
                                  Database
```

## 3.2 Repository frontend

L'arborescence de référence est donnée au chapitre 9 (section 9.2).

Pipeline :

```text
push / pull request
 |
 v
npm ci
 |
 v
lint
 |
 v
audit des dépendances
 |
 v
tests
 |
 v
npm run build
 |
 v
déploiement
```

## 3.3 Repository backend

L'arborescence de référence est donnée au chapitre 9 (section 9.3).

Pipeline :

```text
push / pull request
 |
 v
npm ci
 |
 v
lint
 |
 v
audit des dépendances
 |
 v
tests
 |
 v
SSH + rsync
 |
 v
npm ci --omit=dev sur le serveur
 |
 v
PM2 reload
 |
 v
health check
```

Les workflows du chapitre 6 s'appliquent : il suffit de garder uniquement la partie frontend dans un repository et la partie backend dans l'autre.

## 3.4 Un ou deux hébergeurs

Avec deux repositories, on peut garder un seul serveur pour tout (comme la stratégie 1), ou choisir un hébergement adapté à chaque composant :

```text
                 GITHUB
                    |
          +---------+---------+
          |                   |
          v                   v
     Hébergeur A         Hébergeur B
      Frontend             Backend
          |                   |
          v                   v
        React              Express
                              |
                              v
                           Database
```

Exemples d'hébergeurs :

- frontend statique : Netlify, Vercel, Cloudflare Pages, ou un VPS avec Nginx ;
- backend : un VPS (OVHcloud, Hetzner, Scaleway, DigitalOcean...) avec Nginx et PM2, ou une plateforme comme Render.

Le chapitre 16 détaille Render et l'hébergement mutualisé (cPanel).

Si la base de données est un service managé séparé du serveur API, elle est accessible par le réseau : il faut alors limiter les connexions à l'adresse IP du serveur API et utiliser une connexion chiffrée (TLS).

## 3.5 Communication frontend/backend

Le frontend appelle l'URL publique de l'API :

```text
https://api.example.com
```

Il ne doit jamais utiliser une adresse interne telle que :

```text
http://127.0.0.1:3000
```

en production : `127.0.0.1` désigne la machine de l'utilisateur, pas le serveur.

L'URL de l'API est fournie au build via une variable d'environnement (par exemple `VITE_API_URL`, voir chapitre 6 section 6.3).

Frontend et backend étant sur deux origines différentes, **CORS doit être configuré** côté Express (voir chapitre 2 section 2.5).

Attention aux cookies : si le frontend est servi depuis un domaine fourni par l'hébergeur (par exemple `monapp.netlify.app`) et l'API depuis `api.example.com`, les deux sites sont considérés comme différents et de nombreux navigateurs bloquent les cookies dits tiers. Utiliser des sous-domaines d'un même domaine (`app.example.com` et `api.example.com`) évite ce problème.

## 3.6 Attention au contrat d'API

Lorsque frontend et backend sont déployés séparément, une version du frontend peut se retrouver face à une version différente du backend. Il faut gérer la compatibilité.

Solutions possibles :

- documentation OpenAPI ;
- versionnement de l'API ;
- tests d'intégration ;
- règle simple : ne jamais supprimer ou renommer un champ utilisé par le frontend en production sans période de transition.

Exemple de versionnement :

```text
Frontend v1
   |
   v
API /v1
```

Puis, lors d'un changement incompatible :

```text
Frontend v2
   |
   v
API /v2   (API /v1 reste disponible le temps de la migration)
```

## Avantages

- déploiements indépendants ;
- équipes frontend/backend découplées ;
- possibilité de faire évoluer l'infrastructure séparément ;
- scalabilité indépendante ;
- permissions GitHub plus faciles à séparer ;
- historique Git plus ciblé.

## Limites

- deux repositories à maintenir ;
- deux pipelines ;
- coordination d'API nécessaire ;
- configuration CORS obligatoire ;
- davantage de configuration ;
- potentiellement plusieurs infrastructures à administrer.
