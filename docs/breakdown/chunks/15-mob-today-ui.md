# Mobile daily cooking screen UI

| | |
|---|---|
| ID | `mob-today-ui` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `mob-design-system` (base components, tokens and assets) |
| Unlocks | `mob-plan-api` |

## Goal
The Today screen with the shared cook / leftover / done / empty status logic, the expandable cook card, marking as cooked and the "Ízlett?" card, on an in-memory stub.

## In scope
- Shared domain logic: for today, cook day (not cooked → cook, cooked → done), leftover day (and from which day), or nothing planned
- Date ("Péntek, november 20.") and week strip (honey dot = cook day, sage = leftover, ring = nothing planned, check = cooked); strip and "Heti terv" open the week view
- Cook day: no headline, only the date. Card expands to show ingredients; "Elkészítem" opens recipe details or plays the video; "Megfőztem" below it
- Leftover day: headline "Ma nem kell főzni" with the carry-over line ("A tegnapi … mára is elég."); the card doesn't expand, tapping opens the recipe
- Video meal: the video with a play button, no ingredients
- No "Maradék X" label, no preview of tomorrow
- Empty states: nothing planned today ("Hozzáadás mára", "Heti terv megnyitása"); no recipes yet ("Első recept felvétele")

## Out of scope
- Recipe details screen and video player themselves (`mob-recipe-library-ui`)
- Real plan data and cooked marks (`mob-plan-api`)
- Offline queueing of cooked marks (`mob-offline-cache`)

## Builds on
- `mob-design-system` components; `mob-scaffolding` navigation, module layout and the repository/stub conventions
- Shared specs: `meal-plan`

## Provides
- Main screen of the app

## Done when
- The status logic handles every carry-over and cooked combination and the screen shows each state on the stub on both platforms.

## Decisions from the breakdown
- Built on hand-written in-memory stubs behind the feature's repository interfaces, with seeded Hungarian data; the data layer comes later in its `*-api` chunk (2026-11-26)
- Status computed on device so it works offline
- One lunch per day, so there are no mixed days

## Open questions for the deep-dive session
- Wording when carried over from two days ago (day 3 of a meal)
- Stub seed data and how to switch stub states (errors, offline, another member's change) for manual checks

## Meal done state ("Megfőztem") and "Ízlett?"
See `design/DESIGN-DECISIONS.md` › *Marking a meal as cooked*, *Recipes* and *Notifications*.
- Cook card: "Elkészítem" opens the recipe details or plays the video; "Megfőztem" marks the meal as cooked. The recipe details / video player opened from the card also show "Megfőztem".
- Done state: headline "Kész a mai ebéd", line "‹Name› megfőzte.", collapsed non-expanding card with a sage "Megfőzve" tag; tap opens the recipe or video.
- Arriving from recipe details / video player after "Megfőztem": Today shows the done state and the undo toast (Figma 2.12).
- Undo: toast with "Visszavonás", and a "Mégsem főztem meg" text button under the done card until the day ends.
- Status logic gains the cooked input: cook day not cooked → cook; cook day cooked → done; leftover days unchanged.
- "Ízlett?" card: shown from the day after a new recipe/video was marked as cooked, until someone in the household answers. Not shown if it was never marked.
- Answers: Ízlett (recipe) → Kedvencek, with a toast; Ízlett (video) → the editor opens with name and video prefilled (`mob-recipe-editor-ui`); Nem ízlett → deleted for the whole household, with an undo toast. The first answer decides for everyone and the card disappears for all members.
- Marking and unmarking work offline (queued, `mob-offline-cache`) and update live when another member marks or unmarks (`mob-realtime-client`).
- The "Ízlett?" answer needs a connection: offline the card is shown with its buttons disabled.
