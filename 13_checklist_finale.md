# 13. Checklist finale

## GitHub

- [ ] Repository créé
- [ ] Branches `main` et `develop` protégées (pull request, CI verte et relecture obligatoires)
- [ ] Pull Requests utilisées pour tout changement
- [ ] `.gitignore` présent avant le premier commit
- [ ] `README.md` présent
- [ ] Documentation `docs/` présente
- [ ] Aucun secret commité
- [ ] Secret scanning et push protection activés
- [ ] Dependabot alerts activées et `.github/dependabot.yml` présent
- [ ] Environnements `staging` et `production` créés
- [ ] Approbation obligatoire sur l'environnement `production`

## Frontend React

- [ ] `npm ci` fonctionne
- [ ] lint sans erreur ni avertissement
- [ ] `npm audit --audit-level=high` sans vulnérabilité bloquante
- [ ] tests fonctionnels
- [ ] build fonctionnel
- [ ] variables d'environnement documentées dans `.env.example`
- [ ] aucune variable `VITE_*` ne contient de secret
- [ ] `VITE_API_URL` défini pour chaque environnement
- [ ] build servi par Nginx

## Backend Express

- [ ] `npm ci` fonctionne
- [ ] lint sans erreur ni avertissement
- [ ] `npm audit --omit=dev --audit-level=high` sans vulnérabilité bloquante
- [ ] tests fonctionnels
- [ ] build fonctionnel si le projet en a un (TypeScript)
- [ ] endpoint `/health` qui vérifie aussi la base
- [ ] Express écoute sur `127.0.0.1`
- [ ] `trust proxy` configuré
- [ ] CORS limité à l'origine du frontend (ou préfixe `/api` sur un seul domaine)
- [ ] `helmet` et limitation de débit en place
- [ ] variables d'environnement dans `shared/backend.env` (droits 600)
- [ ] PM2 configuré (`ecosystem.config.js`, `pm2 startup`, `pm2 save`)
- [ ] rotation des logs (`pm2-logrotate`)
- [ ] aucun secret dans les logs

## Nginx

- [ ] domaine frontend configuré
- [ ] domaine API configuré
- [ ] HTTPS actif (Certbot)
- [ ] HTTP redirigé vers HTTPS
- [ ] renouvellement testé (`certbot renew --dry-run`)
- [ ] reverse proxy Express configuré
- [ ] `try_files $uri $uri/ /index.html` pour la SPA React
- [ ] `sudo nginx -t` sans erreur

## Serveur

- [ ] Node.js de la même version majeure que `.nvmrc`
- [ ] utilisateur `deploy` dédié, sans `sudo`
- [ ] SSH par clé uniquement
- [ ] connexion `root` et mots de passe désactivés
- [ ] firewall configuré (22, 80, 443 uniquement)
- [ ] base écoutant uniquement sur `127.0.0.1`
- [ ] utilisateur de base dédié à l'application
- [ ] mises à jour de sécurité automatiques
- [ ] sauvegardes automatiques et copiées hors du serveur
- [ ] monitoring de disponibilité et d'espace disque prévu

## CI/CD

- [ ] workflow CI sur chaque pull request
- [ ] lint
- [ ] audit des dépendances
- [ ] Dependency Review sur les pull requests
- [ ] tests
- [ ] build
- [ ] secrets GitHub configurés par environnement
- [ ] `SSH_KNOWN_HOSTS` configuré (pas de `StrictHostKeyChecking=no`)
- [ ] déploiement staging automatique
- [ ] déploiement production après approbation
- [ ] health check après déploiement
- [ ] rollback automatique en cas d'échec du health check

## Production

- [ ] staging validé
- [ ] release versionnée (date + SHA du commit)
- [ ] sauvegarde avant migration
- [ ] migration testée sur staging et compatible avec la version précédente
- [ ] health check après déploiement
- [ ] rollback manuel documenté et testé
- [ ] restauration d'une sauvegarde testée
- [ ] logs accessibles
- [ ] documentation à jour
