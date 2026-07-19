# OTI — Manager Handover Document
> Last updated: July 19, 2026 (session 17 — Phase 1 fully closed. Phase 2B Revenue Campaign designed (Tasks 19–22). Etherscan key rotation strategy locked (D25, 10-key ceiling). BAS schema registration added as missing step. Campaign-first Phase 2B approach documented (D27). All five docs updated with July 17 + July 19 session content. New Manager being onboarded now.)
> **If you are a new Manager reading this: start here. Then read ARCHITECTURE.md, ROADMAP.md, TASKS.md, FIXES.md, and DECISIONS.md in that order.**
> **⚠️ D16 (evidence rule): no signal value or test result may be estimated or guessed — only real on-chain data. A Builder's "verified" claim is NOT evidence. Ask: which wallet, which raw API response, which psql output.**
> **⚠️ Read TOKENOMICS.md before touching anything token-related — price/liquidity sections deliberately removed at Ahmad's request. Do not add them back.**

---

## ⚠️ Where the Code Lives

This Replit workspace is the **Manager's workspace** — documentation, prompt writing, roadmap management ONLY. No OTI source code lives here.

- **Backend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI- — PRIVATE
- **Frontend repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-O-T-I-Frontend- — PUBLIC
- **Docs repo:** https://github.com/openflow-OTI/OpenFlow-Trust-Infrastructure-OTI-Docs — this workspace

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
| Backend Builder | ✅ Idle — waiting for Task 19 prompt |
| Frontend Builder | ✅ Idle — waiting for Task 22 prompt (after Task 21 is done) |
| Development Manager | Being replaced — you are the new Manager |

---

## Current Production State (July 19, 2026)

**Live and working:**
- Backend: `https://workspaceapi-server-production-5c0c.up.railway.app`
- Frontend: `https://otiscore.vercel.app` (`/`, `/score`, `/whitepaper`, `/admin`, `/services`, `/register`, `/report`)
- Developer docs: `https://otiscore.vercel.app/docs/`
- 12 chains live (7 EVM via Etherscan V2 + TON, Solana, Sui, Bitcoin, Tron)
- BSC/Base/Optimism intentionally return 503 (Etherscan Lite $49/mo — Ahmad's decision)
- Two-tier cache: L1 LRU (500 entries, 5-min TTL) + L2 chain_scores DB (30-day rescore window)
- Keep-highest write logic on chain_scores
- API key + quota system live
- Admin panel fully secured (x-admin-secret header, adminAuth.ts middleware)
- WOR (Wallet Ownership Registry) fully live — /register, /report, admin WOR tab
- compromised_wallets is single source of truth for all flagged-wallet views
- /services hub live
- Score sharing PNG cards live
- All fixes: BF1–BF40 ✅, FF1–FF27 ✅
- All tasks: Task 8–18 ✅
- **Phase 1: FULLY CLOSED. Phase 2 (WOR): FULLY LIVE.**

**Critical infrastructure note — Railway migrations:**
Railway does NOT auto-run `drizzle-kit push`. Every schema change needs Ahmad to manually run it against the Railway production DATABASE_URL after deploy. Run from: `cd /app/lib/db` then `drizzle-kit push`. (Note: railway.json now has this in the build step — verify if live in prod before assuming it's automatic.)

**Critical infrastructure note — subscriptions table:**
Real columns: `id, api_key, plan, owner_address, created_at, expires_at, updated_at`. NO `status` column, NO `email` column. Use raw SQL on this table only — never Drizzle ORM selects.

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
| BSC/Base/Optimism return 503 | Waiting for Etherscan Lite decision |
| Admin page is URL-only, no nav link | Ahmad's decision |
| WOR self-reports are automated — no admin review queue | Ahmad's decision |
| compromised_wallets is sole source of truth for flagged wallets | BF38/39/40 lesson |
| One task per Builder at a time | Ahmad's hard rule |
| Fixes never get task numbers — FIXES.md only | Ahmad's explicit correction |
| Price/liquidity absent from TOKENOMICS.md | Deliberately removed by Ahmad — do not add back |
| DECISIONS.md is Manager-write, Builder-read | Ahmad's direction |
| Homepage at / stays unchanged | Ahmad's decision July 15, 2026 |
| OTI token is independent from FLOW | Separate tokenomics, separate fundraising |
| Contract addresses scored same as any wallet | No address-type gatekeeping |
| Etherscan key rotation: max 10 free keys | ToS boundary — see D25 |
| ETH scores used for BNB campaign — BSC blocker bypassed | D26 — same 0x address across EVM chains |
| Campaign-first Phase 2B — revenue before full stack | D27 — fund Post-Campaign Remaining with proceeds |

---

## Phase Status

| Phase | Status |
|---|---|
| Phase 1 — Foundation | ✅ COMPLETE |
| Phase 2 — WOR | ✅ COMPLETE |
| Phase 2B — Revenue Campaign (Tasks 19–22) | 🔴 NEXT — Task 19 ready to assign |
| Phase 2B — Post-Campaign Remaining | ⏳ After campaign revenue is in |
| Phase 3 — Monetization + OTI Token | ⏳ Planned |
| Phase 4 — Growth Features | ⏳ Planned |
| Phase 5 — Distribution (bots, widget, extension) | ⏳ Planned |

---

## WHERE TO CONTINUE — READ THIS FIRST

### The Current Mission: Phase 2B Revenue Campaign

Ahmad needs money to continue building and running servers. The plan: run an XMTP campaign targeting Ethereum wallets with score ≥ 75, ask them to pay $1 in BNB to mint an OTI Trust Attestation on BNB Chain via BAS. Projected revenue: $1,000–$5,000 from first campaign. Total cost: $7–25.

**The key insight (D26):** No BSC scoring needed. Every EVM 0x address is the same person on Ethereum AND BNB Chain. OTI already scores on Ethereum (live). Payment collected on BNB Chain (cheap gas). The $49/mo Etherscan Lite blocker does not apply to this campaign.

**The 4 components, in strict dependency order:**

```
Task 19 (Backend)  →  Task 20 (Backend)  →  Task 21 (Backend)  →  Task 22 (Frontend)
Key Rotation           Sign Endpoint +        Smart Contract +       Conversion
~1 hour                BAS Schema Reg         XMTP Sender Script     Dashboard
                       ~2 hours               ~5 hours               ~2 hours
```

### Task 19 is next. Ahmad must provide these before it starts:

**Ahmad's actions before Task 19:**
1. Register 10 Etherscan accounts (separate emails, not all in one session — ToS risk). Get 10 API keys. Pass them to Manager as a comma-separated list.
2. Manager adds them to Railway as `ETHERSCAN_API_KEYS=key1,key2,...,key10`
3. Then send Task 19 prompt to Backend Builder

**Ahmad's actions before Task 20:**
1. Generate an ETH key pair (Ahmad stores the private key — never shares it with Builder). Provide the PUBLIC key to Manager for the smart contract.
2. Have ~$0.01 BNB for BAS schema registration gas + ~$20 BNB for smart contract mainnet deployment.

**Ahmad's actions before Task 21:**
1. Have a funded BNB Chain wallet (~$20 in BNB) to deploy the smart contract and receive revenue
2. Have Coinbase Wallet on mobile for testing the receive flow
3. Have BNB testnet funds from faucet.bnbchain.org for testnet testing
4. Decide on the XMTP message copy (exact text user sees in Coinbase Wallet inbox)

**Full task prompts are written and ready in TASKS.md — Tasks 19, 20, 21, 22.**

---

## Next Manager Actions (In Order)

1. **Wait for Ahmad to provide the 10 Etherscan API keys.** When he does, add them to Railway as `ETHERSCAN_API_KEYS` and then send the Task 19 prompt (from TASKS.md) to the Backend Builder.

2. **After Task 19 is verified live by Ahmad:** Send Task 20 prompt. Ahmad must have his ETH signing key pair ready before this step.

3. **After Task 20 is verified live by Ahmad:** Send Task 21 prompt. Ahmad must have BNB Chain wallet funded and Coinbase Wallet on mobile ready.

4. **After Task 21 smart contract is on mainnet and verified:** Send Task 22 prompt to Frontend Builder (can overlap with XMTP sender script if Ahmad wants the dashboard ready before the campaign launches).

5. **After Tasks 19–22 all confirmed live:** Ahmad launches the XMTP campaign. Monitor conversion dashboard. Use revenue to fund Phase 2B Post-Campaign Remaining.

---

## Phase 2B Post-Campaign Remaining (build after campaign revenue is in)

Full list in ROADMAP.md. Summary:
- Score Source Switcher (admin panel: OTI DB / BAS Attestation / Auto)
- `GET /v1/badge/:wallet` — widget API endpoint
- Embeddable JS widget (3-state: pill → hover tooltip → expanded panel)
- Attestation claim flow inside widget (no redirect)
- Proactive background scoring pipeline (feeds airdrop eligibility list — D22, must track from day one)
- `wallet_attestations` DB table
- Full BAS SDK integration in backend
- Public `/verify/:address` page
- Five-tier badge visuals (design finalized July 17, 2026 — all specs in ROADMAP.md)
- Admin panel: attestation stats, Score Source selector, fee settings

---

## Standing Rules Every Session

1. **D16 evidence standard:** Builder's "verified" is not evidence. Always ask: which wallet address, which raw curl/psql output, was it a fresh call or cached. No exceptions.
2. **One active task per Builder at a time.** Queue next task only after Ahmad confirms current one live.
3. **Builder never marks ✅ themselves.** Manager tells Builder to mark ✅ only after Ahmad confirms live.
4. **Builder file copies never auto-sync.** Manager must explicitly tell each Builder to update their own TASKS.md/FIXES.md copy every time status changes.
5. **Before ending any session:** Update this file's "Current Production State" and "Next Manager Actions". Update TASKS.md active queue. If a fix was closed, update FIXES.md. Never close a session with stale docs.
6. **Fixes never get task numbers.** FIXES.md only. BF## for backend, FF## for frontend.
7. **Next BF number: BF41. Next FF number: FF28. Next Task number: Task 23** (Tasks 19–22 are already assigned to the campaign).

---

## Key Lessons — Carry Forward Every Session

- **WalletConnect challenge TTL = 15 minutes.** Full sign flow must complete within this window or it silently hits the 400 branch.
- **compromised_wallets is the single source of truth.** Score endpoint, admin WOR Compromised view, dashboard stats — ALL must query this table. Never split across compromised_wallets and wallet_ownership.status. BF38/39/40 all caused by this.
- **Railway migrations don't auto-run.** Ahmad manually runs drizzle-kit push after every schema change.
- **Builder onboarding gap.** New Builder starts with zero API keys/secrets. Re-add all of them. Always full onboarding before resuming work.
- **XMTP fees are $0 now — run campaign before they activate.** When mainnet fees kick in (~$50–100 per 1M messages), Campaign 2 economics change. First campaign should run ASAP while it's free.
- **BAS schema UID must be registered before smart contract is written** (Task 20 Part A). This is an on-chain transaction — Ahmad signs it. The resulting schema UID is hardcoded into the smart contract. Missing this step = can't write the contract.
- **Main app uses npm. oti-docs/ uses pnpm.** Never mix.

---

## The Replit Multi-Account System

Ahmad uses multiple Replit free-tier accounts:
- **Manager account** — docs, prompts, roadmap
- **Frontend Builder account** — React/Vite on Vercel
- **Backend Builder account** — Node.js/Express on Railway

When credits exhaust → Ahmad pushes to GitHub via Replit Git → opens new account → handover → continue. All context lives in the GitHub docs — never in chat memory.

**Doc files (all in extracted_docs/docs/ in this workspace):**
- `MANAGER_HANDOVER.md` — this file, start here
- `ARCHITECTURE.md` — what every piece of the codebase is
- `ROADMAP.md` — all planned features with full specs
- `TASKS.md` — master task list with full Builder prompts (Tasks 19–22 are written and ready)
- `FIXES.md` — all bug fixes by Builder
- `DECISIONS.md` — why things exist the way they do
- `BUSINESS_MODEL.md` — revenue layers including campaign (Layer 0)
- `TOKENOMICS.md` — OTI token (30M fixed supply, BSC first, independent of FLOW)

---

## Critical Context That Must Never Be Lost

1. **Phase 2B campaign uses Ethereum scores for BNB Chain campaign.** Same 0x address = same person on both chains. No BSC Etherscan Lite subscription needed. This is the core insight that makes the campaign viable now.
2. **Etherscan key rotation: 10 keys maximum.** Above 10 = ToS violation risk. Scale beyond 10 = Etherscan Standard plan ($199/mo). See D25.
3. **OTI signing private key is Ahmad's.** Builder generates the key pair, Ahmad stores the private key (never shares it), Builder gets the public key for the smart contract. Private key lives in Railway env vars only.
4. **BAS schema UID** must be registered before Task 21 smart contract is written. Ahmad pays the gas (~$0.01 BNB).
5. **XMTP penetration is 5–15% of all EVM wallets.** Real send list from 3M scored eligibles = ~150K–450K wallets. Revenue at 0.25% conversion on 200K sends = ~$500. At 0.5% on 400K = ~$2,000.
6. **Airdrop eligibility tracking (D22):** The first 1M wallets in OTI's scoring database receive OTI tokens. Counter starts when proactive background scoring begins (Phase 2B Post-Campaign). Tracking list must be built into the background scorer from day one — cannot be retroactively reconstructed.
7. **Score Source Switcher (D23):** Widget data source (OTI DB / BAS / Auto) is a server-side system_settings value — never hardcoded in embed code. One admin panel change propagates to all embedded widgets worldwide within 60 seconds.
8. **Widget embed (D24): 4 hard constraints** — zero external dependencies, scoped styles, graceful empty state (renders nothing if no score), no redirect on any interaction.
9. **Partner Revenue Share:** Partners earn % of attestation fees from their widget. Attribution tracking is required from Phase 3 day one — cannot be retrofitted.
10. **Task numbering:** Tasks 8–18 complete. Tasks 19–22 = Phase 2B Campaign (prompts written). Next new task = Task 23.
