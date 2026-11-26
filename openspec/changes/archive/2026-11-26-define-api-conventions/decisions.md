# Decisions: define-api-conventions

Confirmed with Szebenyi Gergely on 2026-11-26.

## Summary

This change sets down the cross-cutting conventions every Csuporka API exchange follows, so backend and mobile share one understanding before any endpoint exists: which contract is authoritative, base path and versioning, body and data formats, the error format and its status mapping, token transport, the real-time wire conventions, and retry rules. It adds `INTERNAL_ERROR` to the domain error catalogue. The list of operations follows in a separate change; together they form the plan's `api-contract` chunk. The change is not breaking: nothing is implemented yet.

## Decisions

### D1 — Scope and relation to the operations list
- **Question**: The plan's api-contract chunk was defined as an operations list; how does this conventions change relate to it?
- **Decision**: This change covers cross-cutting conventions only. A follow-up change adds the operations list; together they make up `api-contract`.
- **Alternatives**: Conventions and operations in one change; conventions only and drop the operations list.
- **Why**: Keeps each change reviewable.
- **Depends on**: —

### D2 — Platforms and breaking status
- **Question**: Which platforms does the change target, and is it breaking?
- **Decision**: Backend and mobile (one Compose Multiplatform app for Android and iOS). Not breaking, since no API is implemented yet.
- **Alternatives**: —
- **Why**: The store holds rules shared by both platforms; nothing exists to migrate.
- **Depends on**: —

### D3 — Authoritative sources
- **Question**: The generated spec (springdoc output) is authoritative for the wire format. If it contradicts a shared spec, which wins?
- **Decision**: The generated spec is authoritative for shape (paths, fields, types); the shared specs are authoritative for meaning (rules, codes, semantics) and win any conflict. A contradiction is a backend bug, or needs a shared-spec change first.
- **Alternatives**: The generated spec wins and the shared specs are updated to match.
- **Why**: —
- **Depends on**: —

### D4 — Meaning of "import/export format"
- **Question**: What did "export format" and "import format" mean?
- **Decision**: The general format of request and response bodies (JSON, encoding, field naming, nulls, enums). Not the website recipe import draft, not spec publication, not a data export feature.
- **Alternatives**: Website recipe import draft; how clients get the spec; data export / backup.
- **Why**: —
- **Depends on**: —

### D5 — Base path
- **Question**: What base path does the REST API use?
- **Decision**: `/api/v1/<plural-resource>` (e.g. `/api/v1/recipes`). The WebSocket endpoint is `/ws`. The health check stays at `/actuator/health`, public, status only.
- **Alternatives**: `/v1` without `/api`.
- **Why**: Matches the backend's existing rules and its public-route rule (`/api/v1/auth/**`).
- **Depends on**: —

### D6 — Versioning
- **Question**: How does API versioning work while the only client is the unreleased app installed from the IDE?
- **Decision**: The version stays `v1`. Additive changes are free. Breaking changes are allowed while unreleased but go through a BREAKING shared-spec change that both platforms migrate to together. A `/v2` path is introduced only after a real release.
- **Alternatives**: Strict versioning now (parallel major versions); no version in the path.
- **Why**: Avoids parallel versions for an unreleased pet project while keeping breaking changes explicit.
- **Depends on**: D5

### D7 — Household scoping in paths
- **Question**: How do paths address household-owned resources (recipes, plan, grocery)?
- **Decision**: Implicitly from the caller's session; household IDs never appear in paths. Another household's resource ID gives `NOT_FOUND`.
- **Alternatives**: Explicit household ID in the path.
- **Why**: A user belongs to exactly one household.
- **Depends on**: D5

### D8 — Body format, field naming and nulls
- **Question**: How are JSON fields named, and how are absent values sent?
- **Decision**: JSON in UTF-8; `application/json` for bodies and `application/problem+json` for errors. Field names are camelCase. Responses include every field the schema defines, with `null` for no value; empty collections are `[]`. In requests, an optional field sent as `null` and one left out mean the same.
- **Alternatives**: camelCase with absent fields omitted from responses.
- **Why**: — (content types and `[]` for empty collections were filled in as defaults and confirmed in the summary)
- **Depends on**: D4

### D9 — Data formats
- **Question**: Do the backend's data-format rules become shared rules, and what exactly does an instant look like?
- **Decision**: IDs are UUID strings. Instants are ISO-8601 strings in UTC with the `Z` suffix; the server sends at most millisecond precision, and both sides accept any number of fraction digits. Calendar dates (cook day, week) are `YYYY-MM-DD` in Europe/Budapest; a week is identified by its Monday's date; every date-based rule uses Europe/Budapest regardless of the device's time zone.
- **Alternatives**: Adopt with changes; instants in whole seconds only.
- **Why**: Promotes the rules both repos already use to shared rules; millisecond precision keeps quick successive actions ordered.
- **Depends on**: D4

### D10 — Enums and units
- **Question**: How are enums sent, what does adding a value mean, and how are ingredient units sent?
- **Decision**: Enums are UPPER_SNAKE_CASE strings. Adding a value is a breaking change that goes through a shared-spec change. Units are enum constants: `G`, `ML`, `DB`, `KK`, `TK`, `EK`, `CSIPET`, `GEREZD`, `FEJ`, `SZAL`, `CSOMAG`, `MAREK`; clients show the Hungarian abbreviation.
- **Alternatives**: Clients tolerate unknown enum values (non-breaking additions); units as literal Hungarian strings.
- **Why**: Fine with one client and breaking changes allowed (D6); units stay consistent with every other enum.
- **Depends on**: D6, D8

### D11 — Quantities
- **Question**: How are ingredient quantities (greater than 0, at most 2 decimals) sent?
- **Decision**: As JSON numbers (e.g. `0.5`, `250`, `1.25`); the 2-decimal rule is validated.
- **Alternatives**: Decimal strings.
- **Why**: —
- **Depends on**: D8

### D12 — Unknown fields
- **Question**: What happens with fields the receiver doesn't know?
- **Decision**: Both sides ignore them: clients ignore unknown response fields, the server ignores unknown request fields.
- **Alternatives**: The server rejects unknown request fields with `VALIDATION_FAILED`.
- **Why**: Makes additive changes non-breaking (D6).
- **Depends on**: D6

### D13 — Text normalisation and length
- **Question**: How is text input normalised and measured for length limits?
- **Decision**: The server trims leading and trailing whitespace, then validates and stores the trimmed value. Length is counted in Unicode code points ("á" counts as 1). Clients count the same way.
- **Alternatives**: Count grapheme clusters.
- **Why**: —
- **Depends on**: —

### D14 — Updates
- **Question**: How are edits to a resource sent?
- **Decision**: Form edits send the whole resource and replace it, matching last-write-wins. Small state changes (status, tick, cooked mark, rename) are their own operations. No partial (PATCH) bodies.
- **Alternatives**: Partial updates with PATCH.
- **Why**: Matches the existing last-write-wins rules for recipes and plans.
- **Depends on**: —

### D15 — Successful write responses
- **Question**: What does a successful write return?
- **Decision**: Creates return 201 with the resource as saved; edits and state changes return 200 with the resulting resource; deletes return 204 with no body. The live-sync notice still reaches every device.
- **Alternatives**: No body on writes; the client refetches.
- **Why**: The client updates without a refetch. (Status codes 201/200/204 were filled in as defaults and confirmed in the summary.)
- **Depends on**: D14

### D16 — Deleting something already gone
- **Question**: What does deleting a resource that no longer exists return?
- **Decision**: 404 `NOT_FOUND`, like any operation on a missing resource; the client treats it as already gone and refreshes.
- **Alternatives**: Succeed silently (204).
- **Why**: Consistency; doesn't hide a wrong ID.
- **Depends on**: D15

### D17 — List order and pagination
- **Question**: How are lists ordered, and is there pagination?
- **Decision**: Every list the API returns has a defined, stable order, stated per operation in the follow-up operations change; clients show it as received unless the screen sorts by itself. No pagination in v1: every list is returned whole.
- **Alternatives**: Order unspecified and clients sort; one global default order. A paging convention defined now.
- **Why**: Household lists are small, and the recipe list must be complete for the offline cache and on-phone search.
- **Depends on**: D1

### D18 — Error body
- **Question**: What does an error body look like, and how is each invalid field described?
- **Decision**: RFC 9457 ProblemDetail (`type`, `title`, `status`, `detail`) plus `code` from the domain error catalogue. `VALIDATION_FAILED` adds `errors`: a list of `{field, reason}`, where `field` is a path such as `name` or `ingredients[2].quantity` and `reason` is one of `REQUIRED`, `TOO_SHORT`, `TOO_LONG`, `OUT_OF_RANGE`, `INVALID_FORMAT`, `TOO_MANY`.
- **Alternatives**: Field path with a free-text message; field path only.
- **Why**: Clients show the right Hungarian text next to the right input.
- **Depends on**: —

### D19 — User-facing error text
- **Question**: Who writes the error text users see?
- **Decision**: Clients, from `code` and `reason`, in Hungarian. The server's `title` and `detail` are English, for developers, and never shown to users.
- **Alternatives**: The server sends Hungarian text that clients show as is.
- **Why**: —
- **Depends on**: D18

### D20 — Status mapping
- **Question**: How do error codes map to HTTP status?
- **Decision**: Clients always branch on `code`; the status groups codes: 400 `VALIDATION_FAILED`; 401 `UNAUTHENTICATED`, `INVALID_CREDENTIALS`, `REFRESH_TOKEN_INVALID` (clients refresh only on `UNAUTHENTICATED`); 404 `NOT_FOUND`, also for a resource that exists but belongs to another household (never 403); 409 `EMAIL_TAKEN`, `ALREADY_MEMBER`, `HOUSEHOLD_NOT_SOLO`, `VERDICT_ALREADY_GIVEN`; 422 `WEEK_NOT_PLANNABLE`, `MEAL_LOCKED`, `VERDICT_NOT_DUE`, `UNDO_WINDOW_EXPIRED`, `NO_LONGER_APPLICABLE`, `INVITE_INVALID`, `NO_RECIPE_FOUND`; 500 `INTERNAL_ERROR`.
- **Alternatives**: Minimal 400/401/404; status not part of the contract.
- **Why**: 404 instead of 403 doesn't reveal that another household's resource exists.
- **Depends on**: D18, D21

### D21 — Errors outside the catalogue
- **Question**: What code do unexpected server failures, malformed JSON and unknown routes get?
- **Decision**: Add `INTERNAL_ERROR` (500, no internals leaked) to the domain error catalogue. A malformed or unreadable body is `VALIDATION_FAILED` (400). An unknown route is `NOT_FOUND` (404).
- **Alternatives**: Several transport codes (`MALFORMED_REQUEST`, `SERVICE_UNAVAILABLE`, ...); no code for errors outside the catalogue.
- **Why**: Keeps the rule that every rejection carries a code, with one new code.
- **Depends on**: —

### D22 — Token transport
- **Question**: Where do token transport rules go?
- **Decision**: This change adds the transport rules: the access token in the `Authorization: Bearer` header, also on the WebSocket handshake; the refresh token only in JSON bodies; no cookies. Lifetimes and rotation stay in the identity spec.
- **Alternatives**: Move lifetimes into the API conventions too.
- **Why**: Avoids duplicating identity rules.
- **Depends on**: —

### D23 — Real-time conventions: where
- **Question**: springdoc can't describe the WebSocket; where are its conventions written down?
- **Decision**: This change sets the real-time conventions (path, message format, close codes). Exact message shapes are documented by `be-realtime` next to the generated spec.
- **Alternatives**: Leave all real-time wire conventions to `be-realtime`.
- **Why**: —
- **Depends on**: D5

### D24 — Real-time wire format
- **Question**: What do real-time messages and closes look like on the wire?
- **Decision**: Each notice is a JSON text message with an UPPER_SNAKE `type`, the affected resource, `actorId` and `occurredAt`, following the same data-format rules as REST. A handshake without a valid access token is rejected with HTTP 401 and `UNAUTHENTICATED` before the upgrade. Close codes: 4001 `CREDENTIAL_EXPIRED` (client refreshes and reconnects), 4002 `HOUSEHOLD_CHANGED` (client reconnects and refetches).
- **Alternatives**: Close reasons told apart by text only.
- **Why**: —
- **Depends on**: D23, D9

### D25 — Household switch on the live connection
- **Question**: When a user joins another household, how does their open connection switch?
- **Decision**: The server closes the joiner's connections with `HOUSEHOLD_CHANGED`; each device reconnects into the new household's channel and refetches.
- **Alternatives**: The server moves the open connection silently.
- **Why**: A refetch is needed after a join anyway.
- **Depends on**: D24

### D26 — Request ID
- **Question**: Does every response carry a request ID that clients use?
- **Decision**: Yes: every response, including errors, carries `X-Request-Id`; clients attach it to their error reports.
- **Alternatives**: Not part of the contract.
- **Why**: Ties app and backend logs together.
- **Depends on**: —

### D27 — Duplicate creates
- **Question**: What protects against duplicate creates (e.g. a double tap on "Mentés")?
- **Decision**: Clients block a second submit while one is in flight. Creates aren't idempotent on the server; no idempotency keys.
- **Alternatives**: Idempotency key on creates.
- **Why**: Replayable writes are already idempotent by design.
- **Depends on**: —

### D28 — Automatic retries
- **Question**: What may clients retry automatically after a timeout or dropped connection?
- **Decision**: Only reads and the replayable target-state writes (grocery ticks, cooked marks). Every other write is never retried automatically; the user sees the error and decides.
- **Alternatives**: Nothing automatically.
- **Why**: Safe without server idempotency keys (D27).
- **Depends on**: D27

## Facts

- Error codes, the "every rejection carries exactly one code" rule and `NOT_FOUND` for another household's resource already exist; the status mapping is left to the API contract — source: `openspec/specs/domain-errors/spec.md`
- Access credential 50 minutes, refresh 30 days with rotation and per-device reuse detection; public operations are registration, login, refresh and health — source: `openspec/specs/identity/spec.md`
- Only grocery ticks and cooked marks are replayable, as target-state writes carrying the action time — source: `openspec/specs/offline-replay/spec.md`
- Recipe and plan edits are last-write-wins — source: `openspec/specs/recipe-library/spec.md`, `openspec/specs/meal-plan/spec.md`
- Ingredient quantity > 0 with at most 2 decimals; fixed unit list; text limits measured after trimming — source: `openspec/specs/recipe-library/spec.md`
- A join moves the user's devices to the new household's notices; notices name the actor; no catch-up log — source: `openspec/specs/live-sync/spec.md`
- The backend already uses `/api/v1`, `/ws`, UUID strings, UTC instants, Budapest dates, ProblemDetail with `code`, Bearer access tokens and refresh tokens in JSON bodies — source: `~/IDEA Projects/csuporka/AGENTS.md`
- The mobile app maps errors by `code` and handles concurrent 401s with a single refresh — source: `~/Projektek/Csuporka/AGENTS.md`
- The contract is code-first: springdoc output is the wire contract; the operations list is a separate part of `api-contract` — source: `docs/breakdown/chunks/01-api-contract.md`, `docs/breakdown/INTERVIEW.md`

## Out of scope

- The list of operations (follow-up change in `api-contract`)
- How the generated spec is published to the mobile repo (`be-platform`, `mob-network`)
- Exact real-time message shapes per notice type (`be-realtime`)
- Rate limiting, CORS and web clients, pagination, idempotency keys, localisation of server texts

## Deferred

- The stable order of each list — settle before: the operations change's specs
