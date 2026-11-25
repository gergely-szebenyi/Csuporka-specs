# grocery-list Specification

## Purpose

Defines the household's weekly grocery list generated from the meal plan: which lines it contains, how the per-recipe and in-store views group them, how shared ticks behave when the plan changes, and when a week's list freezes.

## Requirements

### Requirement: Generated from the week's meals
Each week SHALL have one grocery list per household, generated from the meals whose cook day is in that week. Every ingredient row of a meal's recipe becomes one line, with the recipe's stored quantity and unit. Plans have no servings, so amounts are never scaled.

#### Scenario: Recipe amounts used as stored
- **WHEN** a recipe for 4 servings with "tejföl 200 g" is planned this week
- **THEN** this week's list has the line "tejföl 200 g" for that meal

### Requirement: Multi-day meals count once
A meal SHALL add its ingredients once, however many days it lasts, to the list of the week that contains its cook day.

#### Scenario: Three-day meal
- **WHEN** a meal lasts 3 days
- **THEN** its ingredients appear once on the list

#### Scenario: Meal crossing into next week
- **WHEN** a 2-day meal is cooked on this week's Sunday
- **THEN** its ingredients are on this week's list only

### Requirement: Meals without ingredients
A video meal SHALL appear on the list as a video entry without ingredient lines, which links to the video. A recipe with no ingredient rows SHALL add no lines.

#### Scenario: Video meal
- **WHEN** a saved video is planned this week
- **THEN** the list shows a video entry for it and no ingredient lines

### Requirement: Per-recipe view
The per-recipe view ("Receptenként") SHALL group lines by meal, showing each meal's days.

#### Scenario: Two meals
- **WHEN** "Gulyás" (Monday–Tuesday) and "Pörkölt" (Thursday) are planned
- **THEN** the per-recipe view shows two groups with their own lines and days

### Requirement: In-store view
The in-store view ("Boltban (összesítve)") SHALL group all lines of the week by ingredient name, matched case-insensitively and accent-insensitively. Each line SHALL keep its own amount and recipe. Amounts are not added up.

#### Scenario: Same ingredient in two recipes
- **WHEN** "Gulyás" has "tejföl 300 g" and "Pörkölt" has "Tejfol 200 g"
- **THEN** the in-store view shows one group with two lines: 300 g (Gulyás) and 200 g (Pörkölt)

### Requirement: One shared tick per line
Each line SHALL have one tick state, shared by both views and by all members. Ticking sets a target state (ticked or unticked), so repeating it changes nothing.

#### Scenario: Tick seen in both views
- **WHEN** a member ticks a line in the in-store view
- **THEN** the same line is ticked in the per-recipe view for every member

### Requirement: Regeneration keeps unchanged ticks
The list SHALL regenerate when the plan or a planned recipe changes. A line SHALL keep its tick only if its meal and ingredient are the same and its name, quantity and unit are unchanged. A changed line SHALL come back unticked, and lines of removed meals or ingredients SHALL disappear.

#### Scenario: Amount changed
- **WHEN** a ticked line "liszt 250 g" changes to "liszt 300 g" after a recipe edit
- **THEN** the line comes back unticked

#### Scenario: Unrelated edit
- **WHEN** a recipe's steps are edited and its ingredients stay the same
- **THEN** all its ticked lines stay ticked

### Requirement: Viewable weeks
Members SHALL be able to read the list of any past week, this week and next week. A request for a later week SHALL be rejected with `WEEK_NOT_PLANNABLE`.

#### Scenario: Next week
- **WHEN** a member opens next week's list
- **THEN** the list generated from next week's meals is returned

### Requirement: Past weeks freeze
When a week ends (Monday 00:00, Hungarian time), its list SHALL freeze with its lines and ticks. Later changes to recipes SHALL NOT change it. A tick or untick made after the freeze SHALL be rejected with `NO_LONGER_APPLICABLE`.

#### Scenario: Recipe edited after the week
- **WHEN** a recipe planned last week is edited this week
- **THEN** last week's list still shows the old lines

### Requirement: Tick time decides late ticks
A tick SHALL carry the time it was made. A tick made before its week froze SHALL apply even if it arrives later. A tick for a line that no longer exists SHALL be rejected with `NO_LONGER_APPLICABLE`.

#### Scenario: Sunday tick arriving Monday
- **WHEN** a member ticks a line offline on Sunday at 18:00 and it reaches the server on Monday at 08:00
- **THEN** the tick is applied to the frozen list of that week

#### Scenario: Line removed meanwhile
- **WHEN** a tick arrives for a line whose meal was removed in the meantime
- **THEN** the tick is rejected with `NO_LONGER_APPLICABLE`
