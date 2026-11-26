# Mobile households & profile connected to the backend

| | |
|---|---|
| ID | `mob-households-api` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 5 |
| Depends on | `mob-households-ui` (screens and repository interface), `mob-auth-api` (authenticated client and session), `be-households` (household endpoints in the generated spec) |
| Unlocks | `mob-realtime-client` |

## Goal
Connect households and profile to the backend.

## In scope
- Households data layer on the generated DTOs: current user and household, name ("Létrehozás"), members, invite QR token, join with merge
- Replacing the households stub in DI; logout through the real auth session
- Mapping household errors (expired or revoked invite, already a member) to the UI's error states

## Out of scope
- Household screens (`mob-households-ui`)
- Live membership updates (`mob-realtime-client`)

## Builds on
- `mob-households-ui` interfaces, `mob-auth-api` authenticated client, `be-households` endpoints

## Provides
- Real household data for the app

## Done when
- Against the local backend, create, invite, join (with recipe and video merge) and logout work on both platforms with two accounts.

## Decisions from the breakdown
- The data layer goes behind the repository interfaces from the UI chunk; screens and ViewModels stay as they are (2026-11-26)
- Verified by hand against the locally running backend; automated tests come in `mob-tests`

## Open questions for the deep-dive session
- Differences between the stub's domain models and the generated DTOs: fix in the mapper or adjust the domain model
