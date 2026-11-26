# Backend authentication

| | |
|---|---|
| ID | `be-auth` |
| Area | Backend (Spring Boot) |
| Size | L |
| Wave | 3 |
| Depends on | `be-platform` (filter chain, error codes, `user registered` event type) |
| Unlocks | `mob-auth-api` |

## Goal
Users, email/password registration and login, JWT access (50 min) + refresh (30 days) with rotation, per-device families and reuse detection, logout.

## In scope
- User model and storage (incl. display name shown to household members), BCrypt password hashing
- Register (name, email, password), login, logout
- Password rules: at least 8 characters and at least one number (validated on the server too)
- JWT access token: HS256, 50 minutes, claims `sub`, `type`, `iat`, `exp`, validated by the `be-platform` filter chain
- Refresh token: opaque, SHA-256 hashed, 30 days, rotated on every refresh
- One refresh-token family per login/device; presenting an already-used token revokes that family only (that device is logged out)
- Publishing the `user registered` event (type from `be-platform`; `be-households` creates the household of one from it)
- Endpoints documented through springdoc, so they show up in the generated spec for `mob-auth-api`

## Out of scope
- Security filter chain and token validation (`be-platform`)
- Account deletion (out of v1)
- Google and Apple sign-in (removed from v1)
- Email verification, password reset, change password (out of v1)
- Rate limiting (dropped for now; revisit before any real deployment)

## Builds on
- `be-platform` security chain, principal and error codes
- Shared specs: `identity`

## Provides
- Users, sessions and tokens; auth endpoints for `mob-auth-api`

## Done when
- All auth endpoints pass tests, including refresh rotation, reuse detection revoking only that device's family, expiry and logout, and they appear in the generated spec.

## Decisions from the breakdown
- Conventional backend JWT auth, not Supabase Auth
- No emails in v1, so no verification or reset flows
- HS256, per-device families with reuse detection, no rate limiting (2026-11-23)
- Tokens travel in JSON bodies, no cookies (2026-11-23)

## Open questions for the deep-dive session
- Cleanup job for expired refresh tokens (schedule)
