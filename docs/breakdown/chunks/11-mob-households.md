# Mobile households, profile & account

| | |
|---|---|
| ID | `mob-households` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `mob-api-fakes`, `mob-design-system` |
| Unlocks | `mob-tests` |

## Goal
Onboarding choice (create / join by QR / solo), create household, invite QR display, camera QR scanner, join confirmation, profile with planning-reminder settings, theme and logout.

## In scope
- Onboarding: create, join by QR, or "Később – egyedül kezdem"
- Create household: names the user's own household (every user has one from registration)
- Household screen: name, members, "Meghívás QR-kóddal"
- Invite QR display (valid 24 hours, works for several people; a new QR replaces the old one)
- Camera QR scanner (platform source sets) with permission handling
- Join confirmation sheet: your recipes and videos merge in, the household's plan replaces yours
- Profile (opened from the avatar): header; household row, or for a solo user "Létrehozás" / "Csatlakozás"
- Profile › Értesítések: planning reminder switch and "Mikor?" sheet (day + time, default Sunday 18:00), stored on the phone only (used by mob-reminder)
- Profile › Megjelenés: Téma sheet (Rendszer szerint / Világos / Sötét)
- Profile › Fiók: logout with a confirmation dialog

## Out of scope
- The "Ízlett? értesítés" switch (no push in v1)
- Scheduling the reminder itself (mob-reminder)
- Leaving a household (out of v1)
- Account deletion (out of v1)

## Builds on
- mob-api-fakes, mob-design-system

## Provides
- Household and profile flows; reminder settings for mob-reminder

## Done when
- All household and profile flows work against the fake API on both platforms, including camera scanning on devices.

## Decisions from the breakdown
- QR join is in v1; share-sheet import is not
- Invites are QR only (no link)
- Reminder settings are per device, not synced (2026-11-23)

## Open questions for the deep-dive session
- QR scanning library per platform
