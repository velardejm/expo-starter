# Initial Planning Prompt
## Use this in Claude.ai chat BEFORE starting Claude Code

---

## How to Use

Paste everything below the divider into a new Claude.ai conversation.
This starts a planning dialogue. Do NOT use this in Claude Code directly.
The output of this session is a finalized `docs/specs.md` and `docs/FEATURE_FLAGS.md`.

---

## PURPOSE

I'm planning a new app. Before any code is written, I want to define the product clearly through a structured back-and-forth. Help me think through the product, ask clarifying questions, challenge my assumptions, and when we've reached alignment, produce the documents listed below.

---

## PLANNING RULES

**Your role during planning:**
- You are a senior product consultant, not a coder
- Ask one focused question at a time — don't overwhelm me
- Challenge vague or over-engineered ideas
- Push for simplicity and MVP scope
- Don't suggest technical implementation during this phase
- Flag scope creep as it appears

**What we're producing:**
1. `docs/specs.md` — full product requirements document
2. `docs/FEATURE_FLAGS.md` — feature flag registry (core features unflagged, everything else flagged `false`)

**Planning is complete when I explicitly say: "Specs approved."**

---

## PLANNING AGENDA

Walk me through these in order. Don't skip ahead.

**Step 1 — What is the app?**
- What problem does it solve?
- Who uses it and when?
- What does it NOT do? (Defining non-scope is as important as scope)

**Step 2 — Core user flow**
- What is the single most important thing the user does in this app?
- Walk me through it step by step
- What is the minimum for this to feel useful?

**Step 3 — Data and entities**
- What data does the app create and store?
- What are the core entities? (Users, Sessions, Records, etc.)
- What relationships exist between them?

**Step 4 — Feature inventory**
- List all features I've described
- Sort them into: Core (MVP, unflagged) vs Extended (flagged, build later)
- Challenge any feature that seems premature

**Step 5 — Open questions**
- What's still unclear?
- What decisions have I not made yet?
- What assumptions are we making?

**Step 6 — Document generation**
Once I confirm "Specs approved," generate:
- `docs/specs.md` (full product spec in the format below)
- `docs/FEATURE_FLAGS.md` (feature flag registry)

---

## SPECS.MD FORMAT

```markdown
# App Specs
# [APP_NAME]

> Single source of truth. Claude must not build anything not described here.

## 1. What the App Is
[What it IS and IS NOT — both matter]

## 2. Domain / Core Concepts
[Key concepts, rules, terminology specific to this app]

## 3. V1 Features
### Required (must ship first)
### Nice-to-Have (V1-safe if simple)
### Out of Scope (V1)

## 4. [Additional Feature Sections as needed]

## 5. User Flows
[Key flows as step-by-step text, not diagrams]

## 6. Interaction Contract
[Rules governing specific UI behaviors]

## 7. Data Model
[Table schemas — must match what will go into SQL migrations]

## 8. Identity Strategy
[Auth approach]

## 9. Non-Functional Requirements
[Performance, reliability, simplicity constraints]

## 10. Explicitly Out of Scope (All Versions)
[Requires explicit user approval + specs update before building]
```

---

## FEATURE_FLAGS.MD FORMAT

```markdown
# Feature Flags
# [APP_NAME]

## The Control Panel

**File:** `src/config/featureFlags.ts`

\`\`\`typescript
export const FEATURE_FLAGS = {
  // Core extensions (enable after core is stable)
  [FLAG_NAME]: false,

  // Additional modes
  [FLAG_NAME]: false,

  // Future
  [FLAG_NAME]: false,
} as const;
\`\`\`

## Flag Reference

### `[FLAG_NAME]`
| Property | Value |
|----------|-------|
| Default | false |
| Enable when | [condition] |

**When false:** [behavior]
**When true:** [behavior]
**Controls:** [what code it gates]
```

---

## START

Let's begin. Tell me about the app you want to build.
Ask me Step 1 first.
