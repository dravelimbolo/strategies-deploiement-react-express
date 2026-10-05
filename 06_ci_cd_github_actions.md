# 6. CI/CD avec GitHub Actions

## 6.1 CI

CI signifie Continuous Integration (intégration continue). À chaque pull request, le code est automatiquement vérifié :

```text
Code
 |
 v
Installation des dépendances (npm ci)
 |
 v
Lint (qualité du code)
 |
 v
Audit des dépendances (vulnérabilités connues)
 |
 v
Tests
 |
 v
Build
```

Si une étape échoue, le pipeline s'arrête et la pull request est signalée en rouge. La CI permet de détecter les erreurs avant qu'elles n'arrivent en production.

## 6.2 CD : Delivery ou Deployment

CD recouvre deux pratiques proches :

- **Continuous Delivery** (livraison continue) : chaque version validée est **prête** à partir en production, mais une personne approuve le déploiement.
- **Continuous Deployment** (déploiement continu) : chaque version validée part **automatiquement** en production, sans intervention humaine.

Dans ce cours, on applique :

- du déploiement continu vers **staging** (chaque push sur `develop`) ;
- de la livraison continue vers **production** (push sur `main`, puis approbation manuelle dans GitHub, voir section 6.9).

Pipeline :

```text
Git push
   |
   v
CI (lint, audit, tests, build)
   |
   +--> échec -> arrêt, rien n'est déployé
   |
   v
Déploiement
   |
   v
Health check
   |
   +--> échec -> rollback automatique
```

## 6.3 Construire une fois, le cas particulier de React

Le principe recommandé est « build once, deploy many » : on construit une fois, on teste ce résultat, et c'est exactement lui qu'on déploie.

- **Backend Express (JavaScript)** : il n'y a pas de build. On déploie le code source du commit testé, et les dépendances sont réinstallées sur le serveur avec `npm ci`, à partir du même `package-lock.json`. Le résultat est donc identique.
- **Frontend React** : les variables d'environnement Vite (`VITE_*`) sont **écrites dans le JavaScript au moment du build**. Un build fait avec l'URL de l'API de staging contient cette URL : il ne peut pas servir tel quel en production.

```js
// Dans le code React
const apiUrl = import.meta.env.VITE_API_URL;
```

Conséquence : le frontend est construit **une fois par environnement**, dans le job de déploiement, à partir du même commit validé par la CI. C'est l'approche retenue dans ce cours car elle est simple.

Alternative plus avancée : charger la configuration à l'exécution (un fichier `config.json` servi par Nginx et lu au démarrage de l'application), ce qui permet un build unique pour tous les environnements.

Rappel : toute variable `VITE_*` est visible par n'importe quel utilisateur dans le navigateur. Elle ne doit jamais contenir de secret (voir chapitre 8).

## 6.4 Lint

Le lint analyse le code sans l'exécuter pour détecter les erreurs probables (variable non définie, import inutilisé, règle des hooks React non respectée...) et imposer un style cohérent.

L'outil standard en JavaScript est **ESLint**.

- Le template Vite React fournit déjà un fichier `eslint.config.js`.
- Pour le backend, on l'initialise avec :

```bash
npm init @eslint/config@latest
```

Scripts à déclarer dans chaque `package.json` :

```json
{
  "scripts": {
    "lint": "eslint . --max-warnings=0",
    "lint:fix": "eslint . --fix"
  }
}
```

L'option `--max-warnings=0` fait échouer la CI dès qu'il reste un avertissement. Sans elle, les avertissements s'accumulent sans que personne ne les traite.

Optionnel : **Prettier** pour le formatage automatique, avec `prettier --check .` en CI.

## 6.5 Audit de sécurité des dépendances

Une application React + Express embarque souvent des centaines de paquets npm, dont la plupart sont des dépendances indirectes. Une vulnérabilité découverte dans l'un d'eux concerne directement votre application.

Équivalences avec d'autres écosystèmes :

| Écosystème | Outil d'audit |
|---|---|
| Python | `pip-audit` |
| Node.js / npm | `npm audit` |
| Multi-langages | OSV-Scanner |

### `npm audit`

`npm audit` compare `package-lock.json` à la base de vulnérabilités GitHub Advisory.

```bash
# Rapport complet
npm audit

# Échoue (code de sortie non nul) s'il existe une vulnérabilité de niveau high ou critical
npm audit --audit-level=high

# Uniquement les dépendances de production (utile pour le backend)
npm audit --omit=dev --audit-level=high

# Vérifie les signatures des paquets téléchargés depuis le registre npm
npm audit signatures
```

Corriger :

```bash
npm audit fix
```

Ne pas utiliser `npm audit fix --force` sans réfléchir : cette option peut installer des versions majeures incompatibles. Après une correction, relancer le lint, les tests et le build.

Si une vulnérabilité concerne une dépendance indirecte non encore corrigée par le paquet parent, on peut forcer une version corrigée avec le champ `overrides` de `package.json` :

```json
{
  "overrides": {
    "paquet-vulnerable": "^2.3.4"
  }
}
```

### Dependabot

Dependabot, intégré à GitHub, ouvre automatiquement des pull requests pour mettre à jour les dépendances vulnérables ou obsolètes. Fichier `.github/dependabot.yml` pour un monorepo :

```yaml
version: 2
updates:
  - package-ecosystem: npm
    directory: /frontend
    schedule:
      interval: weekly

  - package-ecosystem: npm
    directory: /backend
    schedule:
      interval: weekly

  - package-ecosystem: github-actions
    directory: /
    schedule:
      interval: weekly
```

Activer aussi les **Dependabot alerts** dans Settings > Code security.

Chaque pull request de Dependabot passe par la CI : elle n'est fusionnée que si le lint, les tests et le build restent verts.

### Dependency Review

L'action `actions/dependency-review-action` analyse, dans une pull request, les dépendances **ajoutées ou modifiées** et bloque la pull request si l'une d'elles est vulnérable. Elle est intégrée au workflow de la section 6.6.

### Pour aller plus loin

- **CodeQL** (GitHub code scanning) : analyse statique de sécurité du code JavaScript lui-même, gratuite pour les repositories publics.
- **OSV-Scanner** : audit multi-langages, utile si le projet contient aussi du Python ou d'autres écosystèmes (`osv-scanner scan source -r .`).

## 6.6 Workflow CI complet (monorepo)

Fichier `.github/workflows/ci.yml` :

```yaml
name: CI

on:
  pull_request:
  workflow_call:

permissions:
  contents: read

jobs:
  frontend:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: frontend
    steps:
      - uses: actions/checkout@v5

      - uses: actions/setup-node@v5
        with:
          node-version-file: .nvmrc
          cache: npm
          cache-dependency-path: frontend/package-lock.json

      - run: npm ci
      - run: npm run lint
      - run: npm audit --audit-level=high
      - run: npm test
      - run: npm run build

  backend:
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: backend
    steps:
      - uses: actions/checkout@v5

      - uses: actions/setup-node@v5
        with:
          node-version-file: .nvmrc
          cache: npm
          cache-dependency-path: backend/package-lock.json

      - run: npm ci
      - run: npm run lint
      - run: npm audit --omit=dev --audit-level=high
      - run: npm audit signatures
      - run: npm test

  dependency-review:
    if: github.event_name == 'pull_request'
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v5
      - uses: actions/dependency-review-action@v4
        with:
          fail-on-severity: high
```

Explications :

- `on: pull_request` : la CI tourne sur chaque pull request.
- `on: workflow_call` : ce workflow peut être réutilisé par le workflow de déploiement (section 6.7), ce qui évite de dupliquer les étapes.
- `permissions: contents: read` : le jeton GitHub du workflow ne peut que lire le code (principe du moindre privilège).
- `node-version-file: .nvmrc` : la CI utilise la même version de Node que les développeurs et le serveur (voir chapitre 7).
- `cache: npm` : accélère `npm ci` en réutilisant le cache npm entre deux exécutions.
- `npm test` doit lancer les tests une seule fois puis s'arrêter (par exemple `vitest run` côté frontend, `jest` ou `node --test` côté backend).

Les numéros de version des actions (`@v5`, `@v4`) évoluent : Dependabot (section 6.5) propose les mises à jour.

## 6.7 Workflow de déploiement

Fichier `.github/workflows/deploy.yml` :

```yaml
name: Deploy

on:
  push:
    branches: [develop, main]

permissions:
  contents: read

concurrency:
  group: deploy-${{ github.ref_name }}
  cancel-in-progress: false

jobs:
  ci:
    uses: ./.github/workflows/ci.yml

  deploy:
    needs: ci
    runs-on: ubuntu-latest
    environment: ${{ github.ref_name == 'main' && 'production' || 'staging' }}
    env:
      APP_DIR: ${{ vars.APP_DIR }}
      SSH_TARGET: ${{ secrets.SSH_USER }}@${{ secrets.SSH_HOST }}
    steps:
      - uses: actions/checkout@v5

      - uses: actions/setup-node@v5
        with:
          node-version-file: .nvmrc
          cache: npm
          cache-dependency-path: frontend/package-lock.json

      - name: Nommer la release
        run: echo "RELEASE=$(date -u +%Y%m%d-%H%M%S)-${GITHUB_SHA::7}" >> "$GITHUB_ENV"

      - name: Construire le frontend pour cet environnement
        working-directory: frontend
        env:
          VITE_API_URL: ${{ vars.VITE_API_URL }}
        run: |
          npm ci
          npm run build

      - name: Configurer la connexion SSH
        env:
          SSH_PRIVATE_KEY: ${{ secrets.SSH_PRIVATE_KEY }}
          SSH_KNOWN_HOSTS: ${{ secrets.SSH_KNOWN_HOSTS }}
        run: |
          install -m 700 -d ~/.ssh
          printf '%s\n' "$SSH_PRIVATE_KEY" > ~/.ssh/id_ed25519
          chmod 600 ~/.ssh/id_ed25519
          printf '%s\n' "$SSH_KNOWN_HOSTS" > ~/.ssh/known_hosts

      - name: Déployer le backend
        run: |
          rsync -az \
            --exclude node_modules --exclude .env --exclude tests \
            backend/ "$SSH_TARGET:$APP_DIR/backend/releases/$RELEASE/"
          ssh "$SSH_TARGET" \
            "bash '$APP_DIR/backend/releases/$RELEASE/scripts/activate-release.sh' '$APP_DIR' '$RELEASE'"

      - name: Déployer le frontend
        run: |
          rsync -az frontend/dist/ "$SSH_TARGET:$APP_DIR/frontend/releases/$RELEASE/"
          ssh "$SSH_TARGET" \
            "ln -sfn '$APP_DIR/frontend/releases/$RELEASE' '$APP_DIR/frontend/current.tmp' \
             && mv -Tf '$APP_DIR/frontend/current.tmp' '$APP_DIR/frontend/current'"

      - name: Vérifier depuis l'extérieur
        run: |
          curl -fsS --retry 5 --retry-delay 3 "${{ vars.PUBLIC_URL }}" > /dev/null
          curl -fsS --retry 5 --retry-delay 3 "${{ vars.VITE_API_URL }}/health" > /dev/null
```

Fonctionnement :

1. Un push sur `develop` déploie en staging, un push sur `main` déploie en production.
2. Le job `ci` réutilise `ci.yml` : rien n'est déployé si le lint, l'audit, les tests ou le build échouent.
3. `environment` choisit l'environnement GitHub (`staging` ou `production`) : chaque environnement a ses propres secrets et variables, avec les **mêmes noms**.
4. `concurrency` empêche deux déploiements simultanés sur le même environnement.
5. Le nom de release contient la date UTC et le début du SHA du commit : on sait toujours quel commit tourne.
6. Le backend est envoyé avec `rsync`, puis le script `activate-release.sh` (chapitre 7) installe les dépendances, bascule la version, recharge PM2, vérifie `/health` et revient automatiquement en arrière en cas d'échec.
7. Le frontend est envoyé, puis le lien `current` est basculé de façon atomique (section 7.8).
8. Une dernière vérification est faite depuis Internet, à travers Nginx et HTTPS.

Les secrets ne sont jamais écrits directement dans les commandes `run` : ils passent par des variables d'environnement (`env:`), ce qui évite qu'ils soient interprétés par le shell.

## 6.8 Secrets et variables GitHub

Les secrets sont stockés dans GitHub (Settings > Environments > nom de l'environnement) :

| Nom | Type | Contenu |
|---|---|---|
| `SSH_PRIVATE_KEY` | Secret | Clé privée de déploiement (section 6.10) |
| `SSH_KNOWN_HOSTS` | Secret | Empreinte du serveur (section 6.10) |
| `SSH_HOST` | Secret | Adresse IP ou nom du serveur |
| `SSH_USER` | Secret | `deploy` |
| `APP_DIR` | Variable | `/var/www/myapp` |
| `VITE_API_URL` | Variable | `https://api.example.com` (ou l'URL de staging) |
| `PUBLIC_URL` | Variable | `https://app.example.com` (ou l'URL de staging) |

Les secrets de l'application elle-même (mot de passe de la base, `JWT_SECRET`...) ne passent **pas** par GitHub : ils restent sur le serveur, dans `shared/backend.env` (chapitre 7).

Ne jamais écrire une clé privée ou un mot de passe directement dans un fichier de workflow.

## 6.9 Environnements GitHub et approbation de la production

Dans Settings > Environments, créer deux environnements :

- `staging` : pas de protection, le déploiement est automatique ;
- `production` :
  - **Required reviewers** : une ou plusieurs personnes doivent approuver chaque déploiement ;
  - **Deployment branches** : limiter à la branche `main`.

Ainsi, même si quelqu'un pousse par erreur sur `main`, la production n'est mise à jour qu'après une approbation explicite.

Les protections d'environnement sont gratuites pour les repositories publics. Pour les repositories privés, certaines nécessitent un plan GitHub payant.

## 6.10 Clé SSH de déploiement

Générer une clé **dédiée** au déploiement, sur votre machine (pas sur le serveur) :

```bash
ssh-keygen -t ed25519 -C "github-actions-deploy" -f deploy_key -N ""
```

Deux fichiers sont créés :

- `deploy_key.pub` (clé publique) : à ajouter à la fin de `/home/deploy/.ssh/authorized_keys` sur le serveur ;
- `deploy_key` (clé privée) : à copier dans le secret `SSH_PRIVATE_KEY`, puis à **supprimer** de votre machine.

Récupérer l'empreinte du serveur pour `SSH_KNOWN_HOSTS` :

```bash
ssh-keyscan -t ed25519 votre-serveur.example.com
```

Comparer l'empreinte obtenue avec celle affichée sur le serveur (`ssh-keygen -lf /etc/ssh/ssh_host_ed25519_key.pub`) avant de l'enregistrer.

Ne jamais désactiver la vérification avec `StrictHostKeyChecking=no` : cela permettrait à un serveur pirate de se faire passer pour le vôtre.

La clé de déploiement est utilisée avec l'utilisateur `deploy`, jamais avec `root` (voir chapitre 8).

## 6.11 Monorepo : filtrer par chemins

Dans un monorepo, on peut éviter de lancer la CI frontend quand seul le backend a changé, avec un filtre `paths` :

```yaml
on:
  pull_request:
    paths:
      - 'frontend/**'
      - '.nvmrc'
```

Cela implique de séparer `ci.yml` en `ci-frontend.yml` et `ci-backend.yml`.

Piège fréquent : si un workflow filtré est déclaré **obligatoire** dans la protection de branche et qu'il ne se lance pas (car aucun fichier concerné n'a changé), la pull request reste bloquée en attente. Pour un projet de petite taille, lancer toute la CI à chaque fois (comme en section 6.6) est plus simple.

## 6.12 Deux repositories

Chaque repository possède son propre pipeline :

```text
frontend repo -> ci.yml (job frontend) + deploy.yml (partie frontend)

backend repo  -> ci.yml (job backend)  + deploy.yml (partie backend)
```

Les workflows des sections 6.6 et 6.7 se découpent directement : chaque repository garde la partie qui le concerne, sans `working-directory`. Si le frontend est hébergé chez un fournisseur de sites statiques (Netlify, Vercel, Cloudflare Pages...), l'étape SSH est remplacée par l'outil de déploiement de ce fournisseur.

Cette séparation rend les releases indépendantes.
