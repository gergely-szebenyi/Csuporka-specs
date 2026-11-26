# Csuporka — Project Breakdown

> Initial plan created with Project Breakdown v2 on 2026-11-20, revised 2026-11-23 and 2026-11-26 (code-first contract, UI-first mobile; see INTERVIEW.md). This is starting context for the chunk sessions, not a living spec. Each chunk session produces its own detailed documents.

## Vision
Csuporka is a household meal-planning app for Android and iOS. Members of a household share a recipe book, plan the week's meals together, see each day whether they need to cook or can eat yesterday's leftovers, and get a grocery list generated from the plan. Changes sync live across all members' devices, and the current week is readable offline. It's a pet project: it runs locally and isn't deployed in v1.

## Scope of the first version
**In:** email/password sign-in (JWT access 50 min / refresh 30 days, per-device refresh families); manual recipe creation; website recipe import via schema.org structured data (review before save); household-wide Kedvencek / Újdonság filters with the "Ízlett?" decision card; saved video links (title + thumbnail, full-screen in-app player, can be planned); weekly planner (this and next week plannable, every past week view-only, meals of 1–3 days); daily cooking screen with expandable cards, carry-over messages and "Megfőztem"; generated grocery list (per-recipe and in-store views, multi-day dedupe, shared ticks); households (create, QR invite/join with recipe and video merge); real-time sync across household devices (WebSocket); weekly-planning reminder as a local notification (per-device day and time, deep-links to next week); offline: view all stored recipes and this week, tick the grocery list, mark a meal as cooked (queued); phones only (small, standard, large); light / dark / system theme; Hungarian UI; CI on GitHub Actions (backend build and tests, Android build and tests).

**Out:** Google and Apple sign-in; server push notifications (FCM/APNs) and the "Ízlett?" push; rate limiting; hosting, staging, CD and store delivery (TestFlight / Play); iOS builds in CI; video recipe extraction; import via OS share sheet; AI extraction; offline editing of recipes and the plan; photo upload and website photos; leaving a household; account deletion; tablet and desktop layouts; transactional email (verification, password reset); terms / privacy text; English or other locales.

**Precedence and later changes:** on product behaviour, `design/DESIGN-DECISIONS.md` and the Figma file win, then the chunk files, then this file. The backend and mobile repos' `AGENTS.md` files repeat the key product rules and own the engineering rules; they were aligned with these docs on 2026-11-23 and still describe the spec-first contract and fakes until `be-platform` / `mob-network` (or an earlier pass) updates them for the 2026-11-26 code-first revision. The chunk files were aligned with it and with the Figma designs on 2026-11-20, and with the 2026-11-23 interview. The Figma file was updated to match on 2026-11-23 (sign-in is email-only, the "Ízlett?" switch and push screen are gone).

**Behavioural contract:** the shared specs in this store (`openspec/specs/`: identity, households, recipe-library, meal-plan, grocery-list, live-sync, offline-replay, domain-errors) define the rules and error codes both platforms follow. Wire formats come from the backend's springdoc output.

## Tech stack
| Area | Choice | Why (main reason) | Alternatives considered |
|---|---|---|---|
| Backend | Kotlin + Spring Boot 4 (MVC, virtual threads), REST, modular monolith | Fixed by spec | — |
| API contract | Code-first: each backend feature's springdoc output is the wire contract; behaviour and error codes in the shared specs | One source of truth in code; mobile connects per feature once its backend chunk is done (2026-11-26) | Spec-first OpenAPI YAML + mobile fakes (used until 2026-11-26) |
| Real-time | Raw WebSocket in the Spring Boot app; change notices, clients refetch | Live household sync was required | STOMP; pull-based delta sync; server event log |
| Mobile | Kotlin + Compose Multiplatform (shared UI + logic); feature modules, MVI, Koin, Ktor, navigation-compose; DTOs generated from the backend's springdoc spec | Fixed by spec; one codebase for Android + iOS | Two native apps |
| Database | PostgreSQL on Supabase (DB only), one project, session pooler | Fixed by spec; free plan | Separate dev/prod projects (paid) |
| Auth | Backend JWT (HS256 access + rotating opaque refresh) | Fixed by spec | Supabase Auth (excluded); Google/Apple (dropped) |
| Recipe import | jsoup + schema.org Recipe parsing, synchronous; video links via oEmbed with Open Graph fallback | Simple, no AI cost | Async LLM pipeline |
| Reminder | Local notification on the phone | Free on iOS (remote push needs a paid Apple account) | FCM server push |
| Errors | Sentry (free tier) | Visibility while developing | Logs only |
| Tests | Backend: unit tests with fakes, embedded Postgres for database tests, in every chunk. Mobile: written in `mob-tests` after the feature chunks; each `*-api` chunk is checked by hand against the local backend | No Docker on the dev machine; the owner learns the mobile code first | Testcontainers; mobile tests per chunk |
| CI | GitHub Actions on both repos: backend `./gradlew build`; Android build + unit tests. iOS is built locally | Catch broken builds; macOS runners cost 10× minutes on private repos | No CI; iOS in CI |
| Hosting | None in v1: backend runs locally | Pet project | VPS with staging + prod |

## Areas
| Area | What it covers | Chunks |
|---|---|---|
| Backend (Spring Boot) | REST + WebSocket service, platform utilities, auth, domain logic; source of the code-first API contract | `be-scaffolding`, `be-platform`, `be-auth`, `be-households`, `be-recipe-import`, `be-realtime`, `be-recipes`, `be-meal-plan`, `be-grocery` |
| Mobile (Compose Multiplatform) | One CMP codebase for Android + iOS: feature UIs on stubs, then one data-layer chunk per feature, live sync, offline cache, tests | `mob-scaffolding`, `mob-design-system`, `mob-auth-ui`, `mob-grocery-ui`, `mob-households-ui`, `mob-network`, `mob-planner-ui`, `mob-recipe-editor-ui`, `mob-recipe-library-ui`, `mob-today-ui`, `mob-auth-api`, `mob-reminder`, `mob-households-api`, `mob-recipes-api`, `mob-plan-api`, `mob-grocery-api`, `mob-offline-cache`, `mob-realtime-client`, `mob-tests` |

## Existing foundations
- **Shared specs** (this store, `openspec/specs/`): the behavioural contract from the archived `2026-11-25-define-domain-model` change: identity, households, recipe library (incl. import), meal plan, grocery list, live sync, offline replay and the domain error codes. It replaces the old `api-contract` chunk.
- **Backend repo** (`~/IDEA Projects/csuporka`): Spring Initializr skeleton (`com.prekogdevs.csuporka`), `spring-*` skills, `AGENTS.md` (= `CLAUDE.md`), Supabase MCP on the single project, `docs/go-live.md` (checklist before any deployment, incl. the VPS that stays out of v1).
- **Mobile repo** (`~/Projektek/Csuporka`): fresh KMP wizard scaffold (`com.prekogdevs.csuporka`), `kmp-*` skills, `AGENTS.md` (= `CLAUDE.md`) with the mobile engineering rules and the way of working. It copies the backend's generated spec with `syncOpenApiSpec` (reworked in `mob-network`).
- **Design:** the Figma file [Csuporka designs](https://www.figma.com/design/BIN7ZRwVeUw7KANqxFNrFw/Csuporka-designs) ("Kamra, refined", light/dark, three phone sizes, Hungarian copy). Product decisions behind it are in `design/DESIGN-DECISIONS.md`.

## Structural approach
- **Code-first contract.** The backend's springdoc output is the wire contract. A feature's endpoints and DTOs exist for mobile only once its backend chunk is done, so there is no fake API layer. Behaviour and error codes come from the shared specs.
- **Backend:** `be-platform` (wave 2) puts the security filter chain, principal, springdoc publishing, error-code catalogue, the `user registered` event type and auth test helpers into `:common` / `:app`. That lets `be-auth`, `be-households` and `be-recipe-import` run in parallel in wave 3.
- **Backend modules** (auth, household, recipe, recipeimport, mealplan, grocery, realtime) never depend on each other. Reads go through query interfaces in `:common`; cross-module writes go through idempotent event listeners (eventually consistent). `be-realtime` turns events into change notices for household channels.
- **Mobile, UI first:** after `mob-design-system`, every feature module builds its UI in parallel in wave 3 (`mob-*-ui`): screens, MVI ViewModels, domain models, repository interfaces and hand-written in-memory stubs, with feature logic (Today status, planner rules) finished there. `mob-network` builds the HTTP client and DTO generation alongside them.
- **Mobile, then connected per feature:** each `mob-*-api` chunk puts the real data layer behind the existing interfaces once its backend chunk is done (`mob-auth-api` first, since every other one needs the session). Each is checked by hand against the backend running locally.
- **Live and offline last:** `mob-realtime-client` and `mob-offline-cache` come after every feature is connected (they refresh and cache those repositories); `mob-tests` closes the plan.
- Recipes, saved videos and plans are owned by a household; every user gets a household of one at registration; a user belongs to one household. Invites: 24 h, reusable. Account deletion is out of v1.
- Grocery list generation and tick state live on the backend (one tick per line, used by both views). Cook/no-cook carry-over status and "is next week fully planned" are computed in shared mobile code.
- Offline: all stored recipes, the saved video list, this week's plan and grocery list are cached. Grocery ticks and cooked marks are queued and replayed (idempotent endpoints); every other edit is disabled while offline.

## Waves
| Wave | Chunks (parallelizable) |
|---|---|
| 1 | `be-scaffolding` (Backend, M), `mob-scaffolding` (Mobile, M) |
| 2 | `be-platform` (Backend, M), `mob-design-system` (Mobile, M) |
| 3 | `be-auth` (Backend, L), `be-households` (Backend, M), `be-recipe-import` (Backend, M), `mob-auth-ui` (Mobile, M), `mob-grocery-ui` (Mobile, M), `mob-households-ui` (Mobile, M), `mob-network` (Mobile, M), `mob-planner-ui` (Mobile, L), `mob-recipe-editor-ui` (Mobile, L), `mob-recipe-library-ui` (Mobile, L), `mob-today-ui` (Mobile, M) |
| 4 | `be-realtime` (Backend, M), `be-recipes` (Backend, M), `mob-auth-api` (Mobile, M), `mob-reminder` (Mobile, M) |
| 5 | `be-meal-plan` (Backend, M), `mob-households-api` (Mobile, M), `mob-recipes-api` (Mobile, M) |
| 6 | `be-grocery` (Backend, M), `mob-plan-api` (Mobile, M) |
| 7 | `mob-grocery-api` (Mobile, S) |
| 8 | `mob-offline-cache` (Mobile, M), `mob-realtime-client` (Mobile, M) |
| 9 | `mob-tests` (Mobile, L) |

**Critical path:** `be-scaffolding` → `be-platform` → `be-households` → `be-recipes` → `be-meal-plan` → `be-grocery` → `mob-grocery-api` → `mob-realtime-client` → `mob-tests` (9 waves). The backend sets the pace. Mobile does most of its work in waves 1–3 and then connects each feature as its backend chunk lands.

## Chunk index
| # | Chunk | Area | Size | Wave | Depends on | Unlocks |
|---|---|---|---|---|---|---|
| 01 | [Backend scaffolding & architecture](chunks/01-be-scaffolding.md) | Backend | M | 1 | — | `be-platform` |
| 02 | [Mobile scaffolding & app shell](chunks/02-mob-scaffolding.md) | Mobile | M | 1 | — | `mob-design-system`, `mob-network` |
| 03 | [Backend platform utilities (security, springdoc, shared kernel)](chunks/03-be-platform.md) | Backend | M | 2 | `be-scaffolding` | `be-auth`, `be-households`, `be-recipe-import`, `mob-network` |
| 04 | [Mobile design system (Kamra, refined)](chunks/04-mob-design-system.md) | Mobile | M | 2 | `mob-scaffolding` | `mob-auth-ui`, `mob-grocery-ui`, `mob-households-ui`, `mob-planner-ui`, `mob-recipe-editor-ui`, `mob-recipe-library-ui`, `mob-today-ui` |
| 05 | [Backend authentication](chunks/05-be-auth.md) | Backend | L | 3 | `be-platform` | `mob-auth-api` |
| 06 | [Backend households & membership](chunks/06-be-households.md) | Backend | M | 3 | `be-platform` | `be-realtime`, `be-recipes`, `mob-households-api` |
| 07 | [Backend website recipe import & video link preview](chunks/07-be-recipe-import.md) | Backend | M | 3 | `be-platform` | `mob-recipes-api` |
| 08 | [Mobile authentication UI](chunks/08-mob-auth-ui.md) | Mobile | M | 3 | `mob-design-system` | `mob-auth-api` |
| 09 | [Mobile grocery list UI](chunks/09-mob-grocery-ui.md) | Mobile | M | 3 | `mob-design-system` | `mob-grocery-api` |
| 10 | [Mobile households, profile & account UI](chunks/10-mob-households-ui.md) | Mobile | M | 3 | `mob-design-system` | `mob-households-api` |
| 11 | [Mobile networking foundation](chunks/11-mob-network.md) | Mobile | M | 3 | `mob-scaffolding`, `be-platform` | `mob-auth-api` |
| 12 | [Mobile weekly planner UI](chunks/12-mob-planner-ui.md) | Mobile | L | 3 | `mob-design-system` | `mob-reminder`, `mob-plan-api` |
| 13 | [Mobile recipe editor, website import & saving videos UI](chunks/13-mob-recipe-editor-ui.md) | Mobile | L | 3 | `mob-design-system` | `mob-recipes-api` |
| 14 | [Mobile recipe library, details & saved videos UI](chunks/14-mob-recipe-library-ui.md) | Mobile | L | 3 | `mob-design-system` | `mob-recipes-api` |
| 15 | [Mobile daily cooking screen UI](chunks/15-mob-today-ui.md) | Mobile | M | 3 | `mob-design-system` | `mob-plan-api` |
| 16 | [Backend real-time sync server](chunks/16-be-realtime.md) | Backend | M | 4 | `be-households` | — |
| 17 | [Backend recipes & saved videos](chunks/17-be-recipes.md) | Backend | M | 4 | `be-households` | `be-meal-plan`, `mob-recipes-api` |
| 18 | [Mobile auth connected to the backend](chunks/18-mob-auth-api.md) | Mobile | M | 4 | `mob-auth-ui`, `mob-network`, `be-auth` | `mob-households-api`, `mob-recipes-api`, `mob-plan-api`, `mob-grocery-api` |
| 19 | [Mobile planning reminder (local notification) & deep link](chunks/19-mob-reminder.md) | Mobile | M | 4 | `mob-planner-ui` | `mob-tests` |
| 20 | [Backend weekly meal plan](chunks/20-be-meal-plan.md) | Backend | M | 5 | `be-recipes` | `be-grocery`, `mob-plan-api` |
| 21 | [Mobile households & profile connected to the backend](chunks/21-mob-households-api.md) | Mobile | M | 5 | `mob-households-ui`, `mob-auth-api`, `be-households` | `mob-realtime-client` |
| 22 | [Mobile recipes, saved videos & import connected to the backend](chunks/22-mob-recipes-api.md) | Mobile | M | 5 | `mob-recipe-library-ui`, `mob-recipe-editor-ui`, `mob-auth-api`, `be-recipes`, `be-recipe-import` | `mob-offline-cache`, `mob-realtime-client` |
| 23 | [Backend grocery list](chunks/23-be-grocery.md) | Backend | M | 6 | `be-meal-plan` | `mob-grocery-api` |
| 24 | [Mobile planner & Today connected to the backend](chunks/24-mob-plan-api.md) | Mobile | M | 6 | `mob-planner-ui`, `mob-today-ui`, `mob-auth-api`, `be-meal-plan` | `mob-offline-cache`, `mob-realtime-client` |
| 25 | [Mobile grocery list connected to the backend](chunks/25-mob-grocery-api.md) | Mobile | S | 7 | `mob-grocery-ui`, `mob-auth-api`, `be-grocery` | `mob-offline-cache`, `mob-realtime-client` |
| 26 | [Mobile offline cache](chunks/26-mob-offline-cache.md) | Mobile | M | 8 | `mob-recipes-api`, `mob-plan-api`, `mob-grocery-api` | `mob-tests` |
| 27 | [Mobile real-time sync client](chunks/27-mob-realtime-client.md) | Mobile | M | 8 | `mob-households-api`, `mob-recipes-api`, `mob-plan-api`, `mob-grocery-api` | `mob-tests` |
| 28 | [Mobile tests](chunks/28-mob-tests.md) | Mobile | L | 9 | `mob-realtime-client`, `mob-offline-cache`, `mob-reminder` | — |

## Global open questions
None left. Each chunk lists its own implementation questions.

Answered by the design: one lunch per day; weeks start on Monday; a meal is a cook day plus a length of 1–3 days; a user belongs to one household; planned meals have no servings; every recipe gets a random photo from the bundled set.

Answered by the owner (2026-11-20, 2026-11-23, 2026-11-26): see INTERVIEW.md.

## Viewing the graph
Open `graph.html` in a browser. Regenerate after editing `graph.json`:
`python3 ~/Csuporka/project-breakdown/scripts/build_graph.py docs/breakdown/graph.json`

## Starting a chunk session
Use this prompt to begin work on a chunk in a fresh session:

    We're starting chunk `<id>` from docs/breakdown/chunks/<NN-id>.md.
    Read docs/breakdown/OVERVIEW.md for the project context and that chunk file
    for scope, dependencies and open questions. Check that every chunk it depends
    on is complete, and read what they provide. Then interview me about the open
    questions and edge cases for this chunk before planning the implementation.
    Stay within the chunk's scope; anything outside it belongs to another chunk.
