# 8. Sécurité en production

## 8.1 Secrets

Ne jamais versionner :

- mots de passe ;
- clés API ;
- secrets JWT ;
- clés privées SSH ;
- identifiants de base de données.

Où les ranger :

| Secret | Emplacement |
|---|---|
| Accès SSH au serveur pour le déploiement | Secrets d'environnement GitHub (chapitre 6) |
| Secrets de l'application (base, JWT, API tierces) | `shared/backend.env` sur le serveur, droits `600` (chapitre 7) |
| Valeurs locales du développeur | `.env` local, ignoré par Git (chapitre 5) |

Si un secret a été publié par erreur : le révoquer et le régénérer immédiatement (chapitre 5 section 5.4).

## 8.2 Base de données

La base doit être accessible uniquement par les services qui en ont besoin.

```text
Internet ----X----> Database    (bloqué)

Express  ---------> Database    (autorisé)
```

Express est le seul service autorisé à communiquer avec la base.

### Écouter uniquement en local

Quand la base est sur le même serveur qu'Express, elle ne doit écouter que sur l'interface locale :

- PostgreSQL : dans `postgresql.conf`, `listen_addresses = 'localhost'` ;
- MySQL / MariaDB : dans la configuration du serveur, `bind-address = 127.0.0.1`.

C'est généralement la valeur par défaut sur Ubuntu, mais il faut le vérifier. Pour contrôler les ports ouverts :

```bash
sudo ss -tlnp
```

Le port `5432` (PostgreSQL) ou `3306` (MySQL) doit apparaître avec l'adresse `127.0.0.1`, jamais `0.0.0.0`.

### Un utilisateur dédié à l'application

L'application ne se connecte jamais avec le compte administrateur de la base (`postgres` ou `root`). On crée un utilisateur qui n'a des droits que sur la base de l'application.

Exemple PostgreSQL :

```sql
CREATE USER myapp_user WITH PASSWORD 'mot-de-passe-long-et-aleatoire';
CREATE DATABASE myapp OWNER myapp_user;
```

Utiliser des requêtes paramétrées (ou un ORM) pour se protéger des injections SQL : ne jamais construire une requête SQL en concaténant des valeurs envoyées par l'utilisateur.

## 8.3 Accès SSH

Utiliser :

- des clés SSH, jamais de mot de passe ;
- un compte administrateur personnel (avec `sudo`) pour l'administration ;
- l'utilisateur `deploy`, **sans** `sudo`, pour les déploiements ;
- des permissions minimales.

Avec l'architecture de ce cours, le déploiement n'a besoin d'aucun droit `sudo` : `deploy` possède `/var/www/myapp` et lance PM2 sous son propre compte.

Désactiver la connexion par mot de passe et la connexion directe de `root`, dans `/etc/ssh/sshd_config` :

```text
PasswordAuthentication no
PermitRootLogin no
```

Puis :

```bash
sudo sshd -t
sudo systemctl reload ssh
```

Sur de nombreuses images cloud, un fichier du dossier `/etc/ssh/sshd_config.d/` (par exemple `50-cloud-init.conf`) contient `PasswordAuthentication yes` et prend le dessus : vérifier ce dossier. La commande `sudo sshd -T | grep -i -E 'passwordauthentication|permitrootlogin'` affiche les valeurs réellement appliquées.

Attention : avant de recharger SSH, vérifier depuis un **second terminal** que la connexion par clé fonctionne avec votre compte administrateur. Sinon, vous risquez de perdre l'accès au serveur.

Optionnel : `fail2ban` bloque temporairement les adresses IP qui multiplient les tentatives de connexion échouées.

## 8.4 Firewall

N'ouvrir que les ports nécessaires : SSH (22), HTTP (80) et HTTPS (443).

```bash
sudo ufw default deny incoming
sudo ufw default allow outgoing
sudo ufw allow OpenSSH
sudo ufw allow 'Nginx Full'
sudo ufw enable
sudo ufw status
```

Toujours autoriser `OpenSSH` **avant** `ufw enable`, sinon la connexion SSH en cours est coupée.

Les ports 3000 (Express) et 5432/3306 (base) ne sont pas ouverts : ils ne sont accessibles que depuis le serveur lui-même.

Certains hébergeurs proposent aussi un firewall dans leur interface : les deux peuvent se cumuler.

## 8.5 HTTPS

Nginx termine TLS :

```text
Client
  |
 HTTPS (chiffré)
  |
 Nginx
  |
 HTTP local (127.0.0.1)
  |
 Express
```

Le trafic public doit toujours utiliser HTTPS. Certificats gratuits avec Let's Encrypt et Certbot :

```bash
sudo apt install -y certbot python3-certbot-nginx
sudo certbot --nginx -d app.example.com -d api.example.com
```

Certbot :

- obtient les certificats ;
- modifie les fichiers Nginx du chapitre 7 pour écouter en HTTPS ;
- propose d'ajouter la redirection HTTP vers HTTPS (à accepter) ;
- installe un renouvellement automatique.

Vérifier le renouvellement :

```bash
sudo certbot renew --dry-run
```

Prérequis : les enregistrements DNS des domaines doivent déjà pointer vers l'adresse IP du serveur.

## 8.6 Protection de l'API Express

Paquets recommandés :

- `helmet` : ajoute des headers HTTP de sécurité ;
- `express-rate-limit` : limite le nombre de requêtes par client, en particulier sur les routes de connexion ;
- `cors` : n'autorise que les origines connues (chapitre 2 section 2.5).

```js
const helmet = require('helmet');
const rateLimit = require('express-rate-limit');

app.use(helmet());
app.use('/auth', rateLimit({ windowMs: 15 * 60 * 1000, limit: 100 }));
```

Ne jamais renvoyer au client le détail d'une erreur interne (pile d'appels, requête SQL) en production.

## 8.7 Variables frontend

Une variable injectée dans une application React n'est **jamais** secrète.

Avec Vite, toutes les variables `VITE_*` sont copiées dans les fichiers JavaScript envoyés au navigateur. N'importe quel utilisateur peut les lire avec les outils de développement.

```text
VITE_API_URL=https://api.example.com   -> acceptable, ce n'est pas un secret
VITE_STRIPE_SECRET_KEY=sk_live_...      -> grave : la clé est publique
```

Les secrets doivent rester côté backend. Si le frontend a besoin d'un service protégé par une clé, c'est le backend qui appelle ce service.

## 8.8 Dépendances

Les dépendances npm sont une surface d'attaque importante. Le chapitre 6 (section 6.5) met en place :

- `npm audit` dans la CI, qui bloque le pipeline en cas de vulnérabilité de niveau high ou critical ;
- `npm audit signatures` pour vérifier l'origine des paquets ;
- Dependabot pour les mises à jour ;
- Dependency Review pour bloquer les pull requests qui ajoutent une dépendance vulnérable.

Bonnes pratiques complémentaires :

- toujours installer avec `npm ci` en CI et en production ;
- vérifier le nom exact d'un paquet avant de l'installer (des paquets malveillants imitent des noms connus) ;
- supprimer les dépendances inutilisées.

## 8.9 Mises à jour du système

Installer automatiquement les correctifs de sécurité du système :

```bash
sudo apt install -y unattended-upgrades
sudo dpkg-reconfigure -plow unattended-upgrades
```

## 8.10 Logs

Les logs ne doivent pas contenir :

- tokens ;
- mots de passe ;
- données personnelles sensibles ;
- clés API.

Attention aux `console.log(req.body)` ou `console.log(req.headers)` oubliés : ils écrivent mots de passe et tokens dans les fichiers de logs.

## 8.11 Sauvegardes

La base de données doit être sauvegardée selon une politique adaptée (par exemple : une sauvegarde par jour, conservée 14 jours).

Exemple PostgreSQL :

```bash
pg_dump -Fc myapp > /var/www/myapp/backups/myapp-$(date +%F).dump
```

Exemple MySQL :

```bash
mysqldump --single-transaction myapp > /var/www/myapp/backups/myapp-$(date +%F).sql
```

Ces commandes doivent être lancées par un utilisateur autorisé à lire la base (identifiants fournis par un fichier `~/.pgpass` ou `~/.my.cnf` protégé, jamais en clair dans la commande).

Règles :

- automatiser la sauvegarde (tâche `cron` ou timer systemd) ;
- copier les sauvegardes **hors du serveur** : si le serveur est perdu, des sauvegardes stockées dessus sont perdues aussi ;
- protéger les fichiers de sauvegarde (ils contiennent toutes les données) ;
- **tester la restauration** régulièrement : une sauvegarde jamais restaurée n'est pas une sauvegarde fiable.

Exemple de test de restauration PostgreSQL dans une base temporaire :

```bash
createdb myapp_restore_test
pg_restore -d myapp_restore_test /var/www/myapp/backups/myapp-2026-10-01.dump
```

## 8.12 Health checks

Le backend expose une route `/health` utilisée après chaque déploiement :

```js
app.get('/health', async (req, res) => {
  try {
    await db.query('SELECT 1'); // vérifie aussi la connexion à la base
    res.status(200).json({ status: 'ok' });
  } catch (err) {
    res.status(503).json({ status: 'error' });
  }
});
```

```text
Déploiement
   |
   v
GET /health
   |
   +--> 200 OK -> succès
   |
   +--> erreur -> rollback automatique + alerte
```

La route `/health` ne doit renvoyer aucune information sensible (version exacte des logiciels, chaîne de connexion...).
