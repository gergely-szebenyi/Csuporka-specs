# Csuporka — Decision log

> Why the plan looks the way it does. Only decisions that are still valid are listed; superseded ones were removed on 2026-11-23. Dates show when a decision was made. What to build is in `OVERVIEW.md`, the chunk files and `design/DESIGN-DECISIONS.md`.

## Starting point (2026-11-20)
- Input: the spec "Csuporka (MVP) – Project Overview and Requirements" and a high-fidelity mockup (`old_design.html`, since deleted). The Figma file [Csuporka designs](https://www.figma.com/design/BIN7ZRwVeUw7KANqxFNrFw/Csuporka-designs) and `design/DESIGN-DECISIONS.md` replaced the mockup as the reference.
- Backend repo (`~/IDEA Projects/csuporka`): Spring Initializr skeleton. Mobile repo: Compose Multiplatform app on Firebase mock data.

## Scope
- **Video recipes:** no video recipe extraction ("not the scope of the MVP", overrides spec item 5). Users save video links instead: title + thumbnail, full-screen in-app player, plan a video, make a recipe from it, rename, delete. Every video plays embedded. (2026-11-20)
- **Website import:** schema.org structured data only, no AI, synchronous, no job queue, no "import ready" notification. Import via the OS share sheet is out. (2026-11-20)
- **Households:** join by QR (invite QR, camera scanner, merge on join) is in. Leaving a household is out. (2026-11-20)
- **Account deletion:** out of v1, together with its household cleanup. (2026-11-25)
- **Devices and language:** phones only (tablets out); Hungarian only, strings externalized; one time zone, Europe/Budapest. (2026-11-20)
- **Email:** none in v1, so no verification and no password reset. (2026-11-20)
- **Sign-in:** email and password only; Google and Apple sign-in removed everywhere. (2026-11-23)
- **Offline:** view all stored recipes and this week, tick the grocery list (queued), mark a meal as cooked (queued). Only the current week is cached. The spec's "changes made offline should sync" is otherwise dropped. (2026-11-20) The "Ízlett?" answer is online only. (2026-11-23)
- **Notifications:** no server push, because iOS remote push needs the paid Apple Developer Program. The weekly planning reminder is a local notification, rescheduled on every plan change the app sees, with its settings stored on the phone. "Planned" means every day of next week is covered. The "Ízlett?" push and its Profile switch are gone; the in-app card stays. (2026-11-23)
- **Deployment:** none. The backend runs on the developer's Mac, apps are installed from the IDE. Rate limiting is dropped for now; it and everything else needed before a deployment is in the backend repo's `docs/go-live.md`. (2026-11-23)
- **CI:** GitHub Actions on both repos: the backend runs `./gradlew build`, the mobile app builds Android and runs the unit tests. iOS is built locally, because macOS runner minutes count 10× on private repos. (2026-11-23)

## Stack
- **Fixed by the spec:** Kotlin + Spring Boot backend; Compose Multiplatform mobile app (shared UI and logic, platform source sets for camera and notifications); Supabase Postgres used as a database only; backend-issued JWT auth (50 min access, 30 day refresh with rotation). (2026-11-20)
- **API contract:** ~~spec-first OpenAPI YAML~~ superseded on 2026-11-26: code-first, see the revision below.
- **Real-time:** WebSocket (2026-11-20). Raw WebSocket with JSON messages, the token in the `Authorization` header checked at the handshake, change notices only, clients refetch after reconnecting (no event log). (2026-11-23)
- **Supabase:** one project (free plan), used directly for local development through the session pooler (port 5432). No dev branch, no staging. The Supabase MCP is used freely; the schema changes only through Flyway. (2026-11-23)
- **Recipe import:** jsoup; video previews through oEmbed with an Open Graph fallback. (2026-11-23)
- **Backend tooling:** Sentry free tier; one `application.yaml` plus `.env`, no profiles (secrets move to 1Password before going live); no Docker on the dev machine, so database tests use embedded Postgres; Awaitility for async tests. (2026-11-23)
- **Mobile:** feature modules (`domain` / `data` / `presentation`), MVI, Koin, Ktor, navigation-compose; DTOs generated from the backend's generated spec, which the mobile repo copies with `syncOpenApiSpec` (source changed 2026-11-26); access token in memory, refresh token in the JSON body and in encrypted platform storage; Android minSdk 28, iOS 18.2+. (2026-11-23)
- **Mobile repo:** `~/Projektek/Csuporka`, restarted from the KMP wizard on 2026-11-23; the earlier Firebase-based app is abandoned. (2026-11-23)
- **Mobile way of working:** learning mode (small steps as diffs, the owner types new concepts, commits after approval); mobile tests are written in a separate `mob-tests` chunk after the feature chunks. (2026-11-23)
- **Left to the chunk sessions:** KMP local database (SQLDelight vs Room KMP), Ktor client config, QR scanning library, notification approach, video embedding, embedded Postgres library.

## Architecture
- **Backend modules:** auth, household, recipe (recipes + saved videos), recipeimport, mealplan, grocery, realtime. They never depend on each other. Reads go through query interfaces in `:common`, implemented by the owning module; cross-module writes (join replaces the plan, recipe deletion empties plan days, household creation at registration) go through idempotent event listeners, eventually consistent. (2026-11-23)
- **Events:** features publish in-process events; `be-realtime` turns them into change notices, which keeps real-time off the feature critical path. (2026-11-20)
- **Mobile:** ~~builds against a fake API layer, meets the backend only in `mob-integration`~~ superseded on 2026-11-26: feature UIs on in-memory stubs, then one `*-api` chunk per feature, see the revision below.
- **Where logic lives:** grocery generation and ticks on the backend (shared by the household); cook / no-cook status and "is next week fully planned" in shared mobile code, so they work offline. (2026-11-20, 2026-11-23)

## Product rules
- **Households:** the household owns recipes, saved videos and plans. Every user gets a household of one at registration; "Létrehozás" names it. A user belongs to one household; all members are equal. (2026-11-20, 2026-11-23)
- **Invites:** QR only, valid 24 hours, reusable; a new invite revokes the old one. (2026-11-23)
- **Planning:** one lunch per day; weeks start on Monday and are identified by their Monday's date; a meal is a cook day plus 1–3 days; planned meals have no servings. Only this week and next week can be planned; every past week can be viewed. (2026-11-20, 2026-11-23)
- **Recipes:** only the name is required; servings exist only on recipes (imported recipes bring servings and amounts); no photo upload and no website photos, every recipe gets a random photo from the bundled set. Units: g, ml, db, kk, tk, ek, csipet, gerezd, fej, szál, csomag, marék; imported units are converted into them. Search runs on the phone. (2026-11-20)
- **Kedvencek / Újdonság**, profile settings, planning rules and grocery behaviour follow the Figma design. (2026-11-20)
- **"Nem ízlett":** soft delete, undo only while the toast shows. Deleted recipes show as "Törölt recept" in past weeks. (2026-11-20, 2026-11-23)
- **Grocery:** the merged view lists every line with its recipe, amounts are not added up, names are grouped ignoring case and accents. Each line has one tick, shared by both views. A line whose amount changes is unticked. (2026-11-20, 2026-11-23)
- **Sizes:** `mob-recipe-library` and `mob-recipe-editor` are L because of saved videos. (2026-11-20)

## Removed chunks (2026-11-23)
- `infra-environments` (staging + prod on a VPS, CD): nothing is deployed in v1.
- `mob-release` (signing, TestFlight / Play uploads): apps are installed from the IDE.
- `be-notifications` (server-side reminder scheduler, FCM): the reminder is a local notification.
- `mob-push` became `mob-reminder` (local notification and deep link).
- The remaining chunks were renumbered 01–23; adding `mob-tests` later made them 01–24.

## Documents
- The plan and design docs live in this planning folder, not in a code repo. The backend and mobile repos each have an `AGENTS.md` with their engineering rules, repeating the key product rules; all three are kept in sync (last aligned 2026-11-23).
- Precedence on product behaviour: `design/DESIGN-DECISIONS.md` + Figma, then the chunk files, then `OVERVIEW.md`, then the repos' `AGENTS.md`. On engineering rules, each repo's `AGENTS.md` wins over its skills.
- The Figma file was updated to match the decisions on 2026-11-20 and 2026-11-23.

## Revision: code-first contract (2026-11-26)
- **API contract is code-first:** each backend feature's springdoc output is the wire contract, published for mobile when that feature is built. The spec-first OpenAPI YAML (`contract/openapi.yaml`) and the springdoc-vs-YAML check are dropped. Mobile therefore can't build against a contract-based fake layer, so `mob-api-fakes` is removed. (2026-11-26)
- **Behavioral contract = shared specs:** the archived `define-domain-model` change in the shared-specs store (identity, households, recipe library, meal plan, grocery list, live sync, offline replay, domain error codes) is the behavioral contract. It counts as done, so `api-contract` is no longer a chunk. Wave 1 is `be-scaffolding` + `mob-scaffolding`. (2026-11-26)
- **Backend platform utilities first:** wave 2 on the backend is one chunk of cross-cutting utilities in `:common` / `:app` (security filter chain, springdoc and how the generated spec reaches mobile, shared error model) that every backend feature builds on. `be-auth` moves from wave 2 to wave 3. (2026-11-26)
- **Mobile builds UI first, connects later:** wave 3 builds every mobile feature's UI in parallel, one chunk per feature module: screens, MVI ViewModels, domain models and repository interfaces on hand-written in-memory stubs, with feature logic (Today status, planner rules) finished here. Wave 4 onwards adds one "connect to the API" chunk per feature, which puts the data layer (generated DTOs, mappers, Ktor) behind those interfaces once the matching backend chunk is done. (2026-11-26)
- **Clients:** mobile only. "Web clients" in the request was a slip; no web target. (2026-11-26)
- **Deployment:** stays out of v1, as decided on 2026-11-23. No VPS/infra chunk is added at the end; `docs/go-live.md` keeps tracking it. (2026-11-26)
- **Graph changes approved (2026-11-26):**
  - New `be-platform` (wave 2): security filter chain, authenticated principal, springdoc and how its spec reaches mobile, error-code catalogue, auth test helpers.
  - New `mob-network` (wave 3): Ktor client, DTO generation from springdoc's output, error mapping, connectivity. Keeps `mob-auth-api` at M.
  - Mobile feature chunks split into `mob-*-ui` (wave 3, in-memory stubs) and `mob-*-api` (after the matching backend chunk). Recipe library and editor share one `mob-recipes-api`; planner and Today share `mob-plan-api`.
  - Removed: `api-contract` (done in the shared specs), `mob-api-fakes`, `mob-integration`. Each `*-api` chunk is verified against the local backend, the two-device check moved to `mob-realtime-client` and the offline check to `mob-offline-cache`.
  - `be-recipe-import` and `be-households` depend on `be-platform` only (wave 3, next to `be-auth`). Households listens for a "user registered" event in `:common` that `be-auth` publishes.
  - `mob-offline-cache` comes after the recipe, plan and grocery data layers; `mob-reminder` only needs `mob-planner-ui`.
  - Accepted trade-off: the backend chain is the critical path (9 waves, up from 7). Wave-3 domain models are written before the DTOs, so the `*-api` mappers absorb any differences.
  - Follow-up: both repos' `AGENTS.md` still describe the spec-first contract and fakes and need updating.
  - Refinement while writing the files: `mob-realtime-client` depends on all four mobile `*-api` chunks (wave 8, next to `mob-offline-cache`), because change notices refresh repositories that only exist once each feature is connected. The two-device check then covers every feature. The critical path stays at 9 waves.
  - `mob-scaffolding` also sets the conventions for repository interfaces and in-memory stubs (Flow, domain error type), so the eight parallel wave-3 UI chunks build alike.
