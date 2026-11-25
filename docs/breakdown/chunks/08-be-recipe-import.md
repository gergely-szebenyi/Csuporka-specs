# Backend website recipe import & video link preview

| | |
|---|---|
| ID | `be-recipe-import` |
| Area | Backend (Spring Boot) |
| Size | M |
| Wave | 3 |
| Depends on | `be-auth` (endpoint is authenticated) |
| Unlocks | `mob-integration` |

## Goal
Fetch a URL safely and extract a recipe draft from schema.org Recipe structured data (JSON-LD/microdata), normalizing ingredients into quantity/unit/name; clear error when nothing is found. Read the title and thumbnail of a video link for saved videos.

## In scope
- Safe URL fetching (SSRF protection, timeouts, size limits)
- schema.org Recipe extraction from JSON-LD and microdata: name, servings, ingredients, steps, source URL (no photo)
- Ingredient line parsing into quantity/unit/name (Hungarian and English), converted into the unit list (g, ml, db, kk, tk, ek, csipet, gerezd, fej, szál, csomag, marék) when the source uses another unit (e.g. dkg → g, l → ml)
- Returning a draft for review — not saving
- Video link preview: title, thumbnail URL and source domain for any link, best effort (empty result is not an error)
- Fixture test suite of real sites

## Out of scope
- Saving the recipe or video (client calls be-recipes after review)
- Video recipe extraction and AI extraction (out of v1)
- Website photos (v1 doesn't use them)

## Builds on
- be-auth security chain

## Provides
- Import and link-preview endpoints used by mob-recipe-editor

## Done when
- Import endpoint returns correct drafts for a fixture set of real recipe sites (incl. Hungarian ones) and the specified error for pages without recipe data; link preview returns title and thumbnail for common video sites (YouTube, TikTok, Facebook) and an empty preview otherwise.

## Decisions from the breakdown
- schema.org parsing only, no AI
- Synchronous request, no job queue, no 'recipe ready' push

- jsoup for HTML and JSON-LD; video previews via oEmbed (YouTube, TikTok, ...) with Open Graph tags as the fallback (2026-11-23)
- User-supplied URLs are fetched by a dedicated SSRF-safe fetcher, not an `@HttpExchange` client

## Open questions for the deep-dive session
- Handling sites that block bots
