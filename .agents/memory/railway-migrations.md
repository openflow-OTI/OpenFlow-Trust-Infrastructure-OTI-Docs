---
name: Railway Migration Gap
description: Railway deploy does NOT auto-run drizzle-kit push — every schema change needs a manual prod DB migration by Ahmad.
---

## Rule
Railway's deploy pipeline only runs `pnpm install && build && start`. It does NOT run `drizzle-kit push` or any migration step. Every new column or table added by the Backend Builder will leave Railway production out of sync until Ahmad manually runs the migration.

**Why:** Confirmed July 5, 2026 — Task 6 code deployed cleanly but `GET /api/admin/keys` returned 500 because the `updated_at` column existed in dev DB but not in Railway prod DB.

**How to apply:**
- After every Backend Builder PR that includes a schema change, remind Ahmad to run `drizzle-kit push` against Railway prod via the Railway Shell tab.
- Optional one-line fix: add `drizzle-kit push` to `railway.json` buildCommand (NOT nixpacks.toml — that is sacred).
- The Railway Shell tab is the easiest path for Ahmad — no local setup needed.
