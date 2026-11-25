# Mobile tests

| | |
|---|---|
| ID | `mob-tests` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 5 |
| Depends on | every mobile chunk before it (`mob-scaffolding` … `mob-reminder`) |
| Unlocks | `mob-integration` |

## Goal
The mobile app's tests, written in one go after the feature chunks: unit, integration and UI tests on Android and iOS, run in CI, so `mob-integration` starts from a tested app.

## In scope
- Unit tests: Today status logic for every carry-over and cooked combination (incl. day 3 of a meal, meals crossing into next week, DST weeks), "is next week fully planned", ViewModels, validators (password rules), mappers, servings rescaling
- Fakes from `mob-api-fakes`: every contract operation incl. its error cases ("no recipe found", video without title or thumbnail)
- Integration tests with Ktor `MockEngine`: data sources, safe call helpers, token refresh (single refresh for concurrent `401`s, rotation, forced logout), offline write queue replay
- Real-time client against a local mock server: change notices trigger refetches, reconnect refetches, background disconnect
- Offline cache: load from cache with the network off, queue and replay grocery ticks and cooked marks
- UI tests (`*UiTest`, `runComposeUiTest`) for the main flows with `MockEngine` at the boundary
- `testAndroidHostTest` added to the GitHub Actions workflow; the test tasks join the commit gates

## Out of scope
- Tests against the real backend (`mob-integration`)
- Android device tests as a gate (only on request)

## Builds on
- Every mobile chunk; the test patterns in the mobile repo's `kmp-*-testing` skills and `AGENTS.md`

## Provides
- A tested mobile app and test gates for `mob-integration`

## Done when
- Every item above has tests that are green on Android (host) and the iOS simulator, and the Android host tests pass on GitHub Actions.

## Decisions from the breakdown
- Tests are written at the end of the mobile work, not in each chunk (owner, 2026-11-23)
- Fakes, not mocks; no MockK or Mockito

## Open questions for the deep-dive session
- Which UI flows get UI tests
- Order of work if time runs short (logic first?)
