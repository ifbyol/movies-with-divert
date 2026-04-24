# CLAUDE.md — Rent Service

Java/Spring Boot 3 REST API that manages movie rentals. Persists data in PostgreSQL and publishes rental events to Kafka. Listens on port 8080.

## Start

```bash
mvn spring-boot:run
```

Build a JAR and run it:
```bash
mvn package -DskipTests
java -jar target/*.jar
```

## Required environment variables

Defaults match the Kubernetes deployment. Override for local dev:

| Variable | Default | Description |
|----------|---------|-------------|
| `DB_HOST` | `postgresql` | PostgreSQL hostname |
| `DB_PORT` | `5432` | PostgreSQL port |
| `DB_NAME` | `rentals` | Database name |
| `DB_USERNAME` | `okteto` | Database user |
| `DB_PASSWORD` | `okteto` | Database password |
| `KAFKA_BOOTSTRAP_SERVERS` | `kafka:9092` | Kafka broker address |

Example for local dev:
```bash
export DB_HOST=localhost
export KAFKA_BOOTSTRAP_SERVERS=localhost:9092
mvn spring-boot:run
```

## Key files

| File | Purpose |
|------|---------|
| `src/main/resources/application.properties` | All config with env-var defaults |
| `src/main/java/` | Spring Boot source |
| `pom.xml` | Maven dependencies — Spring Data JPA, Spring Kafka, PostgreSQL driver |
| `Dockerfile` | Multi-stage Maven build |

## Dependencies

- **PostgreSQL** must be reachable at `DB_HOST:DB_PORT` — schema is managed by Hibernate (`ddl-auto=update`)
- **Kafka** must be reachable at `KAFKA_BOOTSTRAP_SERVERS` — the service publishes events on rental/return actions

## Okteto dev environment

```bash
# From repo root
export OKTETO_SHARED_NAMESPACE="movies-shared"
okteto up -f okteto.rentals.yaml
```

This deploys both `rent` and `worker` along with PostgreSQL and Kafka in your namespace.

## Tech

- Java, Spring Boot 3.3.2
- Spring Data JPA + Hibernate (PostgreSQL dialect)
- Spring Kafka
- Maven build
