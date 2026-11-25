# Mobile scaffolding & app shell

| | |
|---|---|
| ID | `mob-scaffolding` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 1 |
| Depends on | none |
| Unlocks | `mob-api-fakes`, `mob-design-system` |

## Goal
Compose Multiplatform project for Android + iOS, module structure, DI, navigation shell (bottom nav, routes incl. deep-link entry), layouts for the three phone sizes, Hungarian string resources, CI build (Android on GitHub Actions).

## In scope
- Compose Multiplatform project (Android + iOS) and module structure
- DI setup
- Navigation shell: bottom nav **Receptek · Tervező (honey pot, middle) · Bevásárlás**; Tervező opens Today, the week view is a route from it
- Route definitions incl. recipe details, Today and week view by week (deep link from the local planning reminder)
- Layout scaffolding for phones: small 375×667, standard 390×844, large 430×932; one look on iOS and Android
- Hungarian string resources
- Module layout from the mobile repo's `AGENTS.md`: `:composeApp` (renamed from `:shared`), `:core:*`, `:feature:<name>:*`, `build-logic` convention plugins
- Builds for both targets
- CI: GitHub Actions workflow that builds Android on every push (iOS is built locally; tests join in `mob-tests`)

## Out of scope
- Visual styling (mob-design-system)
- Store signing/uploads (not in v1; builds are installed from the IDE)
- Tablet and desktop layouts (out of v1)
- iOS builds in CI (macOS runners are too expensive)

## Builds on
- The Figma designs for navigation structure

## Provides
- App skeleton every mobile chunk builds in

## Done when
- Both apps build and launch locally, navigating between placeholder tabs on all three phone sizes; the Android build passes on GitHub Actions.

## Decisions from the breakdown
- Shared UI and logic in Compose Multiplatform; platform code only in platform source sets
- Hungarian only, strings externalized
- Phones only
- CI on GitHub Actions for Android only (2026-11-23)
- Feature modules (`domain` / `data` / `presentation`), MVI, Koin, navigation-compose; Android minSdk 28, iOS 18.2+ (2026-11-23)
- Tests are written later, in `mob-tests` (2026-11-23)

## Open questions for the deep-dive session
- Final feature module list (proposal in the mobile `AGENTS.md`)
- Screen orientation (portrait only or both)
