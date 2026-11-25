# Backend weekly meal plan

| | |
|---|---|
| ID | `be-meal-plan` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 5 |
| Depends on | `be-recipes` (recipes and videos to place on days) |
| Unlocks | `be-grocery` |

## Goal
Week plans per household: meals (a recipe or saved video with a cook day and a length of 1–3 days), overlap replace, remove, change length, replace recipe, past/this/next week queries, cooked state; publishes change events.

## In scope
- Meal = recipe or saved video, cook day, length 1–3 consecutive days; one lunch per day; no servings
- Only this week and next week are writable; past weeks are read-only
- A meal may run into next week when that week is plannable
- Adding over planned days replaces the overlapping meals (client warns first)
- Remove a meal (removing the cook day removes its leftover days), change its length, replace its recipe
- Week queries for any past week (read-only), this week and next week
- Listener for the household-join event: replace the joiner's plan with the household's (idempotent)
- Listener for recipe/video deletion events: future days emptied, past weeks show "Törölt recept" (idempotent)
- Publishing plan change events

## Out of scope
- Cook/no-cook status computation (mob-today, client side)
- Grocery generation (be-grocery)

## Builds on
- be-recipes, household scoping

## Provides
- Plan data for grocery (read through a query interface in `:common`)

## Done when
- Plan endpoints pass integration tests across week boundaries (incl. a meal crossing into next week and overlap replace), with events emitted.

## Decisions from the breakdown
- Carry-over status is computed on the client so it works offline; backend stores meals (cook day + length) and the cooked mark
- Weeks start on Monday
- One time zone: Hungarian time (Europe/Budapest)

- The planning reminder is a local notification on the phone; "planned" means every day of next week is covered, computed on the client (2026-11-23)

## Open questions for the deep-dive session
- None left from the breakdown

## Meal done state ("Megfőztem")
See `design/DESIGN-DECISIONS.md` › *Marking a meal as cooked* and *Notifications*.
- Store `cookedAt` / `cookedBy` on the cook-day entry; idempotent mark/unmark endpoints (set a target state, safe to replay from the offline queue), only on the cook day itself, judged by the marking time the client sends (an offline mark replayed later still counts) (2026-11-23); publish a cooked-changed event.
- Changing the number of days keeps the mark; replacing/removing the cook day removes it with the meal.
- Cook/no-cook status is still computed on the client, now from the date plus the cooked mark.
