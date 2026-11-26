# Mobile auth connected to the backend

| | |
|---|---|
| ID | `mob-auth-api` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 4 |
| Depends on | `mob-auth-ui` (auth repository interface and session state), `mob-network` (HTTP client, generated DTOs, error mapping), `be-auth` (auth endpoints in the generated spec) |
| Unlocks | `mob-households-api`, `mob-recipes-api`, `mob-plan-api`, `mob-grocery-api` |

## Goal
Connect the auth UI to the backend: real auth data layer, secure token storage, automatic refresh and forced logout.

## In scope
- Auth repository implementation (register, login, refresh, logout) on the generated DTOs, replacing the stub in DI
- Access token in memory only; refresh token from the JSON body, stored encrypted (Keychain / Keystore) via `expect`/`actual` on the platform APIs
- Session restored on app start with the stored refresh token
- Ktor bearer auth: refresh on 401 (one refresh for concurrent 401s) and retry; forced logout when refresh fails
- Mapping backend auth errors (wrong credentials, email taken, password rules) to the UI's existing error states

## Out of scope
- Auth screens (`mob-auth-ui`)
- Google and Apple sign-in (removed from v1)

## Builds on
- `mob-auth-ui` interface and session state, `mob-network` client and error mapping, `be-auth` endpoints

## Provides
- The authenticated HTTP client and session every other `mob-*-api` chunk uses

## Done when
- Against the locally running backend, register, login, app restart with a stored session, token refresh and forced logout work on Android and iOS.

## Decisions from the breakdown
- Tokens in JSON bodies, no cookies; no library for secure storage unless agreed (2026-11-23)
- Verified by hand against the locally running backend; automated tests come in `mob-tests` (2026-11-26)

## Open questions for the deep-dive session
- Differences between the stub's domain models and the generated DTOs: fix in the mapper or adjust the domain model
