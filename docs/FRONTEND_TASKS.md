# OTI — Frontend Builder Task List
> Last updated: July 30, 2026 (session 22 — Phase 0 Ecosystem Whitelist Infrastructure tasks added: Tasks 23–27. OTI Economics confirmed: 35M supply, dual bonding curve DCS + ERP parameters confirmed.) | Maintained by: Development Manager
> **This file contains your tasks only. Read BUILDER_ONBOARDING.md, ARCHITECTURE.md, DECISIONS.md, and TOKENOMICS.md before starting anything here.**
> **`DECISIONS.md` documents why certain things exist the way they do — read it before touching any chain logic, scoring display, or address validation code. You never update DECISIONS.md yourself.**
> Build in the exact order listed. Some tasks have hard dependencies — do not start them until the dependency is confirmed merged and deployed.
>
> **⚠️ This is YOUR copy of this file, in YOUR own account/repo.** The Manager's copy of this file is separate and does not sync with yours. Only mark a task done here, or add a new task here, when the Manager explicitly tells you to.

---

## Where You Stand Right Now

**As of July 30, 2026 — you have NO active task.** Phase 0 task prompts (Tasks 23–27) are fully written and queued below. **Wait for the Manager to send you Task 23 explicitly before starting anything.** Do not begin any task on your own initiative.

---

## ✅ Your Completed Tasks (genuine new-build work)

### TASK 8 — Professional Results Page Redesign ✅
**Completed:** July 7, 2026. Full visual rebuild of the wallet score results page: score panel in its own bordered card with a chain-color ring gauge, score tier label (HIGHLY TRUSTED → HIGH RISK), a separate "Trust Signals" card with weighted signal bars, truncated wallet address with copy button, native-share-sheet Share button, 3× PNG "Save as Image" export, "⚑ Report this wallet" placeholder ghost link (for WOR, Phase 2), and a footer. This task also established the OTI color system below — every later task must use it.

### TASK 8B — Professional Wallet Input Page Redesign ✅
**Completed:** July 8, 2026. Matching redesign of the landing/input screen: logo sizing and position, gradient wordmark, tagline, mint-glow form card, styled rate-limit badge, "Try an example →" link, WOR ghost links, footer, faint watermark, plus a polish round fixing zkSync/Linea icon visibility, chain icon/dropdown sizing, and mobile vertical spacing.

### TASK 9 — Admin Panel UI ✅
**Completed:** July 7, 2026. Built the `/admin` route (URL-only, no public nav link, per Ahmad's decision): password gate, Dashboard, API Keys (create/list/edit/delete, reveal-once creation modal), Query History, Cache, and Plan Configs tabs. All calls authenticate with `x-admin-secret` from sessionStorage.

### TASK 11A — Restructure Vercel App: Marketing Front Door + Scoring at /score ✅
**Completed:** July 8, 2026. Moved the scoring tool intact to `/score` and built a full marketing homepage at `/`: navbar, hero with chain row, How It Works (3 steps), Trust Signals (5 cards), Use Cases (9-tile grid), Get the API (cURL example), Find Us/Integrations row, footer with social links. Crisp.chat live-chat widget embedded in `<head>` (site ID left blank until Ahmad provides it). Brand consistency with `/score` verified via cache-busted screenshot and JS bundle inspection by the Manager, and directly by Ahmad.

### TASK 11B — Whitepaper Page ✅
**Completed:** July 8, 2026 (build), fixes verified July 8–9, 2026. Built `/whitepaper`: full section-by-section technical/business document, sticky TOC sidebar on desktop (accordion on mobile), mint section numbers, print-to-PDF via `window.print()`, shared navbar/footer/color system with the homepage. Three post-launch rendering fixes (body text color, mobile horizontal scroll, Roadmap section removal) are logged in `FIXES.md` FF16.

### TASK 11 — Developer Docs Site ✅
**Completed:** July 9, 2026. Docusaurus site (`oti-docs/`) covering Getting Started, API Reference, Score Explanation, Supported Chains, Rate Limits, and JS/Python/cURL examples, plus an OG social-share image and a live "Try It Live" widget. Deployed as its own Vercel project (pnpm-based build — oti-docs uses pnpm; the main frontend app uses npm — do not mix them) and proxied onto the main site at `/docs/` via `vercel.json` rewrites. Confirmed fully live via curl on `/docs`, `/docs/`, and `/docs/api-reference`. One open follow-up: `FIXES.md` BF11/BF-style item — re-verify "Try It Live" hits the real Railway backend post-redeploy (tracked as BF11 in `FIXES.md`, owned by whichever Builder picks it up next — currently unassigned).

---

## 🎨 OTI Color System (Locked — July 7, 2026)

> This palette replaced the old flat black + mint system. Every future task must use these exact values. Do not revert to pure black (`#000000`) or the old surface colors.

| Token | Value | Usage |
|---|---|---|
| Background | `#05080f` | Page background — deep blue-black |
| Surface | `#0b0f1a` | Card backgrounds, panels |
| Surface-2 | `#0f1520` | Inner elements, signal bar tracks |
| Borders | `#1c2535` | All card/panel borders |
| Body text | `#e8f4ff` | Main readable text — slight blue tint |
| Dimmed text | `#7a8fa8` | Metadata, labels, secondary text |
| Mint (primary) | `#00e5a0` | CTAs, highlights, active states, mint accents |
| Mint (gradient end) | `#3EFFC1` | Gradient highlights |

**Chain brand colors (ring gauge + panel tint):** Ethereum `#627EEA` · Bitcoin `#F7931A` · Solana `#9945FF` · BNB `#F3BA2F` · (all 15 chains have their exact brand hex in the codebase — check there before inventing a new one).

**Special effects (do not remove):**
- Navbar: `backdrop-filter: blur(14px)` frosted glass
- Submit button: green glow on hover `box-shadow: 0 0 24px rgba(0,229,160,0.40)`
- Score panel: border + background tint shifts toward the selected chain's brand color via `color-mix()`
- Signal bars: 5px height, soft color glow on fill
- All `color-mix()` declarations have plain-value fallbacks above them for older browsers

---

### TASK 17 — WOR UI — Phase 2 ✅
/register (3-step: address check → MetaMask sign → passkey set) and /report (3-step: status check → sign + passkey → confirm dialog). "⚑ Report this wallet" link activated on results page. Admin WOR tab (Registry, Compromised, Manual Override). Verified live by Ahmad July 14, 2026. Follow-up polish tracked as FF24.

### TASK 18 — Services Hub Page (`/services`) ✅ — July 15, 2026
New page at `/services`. Built with MarketingNavbar/Footer chrome, 2-col→1-col (≤720px) card grid. Ahmad confirmed live July 15, 2026. Closed.

---

## 🔴 Your Task Queue — Phase 0 Ecosystem Whitelist Infrastructure

Build in strict order. Do not start a task until the Manager explicitly assigns it and the previous task is confirmed live by Ahmad. One task at a time.

---

### TASK 23 — GitHub Repo Cleanup
**Priority: FIRST — start when Manager assigns**

Strip all internal workspace files from the public frontend GitHub repo. The repo must contain only source code.

**What to remove (delete from repo root and any subdirectory):**
- `TASKS.md`, `FIXES.md`, `ARCHITECTURE.md`, `BUILDER_ONBOARDING.md`, `DECISIONS.md`, `ROADMAP.md`, `MANAGER_HANDOVER.md`, `TOKENOMICS.md`, `BUSINESS_MODEL.md`
- Any `.md` file that is not a `README.md`
- Any `docs/` folder containing internal planning files

**What to keep:**
- All `.tsx`, `.ts`, `.css`, `.json`, `.html`, `.js` source files
- `package.json`, `package-lock.json`, `tsconfig.json`, `vite.config.ts`, all build config
- `vercel.json` — **do NOT touch, ever**
- `public/` folder, `src/` folder
- A clean `README.md` — if none exists, create one: project name, one-line description, live URL (`https://otiscore.vercel.app`), tech stack only. No internal details.

**How to do it:** Delete files locally in your Replit workspace, push the deletion commit. Ahmad handles the GitHub merge.

**Evidence required to close:** List the files you deleted and confirm the push. Ahmad reviews the repo. Manager closes after confirmation.

---

### TASK 24 — Docusaurus Docs Site Audit
**Priority: SECOND | Depends on: Task 23 confirmed merged**

Audit the live developer docs at `https://otiscore.vercel.app/docs/` and remove any sensitive internal information.

**Keep (public developer-facing):**
- Getting Started guide, API Reference, Supported Chains, Rate Limits, code examples (JS/Python/cURL), Score Explanation, Try It Live widget

**Remove:**
- Any mention of Etherscan key rotation strategy or how many keys OTI uses
- Any mention of admin routes, `/api/admin/`, or the `x-admin-secret` pattern
- Any mention of Railway, the backend host URL, or internal architecture decisions
- Any signal weights or scoring algorithm internals
- Bug fix history, version changelogs, internal known issues
- Any "presale", "private sale", "invest", "ROI", "yield" language

**Chain count correction:** Update any chain count to **12**. The 12 live chains: Ethereum, Bitcoin, Solana, BNB Chain, Polygon, Arbitrum, Optimism, Base, Avalanche, Tron, zkSync, Linea. Do not list Sui as live (BF41 — offline). Remove Fantom, Scroll, Sepolia, Holesky if they appear anywhere.

**D32 standard:** Rewrite any sentence that sounds AI-generated. Plain, direct English only. No "robust", "seamless", "harness", "leverage", "delve."

**Evidence required to close:** List every doc page edited and what was removed. Manager spot-checks the live URL.

---

### TASK 25 — Whitepaper Rewrite
**Priority: THIRD | Depends on: Task 24 confirmed complete**

Rewrite `/whitepaper` by merging the existing page with `docs/whitepaper-additions-draft.md`. One authoritative document that reflects OTI as it exists today.

**Before using the additions draft — fix these stale items:**
- Remove Fantom, Scroll, Sepolia, Holesky from any chain table. Only 12 live chains (same list as Task 24 above).
- Replace all "presale/invest/ROI/yield/private sale" language with whitelist vocabulary (table below).
- Do not list Sui as live.

**Vocabulary enforcement (mandatory everywhere):**
| OLD — banned | NEW — required |
|---|---|
| Token Sale / Private Sale / Presale | Ecosystem Whitelist / Node Testing Program |
| Buy Tokens / Invest | Acquire Network Access Fuel / Claim Allocation |
| Staking Payouts / ROI / Yield | Node Collateral Lockup / Linear Network Vesting |
| Investors | Whitelisted Operators / Community Contributors |
| Trading / Listing | Public Utility Liquidity Pool Seeding |

**Required sections:**
1. Executive Summary
2. The Problem
3. The Solution
4. How It Works — wallet score flow, chain support, cache architecture
5. Trust Signals — five signals described (do not reveal exact weights)
6. Wallet Ownership Registry (WOR)
7. OTI Economics — use TOKENOMICS.md figures only: 35M supply, allocation table, DCS + ERP summary, revenue distribution, utility list. Do not invent numbers.
8. Ecosystem Whitelist Program — Genesis Mode, DCS/ERP overview, invite-only gate
9. Roadmap — high-level phases only, no internal task numbers
10. Legal Disclaimer — utility token disclaimer, geographic restrictions

**D32 standard:** Every sentence must read like a human wrote it. No AI tells.

**Evidence required to close:** Builder confirms the page is live. Ahmad reads it, confirms no presale vocabulary and all economics figures match TOKENOMICS.md.

---

### TASK 26 — Privacy Policy + Terms & Conditions Pages
**Priority: FOURTH | Depends on: Task 25 confirmed complete**

Build two new pages at `/privacy` and `/terms`. Use the verbatim text below — do not rewrite, paraphrase, or "improve" any of it.

**Terms & Conditions — verbatim content for `/terms`:**

> OTI Ecosystem Whitelist — Terms & Conditions
>
> 1. PURPOSE OF THE WHITELIST: The OTI Whitelist Token Onboarding Program is strictly built to distribute network utility vouchers (Access Fuel) to future network testers, node operators, and B2B developers.
>
> 2. NO EXPECTATION OF PROFIT: Participants explicitly acknowledge that OTI tokens are utility tools used for wallet attestation fees and API queries. This program is not an investment, security, or financial contract. There is zero promise of future financial returns, passive yield, or profit.
>
> 3. PROTOCOL LOCKUP & VESTING: By accessing this software, the user agrees to the automatic 75% Node Collateral Lockup. Tokens will release linearly on a daily schedule to maintain network stability and protect circulating supply from systemic spamming.
>
> 4. GEOGRAPHIC RESTRICTIONS: This program is prohibited to residents, citizens, or IP addresses originating from high-risk or strictly regulated jurisdictions, including but not limited to the United States of America, China, and sanctioned nations.
>
> 5. ADMINISTRATIVE RIGHTS: The OTI core administration team reserves the absolute right to delete, freeze, or ban any invite code or wallet address found to be operating maliciously or misrepresenting the technical nature of the protocol.

**Privacy Policy — verbatim content for `/privacy`:**

> OTI Ecosystem Whitelist — Privacy Policy
>
> 1. DATA COLLECTION PRINCIPLES: The OTI platform operates under true Web3 data minimization protocols. We do not collect, request, or store your real name, physical address, phone number, or government identity documentation.
>
> 2. TYPES OF DATA LOGGED: The system strictly logs decentralized interaction points: (a) Public crypto wallet addresses used to claim allocations, (b) Validated single-use admin invite codes, and (c) Basic on-chain transaction hashes.
>
> 3. COOKIES & TRACKING: We do not deploy advertising tracker cookies or pixel trackers. Local device session storage may be temporarily used to verify your password token state for active sessions.
>
> 4. THIRD-PARTY DISCLOSURE: No collected Web3 identifier data is sold, rented, or passed to corporate advertisers. Data remains siloed in decentralized server instances solely used to manage active whitelist states.

**Page requirements:**
- Locked OTI color system — same navbar/footer as every other page
- Fully readable on mobile (375px)
- Add `/privacy` and `/terms` links to the site footer now (even before /whitelist is live)
- Clean, simple layout: page title, numbered sections, body text in `#e8f4ff`. No decorative elements — this is a legal page.

**Evidence required to close:** Both pages live at `/privacy` and `/terms`. Footer links verified. Ahmad confirms content is verbatim.

---

### TASK 27 — /whitelist Page
**Priority: FIFTH | Depends on: Task 26 confirmed complete AND Task 28 backend endpoints live on Railway**
**⚠️ Do not start the invite-code verification flow until Manager confirms Task 28's `/api/verify-invite` is live. You can build the entry gate static structure before that.**

Build the `/whitelist` page — the Ecosystem Whitelist Node Program portal.

**State 1 — Unauthenticated Gate (default for all visitors):**
- Professional locked security panel. No wallet connect. No token charts. No contract addresses visible.
- Header: "OTI Infrastructure Hub — Private Whitelist Node Platform"
- Subtext: "Access is restricted to whitelisted node operators and infrastructure partners. If you have received an invite code from the OTI team, enter it below."
- Invite code input field (hint: OTI-XXXX-XXXX)
- Mandatory checkbox: "I have read and agree to the [Terms & Conditions](/terms) and [Privacy Policy](/privacy) of this program." — links open in new tab. Button disabled until checked.
- "Request Access" button — calls `POST /api/verify-invite`. Inline error on invalid/used code: "This code is invalid or has already been used."

**State 2 — Authenticated Portal (after valid code + terms accepted):**

**Live DCS counter (top, prominent):**
- "Current Contribution Rate: $X.XXXXXX per OTI" — live, from `GET /api/whitelist/state`
- Progress bar toward $25,000 total DCS target
- Sub-label: "Rate increases as allocation fills — early contributors receive more OTI per dollar"

**Live ERP counter (directly below DCS, prominent):**
- "Current Referral Bonus: X,XXX OTI" — live, from same endpoint
- Sub-label: "Referral bonus decreases as DCS fills — claim early for maximum referral value"
- Both counters update simultaneously every 30 seconds: DCS rate goes UP, ERP bonus goes DOWN. Make this visually clear — they move in opposite directions intentionally.

**Allocation claim section:**
- "Connect your wallet to claim your Node Access Allocation"
- Wallet connect button (MetaMask / WalletConnect)
- Once connected: truncated wallet address + OTI allocation amount for this invite code
- "Claim Allocation" button

**Ecosystem Rewards section:**
- Three reward cards: Referral (X,XXX OTI — live ERP rate), Social Post/Tag (1,000 OTI), Share+Follow (500 OTI each)
- All social rewards: "Credited after manual admin review — typically within 48 hours"

**Vesting summary:**
- "75% Node Collateral Lockup — releases linearly on a daily schedule"
- "25% immediately accessible as Access Fuel"
- Values pulled from backend config — never hardcoded in the frontend

**Design:** Locked OTI color system, mint accents, fully mobile-responsive (375px). No AI copy. Plain direct English only.

**Navbar:** Do NOT add `/whitelist` to the navbar yet. URL-only for now, like `/admin`. Ahmad will say when to add it.

**Evidence required to close:** Ahmad tests end-to-end with a real invite code: enters code → authenticated portal → both live counters loading → wallet connect works → allocation visible. Manager verifies via screenshot.

---

## ⏳ Future Tasks (Beyond Phase 0)

- **Phase 5:** Firefox browser extension (then Chrome) — separate repo, content script that detects wallet addresses on Etherscan/OpenSea and injects OTI score badges
- **Phase 3:** Self-serve developer portal — sign up, get API key, choose plan

---

## Keeping This File Updated

You never update either file on your own initiative. Every update — whether marking a task done or adding a new one — happens only when the Manager explicitly tells you to.

**When you complete a task:**
1. Notify the Manager — do not mark anything done yourself
2. Wait for the Manager to review and confirm your work
3. Only when the Manager explicitly tells you to mark it done:
   - Move it to the ✅ Completed Tasks section in this file
   - Mark it ✅ in `TASKS.md` as well (or in `FIXES.md`, if it's a fix, not a task)
4. All your files must always match the Manager's

**When the Manager assigns you a new task:**
1. The Manager will explicitly tell you to add the new task to your file
2. Only then: add it to the bottom of your Queue section in this file AND add it to `TASKS.md`
3. Read it fully before starting — every task has a "Why" and a "Definition of done"
4. Do not start a task that has a ⚠️ dependency warning until the Manager confirms it is cleared

**General rules:**
- Never touch Git, never push, never open a PR — Ahmad handles all of that himself
- Never touch `vercel.json` — ever
- Never manually edit `src/api/schema.gen.ts` — run `npm run codegen` instead
- Main app uses **npm**. Docs site (`oti-docs/`) uses **pnpm**. Never mix them.
- Ahmad loves the chain selector — do not touch it
- All CSS goes in `src/index.css` — no new component libraries
- Test on mobile (375px) — most users are on mobile
- If a task is blocked, tell the Manager immediately — do not sit on a blocker
- Never update any file without the Manager's explicit instruction
