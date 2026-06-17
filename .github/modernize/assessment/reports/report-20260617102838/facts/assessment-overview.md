# Assessment Overview

This document provides navigation to all supplementary analysis documents generated as part of the PiggyMetrics application assessment. Each document focuses on a specific aspect of the application's architecture and design.

## Supplementary Documents

| Document | Description |
|---|---|
| [Architecture Diagram](./architecture-diagram.md) | Two-layer architecture visualization: high-level application topology (services, data stores, external integrations) and component relationship diagram showing internal Spring component interactions |
| [Dependency Map](./dependency-map.md) | Visual map of all external library dependencies grouped by functional category (web frameworks, security, messaging, observability, etc.) with version risk analysis |
| [API & Service Communication Contracts](./api-service-contracts.md) | Complete inventory of REST API endpoints, inter-service Feign communication patterns, resilience policies (Hystrix circuit breakers, timeouts), and service communication sequence diagram |
| [Data Architecture](./data-architecture.md) | Database configuration, entity model ER diagram, MongoDB collection ownership per service, repository interfaces, caching strategy, and data sensitivity classification (PII/credentials) |
| [Configuration Inventory](./configuration-inventory.md) | Comprehensive inventory of all configuration sources (Spring Cloud Config, bootstrap files, Docker environment variables), secrets management approach, startup dependency chain, and framework version matrix |
| [Business Workflows](./business-workflows.md) | Core business process documentation including user registration, budget management, financial statistics normalization, scheduled notifications, domain entity descriptions, and business rules |
