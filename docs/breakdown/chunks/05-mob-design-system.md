# Mobile design system (Kamra, refined)

| | |
|---|---|
| ID | `mob-design-system` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 2 |
| Depends on | `mob-scaffolding` (app skeleton and modules) |
| Unlocks | `mob-auth-ui`, `mob-grocery-ui`, `mob-households-ui`, `mob-planner-ui`, `mob-recipe-editor-ui`, `mob-recipe-library-ui`, `mob-today-ui` |

## Goal
Light/dark tokens, Fredoka + Nunito typography, brand assets and base components from the Figma Design System page.

## In scope
- Light and dark colour tokens from the Figma `Color` collection (cream / walnut / honey, plus sage for "leftovers, no cooking")
- Theme switching: System / Light / Dark (the setting itself lives in Profile, `mob-households-ui`)
- Fredoka (headings) + Nunito (text) typography
- Icons from the Figma Design System page
- Brand assets: honey-pot mark (also in the bottom nav), wordmark, empty-state jar, and the bundled dish photo set used as recipe photos (one image for every photo key the backend can assign)
- Base components: buttons, text buttons, chips, tags (Főzni kell / leftover / Megfőzve), cards, fields, search, list rows, segmented control, toggles, radios, checkboxes, steppers, sheets, dialogs, toast, empty states, offline banner
- Component catalogue screen

## Out of scope
- Feature screens (the `mob-*-ui` chunks)
- Tablet layouts (out of v1)

## Builds on
- Figma Design System page (variables, text styles, components); `design/DESIGN-DECISIONS.md` › *Visual direction*

## Provides
- Visual language used by every feature UI chunk

## Done when
- A component catalogue screen shows every base component in light and dark on the three phone sizes, matching the Figma Design System page.

## Decisions from the breakdown
- Figma is the visual reference
- One look on iOS and Android

## Open questions for the deep-dive session
- How icons and photos are exported from Figma (vector drawables / ImageVector, image formats)
- The photo key list before the backend's enum exists in the generated spec (agree the keys with `be-recipes`)
