# Backend recipes & saved videos

| | |
|---|---|
| ID | `be-recipes` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 4 |
| Depends on | `be-households` (household scoping via `HouseholdMembershipQuery`) |
| Unlocks | `be-meal-plan`, `mob-recipes-api` |

## Goal
Household recipe CRUD with ingredients, quantities, units and steps; household-wide Kedvencek / Újdonság status and the "Ízlett?" answer; saved videos; publishes change events.

## In scope
- Recipe CRUD: name (only required field), servings, ingredients (quantity, unit, name), steps, note, source or video link, photo key
- On create, assign a random photo key from a fixed enum (kept on later edits); the enum shows up in the generated spec (2026-11-23)
- Kedvencek / Újdonság status: household-wide, exactly one, chosen on create and changeable by hand
- "Ízlett?" answer: Ízlett → Kedvencek; Nem ízlett → recipe deleted for the household (undo supported); the first answer wins
- Saved videos: create (URL, title, thumbnail, source domain), rename, delete (recipes made from it stay)
- List endpoints for the library return full recipes, for the offline cache and on-phone search (2026-11-23)
- Delete behaviour when a recipe or video is planned: future planned days become empty, past weeks show "Törölt recept"
- Publishing recipe and video change events

## Out of scope
- Website import and link preview (`be-recipe-import`)
- Planner references and storage (`be-meal-plan`)
- Photo upload (out of v1)

## Builds on
- `be-households` scoping, `be-scaffolding` event bus, `be-platform` conventions
- Shared specs: `recipe-library`

## Provides
- Recipe and video data and the ingredient model used by meal plan and grocery generation; endpoints for `mob-recipes-api`

## Done when
- Recipe, video and status endpoints pass integration tests with household scoping and emitted events.

## Decisions from the breakdown
- Recipes and saved videos are household-owned; Kedvencek / Újdonság is household-wide, not per user
- No photo upload and no website photos; every recipe gets a random photo from the bundled set
- "Nem ízlett" is a soft delete: undo only while the toast shows; afterwards the recipe stays deleted, keeping its name so past weeks show "Törölt recept" (2026-11-23)

## Open questions for the deep-dive session
- After "Ízlett" turns a saved video into a recipe: does the saved video stay in Mentett videók?
