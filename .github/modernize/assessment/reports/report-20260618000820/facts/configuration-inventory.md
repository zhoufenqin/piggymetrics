# Configuration & Externalized Settings Inventory

PiggyMetrics relies on a mix of Spring bootstrap files, a centralized config-service shared YAML repository, Docker Compose environment variables, and a checked-in `.env` file for local secrets. The configuration model is consistent across services but depends heavily on externalized passwords and shared defaults.

## Configuration Sources

| Source | Type | Path/Location | Notes |
| --- | --- | --- | --- |
| Root environment file | Environment variables | `.env` | Local development secret values for config, service, and Mongo passwords |
| Production container topology | Docker Compose | `docker-compose.yml` | Declares images, exposed ports, and env-var wiring for all services |
| Development container overrides | Docker Compose override | `docker-compose.dev.yml` | Replaces images with local builds and exposes internal service ports |
| Config server bootstrap | Spring application config | `config/src/main/resources/application.yml` | Enables native config profile and protects config-service with Spring Security |
| Service bootstrap files | Spring bootstrap config | `*/src/main/resources/bootstrap.yml` | Declares service name plus config-service URI, credentials, and fail-fast behavior |
| Shared app defaults | Centralized config repository | `config/src/main/resources/shared/application.yml` | Common logging, Eureka, OAuth2 user-info, RabbitMQ, and Hystrix defaults |
| Service-specific shared YAML | Centralized config repository | `config/src/main/resources/shared/*.yml` | Per-service ports, Mongo hosts, gateway routes, mail settings, and schedules |
| CI build file | CI configuration | `.travis.yml` | Uses secure environment values for Docker publishing workflow |
| Dockerfiles | Runtime image config | `*/Dockerfile`, `mongodb/Dockerfile` | Pin Java runtime base image and custom MongoDB image behavior |

## Build Profiles

| Profile | Activation | Purpose | Key Dependencies/Plugins |
| --- | --- | --- | --- |
| Default Maven build | Automatic | Builds all modules from the reactor root | Spring Boot Maven Plugin and JaCoCo in service modules |
| Additional named Maven profiles | None detected | No `<profiles>` blocks are declared in the repository POM files | N/A |

## Runtime Profiles

| Profile | Activation Method | Config Files | Key Overrides |
| --- | --- | --- | --- |
| `native` | `spring.profiles.active: native` in config-service application config | `config/src/main/resources/application.yml` | Makes config-service read bundled `classpath:/shared` YAML files |
| Service default runtime | Bootstrap on startup through config-service | Each module `bootstrap.yml` plus shared YAML from config-service | Service name, config-service URI, discovery endpoint, and shared infrastructure defaults |
| Local development compose | `docker-compose -f docker-compose.yml -f docker-compose.dev.yml up` | `docker-compose.dev.yml` | Swaps prebuilt images for local builds and publishes backend ports |

## Properties Inventory

### Shared bootstrap properties

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `spring.application.name` | Service-specific module name | All | Each module `bootstrap.yml` |
| `spring.cloud.config.uri` | `http://config:8888` | All | Each module `bootstrap.yml` |
| `spring.cloud.config.fail-fast` | `true` | All | Each module `bootstrap.yml` |
| `spring.cloud.config.username` | `user` | All | Each module `bootstrap.yml` |
| `spring.cloud.config.password` | `${CONFIG_SERVICE_PASSWORD}` | All | Each module `bootstrap.yml` |

### Shared application properties

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `logging.level.org.springframework.security` | `INFO` | Shared | `shared/application.yml` |
| `hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds` | `10000` | Shared | `shared/application.yml` |
| `eureka.instance.prefer-ip-address` | `true` | Shared | `shared/application.yml` |
| `eureka.client.serviceUrl.defaultZone` | `http://registry:8761/eureka/` | Shared | `shared/application.yml` |
| `security.oauth2.resource.user-info-uri` | `http://auth-service:5000/uaa/users/current` | Shared | `shared/application.yml` |
| `spring.rabbitmq.host` | `rabbitmq` | Shared | `shared/application.yml` |

### Config-service properties

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `spring.cloud.config.server.native.search-locations` | `classpath:/shared` | `native` | `config/application.yml` |
| `spring.profiles.active` | `native` | Config service | `config/application.yml` |
| `spring.security.user.password` | `${CONFIG_SERVICE_PASSWORD}` | Config service | `config/application.yml` |
| `server.port` | `8888` | Config service | `config/application.yml` |

### Auth-service properties

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `spring.data.mongodb.host` | `auth-mongodb` | Shared | `shared/auth-service.yml` |
| `spring.data.mongodb.username` | `user` | Shared | `shared/auth-service.yml` |
| `spring.data.mongodb.password` | `${MONGODB_PASSWORD}` | Shared | `shared/auth-service.yml` |
| `spring.data.mongodb.database` | `piggymetrics` | Shared | `shared/auth-service.yml` |
| `spring.data.mongodb.port` | `27017` | Shared | `shared/auth-service.yml` |
| `server.servlet.context-path` | `/uaa` | Shared | `shared/auth-service.yml` |
| `server.port` | `5000` | Shared | `shared/auth-service.yml` |

### Account-service properties

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `security.oauth2.client.clientId` | `account-service` | Shared | `shared/account-service.yml` |
| `security.oauth2.client.clientSecret` | `${ACCOUNT_SERVICE_PASSWORD}` | Shared | `shared/account-service.yml` |
| `security.oauth2.client.accessTokenUri` | `http://auth-service:5000/uaa/oauth/token` | Shared | `shared/account-service.yml` |
| `security.oauth2.client.grant-type` | `client_credentials` | Shared | `shared/account-service.yml` |
| `security.oauth2.client.scope` | `server` | Shared | `shared/account-service.yml` |
| `spring.data.mongodb.host` | `account-mongodb` | Shared | `shared/account-service.yml` |
| `server.servlet.context-path` | `/accounts` | Shared | `shared/account-service.yml` |
| `server.port` | `6000` | Shared | `shared/account-service.yml` |
| `feign.hystrix.enabled` | `true` | Shared | `shared/account-service.yml` |

### Statistics-service properties

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `security.oauth2.client.clientId` | `statistics-service` | Shared | `shared/statistics-service.yml` |
| `security.oauth2.client.clientSecret` | `${STATISTICS_SERVICE_PASSWORD}` | Shared | `shared/statistics-service.yml` |
| `spring.data.mongodb.host` | `statistics-mongodb` | Shared | `shared/statistics-service.yml` |
| `server.servlet.context-path` | `/statistics` | Shared | `shared/statistics-service.yml` |
| `server.port` | `7000` | Shared | `shared/statistics-service.yml` |
| `rates.url` | `https://api.exchangeratesapi.io` | Shared | `shared/statistics-service.yml` |

### Notification-service properties

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `security.oauth2.client.clientId` | `notification-service` | Shared | `shared/notification-service.yml` |
| `security.oauth2.client.clientSecret` | `${NOTIFICATION_SERVICE_PASSWORD}` | Shared | `shared/notification-service.yml` |
| `server.servlet.context-path` | `/notifications` | Shared | `shared/notification-service.yml` |
| `server.port` | `8000` | Shared | `shared/notification-service.yml` |
| `remind.cron` | `0 0 0 * * *` | Shared | `shared/notification-service.yml` |
| `backup.cron` | `0 0 12 * * *` | Shared | `shared/notification-service.yml` |
| `spring.data.mongodb.host` | `notification-mongodb` | Shared | `shared/notification-service.yml` |
| `spring.mail.host` | `smtp.gmail.com` | Shared | `shared/notification-service.yml` |
| `spring.mail.port` | `465` | Shared | `shared/notification-service.yml` |
| `spring.mail.username` | `dev-user` | Shared | `shared/notification-service.yml` |
| `spring.mail.password` | `dev-password` | Shared | `shared/notification-service.yml` |

### Gateway and infrastructure properties

| Property Key | Default | Profiles | Source |
| --- | --- | --- | --- |
| `hystrix.command.default.execution.isolation.thread.timeoutInMilliseconds` | `20000` | Gateway | `shared/gateway.yml` |
| `ribbon.ReadTimeout` | `20000` | Gateway | `shared/gateway.yml` |
| `ribbon.ConnectTimeout` | `20000` | Gateway | `shared/gateway.yml` |
| `zuul.routes.auth-service.path` | `/uaa/**` | Gateway | `shared/gateway.yml` |
| `zuul.routes.account-service.path` | `/accounts/**` | Gateway | `shared/gateway.yml` |
| `zuul.routes.statistics-service.path` | `/statistics/**` | Gateway | `shared/gateway.yml` |
| `zuul.routes.notification-service.path` | `/notifications/**` | Gateway | `shared/gateway.yml` |
| `server.port` | `4000` | Gateway | `shared/gateway.yml` |
| `server.port` | `8761` | Registry | `shared/registry.yml` |

## Startup Parameters & Resource Requirements

| Service | JVM/Runtime Options | Memory | Instance Count |
| --- | --- | --- | --- |
| All Java services | No explicit `-Xms`, `-Xmx`, or `-Dspring.profiles.active` startup flags declared in repo | No memory limits declared in Docker Compose | Single instance per service in compose files |
| rabbitmq | Container defaults | No memory limits declared | Single instance |
| MongoDB containers | Container defaults | No memory limits declared | One instance per business service |

The repository documents a practical environment requirement instead of per-service sizing: the README notes that running eight Spring Boot applications, four MongoDB instances, and RabbitMQ requires at least 4 GB RAM.

## Startup Dependency Chain

1. `config` starts first and exposes centralized configuration on port 8888.
2. `registry` waits on `config` through Docker Compose `depends_on` with `condition: service_healthy`.
3. `gateway`, `auth-service`, `account-service`, `statistics-service`, `notification-service`, `monitoring`, and `turbine-stream-service` all wait on `config` the same way.
4. After bootstrap, service discovery depends on `registry` being reachable at `http://registry:8761/eureka/`.
5. Business-service features then depend on their own MongoDB containers, while observability depends on RabbitMQ and Turbine stream availability.

## Secrets & Sensitive Configuration

| Secret Reference | Type | Storage (masked) |
| --- | --- | --- |
| `CONFIG_SERVICE_PASSWORD` | Config server bootstrap password | Environment variable / `.env` as `[MASKED]` |
| `ACCOUNT_SERVICE_PASSWORD` | OAuth2 client secret | Environment variable / `.env` as `[MASKED]` |
| `STATISTICS_SERVICE_PASSWORD` | OAuth2 client secret | Environment variable / `.env` as `[MASKED]` |
| `NOTIFICATION_SERVICE_PASSWORD` | OAuth2 client secret | Environment variable / `.env` as `[MASKED]` |
| `MONGODB_PASSWORD` | MongoDB password | Environment variable / `.env` as `[MASKED]` |
| `spring.mail.password` | SMTP password | Service YAML currently checked in as `[MASKED]` for inventory purposes |

### Secrets Provisioning Workflow

Secrets originate primarily from environment variables passed into Docker Compose and referenced from the shared YAML configuration with Spring placeholder syntax. Every service needs `CONFIG_SERVICE_PASSWORD` to bootstrap from config-service, the business services also need their own OAuth2 client secrets for client-credentials flows, and the MongoDB-backed services need `MONGODB_PASSWORD` for database access. The repository also contains a checked-in development SMTP credential value in shared configuration, which indicates a local-development secret workflow rather than an external vault or managed secret store.

## Feature Flags

| Flag Name | Default | Controlled By |
| --- | --- | --- |
| None detected | N/A | No `@ConditionalOnProperty`, feature-management library, or explicit toggle files were found |

## Framework & Runtime Versions

| Component | Version | Source |
| --- | --- | --- |
| Java language target | 1.8 | Root `pom.xml` |
| Spring Boot | 2.0.3.RELEASE | Root parent POM |
| Spring Cloud | Finchley.RELEASE | Root `spring-cloud.version` property |
| Guava | 19.0 | `statistics-service/pom.xml` |
| Embedded Mongo test library | 1.50.3 | Service module POMs |
| JaCoCo Maven Plugin | 0.7.6.201602180812 | Service module POMs |
| Java runtime base image | `java:8-jre` | Service Dockerfiles |
| MongoDB base image | `mongo:3` | `mongodb/Dockerfile` |
| RabbitMQ image | `rabbitmq:3-management` | `docker-compose.yml` |
| Maven | Wrapper not present; repository assumes installed Maven | README build instructions |
