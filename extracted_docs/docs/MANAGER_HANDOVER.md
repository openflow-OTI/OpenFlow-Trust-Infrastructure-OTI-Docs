# OTI — Manager Handover Document
> Last updated: July 29, 2026 (session 21 — Full corrections session with Ahmad. Whitelist pivot confirmed (not presale). GitHub situation clarified (frontend repo only). Anonymous rate limit already removed by Ahmad. Tokenomics full redesign pending advisor session. Whitelist system spec received. All corrections recorded below.)
> **If you are a new Manager reading this: start here. Then read ARCHITECTURE.md, ROADMAP.md, DECISIONS.md, and TASKS.md in that order. TOKENOMICS.md is being redesigned — do not use it as reference until Ahmad returns from advisor session.**
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
| Backend Builder | Idle — waiting for Ahmad's return from advisor session |
| Frontend Builder | Idle — waiting for Ahmad's return from advisor session |
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
| TOKENOMICS.md is being redesigned — do not reference until Ahmad returns | Session 21 |
| Frontend GitHub repo needs cleanup of internal workspace files | Session 21 |
| Developer API limits set by Ahmad via admin panel only | Session 21 |
| Private sale domain: /whitelist on existing Vercel project | Session 21 |

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

## OTI Economics — PENDING ADVISOR SESSION

**TOKENOMICS.md is being completely redesigned.** Ahmad has consulted an advisor. When Ahmad returns:

1. Full docs vs. reality review — every document checked against what is actually live
2. OTI Economics redesign — total token supply (30M was incorrect), full allocation, vesting architecture, revenue distribution, regulatory framing, token utility list

**What is confirmed so far:**
- Fixed supply (no inflation, no post-launch mint) — exact number TBD
- Whitelist allocation: 25% of total supply (covers all whitelist program events)
- All vesting/lockup percentages: configurable via admin dashboard, not hardcoded
- Off-chain referral tracking
- Commissions paid in OTI token
- Token has genuine utility (not speculative) — see OTI utilities list below

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

**TOKENOMICS.md has been rewritten as "OTI Economics" — July 29, 2026. Total supply confirmed: 35,000,000 OTI. Full allocation breakdown and revenue distribution documented. Correct file to reference going forward.**

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
| Phase 0 (NEW) — Ecosystem Whitelist Infrastructure | NEXT — pending Ahmad return from advisor session |
| XMTP Campaign (Tasks 19–22) | ONGOING PROGRAM — runs when funded, tasks written and ready |
| Phase 2B — Post-Campaign Remaining | Planned |
| Phase 3 — Monetization + OTI Token full ecosystem | Planned |
| Phase 4 — Growth Features | Planned |
| Phase 5 — Distribution (bots, widget, extension) | Planned |

---

## What to Build Next (Full List — in order, when Ahmad returns)

**1. Frontend GitHub Repo Cleanup (Frontend Builder)**
Strip all internal workspace files from the public frontend repo. No TASKS.md, FIXES.md, ARCHITECTURE.md, BUILDER_ONBOARDING.md, or any internal documentation. The repo must show only actual source code and look professional.

**2. Docusaurus Docs Site Audit (Frontend Builder)**
Check for any sensitive internal workspace information exposed in the public developer docs. Differentiate between: (a) public information for transparency — API reference, supported chains, rate limits, code examples; (b) private information that must not be online — key rotation strategy, admin route structure, internal architecture details, bug history. Strip everything in category (b). Update chain counts. D32 standard throughout.

**3. Whitepaper Rewrite (Frontend Builder)**
Single merged task combining: (a) the original whitepaper update task + (b) the previous Manager's additions draft (in `docs/whitepaper-additions-draft.md`). Must reflect: new whitelist framing (not presale), accurate chain count (12 working), five-signal methodology, WOR, BAS attestation layer, OTI Economics (once designed), roadmap. D32 standard. Chain table in the additions draft is stale — Fantom/Scroll/Sepolia/Holesky entries must be corrected before use.

**4. Privacy Policy + Terms & Conditions (Frontend Builder)**
New pages at `/privacy` and `/terms`. Use the verbatim text provided by Ahmad in the Whitelist section above. Delete the old presale versions entirely.

**5. Whitelist Page + System (Frontend Builder + Backend Builder)**
Full spec is in the "New Direction" section above. Frontend: /whitelist page with entry gate, live progress counter, invite code input, terms checkbox. Backend: new DB tables, `/api/verify-invite` endpoint, batch code generator, code management API. Admin dashboard: batch generator UI, code management panel, ban toggle, metric override input. Smart contracts on BNB Chain: vesting parameters all admin-configurable, testnet first.

---

## Standing Rules Every Session

1. **D16 evidence standard:** Builder's "verified" is not evidence. Always ask: which wallet address, which raw curl/psql output, was it a fresh call or cached.
2. **One active task per Builder at a time.** Queue next task only after Ahmad confirms current one live.
3. **Builder never marks complete themselves.** Manager tells Builder to mark complete only after Ahmad confirms live.
4. **Builder file copies never auto-sync.** Manager must explicitly tell each Builder to update their own TASKS.md/FIXES.md copy every time status changes.
5. **Before ending any session:** Update this file and TASKS.md. If a fix was closed, update FIXES.md. Never close a session with stale docs.
6. **Fixes never get task numbers.** FIXES.md only. BF## for backend, FF## for frontend.
7. **Next BF number: BF42. Next FF number: FF28. Next Task number: Task 23**
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
- **TOKENOMICS.md is being redesigned.** Do not reference the old file for any token-related decisions until Ahmad returns with advisor output.
- **All vocabulary in public-facing content must use whitelist framing** — no "sale", "invest", "ROI", "yield", "investor" language anywhere.

---

## The Replit Multi-Account System

Ahmad uses multiple Replit free-tier accounts:
- **Manager account** — docs, prompts, roadmap (this workspace)
- **Frontend Builder account** — React/Vite on Vercel
- **Backend Builder account** — Node.js/Express on Railway

When credits exhaust, Ahmad pushes to GitHub via Replit Git, opens a new account, and the new Manager reads this file to continue.

**Doc files (all in extracted_docs/docs/ in this workspace):**
- `MANAGER_HANDOVER.md` — this file, start here
- `ARCHITECTURE.md` — what every piece of the codebase is
- `ROADMAP.md` — all planned features with full specs (needs update — whitelist framing)
- `TASKS.md` — master task list (Tasks 19–22 written and ready; new whitelist tasks to be added when Ahmad returns)
- `FIXES.md` — all bug fixes by Builder (BF41 open — Sui broken)
- `DECISIONS.md` — why things exist the way they do
- `BUSINESS_MODEL.md` — revenue layers (needs update — whitelist framing)
- `TOKENOMICS.md` — **Rewritten July 29, 2026 as "OTI Economics." 35M supply, full allocation and revenue distribution documented. Use this version.**
- `docs/whitepaper-additions-draft.md` — previous Manager's whitepaper additions (chain table is stale, merge with whitepaper task)

---

## Critical Context That Must Never Be Lost

1. **Ahmad is in an advisor session.** When he returns: full docs vs. reality review, then OTI Economics redesign, then write all task prompts.
2. **Whitelist not presale — everywhere.** Every piece of public content, every task prompt, every piece of code the Builders write must use the correct vocabulary.
3. **Frontend GitHub repo has internal files in it** from Builder pushes. Must be cleaned before whitelist launches.
4. **TOKENOMICS.md is being redesigned.** Total supply is not 30M. Do not reference old tokenomics.
5. **Anonymous rate limit already removed** by Ahmad via admin panel.
6. **XMTP campaign is an ongoing program**, not a one-off, not ranked second. Runs when funded. Task prompts 19–22 are written and ready.
7. **All vesting/lockup parameters must be admin-configurable** — this is a hard requirement, not a preference.
8. **Docusaurus audit required** before whitelist launches — sensitive internal info may be exposed in public developer docs.
9. **Task numbering:** Tasks 8–18 complete. Tasks 19–22 = XMTP Campaign (ready, waiting for funding). New whitelist tasks start at Task 23.
10. **Next session starts with:** Ahmad shares advisor output on OTI Economics → full docs vs. reality review → update all files → write task prompts → assign first Builder.
