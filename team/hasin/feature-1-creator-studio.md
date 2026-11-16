# Hasin — Feature 1 & Feature 2 Workspace

**Sprint:** 2  
**Features:** F1 — Teacher Content Creator Studio; F2 — Classroom & Role Management

**Status:** ⏳ **Pending (Sprint 2)**

**Assigned Features:**
- **F1: Teacher Content Creator Studio** (Primary focus now)
- **F2: Classroom & Role Management** (Shared with backbone)

---

## 1. Feature 1: Teacher Content Creator Studio (Overview)
The Creator Studio gives instructors full control over creating, designing, and publishing castles.

### File Ownership Map for F1 — *(Pending implementation)*
- **Controller:** `src/main/java/com/codequest/studio/StudioController.java`
- **Service:** `src/main/java/com/codequest/studio/StudioService.java`
- **Forms & DTOs:**
  - `com.codequest.studio.form.LevelForm`
  - `com.codequest.studio.form.FloorForm`
  - `com.codequest.studio.form.ObstacleForm`
  - `com.codequest.studio.form.AssessmentForm`
- **Thymeleaf Templates:**
  - `templates/studio/level-list.html` (List castles authored by this teacher)
  - `templates/studio/level-form.html` (Create / edit level metadata)
  - `templates/studio/level-design.html` (Tower ladder, floors, publish button)
  - `templates/studio/floor-design.html` (Monster selection & assessment builder)
  - `templates/studio/assessment-form.html` (Dynamic editor driven by AssessmentService)

---

## 2. Business Rules & Constraints
1. **Ownership Check:** A teacher can ONLY edit, publish, or delete levels that belong to them (`level.teacher.id == currentUser.id`).
2. **Floor Modification Safety:** Adding, deleting, or reordering floors is only allowed while `level.published == false` and the level has no `LevelProgress` records.
3. **Publish Validation Rules:**
   - Must have at least 1 floor.
   - Every floor must have an obstacle (monster).
   - Every obstacle must have at least 1 assessment.
   - Every assessment definition must validate through its `AssessmentHandler`.
