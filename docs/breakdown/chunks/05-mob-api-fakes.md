# Mobile data layer & fake API

| | |
|---|---|
| ID | `mob-api-fakes` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 2 |
| Depends on | `api-contract` (models and operations to fake), `mob-scaffolding` (project structure and DI) |
| Unlocks | `mob-auth`, `mob-grocery`, `mob-households`, `mob-offline-cache`, `mob-planner`, `mob-realtime-client`, `mob-recipe-editor`, `mob-recipe-library`, `mob-today` |

## Goal
Contract models, repository interfaces, HTTP client abstraction and error model, and fake implementations with realistic data, latency and error cases, plus a fake real-time event stream and fake connectivity state.

## In scope
- Contract models in shared code
- Repository interfaces per domain (auth, account, households, recipes, saved videos, import/link preview, plan, grocery)
- HTTP client abstraction and error model (real implementation stubbed or minimal)
- Fake implementations with seeded Hungarian data (recipes in Kedvencek and Újdonság, saved videos, a planned week incl. a video meal and a multi-day meal), latency and error cases (e.g. "no recipe found", video without title/thumbnail)
- Fake real-time event stream
- Fake connectivity state (online/offline toggle)
- Build-time switch between fake and real

## Out of scope
- Real socket client (mob-realtime-client)
- Local DB cache and offline write queue (mob-offline-cache)
- Real backend wiring (mob-integration)

## Builds on
- api-contract, mob-scaffolding DI

## Provides
- The data layer every mobile feature screen uses

## Done when
- Every contract operation has a fake behind a repository, switchable at build time, including its error cases (tests in `mob-tests`).

## Decisions from the breakdown
- Clients build against fakes and meet the backend only in mob-integration
- Contract models are generated from the copied YAML (`api/openapi.yaml`, `syncOpenApiSpec`), models only (2026-11-23)

## Open questions for the deep-dive session
- Ktor client config
- Reactive model (Flow) conventions for repositories
- Debug menu to trigger fake errors/events

## Meal done state ("Megfőztem")
See `design/DESIGN-DECISIONS.md` › *Marking a meal as cooked* and *Notifications*.
- Fake mark/unmark-as-cooked on the plan repository, with the cooked event on the fake real-time stream (simulate another member marking it).
