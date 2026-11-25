# Mobile weekly planner

| | |
|---|---|
| ID | `mob-planner` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 3 |
| Depends on | `mob-api-fakes` (plan repository), `mob-design-system` (components) |
| Unlocks | `mob-reminder` |

## Goal
Week view opened from Today, week navigation (past weeks, this week, next week), adding a recipe or saved video as a meal (cook day + 1–3 days), overlap warning, removing and editing meals, cook status tags, live updates, empty and offline states; accepts a target week for deep links.

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
- Toast when another household member changes the plan (from fake events)
- Empty and unplanned week states
- Offline: readable, adding disabled
- Route that opens a given week (for deep links)

## Out of scope
- Planning reminder (mob-reminder)
- Today screen (mob-today)
- Tablet layouts (out of v1)

## Builds on
- mob-api-fakes, mob-design-system

## Provides
- Planner route with week parameter used by mob-reminder

## Done when
- All planner flows work against the fake API on all phone sizes, including crossing into next week, overlap replace, and opening a specific week by route.

## Decisions from the breakdown
- One lunch per day; weeks start on Monday
- A meal is cooked once (cook day) and lasts 1–3 consecutive days
- Planned meals have no servings
- A meal that runs into next week puts its ingredients on the cook day's week's grocery list

## Open questions for the deep-dive session
- None left from the breakdown; review edge cases in the session

## Meal done state ("Megfőztem")
See `design/DESIGN-DECISIONS.md` › *Marking a meal as cooked* and *Notifications*.
- A cook day marked as cooked shows a check instead of its dot in the week strip. Past weeks keep the checks.
- Week view status tag on every cook-day row: cooked → "Megfőzve" (sage); today/future not cooked → "Főzni kell" (honey); multi-day adds " · N napra"; past never marked → no tag. Leftover rows unchanged.
- The day menu has no cooked toggle; marking happens on Today.
