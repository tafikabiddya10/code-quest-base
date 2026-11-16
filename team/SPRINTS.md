# CodeQuest — Sprint Plan & Git Submission Guide

This file is the single source of truth for **which work belongs to which sprint**
and **which files to stage** when you submit each sprint.

> **STATUS:** the Sprint 2 features (F1–F5) are **⏳ Pending (Sprint 2)** and
> have not been implemented yet. The staging lists and git commands below
> describe the *planned* submission flow — the Sprint 2 paths listed there are
> to be created during the sprint. See the root `README.md` §2 for the
> pending-features table.

---

## 1. Recommended Split: Foundation vs Features

The project was built in phases. The safest two-sprint split is:

| Sprint | Theme | What is included | Owners |
|---|---|---|---|
| **Sprint 1** | **Foundation** | Phase 0 design + Phase 1 backbone: entities, repositories, security, layout, seed data, assessment plug-in, login/register, teacher/student home pages | All |
| **Sprint 2** | **Features + demo** | **F1 Creator Studio**, **F2 Classroom & Roles**, **F3 Three-Level Hints**, **F4 Discussion Board**, **F5 Profile & Play**, Phase 3 integration/docs — ⏳ **Pending (Sprint 2)** | Hasin (F1/F2), Adnan (F3), Tafika (F4), Tanvir (F5) |

**Why this split:**
- It matches the build prompt's phase gates: "Phase 1 is complete when a teammate
  can clone, run, log in as teacher and as student, and see the home pages."
- Sprint 1 is runnable on its own (login, roles, seed castle, placeholder dashboards).
- Sprint 2 contains all five vertical slices.
- It is dependency-safe: F1's `StudioController` imports F3's `HintService` and F4's
  `DiscussionService`, so F1/F2 cannot be committed alone. Putting all features in
  Sprint 2 avoids that problem.

**Important dependency fact:** do **not** try "Sprint 1 = F1 + F2 only". It will not
compile because `com.codequest.studio.StudioController` imports
`com.codequest.hints.HintService` and `com.codequest.discussion.DiscussionService`.
F3/F4 must be in the same commit as F1, or F1 must be refactored first.

---

## 2. Alternative Split (if your course wants visible features in Sprint 1)

If your course requires Sprint 1 to show teacher/student features, use this instead:

| Sprint | What is included |
|---|---|
| **Sprint 1** | Phase 0 + Phase 1 backbone + **F1, F2, F3, F4** (F3/F4 are required for F1 to compile) |
| **Sprint 2** | **F5 Profile & Play** + Phase 3 integration/docs |

Both splits compile. Pick the one your course expects. The staging lists for both
are in sections 4 and 5.

---

## 3. Per-Person Sprint Map (recommended split)

| Person | Folder | Sprint 1 | Sprint 2 | Feature doc |
|---|---|---|---|---|
| **Hasin** | `team/hasin/` | Phase 1 backbone support | F1 Creator Studio, F2 Classroom & Roles | `feature-1-creator-studio.md` |
| **Adnan** | `team/adnan/` | Phase 1 backbone support | F3 Three-Level Hints | `feature-3-hints.md` |
| **Tafika** | `team/tafika/` | Phase 1 backbone support | F4 Discussion Board | `feature-4-discussion.md` |
| **Tanvir** | `team/tanvir/` | Phase 1 backbone support | F5 Profile, Progress & Castle Play | `feature-5-profile-and-play.md` |

Each person's folder also has a `README.md` with the same mapping and the code
paths they own.

---

## 4. Sprint 1 Staging List (recommended split)

### Files to stage
```text
.gitignore
pom.xml
docker-compose.yml
README.md
HANDOVER.md
docs/
team/SPRINTS.md
team/hasin/README.md
team/adnan/README.md
team/tafika/README.md
team/tanvir/README.md

src/main/java/com/codequest/CodeQuestApplication.java
src/main/java/com/codequest/domain/
src/main/java/com/codequest/repository/
src/main/java/com/codequest/assessment/
src/main/java/com/codequest/common/

src/main/resources/application.properties
src/main/resources/application-test.properties
src/main/resources/static/css/app.css
src/main/resources/templates/fragments/
src/main/resources/templates/home/
src/main/resources/templates/auth/
src/main/resources/templates/teacher/
src/main/resources/templates/student/home.html
src/main/resources/templates/error/
src/main/resources/templates/assessment/

src/test/java/com/codequest/CodeQuestApplicationTests.java
```

**Optional Sprint 1 inputs** (the project brief, if your course wants them):
```text
spec.md
codequest_build_prompt.md
```

> `README.md`, `HANDOVER.md`, `app.css`, `layout.html` and `student/home.html` are
> shared files. They currently mention or link to later features too. That is fine
> for Sprint 1: they are project documentation/shared UI, not feature code. The
> feature links will simply 404 until Sprint 2 is applied.

---

## 5. Sprint 2 Staging List (recommended split)

### F1–F5 code and tests
```text
team/hasin/feature-1-creator-studio.md
team/adnan/feature-3-hints.md
team/tafika/feature-4-discussion.md
team/tanvir/feature-5-profile-and-play.md

src/main/java/com/codequest/studio/
src/main/java/com/codequest/classroom/
src/main/java/com/codequest/hints/
src/main/java/com/codequest/discussion/
src/main/java/com/codequest/profile/

src/main/resources/templates/studio/
src/main/resources/templates/classroom/
src/main/resources/templates/hints/
src/main/resources/templates/discussion/
src/main/resources/templates/profile/
src/main/resources/templates/student/levels.html

src/test/java/com/codequest/hints/
src/test/java/com/codequest/discussion/
src/test/java/com/codequest/profile/
```

> `src/main/resources/templates/assessment/mcq-play.html` was already staged in
> Sprint 1 as part of the backbone's assessment plug-in. Do not stage it twice.

### Alternative split (section 2) staging lists

**Sprint 1 alternative:** everything from section 4 **plus**:
```text
team/hasin/feature-1-creator-studio.md
team/adnan/feature-3-hints.md
team/tafika/feature-4-discussion.md

src/main/java/com/codequest/studio/
src/main/java/com/codequest/classroom/
src/main/java/com/codequest/hints/
src/main/java/com/codequest/discussion/

src/main/resources/templates/studio/
src/main/resources/templates/classroom/
src/main/resources/templates/hints/
src/main/resources/templates/discussion/

src/test/java/com/codequest/hints/
src/test/java/com/codequest/discussion/
```

**Sprint 2 alternative:** only F5:
```text
team/tanvir/feature-5-profile-and-play.md

src/main/java/com/codequest/profile/
src/main/resources/templates/profile/
src/main/resources/templates/student/levels.html
src/test/java/com/codequest/profile/
```

---

## 6. Git Workflow (two clean commits)

Run these from the project root **only when you are ready to submit**. They do not
rewrite history; they create two new commits on `main`.

### Step 1 — check the working tree and tests
```bash
git status
mvn test        # backbone tests should pass (feature tests arrive with Sprint 2)
```

### Step 2 — commit Sprint 1 (recommended split)
```bash
git add .gitignore pom.xml docker-compose.yml README.md HANDOVER.md docs/
git add team/SPRINTS.md team/hasin/README.md team/adnan/README.md team/tafika/README.md team/tanvir/README.md
git add src/main/java/com/codequest/CodeQuestApplication.java
git add src/main/java/com/codequest/domain/ src/main/java/com/codequest/repository/
git add src/main/java/com/codequest/assessment/ src/main/java/com/codequest/common/
git add src/main/resources/application.properties src/main/resources/application-test.properties
git add src/main/resources/static/css/app.css
git add src/main/resources/templates/fragments/ src/main/resources/templates/home/
git add src/main/resources/templates/auth/ src/main/resources/templates/teacher/
git add src/main/resources/templates/student/home.html
git add src/main/resources/templates/error/ src/main/resources/templates/assessment/
git add src/test/java/com/codequest/CodeQuestApplicationTests.java

git commit -m "Sprint 1: foundation, security, seed data and assessment plug-in"
git tag -a sprint-1 -m "Sprint 1 submission"
```

### Step 3 — commit Sprint 2 (recommended split)
**Note:** run this once the F1–F5 feature code has been implemented in Sprint 2.
```bash
git add team/hasin/feature-1-creator-studio.md
git add team/adnan/feature-3-hints.md team/tafika/feature-4-discussion.md
git add team/tanvir/feature-5-profile-and-play.md
git add src/main/java/com/codequest/studio/ src/main/java/com/codequest/classroom/
git add src/main/java/com/codequest/hints/ src/main/java/com/codequest/discussion/
git add src/main/java/com/codequest/profile/
git add src/main/resources/templates/studio/ src/main/resources/templates/classroom/
git add src/main/resources/templates/hints/ src/main/resources/templates/discussion/
git add src/main/resources/templates/profile/ src/main/resources/templates/student/levels.html
git add src/test/java/com/codequest/hints/ src/test/java/com/codequest/discussion/
git add src/test/java/com/codequest/profile/

git commit -m "Sprint 2: Creator Studio, Classroom, Hints, Discussion, Profile & Play"
git tag -a sprint-2 -m "Sprint 2 submission"
```

### Step 4 — push
```bash
git push origin main
git push origin sprint-1
git push origin sprint-2
```

If your course wants a branch per sprint instead of tags:
```bash
git branch sprint-1 <sprint-1-commit>
git branch sprint-2 <sprint-2-commit>
git push origin sprint-1 sprint-2
```

---

## 7. How to Verify a Sprint Commit in Isolation

Because git commits do not change your working tree, the best check is a temporary
worktree. This proves the Sprint 1 commit really compiles without feature packages.

```bash
# after committing Sprint 1
git worktree add /tmp/opencode/sprint-1-check sprint-1
cd /tmp/opencode/sprint-1-check
mvn -q test
cd /home/adnan/Desktop/Project
git worktree remove /tmp/opencode/sprint-1-check
```

If Sprint 1 fails to compile, a shared file or a feature import leaked into the
commit. The staging lists above are designed to avoid that.

---

## 8. Quick Answer to "What do I push?"

- **Recommended Sprint 1:** backbone only (entities, repositories, security, layout,
  seed, assessment plug-in, home pages).
- **Recommended Sprint 2:** F1 + F2 + F3 + F4 + F5 + their tests and feature docs.
- **Never push:** `target/`, IDE files, the PDFs (unless required), or the unrelated
  `File.txt` / `hello.py` deletions.
