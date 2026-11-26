# api-conventions Specification

## Purpose

Sets the cross-cutting rules every REST and live-connection exchange between the backend and the clients follows (authority, paths, versioning, formats, errors, tokens, request IDs and retries), so both platforms agree before any operation is implemented.

## Requirements

### Requirement: Authoritative sources
The API specification generated from the backend SHALL be authoritative for wire shape: paths, fields and types. The shared specs SHALL be authoritative for meaning: rules, error codes and their semantics. When the two contradict each other, the shared specs SHALL win, and the generated specification SHALL be fixed or a shared-spec change made first.

#### Scenario: Undocumented error code
- **WHEN** the generated specification lists an error code that is not in the shared error catalogue
- **THEN** the backend is wrong until a shared-spec change adds the code

#### Scenario: Field shape
- **WHEN** a client needs the exact name and type of a response field
- **THEN** it takes them from the generated specification

### Requirement: Base path
Every REST operation SHALL live under `/api/v1/`, followed by the plural name of its resource (e.g. `/api/v1/recipes`). The live connection SHALL be at `/ws`. The service health check SHALL be at `/actuator/health`, public, reporting status only.

#### Scenario: Recipe resource
- **WHEN** a client calls a recipe operation
- **THEN** its path starts with `/api/v1/recipes`

### Requirement: Stable major version
The API SHALL stay on version `v1` until a real release. Adding operations, response fields or optional request fields SHALL NOT require a new version and SHALL NOT break existing clients.

#### Scenario: New response field
- **WHEN** the backend adds a field to a response
- **THEN** the path keeps `/api/v1/` and existing clients keep working

### Requirement: Breaking changes go through the shared specs
Before a real release, a breaking API change (removing or renaming a field or operation, changing a type or meaning, adding an enum value) SHALL be made only through a shared-spec change marked BREAKING, which every targeted platform adopts together. After a real release, a breaking change SHALL need a new major version path.

#### Scenario: Renamed field before release
- **WHEN** a response field must be renamed while nothing is released
- **THEN** a BREAKING shared-spec change is made, and backend and mobile migrate together on `/api/v1/`

### Requirement: Household scope from the session
Household-owned resources SHALL be addressed without a household ID in the path. The household SHALL be the one the caller's session belongs to.

#### Scenario: Listing recipes
- **WHEN** a member lists recipes
- **THEN** the path carries no household ID, and only their own household's recipes are returned

### Requirement: JSON bodies
Request and response bodies SHALL be JSON encoded in UTF-8, with the content type `application/json`. Error bodies SHALL use `application/problem+json`.

#### Scenario: Hungarian text
- **WHEN** a recipe named "Túrós csusza" is saved and read back
- **THEN** the body is UTF-8 JSON and the name comes back unchanged

### Requirement: Field naming
JSON field names SHALL be camelCase.

#### Scenario: Multi-word field
- **WHEN** a response carries the time an action was taken
- **THEN** the field is named in camelCase, e.g. `occurredAt`

### Requirement: Absent values
Responses SHALL include every field their schema defines, using `null` for a field without a value and `[]` for an empty collection, never omitting the field. In requests, an optional field sent as `null` and an optional field left out SHALL mean the same.

#### Scenario: Recipe without a note
- **WHEN** a recipe without a note and without steps is read
- **THEN** the response contains `"note": null` and `"steps": []`

#### Scenario: Optional field left out
- **WHEN** a client saves a recipe and leaves out the optional note field
- **THEN** the result is the same as sending `"note": null`

### Requirement: Unknown fields are ignored
Clients SHALL ignore response fields they don't know. The backend SHALL ignore request fields it doesn't know.

#### Scenario: Older client
- **WHEN** a response contains a field the client was built without
- **THEN** the client processes the response and disregards that field

### Requirement: Identifiers
Every identifier SHALL be a UUID sent as a string.

#### Scenario: Recipe ID
- **WHEN** a recipe is returned
- **THEN** its ID is a UUID string such as `"3f2b8c1e-5d4a-4e6b-9c7d-1a2b3c4d5e6f"`

### Requirement: Instants
A point in time SHALL be an ISO-8601 string in UTC with the `Z` suffix. The backend SHALL send at most millisecond precision. Both sides SHALL accept any number of fraction digits.

#### Scenario: Sent instant
- **WHEN** the backend returns when a meal was marked cooked
- **THEN** the value looks like `"2026-11-26T17:04:05.123Z"`

#### Scenario: Finer precision received
- **WHEN** a client sends an action time with microsecond precision
- **THEN** the backend accepts it

### Requirement: Calendar dates and weeks
A calendar date (e.g. a cook day) SHALL be a `YYYY-MM-DD` string in Europe/Budapest time. A week SHALL be identified by the date of its Monday. Every date-based rule SHALL use Europe/Budapest, whatever the device's time zone.

#### Scenario: Week ID
- **WHEN** a client asks for the week containing Thursday 2026-11-26
- **THEN** it identifies the week as `"2026-11-23"`

#### Scenario: Device in another time zone
- **WHEN** a phone set to UTC asks what day it is at 23:30 UTC on 2026-11-26
- **THEN** the date rules treat it as 2026-11-27, Budapest time

### Requirement: Enums
Enum values SHALL be UPPER_SNAKE_CASE strings. Adding a value to an existing enum SHALL count as a breaking change.

#### Scenario: Recipe status
- **WHEN** a recipe is in Kedvencek
- **THEN** its status is sent as an UPPER_SNAKE constant such as `FAVOURITE`

### Requirement: Units
Ingredient units SHALL be sent as the enum constants `G`, `ML`, `DB`, `KK`, `TK`, `EK`, `CSIPET`, `GEREZD`, `FEJ`, `SZAL`, `CSOMAG`, `MAREK`, one for each unit in the fixed unit list. Clients SHALL show the Hungarian abbreviation.

#### Scenario: Unit with an accent
- **WHEN** an ingredient is measured in "szál"
- **THEN** its unit is sent as `SZAL`

### Requirement: Quantities
Ingredient quantities SHALL be JSON numbers.

#### Scenario: Half a unit
- **WHEN** an ingredient needs half a unit
- **THEN** its quantity is sent as `0.5`

### Requirement: Text normalisation
The backend SHALL trim leading and trailing whitespace from text input, then validate and store the trimmed value. Text length limits SHALL be counted in Unicode code points, the same way on every platform.

#### Scenario: Surrounding spaces
- **WHEN** a recipe is saved with the name "  Gulyás  "
- **THEN** it is stored and returned as "Gulyás"

#### Scenario: Accented letters
- **WHEN** a name of 100 characters includes accented letters such as "á"
- **THEN** each accented letter counts as one character, and the name is within the 100-character limit

### Requirement: Whole-resource edits
Saving an edited resource SHALL send the whole resource, which replaces the stored one. Small state changes (status, tick, cooked mark, rename) SHALL be separate operations. Partial update bodies SHALL NOT be used.

#### Scenario: Editing a recipe
- **WHEN** a member edits one ingredient of a recipe and saves
- **THEN** the client sends the complete recipe, and it replaces the stored one

### Requirement: Write responses
A successful create SHALL respond with status 201 and the resource as saved. A successful edit or state change SHALL respond with status 200 and the resulting resource. A successful delete SHALL respond with status 204 and no body.

#### Scenario: Creating a recipe
- **WHEN** a member creates a recipe
- **THEN** the response has status 201 and contains the saved recipe, including its ID and photo key

#### Scenario: Deleting a video
- **WHEN** a member deletes a saved video
- **THEN** the response has status 204 and no body

### Requirement: Deleting a missing resource
Deleting a resource that no longer exists SHALL be rejected with `NOT_FOUND`.

#### Scenario: Two members delete the same recipe
- **WHEN** member B deletes a recipe that member A deleted a moment earlier
- **THEN** B's request is rejected with `NOT_FOUND`

### Requirement: Stable list order
Every list the API returns SHALL come in a defined, stable order stated for that operation. The same data SHALL always come back in the same order.

#### Scenario: Repeated read
- **WHEN** a client reads the same unchanged list twice
- **THEN** both responses list the items in the same order

### Requirement: No pagination
Every list SHALL be returned whole, without pagination.

#### Scenario: Large recipe library
- **WHEN** a household with 300 recipes lists its recipes
- **THEN** all 300 come back in one response

### Requirement: Error body
Every rejected request SHALL return an RFC 9457 problem detail with `type`, `title`, `status` and `detail`, plus `code` holding exactly one code from the shared error catalogue.

#### Scenario: Locked meal
- **WHEN** a member changes a meal whose cook day has passed
- **THEN** the body is a problem detail whose `code` is `MEAL_LOCKED`

### Requirement: Validation field errors
A `VALIDATION_FAILED` rejection SHALL include `errors`, one entry per invalid field, each with `field` and `reason`. `field` SHALL be the path to the input, using dots and zero-based indexes (e.g. `ingredients[2].quantity`). `reason` SHALL be one of `REQUIRED`, `TOO_SHORT`, `TOO_LONG`, `OUT_OF_RANGE`, `INVALID_FORMAT`, `TOO_MANY`.

#### Scenario: Two invalid fields
- **WHEN** a recipe is saved with a blank name and a third ingredient whose quantity is 0
- **THEN** `errors` contains `{field: "name", reason: "REQUIRED"}` and `{field: "ingredients[2].quantity", reason: "OUT_OF_RANGE"}`

#### Scenario: Password without a digit
- **WHEN** a user registers with a password that contains no digit
- **THEN** `errors` contains `{field: "password", reason: "INVALID_FORMAT"}`

### Requirement: User-facing error text
Clients SHALL build the text they show users in Hungarian from `code` and, for validation errors, `reason`. The backend's `title` and `detail` SHALL be English, meant for developers, and SHALL NOT be shown to users.

#### Scenario: Taken email
- **WHEN** registration is rejected with `EMAIL_TAKEN`
- **THEN** the app shows its own Hungarian message for that code, not the response's `detail`

### Requirement: Status mapping
Each code SHALL come with this HTTP status:
- 400: `VALIDATION_FAILED`
- 401: `UNAUTHENTICATED`, `INVALID_CREDENTIALS`, `REFRESH_TOKEN_INVALID`
- 404: `NOT_FOUND`
- 409: `EMAIL_TAKEN`, `ALREADY_MEMBER`, `HOUSEHOLD_NOT_SOLO`, `VERDICT_ALREADY_GIVEN`
- 422: `WEEK_NOT_PLANNABLE`, `MEAL_LOCKED`, `VERDICT_NOT_DUE`, `UNDO_WINDOW_EXPIRED`, `NO_LONGER_APPLICABLE`, `INVITE_INVALID`, `NO_RECIPE_FOUND`
- 500: `INTERNAL_ERROR`

#### Scenario: Plan outside the window
- **WHEN** a member plans a meal three weeks ahead
- **THEN** the response has status 422 and code `WEEK_NOT_PLANNABLE`

### Requirement: Clients decide by code
Clients SHALL decide how to handle a rejection by its `code`, not by its HTTP status.

#### Scenario: Two conflicts
- **WHEN** a join is rejected with status 409
- **THEN** the client tells `HOUSEHOLD_NOT_SOLO` and `ALREADY_MEMBER` apart by the code

### Requirement: Another household's resources are not found
A request for a resource that exists but belongs to another household SHALL be rejected exactly like a request for a resource that doesn't exist: status 404 with `NOT_FOUND`, never a "forbidden" response, so the response doesn't reveal that the resource exists.

#### Scenario: Guessing another household's recipe
- **WHEN** a user reads a recipe ID that belongs to another household
- **THEN** the response is 404 with `NOT_FOUND`, the same as for an unknown ID

### Requirement: Refresh only on an expired session
A client SHALL refresh its session automatically only after a rejection with `UNAUTHENTICATED`. `INVALID_CREDENTIALS` and `REFRESH_TOKEN_INVALID` SHALL NOT trigger a refresh.

#### Scenario: Wrong password
- **WHEN** login is rejected with status 401 and `INVALID_CREDENTIALS`
- **THEN** the client shows the error and does not try to refresh

#### Scenario: Expired access credential
- **WHEN** a recipe request is rejected with `UNAUTHENTICATED`
- **THEN** the client refreshes once and retries the request

### Requirement: Rejections outside the domain rules
A request with a malformed or unreadable body SHALL be rejected with `VALIDATION_FAILED`. A request to a path that doesn't exist SHALL be rejected with `NOT_FOUND`. An unexpected server failure SHALL be rejected with `INTERNAL_ERROR`, without revealing internal details.

#### Scenario: Broken JSON
- **WHEN** a client sends a body that is not valid JSON
- **THEN** the response has status 400 and code `VALIDATION_FAILED`

#### Scenario: Server failure
- **WHEN** an unexpected failure occurs while handling a request
- **THEN** the response has status 500 and code `INTERNAL_ERROR`, with no stack trace or internal names in the body

### Requirement: Access credential transport
The access credential SHALL be sent in the `Authorization` header as `Bearer <token>`, on REST requests and on the live-connection handshake. Its lifetime is defined by the identity spec.

#### Scenario: Authenticated request
- **WHEN** a signed-in client lists recipes
- **THEN** the request carries `Authorization: Bearer` followed by its access credential

### Requirement: Refresh credential transport
The refresh credential SHALL travel only in JSON bodies: login and registration return it, refresh returns a new one, and refresh and logout take it in the body. Cookies SHALL NOT be used.

#### Scenario: Refreshing
- **WHEN** a client refreshes its session
- **THEN** it sends its refresh credential in the request body and receives the new pair in the response body

### Requirement: Live connection handshake
A live connection without a valid access credential SHALL be refused at the handshake with HTTP status 401 and `UNAUTHENTICATED`, before the connection opens.

#### Scenario: Expired credential at connect
- **WHEN** a client opens `/ws` with an expired access credential
- **THEN** the handshake fails with 401 and `UNAUTHENTICATED`, and no connection is opened

### Requirement: Live message format
Each change notice SHALL be a JSON text message with an UPPER_SNAKE `type`, the affected resource, `actorId` and `occurredAt`, following the same format rules as REST bodies.

#### Scenario: Plan notice
- **WHEN** a member changes the plan of week 2026-11-23
- **THEN** devices receive a message with `type` `PLAN_CHANGED`, the week `"2026-11-23"`, the member's ID as `actorId` and `occurredAt` in UTC

### Requirement: Live close codes
The server SHALL close a live connection with a fixed code that tells the client what to do: 4001 `CREDENTIAL_EXPIRED` when the access credential expires (the client refreshes and reconnects), and 4002 `HOUSEHOLD_CHANGED` when the user's household changes (the client reconnects and refetches).

#### Scenario: Credential expires while connected
- **WHEN** the access credential of an open connection expires
- **THEN** the server closes it with 4001, and the client refreshes and reconnects

### Requirement: Household change closes live connections
When a user joins another household, the server SHALL close all of that user's live connections with 4002 `HOUSEHOLD_CHANGED`. On reconnecting, each device SHALL receive the new household's notices and refetch its data.

#### Scenario: Join
- **WHEN** a solo user joins a household while two of their devices are connected
- **THEN** both connections close with 4002, and after reconnecting both devices receive the new household's notices

### Requirement: Request ID
Every REST response, including rejections, SHALL carry an `X-Request-Id` header. Clients SHALL attach it to the error reports they record.

#### Scenario: Failed request
- **WHEN** a request fails with `INTERNAL_ERROR`
- **THEN** the response has an `X-Request-Id`, and the client's error report includes it

### Requirement: Duplicate creates
Clients SHALL NOT send a second create while the first is still in flight. The backend SHALL NOT deduplicate creates.

#### Scenario: Double tap on save
- **WHEN** a member taps "Mentés" twice quickly on a new recipe
- **THEN** the client sends one create, and one recipe is created

### Requirement: Automatic retries
After a timeout or lost connection, clients SHALL retry automatically only reads and the replayable writes (grocery ticks and cooked marks). Every other write SHALL NOT be retried automatically; the user sees the failure and decides.

#### Scenario: Timed-out recipe create
- **WHEN** creating a recipe times out
- **THEN** the client does not resend it on its own and shows the failure

#### Scenario: Timed-out tick
- **WHEN** ticking a grocery line times out
- **THEN** the client may resend the same target-state write
