# Mobile real-time sync client

| | |
|---|---|
| ID | `mob-realtime-client` |
| Area | Mobile (Compose Multiplatform) |
| Size | M |
| Wave | 3 |
| Depends on | `mob-api-fakes` |
| Unlocks | `mob-tests` |

## Goal
WebSocket client that keeps household data live: authenticated connection, lifecycle tied to foreground/background, reconnect with backoff, and refreshing repositories when a change notice arrives.

## In scope
- Raw WebSocket client (JSON messages) with the access token in the `Authorization` header of the handshake; refresh the token and reconnect when the server rejects it or closes the connection because the token expired (2026-11-23)
- Lifecycle: connect in foreground, disconnect in background
- Reconnect with backoff; after reconnecting, refetch the data currently shown (the server keeps no event log)
- Change notices (e.g. plan changed for week X) trigger a refetch of that resource in the repositories, in place of the fake stream
- Checked by hand against a local mock server (automated tests in `mob-tests`)

## Out of scope
- Offline queue (mob-offline-cache)

## Builds on
- mob-api-fakes repositories and the contract's change-notice catalogue

## Provides
- Live household sync

## Done when
- Against a local mock server, change notices refresh repositories, reconnects refetch, and the connection closes in background on both platforms (checked by hand).

## Decisions from the breakdown
- Real-time sync chosen over pull
- Raw WebSocket + JSON, change notices only, refetch on reconnect, token checked at the handshake (2026-11-23)

## Open questions for the deep-dive session
- Debouncing several notices for the same resource
