---
name: rn-types
description: Centralized TypeScript type system for React Native (Expo) apps. Use this skill whenever defining, modifying, or importing any TypeScript types, interfaces, or unions. Types must follow the single-definition rule — defined once, imported everywhere.
---

## The Rule

**Every type is defined exactly once. If you need a type that already exists, import it — never redefine it.**
No `any`. Use `unknown` with type guards where input is truly dynamic.

---

## Type Location Map

| Type category | Where it lives |
|---------------|---------------|
| Domain entities (User, Session, Build…) | `src/domain/[entity]/[Entity].ts` |
| Status/union types for an entity | Same file as the entity |
| Feature-specific UI state | `src/features/[feature]/types.ts` |
| Shared UI prop types (used by 2+ features) | `src/shared/types.ts` |
| Navigation param types | `src/shared/navigation.ts` |
| App-wide constants as types | `src/config/constants.ts` |
| API response shapes (DTOs) | `src/domain/[entity]/[entity]Mappers.ts` (internal only) |

---

## Domain Entity Pattern

```typescript
// src/domain/user/User.ts

// Status and union types first
export type UserRole = 'host' | 'participant' | 'guest';

// Main interface
export interface User {
  id: string;           // UUID — always string, never number
  displayName: string;
  role: UserRole;
  isAnonymous: boolean;
  createdAt: string;    // ISO 8601 timestamp string — never Date object
}

// Derived / utility types from the entity
export type UserPreview = Pick<User, 'id' | 'displayName'>;
export type UserUpdate = Partial<Pick<User, 'displayName'>>;
```

---

## Shared Types File

```typescript
// src/shared/types.ts
// Only types used by 2+ features belong here.
// Feature-specific types stay in their feature folder.

// Common async state wrapper
export type AsyncState<T> =
  | { status: 'idle' }
  | { status: 'loading' }
  | { status: 'success'; data: T }
  | { status: 'error'; error: string };

// Common list state
export type ListState<T> = {
  items: T[];
  isLoading: boolean;
  error: string | null;
};

// Common form field state
export type FieldState = {
  value: string;
  error: string | null;
  touched: boolean;
};
```

---

## Navigation Types

```typescript
// src/shared/navigation.ts
// Centralizes all route param types.
// Import this in screens that use useLocalSearchParams or router.push with params.

export type RootTabParamList = {
  index: undefined;
  explore: undefined;
  profile: undefined;
};

export type StackParamList = {
  '[id]': { id: string };
  'create': undefined;
};
```

---

## Feature Types Pattern

```typescript
// src/features/session/types.ts
// Types specific to this feature only.
// If another feature needs one of these, move it to src/shared/types.ts.

import type { Side } from '@/domain/session/Session';

export type BattleLogEntry = {
  battleId: string;
  winnerSide: Side;
  timestamp: string;
};

export type SessionScreenState =
  | 'waiting'
  | 'build-selection'
  | 'active'
  | 'ending'
  | 'ended';
```

---

## Rules

1. **No `any`** — use `unknown` with a type guard if input is truly dynamic:
   ```typescript
   function isUser(value: unknown): value is User {
     return (
       typeof value === 'object' &&
       value !== null &&
       'id' in value &&
       'displayName' in value
     );
   }
   ```

2. **String literal unions over enums:**
   ```typescript
   // ✅ Correct
   type Status = 'pending' | 'active' | 'ended';

   // ❌ Wrong
   enum Status { Pending, Active, Ended }
   ```

3. **Timestamps as strings**, not `Date` objects:
   ```typescript
   createdAt: string;   // ✅ ISO 8601: "2025-01-15T10:30:00Z"
   createdAt: Date;     // ❌ Never — causes serialization issues
   ```

4. **IDs always as `string`** — never `number`, even if the DB uses integers.

5. **`as const` for config objects:**
   ```typescript
   export const STORAGE_KEYS = {
     USER_ID: 'user_id',
     DISPLAY_NAME: 'display_name',
   } as const;

   export type StorageKey = typeof STORAGE_KEYS[keyof typeof STORAGE_KEYS];
   ```

6. **No re-export of types from barrel files unless they are part of the feature's public API.** Internal types stay internal.

7. **`interface` for domain entities. `type` for unions, mapped types, and utilities.**

8. **`Pick`, `Omit`, `Partial` over duplicating fields:**
   ```typescript
   // ✅ Derive from existing type
   type CreateBuild = Omit<Build, 'id' | 'createdAt'>;

   // ❌ Never duplicate fields manually
   type CreateBuild = { blade: string; ratchet: string; bit: string; };
   ```

---

## Type Checking

Run before marking any task complete:
```bash
npx tsc --noEmit
```

Zero errors required. Never suppress with `// @ts-ignore` without user approval.
