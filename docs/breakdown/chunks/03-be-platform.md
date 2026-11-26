# Backend platform utilities (security, springdoc, shared kernel)

| | |
|---|---|
| ID | `be-platform` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 2 |
| Depends on | `be-scaffolding` (modules, ProblemDetail handling, test harness, CI) |
| Unlocks | `be-auth`, `be-households`, `be-recipe-import`, `mob-network` |

## Goal
The cross-cutting backend utilities in `:common` and `:app` that every feature chunk builds on, so a feature chunk only adds its own module.

## In scope
- Spring Security filter chain: stateless JWT bearer validation (HS256, secret from `.env`), public routes (`/api/v1/auth/**`, health), 401/403 as ProblemDetail with codes
- Authenticated principal (user id from `sub`) resolvable in controllers; the `JwtDecoder` is reusable by the WebSocket handshake (`be-realtime`)
- springdoc-openapi: `/api/v1` docs, bearer security scheme, ProblemDetail error schema, conventions (stable `operationId`s, tags, Kotlin nullability = `required`/`nullable`)
- Publishing the generated spec where the mobile repo's `syncOpenApiSpec` copies it from (location and regeneration decided here)
- The error-code catalogue from the shared specs' domain-errors spec in `:common`, mapped to the ProblemDetail `code`
- The `user registered` event type in `:common` (published by `be-auth`, consumed by `be-households`), so both can be built in parallel in wave 3
- Test helpers: minting test tokens and making authenticated requests, so feature chunks test without `be-auth`
- Update the backend `AGENTS.md` sections on the contract (code-first, no `contract/openapi.yaml`)

## Out of scope
- Users, login, issuing and refreshing tokens (`be-auth`)
- Feature endpoints
- Real-time message formats (`be-realtime`)
- Rate limiting (dropped for now)

## Builds on
- `be-scaffolding` skeleton and error handling
- Shared specs (`openspec/specs/domain-errors`, `identity`) in this store

## Provides
- Security chain, principal and test helpers for every `be-*` chunk
- The generated spec and its location for `mob-network`
- Error codes for backend and mobile error mapping

## Done when
- A sample protected endpoint rejects missing, expired and invalid tokens and accepts a test-minted one; the generated OpenAPI spec is produced by the build in the agreed location; ./gradlew build passes locally and on CI.

## Decisions from the breakdown
- Code-first API contract: springdoc output is the wire contract; the spec-first YAML is dropped (2026-11-26)
- Behavioural contract lives in the shared specs (2026-11-26)
- HS256 access tokens issued by `be-auth`; modules never depend on each other, cross-module contracts live in `:common` (2026-11-23)

## Open questions for the deep-dive session
- Where the generated spec lives (committed file vs. fetched from the running app) and what regenerates it (springdoc Gradle plugin vs. a test that writes it out)
- How a breaking spec change is noticed on the mobile side (diff in the mobile repo's committed copy is the current plan)
- Whether the current household is resolved here (via a query interface) or by each module
