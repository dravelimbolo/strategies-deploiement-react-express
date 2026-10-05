# Contribuer au cours

Merci de votre intérêt pour ce cours. Les corrections (faute, commande erronée, lien cassé) et les propositions d'amélioration sont les bienvenues.

## Signaler un problème

Ouvrez une **issue** en précisant :

- le fichier et la section concernés (par exemple `07_deploiement_sans_docker.md`, section 7.6) ;
- ce qui est incorrect ou peu clair ;
- si possible, la correction proposée et une source (documentation officielle).

Pour une question de compréhension, précisez ce que vous avez essayé et le message d'erreur obtenu.

## Organisation des branches

Le dépôt suit une organisation inspirée de **Git Flow**, la même que celle enseignée au chapitre 10 :

| Branche | Rôle |
|---|---|
| `main` | Version publiée du cours. On n'y pousse jamais directement. |
| `develop` | Branche d'intégration : les contributions y sont fusionnées avant publication. |
| `feature/<sujet>` | Nouveau contenu ou amélioration, créée depuis `develop`. |
| `fix/<sujet>` | Correction (erreur, faute, lien), créée depuis `develop`. |
| `hotfix/<sujet>` | Correction urgente du contenu publié, créée depuis `main`, puis fusionnée dans `main` et `develop`. |
| `release/<version>` | Préparation d'une version publiée, créée depuis `develop` puis fusionnée dans `main`. |

Les noms de branches sont en minuscules, avec des tirets entre les mots : `feature/chapitre-docker`, `fix/commande-certbot`.

## Proposer une modification

Pour les contributeurs externes, le fonctionnement est celui du **GitHub Flow** : une branche courte, une pull request, une relecture, une fusion.

1. Forker le dépôt.
2. Créer une branche depuis `develop` :

```bash
git checkout develop
git pull
git checkout -b fix/commande-certbot
```

3. Faire les modifications et les commiter (voir les conventions ci-dessous).
4. Pousser la branche sur votre fork et ouvrir une **pull request vers `develop`** (jamais vers `main`).
5. Décrire dans la pull request ce qui change et pourquoi.

Le mainteneur relit, demande éventuellement des ajustements, puis fusionne. Le passage de `develop` vers `main` est fait par le mainteneur lors de la publication d'une nouvelle version.

## Messages de commit

Les messages suivent la convention **Conventional Commits**, en français :

```text
<type>: <description courte à l'infinitif>
```

| Type | Usage |
|---|---|
| `docs` | Ajout ou modification de contenu du cours |
| `fix` | Correction d'une erreur (commande, configuration, explication) |
| `chore` | Maintenance du dépôt (licence, `.gitignore`, organisation) |
| `ci` | Workflows GitHub Actions du dépôt |

Exemples :

```text
docs: ajouter la configuration CORS au chapitre 2
fix: corriger la commande pm2 startup
chore: mettre à jour le .gitignore
```

Un commit correspond à une modification cohérente. Éviter les commits du type « modifs » ou « wip ».

## Règles de rédaction

- Écrire en français, avec un vocabulaire accessible aux débutants. Tout nouveau terme technique est ajouté au glossaire (chapitre 14).
- Pas d'emoji.
- Pas de tiret cadratin ni de tiret utilisé comme ponctuation : utiliser les deux points, une virgule ou une nouvelle phrase.
- Les commandes et fichiers doivent être **exacts et testés** : les apprenants les copient tels quels.
- Indiquer le langage de chaque bloc de code (`bash`, `yaml`, `nginx`, `js`, `text`...).
- Utiliser les conventions du cours : `example.com`, `myapp`, utilisateur `deploy`, arborescence serveur du chapitre 9.
- Garder la cohérence entre les chapitres : si une modification touche une notion présente ailleurs, mettre à jour les autres chapitres et la checklist (chapitre 13).
- Ne jamais inclure de vrai secret, mot de passe, adresse IP ou nom de domaine personnel dans un exemple.

## Licence

En contribuant, vous acceptez que votre contribution soit publiée sous la licence du dépôt : [CC BY 4.0](LICENSE).
