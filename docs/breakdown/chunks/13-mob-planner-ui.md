# Mobile weekly planner UI

| | |
|---|---|
| ID | `mob-planner-ui` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 3 |
| Depends on | `api-contract` (meal plan operations the plan interface mirrors), `mob-design-system` (base components, tokens and assets) |
| Unlocks | `mob-reminder`, `mob-plan-api` |

## Goal
The week view with navigation, adding a recipe or saved video as a meal, overlap replace, editing and removing meals, and cook status tags, on an in-memory stub.

## In scope
- Week view opened from Today ("Heti terv" link or the week strip)
- Week navigation: back through every past week, this week, next week
- Only this week and next week can be planned; past weeks (and past days) are view-only ("Elmúlt hét – csak megtekinthető")
- Picker ("Mit főzzünk?"): search, Mind / Kedvencek / Újdonság, Újdonság tabs Új receptek | Mentett videók
- Add sheet ("Mikor főzöd?"): this/next week toggle, day grid, "Hány napra?" 1–3 days
- A meal may run into next week when that week is plannable (so a meal starting next week can't run past its Sunday)
- Overlap: warn which day is already planned, then replace on confirm ("Csere és hozzáadás")
- Videos are planned like recipes; the sheet notes that a video has no ingredient list
- Multi-day meals shown as cook day + leftover rows ("2. nap – nem kell főzni")
- Remove via swipe with undo toast; removing the cook day removes the whole meal
- Day menu: Recept megnyitása, Csere másik receptre, Napok száma (change length later), Eltávolítás a tervből
- Toast when another household member changes the plan (triggered from the stub here; live in `mob-realtime-client`)
- Empty and unplanned week states; offline: readable, adding disabled (driven by a connectivity input)
- Route that opens a given week (for deep links)
- Planning rules in shared code; plan repository interface with an in-memory stub (shared with `mob-today-ui`)

## Out of scope
- Planning reminder (`mob-reminder`)
- Today screen (`mob-today-ui`)
- Real plan data (`mob-plan-api`)
- Tablet layouts (out of v1)

## Builds on
- `mob-design-system` components; `mob-scaffolding` navigation, module layout and the repository/stub conventions
- Shared specs: `meal-plan`

## Provides
- Planner route with week parameter and the plan data interface used by `mob-reminder`

## Done when
- All planner flows work on the stub on all phone sizes, including crossing into next week, overlap replace and opening a specific week by route.

## Decisions from the breakdown
- Built on hand-written in-memory stubs behind the feature's repository interfaces, with seeded Hungarian data; the data layer comes later in its `*-api` chunk (2026-11-26)
- One lunch per day; weeks start on Monday
- A meal is cooked once (cook day) and lasts 1–3 consecutive days
- Planned meals have no servings
- A meal that runs into next week puts its ingredients on the cook day's week's grocery list
- Repository interfaces mirror the operations in `api-contract`, so the `*-api` chunk only adds DTOs and mapping (2026-11-26)

## Open questions for the deep-dive session
- Which of planner/Today owns the plan domain models and repository interface
- Stub seed data and how to switch stub states (errors, offline, another member's change) for manual checks

## Meal done state ("Megfőztem")
See `design/DESIGN-DECISIONS.md` › *Marking a meal as cooked* and *Notifications*.
- A cook day marked as cooked shows a check instead of its dot in the week strip. Past weeks keep the checks.
- Week view status tag on every cook-day row: cooked → "Megfőzve" (sage); today/future not cooked → "Főzni kell" (honey); multi-day adds " · N napra"; past never marked → no tag. Leftover rows unchanged.
- The day menu has no cooked toggle; marking happens on Today.
