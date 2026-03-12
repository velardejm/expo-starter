---
name: checklist
description: Checklist generation standard for phased development. Use this skill whenever creating a new phase checklist or feature checklist. Enforces consistent structure, task ownership, definition of done, test plans, and mandatory session-close obligations.
---

## When to Create a Checklist

**Before any code is written for a phase or feature.** No exceptions.

1. Create the checklist file in `checklists/[phase-or-feature].md`
2. Present it to the user
3. Wait for explicit approval ("looks good", "approved", "proceed")
4. Only then spawn specialists and begin work

---

## Checklist File Naming

```
checklists/phase-1-infrastructure.md
checklists/phase-2-auth-storage.md
checklists/phase-3-domain-layer.md
checklists/phase-4-[feature-name].md
checklists/feature-[name].md          ← for mid-phase features
```

---

## Full Checklist Template

```markdown
# Phase N: [Phase Name]
# [APP_NAME]

## Goal
[One sentence: what the user can do after this phase is complete]

## Why it exists
[What problem it solves. Why it must happen before the next phase.]

## Rules in force
- SQL before application code (S11)
- TypeScript-first: types before implementation (S7)
- Read rn-styles skill before any UI work
- Read rn-types skill before any type definitions
- Read rn-navigation skill before any routing changes
- Read rn-component skill before any component creation
- Read rn-repository skill before any repository/mapper work
- Read supabase-migration skill before any SQL file creation
- [Any feature flags relevant to this phase]
- [Dependencies on previous phase outputs]

---

## Tasks

---

### Task N.1 — [Task Name]
- **Owner:** [react-native-specialist | supabase-specialist]
- **Skills to read:** [rn-types | rn-styles | rn-component | rn-navigation | rn-repository | supabase-migration]
- **Dependencies:** [What must exist first — specific file or SQL run]
- **Definition of Done:**
  - [ ] [Specific, verifiable criterion — not vague]
  - [ ] [File created: `path/to/file.ts`]
  - [ ] [SQL saved to `sql/NNN-description.sql`] *(if applicable)*
  - [ ] [User runs SQL manually in Supabase] *(if applicable)*
  - [ ] No TypeScript errors in this file

---

### Task N.2 — [Task Name]
- **Owner:** [react-native-specialist | supabase-specialist]
- **Skills to read:** [list applicable skills]
- **Dependencies:** Task N.1 complete
- **Definition of Done:**
  - [ ] [Criterion]
  - [ ] [Criterion]

---

## Manual Test Plan

- [ ] **Happy path:** [Step by step — what the tester does and what they expect to see]
- [ ] **Offline:** [What should happen when network is off — or "N/A for this phase"]
- [ ] **Edge case:** [Specific scenario — e.g., "app backgrounded mid-flow"]
- [ ] **Regression:** [Which previously-working feature to verify hasn't broken]
- [ ] **TypeScript:** `npx tsc --noEmit` passes with zero errors

---

## Completion Criteria

Phase N is complete when:
1. All task checkboxes above are checked off
2. All manual test cases pass
3. `npx tsc --noEmit` passes
4. `docs/PROJECT_STATE.md` updated with Phase N snapshot
5. `docs/DIARY.md` entry appended for this phase
6. `checklists/DEV_CHECKLIST.md` phase row updated to ✅ Complete

---

## What is NOT in this phase

- [Explicit exclusion — what was deliberately left out]
- [Reference to which phase handles it]
```

---

## Task Ownership Rules

| Task type | Owner |
|-----------|-------|
| SQL schema, RLS, helper functions | `supabase-specialist` |
| TypeScript types, domain entities | `react-native-specialist` |
| Repositories and mappers | `react-native-specialist` |
| React components, screens, hooks | `react-native-specialist` |
| Navigation changes | `react-native-specialist` |
| Context providers | `react-native-specialist` |

**Never assign a task to both specialists.** If a task has both SQL and TypeScript work, split it into two tasks.

---

## Spawning Specialists

When spawning a specialist, the orchestrator must provide:

```
Task: [Task N.M name and description]

Files to read:
- docs/specs.md (product requirements)
- docs/FEATURE_FLAGS.md (flag registry)
- checklists/phase-N-[name].md (this checklist — task N.M specifically)
- skills/[relevant-skill]/SKILL.md (read before starting)

Output expectations:
- Create file: [path]
- Create file: [path]
- SQL saved to: sql/NNN-[description].sql (if applicable)

Definition of Done:
[Copy the exact DoD from the checklist task]
```

---

## DEV_CHECKLIST.md Update

After creating a new phase checklist, update `checklists/DEV_CHECKLIST.md`:

```markdown
| N | [Phase Name] | 🔄 In Progress | `checklists/phase-N-[name].md` |
```

After phase is complete:
```markdown
| N | [Phase Name] | ✅ Complete | `checklists/phase-N-[name].md` |
```

---

## Common Mistakes to Avoid

- **Vague DoD:** "implement the screen" → too vague. Be specific: "Create `src/features/auth/ui/LoginScreen.tsx` with email input, password input, and submit button"
- **Missing skill reference:** every task must list which skill(s) to read
- **Missing test plan:** every checklist needs at least a happy path test
- **No completion criteria:** every checklist must end with the 6 completion criteria
- **Forgetting PROJECT_STATE and DIARY:** these are mandatory in completion criteria, not optional
