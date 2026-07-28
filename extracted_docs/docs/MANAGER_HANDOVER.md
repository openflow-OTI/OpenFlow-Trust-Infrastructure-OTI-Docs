# OTI — Manager Handover Document
> Last updated: July 28, 2026 (session 20 — Strategic pivot recorded: private sale first, XMTP campaign second. New token distribution model recorded. GitHub security decision recorded. Full new direction summary added. Questions for Ahmad added for next Manager session.)
> **If you are a new Manager reading this: start here. Then read ARCHITECTURE.md, ROADMAP.md, TOKENOMICS.md, DECISIONS.md, and TASKS.md in that order.**
> **D16 (evidence rule): no signal value or test result may be estimated or guessed — only real on-chain data. A Builder's "verified" claim is NOT evidence. Ask: which wallet, which raw API response, which psql output.**
> **Read TOKENOMICS.md before touching anything token-related — price/liquidity sections deliberately absent at Ahmad's request. Do not add them.**

---

## Where the Code Lives

This Replit workspace is the Manager's workspace — documentation, prompt writing, roadmap management only. No OTI source code lives here.

- **Backend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI- — PRIVATE
- **Frontend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-O-T-I-Frontend- — PUBLIC (needs audit — see D31)
- **Docs repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI-Docs — this workspace (currently public — needs to go private or be stripped of internal files before private sale launches)

**GitHub security — urgent (D31):** The docs repo is currently public and contains internal files: FIXES.md, TASKS.md, ARCHITECTURE.md, BUILDER_ONBOARDING.md, MANAGER_HANDOVER.md. This must be resolved before the private sale launches. Ahmad to either make the docs repo private or strip it to public-facing content only. From this point forward, Builders do not push to GitHub — Ahmad pushes only.

---

## About Ahmad

- CEO of OpenFlow Labs, sole GitHub merge authority
- Not a software engineer — explain clearly, never condescendingly
- Works primarily from his phone
- Strong product vision — trust it, build around it
- Call him Ahmad (never "sir", never "boss")
- Every Manager reply to Ahmad must be in a copy box so he can paste it to Builders
- One task at a time per Builder — hard, non-negotiable rule
- Bug fixes never get task numbers — they live in FIXES.md only

---

## The Team

| Role | Status |
|---|---|
| Ahmad (CEO) | Always active — sole merge authority |
| Backend Builder | Idle — Tasks 19–22 on hold pending private sale direction |
| Frontend Builder | Idle — Tasks 19–22 on hold pending private sale direction |
| Development Manager | Being replaced — you are the new Manager |

---

## Current Production State (July 28, 2026)

**Live and working:**
- Backend: `https://workspaceapi-server-production-5c0c.up.railway.app`
- Frontend: `https://otiscore.vercel.app` (`/`, `/score`, `/whitepaper`, `/admin`, `/services`, `/register`, `/report`)
- Developer docs: `https://otiscore.vercel.app/docs/`
- 12 chains scored (7 EVM + 5 non-EVM). Sui broken via BF41 (JSON-RPC deprecated — Ahmad will fix later when presale pays). BSC/Base/Optimism return 503 (need Etherscan Lite $49/mo — Ahmad subscribes when presale pays). Features for all chains remain in the code.
- Two-tier cache: L1 LRU (500 entries, 5-min TTL) + L2 chain_scores DB (30-day rescore window)
- Keep-highest write logic on chain_scores
- API key + quota system live (anonymous limit currently 3/day — to be raised, see D33)
- Admin panel fully secured (x-admin-secret header, adminAuth.ts middleware)
- WOR (Wallet Ownership Registry) fully live — /register, /report, admin WOR tab
- compromised_wallets is single source of truth for all flagged-wallet views
- /services hub live
- Score sharing PNG cards live
- All fixes: BF1–BF40, FF1–FF27 complete
- All tasks: Task 8–18 complete
- Phase 1: COMPLETE. Phase 2 (WOR): COMPLETE.

**Critical infrastructure notes:**
- Railway does NOT auto-run `drizzle-kit push`. Every schema change needs Ahmad to manually run it against Railway production DATABASE_URL after deploy. Run from: `cd /app/lib/db` then `drizzle-kit push`.
- `subscriptions` table real columns: `id, api_key, plan, owner_address, created_at, expires_at, updated_at`. NO `status` column, NO `email` column. Use raw SQL on this table only — never Drizzle ORM selects.

---

## Sacred Files — Never Touch

| File | Reason |
|---|---|
| `src/lib/scoring.ts` | Core IP — the trust algorithm. NEVER modify. |
| `nixpacks.toml` | Railway build config. Never touch. |
| `vercel.json` | SPA routing + docs proxy rewrites. Never touch. |
| `schema.gen.ts` | OpenAPI-generated types. Never touch manually. |

---

## Active Decisions — Never Reverse Without Ahmad's Approval

| Decision | Source |
|---|---|
| scoring.ts is sacred, never modify | Core IP |
| CORS is fully open | Intentional — public developer API |
| BSC/Base/Optimism return 503 | Waiting for Etherscan Lite — Ahmad subscribes when presale pays |
| Sui features remain in code | BF41 — Ahmad will fix when presale pays |
| Admin page is URL-only, no nav link | Ahmad's decision |
| WOR self-reports are automated — no admin review queue | Ahmad's decision |
| compromised_wallets is sole source of truth for flagged wallets | BF38/39/40 lesson |
| One task per Builder at a time | Ahmad's hard rule |
| Fixes never get task numbers — FIXES.md only | Ahmad's explicit correction |
| Price/liquidity absent from TOKENOMICS.md | Deliberately absent — do not add back |
| DECISIONS.md is Manager-write, Builder-read | Ahmad's direction |
| Homepage at / stays unchanged | Ahmad's decision July 15, 2026 |
| OTI token is independent from FLOW | Separate tokenomics, separate fundraising |
| Contract addresses scored same as any wallet | No address-type gatekeeping |
| Etherscan key rotation: max 10 free keys | ToS boundary — see D25 |
| ETH scores used for BNB campaign — BSC blocker bypassed | D26 |
| GitHub: no internal files in public repos | D31 — Ahmad July 28, 2026 |
| No AI exposure in any public-facing content | D32 — Ahmad July 28, 2026 |
| Free product: anonymous rate limits to be removed | D33 — Ahmad July 28, 2026 |
| Private sale first, XMTP campaign second | D28 — Ahmad July 28, 2026 |
| Token distribution: 25% free, 75% auto-staked daily 5 years | D29 — Ahmad July 28, 2026 |
| All team allocations locked, daily claimable over 5 years | D30 — Ahmad July 28, 2026 |

---

## Infrastructure Cost & Capacity

> Researched July 27, 2026. All figures verified by previous Manager.

### Monthly & Annual Cost

| Item | Monthly | Annual |
|---|---|---|
| Railway (backend + PostgreSQL) | $5–20 | $60–240 |
| Vercel (frontend + docs) | $0 | $0 |
| Domain | ~$1.25 | ~$15 |
| Etherscan Lite x1 key (BSC + Base + Optimism) | $49 | $588 |
| BAS weekly attestation gas (BNB Chain) | $5–40 | $60–480 |
| All other providers (mempool.space, TronScan, Toncenter, RouteScan, Sui GraphQL, Solana RPC, zkSync Free) | $0 | $0 |
| **TOTAL lean (no active campaign)** | **~$60–65/mo** | **~$723–780/yr** |
| **TOTAL active campaign** | **~$100–110/mo** | **~$1,200–1,320/yr** |

**Hard floor (absolute minimum):** ~$55–70/mo — Railway + Etherscan Lite + domain.

### Total Wallet Universe: ~209.8M Across All Chains
Full breakdown in ARCHITECTURE.md. BSC/Base/Optimism are the bottleneck — 1 paid Etherscan key covers only 26% of those chains annually. Full coverage needs 4 keys at $196/mo.

---

## Phase Status (July 28, 2026)

| Phase | Status |
|---|---|
| Phase 1 — Foundation | COMPLETE |
| Phase 2 — WOR | COMPLETE |
| Phase 0 (NEW) — Private Sale Infrastructure | NEXT — full design required, see below |
| Phase 2B — XMTP Revenue Campaign (Tasks 19–22) | ON HOLD — runs after private sale raises money |
| Phase 2B — Post-Campaign Remaining | Planned — after campaign revenue |
| Phase 3 — Monetization + OTI Token | Planned |
| Phase 4 — Growth Features | Planned |
| Phase 5 — Distribution (bots, widget, extension) | Planned |

---

## The New Direction — Private Sale First (Confirmed July 28, 2026)

Ahmad has confirmed a strategic pivot. All building operations are paused until the private sale infrastructure is designed and built. The full context of this decision is in DECISIONS.md (D28).

### What the Private Sale Is

A private sale of OTI tokens on BNB Chain, targeted at committed buyers who understand the product and believe in the vision. Buyers access a purpose-built private sale site, read everything about OTI (whitepaper, documentation, FAQ, pricing), connect their wallet, and purchase OTI tokens. The smart contract handles distribution automatically.

**Token distribution on purchase (D29, D30):**
- 25% of purchased tokens: sent immediately to buyer wallet / visible in OTI private sale dashboard
- 75% of purchased tokens: auto-staked in distribution contract, claimable daily over 5 years (1,825 daily equal portions)
- All team-controlled allocations (Team, Ecosystem, Rewards Pool, Treasury): 100% locked, daily claimable over 5 years — same mechanism
- Only exception: Liquidity & Market Making (3M OTI) — fully available at listing for DEX depth

**Sale parameters not yet decided (ask Ahmad):**
- How many tokens in the private sale and at what price
- Referral commission rate and whether referral tracking is on-chain or off-chain
- Whether the private sale replaces the $10k pre-listing round from the old TOKENOMICS.md or is a new structure

### What Needs to Be Built Before the Private Sale Can Open

This is the complete list, in rough priority order. Ahmad must confirm the sequence before any Builder starts work.

**1. GitHub cleanup (urgent — no Builder needed, Ahmad action)**
Make the docs repo private on GitHub. Audit the frontend public repo for any internal comments or structural information that should not be public.

**2. Whitepaper rewrite (Frontend Builder)**
The current whitepaper exists at `/whitepaper` but undersells the technical depth and does not reflect the new token/presale structure. The rewrite must:
- Show the five-signal scoring algorithm methodology (not implementation details — describe the signals, explain the approach)
- Cover the infrastructure: chains supported, providers, the two-tier cache, keep-highest logic
- Cover the WOR system and what makes it novel
- Cover the BAS attestation layer and what it enables for wallet trust
- Cover the OTI token: supply, allocation, the new distribution model (25% free / 75% auto-staked daily 5 years), team lockup
- Cover the roadmap: what is live now, what is coming
- Cover the market: who needs on-chain trust infrastructure and why
- Write in clean, professional prose — no emojis, no AI-native patterns (D32)
- All chain counts must be accurate. Do not claim 15 chains. Do not reference Scroll, Sepolia, Holesky.

**3. Privacy Policy and Terms & Conditions (Frontend Builder)**
Both pages must be live before the private sale opens. They must cover:
- The scoring product: what data OTI processes (on-chain public data only, no PII), how it's used, what OTI does not do
- The API: developer terms, rate limits, acceptable use
- The private sale: what buyers are purchasing, the vesting/distribution terms, the risks, non-guarantee of returns
- The referral system: commission structure, payout terms

**4. Remove anonymous rate limits — free product (Ahmad admin panel action + Backend/Frontend Builder)**
- Ahmad: update the `anonymous` plan in `plan_configs` table via admin panel to a high or unlimited daily limit (this is a database value, not a code change)
- Frontend Builder: update any hardcoded "3 per day" text on the frontend to reflect the new limit
- Backend Builder: build self-serve API key signup flow so developers can get a key without contacting Ahmad

**5. Private sale site (Frontend Builder + Backend Builder)**
A dedicated private sale page or section, professionally designed, containing:
- Full explanation of OTI and what it's building
- Token economics: supply, allocation breakdown, the private sale terms
- Distribution model clearly explained (25% free / 75% daily over 5 years)
- Roadmap overview
- FAQ — thorough and honest
- Referral system explanation and interface
- Smart contract integration: connect wallet, purchase, view dashboard (staked balance, daily claimable, accumulated)

**6. BNB Chain smart contracts (Backend Builder)**
Four contracts minimum, all deployed on BNB Chain testnet first, then mainnet after Ahmad confirms:
- Sale contract: accepts BNB or USDT, records the purchase
- Distribution contract: executes 25%/75% split on purchase, makes 75% claimable daily over 1,825 days
- Referral contract or backend system: tracks referrals, distributes commissions
- Admin controls: pause sale, update parameters, withdraw raised funds

**7. Docusaurus docs site audit (Frontend Builder)**
Review all content for accuracy:
- Chain count and chain names must be accurate
- Remove any content that reveals internal architecture (key rotation details, how providers work, admin route structure)
- Remove reference to Scroll, Sepolia, Holesky
- Sui and BSC/Base/Optimism: describe honestly — supported in code, not yet enabled (not "broken", just "coming soon")
- Update API reference to match current actual response shape
- All writing must meet D32 standard (no AI-native patterns)

---

## Questions for Ahmad — Next Manager Session Must Cover These

The previous Manager session ended before Ahmad could answer these. The next Manager must ask Ahmad these questions before assigning any Builder tasks.

**1. Private sale token terms:**
How many OTI tokens are available in this private sale, and at what price per token? This determines the smart contract sale cap, the dashboard display, and all the economics in the whitepaper. (Ahmad said he will decide this during the build phase — remind him we need it before the sale contract can be written.)

**2. Referral system structure:**
- What is the referral commission — a percentage of what the referred buyer pays?
- Is referral tracking on-chain (handled by the smart contract, trustless) or off-chain (handled by the backend, simpler to build)?
- Does the referrer receive their commission in BNB/USDT or in OTI tokens?

**3. GitHub docs repo:**
Should the docs repo go fully private, or be restructured to only contain public-facing content (whitepaper source, public docs)? This determines whether Builders still have access to the repo at all.

**4. Broken chains and 503 chains — public messaging:**
For the private sale site and whitepaper, how should Sui (broken) and BSC/Base/Optimism (503) be described? Options: (a) simply not mention them; (b) list as "coming soon" with no explanation; (c) describe as "code-complete, activation pending infrastructure subscription." Ahmad's answer determines whitepaper and docs site language.

**5. Self-serve developer API keys:**
When anonymous rate limits are removed, what is the new free developer daily limit? And what plan tiers does Ahmad want available at private sale launch — just free, or also a paid tier?

**6. Domain:**
Is Ahmad planning to acquire a proper domain (e.g. `openflowlabs.io` or `getoti.com`) before the private sale launches? The private sale site would look significantly more professional on a real domain than on `otiscore.vercel.app`.

**7. Builder resumption:**
Which Builder should start first? The recommended order is: Frontend Builder (whitepaper rewrite and privacy/T&Cs) first, while the Backend Builder designs the smart contracts. But Ahmad confirms which goes first.

**8. Private sale site — separate domain or same Vercel project?**
Should the private sale live at `otiscore.vercel.app/sale` or on a dedicated domain/subdomain? This affects the Frontend Builder's architecture for that page.

---

## XMTP Campaign — Still Valid, On Hold

Tasks 19–22 are written and ready in TASKS.md. The XMTP campaign is NOT cancelled — it is the second funding mechanism, running after private sale money is secured. Full task prompts exist; no rework needed. When Ahmad is ready to run the campaign, the next Manager simply sends Task 19 prompt to the Backend Builder.

**Ahmad's actions still required before Task 19 can start:**
1. Register 10 Etherscan accounts (separate emails, not all in one session — ToS risk). Get 10 API keys.
2. Manager adds them to Railway as `ETHERSCAN_API_KEYS=key1,key2,...,key10`
3. Then send Task 19 prompt to Backend Builder.

---

## Phase Status

| Phase | Status |
|---|---|
| Phase 1 — Foundation | COMPLETE |
| Phase 2 — WOR | COMPLETE |
| Private Sale Infrastructure (new, highest priority) | NEXT — design required before any Builder starts |
| Phase 2B — Revenue Campaign (Tasks 19–22) | ON HOLD — runs after private sale money secured |
| Phase 2B — Post-Campaign Remaining | Planned |
| Phase 3 — Monetization + OTI Token (full) | Planned |
| Phase 4 — Growth Features | Planned |
| Phase 5 — Distribution | Planned |

---

## Standing Rules Every Session

1. **D16 evidence standard:** Builder's "verified" is not evidence. Always ask: which wallet address, which raw curl/psql output, was it a fresh call or cached.
2. **One active task per Builder at a time.** Queue next task only after Ahmad confirms current one live.
3. **Builder never marks complete themselves.** Manager tells Builder to mark complete only after Ahmad confirms live.
4. **Builder file copies never auto-sync.** Manager must explicitly tell each Builder to update their own TASKS.md/FIXES.md copy every time status changes.
5. **Before ending any session:** Update this file's "Current Production State" and next Manager actions. Update TASKS.md active queue. If a fix was closed, update FIXES.md. Never close a session with stale docs.
6. **Fixes never get task numbers.** FIXES.md only. BF## for backend, FF## for frontend.
7. **Next BF number: BF42. Next FF number: FF28. Next Task number: Task 23** (Tasks 19–22 are the campaign, on hold but written.)
8. **No AI exposure (D32).** All public-facing content written by any Builder must be reviewed against this standard before it goes live. No emojis, no AI-native writing patterns.
9. **Builders do not push to GitHub.** Ahmad pushes only.

---

## Key Lessons — Carry Forward Every Session

- **WalletConnect challenge TTL = 15 minutes.** Full sign flow must complete within this window or it silently hits the 400 branch.
- **compromised_wallets is the single source of truth.** Score endpoint, admin WOR Compromised view, dashboard stats — ALL must query this table. Never split across compromised_wallets and wallet_ownership.status. BF38/39/40 all caused by this.
- **Railway migrations don't auto-run.** Ahmad manually runs drizzle-kit push after every schema change.
- **Builder onboarding gap.** New Builder starts with zero API keys/secrets. Re-add all of them. Always full onboarding before resuming work.
- **XMTP fees are $0 now — run campaign before they activate.** When mainnet fees kick in (~$50–100 per 1M messages), Campaign 2 economics change.
- **BAS schema UID must be registered before smart contract is written** (Task 20 Part A). Ahmad pays the gas (~$0.01 BNB). The resulting schema UID is hardcoded into the smart contract.
- **Main app uses npm. oti-docs/ uses pnpm.** Never mix.
- **Private sale smart contracts hold real money.** All contracts must be thoroughly tested on BNB testnet before mainnet deployment. Never skip testnet.

---

## The Replit Multi-Account System

Ahmad uses multiple Replit free-tier accounts:
- **Manager account** — docs, prompts, roadmap
- **Frontend Builder account** — React/Vite on Vercel
- **Backend Builder account** — Node.js/Express on Railway

When credits exhaust, Ahmad pushes to GitHub via Replit Git, opens a new account, and the new Manager reads this file to continue. All context lives in the GitHub docs — never in chat memory.

**Doc files (all in extracted_docs/docs/ in this workspace):**
- `MANAGER_HANDOVER.md` — this file, start here
- `ARCHITECTURE.md` — what every piece of the codebase is
- `ROADMAP.md` — all planned features with full specs
- `TASKS.md` — master task list with full Builder prompts (Tasks 19–22 are written and on hold)
- `FIXES.md` — all bug fixes by Builder
- `DECISIONS.md` — why things exist the way they do (D28–D33 are new from session 20)
- `BUSINESS_MODEL.md` — revenue layers
- `TOKENOMICS.md` — OTI token (30M fixed supply, BSC first, new distribution model as of session 20)

---

## Critical Context That Must Never Be Lost

1. **Private sale is the immediate priority.** All Builders are idle. No tasks are assigned. The next work is designing and building the private sale site and its prerequisites. Get answers to the open questions above before assigning any Builder.
2. **Token distribution model (D29/D30):** 25% free immediately, 75% auto-staked daily over 5 years. All team allocations same mechanism. Only Liquidity bucket (3M OTI) is fully unlocked at listing. This is locked — do not change without Ahmad's explicit direction.
3. **GitHub is a security risk right now (D31).** The docs repo is public with internal files. Ahmad must make it private before the private sale launches. Remind Ahmad of this at the start of the next session.
4. **No AI exposure anywhere in public content (D32).** Review everything against this standard before it goes live.
5. **XMTP campaign (Tasks 19–22) is on hold, not cancelled.** Task prompts are written and ready in TASKS.md. Campaign runs after private sale money is secured.
6. **Sui is broken (BF41), BSC/Base/Optimism return 503.** Ahmad will fix/subscribe when presale pays. Do not remove their features from the code. Do not add them to public-facing chain counts until they are working.
7. **Ahmad has not yet answered:** token price, token amount for sale, referral commission rate, referral on-chain vs off-chain, which Builder starts first. These are open questions for next session.
8. **Phase 2B campaign uses Ethereum scores for BNB Chain.** Same 0x address = same person. No BSC Etherscan Lite needed for the campaign (D26).
9. **BAS schema UID** must be registered before Task 21 smart contract is written (Task 20 Part A). Ahmad pays the gas.
10. **Task numbering:** Tasks 8–18 complete. Tasks 19–22 = XMTP Campaign (on hold, prompts written). Next new task = Task 23.
