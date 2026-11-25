# Backend scaffolding & architecture

| | |
|---|---|
| ID | `be-scaffolding` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 1 |
| Depends on | none |
| Unlocks | `be-auth` |

## Goal
Migrate the Spring Initializr skeleton to the modular-monolith layout in `AGENTS.md`, connect to Supabase, and set up error handling, logging, events and tests that every later chunk builds on.

## In scope
- `build-logic` convention plugins, `gradle/libs.versions.toml`, modules `:app`, `:common` (business modules are added by their chunks)
- Replace `spring-boot-starter-data-jdbc` and the `io.spring.dependency-management` plugin with JPA + the Boot BOM from the catalog
- Postgres on the Supabase **session pooler** (port 5432, IPv4, prepared statements work); settings from `.env`
- Flyway with one schema per module and a first empty migration; `ddl-auto: validate`
- Global error handling: ProblemDetail with `code`
- Structured logging, request IDs, Sentry (DSN from `.env`)
- In-process events: `ApplicationEventPublisher` + Spring Modulith event publication registry (JPA, table via Flyway)
- Health endpoint (Actuator `health` only, public, status only: the one public route outside `/api/v1/auth/**`)
- Test setup: unit tests with fakes; database tests on embedded Postgres (no Docker)
- Contract check: springdoc output compared with the spec from `api-contract`
- CI: GitHub Actions workflow running `./gradlew build` on every push (Linux runner, embedded Postgres, no secrets)

## Out of scope
- Deployment, CD and hosting (not in v1)
- Any feature endpoints

## Builds on
- The existing Initializr project (`com.prekogdevs.csuporka`)

## Provides
- Runnable service skeleton, DB access, migrations, event bus, test harness used by every be-* chunk

## Done when
- Service boots locally against Supabase with a health endpoint and an empty migration; `./gradlew build` passes with an embedded-Postgres integration test, locally and on GitHub Actions.

## Decisions from the breakdown
- Supabase is only the Postgres host; no Supabase Auth or client SDKs
- Flyway, JPA, feature modules, Sentry: decided (2026-11-23)
- One Supabase project, used for local development; nothing deployed
- No Docker: embedded Postgres for database tests
- CI on GitHub Actions; Awaitility allowed for async tests (2026-11-23)

## Open questions for the deep-dive session
- Embedded Postgres library and its Postgres version (match Supabase's major version)
