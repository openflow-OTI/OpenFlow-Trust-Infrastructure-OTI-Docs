# OTI — Frontend Builder Task List
> Last updated: July 7, 2026 (updated by Manager — all fixes done through Task 9C-adjacent work, Task 8 is next) | Maintained by: Development Manager
> **This file contains your tasks only. Read BUILDER_ONBOARDING.md and ARCHITECTURE.md before starting anything here.**
> Build in the exact order listed. Some tasks have hard dependencies — do not start them until the dependency is confirmed merged and deployed.

---

## ✅ Your Completed Tasks

### Task 1 — UI Polish ✅
- Homepage title, back button style, chain icon sizing, dead CSS removed, `isKnownChain()` guard in `useScore.ts`

### Task 2 — Logo Fix ✅
- `Logo.tsx` → `<img src="/logo.jpg">` replacing spiral SVG math
- `generateScoreCard.ts` → uses `loadImage()` before canvas draw

### Task 2B — Logo: SVG Replace ✅
- Original SVG logo file used directly as `public/logo.svg` — no reconstruction needed
- Crisp at 34px on Retina/high-DPI screens, zero blur
- `generateScoreCard.ts` already pointed to `/logo.svg`

### Task 7 — Signal Bars → Weighted Display ✅
- `pnpm codegen` run — `src/api/schema.gen.ts` regenerated from live backend OpenAPI spec
- `SignalBar.tsx` updated: bar fill = `(weighted / maxWeight) × 100%`, label = "weighted/maxWeight" e.g. "25/25"
- Color logic updated to use `weighted / maxWeight` ratio
- `Home.tsx` updated to pass full signal object to SignalBar
- `generateScoreCard.ts` updated — score card PNG shows weighted labels and correct fills
- Verified live against real wallet on Vercel — all 5 bars show weighted values
- Also resolved black screen crash caused by Task 5 API shape mismatch

---

### Task 7B — txCount Cap Indicator ✅
- `src/lib/formatMetadata.ts` — shows "1,000+ transactions" when txCount >= 1000
- `src/lib/generateScoreCard.ts` — same logic in score card PNG
- Verified live on Vercel by Manager

### Task 7C — Dynamic Rate Limit Display ✅
- `src/hooks/useAnonymousLimit.ts` calls public `GET /api/config/anonymous-limit`
- Homepage shows live daily_limit value from DB — confirmed showing "20 per day" (July 7, 2026)
- Falls back gracefully when API returns null

### Task 10 — Navbar API Health Status Dot ✅
- `src/components/Navbar.tsx` — green/red dot connected to `useHealth` hook
- Verified live on Vercel by Manager

### Task 9 — Admin Panel UI ✅
- `/admin` route with password gate, Dashboard, API Keys, Query History, Cache, Plan Configs tabs
- All API calls use `x-admin-secret` from sessionStorage
- Verified login working July 7, 2026

### Plan Configs Tab ✅
- New tab in admin panel — table of all 4 plans, inline edit per row
- PATCH sends `daily_limit` (snake_case) correctly
- Invalidates anonymous-limit query on save so homepage updates immediately

### Admin Panel Desktop Layout Fix ✅
- CSS `:has()` selector removes `max-width: 720px` when admin shell is present

### useAnonymousLimit Hook Fix ✅
- Hook now calls correct public endpoint instead of admin-protected route
- Fallback to 3 when null returned

### Homepage Scrollbar + Layout Fix ✅
- html/body/root use `min-height`; browser scrollbar at far right edge

### API Keys Tab — UI Resilience Fix ✅
- "+ New Key" button always visible even when keys list fails to load
- Error shows inline with Retry button; table guarded behind isSuccess
- Verified working July 7, 2026

---

## 🔴 Your Task Queue — Build In This Exact Order

---

### TASK 8 — Professional Results Page Redesign
**Phase:** 3 — Redesign
**Priority:** HIGH — MVP requirement, Ahmad's explicit request
**Depends on:** Task 7 ✅ done AND Task 2B ✅ done — this task is now unblocked

**Why you are doing this:**
The results page is the first thing a potential enterprise partner, developer, or investor sees after they score a wallet. Right now it doesn't look professional enough to represent OTI as the infrastructure business it is. Ahmad has explicitly flagged this as an MVP requirement — the product must be presentable before any distribution channel launches or any business conversation happens. This redesign does not change any logic — only the visual presentation and layout.

**What to build:**
Redesign the scoring results page (`src/pages/Home.tsx` results section and related components). Keep: the black background, the mint green color scheme, the chain selector (do not touch it), the score gauge color system. Keep the "← Score Another Wallet" button.

Required changes:

1. **Wallet address:** Truncate the middle — show `0xAb58...eC9B` format (first 6 + last 4 chars). Add a copy-to-clipboard icon button next to it.

2. **Score tier label:** Add a label below the score gauge:
   - 85–100 → "HIGHLY TRUSTED" (mint green)
   - 65–84 → "TRUSTED" (light green)
   - 45–64 → "CAUTION" (amber)
   - 25–44 → "SUSPICIOUS" (orange)
   - 0–24 → "HIGH RISK" (red)

3. **Signal section:** Add a section header "Trust Signals" above the signal bars. Each signal bar now shows `weighted/maxWeight` as the score label (from Task 7). Add visual card/panel separation between the score gauge section and the signals section.

4. **Share button:** Make it larger and more prominent — mint green border, full width or centered with min-width, not a tiny button buried at the bottom.

5. **Footer:** Add a minimal footer: "© 2026 OpenFlow Labs · openflowlabs.io"

6. **Report this wallet:** Add a small ghost link below the signals section: "⚑ Report this wallet" — it should do nothing for now (disabled or href="#") but must be present for the WOR system to connect to later.

7. **Empty space:** The large void below the share button must be eliminated. Use the footer to anchor the bottom.

Keep all CSS in `src/index.css`. Do not add a new component library. Follow existing class naming conventions.

**Definition of done:** Results page looks professional on both mobile (375px) and desktop. Wallet address is truncated. Score tier label is visible. Signal bars show weighted values. Share button is prominent. Footer exists. "Report this wallet" link is present.

---

### TASK 11A — Restructure Vercel App: Marketing Front Door + Scoring at /score
**Phase:** 4 — Pre-Distribution
**Priority:** HIGH
**Depends on:** Task 8 (results redesign done) AND Task 2B (SVG logo done)

**Why you are doing this:**
Right now, anyone who visits `otiscore.vercel.app` lands directly on the scoring tool with no context about what OTI is, why it matters, or who built it. Every bot reply, every widget badge, every browser extension popup will link back to this URL. Without a proper front door, that traffic converts to nothing — developers and partners arrive, see a form, and leave. The marketing homepage gives OTI's story, explains the product, and converts curious visitors into API users and potential business partners. The scoring tool moves to `/score` — nothing about it changes visually. One Vercel project, one URL, two purposes.

**Part A — Move the scoring tool to `/score`**

Move the current homepage (wallet address form + results) to the `/score` route. Update `vercel.json` carefully — the SPA rewrite rule `{"source":"/(.*)", "destination":"/index.html"}` must stay intact; do not remove or break it. Update any internal links that currently point to `/`.

**Part B — Build the marketing homepage at `/`**

Brand system:
- Primary mint: `#3EFFC1` (outer/bright), `#2BD9A4` (inner/deeper)
- Background: `#0A0A0A` deep black
- Typography: Geist Sans or Inter — clean, geometric, modern
- Micro-interaction: the spiral logo rotates subtly on hover (CSS `transform: rotate()`, 2–3s ease, infinite)
- All CTAs, highlights, active states: mint only

Sections to build:

**1. Navbar**
- Left: OTI spiral logo (34px) + "OTI" wordmark in white
- Right: "Score a Wallet" button (mint outline) → `/score`; "API Docs" text link → placeholder `#`
- Mobile: hamburger with same links

**2. Hero**
- Headline: "Know Who You're Transacting With"
- Sub-headline: "OTI scores any blockchain wallet 0–100 using five on-chain trust signals. Instant. Free. API-ready."
- CTAs: "Try It Free" (mint filled → `/score`), "View API Docs" (ghost → `#`)
- Faint watermark spiral behind headline (CSS only, no JS library)
- Chain row below CTAs: small chain icons for ETH, SOL, BTC, TON, MATIC, ARB, SUI, TRX + "+7 more" badge

**3. How It Works (3 steps)**
- Step 1: Enter wallet address + select chain
- Step 2: OTI analyzes 5 on-chain behavioral signals
- Step 3: Get a 0–100 trust score with full signal breakdown
- Style: numbered cards, horizontal row on desktop, vertical stack on mobile

**4. Trust Signals (5 cards)**
- 🕐 Wallet Age (25%) — "How long this wallet has been active on-chain"
- 📊 Transaction Count (20%) — "Volume and frequency of transactions"
- 🪙 Token Holding Behavior (20%) — "Diversity and quality of tokens held"
- 🔗 Smart Contract Interactions (20%) — "Depth of DeFi and protocol engagement"
- ⏱ Transaction Timing (15%) — "Consistency and naturalness of activity patterns"

**5. Use Cases (tile grid — 3 cols desktop, 2 tablet, 1 mobile)**

| Tile | Sector | Body |
|---|---|---|
| 💱 | Exchanges & Gateways | Flag compromised wallets before processing withdrawals. |
| 🏦 | DeFi Protocols | Risk-adjust lending and collateral requirements based on wallet trust. |
| 🖼 | NFT Marketplaces | Display trust badges next to seller listings. |
| 💸 | Payment Processors | Require a minimum trust score before processing. |
| 🎮 | Web3 Gaming | Prevent fresh-wallet farming in Play-to-Earn. |
| 🗳 | DAO Governance | Weight votes by wallet trust alongside token balance. |
| 🔐 | Custody Services | Score source wallets before crediting accounts. |
| 📡 | Bridges & Cross-chain | Score the source wallet before allowing a bridge transaction. |
| 🛠 | Developer Tooling | One API call. Any chain. No blockchain infrastructure required. |

**6. Get the API**
- Headline: "Free API Key. No Credit Card."
- Sub-line: "Anonymous: 3 lookups/day. Register free for higher limits."
- cURL example block (copy-pasteable)
- CTA: "Read the Docs" → `#` placeholder

**7. Find Us / Integrations**
Row of icons with labels: Telegram, Discord, Chrome Extension, Firefox Extension — all `#` placeholder links

**8. Footer**
- Left: OTI spiral logo (small) + "© 2026 OpenFlow Labs"
- Center: Score a Wallet · API Docs · GitHub (public frontend repo link) · Privacy Policy (placeholder) · Terms (placeholder)
- Right: Social icons — Twitter/X, LinkedIn, Telegram, Discord (all `#`)

**Chat support — Crisp.chat:**
Add to `index.html` `<head>`:
```html
<script>window.$crisp=[];window.CRISP_WEBSITE_ID="AHMAD_PROVIDES_THIS";</script>
<script src="https://client.crisp.chat/l.js" async></script>
```
If Ahmad has not yet provided the ID, leave `CRISP_WEBSITE_ID=""` — the widget simply won't appear until filled.

**Feedback form — Tally.so:**
Ahmad creates a form at tally.so and provides the embed snippet. Place it in a modal triggered by "Send Feedback" in the footer. If not yet provided, use `mailto:` fallback.

**Definition of done:**
- `/` shows the full marketing homepage — all 8 sections present and styled
- `/score` shows the scoring tool exactly as before (no visual changes to the tool itself)
- Navbar "Score a Wallet" navigates correctly between the two views
- Crisp script is in `<head>` (even if ID is empty placeholder)
- Responsive on mobile (375px) and desktop (1440px)

---

### TASK 11B — Whitepaper Page
**Phase:** 4 — Pre-Distribution
**Priority:** HIGH
**Depends on:** Task 11A must be merged first

**Why you are doing this:**
OTI is positioning itself as infrastructure for enterprises — exchanges, custody platforms, DeFi protocols. Those buyers do not sign contracts based on a landing page alone. They need a technical document that explains what OTI is, how it works, who built it, and what the business model is. A whitepaper also builds credibility with investors and serious developers. The `/whitepaper` route lives inside the same Vercel project — no new deployment, no new domain needed.

**Route:** `/whitepaper`

**Design rules:**
- Same `#0A0A0A` black + `#3EFFC1` mint brand system
- Body text 17–18px, line-height 1.8 — this is a long-form document
- Section numbers displayed in mint: `01`, `02`, `03`
- Sticky table of contents sidebar on desktop (collapses to "Jump to section" accordion on mobile)
- Faint spiral watermark behind page header — `opacity: 0.04`, CSS only
- "Download as PDF" button — uses `window.print()` with `@media print` stylesheet that hides navbar, sidebar, and footer

**Nav update required:**
- Desktop: `Logo | Score a Wallet | API Docs | Whitepaper | [Social icons]`
- Mobile hamburger: Score a Wallet → API Docs → Whitepaper

**Page header:**
- Title: `OpenFlow Trust Infrastructure — Whitepaper`
- Subtitle: `Version 1.0 · July 2026 · OpenFlow Labs`
- Two buttons: `[Download PDF]` and `[Score a Wallet →]`

**Content sections** (copy exactly as provided in TASKS.md — sections 01 through the end of the whitepaper content):
- 01 — Executive Summary
- 02 — About OpenFlow
- 03 — About OpenFlow Labs
- 04 — The Problem
- 05 — How OTI Works
- 06 — Supported Infrastructure
- 07 — Use Cases
- (remaining sections per TASKS.md)

**Definition of done:** `/whitepaper` renders all sections. Sticky sidebar works on desktop. Print-to-PDF hides nav and sidebar. "Score a Wallet" button navigates to `/score`.

---

## ⏳ Future Tasks (Not Yet Active — Manager Will Assign When Ready)

These are coming after the queue above is complete. You do not need to read these in detail now.

- **Phase 5:** Browser extension (Firefox first, then Chrome) — separate repo, content script that detects wallet addresses on Etherscan/OpenSea and injects OTI score badges
- **Phase 6:** Wallet Ownership Registry (WOR) UI — registration flow, compromise report flow, passkey integration
- **Phase 7:** Self-serve developer portal — sign up, get API key, choose plan

---

## Keeping This File Updated

You never update either file on your own initiative. Every update — whether marking a task done or adding a new one — happens only when the Manager explicitly tells you to.

**When you complete a task:**
1. Notify the Manager — do not mark anything done yourself
2. Wait for the Manager to review and confirm your work
3. Only when the Manager explicitly tells you to mark it done:
   - Move it to the ✅ Completed Tasks section in this file
   - Mark it ✅ in `TASKS.md` as well
4. Both files must always match

**When the Manager assigns you a new task:**
1. The Manager will explicitly tell you to add the new task to your file
2. Only then: add it to the bottom of your Queue section in this file AND add it to `TASKS.md`
3. Read it fully before starting — every task has a "Why" and a "Definition of done"
4. Do not start a task that has a ⚠️ dependency warning until the Manager confirms it is cleared

**General rules:**
- Never touch Git, never push, never open a PR — Ahmad handles all of that himself
- Never touch `vercel.json` — ever
- Never manually edit `src/api/schema.gen.ts` — run `pnpm codegen` instead
- Ahmad loves the chain selector — do not touch it
- All CSS goes in `src/index.css` — no new component libraries
- Test on mobile (375px) — most users are on mobile
- If a task is blocked, tell the Manager immediately — do not sit on a blocker
- Never update any file without the Manager's explicit instruction
