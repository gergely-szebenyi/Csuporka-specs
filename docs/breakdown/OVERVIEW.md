# Csuporka — Project Breakdown

> Initial plan created with Project Breakdown v2 on 2026-11-20, revised 2026-11-23 (see INTERVIEW.md). This is starting context for the chunk sessions, not a living spec. Each chunk session produces its own detailed documents.

## Vision
Csuporka is a household meal-planning app for Android and iOS. Members of a household share a recipe book, plan the week's meals together, see each day whether they need to cook or can eat yesterday's leftovers, and get a grocery list generated from the plan. Changes sync live across all members' devices, and the current week is readable offline. It's a pet project: it runs locally and isn't deployed in v1.

## Scope of the first version
**In:** email/password sign-in (JWT access 50 min / refresh 30 days, per-device refresh families); manual recipe creation; website recipe import via schema.org structured data (review before save); household-wide Kedvencek / Újdonság filters with the "Ízlett?" decision card; saved video links (title + thumbnail, full-screen in-app player, can be planned); weekly planner (this and next week plannable, every past week view-only, meals of 1–3 days); daily cooking screen with expandable cards, carry-over messages and "Megfőztem"; generated grocery list (per-recipe and in-store views, multi-day dedupe, shared ticks); households (create, QR invite/join with recipe and video merge); real-time sync across household devices (WebSocket); weekly-planning reminder as a local notification (per-device day and time, deep-links to next week); offline: view all stored recipes and this week, tick the grocery list, mark a meal as cooked (queued); phones only (small, standard, large); light / dark / system theme; Hungarian UI; CI on GitHub Actions (backend build and tests, Android build and tests).

**Out:** Google and Apple sign-in; server push notifications (FCM/APNs) and the "Ízlett?" push; rate limiting; hosting, staging, CD and store delivery (TestFlight / Play); iOS builds in CI; video recipe extraction; import via OS share sheet; AI extraction; offline editing of recipes and the plan; photo upload and website photos; leaving a household; account deletion; tablet and desktop layouts; transactional email (verification, password reset); terms / privacy text; English or other locales.

**Precedence and later changes:** on product behaviour, `design/DESIGN-DECISIONS.md` and the Figma file win, then the chunk files, then this file. The backend and mobile repos' `AGENTS.md` files repeat the key product rules and own the engineering rules; they were aligned with these docs on 2026-11-23. The chunk files were aligned with it and with the Figma designs on 2026-11-20, and with the 2026-11-23 interview. The Figma file was updated to match on 2026-11-23 (sign-in is email-only, the "Ízlett?" switch and push screen are gone).

## Tech stack
| Area | Choice | Why (main reason) | Alternatives considered |
|---|---|---|---|
| Backend | Kotlin + Spring Boot 4 (MVC, virtual threads), REST, modular monolith | Fixed by spec | — |
| API contract | OpenAPI written first; backend's springdoc output checked against it | Mobile builds against fakes in parallel | Code-first only |
| Real-time | Raw WebSocket in the Spring Boot app; change notices, clients refetch | Live household sync was required | STOMP; pull-based delta sync; server event log |
| Mobile | Kotlin + Compose Multiplatform (shared UI + logic); feature modules, MVI, Koin, Ktor, navigation-compose; DTOs generated from the contract | Fixed by spec; one codebase for Android + iOS | Two native apps |
| Database | PostgreSQL on Supabase (DB only), one project, session pooler | Fixed by spec; free plan | Separate dev/prod projects (paid) |
| Auth | Backend JWT (HS256 access + rotating opaque refresh) | Fixed by spec | Supabase Auth (excluded); Google/Apple (dropped) |
| Recipe import | jsoup + schema.org Recipe parsing, synchronous; video links via oEmbed with Open Graph fallback | Simple, no AI cost | Async LLM pipeline |
| Reminder | Local notification on the phone | Free on iOS (remote push needs a paid Apple account) | FCM server push |
| Errors | Sentry (free tier) | Visibility while developing | Logs only |
| Tests | Backend: unit tests with fakes, embedded Postgres for database tests, in every chunk. Mobile: written in `mob-tests` after the feature chunks | No Docker on the dev machine; the owner learns the mobile code first | Testcontainers; mobile tests per chunk |
| CI | GitHub Actions on both repos: backend `./gradlew build`; Android build + unit tests. iOS is built locally | Catch broken builds; macOS runners cost 10× minutes on private repos | No CI; iOS in CI |
| Hosting | None in v1: backend runs locally | Pet project | VPS with staging + prod |

## Areas
| Area | What it covers | Chunks |
|---|---|---|
| API Contract | REST resources, error model, real-time change-notice catalogue | `api-contract` |
| Backend (Spring Boot) | REST + WebSocket service, auth, domain logic | `be-scaffolding`, `be-auth`, `be-households`, `be-recipe-import`, `be-realtime`, `be-recipes`, `be-meal-plan`, `be-grocery` |
| Mobile (Compose Multiplatform) | One CMP codebase for Android + iOS: shared UI/logic, platform source sets for camera and notifications | `mob-scaffolding`, `mob-api-fakes`, `mob-design-system`, `mob-auth`, `mob-grocery`, `mob-households`, `mob-offline-cache`, `mob-planner`, `mob-realtime-client`, `mob-recipe-editor`, `mob-recipe-library`, `mob-today`, `mob-reminder`, `mob-tests`, `mob-integration` |

## Existing foundations
- **Backend repo** (`~/IDEA Projects/csuporka`): Spring Initializr skeleton (`com.prekogdevs.csuporka`), `spring-*` skills, `AGENTS.md` (= `CLAUDE.md`), Supabase MCP on the single project, `docs/go-live.md` (checklist before any deployment). The plan and design docs stay in this planning folder.
- **Mobile repo** (`~/Projektek/Csuporka`): fresh KMP wizard scaffold (`com.prekogdevs.csuporka`), `kmp-*` skills, `AGENTS.md` (= `CLAUDE.md`) with the mobile engineering rules and the way of working. Restarted on 2026-11-23; the earlier Firebase-based app is abandoned. It copies the contract from the backend repo (`syncOpenApiSpec`).
- **Design:** the Figma file [Csuporka designs](https://www.figma.com/design/BIN7ZRwVeUw7KANqxFNrFw/Csuporka-designs) ("Kamra, refined", light/dark, three phone sizes, Hungarian copy). Product decisions behind it are in `design/DESIGN-DECISIONS.md`.

## Structural approach
- Mobile develops against a fake API layer (`mob-api-fakes`) built from the contract, including a fake real-time stream and a fake connectivity state. Mobile meets the real backend only in `mob-integration`.
- Mobile feature chunks depend on fakes + design system, not on `mob-auth`, so they run in parallel.
- Backend business modules (auth, household, recipe, recipeimport, mealplan, grocery, realtime) never depend on each other. Reads go through query interfaces in `:common`; cross-module writes go through idempotent event listeners (eventually consistent). `be-realtime` turns events into change notices for household channels.
- Recipes, saved videos and plans are owned by a household; every user gets a household of one at registration; a user belongs to one household. Invites: 24 h, reusable. Account deletion is out of v1.
- Grocery list generation and tick state live on the backend (shared; one tick per line, used by both views). Cook/no-cook carry-over status and "is next week fully planned" are computed in shared mobile code.
- Offline: all stored recipes, the saved video list, this week's plan and grocery list are cached. Grocery ticks and cooked marks are queued and replayed (idempotent endpoints); every other edit is disabled while offline.

## Waves
| Wave | Chunks (parallelizable) |
|---|---|
| 1 | `api-contract` (API Contract, M), `be-scaffolding` (Backend, M), `mob-scaffolding` (Mobile, M) |
| 2 | `be-auth` (Backend, L), `mob-api-fakes` (Mobile, M), `mob-design-system` (Mobile, M) |
| 3 | `be-households` (Backend, M), `be-recipe-import` (Backend, M), `mob-auth` (Mobile, L), `mob-grocery` (Mobile, M), `mob-households` (Mobile, M), `mob-offline-cache` (Mobile, M), `mob-planner` (Mobile, L), `mob-realtime-client` (Mobile, M), `mob-recipe-editor` (Mobile, L), `mob-recipe-library` (Mobile, L), `mob-today` (Mobile, M) |
| 4 | `be-realtime` (Backend, M), `be-recipes` (Backend, M), `mob-reminder` (Mobile, M) |
| 5 | `be-meal-plan` (Backend, M), `mob-tests` (Mobile, L) |
| 6 | `be-grocery` (Backend, M) |
| 7 | `mob-integration` (Mobile, L) |

**Critical path:** `api-contract` → `be-auth` → `be-households` → `be-recipes` → `be-meal-plan` → `be-grocery` → `mob-integration`. The backend sets the pace; mobile feature work (waves 2–3) has slack.

## Chunk index
| # | Chunk | Area | Size | Wave | Depends on | Unlocks |
|---|---|---|---|---|---|---|
| 01 | [API contract (REST + real-time events)](chunks/01-api-contract.md) | API Contract | M | 1 | — | `be-auth`, `mob-api-fakes` |
| 02 | [Backend scaffolding & architecture](chunks/02-be-scaffolding.md) | Backend | M | 1 | — | `be-auth` |
| 03 | [Mobile scaffolding & app shell](chunks/03-mob-scaffolding.md) | Mobile | M | 1 | — | `mob-api-fakes`, `mob-design-system` |
| 04 | [Backend authentication](chunks/04-be-auth.md) | Backend | L | 2 | `api-contract`, `be-scaffolding` | `be-households`, `be-recipe-import` |
| 05 | [Mobile data layer & fake API](chunks/05-mob-api-fakes.md) | Mobile | M | 2 | `api-contract`, `mob-scaffolding` | all mobile feature chunks |
| 06 | [Mobile design system (Kamra, refined)](chunks/06-mob-design-system.md) | Mobile | M | 2 | `mob-scaffolding` | mobile feature screens |
| 07 | [Backend households & membership](chunks/07-be-households.md) | Backend | M | 3 | `be-auth` | `be-realtime`, `be-recipes` |
| 08 | [Backend website recipe import & video link preview](chunks/08-be-recipe-import.md) | Backend | M | 3 | `be-auth` | `mob-integration` |
| 09 | [Mobile authentication](chunks/09-mob-auth.md) | Mobile | L | 3 | `mob-api-fakes`, `mob-design-system` | `mob-tests` |
| 10 | [Mobile grocery list](chunks/10-mob-grocery.md) | Mobile | M | 3 | `mob-api-fakes`, `mob-design-system` | `mob-tests` |
| 11 | [Mobile households, profile & account](chunks/11-mob-households.md) | Mobile | M | 3 | `mob-api-fakes`, `mob-design-system` | `mob-tests` |
| 12 | [Mobile offline cache](chunks/12-mob-offline-cache.md) | Mobile | M | 3 | `mob-api-fakes` | `mob-tests` |
| 13 | [Mobile weekly planner](chunks/13-mob-planner.md) | Mobile | L | 3 | `mob-api-fakes`, `mob-design-system` | `mob-reminder` |
| 14 | [Mobile real-time sync client](chunks/14-mob-realtime-client.md) | Mobile | M | 3 | `mob-api-fakes` | `mob-tests` |
| 15 | [Mobile recipe editor, website import & saving videos](chunks/15-mob-recipe-editor.md) | Mobile | L | 3 | `mob-api-fakes`, `mob-design-system` | `mob-tests` |
| 16 | [Mobile recipe library, details & saved videos](chunks/16-mob-recipe-library.md) | Mobile | L | 3 | `mob-api-fakes`, `mob-design-system` | `mob-tests` |
| 17 | [Mobile daily cooking screen](chunks/17-mob-today.md) | Mobile | M | 3 | `mob-api-fakes`, `mob-design-system` | `mob-tests` |
| 18 | [Backend real-time sync server](chunks/18-be-realtime.md) | Backend | M | 4 | `be-households` | `mob-integration` |
| 19 | [Backend recipes & saved videos](chunks/19-be-recipes.md) | Backend | M | 4 | `be-households` | `be-meal-plan` |
| 20 | [Mobile planning reminder (local notification) & deep link](chunks/20-mob-reminder.md) | Mobile | M | 4 | `mob-planner` | `mob-tests` |
| 21 | [Backend weekly meal plan](chunks/21-be-meal-plan.md) | Backend | M | 5 | `be-recipes` | `be-grocery` |
| 22 | [Mobile tests](chunks/22-mob-tests.md) | Mobile | L | 5 | every mobile chunk before it | `mob-integration` |
| 23 | [Backend grocery list](chunks/23-be-grocery.md) | Backend | M | 6 | `be-meal-plan` | `mob-integration` |
| 24 | [Mobile ↔ real backend integration](chunks/24-mob-integration.md) | Mobile | L | 7 | `mob-tests`, `be-realtime`, `be-recipe-import`, `be-grocery` | — |

## Global open questions
None left. Each chunk lists its own implementation questions.

Answered by the design: one lunch per day; weeks start on Monday; a meal is a cook day plus a length of 1–3 days; a user belongs to one household; planned meals have no servings; every recipe gets a random photo from the bundled set.

Answered by the owner (2026-11-20): WebSocket for real-time; Hungary only, so one time zone (Europe/Budapest); imported units are converted into the fixed unit list (g, ml, db, kk, tk, ek, csipet, gerezd, fej, szál, csomag, marék); a week is identified by its Monday's date; the merged grocery view lists every line with its recipe instead of adding amounts, grouping names ignoring case and accents; only the current week is cached offline; deleted recipes show as "Törölt recept" in past weeks; every video plays embedded in the app; search runs on the phone over the cached recipes and videos.

Answered by the owner (2026-11-23): see INTERVIEW.md.

## Viewing the graph
Open `graph.html` in a browser. Regenerate after editing `graph.json`:
`python3 project-breakdown/scripts/build_graph.py docs/breakdown/graph.json`

## Starting a chunk session
Use this prompt to begin work on a chunk in a fresh session:

    We're starting chunk `<id>` from docs/breakdown/chunks/<NN-id>.md.
    Read docs/breakdown/OVERVIEW.md for the project context and that chunk file
    for scope, dependencies and open questions. Check that every chunk it depends
    on is complete, and read what they provide. Then interview me about the open
    questions and edge cases for this chunk before planning the implementation.
    Stay within the chunk's scope; anything outside it belongs to another chunk.
