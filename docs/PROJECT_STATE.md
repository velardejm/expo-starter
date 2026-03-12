# PROJECT_STATE.md
# [APP_NAME]

> Session bootstrap document. Read this instead of re-reading source files.
> Claude MUST update this at the end of every working session.
> If this is stale, every future session pays the cost.
> Current as of: [DATE — Phase N completion]

---

## Current Phase

**Phase N — [Phase Name]: [STATUS]**

Next: **Phase N+1** — [description]

---

## Tech Stack

| Layer | Technology | Version |
|-------|-----------|---------|
| Framework | React Native (Expo) | SDK XX |
| Navigation | Expo Router | X.x |
| Language | TypeScript | X.x (strict mode) |
| Backend | Supabase | JS client vX |
| Local storage | AsyncStorage | — |
| Network detection | NetInfo | — |
| UUID generation | expo-crypto | — |
| [Add others] | | |

---

## Directory Structure

```
[Fill in as files are created — one line per file with purpose]

app/
  _layout.tsx              Root layout. [Providers] wrap <Stack>.
  index.tsx                Home screen.

src/
  features/
  domain/
  shared/
  services/
  config/
    featureFlags.ts        All flags = false

sql/
checklists/
docs/
```

---

## Feature Flags

```typescript
[FLAG_NAME]: false
[FLAG_NAME]: false
```

---

## Context Provider Hierarchy

```
1. [ProviderName] [ACTIVE]      — purpose, no deps
2. [ProviderName] [ACTIVE]      — purpose, depends on Provider 1
3. [ProviderName] [Phase N]     — enable with FLAG_NAME
```

---

## Domain Type Reference

```typescript
// [Entity] (src/domain/[entity]/[Entity].ts)
interface [Entity] { id, ... }

// [Entity] (src/domain/[entity]/[Entity].ts)
type [Status] = 'a' | 'b' | 'c'
interface [Entity] { id, status, ... }
```

---

## Key Functions

```typescript
// Repositories
import { [fn1], [fn2] } from '@/domain/[entity]/[entity]Repository';

// Feature hooks
import { use[Feature] } from '@/features/[feature]/model/use[Feature]';
```

---

## Database Schema (Current State)

### Tables

| Table | Status |
|-------|--------|
| `public.[table]` | ✓ Created + RLS |
| `public.[table]` | Pending |

### RLS Helper Functions
- `[function_name](param) → BOOLEAN` — SECURITY DEFINER ✓

### RLS Policy Matrix

| Table | SELECT | INSERT | UPDATE |
|-------|--------|--------|--------|
| [table] | [rule] | [rule] | [rule] |

---

## Navigation Routes

| Route | File | Status |
|-------|------|--------|
| `/` | `app/index.tsx` | Active |

---

## SQL Migrations

| # | File | Purpose | Phase | Status |
|---|------|---------|-------|--------|
| 001 | uuid-ossp extension | Enable UUID generation | Phase 1 | ✓ Run |

---

## Files by Phase

### Phase 1 — Infrastructure
```
[file path]    [purpose]
```

### Phase 2 — [Name]
```
[file path]    [purpose]
```

---

## Architectural Decisions

| Decision | Rationale |
|----------|-----------|
| [decision] | [why] |
