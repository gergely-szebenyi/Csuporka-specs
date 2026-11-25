# Mobile ↔ real backend integration

| | |
|---|---|
| ID | `mob-integration` |
| Area | Mobile (Compose Multiplatform) |
| Size | L |
| Wave | 7 |
| Depends on | `mob-tests` (and through it every mobile chunk), `be-realtime`, `be-recipe-import`, `be-grocery` |
| Unlocks | none |

## Goal
Replace fakes with the real HTTP and real-time implementations against the backend running locally (on the Supabase database), verify every flow end to end on both platforms, fix contract drift, multi-device household sync check.

## In scope
- Switch repositories from fakes to real HTTP + real-time against the local backend (reachable from devices/emulators on the local network)
- Run the end-to-end checklist on Android and iOS
- Fix contract drift on either side
- Two-device household sync, offline mode (incl. queued grocery ticks and cooked marks replaying), the local planning reminder, saving and playing real video links

## Out of scope
- Hosting, staging and store delivery (not in v1)

## Builds on
- Every mobile feature chunk and the backend chunks

## Provides
- The working v1

## Done when
- All v1 flows pass an end-to-end checklist on Android and iOS against the locally running backend, including two devices syncing in one household and offline mode.

## Decisions from the breakdown
- The only chunk where client and backend meet
- No deployment: the backend runs on the developer's Mac (2026-11-23)

## Open questions for the deep-dive session
- E2E checklist format and owner
- Which fixes go back to which chunk vs fixed here
