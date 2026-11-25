# Backend authentication

| | |
|---|---|
| ID | `be-auth` |
| Area | Backend (Spring Boot) |
| Size | L |
| Wave | 2 |
| Depends on | `api-contract` (auth endpoints and token formats), `be-scaffolding` (service skeleton, DB, error model) |
| Unlocks | `be-households`, `be-recipe-import` |

## Goal
Users, email/password registration and login, JWT access (50 min) + refresh (30 days) with rotation, per-device families and reuse detection, logout, security filter chain.

## In scope
- User model and storage (incl. display name shown to household members), BCrypt password hashing
- Register (name, email, password), login, logout
- Password rules: at least 8 characters and at least one number (validated on the server too)
- JWT access token: HS256, 50 minutes, claims `sub`, `type`, `iat`, `exp`
- Refresh token: opaque, SHA-256 hashed, 30 days, rotated on every refresh
- One refresh-token family per login/device; presenting an already-used token revokes that family only (that device is logged out)
- Spring Security filter chain and current-user resolution; token validation reused by the WebSocket handshake
- Publishing a user-registered event (be-households creates the user's household of one from it)

## Out of scope
- Account deletion (out of v1)
- Google and Apple sign-in (removed from v1)
- Email verification, password reset, change password (out of v1)
- Rate limiting (dropped for now; revisit before any real deployment)

## Builds on
- be-scaffolding skeleton and migrations

## Provides
- Authenticated principal for every other endpoint; token validation reused by be-realtime

## Done when
- All auth endpoints from the contract pass tests, including refresh rotation, reuse detection revoking only that device's family, expiry and logout.

## Decisions from the breakdown
- Conventional backend JWT auth, not Supabase Auth
- No emails in v1, so no verification or reset flows
- HS256, per-device families with reuse detection, no rate limiting (2026-11-23)

## Open questions for the deep-dive session
- Cleanup job for expired refresh tokens (schedule)
