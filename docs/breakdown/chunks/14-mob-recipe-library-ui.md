# Mobile recipe library, details & saved videos UI

| | |
|---|---|
| ID | `mob-recipe-library-ui` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 3 |
| Depends on | `mob-design-system` (base components, tokens and assets) |
| Unlocks | `mob-recipes-api` |

## Goal
Recipe library with search and Kedvencek / Újdonság filters, recipe details, saved videos, full-screen player and video menu, on an in-memory stub.

## In scope
- Receptek list: count of recipes and videos, the recipe's bundled photo, heart (Kedvencek) / spark (Újdonság) badges
- Search across recipe names, ingredients and video titles on the phone; results split into recipes and videos
- Filters Kedvencek and Újdonság (household-wide, a recipe is in exactly one); Újdonság has two tabs: Új receptek | Mentett videók
- Recipe details: photo, Kedvenc / Újdonság chips (toggle by hand), "Hozzáadás a tervhez", servings stepper (rescales the shown amounts only: "Átszámolva … – a recept nem változik"), ingredients, playable video link ("Lejátszás az appban"), steps, note, source link
- Recipe menu: edit, delete (dialogs in `mob-recipe-editor-ui`)
- Saved videos: list of cards (thumbnail, title, source)
- Full-screen in-app player with "Hozzáadás a tervhez"; every video plays embedded (YouTube, TikTok, Facebook and other sites); opens in the browser only if the site refuses the embed
- Video menu: add to plan, make a recipe from it (optional), rename, delete (deletes for the household; recipes made from it stay)
- Empty states (no recipes, no saved videos)
- Recipe and saved-video repository interfaces with an in-memory stub (shared with `mob-recipe-editor-ui`; agree who creates them first)

## Out of scope
- Create/edit/import, saving video links (`mob-recipe-editor-ui`)
- Adding to plan sheet (`mob-planner-ui`)
- Real data (`mob-recipes-api`), live updates (`mob-realtime-client`)
- Tablet layouts (out of v1)

## Builds on
- `mob-design-system` components; `mob-scaffolding` navigation, module layout and the repository/stub conventions
- Shared specs: `recipe-library`

## Provides
- Recipe details screen and video player reached from Today, planner and library

## Done when
- Library, details, saved videos, the player and its fallback work on the stub on all three phone sizes.

## Decisions from the breakdown
- Built on hand-written in-memory stubs behind the feature's repository interfaces, with seeded Hungarian data; the data layer comes later in its `*-api` chunk (2026-11-26)
- Recipes and saved videos are household-shared; Kedvencek / Újdonság is household-wide, not per user
- Servings exist only on recipes, to calculate ingredient amounts
- Search filters the cached recipes and videos on the phone (works offline); no search endpoint

## Open questions for the deep-dive session
- Servings scaling rounding rules
- Embedding technique per platform (WebView vs native players)
- Which of the two recipe UI chunks owns the shared recipe domain models and repository interfaces
- Stub seed data and how to switch stub states (errors, offline, another member's change) for manual checks

## Meal done state ("Megfőztem")
See `design/DESIGN-DECISIONS.md` › *Marking a meal as cooked*.
- When recipe details or the video player were opened from the Today cook card ("Elkészítem") on the cook day, they show a "Megfőztem" button at the bottom that marks the meal as cooked. Tapping it navigates back to Today, which shows the done state with the undo toast (Figma 2.12).
