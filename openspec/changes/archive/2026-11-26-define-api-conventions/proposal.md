# Proposal

## Breaking change

None. No API is implemented on either platform yet.

## Platforms

- Backend
- Mobile app (Android and iOS)

## Why

The wire contract is code-first: it is generated from the backend once each feature exists. Before that, both platforms need one shared understanding of how every API exchange works (paths, formats, errors, tokens, live connection), so the backend generates what the app expects and the app's error and retry handling matches the backend.

## What Changes

- Name the authoritative sources: the generated spec for wire shape, the shared specs for meaning; shared specs win a conflict.
- Fix the base path `/api/v1/<plural-resource>`, the `/ws` live endpoint and the versioning rules (stay on v1; additive changes free; breaking changes only through a BREAKING shared-spec change while unreleased).
- Scope household resources by the caller's session, never by an ID in the path.
- Define body conventions: JSON in UTF-8, camelCase, always-present fields with `null`, `[]` for empty lists, unknown fields ignored, whole-resource edits, created/edited resource returned, no pagination, stable list order.
- Define data formats: UUID IDs, ISO-8601 UTC instants, Budapest calendar dates and Monday week IDs, UPPER_SNAKE enums (including units), numeric quantities, trimmed text measured in code points.
- Define the error format (ProblemDetail with `code`, field-level `reason`s for validation), English developer text with client-side Hungarian messages, and the mapping of codes to HTTP status, including 404 instead of 403 for other households' resources.
- Add `INTERNAL_ERROR` to the error catalogue.
- Define token transport, the live connection's message format and close codes, the `X-Request-Id` header, and which requests clients may retry automatically.

## Capabilities

### New Capabilities
- `api-conventions`: cross-cutting rules every REST and live-connection exchange follows: authority, paths, versioning, formats, errors and status mapping, token transport, close codes, request IDs and retries.

### Modified Capabilities
- `domain-errors`: the general codes gain `INTERNAL_ERROR` for unexpected server failures.
