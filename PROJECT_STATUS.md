# CodeQuest — Project Status & 20-Feature Map

**Last updated:** November 15, 2026  
**Current state:** ⚠️ **BARE BACKBONE ONLY — all 5 Sprint 2 feature modules (F1–F5) and the assessment plug-in are ⬜ PENDING / INCOMPLETE.** Their code, templates and tests were stripped from the repository and must be rebuilt (planned package layout: root `README.md` §2). The tables below are historical records of the pre-strip build and sprint planning.  
**Automated tests:** 1 backbone test passes (context boot + seeded accounts + BCrypt).  
**Run:** `mvn spring-boot:run "-Dspring-boot.run.profiles=test"` (H2, no MySQL needed; quote the `-D` flag in PowerShell).

This file answers three questions:
1. What have we built so far?
2. What is still missing from the original 20-feature list?
3. Which numbers belong to which team member?

**Quick answer:** spec features **#1, #2, #11, #12, #15, #16 are DONE**;
**#5 and #18 are PARTIAL**; **#3, #4, #6, #7, #8, #9, #10, #13, #14, #17, #19, #20
are NOT BUILT**.

---

## 1. The 5 Features We Chose to Build

The build prompt deliberately narrowed the 20-feature vision to five demo-critical
features. These are the five vertical slices that are **complete**:

| Our ID | Spec # | Feature | Owner | Status |
|---|---|---|---|---|
| **F1** | **#11** | Course Content Creator Studio | Hasin | ⬜ **PENDING — NOT BUILT** (code stripped; to be rebuilt in Sprint 2) |
| **F2** | **#2** | Classroom & Role Management | Hasin | ⬜ **PENDING — NOT BUILT** (code stripped; to be rebuilt in Sprint 2) |
| **F3** | **#15** | 3-Step Smart Hints | Adnan | ⬜ **PENDING — NOT BUILT** (code stripped; to be rebuilt in Sprint 2) |
| **F4** | **#16** | Level Discussion & Question Board | Tafika | ⬜ **PENDING — NOT BUILT** (code stripped; to be rebuilt in Sprint 2) |
| **F5** | **#1** | Student Profile & Progress Tracker + Play Page | Tanvir | ⬜ **PENDING — NOT BUILT** (code stripped; to be rebuilt in Sprint 2) |

Supporting foundation that they will be rebuilt on: the 12 JPA entities and
12 repositories (kept), Spring Security (kept), the shared layout/CSS (kept),
user-only seed data, and the auth backbone (all live). The assessment plug-in
engine was **removed** and is also pending rebuild.

---

## 2. Master 20-Feature Table (exact spec numbers)

Status key: ✅ **DONE** · 🟡 **PARTIAL** · ⬜ **NOT BUILT**

| # | Feature (from spec) | Owner in brief | Status | What exists / what is missing |
|---|---|---|---|---|
| **1** | Student Profile & Progress Tracker | Tanvir | ✅ **DONE** | `/student/profile`: completion %, login streak, per-castle progress, score table. "Projects" are represented by castles/levels. |
| **2** | Admin & Teacher Permissions System | Hasin | ✅ **DONE** (simplified) | TEACHER/STUDENT roles, route protection, teacher registration code. No separate admin role. |
| **3** | Interactive Game Map | Tanvir | ⬜ **NOT BUILT** | Replaced by the My Castles list and the tower-ladder sidebar in the play page. No 2D map. |
| **4** | Walking Character & Item Pickup | Tanvir | ⬜ **NOT BUILT** | Out of scope for this demo (no WASD/movement/items). |
| **5** | Choice-Based Story Sequences | Tanvir | 🟡 **PARTIAL** | Each castle has an initial NPC `npcDialogue`; there are no branching choices. |
| **6** | Clickable Code Inspector | Tafika | ⬜ **NOT BUILT** | Out of scope. |
| **7** | Simple Visualizer | Tafika | ⬜ **NOT BUILT** | Out of scope. |
| **8** | Timed Code Puzzles & Mini-Games | Adnan | ⬜ **NOT BUILT** | `Attempt` stores `timeTakenSeconds`, but there is no timer or timed puzzle UI. |
| **9** | Drag-and-Drop Architecture Sandbox | Adnan | ⬜ **NOT BUILT** | Out of scope. |
| **10** | AI Code Assistant | Tafika | ⬜ **NOT BUILT** | Out of scope. |
| **11** | Course Content Creator Studio | Hasin | ✅ **DONE** | F1: create/edit castles, floors, monsters, MCQ questions, NPC dialogue, final code, publish validation. |
| **12** | Automatic Code Checker Engine | Adnan | ✅ **DONE** (MCQ) | `AssessmentHandler` + `AssessmentService` check submitted JSON against the answer key. Unit-test evaluation is not implemented. |
| **13** | Student Diagnostic Charts | Hasin | ⬜ **NOT BUILT** | Attempt/progress data exists; no charts yet. |
| **14** | Automatic Achievement Badges | Tanvir | ⬜ **NOT BUILT** | Out of scope. |
| **15** | 3-Step Smart Hints | Adnan | ✅ **DONE** | F3: ordered Nudge → Concept → Near-Solution, point costs, `HintUsage` logging. |
| **16** | Level Discussion & Question Board | Tafika | ✅ **DONE** | F4: per-castle comment stream, student posting, teacher moderation/delete. |
| **17** | System Architecture Guide Reader | Tafika | ⬜ **NOT BUILT** | NPC story and summit code exist, but there is no dedicated guide/diagram reader. |
| **18** | User Activity & Audit Logger | Shared | 🟡 **PARTIAL** | `Attempt` and `HintUsage` rows are timestamped; login date/streak tracked. No general audit table, no item pickup log. |
| **19** | Level Feedback & Star Rating Popup | Adnan | ⬜ **NOT BUILT** | Out of scope. |
| **20** | Teacher Class Progress Dashboard | Hasin | ⬜ **NOT BUILT** | Classroom roster exists (F2), but no completion-rate or struggle analytics. |

**Count:** 6 delivered (1, 2, 11, 12, 15, 16) · 2 partial (5, 18) · 12 not built
(3, 4, 6, 7, 8, 9, 10, 13, 14, 17, 19, 20).

---

## 3. Breakdown by Team Member

### Tanvir Muhtady — Game World + Student Progress
**Assigned complex:** Interactive 2D World Map & Character Movement (**#3, #4**),
Story-Based Missions & Interactive Choices (**#5**), Real-Time Live Coding Battles
(not one of the 20 numbered items).  
**Assigned simple:** Student Profile & Learning Progress (**#1**), Achievement Badges
& Rewards (**#14**).

- ✅ **Done:** **#1** — F5 profile, completion %, streak, per-castle progress, score,
  plus the full castle play page and summit code.
- 🟡 **Partial:** **#5** — linear NPC dialogue only; no choices/branching.
- ⬜ **Remaining:** **#14** badges, **#3** map, **#4** walking character, live coding battles.
- **Suggested next:** **#14** badges (small, data already exists) → **#5** choices →
  **#3/#4** (large, only if the course requires the game-world slice).

### Adnan Sakeeb — Puzzles & Checking
**Assigned complex:** Timed Code Arrangement Puzzles (**#8**), Drag-and-Drop
Architecture Sandbox (**#9**), Automatic Answer Checking & Test Case Evaluation (**#12**).  
**Assigned simple:** Three-Level Hint System (**#15**), Level Feedback & Star Rating (**#19**).

- ✅ **Done:** **#15** — F3 hints; **#12** — MCQ answer-key checking core.
- ⬜ **Remaining:** **#19** star rating, **#8** timed puzzles, **#9** drag-and-drop sandbox,
  and the unit-test part of **#12**.
- **Suggested next:** **#19** (small) → **#8** (medium) → **#9** (large).

### Tafika Biddya — Understanding Code
**Assigned complex:** AI-Powered Code Explainer (**#10**), Interactive Clickable Code
Inspector (**#6**), Software Component Flow Visualizer (**#7**).  
**Assigned simple:** Architecture Guide Reader (**#17**), Level Discussion Board (**#16**).

- ✅ **Done:** **#16** — F4 discussion board.
- ⬜ **Remaining:** **#17** guide reader, **#6** clickable inspector, **#7** visualizer,
  **#10** AI explainer.
- **Suggested next:** **#17** (small/medium) → **#6** → **#7** → **#10** (large).

### Hasin Jahan Ananya — Teachers & Classroom
**Assigned complex:** Real-Time Classroom Activity Monitor (closest numbered item:
**#20**), Real-Time Class Group Chat (not one of the 20 numbered items), Student
Learning Diagnostic Charts (**#13**).  
**Assigned simple:** Teacher Content Creator Studio (**#11**), Classroom & Role
Management (**#2**).

- ✅ **Done:** **#11** — F1 Creator Studio; **#2** — F2 classroom/roles.
- ⬜ **Remaining:** **#20** class progress dashboard, **#13** diagnostic charts,
  real-time monitor/group chat.
- **Suggested next:** **#20** class dashboard (uses existing `Attempt`/`LevelProgress`
  data) → **#13** charts.

---

## 4. Recommended Build Order From Here

### Phase 3 — polish (do first, all owners)
1. Run the full manual checklist F1–F5 end to end with the seeded accounts.
2. Add a second assessment type (`SCRAMBLE` or `FILL_BLANK`) to prove the plug-in path:
   handler + editor fragment + player fragment only.
3. Check the responsive layout on a narrow screen.
4. Final test run and sprint commits/tags (see `team/SPRINTS.md`).

### Then, in increasing effort
1. **#19** star rating (small) — Adnan.
2. **#14** achievement badges (small) — Tanvir.
3. **#17** architecture guide reader (small/medium) — Tafika.
4. **#20** teacher class dashboard + **#13** diagnostic charts (medium) — Hasin.
5. **#5** branching story choices + **#18** fuller audit log (medium) — Tanvir/shared.
6. **#8** timed puzzles, **#9** drag-and-drop sandbox, **#6** inspector, **#7** visualizer,
   **#10** AI explainer (large).
7. **#3** 2D map and **#4** walking character (very large; only if required).

---

## 5. Sprint & Git Status

- **Sprint 1:** Phase 0 design + Phase 1 backbone.
- **Sprint 2:** F1–F5 + Phase 3 polish.
- Exact staging lists and commands: [`team/SPRINTS.md`](team/SPRINTS.md).
- All project work is still **uncommitted**. `sprint-1` / `sprint-2` tags are **not**
  created yet. The commands are ready to run when you submit.
- Do **not** stage the unrelated `File.txt` / `hello.py` deletions or the PDFs.

---

## 6. Resume Tomorrow

```bash
cd /home/adnan/Desktop/Project
mvn test                                      # expect 37 passing
mvn spring-boot:run -Dspring-boot.run.profiles=test
```

- Teacher: `teacher@codequest.com` / `password123`
- Student: `student@codequest.com` / `password123`
- Seeded class code: `CS101-DEMO`
- Read next: `HANDOVER.md` §5, this file §4, and `team/SPRINTS.md`.
- Recommended first task: the Phase 3 manual checklist, then **#19** or **#14** as a
  small win.
