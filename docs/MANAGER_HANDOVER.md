# OTI — Manager Handover Document
> Last updated: July 30, 2026 (session 25 — Backend Builder new account onboarded: answered all 8 orientation questions correctly, confirmed understanding of DCS/ERP mechanics, evidence standard, migration flow, and silent flagging. Task 28 Part A prompt sent. Builder has not yet confirmed schema is ready — follow up on Part A first. D59 correction applied: BSC contract handles BNB + BSC stablecoins only; ETH/BTC/SOL/TON/XRP/MATIC use Layer 2 receiving addresses + CoinGecko pricing + admin manual verification — no BSC Chainlink oracle for non-BSC coins. 14 new whitelist tables total confirmed.)
> (session 24 — Task order updated: Task 28 backend first/urgent today, Tasks 23+24 deferred to last after all building done, D59 multi-coin DCS contribution contract added, D60 T&C/Privacy dual-set confirmed. Backend Builder new account onboarding prompt written.)
> (session 23 — Whitelist system fully redesigned: expanded ERP task system (9 reward types including daily scoring, WOR actions, dev API, whitepaper rounds), wallet-first UX flow, Telegram + X identity gates, session fingerprinting + multi-account detection, all manual admin review replaced by auto-verification. D43–D58 logged. Tasks 27 and 28 fully rewritten in TASKS.md and BACKEND_TASKS.md.)
> **If you are a new Manager reading this: start here. Then read ARCHITECTURE.md, ROADMAP.md, DECISIONS.md, TASKS.md, and TOKENOMICS.md in that order.**
> **D16 (evidence rule): no signal value or test result may be estimated or guessed — only real on-chain data. A Builder's "verified" claim is NOT evidence. Ask: which wallet, which raw API response, which psql output.**

---

## Where the Code Lives

This Replit workspace is the Manager's workspace — documentation, prompt writing, roadmap management only. No OTI source code lives here.

- **Backend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI- — **PRIVATE** (contains scoring algorithm IP + sensitive files)
- **Frontend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-O-T-I-Frontend- — **PUBLIC** (intentional — transparency. Problem: Builder workspace files got pushed here by Ahmad. Needs cleanup — see GitHub section below.)
- **Docs repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI-Docs — this workspace — **PRIVATE** (contains internal docs)

**GitHub situation (updated session 21):**
The issue is NOT the docs repo — it is already private. The issue is the **frontend public repo**. When Builders push code, their workspace copies of internal files (TASKS.md, FIXES.md, ARCHITECTURE.md, BUILDER_ONBOARDING.md, etc.) have been pushed alongside the source code. The frontend repo must be:
- Cleaned of all internal workspace documentation files
- Redesigned to show only the actual built product code
- Made to look professional and transparent — exactly what was built, nothing internal

The backend repo stays private (scoring algorithm IP + sensitive infrastructure). The docs repo stays private (internal project documentation). Only the frontend repo is public, and only for code transparency.

**Builders do not push to GitHub. Ahmad pushes only.**

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
| Backend Builder | New account being created today — Task 28 assigned immediately (Railway subscription urgency) |
| Frontend Builder | Idle — Tasks 25, 26, 27 queued. Tasks 23 and 24 deferred to after all building is complete. |
| Development Manager | Active — you are reading this |

---

## Current Production State (July 29, 2026)

**Live and working:**
- Backend: `https://workspaceapi-server-production-5c0c.up.railway.app`
- Frontend: `https://otiscore.vercel.app` (`/`, `/score`, `/whitepaper`, `/admin`, `/services`, `/register`, `/report`)
- Developer docs: `https://otiscore.vercel.app/docs/`
- 12 chains scored (7 EVM + 5 non-EVM)
- Sui broken via BF41 (JSON-RPC deprecated — Ahmad will fix later when funded)
- BSC/Base/Optimism return 503 (need Etherscan Lite $49/mo — Ahmad subscribes when funded)
- Two-tier cache: L1 LRU (500 entries, 5-min TTL) + L2 chain_scores DB (30-day rescore window)
- Keep-highest write logic on chain_scores
- API key + quota system live
- **Anonymous rate limit: already removed by Ahmad directly via admin panel** (session 21)
- Admin panel fully secured (x-admin-secret header, adminAuth.ts middleware)
- WOR (Wallet Ownership Registry) fully live — /register, /report, admin WOR tab
- compromised_wallets is single source of truth for all flagged-wallet views
- /services hub live
- Score sharing PNG cards live
- All fixes: BF1–BF40, FF1–FF27 complete (BF41 open — Sui broken)
- All tasks: Task 8–18 complete
- Phase 1: COMPLETE. Phase 2 (WOR): COMPLETE.

**Critical infrastructure notes:**
- Railway does NOT auto-run `drizzle-kit push`. Every schema change needs Ahmad to manually run it against Railway production DATABASE_URL after deploy. Run from: `cd /app/lib/db` then `drizzle-kit push`.
- `subscriptions` table real columns: `id, api_key, plan, owner_address, created_at, expires_at, updated_at`. NO `status` column, NO `email` column. Use raw SQL on this table only — never Drizzle ORM selects.
- Developer API limits: Ahmad sets these himself through the admin panel. Manager and Builders never set or hardcode API limit amounts.

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
| BSC/Base/Optimism return 503 | Waiting for funding — leave as-is, no public language change |
| Sui features remain in code | BF41 — Ahmad will fix when funded — leave as-is |
| Admin page is URL-only, no nav link | Ahmad's decision |
| WOR self-reports are automated — no admin review queue | Ahmad's decision |
| compromised_wallets is sole source of truth for flagged wallets | BF38/39/40 lesson |
| One task per Builder at a time | Ahmad's hard rule |
| Fixes never get task numbers — FIXES.md only | Ahmad's explicit correction |
| DECISIONS.md is Manager-write, Builder-read | Ahmad's direction |
| Homepage at / stays unchanged | Ahmad's decision July 15, 2026 |
| OTI token is independent from FLOW | Separate tokenomics, separate fundraising |
| Contract addresses scored same as any wallet | No address-type gatekeeping |
| Etherscan key rotation: max 10 free keys | ToS boundary — see D25 |
| ETH scores used for BNB campaign — BSC blocker bypassed | D26 |
| No AI exposure in any public-facing content | D32 |
| Anonymous rate limits removed — free product | D33 — already done by Ahmad via admin panel |
| Whitelist not presale — regulatory compliance | Session 21 — see full direction below |
| Off-chain referral/invite tracking | Session 21 — referral relationships tracked in DB, not on-chain |
| All vesting/lockup percentages configurable via admin dashboard | Session 21 — nothing hardcoded |
| Referral commissions paid in OTI token, not BNB/USDT | Session 21 |
| DCS accepts 10 coins each on their own native chain — BSC contract handles BNB+stables; ETH/SOL/TON/XRP/BTC/MATIC each have Layer 2 receiving addresses + CoinGecko pricing; no BSC wrappers | D59, Session 24 |
| T&C and Privacy Policy: two sets — general product + whitelist-specific, on same /terms and /privacy pages as separate sections with anchors | D60, Session 24 |
| GitHub cleanup (Task 23) and Docusaurus audit (Task 24) deferred to AFTER all building is complete — full sanitation at end | Session 24 |
| OTI Economics confirmed — 35M fixed supply, full allocation, dual bonding curve (DCS + ERP) documented in TOKENOMICS.md | Sessions 21–22 |
| Frontend GitHub repo needs cleanup of internal workspace files | Session 21 |
| Developer API limits set by Ahmad via admin panel only | Session 21 |
| Private sale domain: /whitelist on existing Vercel project | Session 21 |
| All ERP reward amounts admin-configurable — no hardcoded amounts anywhere | D43, Session 23 |
| ERP expanded: 9 task types including daily scoring, WOR, dev API, whitepaper rounds | D44, Session 23 |
| /whitelist UX: wallet connect → invite code → portal (wallet first) | D45, Session 23 |
| Developer API signup is wallet-only — no email | D46, Session 23 |
| Whitelist gate copy: "Access is restricted to whitelisted users." only | D47, Session 23 |
| "Committed rate" not "contribution rate" everywhere | D48, Session 23 |
| Social tasks auto-verified via URL — no manual admin review queue | D49, Session 23 |
| Telegram phone verification required before any reward claim | D50, Session 23 |
| X (Twitter) account connection required before any reward claim | D51, Session 23 |
| No face verification | D52, Session 23 |
| No hardcoded limits — all caps admin-configurable, 0 = unlimited | D53, Session 23 |
| /whitelist in navbar from day one | D54, Session 23 |
| Invite code generation is Ahmad's responsibility | D55, Session 23 |
| Referral link format: otiscore.vercel.app/whitelist?ref=OTI-XXXX-XXXX | D56, Session 23 |
| No OTI BEP-20 on mainnet yet — placeholder in vesting contract, setTokenAddress() required | D57, Session 23 |
| Session fingerprinting + multi-account detection — IP + fingerprint hash collision flagging | D58, Session 23 |

---

## The New Direction — Ecosystem Whitelist Program (Confirmed Session 21)

**All previous "presale" or "private sale" framing is replaced by "Ecosystem Whitelist Node Program."** This is a regulatory compliance decision. The product and mechanics are the same — the framing, vocabulary, and legal structure are completely different.

### Vocabulary — Enforced Everywhere

| OLD (banned) | NEW (required) |
|---|---|
| Token Sale / Private Sale / ICO / Presale | Ecosystem Whitelist / Node Testing Program |
| Buy Tokens / Invest | Acquire Network Access Fuel / Claim Allocation |
| Staking Payouts / ROI / Yield | Node Collateral Lockup / Linear Network Vesting |
| Investors | Whitelisted Operators / Community Contributors |
| Trading / Listing | Public Utility Liquidity Pool Seeding |

### What the Whitelist Program Is

A gated, invite-code-only access system. A public web crawler or automated scanner sees a locked private network onboarding tool — not a token sale. Only users manually approved by the admin can unlock the portal.

- Access gated by single-use invite codes (format: OTI-XXXX-XXXX), admin-generated in batches
- Target: 10,000 unique whitelisted slots over 12 months
- Total whitelist allocation: **25% of total token supply** — covers all whitelist events (direct claims, referral rewards, social media rewards, community contributor rewards)
- Vesting/lockup parameters: **all configurable via admin dashboard** — not hardcoded in smart contracts
- Off-chain referral tracking: tracked in `whitelist_invites` DB table — admin has full control and adjustability
- Referral commissions: paid in OTI token
- Mandatory Terms & Conditions + Privacy Policy checkbox before code redemption
- Geographic restrictions enforced (US, China, sanctioned nations blocked)

### Milestones

- **Milestone 1 — Alpha Core Genesis:** Launch /whitelist. First batch of whitelisted operators via invite codes.
- **Milestone 2 — Phase 1 Liquidity Seeding:** At $5,000 committed allocation. Deploy initial Public Utility Liquidity Layer on decentralized protocols. OTI token becomes redeemable externally.
- **Milestone 3 — Deep Liquidity Scaling:** At $15,000 committed allocation. Secondary AMM pool funding. Stabilization of utility swap rates for B2B clients.

### Database Tables Required (New)

```sql
-- Invite code management
CREATE TABLE whitelist_invites (
    id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
    invite_code VARCHAR(50) UNIQUE NOT NULL,
    is_used BOOLEAN DEFAULT false,
    used_by_wallet VARCHAR(42) DEFAULT NULL,
    amount_contributed_usd NUMERIC(10, 2) DEFAULT 0.00,
    status VARCHAR(20) DEFAULT 'active', -- 'active', 'banned', 'expired'
    created_at TIMESTAMP WITH TIME ZONE DEFAULT NOW()
);

-- Protocol global aggregates (serves frontend live counter)
CREATE TABLE protocol_state (
    id INT PRIMARY KEY DEFAULT 1,
    total_committed_usd NUMERIC(10, 2) DEFAULT 0.00,
    total_slots_claimed INT DEFAULT 0,
    CONSTRAINT single_row CHECK (id = 1)
);
```

### New Admin Dashboard Features Required

1. **Batch Code Generator:** Admin button to generate X unique OTI-XXXX-XXXX codes → saved to `whitelist_invites` with status 'active'.
2. **Code Management Panel:** Table showing all codes, which wallet redeemed each, funding metrics, slots remaining out of 10,000.
3. **Ban System:** Toggle per code/wallet → changes status to 'banned' → frontend immediately blocks banned addresses from accessing their vesting allocation.
4. **Live Metric Override:** Input field to manually adjust `total_committed_usd` in `protocol_state` (sync from off-chain contributions).

### Frontend /whitelist Page Requirements

- Entry gate: unauthenticated visitors see only a professional security box — "OTI Infrastructure Hub — Private Whitelist Node Platform. Access is restricted to whitelisted node operators and infrastructure partners." No wallet connect buttons, no token charts, no contract details visible to anyone without a valid code.
- Live progress bar: "Total Ecosystem Committed Allocation Tracker" pulling from `protocol_state`. Shows progress toward Milestone 2 ($5k) and Milestone 3 ($15k).
- Invite code text input + mandatory Terms/Privacy checkbox.
- Once verified: wallet connect + allocation claim flow.

### Backend API Required

`POST /api/verify-invite` — validates invite code + wallet address + terms acceptance, marks code as used, increments `protocol_state` totals.

### Terms & Conditions (verbatim — provided by Ahmad, do not rewrite)

"1. PURPOSE OF THE WHITELIST: The OTI Whitelist Token Onboarding Program is strictly built to distribute network utility vouchers (Access Fuel) to future network testers, node operators, and B2B developers.
2. NO EXPECTATION OF PROFIT: Participants explicitly acknowledge that OTI tokens are utility tools used for wallet attestation fees and API queries. This program is not an investment, security, or financial contract. There is zero promise of future financial returns, passive yield, or profit.
3. PROTOCOL LOCKUP & VESTING: By accessing this software, the user agrees to the automatic 75% Node Collateral Lockup. Tokens will release linearly on a daily schedule to maintain network stability and protect circulating supply from systemic spamming.
4. GEOGRAPHIC RESTRICTIONS: This program is prohibited to residents, citizens, or IP addresses originating from high-risk or strictly regulated jurisdictions, including but not limited to the United States of America, China, and sanctioned nations.
5. ADMINISTRATIVE RIGHTS: The OTI core administration team reserves the absolute right to delete, freeze, or ban any invite code or wallet address found to be operating maliciously or misrepresenting the technical nature of the protocol."

### Privacy Policy (verbatim — provided by Ahmad, do not rewrite)

"1. DATA COLLECTION PRINCIPLES: The OTI platform operates under true Web3 data minimization protocols. We do not collect, request, or store your real name, physical address, phone number, or government identity documentation.
2. TYPES OF DATA LOGGED: The system strictly logs decentralized interaction points: (a) Public crypto wallet addresses used to claim allocations, (b) Validated single-use admin invite codes, and (c) Basic on-chain transaction hashes.
3. COOKIES & TRACKING: We do not deploy advertising tracker cookies or pixel trackers. Local device session storage may be temporarily used to verify your password token state for active sessions.
4. THIRD-PARTY DISCLOSURE: No collected Web3 identifier data is sold, rented, or passed to corporate advertisers. Data remains siloed in decentralized server instances solely used to manage active whitelist states."

---

## XMTP Campaign — Ongoing Program (Not Cancelled, Not "Second")

The XMTP campaign is an **ongoing acquisition program**, not a one-time event and not ranked second to anything. It runs continuously as the wallet database grows. When funded and ready:

**What to do next (in order):**
1. Ahmad registers 10 Etherscan accounts (separate emails, not all in one session). Gets 10 API keys.
2. Manager adds them to Railway as `ETHERSCAN_API_KEYS=key1,key2,...,key10`
3. Send Task 19 prompt to Backend Builder (Etherscan key rotation)
4. Task 20: BAS schema registration (Ahmad pays ~$0.01 gas) + signing endpoint
5. Task 21: Smart contract (BNB testnet first, mainnet after Ahmad confirms) + XMTP sender script
6. Task 22: Conversion dashboard (Frontend Builder)

Task prompts 19–22 are fully written in TASKS.md. No rework needed. Campaign targets Ethereum wallets with score ≥75, messages via XMTP, collects $1 BNB per attestation on BNB Chain. Uses ETH scores for BNB chain (same 0x address = same person — D26). Runs continuously, scales with wallet database.

---

## OTI Economics — Confirmed July 29, 2026

**TOKENOMICS.md has been fully rewritten as "OTI Economics."** 35,000,000 OTI fixed supply. No inflation. No post-launch mint. Full allocation breakdown, dual bonding curve system (DCS + ERP), and revenue distribution are documented there — use TOKENOMICS.md as the canonical reference for all token-related decisions.

**Confirmed allocation (35M total):**
- Ecosystem Whitelist: 25% (8.75M) — covers all whitelist events: direct claims, referral rewards, social rewards, community contributor rewards
- Network Reserve: 20% (7M) — DCS sub-pool: 7M OTI, linear bonding curve, raises $25,000
- Founders: 15% (5.25M) — 5-year linear vesting, no cliff
- Strategic Partnerships: 10% (3.5M)
- Liquidity: 10% (3.5M)
- Rewards Pool: 10% (3.5M) — funded by revenue buybacks only, never by minting
- Future Strategic Investment: 5% (1.75M) — ERP sub-pool: 1.75M OTI, inverse reward curve tied to DCS
- Operations Reserve: 5% (1.75M)

**Dual bonding curve system (Genesis Mode):**
- DCS (Dynamic Contribution Scale): 7M OTI, $0.001190/OTI → $0.005952/OTI (5× linear curve), raises $25,000 total
- ERP (Ecosystem Rewards Pool): 1.75M OTI, inverse curve — rewards decrease as DCS fills. Formula: `Current Reward = Base Reward × (DCS Remaining ÷ 7,000,000)`. Base rewards (admin-configurable): referral = 3,000 OTI, post/tag = 1,000 OTI, share/follow = 500 OTI each.
- The two sub-pools are fully independent — ERP rewards do NOT advance the DCS curve. ERP is free OTI on top.
- /whitelist page shows two live counters simultaneously: DCS rate going UP, ERP referral bonus going DOWN.

**Revenue distribution confirmed:** Operations 35%, Network Reserve 25%, Team Operations 20%, Rewards Pool 15% (open-market OTI buybacks — not new issuance), R&D 5%.

**Token utilities confirmed (growing list — not exhaustive):**
- Whitelist ecosystem access (Access Fuel)
- Wallet attestation fee payment (with discount vs other methods)
- API plan subscription payment
- Widget commercial access payment
- Developer staking for priority API access
- Revenue-backed staking rewards (real revenue buyback, not inflation)
- Early adopter/discovery rewards (proactively pre-scored wallets)
- Partner revenue share (commission for attestation conversions through embedded widget)
- Enterprise compliance data access
- Cross-chain expansion unlock fuel
- Score history & analytics access
- Webhook alert subscriptions
- Governance (future phase)

---

## Infrastructure Cost & Capacity

| Item | Monthly | Annual |
|---|---|---|
| Railway (backend + PostgreSQL) | $5–20 | $60–240 |
| Vercel (frontend + docs) | $0 | $0 |
| Domain | ~$1.25 | ~$15 |
| Etherscan Lite x1 key (BSC + Base + Optimism) | $49 | $588 |
| All other providers | $0 | $0 |
| **TOTAL lean** | **~$60–65/mo** | **~$723–780/yr** |

Domain plan: acquire `otiscore.com` once committed whitelist funds are in the ecosystem.

---

## Phase Status (July 29, 2026)

| Phase | Status |
|---|---|
| Phase 1 — Foundation | COMPLETE |
| Phase 2 — WOR | COMPLETE |
| Phase 0 (NEW) — Ecosystem Whitelist Infrastructure | NEXT — Tasks 23–28 written and ready to assign |
| XMTP Campaign (Tasks 19–22) | ONGOING PROGRAM — runs when funded, tasks written and ready |
| Phase 2B — Post-Campaign Remaining | Planned |
| Phase 3 — Monetization + OTI Token full ecosystem | Planned |
| Phase 4 — Growth Features | Planned |
| Phase 5 — Distribution (bots, widget, extension) | Planned |

---

## What to Build Next (Phase 0 — UPDATED ORDER, Session 24)

⚠️ **Railway subscription due today at midnight UTC. Task 28 backend must be built and tested today.**

All task prompts are fully written in TASKS.md and BACKEND_TASKS.md. Assign in this order:

**Task 28 — Backend Builder: Whitelist System — ASSIGN FIRST (URGENT TODAY)**
New Backend Builder account being created. Onboard via BACKEND_BUILDER_ONBOARDING_PROMPT.md, confirm understanding, then send task. 10 DB tables, full API endpoint set, admin Whitelist tab, two smart contracts (OTIDCSContribution.sol accepting BNB + 9 BEP-20 tokens with Chainlink oracles; OTIWhitelistVesting.sol for OTI distribution). Session fingerprinting + multi-account detection. All parameters admin-configurable. Parts A–C buildable immediately. Part D (contracts) after A–C deployed.

**Task 25 — Frontend Builder: Whitepaper Rewrite** ← assign ONLY after Task 28 all parts confirmed done
Merge existing whitepaper + `docs/whitepaper-additions-draft.md`. Correct chain count (12), fix stale chain table (remove Fantom/Scroll/Sepolia/Holesky), use whitelist vocabulary throughout, include confirmed OTI Economics from TOKENOMICS.md. D32 standard.
⚠️ Every new Frontend Builder must be onboarded first using FRONTEND_BUILDER_ONBOARDING_PROMPT.md before receiving any task prompt.

**Task 26 — Frontend Builder: Privacy Policy + Terms & Conditions Pages (EXPANDED — D60)**
Two sections per page, not two separate sets of pages:
- `/privacy`: Section 1 = General OTI Product Privacy (audit existing content from whitepaper + site, write properly). Section 2 = Whitelist-Specific Privacy (verbatim Ahmad text). Footer links to /privacy.
- `/terms`: Section 1 = General OTI Product T&C (same audit process). Section 2 = Whitelist-Specific T&C (verbatim Ahmad text). Footer links to /terms.
Whitelist mandatory checkbox links to anchor sections (#whitelist) on each page, not separate URLs.

**Task 27 — Frontend Builder: /whitelist Page**
Entry gate + authenticated portal with live DCS/ERP dual counters, invite code input, terms/privacy anchor links, wallet connect, allocation display, 9 reward task cards, whitepaper questions, milestones. Depends on Task 28 backend endpoints being live.

**Task 23 — Frontend Builder: GitHub Repo Cleanup — LAST (after all building complete)**
Full sanitation of the public frontend GitHub repo. Strip all internal .md files, leave only source code. Ahmad reviews and merges.

**Task 24 — Frontend Builder: Docusaurus Docs Site Audit — LAST (after all building complete)**
Remove sensitive internal info from public developer docs. Full sanitation pass. D32 standard.

---

## Standing Rules Every Session

1. **D16 evidence standard:** Builder's "verified" is not evidence. Always ask: which wallet address, which raw curl/psql output, was it a fresh call or cached.
2. **One active task per Builder at a time.** Queue next task only after Ahmad confirms current one live.
3. **Builder never marks complete themselves.** Manager tells Builder to mark complete only after Ahmad confirms live.
4. **Builder file copies never auto-sync.** Manager must explicitly tell each Builder to update their own TASKS.md/FIXES.md copy every time status changes.
5. **Before ending any session:** Update this file and TASKS.md. If a fix was closed, update FIXES.md. Never close a session with stale docs.
6. **Fixes never get task numbers.** FIXES.md only. BF## for backend, FF## for frontend.
7. **Next BF number: BF42. Next FF number: FF28. Next Task number: Task 29** (Tasks 23–28 fully written and queued in TASKS.md — Tasks 27 and 28 fully redesigned session 23. Task order updated session 24: Task 28 first/urgent, Tasks 25→26→27 next, Tasks 23 and 24 LAST after all building done.)
8. **No AI exposure (D32).** All public-facing content must be reviewed before going live.
9. **Builders do not push to GitHub.** Ahmad pushes only.
10. **Ahmad sets all API limits and rate limits himself via admin panel.** Manager and Builders never set or hardcode amounts.
11. **All vesting/lockup/distribution percentages are admin-configurable.** Nothing token-related is hardcoded.

---

## Key Lessons — Carry Forward Every Session

- **WalletConnect challenge TTL = 15 minutes.** Full sign flow must complete within this window or it silently hits the 400 branch.
- **compromised_wallets is the single source of truth.** Score endpoint, admin WOR Compromised view, dashboard stats — ALL must query this table.
- **Railway migrations don't auto-run.** Ahmad manually runs drizzle-kit push after every schema change.
- **Builder onboarding gap.** New Builder starts with zero API keys/secrets. Re-add all of them.
- **Main app uses npm. oti-docs/ uses pnpm.** Never mix.
- **Whitelist smart contracts hold real committed allocation.** All contracts must be thoroughly tested on BNB testnet before mainnet deployment. Never skip testnet.
- **BAS schema UID must be registered before smart contract is written** (Task 20 Part A). Ahmad pays the gas (~$0.01 BNB).
- **TOKENOMICS.md is now "OTI Economics."** 35M fixed supply confirmed July 29, 2026. Full allocation, dual bonding curve (DCS + ERP parameters), and revenue distribution are all documented there. This is the canonical token reference — use it.
- **All vocabulary in public-facing content must use whitelist framing** — no "sale", "invest", "ROI", "yield", "investor" language anywhere.

---

## The Replit Multi-Account System

Ahmad uses multiple Replit free-tier accounts:
- **Manager account** — docs, prompts, roadmap (this workspace)
- **Frontend Builder account** — React/Vite on Vercel
- **Backend Builder account** — Node.js/Express on Railway

When credits exhaust, Ahmad pushes to GitHub via Replit Git, opens a new account, and the new Manager reads this file to continue.

**Doc files (all in docs/ in this workspace):**
- `MANAGER_HANDOVER.md` — this file, start here
- `ARCHITECTURE.md` — what every piece of the codebase is
- `ROADMAP.md` — all planned features with full specs (needs update — whitelist framing)
- `TASKS.md` — master task list (Tasks 19–22 written and ready; Tasks 23–28 written and queued)
- `FIXES.md` — all bug fixes by Builder (BF41 open — Sui broken)
- `DECISIONS.md` — why things exist the way they do
- `BUSINESS_MODEL.md` — revenue layers (needs update — whitelist framing)
- `TOKENOMICS.md` — **Rewritten July 29, 2026 as "OTI Economics." 35M supply, full allocation and revenue distribution documented. Use this version.**
- `whitepaper-additions-draft.md` — previous Manager's whitepaper additions (chain table is stale, merge with whitepaper task)
- `BACKEND_BUILDER_ONBOARDING_PROMPT.md` — **paste verbatim to every new Backend Builder before any task.** Covers: OTI overview, whitelist system (DCS/ERP/reward gate/fingerprinting), sacred files, all secrets set, 6 orientation questions. Only send the task after all 6 answers are correct.
- `FRONTEND_BUILDER_ONBOARDING_PROMPT.md` — **paste verbatim to every new Frontend Builder before any task.** Covers: OTI overview, color system, sacred files, whitelist vocabulary enforcement, DCS/ERP mechanics, 6 orientation questions. Only send the task after all 6 answers are correct.
- `BACKEND_TASKS.md` — Backend Builder task queue (Task 28 active)
- `FRONTEND_TASKS.md` — Frontend Builder task queue (Tasks 25→26→27→23→24 queued in that order)

---

## SESSION 29 HANDOVER — July 31, 2026

### What happened this session
- Part C evidence confirmed live on Railway: code generation ✅, code ban ✅, question add/patch ✅, 114 seeded questions ✅, config PUT endpoint live ✅. Flag trigger skipped — Ahmad accepted, will verify visually when frontend is built.
- New Backend Builder onboarding prompt rewritten and saved to BACKEND_BUILDER_ONBOARDING_PROMPT.md — updated to reflect A/B/C complete, Part D as the only active task, 5 orientation questions focused on smart contracts.
- OTI BEP-20 token creation clarified: deferred until after whitelist launches. Builder deploys it as a future task; Ahmad adds liquidity on PancakeSwap himself. Task 27 (Frontend /whitelist page) handles all public-facing token info display.

### Builder status right now
- Backend Builder: new account needed — old account hit Replit monthly quota limit. Onboard with updated BACKEND_BUILDER_ONBOARDING_PROMPT.md, confirm 5 answers, then send Part D brief from BACKEND_TASKS.md.
- Frontend Builder: still idle — all Task 28 parts must be confirmed done first.

### Immediate next actions
1. Open new Backend Builder Replit account
2. Paste BACKEND_BUILDER_ONBOARDING_PROMPT.md verbatim
3. Wait for all 5 answers — confirm correct before sending Part D brief
4. Builder generates deployer wallet → sends DEPLOYER_ADDRESS to Manager → Manager relays to Ahmad → Ahmad sends ~0.05 BNB → Manager confirms → Builder proceeds to mainnet
5. Receive final handover package (all contract addresses + private key) → relay to Ahmad → Ahmad saves and deletes Builder workspace

---

## SESSION 28 HANDOVER — July 30, 2026

### What happened this session
- Railway platform incident resolved. Railway is fully back up and operational.
- All 3 Railway secrets CONFIRMED DEPLOYED and working: `TELEGRAM_BOT_TOKEN` ✅, `TWITTER_CLIENT_ID` ✅, `TWITTER_CLIENT_SECRET` ✅
- Twitter OAuth 2.0 app configured on console.x.com: type = Web App (Confidential client), callback = `https://workspaceapi-server-production-5c0c.up.railway.app/api/whitelist/connect-x/callback`
- New Backend Builder onboarded on a fresh account. Answered all 6 orientation questions correctly — confirmed understanding of DCS/ERP mechanics, reward gate, fingerprinting, D16, Railway migrations, and whitelist_config.
- DECISIONS.md clarification given to Builder: that file is Manager-only and intentionally not in the Builder workspace. Builder accepted this and is not blocked by it.
- Task 28 Part B continuation brief SENT to Backend Builder. Builder is currently: (1) fixing TS7006 implicit-any errors in src/routes/whitelist.ts, (2) confirming clean build, (3) pushing to GitHub, (4) testing all 11 endpoints with D16 evidence.
- Both onboarding prompts rewritten to be fully self-contained — no references to files Builders don't have. DECISIONS.md removed from both. All necessary knowledge is now inline.
- Frontend Builder: IDLE. Ahmad confirmed hard rule — ALL Task 28 parts must be complete before any frontend task is assigned.

### Builder status right now
- **Backend Builder:** ACTIVE — fixing TS7006, will push and test Part B endpoints. Awaiting D16 evidence report.
- **Frontend Builder:** IDLE — do not assign anything until Task 28 all parts are done.
- **BF41:** Open — Sui broken. Leave it.

### NEXT MANAGER: Do these steps in exact order

**STEP 1 — Receive Part B evidence from Backend Builder.**
Builder is working on Part B right now. When they report back, apply D16 to every single endpoint — no "verified" without raw HTTP responses and psql output. Required evidence checklist is in BACKEND_TASKS.md Task 28 Part B. Do not accept code inspection as evidence. Do not accept "it should work." Paste = pass, no paste = not done.

**STEP 2 — Once Part B evidence passes D16, tell Ahmad to push to GitHub and deploy to Railway.**
Part B is only confirmed live after Ahmad deploys and you verify endpoints against the live Railway URL.

**STEP 3 — Send Part C prompt to Backend Builder.**
Part C = Admin Dashboard Whitelist tab. Four sub-views:
1. Code Management Panel (generate/ban/expire OTI-XXXX-XXXX codes)
2. Whitepaper Questions Manager (CRUD for 100-question pool, pre-seed 100 questions at deploy)
3. Flagged Accounts Panel (ban/clear/review flags from whitelist_flags)
4. Protocol Config Override (live edit all whitelist_config keys via admin UI)
Full prompt is in BACKEND_TASKS.md.

**STEP 4 — Once Part C is confirmed live, send Part D prompt.**
Part D = Two BNB Chain smart contracts:
- OTIDCSContribution.sol — accepts BNB + USDT-BEP20 + USDC-BEP20 + BUSD-BEP20, Chainlink oracle for BNB/USD pricing
- OTIWhitelistVesting.sol — 25% immediate + 75% linear daily vesting, setTokenAddress() required before go-live
Builder generates deployer wallet offline → sends DEPLOYER_ADDRESS to Manager → Manager relays to Ahmad → Ahmad sends BNB for mainnet gas. Do NOT let Builder proceed to mainnet until Ahmad confirms BNB sent and Manager confirms balance on BscScan.
Full prompt is in BACKEND_TASKS.md.

**STEP 5 — Only after Task 28 ALL parts done and confirmed: assign Frontend Builder tasks.**
Order: Task 25 → Task 26 → Task 27 → Task 23 → Task 24. One at a time, per D16 evidence before moving to next.

### Key things the next Manager must know

- DECISIONS.md is Manager-only. Never tell Builders to read it. Never send it to them. All relevant decisions are communicated inline via task prompts.
- Every new Builder (Backend or Frontend) must receive the full onboarding prompt first. Files: BACKEND_BUILDER_ONBOARDING_PROMPT.md and FRONTEND_BUILDER_ONBOARDING_PROMPT.md. Both are self-contained — no external file references. Builder must answer all 6 orientation questions correctly before receiving any task prompt.
- One task per Builder at a time. Hard rule from Ahmad. Never queue a second task until the first is confirmed live.
- Railway does NOT auto-run migrations. After every schema change Ahmad manually runs drizzle-kit push from the Railway console.
- All whitelist parameters (reward amounts, vesting %, caps, thresholds) live in whitelist_config. Nothing token-related is ever hardcoded.
- ERP multiplier formula: reward = base_reward × (dcs_oti_remaining / dcs_total_oti). As DCS fills, ERP rewards shrink.
- Whitelist vocabulary is mandatory everywhere — no "presale", "invest", "ROI", "yield", "token sale" in any Builder output ever.
- Next BF: BF42. Next FF: FF28. Next Task: Task 29.

---

## Critical Context That Must Never Be Lost

1. **OTI Economics is confirmed and documented.** 35M supply, full allocation, dual bonding curve (DCS + ERP) — all in TOKENOMICS.md. Phase 0 task prompts (Tasks 23–28) are written in TASKS.md and ready to assign.
2. **Whitelist not presale — everywhere.** Every piece of public content, every task prompt, every piece of code the Builders write must use the correct vocabulary.
3. **Frontend GitHub repo has internal files in it** from Builder pushes. Must be cleaned before whitelist launches.
4. **TOKENOMICS.md is confirmed "OTI Economics."** Total supply: 35M fixed. The old 30M figure is scrapped. Do not reference it anywhere.
5. **Anonymous rate limit already removed** by Ahmad via admin panel.
6. **XMTP campaign is an ongoing program**, not a one-off, not ranked second. Runs when funded. Task prompts 19–22 are written and ready.
7. **All vesting/lockup parameters must be admin-configurable** — this is a hard requirement, not a preference.
8. **Docusaurus audit required** before whitelist launches — sensitive internal info may be exposed in public developer docs.
9. **Task numbering:** Tasks 8–18 complete. Tasks 19–22 = XMTP Campaign (ready, waiting for funding). New whitelist tasks start at Task 23.
10. **Next session starts with:** Task 28 (Backend Whitelist System) is the active Backend Builder task — check progress, confirm evidence per D16 before marking complete. Frontend Builder queue: Task 25 → 26 → 27 → then 23 and 24 last. One task per Builder at a time — hard rule.
11. **D59 — DCS accepts multi-coin:** OTIDCSContribution.sol accepts BNB + 9 BEP-20 tokens. Chainlink oracle addresses for each non-stable are in DECISIONS.md D59.
12. **D60 — T&C/Privacy split:** Two sections per page (/terms and /privacy). General product section + whitelist-specific section. Whitelist checkbox links via anchors.
