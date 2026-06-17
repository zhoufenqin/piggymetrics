# Configuration & Externalized Settings Inventory

PiggyMetrics uses Spring Cloud Config as a centralized configuration server (serving classpath-based native configuration), with 9 bootstrap files pointing all services to it, and secrets injected exclusively via Docker environment variables with no dedicated secret store.

## Configuration Sources

| Source | Type | Path / Location | Notes |
|---|---|---|---|
| Spring Cloud Config Server (native) | Centralized config server | `config/src/main/resources/shared/` | Serves all per-service YAML files from classpath; `profiles.active: native` means no Git remote |
| `config/src/main/resources/application.yml` | Config server own config | In-module classpath | Configures HTTP Basic auth password, server port 8888, and native search location |
| `config/src/main/resources/shared/application.yml` | Shared application config | Served to all services | Hystrix timeout, Eureka client URI, OAuth2 user-info URI, RabbitMQ host |
| `config/src/main/resources/shared/account-service.yml` | Per-service config | Served to account-service | MongoDB connection, server port 6000, OAuth2 client, Feign+Hystrix enabled |
| `config/src/main/resources/shared/auth-service.yml` | Per-service config | Served to auth-service | MongoDB connection, server port 5000, context-path `/uaa` |
| `config/src/main/resources/shared/statistics-service.yml` | Per-service config | Served to statistics-service | MongoDB connection, server port 7000, exchange rates URL |
| `config/src/main/resources/shared/notification-service.yml` | Per-service config | Served to notification-service | MongoDB connection, server port 8000, SMTP config, cron expressions |
| `config/src/main/resources/shared/gateway.yml` | Per-service config | Served to gateway | Zuul routes, Hystrix/Ribbon timeouts |
| `config/src/main/resources/shared/registry.yml` | Per-service config | Served to registry | Eureka server port 8761 |
| `config/src/main/resources/shared/turbine-stream-service.yml` | Per-service config | Served to turbine-stream-service | (inherits shared config; no additional properties) |
| `config/src/main/resources/shared/monitoring.yml` | Per-service config | Served to monitoring | (inherits shared config; no additional properties) |
| `{service}/src/main/resources/bootstrap.yml` | Bootstrap config | Per-service classpath | Sets `spring.application.name`, Config Server URI (`http://config:8888`), `fail-fast: true`, and Basic Auth credentials |
| `.env` | Environment variable file | Repository root | Docker Compose env defaults for local dev (all passwords set to `password`) |
| `docker-compose.yml` | Container orchestration | Repository root | Defines service environment variables for production-like deployment |
| `docker-compose.dev.yml` | Container orchestration overlay | Repository root | Dev override: builds from source, exposes extra ports for debugging |

No external Git repository, Azure App Configuration, AWS AppConfig, HashiCorp Vault, or Consul KV is used. All configuration is stored in the `config` module's classpath.

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies / Plugins |
|---|---|---|---|
| (default) | Automatic | Standard Maven build producing JAR | `spring-boot-maven-plugin` (all modules); `jacoco-maven-plugin` in auth-service |
| N/A | — | No additional Maven profiles defined | No `-P` profiles declared in any `pom.xml` |

The root POM and all module POMs define no additional Maven build profiles. Docker image builds are handled externally (via `Dockerfile` per service and `docker-compose.dev.yml`), not through Maven profiles.

## Runtime Profiles

| Profile | Activation Method | Config Files / Locations | Key Overrides |
|---|---|---|---|
| native | `spring.profiles.active: native` in `config/application.yml` | `classpath:/shared/*.yml` | Config Server uses filesystem/classpath instead of Git |
| (no service profiles) | — | — | Business services do not declare `application-{profile}.yml` files; all config is served by the Config Server |
| test | In-module `src/test/resources/application.yml` and `bootstrap.yml` | Per-service test classpath | Uses `de.flapdoodle.embed.mongo` (embedded MongoDB); disables Config Server lookup (`spring.cloud.config.enabled: false`) |

No `@Profile`-annotated beans or profile-specific application property files (`application-dev.yml`, `application-prod.yml`) exist in business services — all environment-specific configuration is handled by the centralized Config Server and Docker environment variables.

## Properties Inventory

### Shared (all services via `shared/application.yml`)

| Property Key | Value / Default | Source |
|---|---|---|
| `logging.level.org.springframework.security` | `INFO` | Shared YAML |
| `hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds` | `10000` | Shared YAML |
| `eureka.instance.prefer-ip-address` | `true` | Shared YAML |
| `eureka.client.serviceUrl.defaultZone` | `http://registry:8761/eureka/` | Shared YAML |
| `security.oauth2.resource.user-info-uri` | `http://auth-service:5000/uaa/users/current` | Shared YAML |
| `spring.rabbitmq.host` | `rabbitmq` | Shared YAML |

### gateway (`shared/gateway.yml`)

| Property Key | Value / Default | Source |
|---|---|---|
| `hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds` | `20000` (overrides shared) | gateway YAML |
| `ribbon.ReadTimeout` | `20000` | gateway YAML |
| `ribbon.ConnectTimeout` | `20000` | gateway YAML |
| `zuul.ignoredServices` | `*` | gateway YAML |
| `zuul.host.connect-timeout-millis` | `20000` | gateway YAML |
| `zuul.host.socket-timeout-millis` | `20000` | gateway YAML |
| `zuul.routes.auth-service.path` | `/uaa/**` | gateway YAML |
| `zuul.routes.auth-service.url` | `http://auth-service:5000` | gateway YAML |
| `zuul.routes.account-service.path` | `/accounts/**` | gateway YAML |
| `zuul.routes.account-service.serviceId` | `account-service` | gateway YAML |
| `zuul.routes.statistics-service.path` | `/statistics/**` | gateway YAML |
| `zuul.routes.statistics-service.serviceId` | `statistics-service` | gateway YAML |
| `zuul.routes.notification-service.path` | `/notifications/**` | gateway YAML |
| `zuul.routes.notification-service.serviceId` | `notification-service` | gateway YAML |
| `server.port` | `4000` | gateway YAML |

### account-service (`shared/account-service.yml`)

| Property Key | Value / Default | Source |
|---|---|---|
| `security.oauth2.client.clientId` | `account-service` | account-service YAML |
| `security.oauth2.client.clientSecret` | `${ACCOUNT_SERVICE_PASSWORD}` | Env var |
| `security.oauth2.client.accessTokenUri` | `http://auth-service:5000/uaa/oauth/token` | account-service YAML |
| `security.oauth2.client.grant-type` | `client_credentials` | account-service YAML |
| `security.oauth2.client.scope` | `server` | account-service YAML |
| `spring.data.mongodb.host` | `account-mongodb` | account-service YAML |
| `spring.data.mongodb.username` | `user` | account-service YAML |
| `spring.data.mongodb.password` | `${MONGODB_PASSWORD}` | Env var |
| `spring.data.mongodb.database` | `piggymetrics` | account-service YAML |
| `spring.data.mongodb.port` | `27017` | account-service YAML |
| `server.servlet.context-path` | `/accounts` | account-service YAML |
| `server.port` | `6000` | account-service YAML |
| `feign.hystrix.enabled` | `true` | account-service YAML |

### auth-service (`shared/auth-service.yml`)

| Property Key | Value / Default | Source |
|---|---|---|
| `spring.data.mongodb.host` | `auth-mongodb` | auth-service YAML |
| `spring.data.mongodb.username` | `user` | auth-service YAML |
| `spring.data.mongodb.password` | `${MONGODB_PASSWORD}` | Env var |
| `spring.data.mongodb.database` | `piggymetrics` | auth-service YAML |
| `spring.data.mongodb.port` | `27017` | auth-service YAML |
| `server.servlet.context-path` | `/uaa` | auth-service YAML |
| `server.port` | `5000` | auth-service YAML |

### statistics-service (`shared/statistics-service.yml`)

| Property Key | Value / Default | Source |
|---|---|---|
| `security.oauth2.client.clientId` | `statistics-service` | statistics-service YAML |
| `security.oauth2.client.clientSecret` | `${STATISTICS_SERVICE_PASSWORD}` | Env var |
| `spring.data.mongodb.host` | `statistics-mongodb` | statistics-service YAML |
| `spring.data.mongodb.password` | `${MONGODB_PASSWORD}` | Env var |
| `spring.data.mongodb.database` | `piggymetrics` | statistics-service YAML |
| `server.servlet.context-path` | `/statistics` | statistics-service YAML |
| `server.port` | `7000` | statistics-service YAML |
| `rates.url` | `https://api.exchangeratesapi.io` | statistics-service YAML |

### notification-service (`shared/notification-service.yml`)

| Property Key | Value / Default | Source |
|---|---|---|
| `security.oauth2.client.clientId` | `notification-service` | notification-service YAML |
| `security.oauth2.client.clientSecret` | `${NOTIFICATION_SERVICE_PASSWORD}` | Env var |
| `server.port` | `8000` | notification-service YAML |
| `remind.cron` | `0 0 0 * * *` (midnight daily) | notification-service YAML |
| `backup.cron` | `0 0 12 * * *` (noon daily) | notification-service YAML |
| `spring.data.mongodb.host` | `notification-mongodb` | notification-service YAML |
| `spring.data.mongodb.password` | `${MONGODB_PASSWORD}` | Env var |
| `spring.mail.host` | `smtp.gmail.com` | notification-service YAML |
| `spring.mail.port` | `465` | notification-service YAML |
| `spring.mail.username` | `dev-user` | notification-service YAML (hardcoded dev) |
| `spring.mail.password` | `dev-password` | notification-service YAML (hardcoded dev) |
| `spring.mail.properties.mail.smtp.auth` | `true` | notification-service YAML |
| `spring.mail.properties.mail.smtp.ssl.enable` | `true` | notification-service YAML |

### Config Server own config (`config/application.yml`)

| Property Key | Value / Default | Source |
|---|---|---|
| `spring.cloud.config.server.native.search-locations` | `classpath:/shared` | config application.yml |
| `spring.profiles.active` | `native` | config application.yml |
| `spring.security.user.password` | `${CONFIG_SERVICE_PASSWORD}` | Env var |
| `server.port` | `8888` | config application.yml |

## Startup Parameters & Resource Requirements

| Service | JVM Options | Exposed Port | Base Image | Memory Limit |
|---|---|---|---|---|
| account-service | `-Xmx200m` | 6000 | `java:8-jre` | 200 MB heap max |
| auth-service | (no Dockerfile in repo; inferred) | 5000 | `java:8-jre` | Not specified |
| statistics-service | (no Dockerfile in repo; inferred) | 7000 | `java:8-jre` | Not specified |
| notification-service | `-Xmx200m` | 8000 | `java:8-jre` | 200 MB heap max |
| gateway | (no Dockerfile in repo; inferred) | 4000 | `java:8-jre` | Not specified |
| config | (no Dockerfile in repo; inferred) | 8888 | `java:8-jre` | Not specified |
| registry | `-Xmx200m` | 8761 | `java:8-jre` | 200 MB heap max |
| turbine-stream-service | `-Xmx200m` | 8989 | `java:8-jre` | 200 MB heap max |
| monitoring | (no Dockerfile in repo; inferred) | 9000 | `java:8-jre` | Not specified |
| auth-mongodb / account-mongodb / statistics-mongodb / notification-mongodb | N/A | 27017 | `mongo:3` | Not specified |
| rabbitmq | N/A | 15672 | `rabbitmq:3-management` | Not specified |

No Kubernetes resource requests/limits or Docker `mem_limit` directives are configured. Instance count is 1 per service (no horizontal scaling configured in Docker Compose). No `-Xms` settings are defined.

## Startup Dependency Chain

Services must start in the following order (enforced by Docker Compose `depends_on` with `condition: service_healthy`):

1. **RabbitMQ** — no dependencies; must start first (message broker for Spring Cloud Bus and Hystrix streams)
2. **Config Service** — no dependencies; must be healthy before any other service
3. **Registry** — depends on Config (healthy); provides Eureka server for service registration
4. **Gateway** — depends on Config (healthy)
5. **Auth Service** — depends on Config (healthy); business services cannot authenticate without it
6. **Account Service** — depends on Config (healthy); registers with Eureka, connects to Auth Service
7. **Statistics Service** — depends on Config (healthy); registers with Eureka
8. **Notification Service** — depends on Config (healthy); registers with Eureka
9. **Monitoring** — depends on Config (healthy)
10. **Turbine Stream Service** — depends on Config (healthy); connects to RabbitMQ for Hystrix streams

All business service bootstrap files set `spring.cloud.config.fail-fast: true` — services will fail to start if the Config Server is unreachable. No `dockerize` wait-for-TCP or Spring Retry config is explicitly set beyond `fail-fast`. The Config Server healthcheck is the only inter-service readiness gate in Docker Compose.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage |
|---|---|---|
| `${CONFIG_SERVICE_PASSWORD}` | Config Server HTTP Basic password | Environment variable (Docker Compose / `.env`) |
| `${MONGODB_PASSWORD}` | MongoDB database password (all 4 instances) | Environment variable (Docker Compose / `.env`) |
| `${ACCOUNT_SERVICE_PASSWORD}` | OAuth2 client secret for account-service | Environment variable (Docker Compose / `.env`) |
| `${STATISTICS_SERVICE_PASSWORD}` | OAuth2 client secret for statistics-service | Environment variable (Docker Compose / `.env`) |
| `${NOTIFICATION_SERVICE_PASSWORD}` | OAuth2 client secret for notification-service | Environment variable (Docker Compose / `.env`) |
| `spring.mail.username` | SMTP username | Hardcoded `dev-user` in `notification-service.yml` |
| `spring.mail.password` | SMTP password | Hardcoded `dev-password` in `notification-service.yml` |
| OAuth2 token signing key | JWT/token signing | In-memory `InMemoryTokenStore` — not persisted, lost on auth-service restart |

### Secrets Provisioning Workflow

All secrets are passed as plain-text Docker environment variables. The `.env` file in the repository root provides default values (`password`) for local development. For production deployments, operators are expected to override these environment variables at container runtime.

**Flow:**
1. Operator sets environment variables (`CONFIG_SERVICE_PASSWORD`, `MONGODB_PASSWORD`, `ACCOUNT_SERVICE_PASSWORD`, `STATISTICS_SERVICE_PASSWORD`, `NOTIFICATION_SERVICE_PASSWORD`) before running `docker-compose up`
2. Docker Compose injects them into each container's environment
3. Spring Boot reads them via `${ENV_VAR}` placeholder resolution in YAML files served by the Config Server
4. The Config Server's own password is read from `${CONFIG_SERVICE_PASSWORD}` in its `application.yml`; business services pass this same password in their `bootstrap.yml` to authenticate with the Config Server

**Risks:** No dedicated secret store (no Vault, no KMS, no Kubernetes Secrets). SMTP credentials are hardcoded in the shared config file (not parameterized). The `.env` file with default passwords (`password`) is committed to the repository. OAuth2 tokens are stored only in memory — all active sessions are invalidated when auth-service restarts. `NoOpPasswordEncoder` is used for OAuth2 client secrets in `OAuth2AuthorizationConfig`, meaning client secrets are compared in plain text.

## Feature Flags

No feature flag framework (LaunchDarkly, Unleash, Spring Feature Flags, or `.NET FeatureManagement`) is used. No `@ConditionalOnProperty` or `@ConditionalOnExpression` beans are declared. No A/B testing or gradual rollout configuration is present.

| Flag Name | Default | Controlled By |
|---|---|---|
| `feign.hystrix.enabled` | `true` (account-service) | `shared/account-service.yml` |
| N/A | — | No other feature-flag-like properties detected |

## Framework & Runtime Versions

| Component | Version | Source |
|---|---|---|
| Java | 8 (target) | Root `pom.xml` `<java.version>1.8</java.version>` |
| Spring Boot | 2.0.3.RELEASE | Root `pom.xml` parent |
| Spring Cloud | Finchley.RELEASE | Root `pom.xml` BOM import |
| Spring Security OAuth2 | Finchley (managed) | BOM |
| Spring Data MongoDB | 2.0.x (managed by Spring Boot BOM) | BOM |
| Spring Cloud Config Server/Client | Finchley | BOM |
| Netflix Eureka Server/Client | Finchley | BOM |
| Netflix Zuul | Finchley | BOM |
| Netflix Hystrix | Finchley | BOM |
| Netflix Turbine Stream | Finchley | BOM |
| Spring Cloud Sleuth | Finchley | BOM |
| Spring Cloud Bus (AMQP) | Finchley | BOM |
| Spring Cloud OpenFeign | Finchley | BOM |
| Guava | 19.0 | `statistics-service/pom.xml` explicit |
| de.flapdoodle.embed.mongo | 1.50.3 | Per-service `pom.xml` (test scope) |
| Maven | Not pinned (wrapper not present) | Build tool |
| Docker base image (services) | `java:8-jre` | Per-service `Dockerfile` |
| Docker base image (MongoDB) | `mongo:3` | `mongodb/Dockerfile` |
| Docker Compose format | `2.1` | `docker-compose.yml` |
