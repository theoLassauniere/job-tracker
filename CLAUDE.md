# Job Tracker

Application web de suivi de candidatures. Monorepo contenant le frontend et plusieurs microservices backend.

## Architecture

**Style : microservices**, organisés en **monorepo**. Communication **REST synchrone** entre services. Une **API Gateway** est le point d'entrée unique : le frontend ne parle qu'à la gateway, qui route vers les services.

```
job-tracker/
├── frontend/            # SPA React + TypeScript + Vite
├── backend/             # Projets Quarkus indépendants (un POM par service)
│   ├── gateway/         # API Gateway, point d'entrée unique — port 8080
│   ├── auth-service/    # Authentification / utilisateurs — port 8081
│   └── jobs-service/    # Cœur métier : candidatures/offres — port 8082
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
- **Quarkus 3.36.0** sur **Java 21**, un projet par microservice
- Build : **Maven** — **projets indépendants** (pas de parent POM ; chaque service a son propre `pom.xml` autonome)
- `groupId` / package racine : `io.github.theolassauniere.jobtracker.<service>`
- Base de données : **PostgreSQL** — **instance partagée**, base unique `jobtracker`, un **schéma dédié par service** (`auth`, `jobs`). Chaque service ne touche que son schéma. Schémas créés à l'init du conteneur Postgres (`docker/postgres/init/`).
- Persistance : Hibernate ORM avec Panache (`quarkus.hibernate-orm.database.default-schema` par service)
- Communication inter-services : **Quarkus REST Client** (REST synchrone)
- Healthcheck : `quarkus-smallrye-health` (endpoint `/q/health`) sur chaque service
- Config externalisée : toutes les valeurs runtime (port, datasource) passent par variables d'environnement avec valeurs par défaut dev

## Déploiement

L'application doit rester **déployable de bout en bout** :
- **Dockerfiles multi-stage optimisés** pour chaque composant :
  - Backend : build `maven:3.9.16-eclipse-temurin-21` → packaging **JVM fast-jar** (`quarkus-app`), runtime `ubi9/openjdk-21-runtime` (user 185, layers `lib`/`app`/`quarkus` pour le cache).
  - Frontend : build pnpm (`node:24-alpine`) → service statique via `nginx:alpine` (config SPA + proxy `/api/` vers la gateway).
- **docker-compose.yml** à la racine orchestre tout en local. Ports exposés sur l'hôte : **frontend `3000`**, **gateway `8080`**, **postgres `5432`**. Les services auth/jobs ne sont joignables qu'en interne (réseau compose), via la gateway.
- Ordre de démarrage géré par `depends_on` + healthchecks (postgres `pg_isready`, services `/q/health/ready`).
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
- `backend/` : squelette des 3 services Quarkus (gateway, auth-service, jobs-service) — `pom.xml`, `application.properties`, arborescence des packages. **Pas de code métier** (classes/entités à écrire).
- Docker : `Dockerfile` par composant + `docker-compose.yml` + init Postgres. **Non testés en build** (Docker pas installé sur la machine de dev — nécessite Docker Desktop).
