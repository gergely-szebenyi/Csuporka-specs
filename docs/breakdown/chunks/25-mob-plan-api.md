# Mobile planner & Today connected to the backend

| | |
|---|---|
| ID | `mob-plan-api` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 6 |
| Depends on | `mob-planner-ui` (planner screens and plan interface), `mob-today-ui` (Today screen), `mob-auth-api` (authenticated client), `be-meal-plan` (plan endpoints) |
| Unlocks | `mob-offline-cache`, `mob-realtime-client` |

## Goal
Connect the planner and Today to the backend's meal plan.

## In scope
- Plan data layer on the generated DTOs: week queries (any past week, this week, next week), add / replace on overlap / remove / change length / replace recipe
- Cooked mark and unmark as idempotent target-state writes, carrying the time it was marked
- Replacing the plan stub in DI, so planner, Today and the reminder run on real data

## Out of scope
- Planner and Today screens and status logic (`mob-planner-ui`, `mob-today-ui`)
- Offline queue for cooked marks (`mob-offline-cache`)

## Builds on
- `mob-planner-ui` and `mob-today-ui` interfaces, `mob-auth-api` client, `be-meal-plan` endpoints

## Provides
- Real plan data for planner, Today and `mob-reminder`

## Done when
- Against the local backend, every planner and Today flow works on both platforms, including a meal crossing into next week and overlap replace.

## Decisions from the breakdown
- The data layer goes behind the repository interfaces from the UI chunk; screens and ViewModels stay as they are (2026-11-26)
- Verified by hand against the locally running backend; automated tests come in `mob-tests`
- Cooked mark carries the time it was marked; the backend checks it against the cook day (2026-11-23)

## Open questions for the deep-dive session
- Differences between the stub's domain models and the generated DTOs: fix in the mapper or adjust the domain model
