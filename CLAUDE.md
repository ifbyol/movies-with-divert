# CLAUDE.md — Movies with Divert

## What This Repo Is

A microservices demo app showcasing **Okteto's Divert feature**: developers deploy only the service(s) they're working on while sharing a common staging environment for the rest of the stack. Traffic is routed via the `baggage: okteto-divert=<namespace>` HTTP header.

## Architecture

Five services, each in its own subdirectory:

| Service | Language | Framework | Port | Dependencies |
|---------|----------|-----------|------|--------------|
| `frontend/` | JavaScript | React 17 + Webpack | 80 (nginx) | — |
| `catalog/` | JavaScript | Express.js | 8080 | MongoDB |
| `api/` | Go | Gorilla Mux | 8080 | PostgreSQL |
| `rentals/rent/` | Java | Spring Boot 3.3.2 | 8080 | PostgreSQL, Kafka |
| `rentals/worker/` | Go | Sarama | — | Kafka, PostgreSQL |

Supporting infrastructure: MongoDB 7, PostgreSQL 16, Apache Kafka 3.7.0.

## Starting the Application

### Option A: Full stack via Okteto (recommended)

**Step 1 — Deploy the shared environment once:**
```bash
okteto preview deploy \
  --repository https://github.com/okteto-community/movies-with-divert \
  --label=okteto-shared \
  movies-shared
```

**Step 2 — Deploy only the service you're working on:**
```bash
export OKTETO_SHARED_NAMESPACE="movies-shared"

# Pick ONE of these depending on which service you're developing:
okteto up -f okteto.frontend.yaml
okteto up -f okteto.catalog.yaml
okteto up -f okteto.api.yaml
okteto up -f okteto.rentals.yaml
```

**Step 3 — Route traffic to your service:**

Add this header to your browser requests (use ModHeader extension) or curl:
```
baggage: okteto-divert=<your-namespace>
```

Without the header, requests go to the shared services. With the header, they route to your deployed copy.

Example curl:
```bash
# Your service
curl -H "baggage: okteto-divert=<your-namespace>" https://movies-movies-shared.demo.okteto.dev/api/catalog/healthz

# Shared service
curl https://movies-movies-shared.demo.okteto.dev/api/catalog/healthz
```

### Option B: Local development (per service)

These require the backing services (MongoDB, PostgreSQL, Kafka) to be reachable.

```bash
# Frontend
cd frontend && yarn install && yarn start   # http://localhost:8080

# Catalog
cd catalog && yarn install && yarn start    # http://localhost:8080

# API Gateway
cd api && go build -o bin/api cmd/api/main.go && ./bin/api

# Rent service
cd rentals/rent && mvn spring-boot:run

# Worker
cd rentals/worker && go build -o bin/worker cmd/worker/main.go && ./bin/worker
```

## Okteto Manifest Files

| File | Purpose |
|------|---------|
| `okteto.yaml` | Full stack deploy (all services) |
| `okteto.api.yaml` | API Gateway only (with Divert) |
| `okteto.catalog.yaml` | Catalog + MongoDB (with Divert) |
| `okteto.frontend.yaml` | Frontend only (with Divert) |
| `okteto.rentals.yaml` | Rent + Worker + Kafka + PostgreSQL (with Divert) |

## Key Environment Variables

| Variable | Default | Used by |
|----------|---------|---------|
| `OKTETO_SHARED_NAMESPACE` | — | Required for Divert; set before `okteto up` |
| `DB_HOST` | `postgresql` | rentals/rent |
| `DB_PORT` | `5432` | rentals/rent |
| `DB_NAME` | `rentals` | rentals/rent |
| `DB_USERNAME` | `okteto` | rentals/rent |
| `DB_PASSWORD` | `okteto` | rentals/rent |
| `KAFKA_BOOTSTRAP_SERVERS` | `kafka:9092` | rentals/rent |
| `KAFKA_ADDRESS` | `kafka:9092` | rentals/worker |
| `KUBERNETES_NAMESPACE` | `default` | rentals/worker |

## Tests

End-to-end tests use Playwright:
```bash
cd tests && yarn install && yarn test
```

Defined in `okteto.yaml` under the `test` section — runs against the deployed environment.

## Helm Charts

Each service has its own Helm chart under `<service>/chart/`. Deployed via `helm upgrade --install` in the Okteto manifests. Image tags are injected at deploy time via `${OKTETO_BUILD_*_IMAGE}` variables.

## Divert Routing Details

- Driver: **Nginx + Linkerd** (default). To use Istio instead, add `driver: istio` to the divert block in the relevant okteto manifest.
- The `baggage` header is automatically propagated through all inter-service calls once set at the ingress.
- Shared namespace URL pattern: `movies-<shared-namespace>.demo.okteto.dev`

## Troubleshooting

See `TROUBLESHOOTING.md` for common issues and kubectl debug commands.
