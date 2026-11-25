# Mobile recipe editor, website import & saving videos

| | |
|---|---|
| ID | `mob-recipe-editor` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 3 |
| Depends on | `mob-api-fakes` (recipe, video + import repositories), `mob-design-system` (form components) |
| Unlocks | `mob-tests` |

## Goal
New-recipe source sheet; manual create/edit/delete; paste-a-link website import with loading, failure and review-before-save states; saving a video link with title and thumbnail; making a recipe from a video.

## In scope
- "Új recept" sheet: Weboldalról / Videó link / Kézzel
- Manual create and edit: name, servings, ingredient rows (quantity, unit dropdown: g/ml/db/kk/tk/ek/csipet/gerezd/fej/szál/csomag/marék, name), steps, note, source or video link (a video link plays in the app on recipe details)
- Only the name is required; ingredients and steps are optional
- Saving a new recipe: choose "Új recept" (goes to Újdonság) or "Kedvencek"
- Photo: the app assigns a random photo from the bundled set to every new recipe; the editor shows it with "A képet az app választja a mintaképek közül."
- Delete with confirmation; warns about planned days (future planned days become empty, past weeks show "Törölt recept")
- Website import: paste link, loading, failure ("Nem találtunk receptet" → "Felvétel kézzel" with the link prefilled, or "Másik link"), review before save
- Video link: paste URL, fetch title and thumbnail, editable title, placeholder when nothing is found (still savable), saved to Újdonság › Mentett videók
- Recipe from a video: manual editor with the link filled in
- After "Ízlett" on a video: editor with name and video prefilled, saved to Kedvencek

## Out of scope
- Video recipe extraction, share-sheet import (out of v1)
- Photo upload and website photos (out of v1)

## Builds on
- mob-api-fakes, mob-design-system

## Provides
- Recipe and video creation for the whole app

## Done when
- Create, edit, delete, website import (success + failure), saving a video link (with and without a preview) and recipe-from-video work against the fake API on both platforms.

## Decisions from the breakdown
- Import shows a draft for review before saving
- Recipe photos are assigned at random from the bundled set; no upload
- Only the name is required to save a recipe

## Open questions for the deep-dive session
- Reordering ingredients/steps
- Unsaved-changes handling
