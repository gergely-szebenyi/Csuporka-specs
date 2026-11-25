# domain-errors Specification

## Purpose

Lists the stable domain error codes every platform relies on, so each rejection means the same thing on the backend and in every client, independent of transport details.

## Requirements

### Requirement: Stable error codes
Every rejected request SHALL carry exactly one code from this catalogue. Codes SHALL never be renamed or reused for another meaning. The mapping to transport status codes belongs to the API contract, not to this spec.

#### Scenario: Same code everywhere
- **WHEN** a member plans a meal outside the plannable window
- **THEN** the rejection carries the code `WEEK_NOT_PLANNABLE`, whichever client sent it

### Requirement: Validation errors name the fields
A request rejected with `VALIDATION_FAILED` SHALL name each invalid field, so a client can show the problem next to the right input.

#### Scenario: Two invalid fields
- **WHEN** a registration has a blank display name and a 5-character password
- **THEN** the rejection is `VALIDATION_FAILED` naming the display name and the password

### Requirement: Authentication and identity codes
The catalogue SHALL contain these codes:
- `UNAUTHENTICATED`: the access credential is missing, invalid or expired
- `INVALID_CREDENTIALS`: login with a wrong email or password
- `EMAIL_TAKEN`: registration with an email already in use
- `REFRESH_TOKEN_INVALID`: the refresh credential is expired, revoked, reused or unknown

#### Scenario: Reused refresh credential
- **WHEN** a device presents a refresh credential that was already rotated
- **THEN** the rejection carries `REFRESH_TOKEN_INVALID`

### Requirement: General codes
The catalogue SHALL contain these codes:
- `VALIDATION_FAILED`: input breaks a format or limit rule
- `NOT_FOUND`: the resource doesn't exist or belongs to another household

#### Scenario: Another household's meal
- **WHEN** a user changes a meal of another household
- **THEN** the rejection carries `NOT_FOUND`

### Requirement: Household codes
The catalogue SHALL contain these codes:
- `INVITE_INVALID`: the invite is unknown, expired or revoked
- `HOUSEHOLD_NOT_SOLO`: the joiner's current household has other members
- `ALREADY_MEMBER`: the invite belongs to the joiner's own household

#### Scenario: Expired invite
- **WHEN** a user joins with an invite older than 24 hours
- **THEN** the rejection carries `INVITE_INVALID`

### Requirement: Recipe library codes
The catalogue SHALL contain these codes:
- `VERDICT_NOT_DUE`: the item isn't waiting for a taste verdict
- `VERDICT_ALREADY_GIVEN`: another member already answered
- `UNDO_WINDOW_EXPIRED`: the 10-second undo of "Nem ízlett" has passed
- `NO_RECIPE_FOUND`: an import page couldn't be fetched or has no recipe data

#### Scenario: Late undo
- **WHEN** a member undoes "Nem ízlett" after 15 seconds
- **THEN** the rejection carries `UNDO_WINDOW_EXPIRED`

### Requirement: Meal plan and grocery codes
The catalogue SHALL contain these codes:
- `WEEK_NOT_PLANNABLE`: a day or week is outside what may be planned or viewed
- `MEAL_LOCKED`: the meal's cook day is before today
- `NO_LONGER_APPLICABLE`: a tick or cooked mark whose target is gone, or whose action time breaks a time-bound rule

#### Scenario: Replayed write that no longer applies
- **WHEN** a queued tick arrives for a line that was removed
- **THEN** the rejection carries `NO_LONGER_APPLICABLE`
