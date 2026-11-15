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

## Running with Docker / Podman

The `Dockerfile` uses a multi-stage build: a JDK stage builds the JAR, a JRE stage runs it as a non-root user.

1. Start the database (if it isn't running yet):

   ```shell
   podman compose up -d
   ```

2. Build the image:

   ```shell
   podman build -t ticket-system .
   ```

3. Run the container:

   ```shell
   podman run --rm -p 8080:8080 \
     -e DB_URL=jdbc:postgresql://host.containers.internal:5432/tickets \
     ticket-system
   ```

   Inside the container, `localhost` is the container itself, so the database on the host
   is reached via `host.containers.internal` (Docker: `host.docker.internal`).

4. Check: http://localhost:8080/actuator/health → `{"status":"UP"}`

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

## Deployment

The app runs on a VPS managed by [Coolify](https://coolify.io/) (self-hosted PaaS).

- **CI:** GitHub Actions builds and tests every pull request and every push to `main`.
- **CD:** every push to `main` triggers a Coolify deployment via a GitHub App webhook:
  Coolify builds the image from the `Dockerfile` and replaces the running container.
- **Database:** PostgreSQL 18 managed by Coolify, reachable only on the internal Docker network.
- **Configuration:** `DB_URL`, `DB_USERNAME` and `DB_PASSWORD` are set as environment variables in Coolify.