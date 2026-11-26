# Backend households & membership

| | |
|---|---|
| ID | `be-households` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 3 |
| Depends on | `api-contract` (household operations), `be-platform` (principal, `user registered` event type, error codes, test helpers) |
| Unlocks | `be-realtime`, `be-recipes`, `mob-households-api` |

## Goal
Household of one for every new user, naming it ("Létrehozás"), QR invite tokens, join with merge of the joiner's recipes and saved videos, membership checks for every household-scoped module.

## In scope
- Every new user gets a household of one at registration (listener for the `user registered` event from `:common`, idempotent)
- "Létrehozás" names the user's household; members list
- Invite tokens for QR: valid 24 hours, reusable within that time; creating a new one revokes the previous one
- On join: the joiner's recipes and saved videos merge into the household; the household's plan replaces the joiner's own plan (via an event the mealplan module handles)
- `HouseholdMembershipQuery` interface in `:common`, implemented here, used by all household-scoped modules and the WebSocket handshake

## Out of scope
- Users and registration (`be-auth`, built in parallel)
- Recipe and video CRUD (`be-recipes`)
- Broadcasting membership changes (`be-realtime`)
- Leaving a household (out of v1)
- Account deletion (out of v1)

## Builds on
- `be-platform` principal (user id from the token) and the `user registered` event; tests publish the event and use test tokens instead of real registration
- Shared specs: `households`

## Provides
- Household scoping and authorization for recipes, videos, plans, grocery and realtime channels; endpoints for `mob-households-api`

## Done when
- Household endpoints pass tests; non-members are denied; join merges recipes and videos and replaces the joiner's plan.

## Decisions from the breakdown
- Built in parallel with `be-auth`: it only needs the user id from the token and the registration event (2026-11-26)
- Household owns recipes, saved videos and plans; every user has a household of one from registration (2026-11-23)
- A user belongs to exactly one household; all members are equal (no owner role)
- Join by QR invite only in v1 (no invite link)
- Cross-module writes (plan replacement) go through idempotent event listeners; eventual consistency is accepted (2026-11-23)
- Implements the operations from `api-contract`; springdoc publishes the wire format (2026-11-26)

## Open questions for the deep-dive session
- The household is created through an event (eventually consistent): how requests behave before it exists
- Default name of the auto-created household
- Scanning an invite while your current household has other members (leaving isn't in v1)
