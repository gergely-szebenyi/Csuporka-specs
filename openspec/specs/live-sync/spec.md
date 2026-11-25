# live-sync Specification

## Purpose

Defines how household members' devices learn about changes made by others: which changes produce notices, who receives them, what a notice carries, and how clients catch up after being disconnected.

## Requirements

### Requirement: Authenticated household connection
A signed-in device SHALL be able to hold a live connection that delivers notices for its user's household. A connection without a valid access credential SHALL be refused before it opens.

#### Scenario: Invalid credential
- **WHEN** a device opens a live connection with an expired access credential
- **THEN** the connection is refused

### Requirement: Connection ends with the credential
When the access credential of an open connection expires, the server SHALL end the connection with a defined reason, so the client can refresh its session and reconnect.

#### Scenario: Credential expires while connected
- **WHEN** a device's access credential expires during an open connection
- **THEN** the connection ends with the "credential expired" reason

### Requirement: Change notices, not data
Notices SHALL only say what changed: the kind of change, the affected resource (e.g. the plan of week 2026-11-23) and the acting user. Clients SHALL refetch the affected resource to get the new state.

#### Scenario: Plan changed
- **WHEN** a member plans a meal in week 2026-11-30
- **THEN** each device receives a "plan changed" notice for week 2026-11-30, with no meal data in it

### Requirement: Changes that produce notices
A notice SHALL be sent whenever these change for a household:
- its recipes or saved videos, including status, verdict and deletion
- the plan of a week, including cooked marks
- the grocery list or ticks of a week
- its name or members

#### Scenario: Grocery tick
- **WHEN** a member ticks a grocery line
- **THEN** a "grocery changed" notice for that week is sent

#### Scenario: Recipe edit changes the grocery list
- **WHEN** a member edits a recipe that is planned this week
- **THEN** a "recipes changed" notice and a "grocery changed" notice for this week are sent

### Requirement: All household devices receive notices
Every connected device of every household member SHALL receive each notice, including the device that made the change. Devices of other households SHALL NOT receive it.

#### Scenario: Originating device
- **WHEN** a member renames a recipe on their phone
- **THEN** that phone, the member's other devices and the other members' devices all receive the notice

### Requirement: Notices name the actor
Each notice SHALL identify the member who made the change, so a client can tell its own user's changes from other members'. A notice that follows from another change (e.g. a grocery list regenerated after a recipe edit) SHALL carry the actor of that change.

#### Scenario: Toast only for others
- **WHEN** a device receives a "plan changed" notice whose actor is its own user
- **THEN** it can skip the "another member changed the plan" message

### Requirement: Membership change moves the connection
When a user joins another household, the user's devices SHALL stop receiving notices of the old household and SHALL receive those of the new one. Members of the joined household SHALL receive a "members changed" notice.

#### Scenario: Join
- **WHEN** a solo user joins a household
- **THEN** the existing members get a "members changed" notice, and the joiner's devices receive the new household's notices from then on

### Requirement: No catch-up log
Notices missed while a device was disconnected SHALL NOT be replayed. After connecting or reconnecting, a client SHALL refetch the data it shows.

#### Scenario: Reconnect after background
- **WHEN** a device reconnects after missing several notices
- **THEN** it receives no missed notices and refetches what it displays
