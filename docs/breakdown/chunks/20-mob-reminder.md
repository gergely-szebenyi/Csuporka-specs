# Mobile planning reminder (local notification) & deep link

| | |
|---|---|
| ID | `mob-reminder` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 4 |
| Depends on | `mob-planner-ui` (week route for the deep link and the plan data interface) |
| Unlocks | `mob-tests` |

## Goal
The weekly planning reminder as a local notification scheduled on the phone, so no server push (and no paid Apple account) is needed.

## In scope
- Local notification permission prompt (timing per the design)
- Schedule the reminder at the day and time from Profile › Értesítések (default Sunday 18:00, Hungarian time), stored on the phone
- Cancel it while next week is fully planned (every day covered by a meal or a leftover day); reschedule on every plan change the app sees (own edits, change notices, refetches), observed through the plan repository interface
- Text: "Jövő hét még nincs megtervezve. Mi legyen ebédre?"
- Tapping it opens next week in the planner from cold and warm start

## Out of scope
- Server push, FCM, device tokens (not in v1)
- The "Ízlett?" notification (dropped; the in-app card in `mob-today-ui` stays)
- Settings screen (`mob-households-ui`)

## Builds on
- `mob-planner-ui` deep-link route and plan interface (stub now, real once `mob-plan-api` is done); reminder settings from `mob-households-ui`

## Provides
- Planning reminders on device

## Done when
- On both platforms the reminder fires at the chosen time when next week isn't fully planned (on the planner's data source, stub or real), is cancelled when it is, and opens the planner on next week.

## Decisions from the breakdown
- Local notification instead of server push (2026-11-23)
- Accepted: if another member completes the plan while this phone's app is closed, the reminder may still fire
- Built on the plan repository interface, so it works on the stub and keeps working after `mob-plan-api` (2026-11-26)

## Open questions for the deep-dive session
- Notification library or platform code (Android AlarmManager/WorkManager, iOS UNUserNotificationCenter)
- Reading the reminder settings if `mob-households-ui` isn't merged yet
