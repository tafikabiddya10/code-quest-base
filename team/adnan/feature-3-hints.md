> âš ï¸ **STATUS â€” PENDING / NOT BUILT.** This feature was removed from the codebase
> during the backbone strip-down and is **pending implementation in Sprint 2**.
> The document below is kept as a historical blueprint for the rebuild; none of
> the files or routes it describes exist in the current codebase.

# Adnan â€” Feature 3: Three-Level Hint System

**Sprint:** 2  
**Feature:** F3 â€” Three-Level Hint System

**Status:** âœ… **COMPLETE** (implemented November 12, 2026)

---

## 1. What the feature does

Every monster (Obstacle) can carry three progressive hints:

| # | Name | Purpose | Example point cost |
|---|---|---|---|
| 1 | **Nudge** | A small push in the right direction | 2 |
| 2 | **Concept Reminder** | Explains the architectural idea behind the question | 3 |
| 3 | **Near-Solution** | Almost the answer | 5 |

Rules enforced by the server:

1. Hints are revealed **strictly in order**: 1 â†’ 2 â†’ 3.
2. A revealed hint is recorded once in `hint_usages` and its cost is charged **once**.
3. The cost is deducted through the shared `ScoreService.calculateLevelScore(...)`.
4. A student can only reveal hints of a **published** castle whose teacher teaches a class the student has joined.
5. Teachers can only edit hints on castles they own.
6. A hint that a student has already used cannot be deleted (it would corrupt their score history).

Locked hint text is never sent to the browser â€” the `HintView` object contains `text = null`
until the student reveals it (same principle as `Assessment.answerKey`).

---

## 2. Files

### New (all owned by F3)

| File | Purpose |
|---|---|
| `src/main/java/com/codequest/hints/HintService.java` | All business rules: save hints, reveal in order, charge once, recalculate score. |
| `src/main/java/com/codequest/hints/HintController.java` | Teacher save endpoint + student hint library/chamber/reveal endpoints. |
| `src/main/java/com/codequest/hints/form/HintForm.java` | Form backing object for the three text/cost pairs. |
| `src/main/java/com/codequest/hints/view/HintView.java` | Safe read model for the student panel (locked text is null). |
| `src/main/resources/templates/hints/fragments.html` | `hintEditor` (teacher) and `hintPanel` (student) fragments. |
| `src/main/resources/templates/hints/index.html` | "Hint Library": published castles the student can play. |
| `src/main/resources/templates/hints/chamber.html` | F3 verification page: hints for every floor of one castle. |
| `src/test/java/com/codequest/hints/HintFeatureTests.java` | 13 tests: editing, ordering, single charge, score, security, rendering. |

### Existing files touched (small, clearly marked)

| File | Change |
|---|---|
| `src/main/java/com/codequest/repository/HintUsageRepository.java` | Added `existsByHint(Hint)` so a used hint cannot be deleted. |
| `src/main/java/com/codequest/studio/StudioController.java` | Injects `HintService`; adds `hintForm` to the floor-design model (3 lines). |
| `src/main/resources/templates/studio/floor-design.html` | Replaced the placeholder with `hintEditor(${hintForm}, ${obstacle})`. |
| `src/main/resources/templates/profile/play.html` | Left the `hintPanel` include with a comment telling F5 which model attributes to pass. |
| `src/main/resources/templates/student/home.html` | F3 card now links to `/play/hints`. |
| `src/main/resources/static/css/app.css` | Added `.cq-hint-*` styles. |
| `HANDOVER.md` | Marked F3 complete and documented the routes. |

---

## 3. Routes

| Method | Path | Who | What |
|---|---|---|---|
| POST | `/studio/obstacles/{obstacleId}/hints` | TEACHER (owner) | Create/update/delete the 3 hints of a monster. |
| GET | `/play/hints` | STUDENT | List published castles from the student's teachers. |
| GET | `/play/{levelId}/hints` | STUDENT (enrolled) | Hint chamber: every floor + its hint panel. |
| POST | `/play/{levelId}/hints/{hintId}/reveal` | STUDENT (enrolled) | Reveal the next hint, record usage, recalculate score. |

`/play/**` is already protected by `SecurityConfig` (`ROLE_STUDENT`), and `/studio/**`
by `ROLE_TEACHER`; no security config change was needed.

---

## 4. How to run and verify (checklist)

```bash
cd /home/adnan/Desktop/Project
mvn test
mvn spring-boot:run -Dspring-boot.run.profiles=test
```

Open <http://localhost:8080>.

### A. Teacher edits hints

1. Log in as `teacher@codequest.com` / `password123`.
2. Creator Studio â†’ **Tower of Spring MVC** â†’ **Design Castle** â†’ **Design Floor** on Floor 1.
3. Scroll to **ðŸ’¡ 3-Level Hints**. The seeded hints are pre-filled.
4. Change the text/cost of Hint 1 (e.g. cost `1`), click **Save Hints**.
5. A green "Hints saved..." banner appears.
6. Try leaving Hint 1 empty while Hint 2 is filled â†’ red error "Hints must be filled in order".
7. Try a negative cost â†’ red error "cost must be between 0 and 100 points".

### B. Student reveals hints in order

1. Log out, log in as `student@codequest.com` / `password123`.
2. Dashboard â†’ **ðŸ’¡ 3-Level Hints** card â†’ **Open Hint Library**.
3. Open **Tower of Spring MVC** â†’ **Hint Chamber**.
4. Hint 1 shows **Reveal Hint 1 (-2 pts)**; Hints 2 and 3 show ðŸ”’ Locked.
5. Reveal Hint 1 â†’ its text appears, Hint 2 becomes unlocked.
6. Reload the page â†’ Hint 1 stays revealed and is **not charged again**.
7. Reveal Hint 2, then Hint 3.
8. Try opening `/play/{levelId}/hints/{hint3Id}/reveal` directly for a fresh student â†’ server rejects it with "Hints must be revealed in order".

### C. Automated checks

`mvn test` runs 13 F3 tests, including:

- teacher can edit / another teacher cannot,
- no gaps (Hint 2 without Hint 1),
- used hint cannot be deleted,
- strict reveal order,
- reveal is idempotent (charged once),
- score = earned points âˆ’ hint costs (10 âˆ’ 2 = 8),
- outsider student is rejected,
- teacher editor, hint library and hint chamber pages render,
- reveal and save endpoints redirect correctly.

---

## 5. How F5 (Tanvir) plugs the panel into the play page

Add these to the model in the play controller:

```java
model.addAttribute("levelId", level.getId());
model.addAttribute("obstacle", currentObstacle);
model.addAttribute("hintViews", hintService.getHintViews(student, currentObstacle));
model.addAttribute("returnUrl", "/play/" + level.getId());
```

Then include the fragment in `profile/play.html`:

```html
<div th:replace="~{hints/fragments :: hintPanel(${levelId}, ${obstacle}, ${hintViews}, ${returnUrl})}"></div>
```

`HintService` is a normal Spring bean, so F5 can inject it without changing F3 code.

---

## 6. What you just learned

- **Service layer owns rules:** the controller only translates HTTP, the service decides what is allowed.
- **Never trust the browser:** locked hint text is null on the server, not hidden with CSS.
- **One source of truth for scoring:** `ScoreService` is reused by hints and play, so points can never disagree.
- **Database constraints are your friend:** `uk_student_hint` prevents a double charge even if two requests race.
- **Feature slices can share fragments:** `hintPanel` is written once and used by the F3 chamber now and the F5 play page later.
