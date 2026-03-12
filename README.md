# App Development Template
## JM's Claude Code Workflow

This template defines the full development workflow for any new mobile/web app built with Claude Code.
Copy this entire folder into a new project and replace `[APP_NAME]` placeholders.

---

## Template Structure

```
/                         ← Project root
  CLAUDE.md               ← Auto-read by Claude Code each session (root guardrail)
  docs/
    CLAUDE.md             ← Full AI rules, architecture, scope control
    STRATEGIES.md         ← All development strategies (copy as-is, project-agnostic)
    FEATURE_FLAGS.md      ← Feature flag registry for this project
    specs.md              ← Product requirements (fill in during planning phase)
    PROJECT_STATE.md      ← Session bootstrap — updated after every session
    DIARY.md              ← Project diary — append-only log per phase
    initial-prompt.md     ← Paste this to start Claude Code on a fresh project
  checklists/
    DEV_CHECKLIST.md      ← Master phase tracker (all phases, one glance)
    [phase-n-name].md     ← Per-phase checklist (created before each phase)
  sql/
    NNN-description.sql   ← Numbered migration files (append-only)
  scripts/
    new-session.md        ← Paste this at the start of every new Claude Code session
```

---

## The Three Prompts

| Prompt | When to use | File |
|--------|-------------|------|
| **Planning Prompt** | Starting a brand new project | `docs/initial-prompt.md` → Planning Mode |
| **Start Session Prompt** | Opening Claude Code on an existing project | `scripts/new-session.md` |
| **New Phase Prompt** | Beginning work on a new feature/phase | Generated from checklist |

---

## Quick Start for a New Project

1. Copy this template folder into your new project repo
2. Run the **Planning Prompt** in Claude.ai (not Claude Code) to define `docs/specs.md`
3. Once specs are finalized, open Claude Code and paste `docs/initial-prompt.md`
4. Claude Code will create Phase 1 checklist — review and approve before anything is built
5. After each phase: Claude updates `PROJECT_STATE.md` and `DIARY.md` before closing

---

## Core Principles (Non-Negotiable)

- **Checklist-first:** No code until a checklist is written and approved
- **Specs as truth:** Nothing built that isn't in `docs/specs.md`
- **Feature flags:** Every non-core feature ships behind a `false` flag
- **SQL files only:** Claude never executes SQL — you run it manually
- **PROJECT_STATE.md always current:** Claude updates it at end of every session
- **DIARY.md always appended:** Never edited retroactively, only appended
- **Modularity:** Domain → Repository → Feature → UI dependency chain, never reversed
