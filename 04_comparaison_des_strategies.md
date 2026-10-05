# 4. Comparaison des deux stratégies

## 4.1 Deux questions indépendantes

Le choix d'architecture combine en réalité deux questions distinctes :

1. **Organisation du code** : un monorepo ou deux repositories ?
2. **Infrastructure** : un serveur ou plusieurs hébergeurs ?

Toutes les combinaisons existent :

| | Un serveur | Plusieurs hébergeurs |
|---|---|---|
| **Monorepo** | Stratégie 1 de ce cours | Possible : un workflow déploie le frontend chez A et le backend chez B |
| **Deux repositories** | Possible : deux pipelines vers le même VPS | Stratégie 2 avec deux hébergeurs |

Les stratégies 1 et 2 du cours sont donc deux cas typiques, pas les seules options.

Le type d'hébergement (VPS, plateforme comme Render, hébergement mutualisé) est un troisième choix, présenté au chapitre 16. Les deux stratégies fonctionnent avec chacun d'eux.

## 4.2 Tableau comparatif

| Critère | Monorepo + 1 serveur | 2 repos + déploiements indépendants |
|---|---|---|
| Organisation du code | Centralisée | Séparée |
| CI/CD | Un pipeline commun | Un pipeline par repository |
| Couplage front/back | Plus fort | Plus faible |
| Déploiement du frontend | Souvent lié à celui du backend | Autonome |
| Changement touchant front et back | Une seule pull request | Deux pull requests à coordonner |
| Mise en place initiale | Plus simple | Plus de configuration |
| Équipe unique et réduite | Très adaptée | Adaptée |
| Plusieurs équipes | Moins naturelle | Plus naturelle |
| Scalabilité séparée | Limitée par le serveur commun | Plus facile |
| Coordination du contrat d'API | Faible à moyenne | Importante |
| CORS | Évitable avec un préfixe `/api` | Obligatoire |
| Coût initial | Généralement faible | Peut augmenter |
| Indépendance des releases | Faible | Forte |

## 4.3 Choix selon le contexte

### Projet simple ou équipe réduite

Le monorepo sur un serveur simplifie :

- la gestion ;
- le développement ;
- les tests ;
- le déploiement ;
- la documentation.

C'est le point de départ recommandé pour un premier projet.

### Projet avec plusieurs équipes ou besoins d'indépendance

Deux repositories répondent mieux à :

- la séparation des responsabilités ;
- l'autonomie des équipes ;
- des cycles de release différents ;
- une scalabilité indépendante.

Aucune architecture n'est universellement supérieure : le choix dépend du contexte technique et organisationnel. Il est aussi possible de commencer en monorepo puis de séparer plus tard.
