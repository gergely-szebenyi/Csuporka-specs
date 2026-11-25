# offline-replay Specification

## Purpose

Defines which changes a client may make while offline and send later, and how the server treats such delayed writes, so queued actions replay safely and predictably.

## Requirements

### Requirement: Replayable writes
Only these writes SHALL be made offline and sent later: ticking or unticking a grocery line, and marking or unmarking a meal as cooked. Every other change, including the taste verdict, SHALL require a connection.

#### Scenario: Verdict needs a connection
- **WHEN** a member wants to answer "Ízlett?" while offline
- **THEN** the answer can't be queued for later

### Requirement: Target-state writes
A replayable write SHALL state the desired end state (ticked/unticked, cooked/not cooked), never a toggle. Sending the same write more than once SHALL leave the same result as sending it once.

#### Scenario: Duplicate replay
- **WHEN** a queued "ticked" write is sent twice after reconnecting
- **THEN** the line is ticked, the same as after one send

### Requirement: Action time
A replayable write SHALL carry the time the action was taken. Time-bound rules (the cooked mark's cook day, a grocery list's freeze) SHALL be judged by that time, not by when the write arrives.

#### Scenario: Mark judged by action time
- **WHEN** a cooked mark made on the cook day arrives the next day
- **THEN** it is accepted

### Requirement: Arrival order wins
Replayable writes to the same line or meal SHALL apply in the order they reach the server. A delayed offline write MAY overwrite a newer online one.

#### Scenario: Offline untick after online tick
- **WHEN** member A ticks a line online, then member B's earlier offline untick of the same line arrives
- **THEN** the line ends up unticked

### Requirement: Writes that no longer apply are dropped
A replayable write whose target no longer exists, or whose action time breaks a time-bound rule, SHALL be rejected with `NO_LONGER_APPLICABLE`. The client SHALL discard it without telling the user, and show the refreshed state.

#### Scenario: Meal removed while offline
- **WHEN** a member marks a meal cooked offline and another member removed that meal meanwhile
- **THEN** the mark is rejected with `NO_LONGER_APPLICABLE` and the client drops it silently
