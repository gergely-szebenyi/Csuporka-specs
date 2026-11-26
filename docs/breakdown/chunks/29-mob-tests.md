# Mobile tests

| | |
|---|---|
| ID | `mob-tests` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 9 |
| Depends on | `mob-realtime-client` (the connected, live app), `mob-offline-cache` (cache and write queue), `mob-reminder` (reminder scheduling) |
| Unlocks | none |

## Goal
The mobile app's tests, written in one go after the feature chunks: unit, integration and UI tests on Android and iOS, run in CI.

## In scope
- Unit tests: Today status logic for every carry-over and cooked combination (incl. day 3 of a meal, meals crossing into next week, DST weeks), "is next week fully planned", ViewModels, validators (password rules), mappers, servings rescaling
- The UI chunks' in-memory stubs reused as test fakes
- Integration tests with Ktor `MockEngine`: data sources, safe call helpers, token refresh (single refresh for concurrent `401`s, rotation, forced logout), offline write queue replay
- Real-time client against a local mock server: change notices trigger refetches, reconnect refetches, background disconnect
- Offline cache: load from cache with the network off, queue and replay grocery ticks and cooked marks
- UI tests (`*UiTest`, `runComposeUiTest`) for the main flows with `MockEngine` at the boundary
- `testAndroidHostTest` added to the GitHub Actions workflow; the test tasks join the commit gates

## Out of scope
- Automated tests against the real backend (each `*-api` chunk is checked by hand against it)
- Android device tests as a gate (only on request)

## Builds on
- Every mobile chunk; the test patterns in the mobile repo's `kmp-*-testing` skills and `AGENTS.md`

## Provides
- A tested mobile app: the last chunk of v1

## Done when
- Every item above has tests that are green on Android (host) and the iOS simulator, and the Android host tests pass on GitHub Actions.

## Decisions from the breakdown
- Tests are written at the end of the mobile work, not in each chunk (owner, 2026-11-23)
- Fakes, not mocks; no MockK or Mockito

## Open questions for the deep-dive session
- Which UI flows get UI tests
- Order of work if time runs short (logic first?)
