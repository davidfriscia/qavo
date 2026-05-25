# Qavo

> *The foundation your applications stand on.*

**Qavo** is an open source, versioned application platform for building full-stack applications on Java / Spring Boot (backend) and Angular (frontend). It centralizes all common concerns — routing, error handling, logging, validation, authentication, authorization, theming, and responsive UI — into a shared, independently versioned platform that applications depend on without reimplementing.

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Backend](https://img.shields.io/badge/backend-qavo--be-blue)](https://github.com/davidfriscia/qavo-be)
[![Frontend](https://img.shields.io/badge/frontend-qavo--fe-blueviolet)](https://github.com/davidfriscia/qavo-fe)
[![Built with AI](https://img.shields.io/badge/built%20with-AI%20assistance-lightgrey)](#ai-assisted-development)

> **⚠️ Hobby project.** Qavo is a personal side project, developed in spare time. Its primary purpose is to explore and apply artificial intelligence to the full lifecycle of a software project — from architecture and design to code generation and documentation. It is shared openly in that spirit. There are no commitments on release schedules, support, or production readiness. Contributions and feedback are welcome, but expectations should be set accordingly.

---

## What is Qavo?

Qavo is not a single application — it is a **shared platform** that multiple applications build on. Each application imports the Qavo core plus only the capability plugins it needs, and inherits all common behavior automatically.

```
                    ┌───────────────────────────────────┐
                    │             QAVO                  │
                    │   core · security · theming       │
                    │   plugins: auth, registration...  │
                    └────────────┬──────────────────────┘
                                 │ referenced by version
              ┌──────────────────┼──────────────────┐
              ▼                  ▼                  ▼
           App 1              App 2              App 3
```

**Key properties:**

- **Centralized cross-cutting concerns** — security, errors, logging, validation, theming handled once, inherited everywhere.
- **Plugin model** — import only what you need; each capability is an independently versioned module.
- **Pluggable authentication** — local DB auth out of the box; drop-in support for any OIDC/OAuth2 provider (Entra ID, Keycloak, and others).
- **Mobile-first, responsive** — UI layer designed mobile-first, renders correctly on all devices from a single codebase.
- **Built-in theming** — light and dark themes out of the box; applications can define custom themes via design tokens.
- **Independent evolution** — applications upgrade Qavo on their own schedule; no forced lockstep.
- **Open source** — MIT licensed, artifacts on Maven Central and npm.

---

## Repositories

| Repository | Description | Artifacts |
|---|---|---|
| [`davidfriscia/qavo`](https://github.com/davidfriscia/qavo) | This repo — overview, documentation, GitHub Pages | — |
| [`davidfriscia/qavo-be`](https://github.com/davidfriscia/qavo-be) | Backend platform (Java / Spring Boot) | Maven Central `org.qavo` |
| [`davidfriscia/qavo-fe`](https://github.com/davidfriscia/qavo-fe) | Frontend platform (Angular) | npm `@qavo` |

---

## Quick start

### Backend

Add the Qavo BOM to your `pom.xml` and declare the starter:

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>org.qavo</groupId>
      <artifactId>qavo-bom</artifactId>
      <version>1.0.0</version>
      <type>pom</type>
      <scope>import</scope>
    </dependency>
  </dependencies>
</dependencyManagement>

<dependencies>
  <dependency>
    <groupId>org.qavo</groupId>
    <artifactId>qavo-starter-web</artifactId>
  </dependency>
</dependencies>
```

Configure your authentication strategy in `application.yml`:

```yaml
qavo:
  auth:
    strategy: local   # local | oidc | hybrid
```

### Frontend

Install the core packages from npm:

```bash
npm install @qavo/core @qavo/ui @qavo/theming @qavo/http
```

Bootstrap the platform in your Angular application:

```typescript
bootstrapApplication(AppComponent, {
  providers: [
    provideQavo({
      apiBaseUrl: '/api/v1',
      auth: { strategy: 'local' },
      theming: { defaultTheme: 'system' },
    }),
  ]
});
```

---

## Documentation

| Document | Description |
|---|---|
| [Architecture](docs/architecture.md) | Full reference architecture: technology choices, layers, cross-cutting concerns, plugin model, theming, mobile-first design |
| [Contributing](docs/contributing.md) | How to contribute to Qavo |
| [Security](docs/security.md) | Security policy and vulnerability reporting |

Full documentation is also available at **[qavo.org](https://qavo.org)** (GitHub Pages).

---

## Platform modules

### Backend (`org.qavo`)

| Module | Description |
|---|---|
| `qavo-bom` | Bill of Materials — pins all module versions |
| `qavo-starter-web` | Core web layer: REST conventions, error handling, logging, validation |
| `qavo-security` | Authentication abstraction: local DB + OIDC/OAuth2 |
| `qavo-auth-login` | Plugin: login endpoint and session management |
| `qavo-auth-registration` | Plugin: user self-registration |

### Frontend (`@qavo`)

| Package | Description |
|---|---|
| `@qavo/core` | Core services: HTTP client, interceptors, error handling |
| `@qavo/ui` | UI component library: responsive primitives, layout, forms |
| `@qavo/theming` | Theming engine: design tokens, light/dark themes, custom themes |
| `@qavo/http` | HTTP interceptors: auth token injection, error handling |
| `@qavo/auth-login` | Plugin: login UI and flow |
| `@qavo/auth-registration` | Plugin: registration UI and flow |

---

## Versioning

Qavo follows [Semantic Versioning](https://semver.org). Within a MAJOR version, all changes are backward-compatible. Each MAJOR release is accompanied by a migration guide.

| Change | Version bump | Safe to upgrade? |
|---|---|---|
| Bug fixes | PATCH | ✅ Always |
| New features | MINOR | ✅ Always |
| Breaking changes | MAJOR | ⚠️ Follow migration guide |

---

## AI-assisted development

Qavo is designed and built with the assistance of artificial intelligence. AI tools are used for architecture design, code generation, and documentation. Every decision and released artifact is reviewed and owned by human maintainers. See the [full disclosure](docs/architecture.md#12-ai-assisted-development-disclosure) in the architecture document.

---

## License

Qavo is released under the [MIT License](LICENSE).

Copyright (c) 2026 David Friscia