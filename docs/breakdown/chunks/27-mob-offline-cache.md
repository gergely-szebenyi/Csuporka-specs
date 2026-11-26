# Mobile offline cache

| | |
|---|---|
| ID | `mob-offline-cache` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 8 |
| Depends on | `mob-recipes-api` (recipe and video data sources), `mob-plan-api` (plan data source and cooked writes), `mob-grocery-api` (grocery data source and tick writes) |
| Unlocks | `mob-tests` |

## Goal
Local database caching all stored recipes, the saved video list, the current week's plan and grocery list, with an offline write queue for grocery ticks and cooked marks.

## In scope
- Local database for: all stored recipes (ingredients, steps, notes), the saved video list, the current week's plan incl. cooked state, the current week's grocery list incl. ticks
- Cache-backed repository decorators around the real data sources
- Offline write queue for grocery ticks and mark/unmark-as-cooked, replayed on reconnect; a cooked mark keeps the time it was made (2026-11-23)
- Read-only mode signal consumed by screens (other edits disabled, offline banner), based on the `mob-network` connectivity monitor
- Refresh and expiry policy

## Out of scope
- Offline editing of recipes, videos or the plan (out of v1)
- Playing videos offline
- Connectivity monitoring itself (`mob-network`)

## Builds on
- The data layers from `mob-recipes-api`, `mob-plan-api`, `mob-grocery-api`; connectivity state from `mob-network`
- Shared specs: `offline-replay`

## Provides
- Offline access to recipes, this week's plan and the grocery list

## Done when
- With network off, all recipes, the saved video list, this week's plan and the grocery list load from cache on both platforms; ticks and cooked marks are queued and reach the local backend on reconnect; other edits are blocked with the offline banner.

## Decisions from the breakdown
- Offline in v1: view stored recipes and this week, tick the grocery list, mark a meal as cooked; nothing else (the "Ízlett?" answer is online only)
- Only the current week is cached (not next week)
- Comes after the data layers it caches, since there is no fake layer anymore (2026-11-26)

## Open questions for the deep-dive session
- SQLDelight vs Room KMP
- Clearing cache on logout / when joining a household replaces the plan
- Replay conflicts (e.g. the meal was removed while offline)
