# Decisions: define-domain-model

Confirmed with Szebenyi Gergely on 2026-11-25.

## Summary

This change sets down the domain concepts and rules that every Csuporka platform (backend and mobile) relies on: identity and sessions, households, the recipe library (including import), the weekly meal plan, the grocery list, live sync, offline replay and a shared catalogue of error codes. It specifies rules, invariants and testable scenarios only. Wire formats stay in the OpenAPI contract. The change is not breaking: no shared specs or contract exist yet.

## Decisions

### D1 — Platforms and breaking status
- **Question**: Which platforms does the change target, and is it breaking?
- **Decision**: Backend and mobile (one Compose Multiplatform app for Android and iOS). Not breaking, since nothing exists yet.
- **Alternatives**: —
- **Why**: Both platform repos reference this store as the home of shared behavior; no specs or contract exist in any store.
- **Depends on**: —

### D2 — Covered domains
- **Question**: Which business domains does this change cover?
- **Decision**: Household and membership; recipe library; calendar and meal plan; grocery list.
- **Alternatives**: Any subset, with the rest deferred to later changes.
- **Why**: —
- **Depends on**: —

### D3 — Covered cross-cutting concepts
- **Question**: Which cross-cutting concepts are specified as shared domain rules?
- **Decision**: Identity and session; live sync notices; offline replay; error vocabulary.
- **Alternatives**: Any subset.
- **Why**: —
- **Depends on**: —

### D4 — Level of detail
- **Question**: How detailed are the shared specs, given the OpenAPI contract holds the wire format?
- **Decision**: Concepts, invariants and observable behavior with scenarios. No endpoints, field names or JSON shapes.
- **Alternatives**: Rules plus contract shapes; glossary only.
- **Why**: Avoids duplicating or competing with the OpenAPI contract.
- **Depends on**: —

### D5 — Change shape and capability layout
- **Question**: How is the change split into capabilities?
- **Decision**: One change with eight capabilities: `identity`, `households`, `recipe-library`, `meal-plan` (including calendar rules), `grocery-list`, `live-sync`, `offline-replay`, `domain-errors` (one catalogue of stable error codes).
- **Alternatives**: A single domain-model spec; separate changes per domain; nine capabilities with a separate `calendar`; error codes listed inside each domain spec.
- **Why**: Each domain can evolve independently in later changes.
- **Depends on**: D2, D3

### D6 — Change id
- **Question**: What is the change called?
- **Decision**: `define-domain-model`.
- **Alternatives**: `establish-v1-domain`.
- **Why**: —
- **Depends on**: —

### D7 — Terminology
- **Question**: Which language do the specs use for domain terms?
- **Decision**: Canonical English names, with the Hungarian UI label given once in each spec (e.g. Favourites = "Kedvencek", New = "Újdonság", taste verdict = "Ízlett?", cooked mark = "Megfőztem").
- **Alternatives**: Hungarian terms throughout; English only.
- **Why**: —
- **Depends on**: —

### D8 — Precedence
- **Question**: After archive, which source wins on shared domain rules?
- **Decision**: The shared specs win on domain rules. Figma and the design docs stay authoritative for UI and copy only.
- **Alternatives**: Design docs keep winning and the specs get corrected.
- **Why**: —
- **Depends on**: —

### D9 — Error vocabulary
- **Question**: How is the shared error vocabulary specified?
- **Decision**: Each domain error gets a stable code name (e.g. `WEEK_NOT_PLANNABLE`) that both platforms rely on. HTTP status mapping stays in OpenAPI.
- **Alternatives**: Only name the conditions and choose the code strings in the OpenAPI contract.
- **Why**: —
- **Depends on**: D4

### D10 — Account deletion
- **Question**: Is account deletion part of the shared domain?
- **Decision**: Account deletion is out of v1 entirely. The docs, which still list it as in scope, need updating.
- **Alternatives**: In v1 but specified in a later change; a mobile-only concern.
- **Why**: Owner decision.
- **Depends on**: —

### D11 — Display name
- **Question**: Can a user change their display name in v1?
- **Decision**: No. It is set at registration and fixed.
- **Alternatives**: Editable, with a rename notice to members.
- **Why**: Matches the current design; Figma shows no rename.
- **Depends on**: —

### D12 — Validation limits
- **Question**: Do the shared specs fix numeric limits?
- **Decision**: Yes:
  - Recipe name: 1–100 characters, trimmed, not blank.
  - Servings: 1–50, optional.
  - Ingredient name: 1–100 characters.
  - Quantity: greater than 0, at most 2 decimals, optional.
  - Ingredients: at most 100 rows.
  - Steps: at most 50, each at most 1000 characters.
  - Note: at most 2000 characters.
  - Video and source URL: http(s), at most 2048 characters.
  - Display name: 1–50 characters.
  - Household name: 1–50 characters.
  - Email: valid format, unique (case-insensitive).
  - Password: 8–72 characters, at least one digit.
- **Alternatives**: Only required-field and format rules, with caps in OpenAPI.
- **Why**: Backend validation and mobile forms must agree.
- **Depends on**: D4

### D13 — Household always exists
- **Question**: Can a signed-in user ever be without a household?
- **Decision**: No. Once registration succeeds, every request sees the user's household. This is a shared invariant the backend guarantees.
- **Alternatives**: Clients handle a brief "household not ready" error.
- **Why**: —
- **Depends on**: —

### D14 — Household name
- **Question**: What is the household's name before "Létrehozás", and can it be renamed?
- **Decision**: A household is unnamed until "Létrehozás" names it. Any member can rename it at any time.
- **Alternatives**: A default name such as "‹Name› háztartása"; named once and fixed.
- **Why**: —
- **Depends on**: —

### D15 — Joining from a shared household
- **Question**: What happens when a user scans an invite while their household has other members?
- **Decision**: The join is rejected. Only a user who is alone in their household can join another.
- **Alternatives**: Move the user alone, leaving their content behind; merge the whole household's content.
- **Why**: Avoids splitting shared content; leaving a household is out of v1.
- **Depends on**: —

### D16 — Join merge
- **Question**: How are the joiner's recipes and videos merged?
- **Decision**: Everything is kept as it is. No deduplication by name or link; every item keeps its Favourites/New status and any pending taste verdict. The household's plan replaces the joiner's plan. The joiner's old household, now empty, ceases to exist.
- **Alternatives**: Dedupe videos by link; land every merged item in New.
- **Why**: —
- **Depends on**: D15

### D17 — Minimum recipe content
- **Question**: What is the minimum content of a saved recipe?
- **Decision**: Only the name is required; everything else is optional. A meal with no ingredients adds nothing to the grocery list.
- **Alternatives**: Name plus an ingredient or a video link; name plus at least one ingredient.
- **Why**: Recipes made from videos and quick capture for planning need it.
- **Depends on**: —

### D18 — Ingredient row
- **Question**: Which parts of an ingredient row are required?
- **Decision**: Only the name. Quantity and unit are optional. A unit requires a quantity. A quantity without a unit means a bare count.
- **Alternatives**: Name and quantity; all three.
- **Why**: —
- **Depends on**: D17

### D19 — Video to recipe
- **Question**: Does a saved video stay in "Mentett videók" after a recipe is made from it?
- **Decision**: "Ízlett" on a video converts it: the new recipe keeps the link and the video leaves Mentett videók. A manual "make a recipe from it" keeps the video.
- **Alternatives**: The video always stays; the video is always removed.
- **Why**: —
- **Depends on**: —

### D20 — "Nem ízlett" undo window
- **Question**: Does the backend enforce the undo window after "Nem ízlett", and for how long?
- **Decision**: Yes, 10 seconds after the verdict. After that the recipe stays deleted.
- **Alternatives**: Client-only enforcement; 30 s; 5 s.
- **Why**: Covers a typical toast plus network delay.
- **Depends on**: —

### D21 — Deleting a planned recipe or video
- **Question**: When a planned recipe or video is deleted, which side does today fall on?
- **Decision**: Today is emptied. Days from today onward become unplanned, which also removes any cooked mark on them. Days before today keep a "Törölt recept" entry.
- **Alternatives**: Today is kept along with the past.
- **Why**: Owner decision.
- **Depends on**: —

### D22 — Taste verdict due
- **Question**: When is a recipe or video waiting for a taste verdict?
- **Decision**: When it is in New, one of its meals was marked cooked on an earlier day, and nobody has answered. The first answer settles it for the whole household. Moving the item to Favourites by hand also settles it.
- **Alternatives**: Every cooked item without a verdict asks once, whatever its status.
- **Why**: —
- **Depends on**: —

### D23 — Concurrent edits
- **Question**: What happens when two members edit the same recipe or plan days at once?
- **Decision**: Last write wins. Other devices refresh through change notices.
- **Alternatives**: Version-checked saves that reject stale edits.
- **Why**: Good enough for a household.
- **Depends on**: —

### D24 — Import in scope
- **Question**: Are website import and video link preview part of this change?
- **Decision**: Yes, inside `recipe-library`. Import returns a draft and never saves it. Units are converted to the fixed unit list. A page without recipe data gives a "no recipe found" error. Video preview is best effort, and an empty preview is not an error.
- **Alternatives**: A separate later change.
- **Why**: —
- **Depends on**: D5

### D25 — Editable days
- **Question**: Can today be planned, and can a meal whose cook day has passed be changed?
- **Decision**: Days from today onward in this and next week can be planned. A meal whose cook day is before today is locked: it can't be edited or removed.
- **Alternatives**: Trim remaining leftover days of a started meal; tomorrow onward only.
- **Why**: —
- **Depends on**: —

### D26 — Overlap replace
- **Question**: What happens to an existing meal that a new meal overlaps?
- **Decision**: Every meal touching the new meal's days is removed entirely, including its cooked mark.
- **Alternatives**: Truncate when only leftover days overlap.
- **Why**: Simple and predictable.
- **Depends on**: D25

### D27 — Swapping a cooked meal's recipe
- **Question**: What happens to the cooked mark when a meal's recipe is swapped?
- **Decision**: The mark is removed. Changing the meal's length keeps it.
- **Alternatives**: Keep the mark.
- **Why**: The cooked dish wasn't the new one.
- **Depends on**: —

### D28 — Derived day rules
- **Question**: Where are today's status and "next week fully planned" specified?
- **Decision**: In the shared `meal-plan` spec, as domain definitions. Day status is cook, leftover, done or nothing. A week is fully planned when every day is covered by a cook or leftover day.
- **Alternatives**: Mobile store only.
- **Why**: Keeps any future platform or the backend consistent.
- **Depends on**: D5

### D29 — Past grocery lists
- **Question**: Does a past week's grocery list change when recipes are edited later?
- **Decision**: No. It freezes at Monday 00:00 (Budapest), when the week ends, with its lines and ticks as they were.
- **Alternatives**: Always regenerated from the current recipes.
- **Why**: Shows what was really bought.
- **Depends on**: —

### D30 — Tick identity across regeneration
- **Question**: When does a regenerated grocery line keep its tick?
- **Decision**: When the meal and ingredient are the same and the name, quantity and unit are unchanged. Any change unticks the line.
- **Alternatives**: Tie the tick to the recipe's ingredient row, so a name-only fix keeps it.
- **Why**: —
- **Depends on**: —

### D31 — Late offline tick
- **Question**: Is a tick made before the week froze, but replayed after it, accepted?
- **Decision**: Yes. A tick carries the time it was made, and one made before the freeze still applies.
- **Alternatives**: Reject as frozen.
- **Why**: Same rule as the cooked mark.
- **Depends on**: D29

### D32 — Notice recipients and actor
- **Question**: Which devices get a change notice, and does it name the actor?
- **Decision**: Every connected device of the household gets it, the originator included. The notice carries the acting user, so clients can skip their own "someone changed it" toasts.
- **Alternatives**: Exclude the originating device; send no actor.
- **Why**: —
- **Depends on**: —

### D33 — Replay that no longer applies
- **Question**: What happens when a queued offline write no longer applies (meal removed, line gone, mark past its cook day)?
- **Decision**: The server rejects it with a defined "no longer applicable" code. The client drops it silently and shows the refreshed state.
- **Alternatives**: Drop it and tell the user.
- **Why**: —
- **Depends on**: D9

### D34 — Concurrent target-state writes
- **Question**: Two members set the same tick or cooked mark differently, one of them offline. Which state wins?
- **Decision**: Writes apply in the order they reach the server.
- **Alternatives**: The latest client time wins.
- **Why**: Simple, and matches the idempotent target-state design.
- **Depends on**: —

### D35 — Offline split
- **Question**: What part of offline behavior is shared?
- **Decision**: Replay rules only:
  - Which writes may be replayed: grocery ticks and cooked mark/unmark.
  - These writes set a target state.
  - Each carries the time it was made.
  - When a replay is dropped.

  What is cached, and read-only mode, belong to the mobile store.
- **Alternatives**: Also fix what must be readable offline.
- **Why**: —
- **Depends on**: D5

### D36 — One recipe link
- **Question**: Can a recipe have both a website link and a video link?
- **Decision**: No. A recipe has at most one link, which is either a video or the web page it came from.
- **Alternatives**: Separate optional source link and video link.
- **Why**: A recipe comes either from a website or from a video, never both.
- **Depends on**: D17

### D37 — Undo of "Nem ízlett" restores everything
- **Question**: What does undoing "Nem ízlett" bring back?
- **Decision**: Everything: the item, its planned days and the pending verdict. There is no rule against planning untested recipes; in practice the household doesn't plan them ahead.
- **Alternatives**: Restore the item only; forbid planning items that are waiting for a verdict.
- **Why**: Keeps it simple.
- **Depends on**: D20, D21

### D38 — Cooked mark on the wrong day
- **Question**: Which code rejects a cooked mark sent on a day other than the cook day (e.g. the app left open past midnight)?
- **Decision**: `NO_LONGER_APPLICABLE`, the same code used for stale offline writes. The app drops the action and refreshes.
- **Alternatives**: A separate `NOT_COOK_DAY` code.
- **Why**: Rare case, and the app handles it the same way.
- **Depends on**: D33

## Facts

Facts looked up during the review that decisions rely on.

- No specs exist in this store, the backend repo or the mobile repo. The OpenAPI contract (`contract/openapi.yaml`) has not been written yet. — source: `openspec list --specs`; `~/IDEA Projects/csuporka/`; `~/Projektek/Csuporka/openspec/`
- The backend and mobile repos both reference `csuporka-shared-specs` for shared behavior and keep only platform-specific specs. — source: `openspec/config.yaml` in both repos
- Sign-in is email/password only. The access token lasts 50 min. The refresh token lasts 30 days and rotates on every use. There is one refresh family per login/device. Reusing a token revokes only that family. Logout ends the current device's session. — source: docs/breakdown/chunks/04-be-auth.md, 01-api-contract.md
- Every user gets a household of one at registration. A user belongs to exactly one household, and all members are equal. — source: docs/breakdown/INTERVIEW.md › Product rules
- Invites are QR only, valid 24 h, reusable within that time. A new invite revokes the previous one. — source: docs/breakdown/INTERVIEW.md › Product rules
- The household owns recipes, saved videos and plans. Favourites/New is household-wide, and each recipe is in exactly one. — source: docs/breakdown/chunks/19-be-recipes.md
- Fixed unit list: g, ml, db, kk, tk, ek, csipet, gerezd, fej, szál, csomag, marék. — source: docs/breakdown/OVERVIEW.md
- The recipe photo key is chosen at random from a fixed set at creation and kept on edits. There is no photo upload. — source: docs/breakdown/chunks/19-be-recipes.md
- Videos have no Favourites status; they live in New › "Mentett videók". Deleting a video keeps recipes made from it. — source: docs/breakdown/chunks/15-mob-recipe-editor.md, 16-mob-recipe-library.md
- Time zone is Europe/Budapest. Weeks start on Monday and are identified by their Monday's date. There is one lunch per day. A meal is a recipe or saved video with a cook day and a length of 1–3 consecutive days, and has no servings. — source: docs/breakdown/OVERVIEW.md, INTERVIEW.md
- Only this week and next week can be planned, and every past week can be viewed. A meal may run into next week when that week is plannable. A meal starting next week can't run past its Sunday. — source: docs/breakdown/chunks/13-mob-planner.md
- A cooked mark records who marked it and when. It can only be set on the cook day itself, judged by the marking time the client sends, so a replayed offline mark still counts. Unmark is allowed until the day ends. — source: docs/breakdown/chunks/21-be-meal-plan.md, 17-mob-today.md
- The grocery list is generated on the backend from the recipes' stored amounts. A multi-day meal counts once, in its cook day's week. The merged view lists every line with its recipe without adding amounts, grouping names ignoring case and accents. There is one tick per line, shared by both views and the household. Video meals appear as entries without ingredients. — source: docs/breakdown/chunks/23-be-grocery.md
- Real-time messages are change notices only (type plus affected resource). There is no server event log, and clients refetch after reconnecting. — source: docs/breakdown/chunks/18-be-realtime.md
- Only grocery ticks and cooked mark/unmark are replayed offline, as idempotent target-state writes. The taste verdict is online only. — source: docs/breakdown/chunks/01-api-contract.md
- Search runs on the phone, so there is no search endpoint. — source: docs/breakdown/OVERVIEW.md
- The docs still list account deletion as in v1 scope, which conflicts with D10. — source: docs/breakdown/OVERVIEW.md, chunks/07-be-households.md, chunks/11-mob-households.md

## Out of scope

- Account deletion (out of v1), including the last-member household deletion cascade
- Leaving a household
- Server push and email notifications; the weekly planning reminder is local to the phone and specified in the mobile store
- Google and Apple sign-in; email verification; password reset or change
- Editing the display name
- Rate limiting
- Manual extra grocery items; servings scaling into the grocery list
- Photo upload and website photos; video recipe extraction; AI extraction; share-sheet import
- Offline editing of recipes, videos or the plan; what the phone caches and read-only mode (mobile store)
- Wire formats: endpoints, field names, JSON shapes, HTTP status codes (OpenAPI contract)
- UI wording and layout (Figma and design docs)

## Deferred

- Update docs/breakdown (OVERVIEW scope, chunks 07 and 11) to drop account deletion from v1 — settle before: implementation
