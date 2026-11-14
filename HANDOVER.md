# CodeQuest — Handover & Progress Log

**Date:** November 12, 2026  
**Demo Deadline:** November 17, 2026 (University project, team of 4 beginners)  
**Stack:** Java 17, Maven 3.9+, Spring Boot 3.3.4, Spring Data JPA, Thymeleaf, Spring Security, MySQL 8 / H2 in-memory test profile.  
**Purpose:** Comprehensive handover log for any incoming AI or developer to resume without loss of context.

> ⚠️ **STATUS UPDATE (November 15, 2026): the codebase was stripped to the bare
> authentication backbone. ALL Sprint 2 features (F1–F5) and the assessment
> plug-in are ⬜ PENDING / INCOMPLETE** — their code, templates and tests were
> removed from the repository and must be rebuilt (planned package layout in
> root `README.md` §2). Sections below that describe features as ✅ COMPLETE are
> historical records of the pre-strip build and no longer reflect the code.

---

## 1. Executive Summary & Current Status

| Phase / Feature | Owner | Status | Notes |
|---|---|---|---|
| **Phase 0: Design & Specs** | All | ✅ **COMPLETE** | Documented in `docs/phase-0-design.md`. ER diagrams, class diagrams, package-by-feature layout, extensibility review. |
| **Phase 1: Backbone** | Core / All | ✅ **COMPLETE** (stripped to auth-only core) | Auth entities/repos, security (BCrypt, form login, CSRF), layout skeleton, idempotent user-only `DemoDataSeeder`. Feature entities/repos kept as foundation. Assessment plug-in **removed — pending rebuild**. |
| **Feature 1: Creator Studio** | Hasin | ⬜ **PENDING — NOT BUILT** | Code removed in the backbone strip-down. Rebuild in Sprint 2 (historical design in §3 and `team/hasin/feature-1-creator-studio.md`). |
| **Feature 2: Classroom & Roles** | Hasin | ⬜ **PENDING — NOT BUILT** | Code removed. Rebuild in Sprint 2. |
| **Feature 3: 3-Level Hints** | Adnan | ⬜ **PENDING — NOT BUILT** | Code, templates and 13 tests removed. Rebuild in Sprint 2. |
| **Feature 4: Discussion Board** | Tafika | ⬜ **PENDING — NOT BUILT** | Code, templates and 8 tests removed. Rebuild in Sprint 2. |
| **Feature 5: Profile & Play Page** | Tanvir | ⬜ **PENDING — NOT BUILT** | Code, templates and 14 tests removed. Rebuild in Sprint 2. |

**Snapshot (November 15, 2026):** backbone-only base committed on `master`
("Strip feature modules to bare authentication backbone"). 1 backbone test
passing. App boots and serves landing/login/register plus a placeholder
`/dashboard` after login; no feature UI exists.

> **Sprint submission:** see [`team/SPRINTS.md`](team/SPRINTS.md) for the Sprint 1 vs
> Sprint 2 split and the exact `git add` lists. Recommended: Sprint 1 = foundation
> (Phase 0 + Phase 1 backbone), Sprint 2 = all five features + polish.
>
> **Full 20-feature map & team breakdown:** see [`PROJECT_STATUS.md`](PROJECT_STATUS.md).
>
> **Lost in the folders?** see [`PROJECT_MAP.md`](PROJECT_MAP.md) for what every file
> does and an “I want to change X → edit this file” table.

---

## 2. Key Architecture & Design Rules Already Enforced

1. **Database Tables & Reserved Keywords:**
   - User entity uses `@Table(name="users")`.
   - Comment entity uses `@Table(name="comments")`, text column is named `body` (avoids SQL reserved words).
2. **Extensible Assessment Plugin Pattern (`com.codequest.assessment`):**
   - Generic `assessments` table (`type`, `prompt`, `content` [LONGTEXT JSON], `answer_key` [LONGTEXT JSON], `points`, `order_index`).
   - `AssessmentHandler` interface: `getType()`, `getDisplayName()`, `validateDefinition(...)`, `check(...)`.
   - Handlers auto-injected into `AssessmentService`.
   - MCQ implemented; answer keys NEVER sent to browser.
3. **Centralized Scoring:**
   - [`ScoreService`](file:///home/adnan/Desktop/Project/src/main/java/com/codequest/common/service/ScoreService.java) calculates: `max(0, sum(distinct correct assessment points) - sum(distinct used hint costs))`.
4. **Floor Modification Safety:**
   - Floor addition/deletion/reordering is only allowed when `level.published == false` AND no `LevelProgress` records exist.
   - Floor swaps use a temporary `-999` index so `UNIQUE(level_id, floor_number)` constraint is never violated.
5. **Security Specific-before-General Matchers:**
   - In `SecurityConfig`: `/classroom/join` is matched BEFORE `/classroom/**`.
   - `app.teacher-code` (default: `TEACH-2026`) is required during teacher registration.

---

## 3. Directory Layout & File Ownership

```text
/home/adnan/Desktop/Project/
├── docs/
│   └── phase-0-design.md                  # Complete architectural blueprint
├── team/
│   ├── SPRINTS.md                          # Sprint 1/2 split + exact git commands
│   ├── hasin/README.md, feature-1-creator-studio.md
│   ├── adnan/README.md, feature-3-hints.md
│   ├── tafika/README.md, feature-4-discussion.md
│   └── tanvir/README.md, feature-5-profile-and-play.md
├── pom.xml                                # Spring Boot 3.3.4 parent, Java 17
├── docker-compose.yml                     # Local MySQL 8.4 container option
├── README.md                              # Run & troubleshooting guide
├── HANDOVER.md                            # This file
├── PROJECT_STATUS.md                      # 20-feature status + per-person breakdown
├── PROJECT_MAP.md                         # Where every file lives + "edit X here" guide
└── src/
    ├── main/
    │   ├── java/com/codequest/
    │   │   ├── CodeQuestApplication.java
    │   │   ├── domain/                    # 12 Shared JPA Entities
    │   │   │   ├── User.java              # @Table(name="users")
    │   │   │   ├── Role.java              # ROLE_TEACHER, ROLE_STUDENT
    │   │   │   ├── Classroom.java
    │   │   │   ├── ClassroomMember.java
    │   │   │   ├── Level.java
    │   │   │   ├── Floor.java
    │   │   │   ├── Obstacle.java
    │   │   │   ├── Assessment.java
    │   │   │   ├── Attempt.java
    │   │   │   ├── Hint.java
    │   │   │   ├── HintUsage.java
    │   │   │   ├── Comment.java           # @Table(name="comments"), column "body"
    │   │   │   └── LevelProgress.java
    │   │   ├── repository/                # 12 Spring Data Repositories
    │   │   ├── assessment/                # Assessment Plugin Infrastructure
    │   │   │   ├── AssessmentHandler.java
    │   │   │   ├── CheckResult.java
    │   │   │   ├── JsonSupport.java
    │   │   │   ├── AssessmentService.java
    │   │   │   └── mcq/McqAssessmentHandler.java
    │   │   ├── common/                    # Backbone Infrastructure
    │   │   │   ├── config/SecurityConfig.java
    │   │   │   ├── security/CodeQuestUserDetailsService.java
    │   │   │   ├── security/LoginSuccessHandler.java
    │   │   │   ├── service/ScoreService.java
    │   │   │   ├── web/HomeController.java
    │   │   │   ├── web/GlobalExceptionHandler.java
    │   │   │   └── seed/DemoDataSeeder.java
    │   │   ├── studio/                    # F1: Creator Studio (Hasin)
    │   │   │   ├── StudioController.java
    │   │   │   ├── StudioService.java
    │   │   │   └── form/LevelForm, FloorForm, ObstacleForm, AssessmentForm
    │   │   ├── classroom/                 # F2: Classroom & Roles (Hasin)
    │   │   │   ├── ClassroomController.java
    │   │   │   ├── ClassroomService.java
    │   │   │   └── form/ClassroomForm, JoinClassForm
    │   │   ├── hints/                     # F3: 3-Level Hints (Adnan) ✅
    │   │   │   ├── HintController.java
    │   │   │   ├── HintService.java
    │   │   │   ├── form/HintForm.java
    │   │   │   └── view/HintView.java
    │   │   ├── discussion/                # F4: Discussion Board (Tafika) ✅
    │   │   │   ├── DiscussionController.java
    │   │   │   └── DiscussionService.java
    │   │   └── profile/                   # F5: Profile, Progress & Play (Tanvir) ✅
    │   │       ├── PlayController.java
    │   │       ├── PlayService.java
    │   │       ├── ProfileController.java
    │   │       ├── ProfileService.java
    │   │       └── view/                  # AssessmentView, FloorView, PlayState, SubmissionResult, LevelRow, ProfileSummary
    │   └── resources/
    │       ├── application.properties     # Default MySQL profile
    │       ├── application-test.properties# In-memory H2 test profile
    │       ├── static/css/app.css
    │       └── templates/
    │           ├── fragments/layout.html  # Navbar, alerts, footer
    │           ├── home/index.html
    │           ├── auth/login.html, register.html
    │           ├── teacher/home.html
    │           ├── student/home.html
    │           ├── studio/level-list.html, level-form.html, level-design.html, floor-design.html, assessment-form.html
    │           ├── classroom/list.html, form.html, detail.html, join.html
    │           ├── hints/fragments.html   # F3: hintEditor + hintPanel (Adnan) ✅
    │           ├── hints/index.html       # F3: Hint Library ✅
    │           ├── hints/chamber.html     # F3: Hint Chamber ✅
    │           ├── discussion/fragments.html # F4: commentSection + commentModeration ✅
    │           ├── discussion/index.html  # F4: Discussion Boards ✅
    │           ├── discussion/level.html  # F4: one castle's board ✅
    │           ├── student/levels.html    # F5: My Castles list ✅
    │           ├── assessment/mcq-play.html # F5: plug-in MCQ player fragment ✅
    │           ├── profile/play.html      # F5: castle climb ✅
    │           ├── profile/top.html       # F5: summit full code ✅
    │           └── profile/profile.html   # F5: profile stats ✅
    └── test/java/com/codequest/
        ├── CodeQuestApplicationTests.java
        ├── hints/HintFeatureTests.java    # F3: 13 tests ✅
        ├── discussion/DiscussionFeatureTests.java # F4: 8 tests ✅
        └── profile/PlayFeatureTests.java  # F5: 14 tests ✅
```

---

## 4. How to Run & Verify Immediately

```bash
# Run all automated tests (1 backbone test; feature tests are not built yet)
mvn test

# Run app with H2 database in-memory (no MySQL setup needed; quote the -D flag in PowerShell)
mvn spring-boot:run "-Dspring-boot.run.profiles=test"
```

### Seeded Test Credentials:
- **Teacher:** `teacher@codequest.com` / `password123`
- **Student:** `student@codequest.com` / `password123`
- **Teacher Registration Code:** `TEACH-2026`
- **Seeded Castle/Class:** **none** — the seeder now creates only the two demo
  accounts. Castle and classroom content arrive with the Sprint 2 features.

---

## 5. Next Tasks for the Incoming AI

### Feature 3 (Three-Level Hints) — ⬜ PENDING (code removed in strip-down)
Delivered by Adnan in `com.codequest.hints`:
- `HintService`: save/update/delete the 3 hints, strict-order reveal, single charge per hint, `ScoreService` integration.
- `HintController`: `POST /studio/obstacles/{obstacleId}/hints`, `GET /play/hints`, `GET /play/{levelId}/hints`, `POST /play/{levelId}/hints/{hintId}/reveal`.
- `templates/hints/fragments.html`: `hintEditor` (teacher) + `hintPanel` (student).
- `templates/hints/index.html` and `templates/hints/chamber.html`: F3-owned verification pages.
- 13 tests in `HintFeatureTests` (all passing).
- Details and the manual checklist: `team/adnan/feature-3-hints.md`.

**F5 integration note:** implemented — the play page now injects `HintService` and includes
`~{hints/fragments :: hintPanel(${levelId}, ${obstacle}, ${hintViews}, ${returnUrl})}`.

### Feature 4 (Discussion Board) — ⬜ PENDING (code removed in strip-down)
Delivered by Tafika in `com.codequest.discussion`:
- `DiscussionService`: access checks (published castle + joined classroom), 2000-char validation, student/teacher posting, teacher-only delete.
- `DiscussionController`: `GET /play/discussion`, `GET /play/{levelId}/discussion`, `POST /play/{levelId}/comments`, `POST /studio/levels/{levelId}/comments`, `POST /studio/comments/{commentId}/delete`.
- `templates/discussion/fragments.html`: `commentSection` (student) + `commentModeration` (teacher).
- `templates/discussion/index.html` and `templates/discussion/level.html`: F4-owned verification pages.
- 8 tests in `DiscussionFeatureTests` (all passing).
- Details and the manual checklist: `team/tafika/feature-4-discussion.md`.

**F5 integration note:** implemented — the play and summit pages include
`~{discussion/fragments :: commentSection(${level}, ${comments}, ${returnUrl})}`.

### Feature 5 (Profile, Progress & Play — Tanvir) — ⬜ PENDING (code removed in strip-down)
Delivered by Tanvir in `com.codequest.profile`:
- `PlayService`: authorization through `findAvailableForStudent`, `LevelProgress` creation on first visit, answer evaluation through `AssessmentService.check`, append-only `Attempt` logging, floor unlock/advance, castle completion, score refresh through `ScoreService`.
- `PlayController`: `GET /play/{levelId}` and `POST /play/{levelId}/assessments/{assessmentId}/submit`; embeds F3 hints and F4 comments.
- `ProfileService` / `ProfileController`: `GET /student/levels` and `GET /student/profile`; completion %, streak, total score and per-castle rows.
- Templates: `profile/play.html` (climb), `profile/top.html` (summit), `profile/profile.html`, `student/levels.html`, and `assessment/mcq-play.html` rewritten for `AssessmentView`.
- Plug-in player selection: `~{assessment/__${a.typeKey}__-play :: __${a.typeKey}__Play(${a})}` — new types need only a new play fragment.
- 14 tests in `PlayFeatureTests` (all passing; 37 total project tests).
- Details and the manual checklist: `team/tanvir/feature-5-profile-and-play.md`.

### Next: Phase 3 (Polish & Submission) — resume here tomorrow
All five chosen features are ⬜ **PENDING** — they must be rebuilt in Sprint 2
before Phase 3 polish. The historical 20-feature status and per-person breakdown
are in [`PROJECT_STATUS.md`](PROJECT_STATUS.md).

**Resume checklist:**
1. `mvn test` → expect **1 passing backbone test** (feature tests are not built yet).
2. Run the app: `mvn spring-boot:run "-Dspring-boot.run.profiles=test"`.
3. Log in with `teacher@codequest.com` / `student@codequest.com`
   (password `password123`) and confirm the placeholder dashboard loads.
4. Add a second assessment type (`SCRAMBLE` or `FILL_BLANK`) to prove the plug-in path:
   handler + editor fragment + player fragment only.
5. Check the responsive layout on a narrow screen.
6. Commit/tag the sprints using the ready-made commands in [`team/SPRINTS.md`](team/SPRINTS.md).
   Nothing is committed yet; `sprint-1` / `sprint-2` tags do not exist.
7. After Phase 3, pick the next small feature from `PROJECT_STATUS.md` §4:
   **#19** star rating (Adnan) or **#14** achievement badges (Tanvir).

**Do not stage:** `File.txt` / `hello.py` deletions, PDFs, or `target/`.
