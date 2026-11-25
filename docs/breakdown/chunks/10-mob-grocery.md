# Mobile grocery list

| | |
|---|---|
| ID | `mob-grocery` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `mob-api-fakes` (grocery repository and fake events), `mob-design-system` (components) |
| Unlocks | `mob-tests` |

## Goal
Week grocery list with shared tick-off, per-recipe and in-store (merged) views, week navigation, video meals, offline ticking, empty state, live tick updates from the fake event stream.

## In scope
- Header with week and remaining-items count; week navigation (any past week, this week, next week)
- "Receptenként" view: grouped by meal, with its days
- "Boltban (összesítve)" view: every ingredient line of the week grouped by ingredient, each with its own amount and recipe, not added up (e.g. tejföl 300 g (Gulyás), tejföl 200 g (Pörkölt)); names are matched ignoring case and accents
- One tick per line, shared by both views
- Tick/untick with live sync from fake events; a line whose amount changed after a plan or recipe edit comes back unticked
- Video meals: a linked card "Hozzávalók a videóban" that opens the video and says its ingredients aren't on the list
- Past week: view-only
- Offline: viewable and tickable; pending ticks marked "Szinkronizálásra vár" (queue itself in mob-offline-cache)
- Empty state ("Még nincs mit venni" + "Ugrás a tervezőhöz")

## Out of scope
- Grocery generation logic (be-grocery)
- Offline storage and the write queue (mob-offline-cache)

## Builds on
- mob-api-fakes, mob-design-system

## Provides
- Grocery feature

## Done when
- Both views work against the fake API with ticks syncing via fake events, including a video meal, past weeks and offline ticking with the fake connectivity state.

## Decisions from the breakdown
- List is generated server-side; client only displays and ticks
- Grocery list is viewable and tickable offline

## Open questions for the deep-dive session
- Showing a pending tick when another member changed the same item meanwhile
