---
name: Builder file sync
description: Why Frontend/Backend Builder task files never auto-sync with the Manager's copy, and the rule that follows from it.
---

Frontend Builder and Backend Builder each work in their own separate Replit account, against their own GitHub checkout. When they extract the shared OTI docs, they get their OWN physical copies of `TASKS.md`, `FRONTEND_TASKS.md`/`BACKEND_TASKS.md`, etc. — these are separate files from the Manager's copies, not a shared/live document.

**Rule:** every time a task is confirmed done, or a new task is added, the Manager must send that Builder an explicit instruction telling them to update their own copy. Never assume a Builder's file is in sync just because the Manager's copy says so.

**Why:** Ahmad corrected this misunderstanding directly — the Manager had assumed marking a task done in its own file meant the Builder's file was also updated. It wasn't and isn't, automatically, ever.

**How to apply:** Before ending any Manager session, check every task that changed status or was newly added, and confirm a copy-paste instruction was sent (or is being sent) to the relevant Builder(s) telling them to update their own file.
