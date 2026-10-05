# 14. Glossaire

**Artefact** : résultat d'un build (par exemple le dossier `dist/` du frontend) que l'on conserve pour le déployer.

**Audit des dépendances** : vérification automatique des paquets installés contre une base de vulnérabilités connues (`npm audit` pour Node.js, `pip-audit` pour Python).

**Blue/Green** : technique de déploiement où deux versions tournent en parallèle ; le trafic est basculé de l'une à l'autre.

**Build** : transformation du code source en fichiers prêts pour la production (pour React : HTML, CSS et JavaScript optimisés).

**CD** : Continuous Delivery (version prête à déployer, déploiement approuvé par une personne) ou Continuous Deployment (déploiement automatique).

**CI** : Continuous Integration, vérification automatique de chaque modification (lint, audit, tests, build).

**CORS** : Cross-Origin Resource Sharing, mécanisme par lequel un serveur autorise un site d'une autre origine à l'appeler depuis le navigateur.

**cPanel** : interface web d'administration d'un hébergement mutualisé (fichiers, bases de données, certificats, comptes FTP).

**Dependabot** : service GitHub qui ouvre des pull requests pour mettre à jour les dépendances.

**Environnement GitHub** : ensemble nommé (staging, production) regroupant secrets, variables et règles de protection pour les déploiements.

**FTPS** : transfert de fichiers par FTP avec chiffrement TLS. Le FTP simple, non chiffré, est à éviter.

**Health check** : vérification automatique qu'une application répond correctement, en général via une route `/health`.

**Hotfix** : correctif urgent appliqué directement à partir de la branche de production.

**Hébergement mutualisé** : serveur partagé entre de nombreux clients, géré par l'hébergeur, sans accès administrateur.

**Lint** : analyse statique du code pour détecter les erreurs probables et faire respecter des conventions (ESLint en JavaScript).

**Lockfile** : fichier qui fige les versions exactes des dépendances (`package-lock.json`).

**LTS** : Long Term Support, version de Node.js maintenue longtemps, recommandée en production.

**Migration** : script versionné qui modifie la structure de la base de données.

**Monorepo** : un seul repository contenant plusieurs projets (ici frontend et backend).

**Origine** : combinaison du protocole, du domaine et du port d'une URL (`https://app.example.com:443`).

**PaaS** : Platform as a Service, plateforme (Render par exemple) qui exécute votre code sans que vous administriez le serveur.

**Passenger** : gestionnaire d'applications utilisé par l'outil Setup Node.js App de cPanel ; il y joue le rôle de PM2.

**PM2** : gestionnaire de processus Node.js qui lance, surveille et relance l'application.

**Provisioning** : préparation d'un serveur (installation des logiciels, utilisateurs, configuration).

**Release** : version déployée, stockée dans son propre dossier sur le serveur.

**Reverse proxy** : serveur (ici Nginx) qui reçoit les requêtes publiques et les transmet à une application interne (ici Express).

**Rollback** : retour à une version précédente.

**rsync** : outil de copie de fichiers efficace, utilisé ici pour envoyer les releases via SSH.

**SPA** : Single Page Application, application web dont la navigation est gérée en JavaScript dans le navigateur (cas de React).

**Staging** : environnement de préproduction, copie de la production utilisée pour valider une version.

**Symlink (lien symbolique)** : fichier spécial qui pointe vers un autre fichier ou dossier (`current` vers la release active).

**TLS** : protocole de chiffrement utilisé par HTTPS.

**VPS** : Virtual Private Server, serveur virtuel loué chez un hébergeur, administré par l'utilisateur.
