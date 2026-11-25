# Backend households & membership

| | |
|---|---|
| ID | `be-households` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 3 |
| Depends on | `be-auth` |
| Unlocks | `be-realtime`, `be-recipes` |

## Goal
Household of one for every new user, naming it ("Létrehozás"), QR invite tokens, join with merge of the joiner's recipes and saved videos, membership checks for every household-scoped module.

## In scope
- Every new user gets a household of one at registration (listener for the auth module's user-registered event, idempotent)
- "Létrehozás" names the user's household; members list
- Invite tokens for QR: valid 24 hours, reusable within that time; creating a new one revokes the previous one
- On join: the joiner's recipes and saved videos merge into the household; the household's plan replaces the joiner's own plan (via an event the mealplan module handles)
- `HouseholdMembershipQuery` interface in `:common`, implemented here, used by all household-scoped modules and the WebSocket handshake

## Out of scope
- Recipe and video CRUD (be-recipes)
- Broadcasting membership changes (be-realtime)
- Leaving a household (out of v1)
- Account deletion (out of v1)

## Builds on
- be-auth users and principal

## Provides
- Household scoping and authorization for recipes, videos, plans, grocery and realtime channels

## Done when
- Household endpoints pass tests; non-members are denied; join merges recipes and videos and replaces the joiner's plan.

## Decisions from the breakdown
- Household owns recipes, saved videos and plans; every user has a household of one from registration (2026-11-23)
- A user belongs to exactly one household
- All members are equal (no owner role)
- Join by QR invite only in v1 (no invite link)
- Cross-module writes (plan replacement) go through idempotent event listeners; eventual consistency is accepted (2026-11-23)

## Open questions for the deep-dive session
- The household is created through an event (eventually consistent): how requests behave before it exists
- Default name of the auto-created household
- Scanning an invite while your current household has other members (leaving isn't in v1)
