# Development Strategies
# Claude Code — React Native + Supabase Apps

> These strategies are adopted from the very beginning of every new project.
> They are ordered logically: foundational decisions first, operational practices last.
> Copy this file into `docs/STRATEGIES.md` for each new project. Do not modify it per-project
> unless you are intentionally deviating — note any deviation in PROJECT_STATE.md.

---

## FOUNDATION
> These strategies shape everything else. Establish them before writing a single line of code.

---

## S1 — Multi-Agent Architecture

Use specialized Claude Code agents coordinated by an orchestrator.

**Roles:**

| Agent | Responsibility |
|-------|---------------|
| **lead-orchestrator** | Reads specs + strategies + project state. Breaks work into tasks. Delegates to specialists. Never writes code. |
| **react-native-specialist** | All React Native / Expo / UI / feature / hook work. |
| **supabase-specialist** | All Supabase schema design, RLS policies, SQL migration files. |

**Rules:**
- Specialists never cross domains
- Orchestrator approves checklists before implementation begins
- Spawn specialists using the Agent tool with `subagent_type`
- When spawning: provide (1) specific tasks, (2) files to read, (3) output expectations

**Why this works:** Keeps each agent's context focused. No single agent tries to do everything poorly.

---

## S2 — Specs as the Single Source of Truth

`docs/specs.md` is the authoritative product document.

**Rules:**
- No feature may be built unless it is described in `docs/specs.md`
- If a requirement is unclear: ask the user before implementing. Never guess.
- When a new feature is approved: update `docs/specs.md` first, then implement
- `docs/CLAUDE.md` enforces this rule every session

**Why this works:** Eliminates scope creep. Prevents Claude from inventing features.

---

## S3 — CLAUDE.md as AI Guardrail

`docs/CLAUDE.md` is read at the start of every Claude session — before any other file.

**Contents:**
- What the app IS and is NOT
- Highest priority principle for this app
- Scope control
- Architecture rules
- Banned patterns (by name)
- SQL rules
- SESSION CLOSE obligations
- When in doubt: ask, don't guess

**Maintenance:**
- Update CLAUDE.md when a new strategy is adopted or a rule changes
- CLAUDE.md does NOT contain product requirements — those live in specs.md

**Why this works:** Prevents context drift across sessions.

---

## S4 — PROJECT_STATE.md as Session Bootstrap

A single "read this first" snapshot that eliminates re-reading source files at the start of every session.

**Contents (always kept current):**
- Tech stack table with versions
- Directory structure with file purposes
- Domain layer overview (entities + repositories)
- Context provider hierarchy (with dependency order)
- Full database schema (current state)
- Key function signatures per feature/context
- Navigation route map
- SQL migration list (in order, with purpose + run status)
- Recent architectural decisions (rationale only)
- Files by phase (what was created when)

**Maintenance rules:**
- Claude MUST update PROJECT_STATE.md at the end of every working session
- It is a living snapshot — outdated entries are replaced, not appended
- Stale = every future session pays the cost

**Why this works:** The most expensive thing in a Claude session is re-reading unchanged code.

---

## S4b — DIARY.md as Project History

An append-only project log that captures what was done and why.

**Contents per entry:**
- Session goal
- Files created and modified (with purpose)
- SQL run
- Types introduced (with location)
- Phase relationships
- Decisions made (with rationale)
- What was left unfinished
- What next session should start with

**Maintenance rules:**
- Claude MUST append a new entry at the end of every working session
- Never edit past entries
- On a new device or after a long break: read DIARY.md alongside PROJECT_STATE.md

**Why this works:** PROJECT_STATE.md is a snapshot. DIARY.md is the story. Together they eliminate lost context entirely.

---

## ARCHITECTURE
> Decided before the first line of application code.

---

## S5 — Domain-First Folder Structure

Organize code by domain entity and feature capability — not by file type.

**Canonical structure:**
```
app/                          → Expo Router screens (thin: navigation + layout only)
src/
  features/
    [feature-name]/
      ui/                     → Components for this feature only
      model/                  → State, hooks, business logic
      index.ts                → Public API (barrel exports only)
  domain/
    [entity-name]/
      [Entity].ts             → Type/interface definition
      [entity]Repository.ts   → All data access for this entity
      [entity]Mappers.ts      → DTO → domain model conversion
  shared/
    components/               → Reusable UI (used by 2+ features)
    hooks/                    → Reusable logic (used by 2+ features)
    utils/                    → Pure utility functions
  services/
    supabase.ts               → Supabase client singleton
    storage.ts                → AsyncStorage helpers
    network.ts                → Network detection
  config/
    featureFlags.ts           → Build-time feature toggles
    theme.ts                  → Central design tokens (colors, spacing, typography, radius, shadows)
    constants.ts              → App-wide constants
sql/                          → Database migrations (numbered, append-only)
docs/                         → All project documentation
checklists/                   → Per-phase/feature checklists
```

**Dependency rules (enforce strictly):**
```
features → domain → services    (allowed)
features → shared               (allowed)
features → other features       (FORBIDDEN)
domain → features               (FORBIDDEN)
domain → React                  (FORBIDDEN — pure TypeScript only)
```

**Why this works:** Features are self-contained. Opening one folder gives full understanding of that capability.

---

## S6 — Repository Pattern

Features never touch raw Supabase responses. All data access goes through repositories.

**Chain:**
```
features/[name]/model/ → domain/[entity]/[entity]Repository.ts → Supabase
```

**Rules:**
- Repositories are the ONLY files that call `supabase.from()`
- Repositories return typed domain models, never raw rows
- Mappers handle DTO → domain model transformation
- A schema change only requires updating the repository + mapper

**Why this works:** Backend changes don't cascade into UI code.

---

## S7 — TypeScript-First

Define types before writing implementation code.

**Rules:**
- Domain entity types in `src/domain/[entity]/[Entity].ts`
- Feature-specific types in `src/features/[name]/types.ts` (if needed)
- No `any` — use `unknown` with type guards where input is truly unknown
- Status fields as string literal unions, not enums: `type Status = 'pending' | 'active' | 'done'`
- One definition per type — import everywhere, never redefine
- `npx tsc --noEmit` must pass before marking a phase complete

**Why this works:** Types expose design flaws before implementation begins.

---

## S8 — Context Provider Hierarchy

Document provider dependency order explicitly.

**In `app/_layout.tsx`:**
```typescript
// Context Provider Hierarchy — dependency order matters
// 1. UserProvider         [ACTIVE] — identity, no deps
// 2. DataProvider         [ACTIVE] — depends on UserProvider
// 3. FeatureProvider      [ACTIVE] — depends on User + Data
// 4. FlaggedProvider      [Phase N, flagged] — enable with FLAG_NAME
```

**Rules:**
- Providers listed in dependency order (outermost = no dependencies)
- Each provider marked: [ACTIVE] / [Phase N, flagged] / [PLANNED]
- Flagged providers only added when their flag is `true`

**Why this works:** Provider order bugs are silent and hard to debug. This makes the contract explicit.

---

## S9 — Feature Flag Pattern

Every non-core feature is hidden behind a build-time flag.

**Flag file:** `src/config/featureFlags.ts`
```typescript
export const FEATURE_FLAGS = {
  FEATURE_NAME: false,
} as const;
```

**Rules:**
1. Default is always `false` — new flags start disabled
2. Before adding any entry point for a flagged feature: verify flag is `true`
3. Never hardcode a feature that has a flag
4. One flag at a time — only enable one flag per development session
5. Flags are build-time — no remote config, no runtime toggles
6. Remove flags when stable (delete constant + all conditional code)

**Lifecycle:** `PLANNED → BUILDING → TESTING → STABLE → REMOVED`

**Why this works:** Incomplete features never reach production. One boolean controls everything.

---

## SQL RULES

---

## S10 — SQL Files Only, Never Execute

All SQL lives in numbered files. Claude never executes SQL.

**Rules:**
- All SQL saved to `sql/NNN-description.sql` (e.g., `001-enable-uuid-extension.sql`)
- Every file has a header comment: what it does, why, dependencies, run instructions
- You run SQL manually in the Supabase SQL Editor
- Migrations are append-only — never edit existing files
- Schema migrations and RLS policy migrations are SEPARATE files

**Why this works:** You maintain full control. No accidental schema changes.

---

## S11 — SQL Before Application Code

Write and verify RLS policies before writing any feature code that depends on that table.

**Order:**
1. Schema SQL file (create table)
2. RLS SQL file (policies)
3. TypeScript types (domain entity)
4. Repository (data access)
5. Feature code (UI + hooks)

**Why this works:** RLS bugs are invisible until production. Catching them first saves hours.

---

## S12 — SECURITY DEFINER for Cross-Table RLS

Use helper functions to avoid RLS recursion.

**Problem:** Policy on `table_a` queries `table_b`, which has a policy that queries `table_a` → infinite recursion.

**Solution:**
```sql
CREATE OR REPLACE FUNCTION is_[entity]_member(p_[entity]_id UUID)
RETURNS BOOLEAN LANGUAGE sql SECURITY DEFINER SET search_path = public AS $$
  SELECT EXISTS (
    SELECT 1 FROM [members_table]
    WHERE [entity]_id = p_[entity]_id AND user_id = auth.uid()
  );
$$;
```

**Rules:**
- Use direct `auth.uid()` checks whenever possible
- Only create helpers when RLS recursion is unavoidable
- Document each helper: what it checks, why it exists

**Why this works:** Avoids the most common Supabase RLS debugging nightmare.

---

## DEVELOPMENT PROCESS

---

## S13 — Checklist-First Development

Every feature begins with a written checklist, approved by the user before code is written.

**Checklist file:** `checklists/[feature-name].md`

**Template:**
```markdown
# Phase N: [Name]

## Goal
[One sentence: what this accomplishes]

## Why it exists
[What problem it solves; why it belongs in this phase]

## Rules in force
[Which strategies and constraints apply to this phase]

## Tasks

### Task N.M — [Name]
- Owner: [react-native-specialist | supabase-specialist]
- Dependencies: [what must exist first]
- Definition of Done:
  - [ ] Criterion 1
  - [ ] Criterion 2

## Manual Test Plan
- [ ] Happy path: [step by step]
- [ ] Offline: [what should happen]
- [ ] Edge case: [specific scenario]
- [ ] Regression: [what to verify]

## Completion Criteria
Phase N is complete when:
1. [criterion]
2. docs/PROJECT_STATE.md updated
3. docs/DIARY.md entry appended

## What is NOT in this phase
[Explicit exclusions]
```

**Standard phase sequence:**
1. Infrastructure (framework setup, folder scaffold, core dependencies)
2. Auth & Storage Foundation (identity, local persistence, services layer, types)
3. Domain Layer Setup (all entity types, repositories, mappers)
4. [Core Feature 1]
5. [Core Feature 2]
6. Offline Support (queue, network detection, sync)
7–N. Additional features per FEATURE_FLAGS

**Rules:**
- No implementation until checklist is approved
- No task starts until its dependencies are checked off
- `DEV_CHECKLIST.md` tracks all phases at a glance
- A task is not done until its Definition of Done is fully met

**Why this works:** Forces design clarity before implementation. Shared understanding with the user.

---

## S14 — Offline-First with Optimistic UI

User actions never wait for a network response.

**Write pattern:**
```
User action
  → 1. Update local state immediately (optimistic)
  → 2. Persist to AsyncStorage
  → 3. Attempt Supabase write in background
  → 4a. Success: nothing changes (already consistent)
  → 4b. Failure: push to offline queue
```

**Offline queue item:**
```typescript
{ id: string, type: string, payload: unknown, timestamp: number, retries: number }
```
Max 5 retries. FIFO processing on network restore.

**Client UUIDs:** All entity IDs generated client-side via `expo-crypto`. Enables offline creation.

**Why this works:** App feels instant regardless of connectivity.

---

## S15 — Minimal Realtime Subscriptions

Use Supabase realtime only where data must visibly sync between devices.

| Use case | Method |
|----------|--------|
| Persisted state changes | `postgres_changes` |
| Ephemeral signals | `broadcast` |
| Data static during session | Fetch once, no subscription |

**Rules:**
- Always unsubscribe in `useEffect` cleanup
- Subscribe only in the context/hook that owns that data
- Don't subscribe unless realtime is confirmed as a requirement

**Why this works:** Avoids wasting Supabase connection limits on unnecessary subscriptions.

---

## S16 — Manual Test Plan per Feature

Each checklist includes a manual test section.

**Required test categories:**
| Category | What to test |
|----------|-------------|
| Happy path | Standard flow, network connected |
| Offline | Same flow in airplane mode |
| Edge cases | Disconnect mid-flow, app backgrounded |
| Regression | Verify previously-working features still work |

**Rules:**
- Feature is not "done" until all test cases are checked off
- Test on physical device for camera, QR, device-specific behavior
- Failing test = keep task open, add fix task, do not mark complete

**Why this works:** Real-world edge cases are where bugs hide.

---

## S17 — Anti-Overengineering Rules

When there are two ways to implement something, choose the simpler one.

**Banned patterns (do not introduce without explicit user approval):**

| Pattern | Why it's banned |
|---------|----------------|
| Redux / MobX / Zustand | React Context is sufficient |
| Microservices or external APIs beyond Supabase | Supabase handles auth, DB, realtime, storage |
| Complex in-memory caching | AsyncStorage + Supabase is sufficient |
| Analytics aggregation pipelines | Store raw data; calculate later |
| Push notifications in V1 | Add when real demand exists |
| UI component libraries | Plain StyleSheet.create() is readable, dependency-free |
| GraphQL | Supabase REST + realtime is sufficient |
| Background sync services | Offline queue is enough |

**Decision test:**
> "Is there a simpler way to achieve this?"
> If yes — choose it.

**Why this works:** Every abstraction has a maintenance cost. Add complexity only when demanded by real usage.

---

## S18 — Skills for Consistency

Skills are instruction files that Claude and all specialist agents must read before performing specific types of work. They enforce consistent patterns across every phase.

**Required skills (always read before the relevant work):**

| Skill | File | Read when |
|-------|------|-----------|
| rn-styles | `skills/rn-styles/SKILL.md` | Any styles, colors, spacing, theming |
| rn-types | `skills/rn-types/SKILL.md` | Any TypeScript types or interfaces |
| rn-navigation | `skills/rn-navigation/SKILL.md` | Any navigation, tabs, routing |
| rn-component | `skills/rn-component/SKILL.md` | Any UI component creation |
| rn-repository | `skills/rn-repository/SKILL.md` | Any repository, mapper, domain entity |
| supabase-migration | `skills/supabase-migration/SKILL.md` | Any SQL migration file |
| checklist | `skills/checklist/SKILL.md` | Creating any checklist |

**Core enforcements:**
- All styles flow from `src/config/theme.ts` — no hardcoded values anywhere
- All types defined once and imported — never redefined
- Tabs-first navigation via Expo Router `(tabs)/` group
- Shared components in `src/shared/components/` with barrel export
- Repository pattern strictly enforced — features never call `supabase.from()` directly

**When to create a new skill:**
- A code pattern appears in 2+ phases
- A document format must be consistent across the project
- A workflow step is complex enough to be done inconsistently

**Why this works:** Reduces Claude's variability. Same pattern every time, fewer debugging sessions.
