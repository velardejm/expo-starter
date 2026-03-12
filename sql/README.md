# SQL Migrations — [APP_NAME]

> All SQL files in this folder are append-only migration files.
> Never edit an existing file after it has been run.
> Files are numbered sequentially: NNN-description.sql
> You run these manually in the Supabase SQL Editor.
> Claude never executes SQL.

---

## Running Order

Run migrations in numbered order. Never skip. Never re-run unless noted.

| File | Description | Status |
|------|-------------|--------|
| 001-enable-uuid-extension.sql | Enable uuid-ossp | ⬜ |

---

## SQL File Header Template

Every SQL file must start with this header:

```sql
-- ============================================================
-- Migration: NNN-description.sql
-- Project:   [APP_NAME]
-- Phase:     Phase N — [Phase Name]
-- Purpose:   [One sentence — what this migration does]
-- Depends:   [Which previous migrations must be run first, or "None"]
-- Run:       Paste into Supabase SQL Editor → Run
-- ============================================================
```

---

## Naming Convention

```
001-enable-uuid-extension.sql
002-create-[table]-table.sql
003-[table]-rls-policies.sql
004-[function]-helper-functions.sql
```

Schema and RLS are always in separate files.
Helper functions are separate from schema.
