# 16. Autres types d'hébergement : Render et hébergement mutualisé (cPanel)

Le cours utilise un VPS, car c'est là qu'on voit le mieux le rôle de chaque brique. Mais pour un premier projet, il existe des solutions plus simples. Ce chapitre présente deux alternatives très répandues.

## 16.1 Les grandes familles d'hébergement

| Type | Exemples | Ce que vous gérez | Node.js / Express | Pour qui |
|---|---|---|---|---|
| Hébergeur de sites statiques | Netlify, Vercel, Cloudflare Pages, Render (Static Site) | Presque rien | Non (frontend uniquement) | Le frontend React |
| PaaS (plateforme) | Render, Railway, Heroku | Le code et les variables d'environnement | Oui | Débutants, premiers projets |
| Hébergement mutualisé | Offres avec cPanel (o2switch, Hostinger, LWS...) | Les fichiers, via une interface web | Parfois, avec des limites | Sites simples, offre déjà payée |
| VPS | OVHcloud, Hetzner, Scaleway, DigitalOcean | Tout : système, sécurité, logiciels | Oui | Ce cours, projets sérieux |

Plus on descend dans le tableau, plus on a de contrôle, mais plus on a de responsabilités (sécurité, mises à jour, sauvegardes).

Ce qui ne change **jamais**, quel que soit l'hébergement :

- les secrets restent hors de Git ;
- la CI (lint, audit des dépendances, tests, build) passe avant tout déploiement ;
- le frontend appelle l'URL publique de l'API, et CORS doit être configuré si les domaines diffèrent ;
- `package-lock.json` est versionné ;
- la base de données est sauvegardée.

## 16.2 Render (PaaS)

Render est une plateforme qui héberge le frontend, le backend et la base de données à partir d'un repository GitHub. Pas de serveur à administrer : pas de Nginx, pas de PM2, pas de firewall à configurer, HTTPS automatique.

### Correspondance avec le VPS

| Sur le VPS | Sur Render |
|---|---|
| Nginx sert `dist/` | Service de type **Static Site** |
| PM2 lance Express | Service de type **Web Service** |
| PostgreSQL installé sur le serveur | Base **Render Postgres** managée |
| `shared/backend.env` | Variables d'environnement dans le tableau de bord |
| Certbot | HTTPS automatique |
| Script de rollback | Bouton **Rollback** dans l'historique des déploiements |

### Frontend React (Static Site)

Réglages à saisir dans Render :

- **Root Directory** : `frontend` (si monorepo) ;
- **Build Command** : `npm ci && npm run build` ;
- **Publish Directory** : `dist` ;
- **Environment** : `VITE_API_URL` avec l'URL de l'API ;
- **Redirects/Rewrites** : une règle de type *Rewrite* de `/*` vers `/index.html`. C'est l'équivalent du `try_files` de Nginx : sans elle, recharger une page autre que l'accueil donne une erreur 404.

### Backend Express (Web Service)

- **Root Directory** : `backend` (si monorepo) ;
- **Build Command** : `npm ci` ;
- **Start Command** : `node src/server.js` ;
- **Health Check Path** : `/health` ;
- **Environment** : `DATABASE_URL`, `JWT_SECRET`, `CORS_ORIGIN`...

Attention, une différence importante avec le VPS : sur Render, Express doit écouter sur le port fourni par la plateforme et sur **toutes les interfaces** :

```js
app.listen(process.env.PORT || 3000, '0.0.0.0');
```

Sur le VPS, on écoute sur `127.0.0.1` car Nginx est sur la même machine. Sur Render, c'est la plateforme qui joue le rôle de Nginx, depuis une autre machine.

### Garder la CI avant le déploiement

Par défaut, Render redéploie à chaque push, **même si les tests échouent**. Deux solutions :

1. dans les réglages du service, choisir de ne déployer qu'après le succès des vérifications GitHub (option *Auto-Deploy* « After CI Checks Pass » si elle est proposée) ;
2. ou désactiver l'auto-déploiement et déclencher le déploiement depuis GitHub Actions avec un **Deploy Hook** (une URL secrète fournie par Render) :

```yaml
  deploy:
    needs: ci
    runs-on: ubuntu-latest
    environment: production
    steps:
      - name: Déclencher le déploiement Render
        env:
          RENDER_DEPLOY_HOOK_URL: ${{ secrets.RENDER_DEPLOY_HOOK_URL }}
        run: curl -fsS -X POST "$RENDER_DEPLOY_HOOK_URL"
```

L'URL du Deploy Hook est un secret : quiconque la connaît peut lancer un déploiement.

### Limites à connaître

- Sur l'offre gratuite, le Web Service se met en veille après une période d'inactivité : la première requête suivante peut prendre près d'une minute.
- La base Postgres gratuite a une durée de vie limitée : elle convient pour un TP, pas pour de vraies données.
- Les adresses `*.onrender.com` du frontend et de l'API sont considérées comme deux sites différents : pour utiliser des cookies, il faut un domaine personnalisé (`app.example.com` et `api.example.com`), voir chapitre 3 section 3.5.
- Les conditions des offres changent : vérifiez-les sur le site de Render.

## 16.3 Hébergement mutualisé avec cPanel

Un hébergement mutualisé partage un serveur entre de nombreux clients. On n'a ni accès `root`, ni Nginx, ni PM2 : tout se gère depuis l'interface **cPanel**. Le serveur web est généralement **Apache**.

C'est fréquent quand une école, une association ou un client possède déjà une offre de ce type.

### Frontend React : très bien adapté

Un build React n'est qu'un ensemble de fichiers statiques : n'importe quel hébergement mutualisé peut le servir.

1. Construire le projet : `npm run build`.
2. Envoyer le **contenu** de `dist/` dans `public_html/` (ou dans le dossier du sous-domaine).
3. Ajouter dans ce dossier un fichier `.htaccess` pour le fonctionnement en SPA (équivalent Apache du `try_files` de Nginx) :

```apache
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

Placer ce fichier dans `frontend/public/` : Vite le copie automatiquement dans `dist/` à chaque build.

4. Activer HTTPS avec l'outil **SSL/TLS Status** ou **AutoSSL** de cPanel (certificats gratuits).

### Déployer automatiquement depuis GitHub Actions

On ne peut pas utiliser `rsync` + SSH comme sur le VPS (SSH est souvent absent ou limité). On envoie les fichiers par FTPS :

```yaml
      - name: Envoyer le build sur l'hébergement mutualisé
        uses: SamKirkland/FTP-Deploy-Action@v4.3.5
        with:
          server: ${{ secrets.FTP_SERVER }}
          username: ${{ secrets.FTP_USERNAME }}
          password: ${{ secrets.FTP_PASSWORD }}
          protocol: ftps
          local-dir: ./frontend/dist/
          server-dir: ./public_html/
```

Bonnes pratiques :

- créer dans cPanel un **compte FTP dédié** au déploiement, limité au dossier du site (jamais le compte principal cPanel) ;
- utiliser `ftps` (chiffré), jamais `ftp` simple : sinon le mot de passe circule en clair ;
- stocker les identifiants dans les secrets d'environnement GitHub, comme au chapitre 6.

### Backend Express : seulement si l'offre le permet

Certaines offres proposent un outil **Setup Node.js App** (Node.js Selector, basé sur Phusion Passenger). Sans cet outil, on ne peut pas héberger Express sur un mutualisé.

Étapes habituelles :

1. Dans **Setup Node.js App**, créer une application : version de Node (la plus proche de `.nvmrc`), dossier de l'application (par exemple `myapp-api`), URL (par exemple `api.example.com`), fichier de démarrage (`src/server.js`).
2. Envoyer le code du backend dans ce dossier, **sans** `node_modules`.
3. Cliquer sur **Run NPM Install**.
4. Déclarer les variables d'environnement (`DATABASE_URL`, `JWT_SECRET`...) dans l'interface.
5. Démarrer ou redémarrer l'application depuis l'interface.

Passenger remplace PM2 : il lance et relance l'application lui-même. Il ne faut donc pas utiliser PM2 ici. Express doit écouter sur le port fourni (`process.env.PORT`) et ne pas fixer le port en dur.

Après une mise à jour du code, l'application doit être redémarrée : bouton **Restart** dans l'interface, ou création du fichier `tmp/restart.txt` dans le dossier de l'application.

### Base de données

- Généralement **MySQL / MariaDB**, créée avec l'outil **MySQL Databases** et administrée avec phpMyAdmin. PostgreSQL est plus rarement proposé.
- Créer un utilisateur dédié à l'application, avec des droits uniquement sur sa base.
- Ne pas activer l'accès distant (**Remote MySQL**) sauf nécessité, et dans ce cas l'autoriser seulement pour une adresse IP précise.
- Utiliser l'outil de sauvegarde de cPanel, et télécharger régulièrement une copie.

### Limites à connaître

- Ressources partagées et limitées (CPU, mémoire, nombre de processus) : ne convient pas aux applications chargées.
- Versions de Node.js parfois anciennes : vérifier qu'elles correspondent à `.nvmrc`.
- Pas de releases ni de lien `current` : un rollback consiste à redéployer un ancien commit (par exemple avec `git revert` sur `main`).
- Pas de contrôle du firewall, du système ni des mises à jour : c'est l'hébergeur qui s'en charge.

## 16.4 Quel hébergement choisir pour débuter ?

| Situation | Choix conseillé |
|---|---|
| Premier déploiement, apprendre la CI/CD sans administrer de serveur | Render (frontend et backend) |
| Frontend seul, ou offre mutualisée déjà payée | Hébergement statique ou cPanel pour le frontend, Render pour l'API |
| Comprendre toute la chaîne, projet destiné à durer | VPS, en suivant les chapitres 7 et 8 |

Progression recommandée pour les apprenants : déployer d'abord sur Render pour voir le résultat rapidement, puis refaire le même déploiement sur un VPS pour comprendre ce que la plateforme faisait à votre place.
