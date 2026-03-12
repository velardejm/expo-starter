# Phase N: [Phase Name]
# [APP_NAME]

## Goal
[One sentence: what this phase accomplishes for the user or system]

## Why it exists
[What problem it solves. Why it belongs before Phase N+1.]

## Rules in force
- [Which strategies apply — e.g., SQL before app code (S11), TypeScript-first (S7)]
- [Any feature flags relevant to this phase]
- [Any dependency on previous phase outputs]

---

## Tasks

---

### Task N.1 — [Task Name]
- **Owner:** [react-native-specialist | supabase-specialist]
- **Dependencies:** [What must exist first — file, SQL run, etc.]
- **Definition of Done:**
  - [ ] [Specific, verifiable criterion]
  - [ ] [Specific, verifiable criterion]
  - [ ] TypeScript: no errors in this file
  - [ ] SQL saved to `sql/NNN-description.sql` *(if applicable)*
  - [ ] **User runs SQL manually** *(if applicable)*

---

### Task N.2 — [Task Name]
- **Owner:** [react-native-specialist | supabase-specialist]
- **Dependencies:** Task N.1
- **Definition of Done:**
  - [ ] [Criterion]
  - [ ] [Criterion]

---

## Manual Test Plan

- [ ] **Happy path:** [Step by step — what you do and what you expect]
- [ ] **Offline:** [What should happen when network is off]
- [ ] **Edge case:** [Specific scenario — e.g., "app backgrounded mid-flow"]
- [ ] **Regression:** [Which previously-working feature to verify after this change]
- [ ] **TypeScript:** `npx tsc --noEmit` passes

---

## Completion Criteria

Phase N is complete when:
1. All tasks above are checked off
2. All manual tests pass
3. `docs/PROJECT_STATE.md` updated with Phase N snapshot
4. `docs/DIARY.md` entry appended for this phase
5. `checklists/DEV_CHECKLIST.md` phase row updated to ✅ Complete

---

## What is NOT in this phase

- [Explicit exclusion 1 — what was deliberately deferred]
- [Explicit exclusion 2]
- [Reference to which phase will handle it]
