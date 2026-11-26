# Mobile recipes, saved videos & import connected to the backend

| | |
|---|---|
| ID | `mob-recipes-api` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 5 |
| Depends on | `mob-recipe-library-ui` (library screens and repository interfaces), `mob-recipe-editor-ui` (editor screens and import/preview interfaces), `mob-auth-api` (authenticated client), `be-recipes` (recipe and video endpoints), `be-recipe-import` (import and link-preview endpoints) |
| Unlocks | `mob-offline-cache`, `mob-realtime-client` |

## Goal
Connect the recipe library and editor to the backend: recipes, saved videos, status, "Ízlett?", website import and video link preview.

## In scope
- Recipe and saved-video data layer on the generated DTOs: list (full recipes), details, create, edit, delete, Kedvencek / Újdonság status, "Ízlett?" answer with undo
- Website import (draft for review) and video link preview
- Photo key enum from the generated spec mapped to the bundled photos
- Replacing the recipe, import and preview stubs in DI

## Out of scope
- Library and editor screens (`mob-recipe-library-ui`, `mob-recipe-editor-ui`)
- Offline cache (`mob-offline-cache`) and live updates (`mob-realtime-client`)

## Builds on
- Both recipe UI chunks' interfaces, `mob-auth-api` client, `be-recipes` and `be-recipe-import` endpoints

## Provides
- Real recipe and video data for library, editor, planner picker and Today

## Done when
- Against the local backend, every library and editor flow from the UI chunks works on both platforms, including import success/failure and saving a video with and without a preview.

## Decisions from the breakdown
- The data layer goes behind the repository interfaces from the UI chunk; screens and ViewModels stay as they are (2026-11-26)
- Verified by hand against the locally running backend; automated tests come in `mob-tests`

## Open questions for the deep-dive session
- Differences between the stub's domain models and the generated DTOs: fix in the mapper or adjust the domain model
