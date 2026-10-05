# 15. Travaux pratiques

Les TP suivent l'ordre du cours. Chacun produit un résultat vérifiable. Réaliser d'abord tous les TP sur un environnement de staging (un VPS de test ou une machine virtuelle locale).

## TP 0 : premier déploiement sur Render (sans serveur)

Objectif : voir une application en ligne avant d'aborder le VPS.

1. Créer un petit projet React (Vite) et une API Express avec une route `/health` et une route `/api/hello`.
2. Pousser le projet sur GitHub.
3. Créer sur Render un Static Site pour le frontend et un Web Service pour le backend (chapitre 16 section 16.2).
4. Configurer `VITE_API_URL` et `CORS_ORIGIN`.
5. Ajouter la règle de réécriture `/*` vers `/index.html`.

Vérification : la page React affiche la réponse de `/api/hello`, et recharger une page autre que l'accueil ne donne pas d'erreur 404.

À retenir : noter tout ce que Render a fait automatiquement (HTTPS, relance du processus, reverse proxy). Ce sont exactement les tâches à réaliser soi-même sur un VPS dans les TP suivants.

## TP 1 : préparer le monorepo

Objectif : un monorepo propre, sans secret.

1. Créer un repository GitHub et le cloner.
2. Créer `frontend/` avec `npm create vite@latest frontend -- --template react`.
3. Créer `backend/` avec Express, en séparant `src/app.js` et `src/server.js`.
4. Ajouter `.gitignore` (chapitre 5), `.nvmrc`, `.env.example` dans chaque projet et un `README.md`.
5. Ajouter une route `GET /health` et une route `GET /api/hello`.

Vérification : `git status` ne montre ni `node_modules/` ni `.env`.

## TP 2 : lint et tests en local

Objectif : chaque projet passe le lint et les tests.

1. Ajouter les scripts `lint` et `lint:fix` (chapitre 6 section 6.4) dans les deux `package.json`.
2. Ajouter Vitest au frontend et un test simple d'un composant.
3. Ajouter Jest (ou `node --test`) et `supertest` au backend, avec un test de `/health`.
4. Introduire volontairement une variable inutilisée, constater l'échec du lint, puis corriger.

Vérification : `npm run lint` et `npm test` réussissent dans les deux dossiers.

## TP 3 : audit des dépendances

Objectif : savoir lire et traiter un rapport d'audit.

1. Lancer `npm audit` dans chaque projet et lire le rapport.
2. Lancer `npm audit --omit=dev --audit-level=high` dans le backend.
3. Lancer `npm audit signatures`.
4. Dans une branche de test, installer une ancienne version vulnérable d'un paquet connu, relancer l'audit, puis corriger avec `npm audit fix`.
5. Ajouter `.github/dependabot.yml` (chapitre 6 section 6.5).

Vérification : expliquer à l'oral la différence entre une dépendance directe et une dépendance indirecte, et le rôle de `--omit=dev`.

## TP 4 : pipeline CI

Objectif : la CI bloque une pull request défaillante.

1. Ajouter `.github/workflows/ci.yml` (chapitre 6 section 6.6).
2. Protéger la branche `develop` en exigeant que la CI soit verte.
3. Ouvrir une pull request qui casse un test : constater qu'elle ne peut pas être fusionnée.
4. Corriger et fusionner.

Vérification : historique des exécutions visible dans l'onglet Actions, une exécution rouge puis une verte.

## TP 5 : préparer et sécuriser le serveur

Objectif : un serveur prêt à recevoir l'application.

1. Créer un compte administrateur personnel avec clé SSH.
2. Suivre le chapitre 7 sections 7.3 et 7.4 (Node.js, PM2, utilisateur `deploy`, arborescence, `backend.env`).
3. Suivre le chapitre 8 : firewall, désactivation de `root` et des mots de passe SSH, mises à jour automatiques.
4. Installer PostgreSQL ou MySQL, créer l'utilisateur et la base de l'application.

Vérification : `sudo ss -tlnp` montre la base sur `127.0.0.1` uniquement ; `sudo ufw status` montre seulement 22, 80 et 443.

## TP 6 : Nginx, HTTPS et premier déploiement

Objectif : application accessible en HTTPS.

1. Configurer les deux fichiers Nginx (chapitre 7 section 7.5).
2. Générer les certificats avec Certbot (chapitre 8 section 8.5).
3. Créer les environnements GitHub, la clé de déploiement et les secrets (chapitre 6 sections 6.8 à 6.10).
4. Ajouter `deploy.yml`, `ecosystem.config.js` et `scripts/activate-release.sh`.
5. Pousser sur `develop` et suivre le déploiement.
6. Exécuter `pm2 startup` puis `pm2 save`.

Vérification : l'application s'affiche en HTTPS, recharger une page autre que l'accueil ne donne pas d'erreur 404, et l'application est toujours disponible après un redémarrage du serveur (`sudo reboot`).

## TP 7 : rollback

Objectif : savoir revenir en arrière.

1. Déployer une version dont `/health` renvoie une erreur 503 : constater le rollback automatique et le job rouge dans GitHub Actions.
2. Déployer une version avec un bug visible mais un `/health` correct.
3. Effectuer le rollback manuel (chapitre 10 section 10.7).

Vérification : `readlink -f /var/www/myapp/backend/current` pointe vers la release attendue après chaque étape.

## TP 8 : sauvegarde et restauration

Objectif : une sauvegarde réellement utilisable.

1. Créer une sauvegarde de la base (chapitre 8 section 8.11).
2. La restaurer dans une base temporaire.
3. Vérifier que les données sont présentes.

Vérification : le nombre de lignes d'une table est identique dans la base d'origine et dans la base restaurée.

## Validation finale

Parcourir la checklist du chapitre 13 et cocher chaque point, avec une preuve (capture, commande, lien vers une exécution GitHub Actions).
