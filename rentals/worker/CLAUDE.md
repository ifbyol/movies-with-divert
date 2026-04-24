# CLAUDE.md — Rentals Worker

Go Kafka consumer that processes rental events and writes results to PostgreSQL. No HTTP server — runs as a background worker.

## Start

```bash
# Build
make build          # outputs bin/worker  (OKTETO_NAME must be set, or: make build BACKEND=worker)
# or directly:
go build -o bin/worker cmd/worker/main.go

# Run
./bin/worker
```

Debug (Delve remote debugger on port 2345):
```bash
make debug
```

## Required environment variables

| Variable | Default | Description |
|----------|---------|-------------|
| `KAFKA_ADDRESS` | `kafka:9092` | Kafka broker address |
| `KUBERNETES_NAMESPACE` | `default` | Namespace — used to name the Kafka consumer group |
| `OKTETO_DIVERTED_ENVIRONMENT` | *(empty)* | Set by Okteto Divert; controls which Kafka consumer group to join |

PostgreSQL connection is hard-coded in `pkg/database/database.go`:
- host: `postgresql`, port: `5432`, user: `okteto`, password: `okteto`, dbname: `rentals`

For local dev, point `postgresql` to `localhost` via `/etc/hosts` or adjust the constants in `pkg/database/database.go`.

Example:
```bash
export KAFKA_ADDRESS=localhost:9092
export KUBERNETES_NAMESPACE=my-namespace
./bin/worker
```

## What it does on startup

1. Opens a PostgreSQL connection and waits for it to be ready (`Ping`)
2. Runs `LoadData` — drops and recreates the `rentals` table
3. Joins a Kafka consumer group (group name is derived from `KUBERNETES_NAMESPACE` + `OKTETO_DIVERTED_ENVIRONMENT`)
4. Consumes messages and writes rental records to PostgreSQL
5. Handles OS interrupt for graceful shutdown

## Key files

| File | Purpose |
|------|---------|
| `cmd/worker/main.go` | Entry point — wires together Kafka consumer and DB |
| `pkg/database/database.go` | PostgreSQL connection and schema init (hard-coded credentials) |
| `pkg/kafka/` | Sarama consumer group implementation |
| `Makefile` | Build/start/debug targets |

## Dependencies

- **PostgreSQL** must be reachable at `postgresql:5432` (hard-coded)
- **Kafka** must be reachable at `KAFKA_ADDRESS`

## Okteto dev environment

```bash
# From repo root — deploys rent + worker + PostgreSQL + Kafka together
export OKTETO_SHARED_NAMESPACE="movies-shared"
okteto up -f okteto.rentals.yaml
```

## Tech

- Go 1.24
- `Shopify/sarama` v1.32.0 (Kafka client)
- `lib/pq` (PostgreSQL driver)
- `kingpin` for CLI argument parsing
