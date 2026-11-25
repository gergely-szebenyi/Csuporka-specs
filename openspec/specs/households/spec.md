# households Specification

## Purpose

Defines the household as the owner of all shared content: how users always belong to exactly one household, how it is named, and how others join it by QR invite and merge their content in.

## Requirements

### Requirement: Every user has exactly one household
A user SHALL belong to exactly one household at all times. Registration SHALL create a household with the new user as its only member, and that household SHALL be usable by the first request after registration succeeds.

#### Scenario: Household right after registration
- **WHEN** a user registers and immediately requests their household's recipes
- **THEN** the request succeeds with an empty list instead of failing because no household exists yet

### Requirement: Household owns shared content
Recipes, saved videos, meal plans and grocery lists SHALL belong to a household, not to a single user. Every member SHALL see and change the same content.

#### Scenario: Member sees another member's recipe
- **WHEN** one member saves a recipe
- **THEN** every other member of the household sees it in the shared recipe library

### Requirement: Household isolation
A user SHALL only be able to read or change content of their own household. A request for content of another household SHALL be rejected with `NOT_FOUND`, without revealing that the content exists.

#### Scenario: Foreign recipe
- **WHEN** a user requests a recipe that belongs to another household
- **THEN** the request is rejected with `NOT_FOUND`

### Requirement: Members are equal
All members of a household SHALL have the same rights. There SHALL be no owner or admin role. Any member may rename the household, create invites, and change any shared content.

#### Scenario: Any member creates an invite
- **WHEN** a member who did not create the household creates an invite
- **THEN** the invite is created like any other

### Requirement: Household name
A new household SHALL be unnamed. Any member MAY name or rename it at any time ("Létrehozás" names it the first time). A name SHALL be 1–50 characters after trimming; otherwise the request is rejected with `VALIDATION_FAILED`.

#### Scenario: Naming a solo household
- **WHEN** a solo user names their unnamed household "Kovácsék"
- **THEN** the household is named "Kovácsék"

#### Scenario: Name too long
- **WHEN** a member renames the household to a 51-character name
- **THEN** the request is rejected with `VALIDATION_FAILED`

### Requirement: Member list
Members SHALL be able to list their household's members by display name. Other members' email addresses SHALL NOT be exposed.

#### Scenario: Listing members
- **WHEN** a member of a two-person household lists members
- **THEN** both display names are returned, without email addresses

### Requirement: Invites
Any member SHALL be able to create an invite for their household, shared as a QR code. An invite SHALL be valid for 24 hours and MAY be used by several people within that time. Creating a new invite SHALL revoke the household's previous one.

#### Scenario: Reusable invite
- **WHEN** two different users join with the same invite within 24 hours
- **THEN** both joins succeed

#### Scenario: New invite revokes the old one
- **WHEN** a member creates an invite and later creates another
- **THEN** joining with the first invite is rejected with `INVITE_INVALID`

### Requirement: Invalid invites
Joining with an invite that is unknown, expired (older than 24 hours) or revoked SHALL be rejected with `INVITE_INVALID`.

#### Scenario: Expired invite
- **WHEN** a user joins with an invite created 25 hours ago
- **THEN** the join is rejected with `INVITE_INVALID`

### Requirement: Only a solo user can join
A user SHALL only join another household when they are the only member of their current household. Otherwise the join SHALL be rejected with `HOUSEHOLD_NOT_SOLO`. Joining with an invite of one's own household SHALL be rejected with `ALREADY_MEMBER`.

#### Scenario: Member of a shared household scans an invite
- **WHEN** a user whose household has another member joins with a valid invite of a different household
- **THEN** the join is rejected with `HOUSEHOLD_NOT_SOLO` and nothing changes

#### Scenario: Own household's invite
- **WHEN** a member joins with their own household's invite
- **THEN** the join is rejected with `ALREADY_MEMBER`

### Requirement: Join merges recipes and videos
When a user joins, all recipes and saved videos of their old household SHALL move into the joined household unchanged. There is no deduplication by name or link, and each item keeps its Favourites/New status and any pending taste verdict.

#### Scenario: Duplicate recipe names
- **WHEN** a user with a recipe "Gulyás" joins a household that also has "Gulyás"
- **THEN** the household has both recipes afterwards, each with its own status

### Requirement: Join replaces the joiner's plan
When a user joins, the joiner's old meal plans and grocery lists SHALL be discarded. The joined household's plans and grocery lists SHALL apply to the joiner from then on.

#### Scenario: Joiner had planned meals
- **WHEN** a user with meals planned for next week joins a household
- **THEN** the user sees the household's plan for next week, and their own earlier meals are gone

### Requirement: Old household ceases to exist
After a join, the joiner's old household SHALL no longer exist. Any invite it had SHALL stop working.

#### Scenario: Invite of the old household
- **WHEN** a user joins another household and someone then uses an invite of the user's old household
- **THEN** the join is rejected with `INVITE_INVALID`

### Requirement: No leaving and no account deletion in v1
A user SHALL NOT be able to leave a household or delete their account in v1. Membership changes only by joining another household while solo.

#### Scenario: No leave operation
- **WHEN** a member of a shared household looks for a way to leave it
- **THEN** no operation exists to leave the household or to delete the account
