# Backend website recipe import & video link preview

| | |
|---|---|
| ID | `be-recipe-import` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 3 |
| Depends on | `api-contract` (import and link-preview operations), `be-platform` (security chain, error codes, test helpers) |
| Unlocks | `mob-recipes-api` |

## Goal
Fetch a URL safely and extract a recipe draft from schema.org Recipe structured data; read the title and thumbnail of a video link for saved videos.

## In scope
- Safe URL fetching (SSRF protection, timeouts, size limits)
- schema.org Recipe extraction from JSON-LD and microdata: name, servings, ingredients, steps, source URL (no photo)
- Ingredient line parsing into quantity/unit/name (Hungarian and English), converted into the unit list (g, ml, db, kk, tk, ek, csipet, gerezd, fej, szál, csomag, marék) when the source uses another unit (e.g. dkg → g, l → ml)
- Returning a draft for review, not saving
- Video link preview: title, thumbnail URL and source domain for any link, best effort (empty result is not an error)
- Fixture test suite of real sites

## Out of scope
- Saving the recipe or video (client calls `be-recipes` after review)
- Video recipe extraction and AI extraction (out of v1)
- Website photos (v1 doesn't use them)

## Builds on
- `be-platform` security chain (authenticated endpoints, tested with test tokens)
- Shared specs: `recipe-library` (import rules)

## Provides
- Import and link-preview endpoints used by `mob-recipes-api`

## Done when
- Import endpoint returns correct drafts for a fixture set of real recipe sites (incl. Hungarian ones) and the specified error for pages without recipe data; link preview returns title and thumbnail for common video sites (YouTube, TikTok, Facebook) and an empty preview otherwise.

## Decisions from the breakdown
- Depends on `be-platform` only, so it runs in wave 3 next to `be-auth` (2026-11-26)
- schema.org parsing only, no AI; synchronous request, no job queue, no 'recipe ready' push
- jsoup for HTML and JSON-LD; video previews via oEmbed (YouTube, TikTok, ...) with Open Graph tags as the fallback (2026-11-23)
- User-supplied URLs are fetched by a dedicated SSRF-safe fetcher, not an `@HttpExchange` client
- Implements the operations from `api-contract`; springdoc publishes the wire format (2026-11-26)

## Open questions for the deep-dive session
- Handling sites that block bots
