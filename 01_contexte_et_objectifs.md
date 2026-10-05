# 1. Contexte et objectifs

## Stack

Le projet est basé sur :

```text
React (Vite)
  +
Node.js / Express
  +
GitHub
  +
GitHub Actions
  +
Nginx
  +
PM2
  +
PostgreSQL / MySQL
```

## Pourquoi sans Docker

Ce cours se concentre volontairement sur un déploiement **sans Docker**. Installer et configurer soi-même Node.js, Nginx, PM2 et la base de données permet de comprendre le rôle de chaque brique. Ces connaissances restent utiles ensuite, y compris avec Docker.

Si vous débutez complètement, vous pouvez commencer par un hébergement plus simple (Render, ou un hébergement mutualisé avec cPanel), présenté au chapitre 16, puis revenir au VPS.

## Objectifs

L'architecture doit permettre :

- de versionner proprement le code ;
- de vérifier automatiquement la qualité du code (lint) ;
- d'auditer les dépendances pour détecter les vulnérabilités connues ;
- de tester automatiquement le projet ;
- de construire le frontend React ;
- de tester le backend Express ;
- de déployer automatiquement ;
- de séparer les secrets du code ;
- de servir React efficacement ;
- d'exécuter Express de façon persistante ;
- de protéger la base de données ;
- de prévoir un environnement de staging ;
- de pouvoir revenir à une version précédente.

## Deux architectures étudiées

### Stratégie 1 : monorepo et un serveur

```text
GitHub
  |
  v
Monorepo
  |-- frontend/
  |-- backend/
  |
  v
GitHub Actions
  |
  v
Un serveur VPS
  |-- Nginx -> React (fichiers statiques)
  |-- Nginx -> PM2 -> Express
  |-- PostgreSQL/MySQL
```

### Stratégie 2 : deux repositories et déploiements indépendants

```text
GitHub
  |
  +--> Repository Frontend --> CI/CD --> Hébergement React
  |
  +--> Repository Backend  --> CI/CD --> Serveur API
                                            |
                                            v
                                         Database
```

Les deux stratégies sont valables. Leur pertinence dépend surtout de l'organisation de l'équipe, du niveau de séparation recherché et de l'infrastructure disponible. Le chapitre 4 les compare en détail.
