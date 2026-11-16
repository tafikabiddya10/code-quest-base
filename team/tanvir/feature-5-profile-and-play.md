# Tanvir — Feature 5: Student Profile, Progress & Castle Play

**Sprint:** 2  
**Feature:** F5 — Student Profile, Learning Progress & Student Play Page

**Assigned Feature:**
- **F5: Student Profile & Learning Progress + Student Play Page**

**Status:** ⏳ **Pending (Sprint 2)**

---

## 1. Feature 5 Overview
Powers the core student gameplay loop:

1. **My Castles (`/student/levels`):**
   - Lists every published castle made by a teacher of a class the student joined.
   - Shows completion percentage, current floor, score, hints used and a status badge.
   - Links straight into the castle climb.

2. **Castle Play (`/play/{levelId}`):**
   - Meet the initial NPC and read the introductory story.
   - Climb a floor ladder (Tower sidebar) where each floor is CLEARED, CURRENT or LOCKED.
   - Fight the current floor's monster: answer its assessments and submit.
   - Every answer try is stored as an `Attempt`, correct or not.
   - A floor clears only when every assessment on it has a correct attempt; the next floor unlocks.
   - Embeds the F3 hint panel and the F4 discussion section for the current monster/castle.

3. **Summit (`/play/{levelId}` once completed):**
   - Final score, hints used and 100% completion.
   - The full `level.finalCode` revealed line by line.
   - The same F4 discussion section.

4. **Profile (`/student/profile`):**
   - Overall completion percentage (completed castles / available castles).
   - Login streak, total score, and a per-castle progress table.

---

## 2. File Ownership Map — *(Pending implementation)*

- **Controllers & Services:**
  - `com.codequest.profile.PlayController`
  - `com.codequest.profile.PlayService`
  - `com.codequest.profile.ProfileController`
  - `com.codequest.profile.ProfileService`
- **View models:**
  - `com.codequest.profile.view.AssessmentView`
  - `com.codequest.profile.view.FloorView`
  - `com.codequest.profile.view.PlayState`
  - `com.codequest.profile.view.SubmissionResult`
  - `com.codequest.profile.view.LevelRow`
  - `com.codequest.profile.view.ProfileSummary`
- **Thymeleaf Templates:**
  - `src/main/resources/templates/profile/play.html`
  - `src/main/resources/templates/profile/top.html`
  - `src/main/resources/templates/profile/profile.html`
  - `src/main/resources/templates/student/levels.html`
  - `src/main/resources/templates/assessment/mcq-play.html`
- **CSS:** `Feature 5` section at the bottom of
  `src/main/resources/static/css/app.css`.
- **Tests:** `src/test/java/com/codequest/profile/PlayFeatureTests.java`

---

## 3. Routes

| Method | Route | Purpose |
|---|---|---|
| GET | `/student/levels` | My Castles list with progress |
| GET | `/student/profile` | Profile stats and per-castle table |
| GET | `/play/{levelId}` | Castle climb, or summit page when completed |
| POST | `/play/{levelId}/assessments/{assessmentId}/submit` | Submit one answer |

All routes will require `ROLE_STUDENT` (to be covered by `SecurityConfig`).
The submit form uses `th:action`, so Spring Security's CSRF token is injected automatically.

---

## 4. How the Climb Works

### Starting a castle
`PlayService.getOrCreateProgress(student, level)` creates the `LevelProgress` row on the
first visit. Creating that row also triggers F1's floor-edit safety rule: the teacher can
no longer add/delete/reorder floors once a student has started.

### Submitting an answer
1. `PlayController` turns the MCQ `selectedIndex` form field into
   `{"selectedIndex": n}` (future types may post a ready-made `answerJson` hidden field).
2. `PlayService.submitAttempt` checks:
   - the student can access the castle (published + joined classroom),
   - the castle is not already completed,
   - the assessment belongs to the current floor,
   - the assessment has not already been solved.
3. `AssessmentService.check(...)` evaluates the answer via the plug-in handler.
4. The try is saved as an `Attempt` with `saveAndFlush` so the "is this floor solved?"
   query sees it immediately.
5. If the floor is now fully solved, `advanceIfNeeded` moves to the next floor; clearing
   the final floor sets `completed = true` and `completedAt`.
6. The score is recalculated by the shared `ScoreService` (correct points minus hint costs).

Wrong answers never advance the floor, so the student can retry as many times as needed.
All attempts remain in the append-only `attempts` table.

### Reload safety
`advanceIfNeeded` runs on every page load and every submission, so a reload after a
cleared floor still lands the student on the correct current floor. If every floor is
already solved, the castle is marked complete and the summit page is shown.

---

## 5. Assessment Plug-in Integration

The play page does not hard-code MCQ. For each `AssessmentView` it renders:

```html
<div th:replace="~{assessment/__${a.typeKey}__-play :: __${a.typeKey}__Play(${a})}"></div>
```

- `a.typeKey` is the lowercased `Assessment.type` (e.g. `MCQ` -> `mcq`).
- A new type only needs `templates/assessment/<type>-play.html` with a fragment named
  `<type>Play(assessment)`; no controller or service change is required.
- `AssessmentView` parses `content` JSON once and exposes `getOptions()` for MCQ.
  The answer key is never placed in the view model.
- MCQ radio IDs are prefixed with the assessment id, so several questions can share a page.

---

## 6. Profile Calculation

- **Castle completion:** `completed castles / castles available to the student`.
- **Per-castle completion (in progress):** `(currentFloor - 1) / totalFloors`.
  `currentFloor` is the floor being played, so every lower floor is already cleared.
- **Completed castle:** 100%.
- **Total score:** sum of each `LevelProgress.score`; each score already includes hint
  deductions from `ScoreService`.
- **Streak:** read from `User.loginStreak`, which `LoginSuccessHandler` updates on login.

---

## 7. Manual Verification Checklist

Run the app with the H2 test profile:

```bash
mvn spring-boot:run -Dspring-boot.run.profiles=test
```

1. Log in as `student@codequest.com` / `password123`.
2. Dashboard -> **Browse Castles** -> "Tower of Spring MVC" -> **Enter Castle**.
3. On Floor 1, submit a wrong option: an error appears, the floor stays current, and the
   student can retry.
4. Submit the correct option (option 2, "Controller"): the floor is marked cleared and
   Floor 2 becomes current.
5. Open the Hint panel, reveal Hint 1, then solve Floor 1 and confirm the score is
   reduced by the hint cost.
6. Clear Floors 2 and 3 (answers: option 3 "Service", option 1 "Spring Data JpaRepository").
7. The summit page appears with the full Spring MVC code.
8. Go to **Profile**: completion is 100%, the streak is shown, and the castle row is
   marked Completed.
9. Log in as `teacher@codequest.com` / `password123` and confirm the student appears in
   classroom/roster views.

---
