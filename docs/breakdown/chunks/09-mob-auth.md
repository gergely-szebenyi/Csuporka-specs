# Mobile authentication

| | |
|---|---|
| ID | `mob-auth` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 3 |
| Depends on | `mob-api-fakes`, `mob-design-system` |
| Unlocks | `mob-tests` |

## Goal
Welcome, login and register screens with email and password, secure token storage, automatic refresh and retry, logout, session state driving navigation.

## In scope
- Welcome screen: email sign-in and registration, same on iOS and Android
- Email login with wrong-credentials error
- Registration: name (shown to household members), email, password, password again; rules shown as ticks and crosses: at least 8 characters, at least one number, the two passwords match
- Access token in memory only; refresh token from the JSON body, stored encrypted (Keychain / Keystore) via `expect`/`actual` on the platform APIs
- Session restored on app start with the stored refresh token
- Refresh on 401 and retry, forced logout
- Session state controlling navigation

## Out of scope
- Google and Apple sign-in (removed from v1)
- Password reset, email verification (out of v1)

## Builds on
- mob-api-fakes, mob-design-system

## Provides
- Session for the whole app

## Done when
- All auth flows work end to end against the fake API on Android and iOS, including refresh and forced logout.

## Decisions from the breakdown
- No password reset or email verification in v1
- Email and password only (2026-11-23)
- Tokens in JSON bodies, no cookies; no library for secure storage unless agreed (2026-11-23)

## Open questions for the deep-dive session
- None left from the breakdown
