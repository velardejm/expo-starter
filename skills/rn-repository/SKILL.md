---
name: rn-repository
description: Repository and mapper boilerplate for React Native + Supabase apps. Use this skill whenever creating a domain entity — repository, mapper, or entity type file. Enforces consistent data access patterns, error handling, and DTO transformation.
---

## The Pattern

```
Feature hook → Repository function → Supabase → Mapper → Domain type
```

Features never call `supabase.from()` directly. Always through the repository.

---

## File: `src/domain/[entity]/[Entity].ts`

```typescript
// src/domain/[entity]/[Entity].ts
// Domain entity definition.
// Pure TypeScript — no React, no Supabase imports.

export type [Entity]Status = 'active' | 'inactive';  // example status union

export interface [Entity] {
  id: string;                    // UUID — client-generated
  userId: string;                // FK reference — always string
  [field]: [type];
  status: [Entity]Status;
  createdAt: string;             // ISO 8601 — never Date
  updatedAt: string | null;      // nullable timestamps use `| null`
}

// Derived utility types
export type [Entity]Preview = Pick<[Entity], 'id' | '[field]'>;
export type Create[Entity] = Omit<[Entity], 'createdAt' | 'updatedAt'>;
export type Update[Entity] = Partial<Pick<[Entity], '[field]' | 'status'>>;
```

---

## File: `src/domain/[entity]/[entity]Mappers.ts`

```typescript
// src/domain/[entity]/[entity]Mappers.ts
// Transforms raw Supabase rows (DTOs) into typed domain models.
// Pure TypeScript — no React, no Supabase imports.

import type { [Entity] } from './[Entity]';

// The raw shape coming from Supabase (snake_case)
type [Entity]DTO = {
  id: string;
  user_id: string;
  [field_name]: [type];
  status: string;
  created_at: string;
  updated_at: string | null;
};

export function map[Entity]FromDTO(dto: unknown): [Entity] {
  // Validate the DTO has the minimum required fields
  if (!dto || typeof dto !== 'object') {
    throw new Error('[Entity] mapper: received non-object DTO');
  }

  const row = dto as [Entity]DTO;

  if (!row.id) throw new Error('[Entity] mapper: missing id');
  if (!row.user_id) throw new Error('[Entity] mapper: missing user_id');

  return {
    id: row.id,
    userId: row.user_id,
    [field]: row.[field_name],
    status: row.status as [Entity]Status,
    createdAt: row.created_at,
    updatedAt: row.updated_at,
  };
}

// For mapping arrays (use in getAll functions)
export function map[Entity]ListFromDTO(dtos: unknown[]): [Entity][] {
  return dtos.map(map[Entity]FromDTO);
}
```

---

## File: `src/domain/[entity]/[entity]Repository.ts`

```typescript
// src/domain/[entity]/[entity]Repository.ts
// ALL Supabase access for this entity lives here.
// This is the ONLY file that calls supabase.from('[entity_table]').
// Features call these functions — never supabase directly.

import { supabase } from '@/services/supabase';
import { map[Entity]FromDTO, map[Entity]ListFromDTO } from './[entity]Mappers';
import type { [Entity], Create[Entity], Update[Entity] } from './[Entity]';

// ─── READ ────────────────────────────────────────────────────────────────────

export async function get[Entity]ById(id: string): Promise<[Entity] | null> {
  const { data, error } = await supabase
    .from('[entity_table]')
    .select('*')
    .eq('id', id)
    .maybeSingle();   // maybeSingle returns null if not found; single() throws

  if (error) throw error;
  if (!data) return null;
  return map[Entity]FromDTO(data);
}

export async function get[Entity]sByUserId(userId: string): Promise<[Entity][]> {
  const { data, error } = await supabase
    .from('[entity_table]')
    .select('*')
    .eq('user_id', userId)
    .order('created_at', { ascending: false });

  if (error) throw error;
  return map[Entity]ListFromDTO(data ?? []);
}

// ─── WRITE ───────────────────────────────────────────────────────────────────

export async function create[Entity](entity: Create[Entity]): Promise<void> {
  const { error } = await supabase
    .from('[entity_table]')
    .insert({
      id: entity.id,
      user_id: entity.userId,
      [field_name]: entity.[field],
      status: entity.status,
      // created_at handled by DB default
    });

  if (error) throw error;
}

export async function update[Entity](
  id: string,
  updates: Update[Entity]
): Promise<void> {
  const payload: Record<string, unknown> = {};

  // Only include fields that are actually being updated
  if (updates.[field] !== undefined) payload.[field_name] = updates.[field];
  if (updates.status !== undefined) payload.status = updates.status;
  payload.updated_at = new Date().toISOString();

  const { error } = await supabase
    .from('[entity_table]')
    .update(payload)
    .eq('id', id);

  if (error) throw error;
}

export async function delete[Entity](id: string): Promise<void> {
  const { error } = await supabase
    .from('[entity_table]')
    .delete()
    .eq('id', id);

  if (error) throw error;
}

// ─── UPSERT (use for sync / optimistic reconciliation) ───────────────────────

export async function upsert[Entity](entity: [Entity]): Promise<void> {
  const { error } = await supabase
    .from('[entity_table]')
    .upsert({
      id: entity.id,
      user_id: entity.userId,
      [field_name]: entity.[field],
      status: entity.status,
    });

  if (error) throw error;
}
```

---

## Rules

1. **`maybeSingle()` not `single()`** for queries that might return no rows. `single()` throws on no result.
2. **`data ?? []`** for list queries — never assume data is non-null.
3. **Only include fields in `update` payload that are actually changing** — use a `Record` builder pattern.
4. **Timestamps are set by the DB default** (`NOW()`) on insert — don't pass `created_at` from the client.
5. **`updated_at` is set client-side** as `new Date().toISOString()` on update.
6. **Repository functions never return raw Supabase rows** — always mapped domain types.
7. **Client-generated UUIDs** — the `id` is always passed in by the caller, never generated in the repository.
8. **One repository file per entity** — never combine two entities in one repository.
9. **No React imports** in repositories, mappers, or entity files.
10. **Error propagation:** `if (error) throw error` — let the feature layer handle display.

---

## UUID Generation (in feature hooks, not repositories)

```typescript
import * as Crypto from 'expo-crypto';

const newId = Crypto.randomUUID();
```

Always generated by the caller before passing to the repository.

---

## Barrel Export

Each domain entity folder needs an `index.ts`:

```typescript
// src/domain/[entity]/index.ts
export type { [Entity], [Entity]Status, Create[Entity], Update[Entity] } from './[Entity]';
export { get[Entity]ById, get[Entity]sByUserId, create[Entity], update[Entity], delete[Entity] } from './[entity]Repository';
```
