# piggymetrics

## Summary

| Metric | Value |
|--------|-------|
| Total Issues | 9 |
| Mandatory Blockers | 5 |
| Potential Issues | 2 |

## Component Information

| Property | Value |
|----------|-------|
| Language | Java, Dockerfile |
| Frameworks | Spring Boot, Spring Cloud, Spring |
| Build tools | Maven |
| JDK version | 1.8 |

## Cloud Readiness Issues

| Issue Name | Criticality | Story Points | Occurrences |
|------------|-------------|--------------|-------------|
| Use of unsecured network protocols or URI libraries | Mandatory | 3 | [14](#Use_of_unsecured_network_protocols_or_URI_libraries) |
| CRA: Default or well-known password detected | Mandatory | 3 | [8](#CRA_Default_or_well-known_password_detected) |
| CRA: Hard-coded password in Java source code | Mandatory | 8 | [6](#CRA_Hard-coded_password_in_Java_source_code) |
| CRA: Use of weak password hashing (plain MD5/SHA for passwords) | Mandatory | 8 | [2](#CRA_Use_of_weak_password_hashing_plain_MD5_SHA_for_passwords) |
| MongoDB connection found in configuration file | Potential | 5 | [16](#MongoDB_connection_found_in_configuration_file) |
| Password found in configuration file | Potential | 3 | [10](#Password_found_in_configuration_file) |
| Avoid using hardcoded URLs (HTTP protocol) in source code | Optional | 3 | [17](#Avoid_using_hardcoded_URLs_HTTP_protocol_in_source_code) |
| Spring AMQP dependency found | Optional | 5 | [4](#Spring_AMQP_dependency_found) |

### Issue Details

<details id="Use_of_unsecured_network_protocols_or_URI_libraries">
<summary><b>Use of unsecured network protocols or URI libraries</b> — affected files</summary>

- `account-service/src/main/resources/bootstrap.yml (line 6)`
- `auth-service/src/main/resources/bootstrap.yml (line 6)`
- `config/src/main/resources/shared/account-service.yml (line 6)`
- `config/src/main/resources/shared/application.yml (line 18)`
- `config/src/main/resources/shared/application.yml (line 23)`
- `config/src/main/resources/shared/gateway.yml (line 22)`
- `config/src/main/resources/shared/notification-service.yml (line 6)`
- `config/src/main/resources/shared/statistics-service.yml (line 6)`
- `gateway/src/main/resources/bootstrap.yml (line 6)`
- `monitoring/src/main/resources/bootstrap.yml (line 6)`
- `notification-service/src/main/resources/bootstrap.yml (line 6)`
- `registry/src/main/resources/bootstrap.yml (line 6)`
- `statistics-service/src/main/resources/bootstrap.yml (line 6)`
- `turbine-stream-service/src/main/resources/bootstrap.yml (line 6)`

</details>

<details id="CRA_Default_or_well-known_password_detected">
<summary><b>CRA: Default or well-known password detected</b> — affected files</summary>

- `account-service/src/main/java/com/piggymetrics/account/domain/User.java (line 30)`
- `auth-service/src/main/java/com/piggymetrics/auth/domain/User.java (line 38)`
- `notification-service/src/test/resources/application.yml (line 23)`
- `account-service/src/test/java/com/piggymetrics/account/controller/AccountControllerTest.java (line 129)`
- `auth-service/src/test/java/com/piggymetrics/auth/controller/UserControllerTest.java (line 49)`
- `auth-service/src/test/java/com/piggymetrics/auth/repository/UserRepositoryTest.java (line 30)`
- `auth-service/src/test/java/com/piggymetrics/auth/service/UserServiceTest.java (line 33)`
- `auth-service/src/test/java/com/piggymetrics/auth/service/UserServiceTest.java (line 44)`

</details>

<details id="CRA_Hard-coded_password_in_Java_source_code">
<summary><b>CRA: Hard-coded password in Java source code</b> — affected files</summary>

- `account-service/src/test/java/com/piggymetrics/account/controller/AccountControllerTest.java (line 129)`
- `auth-service/src/test/java/com/piggymetrics/auth/controller/UserControllerTest.java (line 49)`
- `auth-service/src/test/java/com/piggymetrics/auth/controller/UserControllerTest.java (line 62)`
- `auth-service/src/test/java/com/piggymetrics/auth/repository/UserRepositoryTest.java (line 30)`
- `auth-service/src/test/java/com/piggymetrics/auth/service/UserServiceTest.java (line 33)`
- `auth-service/src/test/java/com/piggymetrics/auth/service/UserServiceTest.java (line 44)`

</details>

<details id="CRA_Use_of_weak_password_hashing_plain_MD5_SHA_for_passwords">
<summary><b>CRA: Use of weak password hashing (plain MD5/SHA for passwords)</b> — affected files</summary>

- `auth-service/src/main/java/com/piggymetrics/auth/config/OAuth2AuthorizationConfig.java (line 26)`
- `auth-service/src/main/java/com/piggymetrics/auth/config/OAuth2AuthorizationConfig.java (line 79)`

</details>

<details id="MongoDB_connection_found_in_configuration_file">
<summary><b>MongoDB connection found in configuration file</b> — affected files</summary>

- `account-service/src/test/resources/application.yml (line 3)`
- `auth-service/src/test/resources/application.yml (line 3)`
- `config/src/main/resources/shared/account-service.yml (line 12)`
- `config/src/main/resources/shared/auth-service.yml (line 3)`
- `config/src/main/resources/shared/notification-service.yml (line 30)`
- `config/src/main/resources/shared/statistics-service.yml (line 12)`
- `docker-compose.dev.yml (line 23)`
- `docker-compose.dev.yml (line 33)`
- `docker-compose.dev.yml (line 43)`
- `docker-compose.dev.yml (line 53)`
- `docker-compose.yml (line 70)`
- `docker-compose.yml (line 95)`
- `docker-compose.yml (line 121)`
- `docker-compose.yml (line 146)`
- `notification-service/src/test/resources/application.yml (line 16)`
- `statistics-service/src/test/resources/application.yml (line 11)`

</details>

<details id="Password_found_in_configuration_file">
<summary><b>Password found in configuration file</b> — affected files</summary>

- `auth-service/src/main/resources/bootstrap.yml (line 8)`
- `config/src/main/resources/application.yml (line 11)`
- `gateway/src/main/resources/bootstrap.yml (line 8)`
- `monitoring/src/main/resources/bootstrap.yml (line 8)`
- `notification-service/src/main/resources/bootstrap.yml (line 8)`
- `notification-service/src/test/resources/application.yml (line 23)`
- `registry/src/main/resources/bootstrap.yml (line 8)`
- `statistics-service/src/main/resources/bootstrap.yml (line 8)`
- `turbine-stream-service/src/main/resources/bootstrap.yml (line 8)`
- `account-service/src/main/resources/bootstrap.yml (line 8)`

</details>

<details id="Avoid_using_hardcoded_URLs_HTTP_protocol_in_source_code">
<summary><b>Avoid using hardcoded URLs (HTTP protocol) in source code</b> — affected files</summary>

- `.travis.yml (line 18)`
- `account-service/src/main/resources/bootstrap.yml (line 6)`
- `auth-service/src/main/resources/bootstrap.yml (line 6)`
- `config/src/main/resources/shared/account-service.yml (line 6)`
- `config/src/main/resources/shared/application.yml (line 18)`
- `config/src/main/resources/shared/application.yml (line 23)`
- `config/src/main/resources/shared/gateway.yml (line 22)`
- `config/src/main/resources/shared/notification-service.yml (line 6)`
- `config/src/main/resources/shared/statistics-service.yml (line 6)`
- `config/src/main/resources/shared/statistics-service.yml (line 25)`
- `gateway/src/main/resources/bootstrap.yml (line 6)`
- `monitoring/src/main/resources/bootstrap.yml (line 6)`
- `notification-service/src/main/resources/bootstrap.yml (line 6)`
- `registry/src/main/resources/bootstrap.yml (line 6)`
- `statistics-service/src/main/resources/bootstrap.yml (line 6)`
- `statistics-service/src/test/resources/application.yml (line 16)`
- `turbine-stream-service/src/main/resources/bootstrap.yml (line 6)`

</details>

<details id="Spring_AMQP_dependency_found">
<summary><b>Spring AMQP dependency found</b> — affected files</summary>

- `account-service/pom.xml (line 57)`
- `notification-service/pom.xml (line 57)`
- `statistics-service/pom.xml (line 57)`

</details>

## Upgrade Issues

| Issue Name | Criticality | Story Points | Occurrences |
|------------|-------------|--------------|-------------|
| Java Version Has Reached the End of Support | Mandatory | 8 | [1](#Java_Version_Has_Reached_the_End_of_Support) |

### Issue Details

<details id="Java_Version_Has_Reached_the_End_of_Support">
<summary><b>Java Version Has Reached the End of Support</b> — affected files</summary>

- `pom.xml (line 21)`

</details>

---

## Codebase Insights

> **Note:** These documents are generated by AI and may contain inaccuracies or incomplete information. Please review carefully.

1. **[Architecture Diagram](facts/architecture-diagram.md)** — Understand the big picture: system layers and component relationships
2. **[Dependency Map](facts/dependency-map.md)** — Know what the project depends on and where the risks are
3. **[API & Service Contracts](facts/api-service-contracts.md)** — See how services communicate and what contracts they expose
4. **[Data Architecture](facts/data-architecture.md)** — Explore data models, storage, and data flow patterns
5. **[Configuration Inventory](facts/configuration-inventory.md)** — Review how the application is configured across environments
6. **[Business Workflows](facts/business-workflows.md)** — Trace end-to-end business processes and domain logic

[Share feedback](https://aka.ms/ghcp-appmod/feedback)
