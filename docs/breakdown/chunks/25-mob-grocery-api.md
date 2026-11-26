# Mobile grocery list connected to the backend

| | |
|---|---|
| ID | `mob-grocery-api` |
| Area | Mobile (Compose Multiplatform) |
| Size | S |
| Wave | 7 |
| Depends on | `mob-grocery-ui` (grocery screens and interface), `mob-auth-api` (authenticated client), `be-grocery` (grocery endpoints) |
| Unlocks | `mob-offline-cache`, `mob-realtime-client` |

## Goal
Connect the grocery list to the backend.

## In scope
- Grocery data layer on the generated DTOs: both views for any week, idempotent tick / untick
- Replacing the grocery stub in DI

## Out of scope
- Grocery screens (`mob-grocery-ui`)
- Offline tick queue (`mob-offline-cache`), live ticks (`mob-realtime-client`)

## Builds on
- `mob-grocery-ui` interface, `mob-auth-api` client, `be-grocery` endpoints

## Provides
- Real grocery data

## Done when
- Against the local backend, both views, ticks shared between two devices and past weeks work on both platforms.

## Decisions from the breakdown
- The data layer goes behind the repository interfaces from the UI chunk; screens and ViewModels stay as they are (2026-11-26)
- Verified by hand against the locally running backend; automated tests come in `mob-tests`
- Ticks set a target state, so the offline queue can replay them (2026-11-23)

## Open questions for the deep-dive session
- Differences between the stub's domain models and the generated DTOs: fix in the mapper or adjust the domain model
