# Observability Patterns

A small Spring Boot demo application that demonstrates basic web endpoints, a REST client to JSONPlaceholder, and Micrometer tracing integration that can export traces to Zipkin.

This repo contains:
- A Spring Boot application at `src/main/java/charis/dev/observability_demo/ObservabilityDemoApplication.java` that creates a `JsonPlaceholderService` and a `CommandLineRunner` that loads posts.
- A simple REST controller for posts at `src/main/java/charis/dev/observability_demo/post/PostController.java`.
- Security configuration (basic in-memory user) at `src/main/java/charis/dev/observability_demo/security/SecurityConfig.java`.
- Docker Compose `compose.yaml` with a Zipkin service for local tracing.
- `pom.xml` using Spring Boot 4.0.0 and micrometer tracing/zipkin starters.

Quick start (local, requires Docker and Java 21)

1. Start Zipkin (local):

```bash
# from repo root
docker compose -f compose.yaml up -d
# verify
curl http://localhost:9411/
```

2. Run the application (interactive):

```bash
# run with Maven
mvn spring-boot:run
# or build and run jar
mvn -DskipTests package
java -jar target/observability-demo-0.0.1-SNAPSHOT.jar
```

3. Open the application
- Home: http://localhost:8080/
- Posts API: http://localhost:8080/api/posts
- Actuator endpoints: http://localhost:8080/actuator (exposed)
- Zipkin UI: http://localhost:9411/

Tracing notes

- The application includes Micrometer tracing (Brave) and `spring-boot-starter-zipkin` in `pom.xml`.
- By default `application.properties` contains `spring.application.name=observability-demo` and `management.tracing.sampling.probability=1.0`.
- Zipkin exporter properties are configured in `src/main/resources/application.properties`. If Zipkin runs on localhost (Docker), the default `spring.zipkin.base-url` is `http://localhost:9411`.
- If you instrument short-lived code paths (e.g. in tests), spans may be dropped if the JVM exits before the async reporter flushes traces. For debugging run the app interactively.

Security

- The app uses a minimal in-memory user (username: `charis`, password: `password`).
- The root (`/`) and `/actuator/**` endpoints are permitted to all; other endpoints require authentication (form login).

Developer commands / tests

- Run tests:

```bash
mvn test
```

- View dependency tree:

```bash
mvn dependency:tree
```

Troubleshooting traces not showing in Zipkin

1. Ensure Zipkin is reachable from the app (same host or correct base URL). If running the app in Docker, use `host.docker.internal` for the Zipkin host.
2. Confirm sampling is enabled (`management.tracing.sampling.probability=1.0`).
3. Check logs for Zipkin reporter activity. Increase debug logging to see exporter POSTs:

```properties
logging.level.io.zipkin.reporter2=DEBUG
logging.level.io.zipkin=DEBUG
logging.level.brave=DEBUG
logging.level.io.micrometer.tracing=DEBUG
```

