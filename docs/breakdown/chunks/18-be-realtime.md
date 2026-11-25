# Backend real-time sync server

| | |
|---|---|
| ID | `be-realtime` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 4 |
| Depends on | `be-households` (membership to authorize household channels) |
| Unlocks | `mob-integration` |

## Goal
Authenticated WebSocket connections, one channel per household, and change notices for every relevant domain event.

## In scope
- Raw WebSocket endpoint (Spring WebSocket, JSON messages) in the same Spring Boot app
- Token checked during the handshake (`HandshakeInterceptor`, same `JwtDecoder`); missing or invalid tokens are rejected before the connection opens
- Per-household channels; membership checked via `HouseholdMembershipQuery`, and connections dropped when a member's household changes
- Listening to cross-module events after commit and broadcasting change notices (type + affected resource, e.g. plan changed for week 2026-11-23)
- When a connection's access token expires, close it with the contract's close code (2026-11-23)
- No server-side event log: clients refetch after reconnecting

## Out of scope
- Publishing events from features (each feature chunk publishes its own)
- Mobile client (mob-realtime-client)

## Builds on
- Domain-event mechanism from be-scaffolding, token validation from be-auth, membership from be-households

## Provides
- Live household sync for mobile

## Done when
- A test client connected as a household member receives change notices for events published in-process; non-members and invalid tokens are rejected at the handshake.

## Decisions from the breakdown
- Real-time sync chosen over pull-based sync
- Raw WebSocket, change notices only, no catch-up log (2026-11-23)
- Single instance; more instances would need a shared relay (raise it first)

## Open questions for the deep-dive session
- Heartbeat/ping interval
