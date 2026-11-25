# Mobile planning reminder (local notification) & deep link

| | |
|---|---|
| ID | `mob-reminder` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 4 |
| Depends on | `mob-planner` (route that opens a given week) |
| Unlocks | `mob-tests` |

## Goal
The weekly planning reminder as a local notification scheduled on the phone, so no server push (and no paid Apple account) is needed.

## In scope
- Local notification permission prompt (timing per the design)
- Schedule the reminder at the day and time from Profile › Értesítések (default Sunday 18:00, Hungarian time), stored on the phone
- Cancel it while next week is fully planned (every day covered by a meal or a leftover day); reschedule on every plan change the app sees (own edits, change notices, refetches)
- Text: "Jövő hét még nincs megtervezve. Mi legyen ebédre?"
- Tapping it opens next week in the planner from cold and warm start

## Out of scope
- Server push, FCM, device tokens (not in v1)
- The "Ízlett?" notification (dropped; the in-app card in mob-today stays)
- Settings screen (mob-households)

## Builds on
- mob-planner deep-link route, mob-households reminder settings

## Provides
- Planning reminders on device

## Done when
- On both platforms the reminder fires at the chosen time when next week isn't fully planned, is cancelled when it is, and opens the planner on next week.

## Decisions from the breakdown
- Local notification instead of server push (2026-11-23)
- Accepted: if another member completes the plan while this phone's app is closed, the reminder may still fire

## Open questions for the deep-dive session
- Notification library or platform code (Android AlarmManager/WorkManager, iOS UNUserNotificationCenter)
