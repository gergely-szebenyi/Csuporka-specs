# Spec Delta

## Purpose

Defines the household's shared recipe library: recipes and their ingredients, saved video links, the Favourites/New status, the taste verdict, deletion and its effect on the plan, and importing recipes from websites.

## ADDED Requirements

### Requirement: Recipe content
A recipe SHALL have a name and MAY have servings, ingredient rows, steps, a note and one link. Only the name is required, so a recipe with nothing but a name is valid.

#### Scenario: Name-only recipe
- **WHEN** a member saves a recipe with only the name "Rakott krumpli"
- **THEN** the recipe is saved

#### Scenario: Missing name
- **WHEN** a member saves a recipe with ingredients but a blank name
- **THEN** the save is rejected with `VALIDATION_FAILED` for the name

### Requirement: One recipe link
A recipe SHALL have at most one link, which is either a video or the web page the recipe came from. A video link plays in the app; a web page link opens the source.

#### Scenario: Recipe from a website
- **WHEN** a member saves an imported recipe
- **THEN** its link is the page it was imported from, and it has no other link

#### Scenario: Recipe from a video
- **WHEN** a member makes a recipe from a saved video
- **THEN** its link is the video, and it has no other link

### Requirement: Recipe limits
A recipe SHALL satisfy these limits, or the save is rejected with `VALIDATION_FAILED`:
- name 1–100 characters after trimming
- servings 1–50
- at most 100 ingredient rows
- at most 50 steps of at most 1000 characters each
- note at most 2000 characters
- link http(s), at most 2048 characters

#### Scenario: Too many servings
- **WHEN** a member saves a recipe with 60 servings
- **THEN** the save is rejected with `VALIDATION_FAILED` for servings

#### Scenario: Non-web link
- **WHEN** a member saves a recipe whose link is "ftp://example.com/x"
- **THEN** the save is rejected with `VALIDATION_FAILED` for the link

### Requirement: Ingredient rows
An ingredient row SHALL have a name of 1–100 characters and MAY have a quantity and a unit. A quantity SHALL be greater than 0 with at most 2 decimals. A unit SHALL only be given together with a quantity. A quantity without a unit means a count of the named item.

#### Scenario: Row without amount
- **WHEN** a member adds the ingredient "só" with no quantity and no unit
- **THEN** the row is valid

#### Scenario: Unit without quantity
- **WHEN** a member adds an ingredient with unit "g" and no quantity
- **THEN** the save is rejected with `VALIDATION_FAILED` for that row

#### Scenario: Bare count
- **WHEN** a member adds the ingredient "tojás" with quantity 3 and no unit
- **THEN** the row is valid and reads as 3 eggs

### Requirement: Fixed unit list
Units SHALL come from the fixed list: g, ml, db, kk, tk, ek, csipet, gerezd, fej, szál, csomag, marék. Any other unit SHALL be rejected with `VALIDATION_FAILED`.

#### Scenario: Unit outside the list
- **WHEN** a member saves an ingredient with unit "dkg"
- **THEN** the save is rejected with `VALIDATION_FAILED`

### Requirement: Recipe photo
On creation, a recipe SHALL get a photo key chosen at random from a fixed set that every platform knows. The key SHALL stay the same on later edits. Users SHALL NOT upload photos or choose the key.

#### Scenario: Photo kept on edit
- **WHEN** a member edits the name of a recipe created with photo key K
- **THEN** the recipe still has photo key K

### Requirement: Favourites or New status
Every recipe SHALL be in exactly one of Favourites ("Kedvencek") or New ("Újdonság"). The status is shared by the whole household, chosen when the recipe is created, and any member MAY change it at any time.

#### Scenario: Saving as Favourite
- **WHEN** a member creates a recipe and chooses Favourites
- **THEN** every member sees it in Favourites and not in New

#### Scenario: Moving by hand
- **WHEN** a member moves a recipe from New to Favourites
- **THEN** it is in Favourites for the whole household

### Requirement: Saved videos
A saved video SHALL have an http(s) link of at most 2048 characters, a title (at most 100 characters, may be empty), a thumbnail and the source domain. Saved videos have no Favourites status; they are listed in New › Saved videos ("Mentett videók").

#### Scenario: Video without preview
- **WHEN** a member saves a video link for which no title or thumbnail was found
- **THEN** the video is saved with an empty title and no thumbnail

### Requirement: Renaming and deleting videos
Any member MAY rename or delete a saved video. Deleting a video SHALL NOT delete recipes that were made from it or that link to it.

#### Scenario: Deleting a video with a recipe made from it
- **WHEN** a member deletes a saved video that a recipe links to
- **THEN** the video is gone and the recipe keeps its link

### Requirement: Making a recipe from a video by hand
A member MAY create a recipe from a saved video. The recipe starts with the video's link as its link, and the saved video SHALL stay in Saved videos.

#### Scenario: Video stays after manual recipe
- **WHEN** a member makes a recipe from a saved video via its menu
- **THEN** both the new recipe and the saved video exist

### Requirement: Taste verdict is due
A recipe or saved video SHALL be waiting for a taste verdict ("Ízlett?") when it is in New, one of its meals was marked cooked on a day before today, and no member has answered yet. Moving a recipe to Favourites by hand SHALL settle it.

#### Scenario: Due the day after cooking
- **WHEN** a recipe in New was marked cooked yesterday and nobody has answered
- **THEN** it is waiting for a verdict today

#### Scenario: Not due on the cooking day
- **WHEN** a recipe in New is marked cooked today
- **THEN** it is not waiting for a verdict until tomorrow

#### Scenario: Never cooked
- **WHEN** a recipe in New is planned but its meals were never marked cooked
- **THEN** it is not waiting for a verdict

### Requirement: First verdict decides for everyone
The first answer from any member SHALL settle the verdict for the whole household. A later answer SHALL be rejected with `VERDICT_ALREADY_GIVEN`. An answer for an item that isn't waiting for a verdict SHALL be rejected with `VERDICT_NOT_DUE`.

#### Scenario: Two members answer
- **WHEN** one member answers "Ízlett" and another then answers "Nem ízlett" for the same recipe
- **THEN** the recipe moves to Favourites, and the second answer is rejected with `VERDICT_ALREADY_GIVEN`

### Requirement: Liked recipe
Answering "Ízlett" for a recipe SHALL move it to Favourites.

#### Scenario: Liked
- **WHEN** a member answers "Ízlett" for a recipe in New
- **THEN** the recipe is in Favourites for the whole household

### Requirement: Liked video becomes a recipe
Answering "Ízlett" for a saved video SHALL happen by saving a new recipe made from it, which starts with the video's title as its name and its link as the recipe link. The recipe SHALL be saved in Favourites, and the saved video SHALL leave Saved videos.

#### Scenario: Liked video
- **WHEN** a member answers "Ízlett" for a saved video and saves the recipe made from it
- **THEN** the household has the new recipe in Favourites with the video as its link, and the video is no longer in Saved videos

### Requirement: Disliked item is deleted with a short undo
Answering "Nem ízlett" SHALL delete the recipe or saved video for the whole household. Within 10 seconds of the answer, any member MAY undo it, which leaves the library and plan exactly as before the answer, with the verdict still due. After 10 seconds, an undo SHALL be rejected with `UNDO_WINDOW_EXPIRED`.

#### Scenario: Undo in time
- **WHEN** a member answers "Nem ízlett" and undoes it 4 seconds later
- **THEN** the recipe is back in New with its planned days and is still waiting for a verdict

#### Scenario: Undo too late
- **WHEN** a member tries to undo a "Nem ízlett" answer 15 seconds later
- **THEN** the undo is rejected with `UNDO_WINDOW_EXPIRED` and the recipe stays deleted

### Requirement: Deleting a recipe or video
Any member MAY delete a recipe or saved video. It SHALL disappear from the library for the whole household. A deleted item SHALL keep its name for past plan days that refer to it.

#### Scenario: Delete a recipe
- **WHEN** a member deletes a recipe
- **THEN** no member sees it in the library any more

### Requirement: Deletion empties today and later
When a planned recipe or video is deleted (directly, or by a final "Nem ízlett"), every planned day from today onward that used it SHALL become unplanned, including any cooked mark on today. Days before today SHALL keep their entry, shown as a deleted recipe ("Törölt recept").

#### Scenario: Meal running across today
- **WHEN** a recipe is planned yesterday for 3 days and is deleted today
- **THEN** today and tomorrow become unplanned, and yesterday shows "Törölt recept"

#### Scenario: Cooked today
- **WHEN** a recipe planned and marked cooked today is deleted
- **THEN** today becomes unplanned and the cooked mark is gone

### Requirement: Concurrent recipe edits
When two members save changes to the same recipe or video, the later save SHALL replace the earlier one (last write wins).

#### Scenario: Two edits
- **WHEN** member A saves a recipe and member B saves an older copy of it a moment later
- **THEN** the recipe holds B's content

### Requirement: Website import returns a draft
A member SHALL be able to import a recipe from a web page link. Import SHALL return an unsaved draft (name, servings, ingredient rows, steps and the page as its link, no photo) that the member reviews and saves like any recipe. Import itself SHALL NOT save anything.

#### Scenario: Successful import
- **WHEN** a member imports a page that contains recipe structured data
- **THEN** a draft is returned and the library is unchanged until the member saves it

### Requirement: Imported units are converted
Imported ingredient rows SHALL be split into quantity, unit and name. Units SHALL be converted into the fixed unit list (e.g. 2 dkg → 20 g, 0.5 l → 500 ml), so a draft never contains a unit outside it.

#### Scenario: Converting decagrams
- **WHEN** an imported page lists "25 dkg liszt"
- **THEN** the draft has the row quantity 250, unit "g", name "liszt"

### Requirement: Import failure
When the page can't be fetched or contains no recipe data, import SHALL be rejected with `NO_RECIPE_FOUND`. A link that isn't a valid http(s) URL SHALL be rejected with `VALIDATION_FAILED`.

#### Scenario: Page without recipe
- **WHEN** a member imports a news article with no recipe data
- **THEN** import is rejected with `NO_RECIPE_FOUND`

### Requirement: Video link preview
For any video link, a member SHALL be able to request a preview with title, thumbnail and source domain, on a best-effort basis. A preview that finds nothing SHALL return empty fields, not an error.

#### Scenario: Unknown site
- **WHEN** a member requests a preview for a link from a site without video metadata
- **THEN** an empty preview is returned without an error
