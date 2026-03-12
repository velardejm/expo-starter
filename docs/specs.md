# App Specs
# [APP_NAME]

> This is the SINGLE SOURCE OF TRUTH for all product requirements.
> Claude must not invent features, flows, or mechanics not described here.
> If any requirement is unclear: ask the user. Never guess.

---

## 1. What the App Is

**[APP_NAME] is [one-sentence description].**

[2-3 sentences expanding on the core purpose and context of use.]

**The app IS:**
- [Core capability 1]
- [Core capability 2]

**The app is NOT:**
- [Anti-goal 1 — defining non-scope is as important as scope]
- [Anti-goal 2]

---

## 2. Domain / Core Concepts

[Key concepts, rules, and terminology specific to this app's domain.
This section grounds Claude in the real-world context before any technical details.]

### [Concept 1]
[Description]

### [Concept 2]
[Description]

---

## 3. V1 Features

### Required (must ship before anything else)
- [Feature 1 — core, unflagged]
- [Feature 2]

### Nice-to-Have (V1-safe if implementation is simple)
- [Feature — include only if it doesn't complicate the core flow]

### Out of Scope (V1)
- [Feature — with brief reason]
- [Feature]

---

## 4. [Additional Feature Section — e.g., V2 Feature Name]

> Behind `[FLAG_NAME]` flag. Build after [prerequisite] is stable.

[Description of the feature, its rules, and how it relates to V1.]

---

## 5. User Flows

### 5.1 [Primary Flow Name]

```
[Actor]: [action]
→ [result]
→ [next step]

[Actor]: [action]
→ [result]
```

### 5.2 [Secondary Flow Name]

```
[Steps]
```

---

## 6. Interaction Contract

### [Interaction Rule 1 — e.g., "Correction Window"]
[Specific rules governing this UI behavior]

### [Interaction Rule 2]
[Rules]

---

## 7. Data Model

### `[table_name]`
```sql
id              UUID PRIMARY KEY        -- client-generated
[column]        [TYPE] [CONSTRAINTS]
created_at      TIMESTAMPTZ DEFAULT NOW()
```

### `[table_name]`
```sql
id              UUID PRIMARY KEY
[column]        [TYPE] [CONSTRAINTS]
```

---

## 8. Identity Strategy

[How users are identified. Anonymous-first? Email? Social auth?
Include: how account creation is deferred, how data is preserved on upgrade.]

---

## 9. Non-Functional Requirements

### Performance
- [Specific performance requirement — e.g., "action X must feel instant"]

### Reliability
- [e.g., "Never lose [data type]"]

### Simplicity
- Prefer boring, stable technology
- Avoid premature scaling

---

## 10. Explicitly Out of Scope (All Versions)

These require explicit user approval + a specs update before being built:

- [Feature — with reason]
- [Feature]
