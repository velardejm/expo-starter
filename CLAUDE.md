# CLAUDE.md
# [APP_NAME]

---

## Mandatory Bootstrap

At the start of every session, read these files in order before doing anything:

1. `docs/CLAUDE.md` — this file
2. `docs/specs.md` — product requirements (single source of truth)
3. `docs/STRATEGIES.md` — development strategies
4. `docs/FEATURE_FLAGS.md` — feature flag registry
5. `docs/PROJECT_STATE.md` — current project snapshot
6. `docs/DIARY.md` — last 2 entries only (recent context)

Do not proceed until all files are read.

---

## Project Authority

`docs/specs.md` is the SINGLE source of truth for all product requirements.

Claude MUST NOT invent features, flows, or systems not described in `docs/specs.md`.

If any requirement is unclear:
→ Ask the user for clarification BEFORE implementing.
Do not guess. Do not fill in gaps. Do not assume intent.

---

## Product Type

**[APP_NAME] is [one-sentence description].**

It IS:
- [What it does]

It is NOT:
- [What it doesn't do — list the anti-goals]

Every design and engineering decision must support [core purpose].

---

## Highest Priority Principle

**[The single most important design rule for this app.]**

Claude must never introduce anything that violates this principle.

---

## Scope Control

Explicitly forbidden without a specs update + user approval:
- [List forbidden features by name]

Do NOT build for hypothetical future scale.

---

## Feature Flag Rules

See `docs/FEATURE_FLAGS.md` for the full registry.

**Rules Claude MUST follow:**
1. Verify flag is `true` before adding any entry point for a flagged feature
2. When implementing a flagged feature: wrap all entry points with the flag check
3. Never hardcode a feature that has a flag
4. New features default to `false`

---

## Agent Architecture

| Agent | Responsibility |
|-------|---------------|
| lead-orchestrator | Coordinates; never writes code |
| react-native-specialist | All RN / Expo / UI / feature work |
| supabase-specialist | All schema / RLS / SQL migration work |

Specialists do not cross domains.

---

## Checklist-First Rule

Before writing any code for a feature:
1. Create a checklist in `checklists/[feature-name].md`
2. Present it to the user for review and approval
3. Code only begins after approval

No exceptions.

---

## SQL Rules

- All SQL saved to numbered files in `sql/` (e.g., `001-enable-uuid-extension.sql`)
- Claude NEVER executes SQL
- User runs SQL manually in the Supabase SQL Editor
- Migrations are append-only — never edit existing files
- Schema and RLS policy migrations are separate files
- Each file has a header comment explaining what it does and why

---

## Domain-First Architecture

See `docs/STRATEGIES.md` S5–S6.

**Dependency rules:**
- `features` → `domain` → `services` (allowed direction)
- `features` → `shared` (allowed)
- `features` MUST NOT import from other `features`
- `domain` MUST NOT import React (pure TypeScript only)

All Supabase queries go through `src/domain/[entity]/[entity]Repository.ts`.

---

## Optimistic Interaction Rule

User actions must NEVER wait for a network response.

UI updates immediately. Supabase write happens in the background.

Use client-generated UUIDs for all entities.

---

## Architecture Rules

**Banned without explicit user approval:**
- Redux / MobX / Zustand
- Microservices or external APIs beyond Supabase
- Complex in-memory caching
- GraphQL
- UI component libraries (use plain StyleSheet.create())

See `docs/STRATEGIES.md` S17.

---

## Session Close Obligations

Claude MUST do these at the end of every working session:

1. **Update `docs/PROJECT_STATE.md`** — current snapshot of all files, types, routes, SQL
2. **Append to `docs/DIARY.md`** — new entry following the diary format
3. **Update the active checklist** — mark completed tasks, note anything unfinished
4. **Update `checklists/DEV_CHECKLIST.md`** — mark phase status

A session is not complete until all four are done.

---

## PROJECT_STATE.md — Files by Phase

After every phase, add a "Files by Phase" entry:

```
### Phase N — [Phase Name]
src/features/example/ui/ExampleScreen.tsx   Purpose description
sql/013-example-table.sql                   Table schema
Modified: app/_layout.tsx                   What was changed
```

---

## UX Authority

The human founder owns product design.

Claude may:
✅ scaffold screens
✅ wire navigation
✅ build component structure
✅ suggest layout patterns

Claude may NOT:
❌ invent visual identity or branding
❌ choose color palettes without direction
❌ create complex design systems

Use placeholder styling unless directed otherwise.

---

## When in Doubt

Ask the user.
A 30-second clarifying question saves hours of wrong implementation.
