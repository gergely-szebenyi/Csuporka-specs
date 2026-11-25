# identity Specification

## Purpose

Defines who a user is, how they register and sign in with email and password, and how per-device sessions are kept, refreshed and ended, so every platform authenticates users the same way.

## Requirements

### Requirement: User account
A user SHALL have an email, a password and a display name. The display name is shown to other members of the user's household. Email and password are the only way to sign in; there is no third-party sign-in, email verification or password reset.

#### Scenario: Account holds a display name
- **WHEN** a user registers with name "Anna", an email and a password
- **THEN** the account stores "Anna" as the display name shown to household members

### Requirement: Registration validation
Registration SHALL require a display name of 1–50 characters after trimming, a valid email address, and a password of 8–72 characters containing at least one digit. Invalid input SHALL be rejected with `VALIDATION_FAILED`, naming each invalid field.

#### Scenario: Password without a digit
- **WHEN** a user registers with the password "almafaalma"
- **THEN** registration is rejected with `VALIDATION_FAILED` for the password

#### Scenario: Blank display name
- **WHEN** a user registers with a display name of only spaces
- **THEN** registration is rejected with `VALIDATION_FAILED` for the display name

#### Scenario: Valid input
- **WHEN** a user registers with name "Anna", email "anna@example.com" and password "almafa12"
- **THEN** registration succeeds

### Requirement: Unique email
Emails SHALL be unique across all users, compared case-insensitively. Registering with an email that is already taken SHALL be rejected with `EMAIL_TAKEN`.

#### Scenario: Same email in different case
- **WHEN** "anna@example.com" is registered and someone registers "Anna@Example.com"
- **THEN** registration is rejected with `EMAIL_TAKEN`

### Requirement: Display name is fixed
A user's display name SHALL NOT be changeable after registration in v1.

#### Scenario: No rename
- **WHEN** a signed-in user looks for a way to change their display name
- **THEN** no operation exists to change it

### Requirement: Registration signs the user in
A successful registration SHALL start a session for the registering device, exactly as a login would, and the user SHALL already belong to a household (see `households`).

#### Scenario: Signed in after registering
- **WHEN** registration succeeds
- **THEN** the device holds a new session and can call household-scoped operations right away

### Requirement: Login
A user SHALL sign in with email (matched case-insensitively) and password. A wrong email or wrong password SHALL be rejected with the same `INVALID_CREDENTIALS` code, without revealing which one was wrong.

#### Scenario: Wrong password
- **WHEN** a user logs in with a registered email and a wrong password
- **THEN** login is rejected with `INVALID_CREDENTIALS`

#### Scenario: Unknown email
- **WHEN** a user logs in with an email that has no account
- **THEN** login is rejected with `INVALID_CREDENTIALS`

### Requirement: Per-device sessions
Each login or registration SHALL start an independent session for that device, made of a short-lived access credential (valid 50 minutes) and a refresh credential (valid 30 days). A user MAY have sessions on several devices at once.

#### Scenario: Two devices
- **WHEN** a user logs in on a phone and later on a second phone
- **THEN** both phones hold separate sessions that work at the same time

### Requirement: Session refresh with rotation
Presenting a valid refresh credential SHALL issue a new access credential and a new refresh credential, and SHALL invalidate the presented one. An expired, revoked or unknown refresh credential SHALL be rejected with `REFRESH_TOKEN_INVALID`.

#### Scenario: Refresh rotates
- **WHEN** a device refreshes with its current refresh credential
- **THEN** it receives a new pair and the old refresh credential no longer works

#### Scenario: Expired refresh credential
- **WHEN** a device refreshes with a refresh credential older than 30 days
- **THEN** the refresh is rejected with `REFRESH_TOKEN_INVALID` and the device must log in again

### Requirement: Refresh reuse ends only that device's session
Presenting a refresh credential that was already used SHALL revoke that device's whole session and be rejected with `REFRESH_TOKEN_INVALID`. Sessions of the same user on other devices SHALL stay valid.

#### Scenario: Reused refresh credential
- **WHEN** a refresh credential that was already rotated is presented again
- **THEN** that device's session is revoked, and the user's other devices stay signed in

### Requirement: Logout
Logout SHALL end the current device's session only. Its refresh credential SHALL stop working, and other devices SHALL stay signed in.

#### Scenario: Logout on one phone
- **WHEN** a user logs out on one phone
- **THEN** that phone's refresh credential is rejected afterwards, and the user's other phone stays signed in

### Requirement: Authenticated access
Every operation except registration, login, refresh and the service health check SHALL require a valid access credential. A missing, invalid or expired one SHALL be rejected with `UNAUTHENTICATED`.

#### Scenario: Expired access credential
- **WHEN** a device calls a recipe operation with an access credential issued 51 minutes ago
- **THEN** the call is rejected with `UNAUTHENTICATED`, and the device may refresh and retry
