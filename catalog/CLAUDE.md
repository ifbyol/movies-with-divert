# CLAUDE.md — Catalog

Node.js/Express service that serves movie catalog data from MongoDB. Listens on port 8080.

## Start

```bash
yarn install
yarn start        # nodemon server.js — restarts on file changes
```

Debug mode (exposes Node.js inspector on port 9229):
```bash
yarn debug
```

Load seed data into MongoDB:
```bash
yarn load         # runs load.js
```

## Required environment variables

All must be set before starting:

| Variable | Description |
|----------|-------------|
| `MONGODB_HOST` | MongoDB hostname (e.g. `localhost` or `mongodb`) |
| `MONGODB_USERNAME` | MongoDB user |
| `MONGODB_PASSWORD` | MongoDB password |
| `MONGODB_DATABASE` | Database name |

In the Kubernetes/Okteto environment these are injected automatically. For local dev, export them manually or use a `.env` loader.

Example:
```bash
export MONGODB_HOST=localhost
export MONGODB_USERNAME=okteto
export MONGODB_PASSWORD=okteto
export MONGODB_DATABASE=okteto
yarn start
```

## Endpoints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/catalog` | List all movies |
| GET | `/catalog/healthz` | Health check — returns `{"status":"ok","namespace":"..."}` |

## Key files

| File | Purpose |
|------|---------|
| `server.js` | Express app and all route handlers |
| `load.js` | Seed script — loads movie data into MongoDB |
| `data/` | Seed data files |
| `Dockerfile` | Container image |
| `chart/` | Helm chart for Kubernetes deployment |

## Okteto dev environment

```bash
# From repo root
export OKTETO_SHARED_NAMESPACE="movies-shared"
okteto up -f okteto.catalog.yaml
```

This deploys the catalog service and a MongoDB instance in your namespace, then syncs local changes with hot reload via nodemon.

## Tech

- Node.js + Express 4
- MongoDB driver v6 (`mongodb` npm package)
- nodemon for development auto-restart
