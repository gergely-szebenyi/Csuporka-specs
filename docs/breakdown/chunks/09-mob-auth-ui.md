# Mobile authentication UI

| | |
|---|---|
| ID | `mob-auth-ui` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `api-contract` (identity operations the auth interface mirrors), `mob-design-system` (base components, tokens and assets) |
| Unlocks | `mob-auth-api` |

## Goal
Welcome, login and register screens with email and password, and the session state that drives navigation, on an in-memory stub.

## In scope
- Welcome screen: email sign-in and registration, same on iOS and Android
- Email login with wrong-credentials error
- Registration: name (shown to household members), email, password, password again; rules shown as ticks and crosses: at least 8 characters, at least one number, the two passwords match
- Session state (signed out / signed in) controlling the navigation graph; logout action
- Auth repository interface with an in-memory stub (register, login, logout, current session)

## Out of scope
- Token storage, refresh and the real API (`mob-auth-api`)
- Google and Apple sign-in (removed from v1)
- Password reset, email verification (out of v1)

## Builds on
- `mob-design-system` components; `mob-scaffolding` navigation, module layout and the repository/stub conventions
- Shared specs: `identity` (password rules, errors)

## Provides
- Auth screens, the auth repository interface and the session state that `mob-auth-api` implements for real

## Done when
- Register, login, failed login and logout flows work on the stub on Android and iOS, and session state switches the navigation graph.

## Decisions from the breakdown
- Built on hand-written in-memory stubs behind the feature's repository interfaces, with seeded Hungarian data; the data layer comes later in its `*-api` chunk (2026-11-26)
- No password reset or email verification in v1
- Email and password only (2026-11-23)
- Repository interfaces mirror the operations in `api-contract`, so the `*-api` chunk only adds DTOs and mapping (2026-11-26)

## Open questions for the deep-dive session
- Stub seed data and how to switch stub states (errors, offline, another member's change) for manual checks
