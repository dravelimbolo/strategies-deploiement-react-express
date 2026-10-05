# 5. `.gitignore` et `README.md`

## 5.1 `.gitignore`

`.gitignore` indique à Git quels fichiers ne doivent pas être versionnés.

Exemple pour un projet React + Express :

```gitignore
# Dépendances
node_modules/

# Secrets et configuration locale
.env
.env.*
!.env.example

# Fichiers générés
dist/
build/
coverage/

# Logs
*.log
npm-debug.log*

# Fichiers système
.DS_Store
Thumbs.db

# Éditeurs
.vscode/*
!.vscode/extensions.json
!.vscode/settings.json
.idea/

# Caches
.tmp/
.cache/
.eslintcache
```

La ligne `!.env.example` réintègre le fichier d'exemple, qui sinon serait ignoré par `.env.*`.

Pour `.vscode/`, on ignore les réglages personnels mais on peut versionner les recommandations d'extensions et les réglages partagés par l'équipe.

Point important : `.gitignore` n'agit que sur les fichiers **pas encore suivis** par Git. Un fichier déjà commité reste suivi. Pour l'arrêter :

```bash
git rm --cached .env
git commit -m "Ne plus suivre .env"
```

Si ce fichier contenait des secrets, ce n'est pas suffisant : voir la section 5.4.

## 5.2 Pourquoi ignorer `node_modules`

`node_modules` peut être très volumineux et dépend parfois du système d'exploitation.

Les dépendances sont restaurées avec :

```bash
npm ci
```

à partir de :

- `package.json` (les dépendances déclarées) ;
- `package-lock.json` (les versions exactes installées).

Le fichier `package-lock.json` **doit être versionné** : sans lui, `npm ci` échoue et les versions installées en CI ou en production peuvent différer de celles du développeur.

## 5.3 Pourquoi ignorer `.env`

Le fichier `.env` peut contenir :

```text
DATABASE_URL=...
JWT_SECRET=...
API_KEY=...
```

Ces valeurs sont des secrets. Il ne faut pas les commiter.

On fournit à la place un fichier `.env.example`, versionné, avec des valeurs vides ou fictives :

```env
PORT=3000
DATABASE_URL=
JWT_SECRET=
API_KEY=
CORS_ORIGIN=http://localhost:5173
```

Chaque développeur copie ce fichier en `.env` et le remplit localement.

## 5.4 Risque majeur : un secret poussé sur GitHub

Un secret envoyé dans Git reste présent dans l'historique même après suppression du fichier. Sur un repository public, il faut considérer qu'il a été vu : des robots parcourent GitHub en permanence pour récupérer les clés publiées.

Si cela arrive, dans cet ordre :

1. **Révoquer et régénérer immédiatement le secret** (nouveau mot de passe de base, nouvelle clé API, nouveau `JWT_SECRET`). C'est l'étape indispensable.
2. Retirer le fichier du suivi Git et l'ajouter au `.gitignore`.
3. Éventuellement réécrire l'historique (avec `git filter-repo`) : utile pour la propreté, mais jamais suffisant seul.

Prévention :

- configurer `.gitignore` **avant** le premier commit ;
- activer sur GitHub le **secret scanning** et la **push protection** (Settings > Code security), qui bloquent un push contenant un secret reconnu ;
- vérifier `git status` et `git diff --staged` avant chaque commit.

## 5.5 `README.md`

Le README est le point d'entrée documentaire du projet. C'est la page que GitHub affiche à l'ouverture du repository.

Il peut contenir :

- présentation ;
- prérequis ;
- installation ;
- commandes (`npm run dev`, `npm test`, `npm run lint`...) ;
- architecture ;
- configuration (renvoi vers `.env.example`) ;
- tests ;
- développement local ;
- déploiement ;
- liens vers une documentation plus détaillée.

## 5.6 Documentation modulaire

Éviter un README gigantesque. Il ne remplace pas les documents spécialisés.

Organisation possible :

```text
README.md
docs/
├── architecture.md
├── deployment.md
├── security.md
└── api.md
```

Le README renvoie vers ces documents.
