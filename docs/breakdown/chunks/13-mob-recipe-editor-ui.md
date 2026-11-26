# Mobile recipe editor, website import & saving videos UI

| | |
|---|---|
| ID | `mob-recipe-editor-ui` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 3 |
| Depends on | `mob-design-system` (base components, tokens and assets) |
| Unlocks | `mob-recipes-api` |

## Goal
New-recipe source sheet, manual create/edit/delete, website import with review before save, saving a video link and making a recipe from a video, on an in-memory stub.

## In scope
- "Új recept" sheet: Weboldalról / Videó link / Kézzel
- Manual create and edit: name, servings, ingredient rows (quantity, unit dropdown: g/ml/db/kk/tk/ek/csipet/gerezd/fej/szál/csomag/marék, name), steps, note, source or video link
- Only the name is required; ingredients and steps are optional
- Saving a new recipe: choose "Új recept" (goes to Újdonság) or "Kedvencek"
- Photo: the app shows the photo assigned to the recipe with "A képet az app választja a mintaképek közül."
- Delete with confirmation; warns about planned days (future planned days become empty, past weeks show "Törölt recept")
- Website import: paste link, loading, failure ("Nem találtunk receptet" → "Felvétel kézzel" with the link prefilled, or "Másik link"), review before save
- Video link: paste URL, fetch title and thumbnail, editable title, placeholder when nothing is found (still savable), saved to Újdonság › Mentett videók
- Recipe from a video: manual editor with the link filled in
- After "Ízlett" on a video: editor with name and video prefilled, saved to Kedvencek
- Import and link-preview repository interfaces with in-memory stubs (success, "no recipe found", video without title or thumbnail)

## Out of scope
- Real data, import and preview (`mob-recipes-api`)
- Video recipe extraction, share-sheet import (out of v1)
- Photo upload and website photos (out of v1)

## Builds on
- `mob-design-system` components; `mob-scaffolding` navigation, module layout and the repository/stub conventions
- Shared specs: `recipe-library`

## Provides
- Recipe and video creation for the whole app

## Done when
- Create, edit, delete, import (success + failure), saving a video link (with and without a preview) and recipe-from-video work on the stub on both platforms.

## Decisions from the breakdown
- Built on hand-written in-memory stubs behind the feature's repository interfaces, with seeded Hungarian data; the data layer comes later in its `*-api` chunk (2026-11-26)
- Import shows a draft for review before saving
- Recipe photos are assigned at random from the bundled set; no upload
- Only the name is required to save a recipe

## Open questions for the deep-dive session
- Reordering ingredients/steps
- Unsaved-changes handling
- Stub seed data and how to switch stub states (errors, offline, another member's change) for manual checks
