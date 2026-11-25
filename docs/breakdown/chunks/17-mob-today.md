# Mobile daily cooking screen

| | |
|---|---|
| ID | `mob-today` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `mob-api-fakes` (plan + recipe repositories), `mob-design-system` (components) |
| Unlocks | `mob-tests` |

## Goal
Today screen (the Tervező tab's first screen): week strip, cook / leftover / done / empty states with shared status logic, expandable cook-day card, video meals, marking as cooked, the "Ízlett?" card, navigation to recipe details or the video.

## In scope
- Shared domain logic: for today, cook day (not cooked → cook, cooked → done), leftover day (and from which day), or nothing planned
- Date ("Péntek, november 20.") and week strip (honey dot = cook day, sage = leftover, ring = nothing planned, check = cooked); strip and "Heti terv" open the week view
- Cook day: no headline, only the date. Card expands to show ingredients; "Elkészítem" opens recipe details or plays the video; "Megfőztem" below it
- Leftover day: headline "Ma nem kell főzni" with the carry-over line ("A tegnapi … mára is elég."); the card doesn't expand, tapping opens the recipe
- Video meal: the video with a play button, no ingredients
- No "Maradék X" label, no preview of tomorrow
- Empty states: nothing planned today ("Hozzáadás mára", "Heti terv megnyitása"); no recipes yet ("Első recept felvétele")

## Out of scope
- Recipe details screen and video player themselves (mob-recipe-library)

## Builds on
- mob-api-fakes, mob-design-system

## Provides
- Main screen of the app

## Done when
- The status logic handles every carry-over and cooked combination and the screen works against the fake API on both platforms (tests in `mob-tests`).

## Decisions from the breakdown
- Status computed on device so it works offline
- One lunch per day, so there are no mixed days

## Open questions for the deep-dive session
- Wording when carried over from two days ago (day 3 of a meal)

## Meal done state ("Megfőztem") and "Ízlett?"
See `design/DESIGN-DECISIONS.md` › *Marking a meal as cooked*, *Recipes* and *Notifications*.
- Cook card: "Elkészítem" opens the recipe details or plays the video; "Megfőztem" marks the meal as cooked. The recipe details / video player opened from the card also show "Megfőztem".
- Done state: headline "Kész a mai ebéd", line "‹Name› megfőzte.", collapsed non-expanding card with a sage "Megfőzve" tag; tap opens the recipe or video.
- Arriving from recipe details / video player after "Megfőztem": Today shows the done state and the undo toast (Figma 2.12).
- Undo: toast with "Visszavonás", and a "Mégsem főztem meg" text button under the done card until the day ends.
- Status logic gains the cooked input: cook day not cooked → cook; cook day cooked → done; leftover days unchanged.
- "Ízlett?" card: shown from the day after a new recipe/video was marked as cooked, until someone in the household answers. Not shown if it was never marked.
- Answers: Ízlett (recipe) → Kedvencek, with a toast; Ízlett (video) → the editor opens with name and video prefilled (mob-recipe-editor); Nem ízlett → deleted for the whole household, with an undo toast. The first answer decides for everyone and the card disappears for all members.
- Marking and unmarking work offline (queued) and update live when another member marks or unmarks.
- The "Ízlett?" answer needs a connection: offline the card is shown with its buttons disabled.
