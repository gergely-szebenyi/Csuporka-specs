# meal-plan Specification

## Purpose

Defines the household's weekly lunch plan: the calendar it uses, which days can be planned, meals that last one to three days, the cooked mark, and the day statuses and week coverage that every platform derives from the plan.

## Requirements

### Requirement: Calendar
All calendar rules SHALL use Hungarian time (Europe/Budapest), including when "today" starts and ends. A week SHALL run from Monday to Sunday and be identified by the date of its Monday.

#### Scenario: Week identity
- **WHEN** the plan for Wednesday 2026-11-25 is requested
- **THEN** it belongs to the week identified as 2026-11-23

#### Scenario: Day boundary in Hungarian time
- **WHEN** it is 23:30 UTC on 2026-11-25, which is 00:30 on 2026-11-26 in Budapest
- **THEN** "today" is 2026-11-26

### Requirement: One lunch per day
The plan SHALL cover one lunch per day. Each day SHALL be either unplanned, or belong to exactly one meal as its cook day or as one of its leftover days.

#### Scenario: Day belongs to one meal
- **WHEN** a day is a leftover day of one meal
- **THEN** it can't also be the cook day of another meal

### Requirement: Meal
A meal SHALL be a recipe or a saved video with a cook day and a length of 1–3 consecutive days. The days after the cook day are leftover days. Meals have no servings. A length outside 1–3 SHALL be rejected with `VALIDATION_FAILED`.

#### Scenario: Three-day meal
- **WHEN** a member plans "Pörkölt" on Monday for 3 days
- **THEN** Monday is the cook day and Tuesday and Wednesday are leftover days

#### Scenario: Too long
- **WHEN** a member plans a meal for 4 days
- **THEN** the request is rejected with `VALIDATION_FAILED`

### Requirement: Viewable weeks
Members SHALL be able to read the plan of any past week, this week and next week. A request for a week after next week SHALL be rejected with `WEEK_NOT_PLANNABLE`.

#### Scenario: Old week
- **WHEN** a member opens a week from three months ago
- **THEN** its plan is returned read-only

### Requirement: Plannable window
Every day of a new or changed meal SHALL lie from today up to and including next week's Sunday. A meal MAY run from this week into next week. Any day before today or after next Sunday SHALL cause rejection with `WEEK_NOT_PLANNABLE`.

#### Scenario: Crossing into next week
- **WHEN** a member plans a 2-day meal on this week's Sunday
- **THEN** the meal is saved with its leftover day on next week's Monday

#### Scenario: Past next Sunday
- **WHEN** a member plans a 2-day meal on next week's Sunday
- **THEN** the request is rejected with `WEEK_NOT_PLANNABLE`

#### Scenario: Yesterday
- **WHEN** a member plans a meal with yesterday as its cook day
- **THEN** the request is rejected with `WEEK_NOT_PLANNABLE`

#### Scenario: Today
- **WHEN** a member plans a meal with today as its cook day
- **THEN** the meal is saved

### Requirement: Started meals are locked
A meal whose cook day is before today SHALL be locked. Changing its length or recipe, removing it, or replacing it through overlap SHALL be rejected with `MEAL_LOCKED`, even if some of its leftover days are today or later.

#### Scenario: Removing yesterday's meal
- **WHEN** a member removes a meal cooked yesterday whose leftover day is today
- **THEN** the request is rejected with `MEAL_LOCKED`

#### Scenario: Planning over a locked leftover day
- **WHEN** today is a leftover day of a meal cooked yesterday and a member plans a new meal for today
- **THEN** the request is rejected with `MEAL_LOCKED`

### Requirement: Overlap replaces whole meals
When a new or lengthened meal covers any day of other meals, those meals SHALL be removed completely, including their cooked marks and all their days, and the new meal SHALL take its days. Clients warn before sending such a change.

#### Scenario: Overlapping a leftover day
- **WHEN** a 3-day meal starts Monday and a member plans a new meal on Wednesday
- **THEN** the Monday meal is removed with all its days, and Wednesday holds the new meal

### Requirement: Removing a meal
Removing a meal SHALL remove its cook day and all its leftover days, and its cooked mark.

#### Scenario: Remove a two-day meal
- **WHEN** a member removes a meal planned on Tuesday for 2 days
- **THEN** Tuesday and Wednesday become unplanned

### Requirement: Changing a meal's length
Any member MAY change the length of an unlocked meal to 1–3 days within the plannable window. The cooked mark SHALL be kept. Days added by lengthening follow the overlap rule.

#### Scenario: Shortening keeps the mark
- **WHEN** a member shortens a cooked 3-day meal starting today to 1 day
- **THEN** the meal keeps its cooked mark and the two leftover days become unplanned

### Requirement: Swapping a meal's recipe
Any member MAY replace the recipe or video of an unlocked meal. The cook day and length SHALL stay, and any cooked mark SHALL be removed.

#### Scenario: Swap a cooked meal
- **WHEN** a member swaps the recipe of today's meal that was already marked cooked
- **THEN** the meal shows the new recipe and is no longer marked cooked

### Requirement: Cooked mark
Any member SHALL be able to mark a meal as cooked ("Megfőztem") or unmark it. The mark records who marked it and when. Mark and unmark set a target state, so repeating one changes nothing.

#### Scenario: Marking twice
- **WHEN** a member marks today's meal as cooked, and the same request arrives again
- **THEN** the meal stays marked once, with the first marking's member and time

### Requirement: Cooked mark only on the cook day
Marking or unmarking SHALL be accepted only when the time the action was taken falls on the meal's cook day, judged in Hungarian time. Otherwise, or if the meal no longer exists, it SHALL be rejected with `NO_LONGER_APPLICABLE`.

#### Scenario: Offline mark sent after midnight
- **WHEN** a member marks today's meal at 13:00 offline and the mark reaches the server at 00:20 the next day
- **THEN** the mark is accepted, because it was made on the cook day

#### Scenario: Unmarking the next day
- **WHEN** a member tries to unmark yesterday's cooked meal today
- **THEN** the request is rejected with `NO_LONGER_APPLICABLE`

### Requirement: Deleted recipe on past days
A past plan day whose recipe or video was deleted SHALL still show its meal, marked as a deleted recipe ("Törölt recept") with the item's name kept.

#### Scenario: Viewing an old week
- **WHEN** a member views a past week that included a since-deleted recipe
- **THEN** that day shows "Törölt recept"

### Requirement: Day status
Every platform SHALL derive a day's status from the plan:
- **to cook**: cook day, not marked cooked
- **done**: cook day, marked cooked
- **leftover**: leftover day, referring to its meal and to which day of the meal it is
- **nothing**: no meal

#### Scenario: Leftover from two days ago
- **WHEN** a 3-day meal was cooked on Monday and the status of Wednesday is derived
- **THEN** Wednesday is leftover, day 3 of the Monday meal

#### Scenario: Cooked today
- **WHEN** today is a cook day and a member marked it cooked
- **THEN** today's status is done

### Requirement: Fully planned week
A week SHALL count as fully planned when every day from Monday to Sunday is a cook day or a leftover day. That includes leftover days of a meal that started the week before.

#### Scenario: Monday covered by last week's meal
- **WHEN** a 2-day meal starts on this week's Sunday and next week's Tuesday to Sunday are planned
- **THEN** next week counts as fully planned

#### Scenario: One gap
- **WHEN** next week has every day planned except Thursday
- **THEN** next week does not count as fully planned

### Requirement: Concurrent plan edits
When two members change the same plan days, the change that reaches the server later SHALL win (last write wins).

#### Scenario: Two members plan Friday
- **WHEN** member A plans "Gulyás" on Friday and member B plans "Lecsó" on Friday a moment later
- **THEN** Friday holds "Lecsó"
