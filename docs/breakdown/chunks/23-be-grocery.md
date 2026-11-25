# Backend grocery list

| | |
|---|---|
| ID | `be-grocery` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 6 |
| Depends on | `be-meal-plan` (the week's planned meals) |
| Unlocks | `mob-integration` |

## Goal
Generate the week's grocery list from the plan using each recipe's own amounts, count multi-day meals once, per-recipe and merged views sharing one tick per line, video meals as links; regenerates on plan/recipe change.

## In scope
- Generate the list from the week's meals using the recipe's ingredient amounts as stored (plans have no servings)
- Count a multi-day meal once; a meal that runs into next week counts in its cook day's week
- Per-recipe view (grouped by meal, with days)
- Merged in-store view: every ingredient line grouped by ingredient, each with its amount and recipe; amounts are not added up (e.g. tejföl 300 g (Gulyás), tejföl 200 g (Pörkölt))
- One tick per line, shown in both views and shared by the household, per week
- Video meals appear as video entries without ingredients
- Past weeks are read-only; any past week can be read
- Ticks are idempotent (the app replays them after offline use)
- Regenerate on plan/recipe changes while preserving ticks
- Publishing tick/list change events

## Out of scope
- Manual extra grocery items (not in spec)
- Servings scaling into the list (servings only scale the view on recipe details)

## Builds on
- be-meal-plan, be-recipes ingredient model

## Provides
- Grocery endpoints for mob-grocery

## Done when
- Grocery endpoints pass tests for grouping in the merged view, multi-day dedupe, cross-week meals, video meals, tick persistence (one tick shown in both views) and regeneration after plan edits.

## Decisions from the breakdown
- Generated and stored server-side so ticks are shared in the household
- The merged view lists lines, it doesn't add amounts, so no unit conversion is needed
- Grouping matches ingredient names ignoring case and accents ("Tejföl" = "tejfol")

- When regeneration changes a line's amount, that line is unticked (2026-11-23)
- One tick per line for both views, since the merged view no longer adds amounts (2026-11-23)
- Reads plan and recipe data through query interfaces in `:common`

## Open questions for the deep-dive session
- None left from the breakdown
