# Mobile recipe library, details & saved videos

| | |
|---|---|
| ID | `mob-recipe-library` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 3 |
| Depends on | `mob-api-fakes` (recipe + video repositories and fake events), `mob-design-system` (components) |
| Unlocks | `mob-tests` |

## Goal
Recipe library with search and Kedvencek / Újdonság filters, recipe details with servings stepper, ingredients, video and steps; saved videos list, full-screen player and video menu; empty states.

## In scope
- Receptek list: count of recipes and videos, the recipe's bundled photo, heart (Kedvencek) / spark (Újdonság) badges
- Search across recipe names, ingredients and video titles; results split into recipes and videos
- Filters Kedvencek and Újdonság (household-wide, a recipe is in exactly one); Újdonság has two tabs: Új receptek | Mentett videók
- Recipe details: photo, Kedvenc / Újdonság chips (toggle by hand), "Hozzáadás a tervhez", servings stepper (rescales the shown amounts only: "Átszámolva … – a recept nem változik"), ingredients, playable video link ("Lejátszás az appban"), steps, note, source link
- Recipe menu: edit, delete (dialogs in mob-recipe-editor)
- Saved videos: list of cards (thumbnail, title, source)
- Full-screen in-app player with "Hozzáadás a tervhez"; every video plays embedded (YouTube, TikTok, Facebook and other sites); opens in the browser only if the site refuses the embed
- Video menu: add to plan, make a recipe from it (optional), rename, delete (deletes for the household; recipes made from it stay)
- Empty states (no recipes, no saved videos)
- Live updates from fake events

## Out of scope
- Create/edit/import, saving video links (mob-recipe-editor)
- Adding to plan sheet (mob-planner)
- Tablet layouts (out of v1)

## Builds on
- mob-api-fakes, mob-design-system

## Provides
- Recipe details screen and video player reached from Today, planner and library

## Done when
- Library, details, saved videos, the player and its fallback work against the fake API on all phone sizes, with live updates from the fake event stream.

## Decisions from the breakdown
- Recipes and saved videos are household-shared; Kedvencek / Újdonság is household-wide, not per user
- Servings exist only on recipes, to calculate ingredient amounts
- Search filters the cached recipes and videos on the phone (works offline); no search endpoint

## Open questions for the deep-dive session
- Servings scaling rounding rules
- Embedding technique per platform (WebView vs native players)

## Meal done state ("Megfőztem")
See `design/DESIGN-DECISIONS.md` › *Marking a meal as cooked*.
- When recipe details or the video player were opened from the Today cook card ("Elkészítem") on the cook day, they show a "Megfőztem" button at the bottom that marks the meal as cooked. Tapping it navigates back to Today, which shows the done state with the undo toast (Figma 2.12).
