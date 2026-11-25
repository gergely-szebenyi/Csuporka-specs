# Proposal

## Breaking change

None. No shared specs or API contract exist yet.

## Platforms

- Backend
- Mobile app (Android and iOS)

## Why

Csuporka's domain rules are spread across planning docs and both platform repos, and some contradict each other or are still open. Both platforms need one agreed source for the concepts and rules they must implement the same way. Without it, backend validation, mobile logic and the offline queue will drift apart.

## What Changes

- Define users, credentials and per-device sessions. Email and password only, with fixed validation limits.
- Define households: every user always has exactly one household, and all members are equal. Covers naming, the 24 h QR invite, and joining (allowed only when the joiner is alone, merges content as is, and replaces the joiner's plan).
- Define the recipe library:
  - Recipes, where only the name is required.
  - Ingredients in a fixed unit list.
  - Saved videos.
  - Household-wide Favourites/New status.
  - The taste verdict, with a 10 s server-enforced undo of "Nem ízlett".
  - Deletion effects on the plan.
  - Website import and video link preview.
- Define the weekly meal plan:
  - Budapest time and Monday weeks.
  - The plannable window (today onward, this and next week).
  - Meals of 1–3 days, overlap replacement and locked past meals.
  - The cooked mark.
  - Day status, and when a week counts as fully planned.
- Define the grocery list:
  - Generation from the plan, with multi-day meals counted once.
  - Lines grouped by name without adding amounts.
  - One shared tick per line, kept only while the line is unchanged.
  - Lists frozen when the week ends.
- Define live sync. Every change produces a notice to all connected devices of the household, naming the actor. Clients refetch, and there is no event log.
- Define offline replay. Only grocery ticks and cooked mark/unmark replay. They set a target state, carry the time they were made, apply in arrival order, and are dropped when they no longer apply.
- Define a catalogue of stable domain error codes that both platforms rely on.

## Capabilities

### New Capabilities
- `identity`: users, credentials, validation and per-device sessions
- `households`: household, membership, naming, invites, joining and merge
- `recipe-library`: recipes, ingredients, units, saved videos, status, taste verdict, deletion, import and link preview
- `meal-plan`: calendar rules, plannable window, meals, overlap, cooked mark, day status and fully planned week
- `grocery-list`: generation, lines, grouping, shared ticks, regeneration and freezing
- `live-sync`: which changes produce notices, who receives them and what they carry
- `offline-replay`: replayable writes, action time, ordering and drop rules
- `domain-errors`: catalogue of stable error codes

### Modified Capabilities
