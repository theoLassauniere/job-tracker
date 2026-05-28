# Job Tracker

Application web de suivi de candidatures. Monorepo contenant le frontend et plusieurs microservices backend.

## Architecture

**Style : microservices**, organisés en **monorepo**. Communication **REST synchrone** entre services. Une **API Gateway** est le point d'entrée unique : le frontend ne parle qu'à la gateway, qui route vers les services.

```
job-tracker/
├── frontend/            # SPA React + TypeScript + Vite
├── backend/
│   ├── gateway/         # API Gateway (point d'entrée unique)
│   └── <service>/       # Microservices Quarkus (ex: auth-service, jobs-service)
├── docker-compose.yml   # Orchestration locale (services + PostgreSQL + gateway)
└── CLAUDE.md
```

## Stack technique

### Frontend (`/frontend`)
- **React 19** + **TypeScript** + **Vite 8**
- **Tailwind CSS v4** (via `@tailwindcss/vite`)
- Gestionnaire de paquets : **pnpm** (version pinnée via `packageManager` dans `package.json`, géré par corepack). Utiliser `pnpm install` / `pnpm <script>`.
- Scripts : `dev`, `build` (`tsc -b && vite build`), `lint`, `preview`

### Backend (`/backend`)
- **Quarkus** (un projet par microservice)
- Build : **Maven**
- Base de données : **PostgreSQL** (une base/schéma par service, pas de base partagée entre services)
- Persistance : Hibernate ORM avec Panache (à confirmer par le décisionnaire si autre choix)
- Communication inter-services : **Quarkus REST Client** (REST synchrone)

## Déploiement

L'application doit rester **déployable de bout en bout** :
- **Dockerfiles optimisés** pour chaque composant :
  - Backend : builds multi-stage Quarkus (privilégier `quarkus-app` / fast-jar, voire natif GraalVM si décidé), images de base minimales (ex: `ubi-minimal`).
  - Frontend : multi-stage (build Vite → service statique via nginx léger).
- **docker-compose.yml** à la racine pour orchestrer en local : tous les services, la gateway, et PostgreSQL.
- Garder en tête la déployabilité à **chaque** changement (variables d'environnement, ports, healthchecks, pas de valeurs en dur).

## Principes de scalabilité

- Services **stateless** (état en base / cache externe, jamais en mémoire process).
- Découplage fort entre services ; pas d'accès direct à la base d'un autre service.
- Configuration externalisée (variables d'environnement, profils Quarkus).
- Penser horizontal scaling (plusieurs instances d'un même service derrière la gateway).

## Conventions de travail

### Décisions
- **Le propriétaire du repo est le seul décisionnaire.** Pour toute décision d'implémentation non triviale (choix de lib, structure, schéma, dépendance, outillage), **poser la question avec des options** au lieu de trancher seul.

### Commits
- **Commits atomiques** : un commit = un changement cohérent et isolé.
- **Conventional Commits** : `feat:`, `fix:`, `chore:`, `refactor:`, `docs:`, `test:`, `build:`, `ci:`…
  - Optionnel : scope, ex `feat(jobs-service): ...`.
- Ne pas committer sans demande explicite.

## Statut actuel

- `frontend/` : app React/Vite/Tailwind scaffoldée (squelette).
- `backend/` : **pas encore créé** — à initialiser (gateway + premiers services).
- `docker-compose.yml` et Dockerfiles : **à créer**.
