# Backend real-time sync server

| | |
|---|---|
| ID | `be-realtime` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 4 |
| Depends on | `be-households` (`HouseholdMembershipQuery` for per-household channels) |
| Unlocks | none |

## Goal
Authenticated WebSocket connections, one channel per household, and change notices for every relevant domain event.

## In scope
- Raw WebSocket endpoint (Spring WebSocket, JSON messages) in the same Spring Boot app
- Token checked during the handshake (`HandshakeInterceptor`, the `be-platform` `JwtDecoder`); missing or invalid tokens are rejected before the connection opens
- Per-household channels; membership checked via `HouseholdMembershipQuery`, and connections dropped when a member's household changes
- Listening to cross-module events after commit and broadcasting change notices (type + affected resource, e.g. plan changed for week 2026-11-23)
- When a connection's access token expires, close it with a defined close code (2026-11-23)
- No server-side event log: clients refetch after reconnecting
- Documenting the notice types and close codes for the mobile client (springdoc doesn't cover WebSocket)

## Out of scope
- Publishing events from features (each feature chunk publishes its own)
- Mobile client (`mob-realtime-client`)

## Builds on
- Domain-event mechanism from `be-scaffolding`, `JwtDecoder` from `be-platform`, membership from `be-households`
- Shared specs: `live-sync`

## Provides
- Live household sync for `mob-realtime-client`

## Done when
- A test client connected as a household member receives change notices for events published in-process; non-members and invalid tokens are rejected at the handshake.

## Decisions from the breakdown
- Real-time sync chosen over pull-based sync
- Raw WebSocket, change notices only, no catch-up log (2026-11-23)
- Single instance; more instances would need a shared relay (raise it first)

## Open questions for the deep-dive session
- Heartbeat/ping interval
- Where the notice and close-code wire format is written down for mobile (code-first: Kotlin DTOs plus a JSON Schema or Markdown file in the backend repo)
