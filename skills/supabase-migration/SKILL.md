---
name: supabase-migration
description: SQL migration file standards for Supabase. Use this skill whenever writing any SQL — schema creation, RLS policies, helper functions, or seed data. Enforces consistent file naming, headers, policy naming, and the correct order of operations.
---

## The Rules

1. **One concern per file** — schema, RLS, and helper functions are always separate files
2. **Numbered sequentially** — `001`, `002`, `003`… never skip, never reuse
3. **Append-only** — never edit a file that has already been run
4. **User runs manually** — Claude never executes SQL
5. **Header on every file** — no bare SQL files

---

## File Naming Convention

```
001-enable-uuid-extension.sql
002-create-[table]-table.sql
003-[table]-rls-policies.sql
004-[table]-helper-functions.sql
005-create-[table2]-table.sql
006-[table2]-rls-policies.sql
```

**Pattern:** schema first, RLS immediately after, helpers before the RLS files that need them.

---

## Required File Header

Every SQL file must start with this exact header:

```sql
-- ============================================================
-- Migration: NNN-description.sql
-- Project:   [APP_NAME]
-- Phase:     Phase N — [Phase Name]
-- Purpose:   [One sentence — what this migration does]
-- Depends:   [Which previous migrations must be run first, or "None"]
-- Run:       Paste into Supabase Dashboard → SQL Editor → Run
-- ============================================================
```

---

## Schema File Template

```sql
-- ============================================================
-- Migration: 002-create-users-table.sql
-- Project:   [APP_NAME]
-- Phase:     Phase 2 — Auth & Storage Foundation
-- Purpose:   Creates the public.users table for user identity
-- Depends:   001-enable-uuid-extension.sql
-- Run:       Paste into Supabase Dashboard → SQL Editor → Run
-- ============================================================

CREATE TABLE IF NOT EXISTS public.users (
  id              UUID PRIMARY KEY,           -- matches auth.uid()
  display_name    TEXT,
  is_anonymous    BOOLEAN NOT NULL DEFAULT TRUE,
  created_at      TIMESTAMPTZ NOT NULL DEFAULT NOW(),
  updated_at      TIMESTAMPTZ
);

-- Index for common query patterns
CREATE INDEX IF NOT EXISTS users_created_at_idx ON public.users(created_at DESC);
```

---

## RLS Policies File Template

```sql
-- ============================================================
-- Migration: 003-users-rls-policies.sql
-- Project:   [APP_NAME]
-- Phase:     Phase 2 — Auth & Storage Foundation
-- Purpose:   Row-level security policies for public.users
-- Depends:   002-create-users-table.sql (must be run first)
-- Run:       Paste into Supabase Dashboard → SQL Editor → Run
-- ============================================================

-- Enable RLS (always the first line in an RLS file)
ALTER TABLE public.users ENABLE ROW LEVEL SECURITY;

-- SELECT: user can read their own row only
CREATE POLICY "users_select_own"
  ON public.users
  FOR SELECT
  USING (id = auth.uid());

-- INSERT: user can insert their own row only
CREATE POLICY "users_insert_own"
  ON public.users
  FOR INSERT
  WITH CHECK (id = auth.uid());

-- UPDATE: user can update their own row only
CREATE POLICY "users_update_own"
  ON public.users
  FOR UPDATE
  USING (id = auth.uid());

-- Note: No DELETE policy in V1 — users are not deleted
```

---

## RLS Policy Naming Convention

```
[table]_[action]_[rule]

Examples:
  users_select_own
  users_insert_own
  sessions_select_participant
  sessions_insert_host
  battles_update_host
  posts_select_public
```

**Actions:** `select`, `insert`, `update`, `delete`
**Rules:** `own`, `host`, `participant`, `member`, `public`, `admin`

---

## SECURITY DEFINER Helper Functions Template

Use when a policy on `table_a` needs to query `table_b` (avoids RLS recursion).

```sql
-- ============================================================
-- Migration: 009-session-rls-helper-functions.sql
-- Project:   [APP_NAME]
-- Phase:     Phase 3 — Domain Layer
-- Purpose:   SECURITY DEFINER helpers to avoid RLS recursion
--            in session and battle policies
-- Depends:   006-create-sessions-table.sql,
--            007-create-session-participants-table.sql
-- Run:       Paste into Supabase Dashboard → SQL Editor → Run
-- ============================================================

-- is_session_participant
-- Returns TRUE if the current user is a participant in the given session.
-- Used by: sessions SELECT, battles SELECT/INSERT/UPDATE
-- SECURITY DEFINER: bypasses RLS on session_participants to avoid recursion
CREATE OR REPLACE FUNCTION public.is_session_participant(p_session_id UUID)
RETURNS BOOLEAN
LANGUAGE sql
SECURITY DEFINER
SET search_path = public
AS $$
  SELECT EXISTS (
    SELECT 1
    FROM public.session_participants
    WHERE session_id = p_session_id
      AND user_id = auth.uid()
  );
$$;

-- is_session_host
-- Returns TRUE if the current user is the host of the given session.
-- Used by: sessions INSERT/UPDATE, battles INSERT/UPDATE
CREATE OR REPLACE FUNCTION public.is_session_host(p_session_id UUID)
RETURNS BOOLEAN
LANGUAGE sql
SECURITY DEFINER
SET search_path = public
AS $$
  SELECT EXISTS (
    SELECT 1
    FROM public.sessions
    WHERE id = p_session_id
      AND host_user_id = auth.uid()
  );
$$;
```

---

## Common Schema Patterns

### Standard columns (include on every table)
```sql
id          UUID PRIMARY KEY,           -- client-generated
created_at  TIMESTAMPTZ NOT NULL DEFAULT NOW(),
updated_at  TIMESTAMPTZ                 -- nullable; set on update
```

### Foreign key with cascade
```sql
session_id  UUID NOT NULL REFERENCES public.sessions(id) ON DELETE CASCADE
```

### Status column with constraint
```sql
status  TEXT NOT NULL DEFAULT 'pending'
        CHECK (status IN ('pending', 'active', 'ended'))
```

### Composite primary key (join tables)
```sql
PRIMARY KEY (session_id, user_id)
```

### Unique constraint
```sql
UNIQUE (session_id, sequence_number)
```

---

## Migration Order Rules

Always run in this order for a new entity:

1. Schema file (create table)
2. Any SECURITY DEFINER helpers needed by RLS
3. RLS policies file

Never run RLS before schema. Never run policies that reference helpers before the helpers.

---

## UUID Extension (always Migration 001)

```sql
-- ============================================================
-- Migration: 001-enable-uuid-extension.sql
-- Project:   [APP_NAME]
-- Phase:     Phase 1 — Infrastructure
-- Purpose:   Enable the uuid-ossp extension for UUID generation
-- Depends:   None
-- Run:       Paste into Supabase Dashboard → SQL Editor → Run
-- ============================================================

CREATE EXTENSION IF NOT EXISTS "uuid-ossp";
```
