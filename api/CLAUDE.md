# CLAUDE.md — API Gateway

Go reverse proxy / API gateway that aggregates data from the `catalog` and `rent` services. Listens on port 8080.

## Start

```bash
# Build
make build          # outputs bin/api  (OKTETO_NAME must be set, or override: make build BACKEND=api)
# or directly:
go build -o bin/api cmd/api/main.go

# Run
./bin/api
```

Debug (Delve remote debugger on port 2345):
```bash
make debug
```

## No required environment variables

The service hard-codes downstream URLs:
- `http://catalog:8080` — Catalog service
- `http://rent:8080` — Rent service

For local development these hostnames must resolve (e.g. via `/etc/hosts`, a local Docker network, or by changing them).

## Routes

| Method | Path | Proxies to |
|--------|------|-----------|
| GET | `/api/catalog` | `catalog:8080/catalog` |
| GET | `/api/catalog/healthz` | `catalog:8080/catalog/healthz` |
| POST | `/api/rent` | `rent:8080/rent` |
| POST | `/api/rent/return` | `rent:8080/rent/return` |
| GET | `/api/rent` | Fetches rentals from `rent` and enriches with catalog data |

The `BaggageMiddleware` reads and forwards the `baggage: okteto-divert=<namespace>` header so Divert routing propagates through all service calls.

## Key files

| File | Purpose |
|------|---------|
| `cmd/api/main.go` | Entry point — router setup |
| `handlers/gateway.go` | Reverse proxy helper |
| `handlers/rentals.go` | `/api/rent` GET — aggregates catalog + rental data |
| `middleware/` | Baggage header propagation and logging |
| `Makefile` | Build/start/debug targets |
| `chart/` | Helm chart for Kubernetes deployment |

## Okteto dev environment

```bash
# From repo root
export OKTETO_SHARED_NAMESPACE="movies-shared"
okteto up -f okteto.api.yaml
```

## Tech

- Go 1.24
- `gorilla/mux` v1.8.0 for routing
- `net/http/httputil.ReverseProxy` for proxying
