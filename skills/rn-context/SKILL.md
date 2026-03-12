---
name: rn-context
description: React Context pattern for React Native (Expo) apps. Use this skill whenever creating shared state for a feature — context file, hook, provider, and barrel export. Enforces the consistent 4-file pattern used across all features.
---

## The Rule

**Every feature that needs shared state gets the same 4-file structure.**
No Zustand, Redux, or MobX. React Context is sufficient.

---

## The 4-File Pattern

```
src/features/[feature]/
  model/
    [Feature]Context.ts       ← createContext (no logic)
    use[Feature].ts           ← state logic + consumer hook
    [Feature]Provider.tsx     ← JSX provider wrapper
  index.ts                    ← barrel export
```

---

## File 1: `[Feature]Context.ts`

Context definition only. No logic here.

```typescript
// src/features/[feature]/model/[Feature]Context.ts

import { createContext } from 'react';

// 1. Define the context value shape
export interface [Feature]ContextValue {
  // State
  items: [Item][];
  isLoading: boolean;
  error: string | null;

  // Actions
  add[Item]: (item: [Item]) => void;
  update[Item]: (id: string, updates: Partial<[Item]>) => void;
  remove[Item]: (id: string) => void;
}

// 2. Create the context with null default
//    null means "not inside a provider" — caught by the consumer hook
export const [Feature]Context = createContext<[Feature]ContextValue | null>(null);
```

---

## File 2: `use[Feature].ts`

Two exports: the state logic hook (used by the provider) and the consumer hook (used by components).

```typescript
// src/features/[feature]/model/use[Feature].ts

import { useState, useEffect, useContext, useCallback } from 'react';
import * as Crypto from 'expo-crypto';
import { [Feature]Context, type [Feature]ContextValue } from './[Feature]Context';
import { useUser } from '@/features/auth/model/UserProvider';
import {
  get[Item]sByUserId,
  create[Item],
  update[Item],
  delete[Item],
} from '@/domain/[entity]/[entity]Repository';
import type { [Item] } from '@/domain/[entity]/[Entity]';

// ─── Provider-side hook (used only by [Feature]Provider) ─────────────────────

export function use[Feature]State(): [Feature]ContextValue {
  const { user } = useUser();

  const [items, setItems] = useState<[Item][]>([]);
  const [isLoading, setIsLoading] = useState(true);
  const [error, setError] = useState<string | null>(null);

  // Load on mount
  useEffect(() => {
    if (!user) return;

    async function load() {
      try {
        setIsLoading(true);
        const data = await get[Item]sByUserId(user.id);
        setItems(data);
      } catch (err) {
        setError('Failed to load [items].');
      } finally {
        setIsLoading(false);
      }
    }

    load();
  }, [user?.id]);

  // Optimistic add
  const add[Item] = useCallback((item: [Item]) => {
    setItems(prev => [item, ...prev]);                   // optimistic
    create[Item](item).catch(() => {
      setItems(prev => prev.filter(i => i.id !== item.id)); // rollback on failure
    });
  }, []);

  // Optimistic update
  const update[Item] = useCallback((id: string, updates: Partial<[Item]>) => {
    setItems(prev =>
      prev.map(i => i.id === id ? { ...i, ...updates } : i)
    );
    update[Item](id, updates).catch(() => {
      // Optionally: re-fetch to restore correct state
    });
  }, []);

  // Optimistic remove
  const remove[Item] = useCallback((id: string) => {
    setItems(prev => prev.filter(i => i.id !== id));     // optimistic
    delete[Item](id).catch(() => {
      // Optionally: re-fetch to restore correct state
    });
  }, []);

  return {
    items,
    isLoading,
    error,
    add[Item],
    update[Item],
    remove[Item],
  };
}

// ─── Consumer hook (used by feature components) ───────────────────────────────

export function use[Feature]Context(): [Feature]ContextValue {
  const context = useContext([Feature]Context);
  if (!context) {
    throw new Error('use[Feature]Context must be used within [Feature]Provider');
  }
  return context;
}
```

---

## File 3: `[Feature]Provider.tsx`

Thin JSX wrapper. Calls the state hook, provides the value. No logic here.

```typescript
// src/features/[feature]/model/[Feature]Provider.tsx

import React from 'react';
import { [Feature]Context } from './[Feature]Context';
import { use[Feature]State } from './use[Feature]';

interface [Feature]ProviderProps {
  children: React.ReactNode;
}

export function [Feature]Provider({ children }: [Feature]ProviderProps) {
  const value = use[Feature]State();

  return (
    <[Feature]Context.Provider value={value}>
      {children}
    </[Feature]Context.Provider>
  );
}
```

---

## File 4: `index.ts` (barrel)

Exports only what other parts of the app need.

```typescript
// src/features/[feature]/index.ts

export { [Feature]Provider } from './model/[Feature]Provider';
export { use[Feature]Context } from './model/use[Feature]';
export type { [Feature]ContextValue } from './model/[Feature]Context';
```

---

## Registering the Provider in `app/_layout.tsx`

```typescript
// app/_layout.tsx
// Add provider in dependency order — outer = no deps, inner = has deps

import { [Feature]Provider } from '@/features/[feature]';

export default function RootLayout() {
  return (
    <UserProvider>
      {/* [Feature]Provider depends on UserProvider */}
      <[Feature]Provider>
        <Stack screenOptions={{ headerShown: false }} />
      </[Feature]Provider>
    </UserProvider>
  );
}

// Update the provider order comment:
// 1. UserProvider         [ACTIVE] — identity, no deps
// 2. [Feature]Provider    [ACTIVE] — depends on UserProvider
```

---

## Consuming in a Component

```typescript
import { use[Feature]Context } from '@/features/[feature]';

export function [Feature]Screen() {
  const { items, isLoading, add[Item] } = use[Feature]Context();

  // use the state and actions
}
```

---

## Rules

1. **4 files per feature context** — Context, hook, provider, barrel. Never combine them.
2. **`createContext(null)`** — never provide a fake default value. The consumer hook catches the null case.
3. **State logic in `use[Feature]State()`** — not in the provider component. Provider is JSX only.
4. **Consumer hook throws** if used outside the provider — makes bugs immediately obvious.
5. **Optimistic UI always** — update local state first, Supabase write in background.
6. **Rollback on failure** — catch the background write and revert state if it fails.
7. **`useCallback` on all action functions** — prevents unnecessary re-renders.
8. **Providers registered in `app/_layout.tsx`** in dependency order, with comments.
9. **Only export from `index.ts`** what other features actually need. Keep internals internal.
10. **One context per feature.** If a feature grows too large, split into sub-features, not more contexts.
