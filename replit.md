# OTI — Manager Workspace

This is the **Development Manager's documentation workspace** for OTI (OpenFlow Trust Infrastructure). No OTI source code lives here. This workspace is used for:

- Writing task prompts for Builders
- Maintaining project documentation
- Roadmap and decision tracking

## How to start a new Manager session

Read `docs/MANAGER_HANDOVER.md` first — always. Then read the other docs in the order listed in `docs/READING_GUIDE.md`.

## Workspace structure

```
docs/
  MANAGER_HANDOVER.md       — project state, what to do next (start here)
  ARCHITECTURE.md           — system components and how they connect
  ROADMAP.md                — all phases, done/next/future
  TOKENOMICS.md             — OTI Economics: 35M supply, DCS + ERP + 3-contract genesis (CANONICAL — August 1, 2026)
  BUSINESS_MODEL.md         — revenue model, network effect engine
  TASKS.md                  — master task list
  FIXES.md                  — bug fix log (BF41 open — Sui broken)
  DECISIONS.md              — why things exist the way they do
  BACKEND_TASKS.md          — Backend Builder's task prompt copy
  FRONTEND_TASKS.md         — Frontend Builder's task prompt copy
  BACKEND_BUILDER_ONBOARDING_PROMPT.md
  FRONTEND_BUILDER_ONBOARDING_PROMPT.md
  READING_GUIDE.md          — reading order guide
  whitepaper-additions-draft.md — whitepaper additions (merge into Task 25)
```

## Key facts

- **Live frontend:** https://otiscore.vercel.app
- **Live backend:** https://workspaceapi-server-production-5c0c.up.railway.app
- **Live docs:** https://otiscore.vercel.app/docs/
- **Task 28 status:** Parts A/B/C LIVE. Part D (smart contracts) IN PROGRESS — BLOCKED pending Ahmad's confirmation of expanded 3-contract scope.
- **Next task to assign:** Task 25 (Frontend — after Task 28 all parts confirmed done)
- **Next fix numbers:** BF42 (backend), FF28 (frontend)
- **Next task number:** Task 29

## CRITICAL BLOCKER (August 2, 2026)

TOKENOMICS.md (Aug 1) describes 3-contract genesis + DEX Liquidity Engine + dynamic unlock formula.
Backend Builder is working from outdated July 30 spec (2 contracts, fixed 25/75).
**Do NOT let Builder deploy to mainnet until Ahmad confirms the expanded Part D scope.**
See SESSION 31 HANDOVER in MANAGER_HANDOVER.md for the 3 questions to send Ahmad.

## User preferences

- Ahmad is the CEO — call him Ahmad, never "sir" or "boss"
- Every reply to Ahmad goes in a copy box so he can paste it to Builders
- One task per Builder at a time — hard, non-negotiable rule
- Fixes never get task numbers — FIXES.md only
- Builders do not push to GitHub — Ahmad only
- All replies must be concise — Ahmad works from his phone
