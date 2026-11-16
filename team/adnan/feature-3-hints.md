# Adnan — Feature 3: Three-Level Hint System

**Sprint:** 2  
**Feature:** F3 — Three-Level Hint System

**Status:** ⏳ **Pending (Sprint 2)**

---

## 1. What the feature does

Every monster (Obstacle) can carry three progressive hints:

| # | Name | Purpose | Example point cost |
|---|---|---|---|
| 1 | **Nudge** | A small push in the right direction | 2 |
| 2 | **Concept Reminder** | Explains the architectural idea behind the question | 3 |
| 3 | **Near-Solution** | Almost the answer | 5 |

Rules enforced by the server:

1. Hints are revealed **strictly in order**: 1 → 2 → 3.
2. A revealed hint is recorded once in `hint_usages` and its cost is charged **once**.
3. The cost is deducted through the shared `ScoreService.calculateLevelScore(...)`.
4. A student can only reveal hints of a **published** castle whose teacher teaches a class the student has joined.
5. Teachers can only edit hints on castles they own.
6. A hint that a student has already used cannot be deleted (it would corrupt their score history).

Locked hint text is never sent to the browser — the `HintView` object contains `text = null`
until the student reveals it (same principle as `Assessment.answerKey`).

---

## 2. Files

### New (all owned by F3) — *(Pending implementation)*

| File | Purpose |
|---|---|
| `src/main/java/com/codequest/hints/HintService.java` | All business rules: save hints, reveal in order, charge once, recalculate score. |
| `src/main/java/com/codequest/hints/HintController.java` | Teacher save endpoint + student hint library/chamber/reveal endpoints. |
| `src/main/java/com/codequest/hints/form/HintForm.java` | Form backing object for the three text/cost pairs. |
| `src/main/java/com/codequest/hints/view/HintView.java` | Safe read model for the student panel (locked text is null). |
| `src/main/resources/templates/hints/fragments.html` | `hintEditor` (teacher) and `hintPanel` (student) fragments. |
| `src/main/resources/templates/hints/index.html` | "Hint Library": published castles the student can play. |
| `src/main/resources/templates/hints/chamber.html` | F3 verification page: hints for every floor of one castle. |
| `src/test/java/com/codequest/hints/HintFeatureTests.java` | Planned: 13 tests covering editing, ordering, single charge, score, security, rendering. |

### Existing files to touch (small, clearly marked)

| File | Change |
|---|---|
| `src/main/java/com/codequest/repository/HintUsageRepository.java` | Add `existsByHint(Hint)` so a used hint cannot be deleted. |
| `src/main/java/com/codequest/studio/StudioController.java` | Inject `HintService`; add `hintForm` to the floor-design model (3 lines). |
| `src/main/resources/templates/studio/floor-design.html` | Replace the placeholder with `hintEditor(${hintForm}, ${obstacle})`. |
| `src/main/resources/templates/profile/play.html` | Leave the `hintPanel` include with a comment telling F5 which model attributes to pass. |
| `src/main/resources/templates/student/home.html` | F3 card links to `/play/hints`. |
| `src/main/resources/static/css/app.css` | Add `.cq-hint-*` styles. |
| `HANDOVER.md` | Mark the F3 status and document the routes. |

---

## 3. Routes

| Method | Path | Who | What |
|---|---|---|---|
| POST | `/studio/obstacles/{obstacleId}/hints` | TEACHER (owner) | Create/update/delete the 3 hints of a monster. |
| GET | `/play/hints` | STUDENT | List published castles from the student's teachers. |
| GET | `/play/{levelId}/hints` | STUDENT (enrolled) | Hint chamber: every floor + its hint panel. |
| POST | `/play/{levelId}/hints/{hintId}/reveal` | STUDENT (enrolled) | Reveal the next hint, record usage, recalculate score. |

These paths require login: `/play/**` for students and `/studio/**` for teachers
(enforced by `SecurityConfig`), so no security config change is needed.

---

## 4. How to run and verify (checklist)

```bash
cd <project-root>
mvn test
mvn spring-boot:run -Dspring-boot.run.profiles=test
```

Open <http://localhost:8080>.

### A. Teacher edits hints

1. Log in as `teacher@codequest.com` / `password123`.
2. Creator Studio → **Tower of Spring MVC** → **Design Castle** → **Design Floor** on Floor 1.
3. Scroll to **💡 3-Level Hints**. The seeded hints are pre-filled.
4. Change the text/cost of Hint 1 (e.g. cost `1`), click **Save Hints**.
5. A green "Hints saved..." banner appears.
6. Try leaving Hint 1 empty while Hint 2 is filled → red error "Hints must be filled in order".
7. Try a negative cost → red error "cost must be between 0 and 100 points".

### B. Student reveals hints in order

1. Log out, log in as `student@codequest.com` / `password123`.
2. Dashboard → **💡 3-Level Hints** card → **Open Hint Library**.
3. Open **Tower of Spring MVC** → **Hint Chamber**.
4. Hint 1 shows **Reveal Hint 1 (-2 pts)**; Hints 2 and 3 show 🔒 Locked.
5. Reveal Hint 1 → its text appears, Hint 2 becomes unlocked.
6. Reload the page → Hint 1 stays revealed and is **not charged again**.
7. Reveal Hint 2, then Hint 3.
8. Try opening `/play/{levelId}/hints/{hint3Id}/reveal` directly for a fresh student → server rejects it with "Hints must be revealed in order".

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
