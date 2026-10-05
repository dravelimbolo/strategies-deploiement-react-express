# 11. Points d'attention et bonnes pratiques

## 11.1 Ne pas confondre CI et déploiement

La CI valide le code. Le déploiement met le code en service.

Ils sont dans des workflows séparés (`ci.yml` et `deploy.yml`), mais le déploiement réutilise toujours la CI : on ne déploie jamais un code qui n'a pas passé le lint, l'audit et les tests.

## 11.2 Ne pas mettre les secrets dans Git

Utiliser :

- les secrets d'environnement GitHub pour l'accès au serveur ;
- un fichier protégé sur le serveur pour les secrets de l'application ;
- un gestionnaire de secrets si le projet grandit.

Activer le secret scanning et la push protection de GitHub.

## 11.3 Ne pas exposer la base

Seul le backend communique avec la base. La base écoute sur `127.0.0.1` et son port n'est pas ouvert dans le firewall.

## 11.4 Verrouiller les versions

- `package-lock.json` versionné ;
- `npm ci` (jamais `npm install`) en CI et sur le serveur ;
- `.nvmrc` partagé entre développeurs, CI et serveur.

## 11.5 Prévoir les migrations

Une migration de base de données peut être incompatible avec l'ancienne version de l'application, ce qui rend le rollback impossible (chapitre 10 section 10.8).

Ordre recommandé :

```text
Sauvegarde
 |
 v
Migration (compatible avec l'ancien code)
 |
 v
Déploiement du nouveau code
 |
 v
Health check
```

Règle « expand / contract » pour rester compatible :

1. **Expand** : ajouter sans casser (nouvelle colonne facultative, nouvelle table). L'ancien et le nouveau code fonctionnent tous les deux.
2. Déployer le code qui utilise la nouvelle structure.
3. **Contract** : supprimer l'ancienne colonne dans une release **ultérieure**, une fois sûr de ne plus revenir en arrière.

Exemple : pour renommer `name` en `full_name`, on ajoute `full_name`, on copie les données, on déploie le code qui utilise `full_name`, et on supprime `name` plus tard.

Utiliser un outil de migrations versionnées plutôt que des modifications manuelles : Prisma Migrate, Knex, Sequelize, TypeORM ou node-pg-migrate selon le projet.

## 11.6 Prévoir un rollback

Un déploiement professionnel a une procédure de retour arrière documentée **et testée** (chapitre 10).

## 11.7 Contrat d'API

Avec deux repositories, le contrat d'API devient une pièce centrale. OpenAPI, versionnement et tests d'intégration réduisent les incompatibilités (chapitre 3 section 3.6).

## 11.8 Permissions

- Le compte `deploy` n'a pas de droit `sudo`.
- Le jeton des workflows GitHub est limité à la lecture (`permissions: contents: read`).
- L'utilisateur de la base n'a des droits que sur la base de l'application.
- La production exige une approbation manuelle.

## 11.9 Qualité et dépendances

- Lint sans avertissement (`--max-warnings=0`).
- `npm audit` bloquant en CI pour les niveaux high et critical.
- Pull requests Dependabot traitées régulièrement, pas laissées en attente pendant des mois.
- Ne pas désactiver une règle de lint ou ignorer une alerte d'audit sans en écrire la raison.

## 11.10 Monitoring

Prévoir progressivement :

- logs centralisés et rotation des logs (`pm2-logrotate`) ;
- health checks ;
- surveillance de disponibilité externe (Uptime Kuma, UptimeRobot ou équivalent) qui appelle `/health` régulièrement ;
- alertes (mail, messagerie) ;
- métriques (CPU, mémoire, espace disque).

Un disque plein (logs, releases, sauvegardes) est une cause fréquente de panne : surveiller l'espace disque.

## 11.11 Documentation

Documenter au minimum, dans chaque projet :

```text
README.md
docs/architecture.md
docs/deployment.md
docs/security.md
docs/api.md
```

Le rôle de chaque fichier est décrit au chapitre 5.

## 11.12 Sans Docker

Sans Docker, il faut compenser l'absence d'environnement isolé par :

- une version de Node maîtrisée ;
- une configuration serveur documentée ;
- de l'automatisation ;
- une procédure de provisioning (script de préparation du serveur versionné) ;
- un contrôle des dépendances.

## 11.13 Erreurs fréquentes chez les débutants

| Symptôme | Cause probable |
|---|---|
| Erreur CORS dans la console du navigateur | `CORS_ORIGIN` absent ou différent de l'URL réelle du frontend (attention au `https` et au `/` final) |
| Erreur 404 en rechargeant une page React | `try_files ... /index.html` absent de la configuration Nginx |
| Le frontend appelle `localhost` en production | `VITE_API_URL` non défini au moment du build |
| `npm: command not found` pendant le déploiement | Node installé avec nvm sur le serveur au lieu de NodeSource |
| `npm ci` échoue | `package-lock.json` absent ou désynchronisé avec `package.json` |
| `npm ci --omit=dev` échoue sur un script `prepare` | Un outil de développement (husky par exemple) est appelé alors qu'il n'est pas installé en production |
| L'application ne redémarre pas après un reboot | `pm2 startup` et `pm2 save` non exécutés |
| `502 Bad Gateway` sur l'API | Express arrêté ou n'écoutant pas sur le port attendu : voir `pm2 logs` |
| Le site n'a plus de HTTPS après 3 mois | Renouvellement Certbot cassé : tester `certbot renew --dry-run` |
| `Host key verification failed` dans GitHub Actions | Secret `SSH_KNOWN_HOSTS` absent ou incorrect |
