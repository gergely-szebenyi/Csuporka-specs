# Mobile grocery list UI

| | |
|---|---|
| ID | `mob-grocery-ui` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `api-contract` (grocery operations the interface mirrors), `mob-design-system` (base components, tokens and assets) |
| Unlocks | `mob-grocery-api` |

## Goal
The week grocery list with per-recipe and in-store views, ticking, week navigation and video meals, on an in-memory stub.

## In scope
- Header with week and remaining-items count; week navigation (any past week, this week, next week)
- "Receptenként" view: grouped by meal, with its days
- "Boltban (összesítve)" view: every ingredient line of the week grouped by ingredient, each with its own amount and recipe, not added up (e.g. tejföl 300 g (Gulyás), tejföl 200 g (Pörkölt))
- One tick per line, shared by both views; a line whose amount changed comes back unticked
- Video meals: a linked card "Hozzávalók a videóban" that opens the video and says its ingredients aren't on the list
- Past week: view-only
- Offline state: viewable and tickable; pending ticks marked "Szinkronizálásra vár" (queue itself in `mob-offline-cache`)
- Empty state ("Még nincs mit venni" + "Ugrás a tervezőhöz")
- Grocery repository interface with an in-memory stub

## Out of scope
- Grocery generation logic (`be-grocery`)
- Real data (`mob-grocery-api`), live ticks (`mob-realtime-client`)
- Offline storage and the write queue (`mob-offline-cache`)

## Builds on
- `mob-design-system` components; `mob-scaffolding` navigation, module layout and the repository/stub conventions
- Shared specs: `grocery-list`

## Provides
- Grocery feature UI and the interface `mob-grocery-api` implements

## Done when
- Both views, ticking, a video meal and past weeks work on the stub on both platforms.

## Decisions from the breakdown
- Built on hand-written in-memory stubs behind the feature's repository interfaces, with seeded Hungarian data; the data layer comes later in its `*-api` chunk (2026-11-26)
- List is generated server-side; client only displays and ticks
- Grocery list is viewable and tickable offline
- Repository interfaces mirror the operations in `api-contract`, so the `*-api` chunk only adds DTOs and mapping (2026-11-26)

## Open questions for the deep-dive session
- Showing a pending tick when another member changed the same item meanwhile
- Stub seed data and how to switch stub states (errors, offline, another member's change) for manual checks
