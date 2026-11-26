# Mobile networking foundation

| | |
|---|---|
| ID | `mob-network` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `mob-scaffolding` (modules and DI), `be-platform` (the generated spec, its location and the ProblemDetail/error-code format) |
| Unlocks | `mob-auth-api` |

## Goal
The networking foundation every `mob-*-api` chunk builds on: Ktor client, DTOs generated from the backend's springdoc spec, error mapping and connectivity state.

## In scope
- Ktor `HttpClient` setup (JSON, timeouts, logging, base URL reachable from emulators and devices on the local network, port 8080)
- `syncOpenApiSpec` reworked to copy the backend's generated spec into `api/` (committed); OpenAPI Generator produces models only
- ProblemDetail → domain error mapping using the shared error codes; safe-call helper for data sources
- Connectivity monitor (Android + iOS) exposing online/offline state for screens and the offline cache
- The extension point for the bearer auth plugin (wired in `mob-auth-api`)
- Update the mobile `AGENTS.md` sections on the contract (code-first, no fakes)

## Out of scope
- Tokens and session (`mob-auth-api`)
- Feature data layers (the `mob-*-api` chunks)
- Local database and write queue (`mob-offline-cache`)
- WebSocket (`mob-realtime-client`)

## Builds on
- `mob-scaffolding` modules and DI, the domain error type
- `be-platform` generated spec and error format

## Provides
- HTTP client, generated DTOs, error mapping and connectivity state for every `mob-*-api` chunk and `mob-offline-cache`

## Done when
- DTOs generate from the backend's current generated spec; a call to the backend's health/sample endpoint succeeds and an error response maps to a domain error on Android and iOS; connectivity state flips with network on/off.

## Decisions from the breakdown
- Code-first contract: DTOs are generated from the backend's springdoc output; a feature's DTOs appear once its backend chunk is done (2026-11-26)
- Models only from OpenAPI Generator; the copied spec is committed so contract changes show up in the diff (2026-11-23)

## Open questions for the deep-dive session
- Ktor client config (engine per platform, logging in debug only)
- How DTO regeneration is triggered after each backend chunk
