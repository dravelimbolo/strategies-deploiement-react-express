<div align="center">

[![Typing SVG](https://readme-typing-svg.demolab.com?font=Inter&weight=900&size=40&duration=2800&pause=1200&color=E8590C&center=true&vCenter=true&width=700&height=90&lines=Strat%C3%A9gies+de+d%C3%A9ploiement;React+%2B+Express+sans+Docker)](https://github.com/dravelimbolo/strategies-deploiement-react-express)

**`Cours · React · Express · GitHub Actions · Nginx · PM2 · CI/CD`**

_Deux stratégies pour mettre en production une application React + Express, expliquées pas à pas pour les débutants._

<br/>

[![Portfolio](https://img.shields.io/badge/-dravelimbolo.com-111111?style=for-the-badge&logo=safari&logoColor=white)](https://dravelimbolo.com)
[![LinkedIn](https://img.shields.io/badge/-LinkedIn-111111?style=for-the-badge&logo=linkedin&logoColor=white)](https://linkedin.com/in/dravel-imbolo)
[![GitHub](https://img.shields.io/badge/-GitHub-111111?style=for-the-badge&logo=github&logoColor=white)](https://github.com/dravelimbolo)
[![Email](https://img.shields.io/badge/-contact@dravelimbolo.com-111111?style=for-the-badge&logo=gmail&logoColor=white)](mailto:contact@dravelimbolo.com)

<br/>

[![Akieni Academy](https://img.shields.io/badge/Akieni_Academy-mentorat-E8590C.svg)](#mentorat)
[![Licence CC BY 4.0](https://img.shields.io/badge/licence-CC_BY_4.0-E8590C.svg)](LICENSE)
[![PRs bienvenues](https://img.shields.io/badge/PRs-bienvenues-E8590C.svg)](CONTRIBUTING.md)

</div>

---

<div align="center">

```
Monorepo + 1 serveur   ·   2 dépôts indépendants   ·   GitHub Actions   ·   sans Docker
```

_Versionner, tester, auditer, déployer, sécuriser et revenir en arrière, du premier push jusqu'à la production._

</div>

---

## Stack technique

<table align="center">
<tr>
  <td align="center">
    <strong>Frontend</strong><br/>
    <img src="https://skillicons.dev/icons?i=react,vite,js" />
  </td>
  <td align="center">
    <strong>Backend</strong><br/>
    <img src="https://skillicons.dev/icons?i=nodejs,express" />
  </td>
  <td align="center">
    <strong>Bases de données</strong><br/>
    <img src="https://skillicons.dev/icons?i=postgres,mysql" />
  </td>
  <td align="center">
    <strong>DevOps & Outils</strong><br/>
    <img src="https://skillicons.dev/icons?i=git,github,githubactions,nginx,linux,bash" />
  </td>
</tr>
</table>

---

## Les deux stratégies

```
Stratégie 1 :  monorepo  ──>  GitHub Actions  ──>  1 serveur (Nginx · PM2 · base de données)

Stratégie 2 :  dépôt frontend  ──>  GitHub Actions  ──>  hébergement React
               dépôt backend   ──>  GitHub Actions  ──>  serveur API  ──>  base de données
```

```
strategies-deploiement-react-express/
├── 01_contexte_et_objectifs.md                          # Stack, objectifs, les deux architectures
├── 02_strategie_1_monorepo_1_serveur.md                 # Monorepo + 1 serveur · CORS
├── 03_strategie_2_deux_repos_deploiements_independants.md  # 2 dépôts · contrat d'API
├── 04_comparaison_des_strategies.md                     # Tableau comparatif · critères de choix
├── 05_gitignore_et_readme.md                            # Ce qu'on versionne · secrets
├── 06_ci_cd_github_actions.md                           # Lint · npm audit · Dependabot · workflows
├── 07_deploiement_sans_docker.md                        # Serveur · Nginx · PM2 · releases
├── 08_securite_production.md                            # SSH · firewall · HTTPS · sauvegardes
├── 09_structure_projet_recommandee.md                   # Arborescences de référence
├── 10_staging_production_rollback.md                    # Git Flow · environnements · rollback
├── 11_points_dattention_et_bonnes_pratiques.md          # Migrations · erreurs fréquentes
├── 12_synthese.md                                       # Résumé du cours
├── 13_checklist_finale.md                               # Vérifications avant production
├── 14_glossaire.md                                      # Termes techniques expliqués
├── 15_travaux_pratiques.md                              # TP 0 à 8
├── 16_autres_types_hebergement.md                       # Render · hébergement mutualisé cPanel
├── assets/                                              # Logo Akieni Academy
├── CONTRIBUTING.md                                      # Guide de contribution
├── LICENSE                                              # Licence CC BY 4.0
└── README.md
```

### Ce que tu vas apprendre

| Notion | Mise en oeuvre dans le cours |
|---|---|
| **Choix d'architecture** | Monorepo ou deux dépôts, un ou deux hébergeurs, VPS, Render ou cPanel |
| **Intégration continue** | Workflow GitHub Actions : `npm ci` · lint ESLint · `npm audit` · tests · build |
| **Sécurité des dépendances** | `npm audit`, Dependabot, Dependency Review, secret scanning |
| **Déploiement sans Docker** | Nginx (statique + reverse proxy), PM2 en cluster, releases et lien `current` |
| **Sécurité en production** | Utilisateur `deploy`, SSH par clé, firewall, HTTPS Certbot, base non exposée |
| **Environnements** | `develop` vers staging, `main` vers production avec approbation |
| **Retour arrière** | Rollback automatique sur health check, rollback manuel, migrations compatibles |

---

## Par où commencer

> **Prérequis :** bases de JavaScript, React et Express · Git en ligne de commande · un compte GitHub

| Étape | Chapitres | Objectif |
|---|---|---|
| **1** | [01](01_contexte_et_objectifs.md) à [04](04_comparaison_des_strategies.md) | Comprendre les deux stratégies |
| **2** | [16](16_autres_types_hebergement.md) + TP 0 | Premier déploiement simple sur Render |
| **3** | [05](05_gitignore_et_readme.md) à [08](08_securite_production.md) | Pipeline CI/CD et déploiement sur un VPS |
| **4** | [09](09_structure_projet_recommandee.md) à [11](11_points_dattention_et_bonnes_pratiques.md) | Staging, rollback et bonnes pratiques |
| **5** | [15](15_travaux_pratiques.md) puis [13](13_checklist_finale.md) | Travaux pratiques et validation finale |

> Le [glossaire](14_glossaire.md) explique simplement chaque terme technique. La [synthèse](12_synthese.md) résume les règles à retenir.

### Conventions du cours

| Élément | Valeur d'exemple |
|---|---|
| Domaines | `app.example.com` · `api.example.com` |
| Application | `myapp` |
| Utilisateur de déploiement | `deploy` |
| Dossier sur le serveur | `/var/www/myapp` |
| Port Express | `3000` |

---

## Mentorat

<table align="center">
<tr>
  <td align="center">
    <img src="assets/akieni-academy-logo.png" alt="Akieni Academy" width="110" />
  </td>
  <td>
    <strong>Suivi Individuel Mentorat · Akieni Academy</strong><br/><br/>
    Mentor référent : <strong>Dravel-Ameguste IMBOLO</strong><br/>
    Contact : <a href="mailto:contact@dravelimbolo.com">contact@dravelimbolo.com</a>
  </td>
</tr>
</table>

---

## Contributions

<table style="border-spacing:0; font-size:13px; width:100%;">
<tr>
  <th style="padding:8px 12px;">Rôle</th>
  <th style="padding:8px 12px;">Personne</th>
  <th style="padding:8px 12px;">Contribution</th>
</tr>
<tr>
  <td style="padding:8px 12px;"><strong>Auteur & mainteneur</strong></td>
  <td style="padding:8px 12px;"><a href="https://github.com/dravelimbolo">Dravel IMBOLO</a></td>
  <td style="padding:8px 12px;">Conception du cours · rédaction · travaux pratiques · mentorat</td>
</tr>
</table>

Les contributions sont les bienvenues. Avant d'ouvrir une pull request, lis le **[guide de contribution](CONTRIBUTING.md)** : branches Git Flow, commits conventionnels et pull requests vers `develop`.

---

## Licence

Distribué sous licence **Creative Commons Attribution 4.0 International (CC BY 4.0)**. Voir le fichier [LICENSE](LICENSE).

Tu peux partager et adapter ce cours, à condition de citer l'auteur et de fournir un lien vers la licence.

Copyright (c) 2026 **Dravel IMBOLO**.

---

<div align="center">

<br/>

_"Transformons des idées en applications fonctionnelles, robustes et scalables."_

<br/>

</div>
