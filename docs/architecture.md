---
layout: default
title: Architecture
nav_order: 2
---

# Qavo — Reference Application Architecture

> **Qavo** — *the foundation your applications stand on.*

**Stack:** Java / Spring Boot (backend) · Angular (frontend)
**Model:** Versioned shared platform + independent applications
**Distribution:** Open source, hosted on GitHub, publicly published artifacts
**UI approach:** Mobile-first, responsive across all devices (mobile, tablet, desktop)
**Development:** Designed and built with the assistance of artificial intelligence (see §12)
**Document version:** 1.3
**Date:** May 2026

---

## 1. Goals and guiding principles

This document defines the reference architecture for **Qavo**, a platform for building full-stack applications. Its purpose is not to describe a single application, but a **cross-cutting platform** on which many applications can be built, sharing in a standardized and centralized way all the common concerns: routing, error handling, logging, validation, authentication, authorization, theming, and a modular capability system.

The name *Qavo* is a coined, deliberately distinctive word chosen for being short, easy to pronounce, and free of collisions across package registries and product names. It carries no prior meaning to compete with — the platform gives it its meaning.

Eight principles drive every choice that follows.

**Centralization of cross-cutting concerns.** Everything common to multiple applications — security, error handling, logging, validation, theming, response formats — lives in shared components and is never reimplemented in each application. A fix or an improvement propagates to all applications through a version update.

**Independent evolution.** The platform evolves with its own lifecycle and its own versioning. Each application references a specific version and decides autonomously when to upgrade. No application is forced to upgrade in lockstep with the others.

**Modularity by composition.** Functionality is delivered as discrete, independently versioned modules. An application imports the `core` plus only the capability modules (plugins) it actually needs. What is not imported does not exist in the application — no dead code, no extra attack surface.

**Convention over configuration.** The architecture imposes clear conventions (package structure, error format, endpoint naming) so that applications are consistent with each other and predictable to maintain, reducing repeated decisions.

**Clean layer separation.** Each responsibility has its place. Presentation, application, domain, and data-access logic stay decoupled, so that each can evolve and be tested in isolation.

**Mobile-first, responsive everywhere.** The user interface is designed mobile-first: layouts, components, and interactions are conceived for the smallest screens first, then progressively enhanced for larger ones. The same application renders correctly and ergonomically across all device classes — phones, tablets, and desktops — from a single codebase, with no separate "mobile version". This is detailed in §5.8.

**Secure by default.** Security is not a plugin or an opt-in concern — it is a foundational property of every Qavo-based application. Secure HTTP headers, input sanitization, TLS awareness, and resilient communication patterns are active from the first line of configuration, not added later. This principle is detailed throughout §5.5 and §5.9.

**Long-term maintainability.** The architecture favors mature technologies with long-term support (LTS), a stable ecosystem, and a slow obsolescence curve, because code written today must still be maintainable five years from now.

---

## 2. Technology choices

### 2.1 Backend — Java with Spring Boot 3.x

The choice of Java with Spring is confirmed and fully suited to the centralization requirement. Spring Boot is today the de facto standard for enterprise Java applications and natively covers nearly all the required cross-cutting concerns.

The decisive element for this architecture is the **Spring Boot Starter** pattern: it is possible to build a proprietary library (a "company starter") that, once declared as a dependency, automatically activates — through auto-configuration — all the shared configuration (security, error handling, logging, validation). This is the same mechanism Spring itself uses to distribute its features, and it directly answers the requirement of independent evolution: an application changes its shared behavior simply by bumping the version number of the dependency. The same mechanism is what makes Qavo's plugin model (see §6) clean: each capability is its own starter.

**Reference version:** Spring Boot 3.x, on a Java 21 (LTS) baseline. This choice brings:

- Jakarta EE 9+ (`jakarta.*` namespace), aligned with the current ecosystem.
- Long-term support for both Java 21 and the Spring Boot 3.x lines.
- Virtual Threads (Project Loom), useful for applications with heavy concurrent I/O.
- First-class observability via Micrometer and OpenTelemetry integration.

**Complementary backend technologies:**

| Concern | Technology | Notes |
|---|---|---|
| Web layer | Spring Web MVC | Synchronous REST; WebFlux only if heavy reactivity is needed |
| Persistence | Spring Data JPA + Hibernate | Repository pattern, derived queries |
| Schema migrations | Flyway | Versioned DB schema, aligned with the code; each module ships its own migrations |
| Security | Spring Security (form/DB + OAuth2 Resource Server) | Pluggable authentication (see §5.5) |
| Validation | Bean Validation (Jakarta Validation / Hibernate Validator) | Declarative annotations |
| DTO mapping | MapStruct | Compile-time entity ↔ DTO mapping |
| API documentation | springdoc-openapi | Automatic OpenAPI 3 / Swagger UI generation |
| Resilience | Resilience4j | Retry, timeout, circuit breaker — enforced in core (see §5.9) |
| Feature flags | `@ConditionalOnProperty` + Qavo convention | Runtime enable/disable of features without rebuild (see §5.6) |
| Conditional wiring | Spring Boot auto-configuration | `@ConditionalOnClass` / `@ConditionalOnProperty` drive the plugin model |
| Build | Maven (or Gradle) | Maven recommended for BOM management |
| Testing | JUnit 5, Mockito, Testcontainers | Integration tests with a real DB in a container |

### 2.2 Frontend — Angular

Given the expected need (a mix of back-office applications and more content-oriented applications, with SEO not a current priority), the motivated recommendation is **Angular**.

The main reason is the direct alignment with the central requirement of this architecture: *standardization and centralization imposed by the framework*. Angular is a complete, opinionated framework that natively provides, without having to select and integrate third-party libraries:

- Built-in **routing**, with lazy loading, route guards, and data resolution.
- **Forms**, both template-driven and reactive, with first-class field- and form-level validation — exactly the "form and field validation" concern that is required.
- **HttpClient** with an **interceptor** mechanism that is the natural place to centralize authentication (token injection), error handling, and request logging.
- Structured **Dependency Injection**, conceptually akin to Spring's: anyone who knows Spring gets up to speed quickly.
- A **modular** architecture (or standalone components in recent versions) that maps directly onto Qavo's plugin model: each capability is shipped as a separately publishable Angular library.

This means that most cross-cutting concerns on the client are already provided by the framework and simply need to be *configured once* in a shared library, rather than assembled by choosing each time among competing options.

**Comparison with React (the main alternative).** React is a library, not a framework: extremely flexible and with the largest ecosystem, but it requires you to independently select and standardize routing (React Router), forms (React Hook Form), data fetching (TanStack Query), and state management. It is an excellent choice when you want maximum control, but it shifts onto you the burden of defining and maintaining the "standard". For an architecture whose stated goal is to *impose a shared standard*, Angular's batteries-included approach has less friction.

**Reference version:** Angular at its current release (semi-annual release cadence, with LTS). The architecture adopts **standalone components** and **signals** where appropriate, and keeps the door open to **Angular SSR** for the moment when content-oriented applications make SEO a concrete priority. In the current state (SEO not a priority) it starts in SPA mode, without precluding that evolution.

**Complementary frontend technologies:**

| Concern | Technology | Notes |
|---|---|---|
| Language | TypeScript | Static typing end-to-end |
| UI components | Angular Material (or PrimeNG) | Consistent, accessible component library; themed via Qavo tokens (see §5.7) |
| Theming engine | CSS custom properties (design tokens) | Runtime theme switching without rebuild (see §5.7) |
| State (when needed) | Native signals; NgRx only for complex state | Avoid over-engineering |
| API client generation | openapi-generator | TypeScript client generated from the backend OpenAPI |
| Testing | Jasmine/Karma or Jest, Playwright/Cypress | Unit + end-to-end |
| Build | Angular CLI (esbuild/Vite) | Official toolchain |

### 2.3 Stack coherence

The Spring + Angular combination is not accidental: they share a mental model (DI, layered structure, annotations/decorators, modules) and produce a homogeneous stack that is easy to understand and maintain. The contract between the two sides is the **OpenAPI specification**: the backend exposes it (springdoc), and the frontend generates its typed client from it (openapi-generator). The contract thus becomes the single source of truth, and any mismatch surfaces at compile time.

---

## 3. The Qavo platform model

The core of the architecture is the distinction between **Platform** (Qavo) and **Applications**.

```
┌────────────────────────────────────────────────────────────────┐
│                          QAVO                                  │
│              (versioned, evolves independently)                │
│                                                                │
│   Backend (org.qavo)              Frontend (@qavo)             │
│   ┌──────────────────┐             ┌──────────────────┐        │
│   │ qavo-core        │             │ @qavo/core       │        │
│   │ qavo-security    │             │ @qavo/ui         │        │
│   │ qavo-theming     │             │ @qavo/theming    │        │
│   │ qavo-bom         │             │ @qavo/http       │        │
│   │  + plugins ...   │             │  + plugins ...   │        │
│   └──────────────────┘             └──────────────────┘        │
└───────────────┬──────────────────────────────┬─────────────────┘
                │   referenced by version      │
   ┌────────────┼──────────────┐    ┌──────────┼──────────────┐
   ▼            ▼              ▼    ▼          ▼              ▼
┌──────┐    ┌──────┐      ┌──────┐ ┌──────┐ ┌──────┐     ┌──────┐
│ App1 │    │ App2 │      │ App3 │ │ App1 │ │ App2 │     │ App3 │
│ (BE) │    │ (BE) │      │ (BE) │ │ (FE) │ │ (FE) │     │ (FE) │
└──────┘    └──────┘      └──────┘ └──────┘ └──────┘     └──────┘
   each app imports core + only the plugins it needs
```

**Qavo** is a set of versioned libraries (Maven modules on the backend, npm packages on the frontend) that contain all the cross-cutting behavior. It is not an executable application: it is reusable code distributed through public artifact repositories.

**The Applications** are the concrete projects you build. Each declares a dependency on a specific version of Qavo, imports the `core` plus the capability plugins it needs, automatically inherits all the common concerns, and focuses exclusively on its own domain logic.

**Naming and coordinates:**

- Source repositories: `qavo-be` and `qavo-fe` on GitHub.
- Backend artifacts: Maven Central, groupId `org.qavo` (e.g. `qavo-bom`, `qavo-core`, `qavo-starter-web`, `qavo-auth-login`).
- Frontend artifacts: public npm registry, scope `@qavo` (e.g. `@qavo/core`, `@qavo/ui`, `@qavo/theming`, `@qavo/auth-login`).

### 3.1 Open source distribution

Qavo is open source, versioned on **GitHub**, with artifacts published to public repositories so that they are freely accessible.

- **Source code:** public GitHub repositories (`qavo-be`, `qavo-fe`). They host the code, the issue tracker, the documentation, and the releases.
- **Backend artifacts:** published to **Maven Central** (via the Sonatype/Central publishing workflow). Public release requires a verified `groupId` (`org.qavo`, verified against the `qavo.org` domain via a DNS TXT record), GPG-signed artifacts, and the metadata mandated by Central (sources JAR, Javadoc JAR, license, SCM links). Once published, a version is immutable and globally available without authentication.
- **Frontend artifacts:** published to the **public npm registry** under the `@qavo` scope. Public packages are installable by anyone with no authentication.
- **Container images (optional):** the reference application and any tooling images can be published to a public registry such as GitHub Container Registry (GHCR).

Because the artifacts are public and immutable, applications — yours or third parties' — can depend on any released version directly from the standard public registries, with no private infrastructure required.

**Open source hygiene.** A public project carries obligations that the architecture treats as first-class: a clear **license** (e.g. Apache-2.0 or MIT), a `CONTRIBUTING` guide, a `CODE_OF_CONDUCT`, a `SECURITY` policy for responsible vulnerability disclosure, semantic-versioned releases with changelogs, and CI that runs on pull requests. Secrets and credentials must never enter the repository; publishing credentials (Central tokens, npm tokens, GPG keys) live only in the CI secret store.

### 3.2 Versioning strategy

Qavo adopts **Semantic Versioning** (`MAJOR.MINOR.PATCH`) with a clear contract toward applications:

- **PATCH** (`1.2.0 → 1.2.1`): bug fixes, backward-compatible. Safe and encouraged upgrade.
- **MINOR** (`1.2.1 → 1.3.0`): new backward-compatible features. Safe upgrade; new features are optional.
- **MAJOR** (`1.3.0 → 2.0.0`): incompatible changes. Requires action and is accompanied by a migration guide.

On the backend, a **BOM (Bill of Materials)** — `qavo-bom` — is published: a Maven POM that pins consistent versions of all Qavo modules (core and plugins) and their transitive dependencies. The application imports the BOM and does not have to manage the versions of individual libraries by hand — it only declares which Qavo version it uses, then picks the modules it wants without versions.

Each MAJOR version retains support for a defined period even after the next one is released, so that applications have a reasonable window to migrate without pressure. Releases are published from GitHub (tag-triggered CI) to the public registries.

---

## 4. Backend layered architecture

Each backend application follows a clean stratification. Qavo provides the cross-cutting foundations; the application fills the layers with its own logic.

```
┌──────────────────────────────────────────────────────────┐
│  PRESENTATION LAYER  (REST Controllers)                  │
│  - Exposes endpoints, receives DTOs, returns DTOs        │
│  - No business logic                                     │
├──────────────────────────────────────────────────────────┤
│  APPLICATION LAYER  (Services)                           │
│  - Use-case orchestration, transactions                  │
│  - Coordinates domain and infrastructure                 │
├──────────────────────────────────────────────────────────┤
│  DOMAIN LAYER  (Entities, Value Objects, rules)          │
│  - Domain model and business invariants                  │
│  - Independent of frameworks and persistence             │
├──────────────────────────────────────────────────────────┤
│  INFRASTRUCTURE LAYER  (Repositories, external clients)  │
│  - DB access, external services, messaging               │
└──────────────────────────────────────────────────────────┘
        ▲
        │ cross-cutting across all layers
┌───────┴──────────────────────────────────────────────────┐
│  CROSS-CUTTING (provided by Qavo core + plugins)         │
│  Security · Errors · Logging · Validation · Theming ·... │
└──────────────────────────────────────────────────────────┘
```

The dependency rule is one-directional: upper layers know about lower ones, never the reverse. The domain depends on nothing, so it stays pure and testable.

**Package convention** imposed by Qavo (example):

```
com.mydomain.<app>
├── api          → Controllers, DTOs, mappers (presentation)
├── application  → Services, use cases (application)
├── domain       → Entities, value objects, rules (domain)
└── infrastructure
    ├── persistence  → Repositories, JPA entities
    └── client       → Clients to external services
```

---

## 5. Cross-cutting concerns managed by Qavo

This is the central section: each common concern is handled once in Qavo and inherited by the applications.

### 5.1 Routing

**Backend.** Qavo defines REST routing conventions: a common prefix (e.g. `/api/v1`), plural resource-based naming, standard mapping of HTTP verbs (GET/POST/PUT/PATCH/DELETE), and consistent handling of pagination and sorting on collection data. Applications only declare their own controllers, inheriting the prefix and the conventions. Plugins contribute their own routes under reserved namespaces (e.g. `/api/v1/auth/**` for the login plugin).

**Frontend.** Qavo provides an Angular routing skeleton with: standard route guards for authentication and authorization (see §5.5), feature/module lazy-loading handling, a fallback route for not-found pages, and centralized page-title management. The application adds its own routes by nesting them into the skeleton. Plugins expose their routes as lazy-loadable Angular routes that the application mounts where it wants.

**API versioning strategy.** All Qavo-based APIs adopt path-based versioning as the enforced convention: the version segment is part of the URL (`/api/v{n}/resource`), never in a header or query parameter. The current version prefix (e.g. `/api/v1`) is configured once in the core and inherited by all controllers. When a version is deprecated, the core injects standard HTTP deprecation signals — the `Deprecation` and `Sunset` headers (RFC 8594) — so clients have machine-readable notice before removal. Breaking changes always require a MAJOR version bump both in the API path and in the Qavo artifact version, keeping the two versioning axes aligned. This prevents "API sprawl" across applications and makes the contract visible at a glance in every request.

### 5.2 Error handling

The goal is for **every application to return errors in the same format**, handled in a single place.

**Backend.** Qavo provides a global exception handler (`@RestControllerAdvice`) that catches all exceptions and translates them into a standard response format, aligned with the **RFC 9457 (Problem Details for HTTP APIs)** standard:

```json
{
  "type": "https://errors.mydomain.info/validation",
  "title": "Validation error",
  "status": 400,
  "detail": "One or more fields are invalid",
  "instance": "/api/v1/resources/42",
  "timestamp": "2026-05-23T10:15:30Z",
  "traceId": "a1b2c3d4",
  "errors": [
    { "field": "email", "message": "Invalid email format" }
  ]
}
```

Qavo also defines a base exception hierarchy (e.g. `BusinessException`, `ResourceNotFoundException`, `ValidationException`) that applications extend for their own specific cases. The `traceId` links the error to the log entries (see §5.3).

**Frontend.** An **HTTP interceptor** centralizes the handling of error responses: it interprets the Problem Details format, shows consistent user notifications (toast/dialog — themed, see §5.7), redirects to login on 401, shows the access-denied page on 403, and logs the error with the `traceId` for end-to-end tracing.

### 5.3 Logging and observability

**Backend.** Qavo configures structured logging (JSON format) via SLF4J/Logback, automatically enriched with an **MDC (Mapped Diagnostic Context)** that propagates `traceId`, `spanId`, user identifier, and application name on every log entry. This contract is **enforced, not optional**: applications cannot disable or bypass the structured logging pipeline, and the minimum MDC fields (`traceId`, `appName`, `userId`) are always present. It integrates **Micrometer** for metrics and **OpenTelemetry** export, so that logs are correlatable with traces and metrics. It exposes the **Spring Boot Actuator** endpoints (health, metrics, info) in a standard and secure way.

**Standard metrics set.** Beyond application-specific metrics, Qavo enforces a baseline set of operational metrics on every application: HTTP request latency per endpoint (p50/p95/p99), HTTP error rate (4xx and 5xx separately), JVM memory and thread pool usage, database connection pool saturation, and outbound HTTP client latency. These are configured in the core and feed directly into Grafana dashboards without per-application setup.

**Trace propagation enforced.** Every inbound request receives a `traceId` (generated by the core if not already present in an incoming `traceparent` header). Every outbound HTTP call made through the Qavo HTTP client propagates the `traceId` via the W3C `traceparent` header. Every error response includes the `traceId` in the Problem Details body (§5.2). This chain is non-optional: end-to-end traceability is a core guarantee.

This integrates directly with a Grafana-based observability infrastructure: Micrometer/Prometheus metrics feed the dashboards, and structured logs can be collected (e.g. Loki) and correlated via `traceId`.

**Frontend.** Qavo provides a logging service that captures unhandled errors (through a global Angular `ErrorHandler`), structures them, and — optionally — sends them to a backend collection endpoint, always correlated via `traceId`.

### 5.4 Form and field validation

**Backend.** Validation is based on **Bean Validation (Jakarta Validation)**: inbound DTOs are annotated (`@NotNull`, `@Email`, `@Size`, etc.) and validated automatically by the framework. Qavo provides **reusable custom validation constraints** (e.g. fiscal code, VAT number, domain-specific formats) and ensures that violations are translated into the standard error format from §5.2.

**Frontend.** Validation lives in Angular **Reactive Forms**. Qavo provides a library of **reusable validators** mirroring those on the backend (same logic, same messages), standardized form components that display errors consistently (and themed, see §5.7), and a mechanism to reconcile validation errors returned by the backend with the form fields. The principle is **dual validation**: the client provides immediate feedback, the server remains the authoritative source of truth.

### 5.5 Authentication and authorization

Rather than binding to a single Identity Provider, the architecture exposes a **pluggable authentication abstraction**. Each application chooses and configures the authentication strategy it needs, while the authorization model and the security context stay uniform across all strategies.

**Available strategies:**

1. **Local authentication (default, out of the box).** Qavo ships with a complete DB-backed authentication strategy: users, credentials (with strong password hashing, e.g. BCrypt/Argon2), roles, and permissions stored in standard tables. This lets an application be fully functional from day one with no external dependency. Qavo provides the schema (via Flyway migrations), the user/role management services, and the Spring Security configuration.

2. **External Identity Provider via OIDC/OAuth2.** For applications that integrate with a corporate or shared IdP, Qavo configures Spring Security as an **OAuth2/OIDC client and Resource Server** against any standard provider — **Microsoft Entra ID**, **Keycloak**, or any compliant OIDC/OAuth2 provider. The application supplies only the provider-specific configuration (issuer URI, client ID, scopes); Qavo handles token validation (signature, expiry, issuer), claim and role mapping, and session/token lifecycle.

3. **Hybrid.** An application may enable local authentication and one or more external providers simultaneously (e.g. local admin accounts plus corporate SSO), all converging on the same authorization model.

```
                ┌────────────────────────────────────────────┐
                │      Qavo Authentication Abstraction       │
                │  (uniform SecurityContext: user, roles,    │
                │   permissions — independent of strategy)   │
                └───────────────┬───────────┬───────────┬────┘
                                │           │           │
              ┌─────────────────┘           │           └─────────────────┐
              ▼                             ▼                             ▼
    ┌──────────────────┐         ┌──────────────────┐         ┌──────────────────┐
    │  Local (DB)      │         │  Entra ID (OIDC) │         │  Keycloak (OIDC) │
    │  default         │         │  configurable    │         │  configurable    │
    └──────────────────┘         └──────────────────┘         └──────────────────┘
                                         (or any standard OIDC/OAuth2 provider)
```

**Authorization** is handled uniformly regardless of the chosen authentication strategy. It is declarative: at the endpoint level (rules in the security configuration) and at the method level (`@PreAuthorize` with expressions over roles and permissions). Qavo exposes a uniform access point to the security context (current user, roles, tenant), so that applications never have to reimplement it and so that swapping the authentication strategy does not touch business code.

Note the relationship with the plugin model (§6): the authentication *strategy* lives in the security core, while user-facing flows like **registration** and **login UI** are delivered as separate, optional plugins (`qavo-auth-login`, `qavo-auth-registration` / `@qavo/auth-login`, `@qavo/auth-registration`). An application that authenticates purely via corporate SSO can skip the registration plugin entirely.

**Frontend.** Qavo abstracts authentication on the client side as well: a local login flow for the default DB strategy, an OIDC flow (Authorization Code Flow with PKCE) for external providers, an HTTP interceptor that injects credentials/token into backend calls regardless of strategy, route guards, and a structural directive to show/hide UI elements based on the user's permissions. The application selects and configures the strategy; the rest of the UI code is unaware of which one is active.

> **Security note.** Whatever the strategy, credentials and tokens are handled securely: passwords are never stored in plaintext (only strong one-way hashes), tokens are validated server-side, and account creation/credential management for external providers stays within the respective IdP. Qavo does not weaken these guarantees regardless of configuration.

**Secure HTTP headers (enforced by default).** Qavo configures a strict set of HTTP security headers on every response, active from the first request with no application-side action required: `Strict-Transport-Security` (HSTS, max-age 1 year, includeSubDomains), `Content-Security-Policy` (restrictive default, overridable per application), `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, and `Permissions-Policy` disabling unnecessary browser features. Applications can tighten these headers; loosening them requires an explicit override and is discouraged.

**TLS enforcement.** Qavo configures Spring to recognize that it is running behind a TLS-terminating reverse proxy (`server.forward-headers-strategy=framework`), ensuring that all generated URLs, redirects, and the HSTS header reflect the HTTPS scheme correctly. Applications must never generate `http://` absolute URLs in production.

**Input sanitization strategy.** Qavo enforces a layered input sanitization approach: Bean Validation constraints are applied at the DTO boundary (declared, not optional — see §5.4); raw SQL is never permitted (all data access goes through JPA/repository abstractions); and output encoding is the responsibility of the rendering layer (Thymeleaf escapes by default on the server, Angular's template engine escapes by default on the client). There is no single "sanitize everything" filter, which is intentionally avoided as it creates false confidence — instead, the policy is enforced structurally at each layer.

> **Out of scope — enterprise secrets management.** Integration with external secrets vaults (HashiCorp Vault, Azure Key Vault, AWS Secrets Manager) and application-level encryption-at-rest are intentionally excluded from the current implementation. Both are important concerns in enterprise contexts, but introducing a vault abstraction at this stage would add significant complexity — a dependency on vault infrastructure, token lifecycle management, and a pluggable secret provider interface — that exceeds the benefit for a platform of this scope. The adopted policy is: secrets are supplied via environment variables (never hardcoded), and encryption at rest is delegated to the infrastructure layer (database and disk encryption configured at the hosting level). This boundary may be revisited when a concrete application requirement justifies it.

### 5.6 Other baseline cross-cutting concerns

Beyond those above, Qavo core standardizes:

**Internationalization (i18n).** Centralized message bundle management; applications provide locale-specific files following the Qavo naming convention and the core handles locale resolution and fallback.

**Configuration convention.** Qavo enforces a naming convention for configuration properties: all platform parameters are under the `qavo.*` namespace, all application-specific parameters are under `app.*`. This keeps the two concerns visually and structurally separated in every `application.yml`. Spring profiles (`dev`, `test`, `prod`) are the standard mechanism for environment-specific values; secrets are always supplied via environment variables, never hardcoded in property files. A dedicated `qavo.yml` reference document lists every supported `qavo.*` property, its type, default, and override semantics.

> **Out of scope — pluggable config providers.** Integration with external configuration services (Spring Cloud Config Server, AWS Parameter Store, Azure App Configuration) is intentionally excluded from the current implementation. These services add operational infrastructure (a config server to run and secure, credential management, cache invalidation) whose cost exceeds the benefit at this scale. Applications that need dynamic remote configuration can integrate Spring Cloud Config independently, without conflicting with Qavo conventions.

**Feature flags.** Qavo provides a lightweight feature flag mechanism based on `@ConditionalOnProperty` and a dedicated `qavo.features.*` namespace. Flags are declared in configuration and evaluated at startup (static flags) or at request time via a `FeatureFlagService` bean (dynamic flags backed by a simple DB table or property source). This allows features to be enabled or disabled per environment without a rebuild. The mechanism is intentionally minimal — it is not a replacement for a full feature management platform (LaunchDarkly, Unleash) — but it covers the common case of environment-aware feature rollout.

**Auditing.** Automatic created/modified tracking via Spring Data Auditing — `createdBy`, `createdAt`, `lastModifiedBy`, `lastModifiedAt` — populated from the active security context with no application code required.

**Pagination and filtering.** Uniform request/response contract for paginated collections: `page`, `size`, `sort` query parameters on the request; a standard envelope (`content`, `totalElements`, `totalPages`, `number`) on the response. Filtering follows a consistent query parameter convention documented in the API reference.

**API documentation.** Automatic OpenAPI 3 generation via springdoc-openapi, with a consistent info block (title, version, contact, license) populated from Qavo core configuration. Every application gets a Swagger UI at a standard path.

**CORS.** Centralized, secure cross-origin policy configured in the core. The default policy is restrictive (same-origin only); applications explicitly declare the allowed origins for their deployment context.

**Health checks and readiness.** Standard Actuator endpoints (`/actuator/health`, `/actuator/health/readiness`, `/actuator/health/liveness`) secured and exposed consistently, suitable for container orchestration probes.

### 5.9 Resilience

Resilient communication patterns are part of the core, not an application responsibility. Without a standard baseline, each application invents its own retry logic, chooses its own timeouts, and handles downstream failures inconsistently — the result is unpredictable behavior under load and a maintenance burden that compounds across applications. Qavo addresses this by shipping sensible defaults for the three foundational resilience patterns, all based on **Resilience4j** (the standard for Spring Boot 3.x; Hystrix is deprecated and unsupported).

**Timeouts.** Every outbound HTTP call made through the Qavo HTTP client has a default timeout configured in the core (connect timeout and read timeout, both conservative defaults). Database connection acquisition also has a default pool timeout. Applications can override these per use case via `qavo.resilience.http.timeout` and `qavo.resilience.db.connection-timeout`; what they cannot do is accidentally omit a timeout entirely, which would make the application vulnerable to cascading hangs under downstream slowdowns.

**Retry policy.** Transient failures on outbound HTTP calls (network blips, 503 responses, connection resets) are retried automatically with **exponential backoff and jitter**: up to 3 attempts by default, with initial interval 200ms, multiplier 2, and random jitter to avoid thundering-herd effects. Retries are not applied to non-idempotent calls (POST by default) unless the application explicitly opts in. The retry configuration is tunable per named client via `qavo.resilience.retry.*`.

**Circuit breaker.** A Resilience4j circuit breaker wraps outbound HTTP clients, opening when the failure rate exceeds a configurable threshold (default: 50% over a 10-call sliding window). An open circuit fails fast — immediately returning an error without attempting the call — which prevents thread exhaustion and cascading failures. The circuit transitions to half-open after a configurable wait (default: 30 seconds) to probe recovery. Circuit breaker state is exposed as a Micrometer metric, visible in Grafana dashboards alongside the standard metrics set (§5.3). Configuration is via `qavo.resilience.circuit-breaker.*`.

The three patterns compose naturally: a call first checks the circuit breaker (if open, fail fast), then attempts the call with a timeout, and on transient failure applies the retry policy. This layered behavior is wired by the Qavo HTTP client and requires no application-side code.

> **Inter-service contract.** When a Qavo application calls another Qavo application, the HTTP client automatically propagates the `traceId` (§5.3) and the current authentication token (§5.5), applying the same resilience policies. The service-to-service authentication convention is: forward the caller's token (token propagation) for user-initiated flows; use a dedicated service account token for background/system flows. This convention is documented and enforced by the HTTP client configuration, not left to individual developers.

### 5.7 Theming

Theming is a first-class, centralized concern: a single theming engine governs **every** visual aspect of an application, so that look-and-feel is consistent, switchable at runtime, and extensible per application without touching component code.

**Built-in themes.** Qavo ships two complete themes out of the box — a **light** theme and a **dark** theme — both fully realized and accessible (adequate contrast ratios, WCAG-aware). An application gets both for free and can let users switch between them at runtime (with respect for the OS-level `prefers-color-scheme` preference as the default).

**Custom themes.** Applications can define their own themes, either by overriding a subset of the design tokens of a built-in theme (the common case — e.g. just brand colors) or by authoring a brand-new theme from scratch. Themes are additive and registered with the theming engine; switching theme is a runtime operation, never a rebuild.

**Full visual coverage.** A theme is not just a color palette — it governs the entire visual surface through a structured set of **design tokens**. The theming engine covers at least:

- **Surfaces and backgrounds:** page background, panel/card surfaces, elevated surfaces, overlays/scrims.
- **Content colors:** primary/secondary/disabled text, links, icons, on-surface colors for each surface.
- **Brand and accent colors:** primary, secondary, tertiary, and their hover/active/focus/disabled variants.
- **Components and widgets:** buttons (all variants), inputs and form fields, selects, checkboxes/radios/toggles, tables, tabs, menus, chips, badges, tooltips, progress indicators.
- **Modals and layering:** dialogs, drawers, popovers, the backdrop/scrim, and the elevation/shadow scale and z-index layering.
- **Borders, radii, dividers:** border colors and widths, corner radius scale, divider/separator styles.
- **Status and feedback:** semantic colors for **error**, warning, success, and info — applied consistently to alerts, toasts/snackbars, inline field-error messages (tying into §5.4), banners, and validation states.
- **Typography:** font families, the type scale (sizes/weights/line-heights), and letter spacing.
- **Spacing and layout:** the spacing scale, container widths, and breakpoints.
- **Motion:** transition durations and easing curves (with a reduced-motion variant for accessibility).
- **Focus and states:** focus-ring style, hover/active/selected/disabled states across interactive elements.

**Technical design.** Theming is implemented with **CSS custom properties (design tokens)** as the single source of truth, exposed by `@qavo/theming`. Tokens are defined once, consumed by the Qavo UI components (`@qavo/ui`) and by the chosen component library (Angular Material / PrimeNG are themed through the same tokens), and overridable by applications. Because tokens are CSS variables, switching theme (light ↔ dark ↔ custom) is instantaneous and requires no recompilation. The token contract is **versioned as part of the Qavo API**: adding tokens is a MINOR change; renaming or removing a token is a MAJOR change with a migration note — this protects custom application themes from silent breakage.

```
@qavo/theming  ──exposes──►  design tokens (CSS variables)
        │                          ▲              ▲
        │ defines                  │ consume      │ override / extend
        ▼                          │              │
  light theme  ┐            @qavo/ui      application
  dark theme   ┘ built-in   components +    custom theme(s)
                            Material/PrimeNG
```

> **Backend note.** Theming is primarily a frontend concern, but Qavo allows the *active theme and the application's available custom themes* to be persisted as a user preference (via the local-auth user profile or an app setting), so a user's choice follows them across sessions and devices.

### 5.10 Domain boundary guidelines

Qavo does not prescribe how applications decompose their domain — that decision belongs to the application and its context. What the platform provides is a **lightweight set of guidelines** that prevent the two most common failure modes: the unstructured monolith that grows without boundaries, and the premature microservices split that adds distributed-systems complexity before it is warranted.

**Default stance: modular monolith.** The recommended starting point for any Qavo-based application is a **modular monolith**: a single deployable unit whose internal structure is organized into clearly bounded modules, each owning its domain, its data, and its API surface. Modules communicate through well-defined interfaces (service boundaries, not direct repository access across modules), which preserves the option to extract a module into a separate service later if operational reasons justify it. The package convention in §4 (`api / application / domain / infrastructure`) applies within each module.

**When to consider microservices.** Splitting into separate deployable services is a deliberate architectural decision, not the default. It is appropriate when: a module has significantly different scaling requirements from the rest; independent deployment cadence is operationally required; or team ownership boundaries make a shared codebase genuinely impractical. The cost — distributed tracing, network latency, eventual consistency, independent deployment pipelines — must be weighed against the benefit. Qavo's observability (§5.3) and resilience (§5.9) foundations reduce that cost, but do not eliminate it.

**Module ownership.** Each module has a declared owner (a team or, for a solo project, an explicit responsibility boundary). Cross-module changes require coordination with the owning party. This applies even within a monolith: the ownership boundary is what prevents accidental coupling and makes future extraction feasible.

**What Qavo enforces structurally.** The package convention and the layering rule (§4) are the only structurally enforced domain guidelines. Everything else — module granularity, service boundaries, ownership assignment — is a design decision documented in the application's own architecture notes, following these principles as guidance.

### 5.8 Mobile-first and responsive design

Qavo is **mobile-first by default**, while delivering a first-class experience on every device class. This is a centralized concern: the responsive behavior is built into the Qavo UI layer and the theming tokens, so applications inherit it rather than re-solving it each time.

**Mobile-first as a design method.** Layouts, components, and interactions are designed for the smallest viewport first, then *progressively enhanced* for larger screens. Styling starts from the mobile baseline and adds complexity at higher breakpoints (min-width media queries), which keeps the base experience lean and fast on constrained devices and avoids the trap of designing for desktop and cramming it onto a phone.

**One codebase, all devices.** A single application renders correctly and ergonomically across phones, tablets, and desktops — there is no separate "mobile site" or parallel build. The same Angular components adapt their layout to the available space.

**What Qavo standardizes:**

- **A shared breakpoint scale** exposed as design tokens (§5.7), so every application and plugin uses the same responsive thresholds (e.g. handset / tablet / desktop / wide) instead of inventing its own.
- **Responsive layout primitives** in `@qavo/ui` (responsive grid, container, stack, and adaptive navigation patterns such as a bottom/hamburger navigation on mobile that becomes a side nav or top bar on desktop).
- **Fluid typography and spacing** driven by the token scale, so text and spacing adapt across viewports without per-screen overrides.
- **Touch-first ergonomics:** adequate touch-target sizes, touch and pointer event support, and interaction patterns that work for both touch and mouse/keyboard.
- **Responsive data presentation:** patterns for tables and data-dense views that gracefully degrade on small screens (e.g. column collapsing or card layouts), since data-heavy back-office applications are a primary use case.
- **Accessibility across devices:** respecting OS-level settings (text scaling, reduced motion, color scheme) and keeping the layout usable at high zoom levels.

**Relationship with SSR.** As noted in §2.2, the platform starts as an SPA and keeps Angular SSR available for when content-oriented applications need SEO. The mobile-first, responsive approach is independent of that choice and applies in both modes.

> Practically, an application built on Qavo gets responsive, mobile-first behavior "for free" by composing Qavo UI primitives and using the breakpoint tokens; it only adds custom responsive rules where it has genuinely bespoke layout needs.

---

## 6. Modularity and the plugin model

Qavo is **modular on both layers**. Beyond the mandatory `core`, every additional capability — registration, login UI, user self-management, audit console, file storage, notifications, and so on — is delivered as a **separate, independently versioned plugin**. An application imports the `core` plus only the plugins it needs.

```
        Qavo core  (always present)
              │
   ┌──────────┼───────────┬───────────────┬─────────────┐
   ▼          ▼           ▼               ▼             ▼
auth-login  auth-      user-mgmt      notifications   storage   ... more
            registration                                         plugins
   │          │
   └────┬─────┘  an application imports ONLY what it needs:
        ▼
   App "Catalog"  →  core + theming + auth (OIDC)            (no registration)
   App "Portal"   →  core + theming + auth-login + auth-registration + user-mgmt
```

**Backend.** Each plugin is its own Maven module under `org.qavo` (e.g. `qavo-auth-login`, `qavo-auth-registration`, `qavo-user-mgmt`). Adding a plugin to an application is declaring one dependency; the plugin **auto-configures itself** through Spring Boot's conditional mechanisms (`@ConditionalOnClass`, `@ConditionalOnProperty`), ships its own Flyway migrations for any tables it needs, and contributes its own routes, services, and security rules. Removing a plugin is removing the dependency — its code, tables-ownership, and endpoints leave with it.

**Frontend.** Each plugin is its own npm package under `@qavo` (e.g. `@qavo/auth-login`, `@qavo/auth-registration`). Plugins are imported and registered with a provider, expose lazy-loadable routes and components, and consume the same theming tokens (§5.7) so they look native to the host application with no extra styling.

### 6.1 Design decision — plugin modules vs all-in-core-with-flags

A key requirement is whether capabilities like registration and login should be **selectable plugins the application imports**, or **all built into the core and switched on via configuration**. This was evaluated explicitly on the criteria of maintainability and long-term evolution, and the architecture adopts the **plugin** approach. The rationale:

**Why "everything in core, toggled by flags" degrades over time.** Putting every capability inside the core and enabling it with `enabled: true/false` looks convenient, but it ages badly. Every application carries the code of features it does not use — transitive dependencies, tables, endpoints, and attack surface included. A vulnerability in the registration code becomes a concern even for an application that never exposes registration. Worse, the core turns into a collector of unrelated responsibilities that grows without bound, and every core release touches code for unrelated features, raising regression risk and slowing exactly the independent evolution that is the architecture's cardinal requirement. This is the classic "God module" anti-pattern.

**Why plugins win on maintainability.** With separate modules, each capability has its own lifecycle, versioning, tests, and dependencies. An application declares only what it uses, so footprint and attack surface stay minimal. A bug in the registration plugin is fixed and released *without touching* the core or the applications that don't use it. The module boundary forces a clean API between core and plugin, keeping the architecture decoupled by construction. It is the same model by which Spring Boot ships its starters and Angular ships its libraries — and the same pattern already chosen for Qavo core.

**The adopted synthesis — modularity for distribution, configuration for behavior.** The two approaches are not truly in conflict if applied at the right granularity:

- **Modularity is the unit of distribution and existence** (coarse-grained, decided at build time): the application chooses *which plugins to import*. What is not imported does not exist in the app.
- **Configuration is the unit of activation and behavior** (fine-grained, runtime): once a plugin is imported, its behavior is tuned via `application.yml` / providers — which fields appear on the registration form, password policy, whether self-service registration is currently open, and so on.

This takes the best of both: the small footprint, isolation, and independent release cadence of plugins, plus the runtime flexibility of configuration — without the monolith's downsides. On the backend it rests naturally on Spring Boot's conditional auto-configuration; on the frontend on Angular's provider-based module registration.

**Consequence for the platform.** The core stays small and stable, with a slow, careful release cadence. Plugins evolve faster and independently. Applications compose exactly the platform they need and upgrade each piece on their own schedule — which is precisely the independent-evolution principle of §1 applied at the capability level.

---

## 7. Integration with applications

The target experience is: *declare the Qavo version, import the core plus the plugins you need, follow a few conventions, and all the common behavior is already there.*

### 7.1 Backend — onboarding a new application

1. **Declare the BOM and the modules.** Import `qavo-bom`, then declare the core starter plus the plugins you want. Versions come from the BOM. Since artifacts are on Maven Central, no repository configuration is needed:

   ```xml
   <dependencyManagement>
     <dependencies>
       <dependency>
         <groupId>org.qavo</groupId>
         <artifactId>qavo-bom</artifactId>
         <version>1.1.0</version>
         <type>pom</type>
         <scope>import</scope>
       </dependency>
     </dependencies>
   </dependencyManagement>

   <dependencies>
     <!-- always -->
     <dependency>
       <groupId>org.qavo</groupId>
       <artifactId>qavo-starter-web</artifactId>
     </dependency>
     <!-- optional plugins, only what this app needs -->
     <dependency>
       <groupId>org.qavo</groupId>
       <artifactId>qavo-auth-login</artifactId>
     </dependency>
     <dependency>
       <groupId>org.qavo</groupId>
       <artifactId>qavo-auth-registration</artifactId>
     </dependency>
   </dependencies>
   ```

2. **Configure the application-specific values.** In `application.yml`, provide only the values specific to the application: the auth strategy, any plugin behavior, and the active/default theme. Qavo's auto-configuration applies everything else.

   ```yaml
   qavo:
     auth:
       strategy: local          # local | oidc | hybrid
       # for oidc / hybrid:
       # oidc:
       #   issuer-uri: https://login.microsoftonline.com/<tenant>/v2.0
       #   client-id: <client-id>
     registration:              # behavior of the imported plugin
       self-service: true
       require-email-verification: true
     theming:
       default-theme: system    # light | dark | system | <custom-theme-id>
       allow-user-switch: true
   ```

3. **Develop the domain logic.** Follow the package conventions (§4) and write controllers, services, and entities. Security, error handling, logging, base validation, theming, and the imported plugins are already active with no further code.

Auto-configuration follows the Spring Boot principle of **sensible defaults with possible override**: Qavo imposes reasonable defaults, but every aspect can be overridden by the application when a different behavior is needed.

### 7.2 Frontend — onboarding a new application

1. **Install the packages.** From the public npm registry — core, UI, theming, plus the plugins:

   ```bash
   npm install @qavo/core @qavo/ui @qavo/theming @qavo/http
   npm install @qavo/auth-login @qavo/auth-registration   # only if needed
   ```

2. **Initialize Qavo.** Register the Qavo configuration, providing the API endpoint, the auth strategy, the theme setup, and any plugins:

   ```typescript
   bootstrapApplication(AppComponent, {
     providers: [
       provideQavo({
         apiBaseUrl: '/api/v1',
         auth: { strategy: 'local' },              // 'local' | 'oidc'
         theming: {
           defaultTheme: 'system',                 // 'light' | 'dark' | 'system' | custom id
           allowUserSwitch: true,
           // customThemes: [myBrandTheme],         // optional token overrides
         },
       }),
       provideAuthLogin(),                          // plugin registration
       provideAuthRegistration({ selfService: true }),
       // application-specific providers
     ]
   });
   ```

3. **Develop the features.** Use the UI components, validators, guards, HTTP client, and themed plugin screens. The authentication, error, and logging interceptors are already active, and everything renders under the active theme.

### 7.3 API client generation

To keep the two sides aligned, the frontend's TypeScript client is **generated from the OpenAPI** exposed by the backend (openapi-generator), ideally as a CI pipeline step. This makes the API contract the single source of truth and surfaces any mismatch at compile time after a backend update. Plugins that expose endpoints contribute to the same OpenAPI document, so their clients are generated the same way.

---

## 8. Evolution and maintenance of Qavo

**Defined release cycle.** Qavo has its own planned releases. The **core** moves slowly and carefully (it is the stable foundation); **plugins** evolve faster and independently. Each release goes through the CI pipeline (build, test, static analysis, publication) and produces immutable, versioned artifacts published to the public registries.

**Backward compatibility as the default.** Within the same MAJOR line, every new version is backward-compatible. This applies to the API of each module, to the Problem Details error contract, and to the **theming token contract** (§5.7). Features to be removed are first deprecated (with annotations and warnings), remain available for at least one cycle, and only in a later MAJOR are they removed.

**Migration guides.** Each MAJOR version is accompanied by a migration document listing the incompatible changes and the steps to adapt. Release notes for MINOR and PATCH describe new features and fixes. Both live in the GitHub repositories alongside the release.

**Platform testing.** Qavo has its own test suite, including integration tests with Testcontainers. A reference application ("reference app") uses the core plus a representative set of plugins and themes, and is continuously integrated: it serves both as an onboarding example and as a check that no update breaks integration. CI runs on every pull request to protect the public artifacts.

**Controlled application upgrades.** Each application decides when to upgrade, **per module**. For PATCH and MINOR, the upgrade is a version bump and a test run. For MAJOR, the migration guide is followed. No application is ever forced to upgrade in lockstep with another, and a plugin upgrade need not drag the whole platform with it.

---

## 9. Deployment and infrastructure

Qavo integrates naturally with a self-hosted infrastructure based on Docker and a reverse proxy, while remaining deployment-agnostic.

**Containerization.** Each application (backend and frontend) is packaged as a Docker image. The Spring Boot backend produces an optimized image (layered JAR, or GraalVM native image if minimal startup time and footprint are needed). The Angular frontend is compiled into static assets served by a lightweight container (Nginx).

**Reverse proxy.** A reverse proxy (Nginx) acts as the entry point: it terminates TLS, routes traffic to the frontend and backend, and applies the common policies. It is essential that the proxy correctly forwards the `X-Forwarded-*` headers (`X-Forwarded-Proto`, `X-Forwarded-Host`) and that Spring is configured to recognize them, so that URL generation and redirects work correctly behind the proxy — an aspect Qavo can standardize by enabling `server.forward-headers-strategy=framework`.

**Perimeter authentication.** When an external IdP is used, authentication can be handled by the application (OIDC flow in the frontend) and/or at the perimeter with an authentication proxy. When the local strategy is used, authentication is handled entirely by the application against its own DB. The architecture supports both without changing the business code.

**CI/CD pipeline.** A pipeline (e.g. GitHub Actions, with tag-triggered releases) automates for each application: build, test, API client generation, Docker image build, and deploy. For Qavo, the pipeline publishes the versioned artifacts to Maven Central and the public npm registry, signing them as required.

**Production observability.** Micrometer metrics feed Grafana; structured logs are collected and correlated via `traceId`; the Actuator endpoints provide health and readiness to the orchestrator.

---

## 10. Decision summary

| Concern | Decision | Main rationale |
|---|---|---|
| Project name | Qavo (repos `qavo-be`/`qavo-fe`; `org.qavo`; `@qavo`) | Verified free on the registries that matter; clean, distinctive |
| Backend | Java 21 + Spring Boot 3.x | Enterprise standard, native coverage of cross-cutting concerns, starter pattern |
| Common BE distribution | Spring Boot Starter + BOM | Independent evolution through versioning |
| Frontend | Angular | Complete, opinionated framework: imposed standardization, aligned with the requirement |
| Common FE distribution | Versioned npm packages | Same independent-evolution model as the backend |
| BE↔FE contract | OpenAPI + generated client | Single source of truth, mismatches at compile time |
| Security | Pluggable auth + secure headers + TLS + input sanitization — all enforced by default | Secure by default; no opt-in required; Vault/encryption-at-rest deferred (out of scope) |
| Theming | Centralized token engine; built-in light + dark; custom themes per app | Full visual coverage, runtime switching, versioned token contract |
| UI / responsiveness | Mobile-first, responsive across all devices from one codebase | Lean base experience, ergonomic on every device, no separate mobile build |
| Modularity | Core + independently versioned plugins; modularity for distribution, config for behavior | Small core, minimal footprint/attack surface, independent evolution per capability |
| API versioning | Path-based (`/api/v{n}/`), `Deprecation`/`Sunset` headers, enforced by core | Consistent across apps; no API sprawl |
| Resilience | Resilience4j: timeout + retry + circuit breaker in core, sensible defaults | Consistent failure behavior; no per-app reinvention |
| Config convention | `qavo.*` / `app.*` namespaces; feature flags via `qavo.features.*` | Runtime control without rebuild; no config sprawl |
| Domain guidelines | Modular monolith as default; microservices as explicit decision | Prevents unstructured growth; preserves future options |
| Error format | RFC 9457 Problem Details | Standard, consistent, machine-readable |
| Observability | Micrometer + OpenTelemetry + Grafana | Log/metric/trace correlation via traceId |
| Versioning | Semantic Versioning + BOM | Clear contract, controlled per-module upgrades |
| Distribution model | Open source on GitHub, public artifacts (Maven Central, npm) | Freely accessible, immutable releases |
| Deployment | Docker + Nginx + CI/CD | Coherent with self-hosted infrastructure, deployment-agnostic |
| Development process | AI-assisted, with human review and accountability | Transparency; speed without lowering quality/security bar |

---

## 11. Suggested next steps

1. **Claim the names early.** Open `qavo-be` and `qavo-fe` on GitHub, publish placeholder `@qavo/core@0.0.0` on npm, and start the Maven Central namespace verification for `org.qavo` (DNS TXT on `qavo.org`) — it is the bottleneck for the first release.
2. **Define the scope of v1.0** of the core: highest-impact concerns first (local-auth security, error handling, logging, theming with the two built-in themes), with the first plugins (`auth-login`, `auth-registration`) following closely.
3. **Set up the publishing pipeline:** license, contributing/security policies, and CI configured to publish signed artifacts to Maven Central and npm.
4. **Build the "reference app":** a minimal application using core + theming + a couple of plugins, as a test bed and onboarding model.
5. **Write the onboarding guide** for backend and frontend, versioned with Qavo.
6. **Validate with a real case:** bring one of the planned applications onto Qavo and refine based on concrete experience.

---

## 12. AI-assisted development disclosure

Qavo is **designed and built with the assistance of artificial intelligence.** This is stated openly, as a matter of transparency toward users, contributors, and adopters of the platform.

**What this means in practice.** AI tools are used as part of the design and engineering process — for example to draft and review architecture, generate and refactor code, write tests and documentation, and explore design alternatives. AI assistance is a *tool* in the workflow, not a replacement for human responsibility.

**Human accountability.** Every decision, design choice, and released artifact is reviewed and owned by human maintainers. AI-generated output is treated as a draft to be verified, tested, and corrected — not as authoritative by default. The maintainers remain accountable for the security, correctness, and licensing of the code that ships.

**Quality and security implications.** Because AI-generated code can contain subtle errors, outdated patterns, or insecure constructs, Qavo applies extra diligence: human code review on all contributions, automated testing (unit and integration, see §8), static analysis and dependency scanning in CI, and careful review of anything touching the security-sensitive areas (authentication, authorization, cryptography, input validation). AI assistance does not lower the bar for these checks — if anything, it raises it.

**Licensing and provenance.** Contributions — whether human- or AI-assisted — must comply with the project's open source license and must not introduce code whose provenance or license is unclear. Contributors are expected to ensure they have the right to submit what they contribute, consistent with the `CONTRIBUTING` guidelines (§3.1).

**Contributions.** Contributors may use AI assistance, but the same standards apply to everyone: the contribution must be understood, tested, and owned by the person submitting it. The `CONTRIBUTING` guide states this expectation explicitly.

> In short: AI accelerates the work; humans remain responsible for it. This disclosure is intended to set accurate expectations and to invite scrutiny rather than to diminish it.

---

*Qavo reference architecture — version 1.3. To be kept aligned with the evolution of the platform.*
