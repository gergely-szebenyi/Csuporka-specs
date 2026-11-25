# Mobile offline cache

| | |
|---|---|
| ID | `mob-offline-cache` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `mob-api-fakes` (repository interfaces and connectivity abstraction) |
| Unlocks | `mob-tests` |

## Goal
Local database caching all stored recipes, the saved video list, the current week's plan and grocery list; an offline write queue for grocery ticks and cooked marks; connectivity monitoring replacing the fake connectivity state; read-only signalling for everything else; cache refresh and expiry.

## In scope
- Local database for: all stored recipes (ingredients, steps, notes), the saved video list, the current week's plan incl. cooked state, the current week's grocery list incl. ticks
- Cache-backed repository decorators
- Offline write queue for grocery ticks and mark/unmark-as-cooked, replayed on reconnect; a cooked mark keeps the time it was made (2026-11-23)
- Real connectivity monitor (Android + iOS) replacing the fake
- Read-only mode signal consumed by screens (other edits disabled, offline banner)
- Refresh and expiry policy

## Out of scope
- Offline editing of recipes, videos or the plan (out of v1)
- Playing videos offline

## Builds on
- mob-api-fakes

## Provides
- Offline access to recipes, this week's plan and the grocery list

## Done when
- With network off, all recipes, the saved video list, this week's plan and the grocery list load from cache on both platforms; grocery ticks and cooked marks are queued and sync on reconnect; other edits are blocked with the offline banner.

## Decisions from the breakdown
- Offline in v1: view stored recipes and this week, tick the grocery list, mark a meal as cooked; nothing else (the "Ízlett?" answer is online only)
- Only the current week is cached (not next week)

## Open questions for the deep-dive session
- SQLDelight vs Room KMP
- Clearing cache on logout / when joining a household replaces the plan
- Replay conflicts (e.g. the meal was removed while offline)
