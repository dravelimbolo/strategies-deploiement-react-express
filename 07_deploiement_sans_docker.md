# 7. Déploiement sans Docker

## 7.1 Principe

Sans Docker, l'application utilise directement les logiciels installés sur le serveur :

```text
VPS
 |
 +-- Linux (Ubuntu LTS par exemple)
 +-- Node.js
 +-- Nginx
 +-- PM2
 +-- PostgreSQL/MySQL
```

Le serveur est préparé **une seule fois** (sections 7.2 à 7.6). Ensuite, chaque déploiement est automatique (section 7.7 et chapitre 6).

## 7.2 Version de Node

Sans Docker, la cohérence des versions devient essentielle : le développeur, la CI et le serveur doivent utiliser la même version majeure de Node.js.

Recommandations :

- un fichier `.nvmrc` à la racine du repository, contenant la version majeure, par exemple :

```text
24
```

- le même fichier est lu par la CI (`node-version-file: .nvmrc`, chapitre 6) et par `nvm use` chez les développeurs ;
- le champ `engines` dans chaque `package.json` documente la version attendue :

```json
{
  "engines": {
    "node": ">=24 <25"
  }
}
```

- `package-lock.json` versionné et `npm ci` partout.

Utiliser une version **LTS** (Long Term Support) de Node.js. Vérifier la version LTS en cours sur nodejs.org.

## 7.3 Préparer le serveur

Commandes exécutées une fois, avec un compte administrateur (pas `root` directement, voir chapitre 8).

Paquets de base :

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y nginx rsync curl
```

Node.js depuis le dépôt officiel NodeSource (même version majeure que `.nvmrc`) :

```bash
curl -fsSL https://deb.nodesource.com/setup_24.x -o nodesource_setup.sh
sudo bash nodesource_setup.sh
sudo apt install -y nodejs
node -v
```

On évite `nvm` sur le serveur : avec nvm, Node n'est disponible que dans un shell interactif, et les commandes lancées par SSH depuis GitHub Actions échouent souvent avec `npm: command not found`.

PM2 :

```bash
sudo npm install -g pm2
```

Utilisateur de déploiement et arborescence :

```bash
sudo adduser --disabled-password --gecos "" deploy

sudo mkdir -p /var/www/myapp/frontend/releases \
              /var/www/myapp/backend/releases \
              /var/www/myapp/shared/logs \
              /var/www/myapp/backups
sudo chown -R deploy:deploy /var/www/myapp
```

Les dossiers créés sont lisibles par tous les utilisateurs du système, ce qui permet à Nginx (utilisateur `www-data`) de lire les fichiers du frontend.

Clé publique de déploiement (chapitre 6 section 6.10) :

```bash
sudo -u deploy mkdir -p /home/deploy/.ssh
sudo -u deploy chmod 700 /home/deploy/.ssh
sudo -u deploy nano /home/deploy/.ssh/authorized_keys   # coller deploy_key.pub
sudo -u deploy chmod 600 /home/deploy/.ssh/authorized_keys
```

## 7.4 Secrets du backend sur le serveur

Les variables d'environnement du backend sont stockées **hors des releases**, dans un fichier partagé :

```bash
sudo -u deploy nano /var/www/myapp/shared/backend.env
sudo -u deploy chmod 600 /var/www/myapp/shared/backend.env
```

Contenu (en reprenant les noms de `.env.example`) :

```env
NODE_ENV=production
PORT=3000
DATABASE_URL=postgresql://myapp_user:mot-de-passe-long@127.0.0.1:5432/myapp
JWT_SECRET=une-longue-valeur-aleatoire
CORS_ORIGIN=https://app.example.com
```

Générer une valeur aléatoire :

```bash
openssl rand -base64 48
```

À chaque déploiement, le script de release crée dans la nouvelle release un lien `.env` vers ce fichier. L'application le charge avec `dotenv` :

```js
require('dotenv').config();
```

## 7.5 Configuration Nginx

### Frontend React

Fichier `/etc/nginx/sites-available/app.example.com` :

```nginx
server {
    listen 80;
    server_name app.example.com;

    root /var/www/myapp/frontend/current;
    index index.html;

    # Application monopage (SPA) : toute URL inconnue renvoie index.html,
    # et c'est React Router qui affiche la bonne page.
    location / {
        try_files $uri $uri/ /index.html;
    }

    # Fichiers générés par Vite : leur nom contient un hash, on peut les garder longtemps en cache.
    location /assets/ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # index.html ne doit pas être gardé en cache, sinon les utilisateurs ne voient pas la nouvelle version.
    location = /index.html {
        add_header Cache-Control "no-cache";
    }

    gzip on;
    gzip_types text/css application/javascript application/json image/svg+xml;
}
```

Sans la directive `try_files ... /index.html`, recharger la page sur une URL comme `/profil` renvoie une erreur 404, car ce fichier n'existe pas sur le disque.

### API Express

Fichier `/etc/nginx/sites-available/api.example.com` :

```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://127.0.0.1:3000;
        proxy_http_version 1.1;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }
}
```

Activer les deux sites et vérifier la configuration :

```bash
sudo ln -s /etc/nginx/sites-available/app.example.com /etc/nginx/sites-enabled/
sudo ln -s /etc/nginx/sites-available/api.example.com /etc/nginx/sites-enabled/
sudo nginx -t
sudo systemctl reload nginx
```

Toujours lancer `sudo nginx -t` avant un rechargement : une erreur de syntaxe empêcherait Nginx de redémarrer.

Le passage en HTTPS (Certbot) est expliqué au chapitre 8. Certbot modifie automatiquement ces fichiers pour ajouter le certificat et la redirection HTTP vers HTTPS.

Si vous avez choisi un seul domaine avec un préfixe `/api` (chapitre 2 section 2.5, option B), il n'y a qu'un seul fichier : celui du frontend, avec en plus le bloc `location /api/`.

### Côté Express

Express est derrière Nginx. Deux réglages sont nécessaires :

```js
// Faire confiance au proxy Nginx pour connaître l'IP réelle du client et le protocole HTTPS
app.set('trust proxy', 1);

// N'écouter que sur l'interface locale : le port 3000 n'est pas joignable depuis Internet
app.listen(process.env.PORT || 3000, '127.0.0.1');
```

## 7.6 Configuration PM2

Fichier `backend/ecosystem.config.js`, versionné dans le repository :

```js
const path = require('path');

const appDir = process.env.APP_DIR || '/var/www/myapp';

module.exports = {
  apps: [
    {
      name: 'myapp-api',
      cwd: path.join(appDir, 'backend/current'),
      script: 'src/server.js',
      exec_mode: 'cluster',
      instances: 2,
      env: {
        NODE_ENV: 'production',
      },
      out_file: path.join(appDir, 'shared/logs/api.out.log'),
      error_file: path.join(appDir, 'shared/logs/api.err.log'),
      time: true,
    },
  ],
};
```

Points importants :

- `cwd` pointe vers le lien `current` : après un changement de release, PM2 relance le code de la nouvelle version ;
- `exec_mode: 'cluster'` avec au moins 2 instances permet un `pm2 reload` **sans interruption** : les instances sont redémarrées une par une. En mode `fork` (par défaut), un reload équivaut à un redémarrage avec une courte coupure ;
- en mode cluster, ne pas stocker d'état en mémoire (sessions par exemple) : chaque requête peut arriver sur une instance différente.

Démarrage automatique de PM2 au démarrage du serveur (une seule fois) :

```bash
sudo env PATH=$PATH:/usr/bin pm2 startup systemd -u deploy --hp /home/deploy
```

Puis, après le premier déploiement, en tant que `deploy` :

```bash
pm2 save
```

Rotation des logs (sinon ils grossissent sans limite) :

```bash
sudo -u deploy pm2 install pm2-logrotate
```

Commandes utiles (en tant que `deploy`) :

```bash
pm2 status
pm2 logs myapp-api
pm2 monit
```

## 7.7 Script d'activation d'une release backend

Fichier `backend/scripts/activate-release.sh`, versionné dans le repository et appelé par GitHub Actions (chapitre 6 section 6.7) :

```bash
#!/usr/bin/env bash
set -euo pipefail

APP_DIR="$1"
RELEASE="$2"
KEEP_RELEASES=5
HEALTH_URL="http://127.0.0.1:3000/health"

export APP_DIR
RELEASES_DIR="$APP_DIR/backend/releases"
RELEASE_DIR="$RELEASES_DIR/$RELEASE"
CURRENT="$APP_DIR/backend/current"
PREVIOUS="$(readlink -f "$CURRENT" || true)"

switch_to() {
  ln -sfn "$1" "$CURRENT.tmp"
  mv -Tf "$CURRENT.tmp" "$CURRENT"
  pm2 startOrReload "$CURRENT/ecosystem.config.js" --update-env
}

echo "Installation des dépendances de production"
cd "$RELEASE_DIR"
npm ci --omit=dev

echo "Lien vers la configuration partagée"
ln -sfn "$APP_DIR/shared/backend.env" "$RELEASE_DIR/.env"

# Migrations de base de données éventuelles (voir chapitre 11) :
# npm run migrate

echo "Activation de la release $RELEASE"
switch_to "$RELEASE_DIR"

echo "Health check"
for attempt in 1 2 3 4 5 6 7 8 9 10; do
  if curl -fsS "$HEALTH_URL" > /dev/null; then
    echo "Health check OK"
    pm2 save

    echo "Suppression des anciennes releases (on garde les $KEEP_RELEASES dernières)"
    ls -1d "$RELEASES_DIR"/*/ | sort -r | tail -n +$((KEEP_RELEASES + 1)) | while read -r dir; do
      if [ "$(readlink -f "$dir")" != "$(readlink -f "$CURRENT")" ]; then
        rm -rf "$dir"
      fi
    done
    exit 0
  fi
  sleep 3
done

echo "Health check en échec : retour à la release précédente"
if [ -n "$PREVIOUS" ] && [ -d "$PREVIOUS" ]; then
  switch_to "$PREVIOUS"
fi
exit 1
```

Le script se termine par un code d'erreur en cas d'échec : le job GitHub Actions devient rouge, et la release précédente reste en service.

## 7.8 Pourquoi `ln -sfn` puis `mv -Tf`

Remplacer directement le lien `current` avec `ln -sfn` supprime l'ancien lien puis crée le nouveau : pendant un très court instant, `current` n'existe pas.

La technique utilisée crée d'abord un lien temporaire `current.tmp`, puis le renomme en `current`. Sous Linux, un renommage est **atomique** : à tout instant, `current` pointe soit vers l'ancienne release, soit vers la nouvelle.

## 7.9 Ordre d'un déploiement

```text
Build et tests (CI)
 |
 v
Envoi de la nouvelle release sur le serveur (rsync)
 |
 v
Installation des dépendances (npm ci --omit=dev)
 |
 v
Migrations de base de données si nécessaire
 |
 v
Bascule du lien current
 |
 v
Rechargement PM2
 |
 v
Health check
 |
 +--> OK     -> nettoyage des anciennes releases
 |
 +--> échec  -> retour à la release précédente
```

Le health check a lieu **après** la bascule : avec un seul port, la nouvelle version ne peut pas être testée avant d'être active. Pour tester une version avant d'y envoyer du trafic, il faut une approche blue/green (chapitre 10).

## 7.10 Avantage

Le déploiement est direct, chaque brique est visible et compréhensible, et il n'y a pas de couche Docker à apprendre ni à maintenir.

## 7.11 Limite

L'environnement du serveur doit être correctement administré. Deux serveurs peuvent avoir :

```text
Node différent
Nginx différent
dépendances système différentes
configuration différente
```

Une discipline d'administration est donc nécessaire : documenter chaque commande de préparation du serveur (comme dans ce chapitre), et idéalement l'automatiser dans un script de provisioning versionné.
