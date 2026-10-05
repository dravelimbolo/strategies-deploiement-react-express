# 12. Synthèse

## 12.1 Sujet

Ce cours traite de la stratégie de déploiement et de la CI/CD d'une application React + Node.js/Express, sur un serveur Linux, sans Docker.

## 12.2 Notions abordées

1. stratégie de déploiement ;
2. intégration et déploiement continus ;
3. monorepo ou deux repositories ;
4. un hébergeur ou deux hébergeurs ;
5. communication frontend/backend et CORS ;
6. rôle de GitHub et de GitHub Actions ;
7. lint et audit de sécurité des dépendances ;
8. rôle de Nginx et de PM2 ;
9. préparation d'un serveur sans Docker ;
10. `.gitignore` et `README.md` ;
11. sécurité des secrets, du serveur et de la base ;
12. organisation du serveur en releases ;
13. staging et production ;
14. rollback et migrations de base de données.

## 12.3 Les deux architectures

Stratégie 1, monorepo et un serveur :

```text
GitHub
 |
Monorepo
 |-- frontend
 |-- backend
 |
GitHub Actions
 |
VPS
 |-- Nginx -> React (statique)
 |-- Nginx -> PM2 -> Express
 |-- Database
```

Stratégie 2, deux repositories :

```text
GitHub
 |
 +-- repository frontend -> CI/CD -> hébergement React
 |
 +-- repository backend  -> CI/CD -> serveur API -> Database
```

Le choix du nombre de repositories et celui du nombre d'hébergeurs sont deux décisions indépendantes (chapitre 4).

## 12.4 La chaîne CI/CD

```text
Développeur
   |
   v
Pull Request sur GitHub
   |
   v
GitHub Actions : install, lint, audit, test, build
   |
   v
Fusion dans develop -> staging   (automatique)
Fusion dans main    -> production (après approbation)
   |
   v
Serveur
   |
   +--> PM2 / Express (release + health check + rollback automatique)
   |
   +--> Nginx / React (release + bascule atomique)
```

## 12.5 Les règles à retenir

- Aucun secret dans Git, ni dans le frontend.
- `package-lock.json` versionné, `npm ci` partout, même version de Node partout.
- Rien n'est déployé sans lint, audit des dépendances et tests verts.
- Express et la base n'écoutent que sur `127.0.0.1` ; seuls les ports 22, 80 et 443 sont ouverts.
- HTTPS obligatoire.
- Déploiement par releases, rollback documenté et testé.
- Migrations compatibles avec la version précédente du code.
- Sauvegardes hors du serveur, restauration testée.

## 12.6 Conclusion

Les deux architectures peuvent être mises en production sans Docker.

Le choix dépend :

- de la taille du projet ;
- de l'organisation de l'équipe ;
- du besoin d'indépendance des releases ;
- de l'infrastructure disponible ;
- du budget ;
- du besoin de scalabilité ;
- de la gouvernance du code ;
- des exigences de sécurité.

Pour un premier projet, la stratégie 1 avec un préfixe `/api` sur un seul domaine est le point de départ le plus simple.
