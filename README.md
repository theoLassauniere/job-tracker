# Job Tracker

Application web de suivi de candidatures.
Monorepo : **frontend** React/Vite + **backend** microservices Quarkus, derrière une API Gateway.

```
frontend/   SPA React 19 + TypeScript + Vite + Tailwind
backend/
  gateway/        point d'entrée unique (8080)
  auth-service/   authentification / utilisateurs (8081)
  jobs-service/   candidatures / offres (8082)
docker-compose.yml   orchestration locale (services + PostgreSQL)
```

## Prérequis

| Outil | Version | Usage |
|-------|---------|-------|
| Docker Desktop | ≥ 24 | Lancer toute la stack |
| Node.js | 24 (corepack) | Dev frontend |
| pnpm | 11.x | Géré par corepack (voir ci-dessous) |
| JDK | 21 (ou +) | Dev backend |
| Maven | 3.9.x | Build backend |

---

## Option A — Tout lancer avec Docker (recommandé)

À la racine du projet :

```bash
docker compose up --build
```

Au premier lancement, le build prend quelques minutes (compilation Maven + build frontend + téléchargement des images).

Une fois démarré :

| Service | URL |
|---------|-----|
| Frontend | http://localhost:3000 |
| Gateway (API) | http://localhost:8080 |
| Health gateway | http://localhost:8080/q/health |
| PostgreSQL | `localhost:5432` (db/user/pwd : `jobtracker`) |

Arrêter la stack :

```bash
docker compose down          # stoppe et supprime les conteneurs
docker compose down -v       # idem + supprime le volume PostgreSQL (reset des données)
```

> Les services `auth-service` et `jobs-service` ne sont **pas** exposés sur l'hôte :
> ils ne sont joignables qu'en interne, via la gateway.

---

## Option B — Développement local (hot reload)

Pratique pour coder avec rechargement à chaud, sans rebuild Docker.

### 1. Base de données

Lancer uniquement PostgreSQL (les schémas `auth` et `jobs` sont créés automatiquement) :

```bash
docker compose up -d postgres
```

### 2. Backend (un terminal par service)

```bash
cd backend/jobs-service     # ou gateway / auth-service
mvn quarkus:dev
```

Quarkus démarre en mode dev (live reload, Dev UI sur `/q/dev`).
Les services lisent leur config DB depuis les variables d'env, avec des **valeurs par défaut dev** pointant déjà sur `localhost:5432`.

### 3. Frontend

```bash
cd frontend
corepack enable          # active pnpm via corepack (une seule fois)
pnpm install
pnpm dev
```

Frontend dispo sur l'URL affichée par Vite (par défaut http://localhost:5173).

---

## Configuration

Toute la config runtime passe par variables d'environnement (valeurs par défaut pour le dev) :

| Variable | Par défaut | Description |
|----------|------------|-------------|
| `HTTP_PORT` | 8080 / 8081 / 8082 | Port HTTP du service |
| `DB_URL` | `jdbc:postgresql://localhost:5432/jobtracker` | URL JDBC PostgreSQL |
| `DB_USERNAME` / `DB_PASSWORD` | `jobtracker` | Identifiants DB |
| `DB_GENERATION` | `update` | Stratégie Hibernate (`update`, `validate`, `none`…) |

En Docker, ces valeurs sont surchargées par `docker-compose.yml`.

---

## Conventions

- Commits **atomiques** suivant les [Conventional Commits](https://www.conventionalcommits.org/) (`feat:`, `fix:`, `chore:`…).
- Voir `CLAUDE.md` pour les détails d'architecture et les choix techniques.
