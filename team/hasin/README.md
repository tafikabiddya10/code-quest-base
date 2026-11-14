# Hasin — Sprint Index

| Sprint | Feature | Status | Doc |
|---|---|---|---|
| Sprint 1 | Phase 1 backbone support (auth, security, layout, entities) | ✅ Complete | [`../../HANDOVER.md`](../../HANDOVER.md) |
| **Sprint 2** | **F1 — Teacher Content Creator Studio** | ⬜ **PENDING — NOT BUILT** | [`feature-1-creator-studio.md`](feature-1-creator-studio.md) |
| **Sprint 2** | **F2 — Classroom & Role Management** | ⬜ **PENDING — NOT BUILT** | [`feature-1-creator-studio.md`](feature-1-creator-studio.md) |

> ⚠️ **Status note:** the codebase has been stripped down to the bare
> authentication backbone. All Sprint 2 feature code listed under "Code paths
> owned" was **removed** and must be **rebuilt from scratch in Sprint 2**. The
> linked feature doc is kept as a historical blueprint for the rebuild; nothing
> in it exists in the codebase right now.

## Code paths owned (to be created in Sprint 2)
- `src/main/java/com/codequest/studio/`
- `src/main/java/com/codequest/classroom/`
- `src/main/resources/templates/studio/`
- `src/main/resources/templates/classroom/`

## Git scope
Sprint 2 will stage the `studio/` and `classroom/` packages, their templates, and
the feature doc once rebuilt. Sprint 1 contains only the shared backbone
(entities, repositories, security, layout, user seeding). See
[`../SPRINTS.md`](../SPRINTS.md) for the planned `git add` lists.
