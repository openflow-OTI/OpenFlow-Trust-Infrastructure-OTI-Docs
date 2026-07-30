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
  TOKENOMICS.md             — OTI Economics: 35M supply, DCS + ERP bonding curves
  BUSINESS_MODEL.md         — revenue model, network effect engine
  TASKS.md                  — master task list (Tasks 19–28 written and queued)
  FIXES.md                  — bug fix log by Builder (BF41 open)
  DECISIONS.md              — why things exist the way they do
  BACKEND_TASKS.md          — Backend Builder's task prompt copy
  FRONTEND_TASKS.md         — Frontend Builder's task prompt copy
  BUILDER_ONBOARDING.md     — onboarding guide for new Builders
  MANAGER_ONBOARDING_PROMPT.md — copy-paste prompt for new Manager account
  whitepaper-additions-draft.md — whitepaper additions (merge into Task 25)
```

## Key facts

- **Live frontend:** https://otiscore.vercel.app
- **Live backend:** https://workspaceapi-server-production-5c0c.up.railway.app
- **Live docs:** https://otiscore.vercel.app/docs/
- **Next task to assign:** Task 23 (Frontend Builder — GitHub repo cleanup)
- **Next fix numbers:** BF42 (backend), FF28 (frontend)
- **Next task number:** Task 29

## User preferences

- Ahmad is the CEO — call him Ahmad, never "sir" or "boss"
- Every reply to Ahmad goes in a copy box so he can paste it to Builders
- One task per Builder at a time — hard, non-negotiable rule
- Fixes never get task numbers — FIXES.md only
- Builders do not push to GitHub — Ahmad only
- All replies must be concise — Ahmad works from his phone
