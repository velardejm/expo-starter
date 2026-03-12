# New Session Prompt
## Paste this at the start of every Claude Code session

---

## PASTE THIS INTO CLAUDE CODE:

```
We are resuming development on [APP_NAME].

Read these files IN ORDER before doing anything:
1. docs/CLAUDE.md
2. docs/specs.md
3. docs/STRATEGIES.md
4. docs/FEATURE_FLAGS.md
5. docs/PROJECT_STATE.md
6. docs/DIARY.md (last 2 entries only)

After reading, give me:
- One-line summary of where we left off
- Current phase and its status
- The next action item (if known)
- Any blockers or open questions

Do NOT write any code or create any files until I confirm the next task.
```

---

## NOTES

- Use this every time you open Claude Code — even if you were just in a session yesterday
- Claude Code has no memory between sessions; this is the fastest way to restore context
- PROJECT_STATE.md is the single most important file — if it's stale, update it before starting new work
- If you're on a different device, PROJECT_STATE.md + DIARY.md together give full context

---

## VARIANT: Starting a Specific Task

If you already know what you want to do:

```
We are resuming development on [APP_NAME].

Read: docs/CLAUDE.md, docs/PROJECT_STATE.md, docs/FEATURE_FLAGS.md
Also read: checklists/[phase-name].md

I want to work on: [describe task]

Before starting: confirm you've read the files and summarize the current task context.
Do NOT write code until I say go.
```

---

## VARIANT: Resuming After a Long Break (1+ week)

```
We are resuming development on [APP_NAME] after a break.

Read these files IN ORDER:
1. docs/CLAUDE.md
2. docs/specs.md
3. docs/STRATEGIES.md
4. docs/FEATURE_FLAGS.md
5. docs/PROJECT_STATE.md
6. docs/DIARY.md (all entries)

After reading, give me a full briefing:
- What has been built (phases complete)
- What the current state of the codebase is
- What was left unfinished
- Recommended next steps in priority order

Do NOT write any code until I confirm the plan.
```
