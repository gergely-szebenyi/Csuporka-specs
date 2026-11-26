# Mobile households, profile & account UI

| | |
|---|---|
| ID | `mob-households-ui` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `api-contract` (household operations the interface mirrors), `mob-design-system` (base components, tokens and assets) |
| Unlocks | `mob-households-api` |

## Goal
Onboarding choice, create household, invite QR display, camera QR scanner, join confirmation, profile with planning-reminder settings, theme and logout, on an in-memory stub.

## In scope
- Onboarding: create, join by QR, or "Később – egyedül kezdem"
- Create household: names the user's own household (every user has one from registration)
- Household screen: name, members, "Meghívás QR-kóddal"
- Invite QR display (valid 24 hours, works for several people; a new QR replaces the old one)
- Camera QR scanner (platform source sets) with permission handling
- Join confirmation sheet: your recipes and videos merge in, the household's plan replaces yours
- Profile (opened from the avatar): header; household row, or for a solo user "Létrehozás" / "Csatlakozás"
- Profile › Értesítések: planning reminder switch and "Mikor?" sheet (day + time, default Sunday 18:00), stored on the phone only (used by `mob-reminder`)
- Profile › Megjelenés: Téma sheet (Rendszer szerint / Világos / Sötét)
- Profile › Fiók: logout with a confirmation dialog
- Household repository interface with an in-memory stub

## Out of scope
- Real households data (`mob-households-api`)
- The "Ízlett? értesítés" switch (no push in v1)
- Scheduling the reminder itself (`mob-reminder`)
- Leaving a household (out of v1)
- Account deletion (out of v1)

## Builds on
- `mob-design-system` components; `mob-scaffolding` navigation, module layout and the repository/stub conventions

## Provides
- Household and profile flows; reminder settings for `mob-reminder`; the interface `mob-households-api` implements

## Done when
- All household and profile flows work on the stub on both platforms, including camera scanning on devices.

## Decisions from the breakdown
- Built on hand-written in-memory stubs behind the feature's repository interfaces, with seeded Hungarian data; the data layer comes later in its `*-api` chunk (2026-11-26)
- QR join is in v1; share-sheet import is not
- Invites are QR only (no link)
- Reminder settings are per device, not synced (2026-11-23)
- Repository interfaces mirror the operations in `api-contract`, so the `*-api` chunk only adds DTOs and mapping (2026-11-26)

## Open questions for the deep-dive session
- QR scanning library per platform
- How logout reaches the auth session while both are still stubs
- Stub seed data and how to switch stub states (errors, offline, another member's change) for manual checks
