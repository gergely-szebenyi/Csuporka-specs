# Mobile real-time sync client

| | |
|---|---|
| ID | `mob-realtime-client` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 8 |
| Depends on | `mob-households-api` (household repositories to refresh), `mob-recipes-api` (recipe and video repositories to refresh), `mob-plan-api` (plan repository to refresh), `mob-grocery-api` (grocery repository to refresh) |
| Unlocks | `mob-tests` |

## Goal
WebSocket client that keeps household data live: authenticated connection, lifecycle tied to foreground/background, reconnect with backoff, and refreshing repositories when a change notice arrives.

## In scope
- Raw WebSocket client (JSON messages) with the access token in the `Authorization` header of the handshake; refresh the token and reconnect when the server rejects it or closes the connection because the token expired (2026-11-23)
- Lifecycle: connect in foreground, disconnect in background
- Reconnect with backoff; after reconnecting, refetch the data currently shown (the server keeps no event log)
- Change notices (e.g. plan changed for week X) trigger a refetch in the matching repositories (households, recipes, plan, grocery)
- The "another member changed the plan" toast and live ticks wired to real notices
- Two-device household check across every feature (the end-to-end check that used to be `mob-integration`)

## Out of scope
- Offline queue (`mob-offline-cache`)
- Server side (`be-realtime`)

## Builds on
- All four mobile `*-api` chunks' repositories, `mob-auth-api` token refresh, `be-realtime` endpoint and notice format
- Shared specs: `live-sync`

## Provides
- Live household sync

## Done when
- Against the local backend, two devices in one household see each other's recipe, plan, grocery and membership changes live; reconnects refetch and the connection closes in background on both platforms.

## Decisions from the breakdown
- Real-time sync chosen over pull
- Raw WebSocket + JSON, change notices only, refetch on reconnect, token checked at the handshake (2026-11-23)
- Built after every feature is connected, so every notice has a repository to refresh (2026-11-26)

## Open questions for the deep-dive session
- Debouncing several notices for the same resource
