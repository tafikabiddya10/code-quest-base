# Tafika — Feature 4: Level Discussion Board

**Sprint:** 2  
**Feature:** F4 — Level Discussion Board

**Status:** ⏳ **Pending (Sprint 2)**

---

## 1. What the feature does

Every castle has a discussion board where students and teachers can:

- read the comment stream (author name, role and timestamp),
- post a new comment,
- let the **teacher who owns the castle** delete inappropriate or off-topic comments.

Rules enforced by the server:

1. Students can only read/post on **published** castles whose teacher teaches a
   classroom the student has joined.
2. Teachers can post on, and delete any comment of, **their own** castles.
3. Another teacher can neither post nor delete on a castle they do not own.
4. Comment bodies are trimmed, must not be empty, and are capped at **2000 characters**.
5. Comment text is always rendered with `th:text`, so HTML is escaped (no XSS).

---

## 2. Files

### New (all owned by F4) — *(Pending implementation)*

| File | Purpose |
|---|---|
| `src/main/java/com/codequest/discussion/DiscussionService.java` | Business rules: access checks, validation, posting, moderation. |
| `src/main/java/com/codequest/discussion/DiscussionController.java` | Student discussion pages + teacher moderation endpoints. |
| `src/main/resources/templates/discussion/fragments.html` | `commentSection` (student) and `commentModeration` (teacher). |
| `src/main/resources/templates/discussion/index.html` | F4 "Discussion Boards" castle list. |
| `src/main/resources/templates/discussion/level.html` | F4 verification page for one castle's board. |
| `src/test/java/com/codequest/discussion/DiscussionFeatureTests.java` | Planned: 8 tests covering posting, validation, access, moderation, rendering. |

### Existing files to touch (small, clearly marked)

| File | Change |
|---|---|
| `src/main/java/com/codequest/studio/StudioController.java` | Inject `DiscussionService`; add `comments` to the level-design model (3 lines). |
| `src/main/resources/templates/studio/level-design.html` | Pass `level` and `comments` to `commentModeration`. |
| `src/main/resources/templates/profile/play.html` | Leave the `commentSection` include with a comment telling F5 which model attributes to pass. |
| `src/main/resources/templates/student/home.html` | F4 card links to `/play/discussion`. |
| `src/main/resources/static/css/app.css` | Add `.cq-comment-*` styles. |

No entity or repository change is needed — `Comment` (`table "comments"`, column `body`)
and `CommentRepository` already exist in the Phase 1 foundation.

---

## 3. Routes

| Method | Path | Who | What |
|---|---|---|---|
| GET | `/play/discussion` | STUDENT | List published castles from the student's teachers. |
| GET | `/play/{levelId}/discussion` | STUDENT (enrolled) | Comment stream + add form for one castle. |
| POST | `/play/{levelId}/comments` | STUDENT (enrolled) | Post a comment. |
| POST | `/studio/levels/{levelId}/comments` | TEACHER (owner) | Post a comment as the teacher. |
| POST | `/studio/comments/{commentId}/delete` | TEACHER (owner) | Delete a comment (moderation). |

These paths require login: `/play/**` for students and `/studio/**` for teachers
(enforced by `SecurityConfig`), so no security change is needed.

---

## 4. How to run and verify (checklist)

```bash
cd <project-root>
mvn test
mvn spring-boot:run -Dspring-boot.run.profiles=test
```

### A. Student posts

1. Log in as `student@codequest.com` / `password123`.
2. Dashboard → **💬 Level Discussion** card → **Open Discussion Boards**.
3. Open **Tower of Spring MVC** → **Discussion Board**.
4. Type a comment, click **Post Comment** → green banner, comment appears with
   your name, `STUDENT` badge and time.
5. Submit an empty comment → browser blocks it; submitting only spaces shows
   "Comment cannot be empty."

### B. Teacher moderates

1. Log in as `teacher@codequest.com` / `password123`.
2. Creator Studio → **Tower of Spring MVC** → **Design Castle**.
3. The **💬 Discussion Moderation** panel lists the student's comment.
4. Post a reply with **Post as teacher** → it appears on the student board too.
5. Click **× Delete** on a comment → confirm → it disappears from both pages.
6. Open another teacher's castle (create one if needed) → they cannot moderate it.

---

## 5. How F5 (Tanvir) plugs the section into the play page

Add these to the model in the play controller:

```java
model.addAttribute("level", level);
model.addAttribute("comments", discussionService.getComments(level));
model.addAttribute("returnUrl", "/play/" + level.getId());
```

Then include the fragment in `profile/play.html`:

```html
<div th:replace="~{discussion/fragments :: commentSection(${level}, ${comments}, ${returnUrl})}"></div>
```

`DiscussionService` is a normal Spring bean, so F5 can inject it without changing F4 code.

---

## 6. What you just learned

- **Ownership checks belong in the service:** the controller should not decide who may delete.
- **Reuse the existing access query:** `LevelRepository.findAvailableForStudent` keeps F3 and F4 consistent.
- **`th:text` escapes HTML automatically**, which prevents users from injecting scripts into comments.
- **Moderation is just another endpoint** protected by the same `ROLE_TEACHER` rule.
- **Small cross-feature edits are fine** when they are explicit: one constructor line and one fragment include.
