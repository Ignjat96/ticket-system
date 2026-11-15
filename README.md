# Ticket System

Event ticket reservation backend: a learning project focused on senior Java topics
(concurrency, transactions, security, production readiness).

## Tech stack

- Java 21, Spring Boot 4.1
- PostgreSQL 18, Flyway
- Gradle (Kotlin DSL)
- Testcontainers
- springdoc-openapi (Swagger UI)

## Prerequisites

- JDK 21
- [Podman](https://podman.io/) with a compose provider (`podman compose`), or Docker

## Running locally

1. Start the database:

   ```shell
   podman compose up -d
   ```

2. Run the application:

   ```shell
   ./gradlew bootRun        # Linux/macOS
   .\gradlew bootRun        # Windows
   ```

3. Check that it works:

   - Health: http://localhost:8080/actuator/health → `{"status":"UP"}`
   - Swagger UI: http://localhost:8080/swagger-ui.html

4. Stop the database:

   ```shell
   podman compose down
   ```

> Spring Boot's Docker Compose support is disabled (`spring.docker.compose.enabled: false`)
> because it requires the `docker` CLI. Start the database manually as shown above.

## Configuration

Database settings are read from environment variables. The defaults match `compose.yaml`.

| Variable      | Default                                    |
|---------------|--------------------------------------------|
| `DB_URL`      | `jdbc:postgresql://localhost:5432/tickets` |
| `DB_USERNAME` | `myuser`                                   |
| `DB_PASSWORD` | `password`                                 |

## Running tests

Tests use Testcontainers, which start their own PostgreSQL container, so `compose.yaml` doesn't need to be running.

```shell
./gradlew test
```

**Podman note:** if Testcontainers fails with a Ryuk error, switch the Podman machine to rootful mode:

```shell
podman machine stop
podman machine set --rootful
podman machine start
```

## Database migrations

The schema is managed by Flyway (`src/main/resources/db/migration`).
Hibernate only validates it (`ddl-auto: validate`).