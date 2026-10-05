<p align="center">
  <img src="assets/akieni-academy-logo.png" alt="Akieni Academy" width="160">
</p>

<h1 align="center">Stratégies de déploiement d'une application React + Express</h1>

<p align="center">
  <strong>Deux stratégies avec GitHub Actions, sans Docker</strong><br>
  Suivi Individuel Mentorat • Akieni Academy
</p>

| Mentor référent | Contact |
|---|---|
| Dravel-Ameguste IMBOLO | contact@dravelimbolo.com |

## Présentation

Ce cours explique comment mettre en production une application composée d'un frontend React et d'un backend Node.js / Express, avec une chaîne CI/CD GitHub Actions.

Deux stratégies de déploiement sont comparées :

1. **Monorepo et un serveur** : frontend et backend dans le même repository, déployés sur un seul serveur.
2. **Deux repositories et déploiements indépendants** : frontend et backend séparés, déployés sur un ou deux hébergeurs.

Le déploiement principal se fait sur un serveur Linux administré directement (Nginx, PM2, PostgreSQL ou MySQL), sans Docker. Des alternatives plus simples pour débuter (Render, hébergement mutualisé cPanel) sont présentées au chapitre 16.

## Public visé

Apprenants qui savent déjà développer une petite application React et une API Express en local, et qui découvrent la mise en production. Aucune expérience préalable du déploiement ou de l'administration de serveur n'est nécessaire.

## Prérequis

- bases de JavaScript, React et Express ;
- utilisation de Git en ligne de commande (commit, branche, push, pull request) ;
- un compte GitHub ;
- bases du terminal (se déplacer dans les dossiers, éditer un fichier).

Pour les travaux pratiques sur serveur : un VPS Linux (Ubuntu LTS par exemple) et un nom de domaine, ou à défaut une machine virtuelle locale. Pour débuter sans serveur, un compte Render gratuit suffit.

## Objectifs d'apprentissage

À la fin du cours, l'apprenant sait :

- choisir entre un monorepo et deux repositories, et entre un ou deux hébergeurs ;
- choisir un type d'hébergement adapté (PaaS, mutualisé, VPS) ;
- écrire un pipeline GitHub Actions qui installe, analyse (lint), audite les dépendances, teste et construit le projet ;
- préparer et sécuriser un serveur Linux pour héberger React et Express ;
- configurer Nginx (fichiers statiques, reverse proxy, HTTPS) et PM2 ;
- déployer par releases et revenir à une version précédente ;
- organiser staging et production ;
- garder les secrets hors du code.

## Plan du cours

Les chapitres marqués **Essentiel** sont à maîtriser en priorité. Les chapitres **Approfondissement** peuvent être lus dans un second temps.

| N° | Chapitre | Niveau |
|---|---|---|
| 01 | [Contexte et objectifs](01_contexte_et_objectifs.md) | Essentiel |
| 02 | [Stratégie 1 : monorepo et un serveur](02_strategie_1_monorepo_1_serveur.md) | Essentiel |
| 03 | [Stratégie 2 : deux repositories et déploiements indépendants](03_strategie_2_deux_repos_deploiements_independants.md) | Essentiel |
| 04 | [Comparaison des stratégies](04_comparaison_des_strategies.md) | Essentiel |
| 05 | [`.gitignore` et `README.md`](05_gitignore_et_readme.md) | Essentiel |
| 06 | [CI/CD avec GitHub Actions, lint et audit des dépendances](06_ci_cd_github_actions.md) | Essentiel |
| 07 | [Déploiement sans Docker : serveur, Nginx, PM2](07_deploiement_sans_docker.md) | Essentiel |
| 08 | [Sécurité en production](08_securite_production.md) | Essentiel |
| 09 | [Structures recommandées](09_structure_projet_recommandee.md) | Référence |
| 10 | [Staging, production et rollback](10_staging_production_rollback.md) | Approfondissement |
| 11 | [Points d'attention et bonnes pratiques](11_points_dattention_et_bonnes_pratiques.md) | Approfondissement |
| 12 | [Synthèse](12_synthese.md) | Essentiel |
| 13 | [Checklist finale](13_checklist_finale.md) | Référence |
| 14 | [Glossaire](14_glossaire.md) | Référence |
| 15 | [Travaux pratiques](15_travaux_pratiques.md) | Essentiel |
| 16 | [Autres types d'hébergement : Render et cPanel](16_autres_types_hebergement.md) | Essentiel pour débuter |

## Parcours de lecture conseillé

1. Lire les chapitres 01 à 04 pour comprendre les deux stratégies.
2. Lire le chapitre 16 et faire un premier déploiement simple sur Render, pour voir rapidement le résultat.
3. Lire les chapitres 05 à 08 en gardant un terminal ouvert : ils contiennent les commandes et fichiers à reproduire sur un VPS.
4. Lire les chapitres 09 à 11 avant le premier déploiement réel en production.
5. Réaliser les travaux pratiques du chapitre 15, puis valider avec la checklist du chapitre 13.

Le glossaire (chapitre 14) peut être consulté à tout moment : chaque terme technique du cours y est expliqué simplement.

## Conventions utilisées

- `example.com`, `app.example.com` et `api.example.com` sont des domaines d'exemple à remplacer par les vôtres.
- `myapp` est le nom d'exemple de l'application.
- `deploy` est l'utilisateur Linux dédié au déploiement.
- Les commandes précédées de `sudo` s'exécutent avec un compte administrateur du serveur, jamais dans GitHub Actions.
- Les versions d'outils citées (Node.js, actions GitHub) sont celles en vigueur à la rédaction : vérifiez toujours la dernière version stable.

## Contribuer

Les corrections et suggestions sont les bienvenues via une issue ou une pull request vers la branche `develop`. Les règles sont décrites dans [CONTRIBUTING.md](CONTRIBUTING.md).

## Licence

Copyright (c) 2026 Dravel-Ameguste IMBOLO.

Ce cours est publié sous licence [Creative Commons Attribution 4.0 International (CC BY 4.0)](LICENSE). Vous pouvez le partager et l'adapter, y compris à des fins commerciales, à condition de citer l'auteur et de fournir un lien vers la licence.
