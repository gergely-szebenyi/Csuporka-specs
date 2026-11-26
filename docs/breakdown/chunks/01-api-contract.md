# Behavioral API contract (operations)

| | |
|---|---|
| ID | `api-contract` |
| Area | API Contract |
| Size | M |
| Wave | 1 |
| Depends on | none |
| Unlocks | `be-auth`, `be-households`, `be-recipe-import`, `mob-auth-ui`, `mob-grocery-ui`, `mob-households-ui`, `mob-planner-ui`, `mob-recipe-editor-ui`, `mob-recipe-library-ui`, `mob-today-ui` |

## Goal
Write down the API's operations as behaviour in the shared specs, so backend feature chunks implement the same operations the wave-3 mobile stubs mirror, while springdoc output stays the wire contract.

## In scope
- One OpenSpec change in this store (shared-specs-only schema: proposal + delta specs), planned with `careful-planning`, then proposed and archived
- Operations for identity (register, login, refresh, logout, current user), households (current household, name, members, invite, join), recipe library (list, details, create, edit, delete, Kedvencek / Újdonság status, "Ízlett?" verdict and undo, saved videos, website import draft, video link preview), meal plan (week query, add, replace on overlap, remove, change length, swap recipe, cooked mark / unmark) and grocery (week list in both views, tick / untick)
- For every operation: inputs and results in domain terms, session required or public, the error codes it can return (from domain-errors), whether it is an idempotent target-state write that offline replay may resend, and the live-sync notices it triggers
- Reads that the offline cache and on-phone search rely on, e.g. the recipe list returns full recipes
- Gaps in the existing shared specs found while mapping the Figma flows to operations

## Out of scope
- Paths, HTTP methods, field names, JSON shapes, status codes: springdoc output from each backend chunk (code-first)
- Real-time wire format and close codes (`be-realtime`)
- Implementation on either side; platform-specific behaviour (each platform's own spec store)

## Builds on
- The shared specs from the archived `2026-11-25-define-domain-model` change
- The Figma flows, `design/DESIGN-DECISIONS.md` and the chunk files

## Provides
- The operations every backend feature chunk implements
- The operations the `mob-*-ui` repository interfaces and stubs mirror, so the `*-api` chunks only add mapping

## Done when
- The change is archived into openspec/specs; every client action in the Figma flows maps to an operation, every operation lists its error codes from domain-errors, and every write names its live-sync notice.

## Decisions from the breakdown
- Behavioural, not wire: the contract names operations and their rules, never endpoints or JSON (2026-11-26)
- Code-first stays: each backend chunk's springdoc output is the wire contract (2026-11-26)
- Wave-1 root next to the two scaffolding chunks; it doesn't lengthen the critical path (2026-11-26)

## Open questions for the deep-dive session
- One new capability (e.g. `api-operations`) vs operation requirements added to each existing capability
- Granularity: e.g. whether change length and swap recipe are separate operations or one meal edit
- Whether list sizes need limits or paging (probably not in v1: lists are small)
