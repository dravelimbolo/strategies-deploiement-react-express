# 10. Staging, production et rollback

## 10.1 Flux Git

```text
feature/*
    |
    v
Pull Request vers develop  (CI obligatoire + relecture)
    |
    v
develop
    |
    v
Déploiement automatique en STAGING
    |
    v
Validation fonctionnelle sur staging
    |
    v
Pull Request develop vers main  (CI obligatoire + relecture)
    |
    v
main
    |
    v
Approbation manuelle dans GitHub
    |
    v
Déploiement en PRODUCTION
```

Correctif urgent en production :

```text
hotfix/* (créée depuis main)
    |
    v
Pull Request vers main -> PRODUCTION
    |
    v
Fusion de main dans develop (pour ne pas perdre le correctif)
```

Ce modèle est simple à enseigner. Beaucoup d'équipes utilisent aussi le « trunk-based development » : une seule branche principale, des branches de courte durée, et des déploiements très fréquents. Le principe reste le même : rien n'arrive en production sans passer par la CI.

## 10.2 Protéger les branches

Dans Settings > Rules (ou Branches), pour `main` et `develop` :

- interdire le push direct : passage obligatoire par une pull request ;
- exiger que la CI soit verte (status checks obligatoires) ;
- exiger au moins une relecture ;
- interdire le force push et la suppression de la branche.

## 10.3 Pourquoi staging

Staging est une copie de la production, sur un serveur séparé, avec ses propres données et secrets. Il permet de vérifier :

- le build ;
- les variables d'environnement ;
- les migrations de base de données ;
- la communication frontend/backend et la configuration CORS ;
- les routes ;
- les erreurs qui n'apparaissent qu'en production ;
- le comportement Nginx/PM2.

Staging n'utilise jamais la base de données de production.

## 10.4 Releases

Exemple côté backend :

```text
/var/www/myapp/backend/releases/
├── 20261001-120000-a1b2c3d/
├── 20261002-090000-e4f5a6b/
└── 20261003-153000-c7d8e9f/
```

Le lien actif :

```text
current -> releases/20261003-153000-c7d8e9f
```

Pour revenir à la version précédente :

```text
current -> releases/20261002-090000-e4f5a6b
```

## 10.5 Avantage du lien symbolique

On ne modifie jamais une version en service. Le déploiement devient :

```text
Envoi
  |
  v
Installation
  |
  v
Bascule de current
  |
  v
Rechargement
  |
  v
Health check
```

Et le retour arrière se résume à rebasculer `current`.

## 10.6 Rollback automatique

Le script `activate-release.sh` (chapitre 7) revient automatiquement à la release précédente si `/health` ne répond pas après la bascule.

## 10.7 Rollback manuel

Si un problème est découvert plus tard (un bug fonctionnel par exemple), en tant que `deploy` sur le serveur.

Backend :

```bash
cd /var/www/myapp/backend
ls -1 releases | sort -r                       # lister les releases, la plus récente en premier
readlink -f current                            # voir la release active

ln -sfn /var/www/myapp/backend/releases/20261002-090000-e4f5a6b current.tmp
mv -Tf current.tmp current
APP_DIR=/var/www/myapp pm2 startOrReload current/ecosystem.config.js --update-env
curl -fsS http://127.0.0.1:3000/health
```

Frontend :

```bash
cd /var/www/myapp/frontend
ln -sfn /var/www/myapp/frontend/releases/20261002-090000-e4f5a6b current.tmp
mv -Tf current.tmp current
```

Aucun rechargement de Nginx n'est nécessaire pour le frontend.

Une autre solution consiste à annuler le commit fautif sur `main` avec `git revert` : le pipeline redéploie alors automatiquement une version corrigée. Cette méthode est plus lente, mais l'historique Git reste le reflet exact de la production.

Toujours documenter un rollback (date, cause, release restaurée) et corriger la cause avant de redéployer.

## 10.8 Limite : la base de données

Rebasculer `current` restaure le **code**, pas la **base de données**.

Si la nouvelle version a appliqué une migration (suppression d'une colonne par exemple), l'ancienne version du code peut ne plus fonctionner avec la base modifiée. Il faut donc :

- écrire des migrations compatibles avec la version précédente du code (voir chapitre 11 section 11.5) ;
- faire une sauvegarde avant toute migration risquée ;
- en dernier recours, restaurer la sauvegarde, en acceptant de perdre les données écrites depuis.

## 10.9 Blue/Green

Une autre approche :

```text
Nginx
   |
   +--> Blue  (version active, port 3000)
   |
   +--> Green (nouvelle version, port 3001)
```

La nouvelle version est démarrée à côté de l'ancienne, testée (health check, tests de fumée), puis Nginx bascule le trafic vers elle. L'ancienne version reste prête pour un retour immédiat.

Avantage par rapport aux releases avec un seul port : la nouvelle version est vérifiée **avant** de recevoir du trafic.

Cette méthode nécessite davantage de ressources et de configuration. Elle est à envisager une fois le déploiement par releases maîtrisé.
