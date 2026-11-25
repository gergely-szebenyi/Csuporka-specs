# API contract (REST + real-time events)

| | |
|---|---|
| ID | `api-contract` |
| Area | API Contract |
| Size | M |
| Wave | 1 |
| Depends on | none |
| Unlocks | `be-auth`, `mob-api-fakes` |

## Goal
The OpenAPI spec for every v1 resource, written first, so the mobile app can build against fakes while the backend is being built. Plus the catalogue of real-time change notices.

## In scope
- OpenAPI 3 YAML for: auth (register, login, refresh, logout), account (me), households (create, members, invite, join), recipes, saved videos, Kedvencek / Újdonság status and the "Ízlett?" answer, website import, video link preview, plan (any past week, this and next week; meals; cooked mark), grocery (views, one tick per line shared by both views)
- Error model: ProblemDetail with a `code` property; every operation lists the codes it can return
- Auth/refresh flow: bearer access token, refresh rotation, per-device refresh families
- Types: IDs `string`/`uuid`; instants `date-time` (UTC); calendar dates (cook day, week id = that week's Monday) `string`/`date` in Hungarian time
- Idempotent writes for offline replay: grocery ticks and cooked mark/unmark set a target state (PUT), not toggles
- Real-time: WebSocket endpoint, message envelope and change-notice types (e.g. plan changed for week X, grocery ticks changed, recipes changed, membership changed) as JSON Schema
- Stable `operationId`s and tags

## Out of scope
- Implementation on either side
- Google/Apple sign-in, device tokens and push (not in v1)
- Account deletion (not in v1)

## Builds on
- OVERVIEW scope, `design/DESIGN-DECISIONS.md`, the Figma flows

## Provides
- The contract `be-*` chunks implement and `mob-api-fakes` builds against

## Done when
- Spec validates, covers every in-scope feature, and generates/compiles models for mobile; the backend's springdoc output is checked against it.

## Decisions from the breakdown
- Spec-first: the YAML is the source of truth; the backend's generated spec must match it (snapshot check in `:app`)
- Real-time messages are change notices only; the app refetches the affected resource
- The YAML lives in the backend repo: `contract/openapi.yaml` and `contract/realtime/` (2026-11-23)
- Tokens travel in JSON bodies: login and register return the access and refresh token, refresh returns a new pair, refresh and logout take the refresh token in the body. No cookies (2026-11-23)
- The mobile repo copies the YAML with its `syncOpenApiSpec` Gradle task from the local backend checkout and generates its DTOs from the copy (2026-11-23)
- Cooked mark carries the time it was marked; the backend checks that time against the cook day, so a mark replayed after midnight still counts (2026-11-23)
- Recipe photo keys are an enum in the contract: the backend picks from it, the mobile app ships an image for every value (2026-11-23)
- The recipe list returns full recipes (ingredients, steps, note), enough for the offline cache and on-phone search (2026-11-23)
- When the access token expires on an open WebSocket, the server closes the connection with a defined close code; the client refreshes and reconnects (2026-11-23)
- The backend runs on port 8080 and is reachable from emulators and devices on the local network (2026-11-23)
- Only grocery ticks and cooked mark/unmark are replayed offline; the "Ízlett?" answer is online only (2026-11-23)

## Open questions for the deep-dive session
- File layout of the spec (one file vs split per resource)
- Pagination needs (probably none in v1: lists are small)
