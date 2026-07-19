# OTI — Product Roadmap
> Last updated: July 19, 2026 (session 17 — Phase 2B split into Revenue Campaign (Tasks 19–22, build now) and Post-Campaign Remaining; Etherscan key rotation moved from Phase 4 to Task 19 (prerequisite for campaign); BAS schema registration added as required pre-build step; 10-key rotation strategy documented (D25); Ethereum-scores-for-BNB-campaign insight documented (D26); campaign-first Phase 2B approach documented (D27); XMTP wallet targeting reality (5–15% XMTP penetration) documented) | Maintained by: Development Manager
> Source document: OTI Full Distribution & Technical Development Strategy (Founder's Playbook, July 2026)

---

## The Core Thesis (from the Playbook)
> "OTI is not a wallet checker website. It is on-chain behavioral intelligence as a service — the infrastructure layer for trust in Web3. The website is the demo. The API is the product."

**Who buys this:**
- 🏦 Exchanges & Gateways — screen withdrawal destinations, detect compromised wallets (enterprise sale, highest value)
- ⚡ DeFi Protocols — risk-adjust lending/collateral based on wallet trust (API integration)
- 🖼️ NFT Marketplaces — seller trust badges, filter bad actors (API integration)
- 💸 Payment Processors — gate crypto payments behind minimum trust score (API integration)

**Three goals (from Playbook):**
- 🎯 90 days after distribution launches: 1,000 active API users via bots + widget
- 🏗️ 6 months: 3 paying business API integrations
- 🏦 12 months: first enterprise exchange contract

---

## MVP Definition

**MVP = the core product is secure, correct, and presentable to potential partners.**

MVP is complete when:
1. ⚠️ Chain scoring — real working count is currently 12, not 15. Scroll/Sepolia/Holesky not implemented (BF23); Fantom points to wrong network (BF22). EVM chains also have pagination and data-accuracy gaps (BF24–BF27). Full diagnostic in FIXES.md. Must be resolved before Phase 1 closes.
2. ✅ API key + quota system working
3. ✅ Score sharing (PNG card) working
4. ✅ Admin endpoints secured
5. ✅ Score history served from database, not memory
6. ✅ Signal scores return weighted values in API response
7. ✅ Signal bars display weighted contributions on frontend
8. ✅ Results page redesigned to professional standard

MVP's core build is complete. What remains before it's fully trustworthy for distribution is correctness work — tracked as fixes, not roadmap items. See `FIXES.md`.

---

## ✅ Completed Foundation Work

The original scoring engine, API key system, score sharing, security hardening, and full frontend redesign (results page + homepage, black/mint visual system) are all live in production. Every bug found and fixed along the way — from the admin auth gap to Bitcoin's wallet-age bug to the anonymous-limit cache sync issue — is logged in `FIXES.md`, not here. This roadmap only tracks what's still ahead to build.

---

## PHASE 1 — PRE-DISTRIBUTION REQUIREMENTS
**Owner: Both Builders | Status: ✅ Core pages done — one operational item and one open fix remain (see below)**

These are prerequisites. Every bot reply, widget badge, and extension popup links back to OTI. Without a front door and a docs page, that traffic converts to nothing.

### 1A — Marketing Homepage ✅ DONE
Live at `otiscore.vercel.app`. The scoring tool moved to `/score`; `/` is now a professional marketing page (Hero, How It Works, Trust Signals, Use Cases, Get the API, Find Us/Integrations, footer) sharing the same Vercel project and brand system as the scoring app.

### 1B — Developer Docs Site ✅ DONE
Docusaurus site, live at `otiscore.vercel.app/docs/` (proxied from a standalone `oti-docs` Vercel project). Covers Getting Started, API Reference (weighted signal shape), Score Explanation, Supported Chains, Rate Limits, and code examples in JS/Python/cURL.

### 1C — Whitepaper ✅ DONE
Live at `/whitepaper`, sharing the homepage's nav/footer and brand system.

### 1D — Operational Keys (Ahmad, not a Builder) — OPEN
- Create internal bot API key via admin panel (high daily limit, not the anonymous key)
- Create widget shared key with a `widget` plan tier
- CORS is already open globally — no changes needed

### Still open before this phase fully closes
- `FIXES.md` BF10 — non-EVM signal accuracy (critical, in progress with Backend Builder)
- `FIXES.md` BF11 — re-verify "Try It Live" docs widget hits the real Railway backend
- `FIXES.md` FF17 — "AI-native tell" cleanup across homepage/docs/whitepaper (open, high priority)

---

## PHASE 2 — WALLET OWNERSHIP REGISTRY (WOR)
**Owner: Backend Builder + Frontend Builder | Status: Not started — does NOT depend on Phase 1 closing, can run in parallel**

Ahmad's flagship trust feature. Users pre-register wallet ownership via off-chain EIP-191 signing + passkey. If a wallet is compromised, the owner connects the wallet + enters the passkey → instant 0-score flag. No admin review. No blockchain. Fully automated.

Even if the attacker has the private keys, they don't have the pre-registered passkey. This is the differentiator.

**What needs to be built:**
| Component | Owner |
|---|---|
| `wallet_ownership` DB table | Backend |
| `POST /api/wallet/register` | Backend |
| `POST /api/report/compromised` | Backend |
| EIP-191 signature verification | Backend |
| Registration UI (wallet connect + sign + passkey) | Frontend |
| Report UI (wallet connect + passkey + submit) | Frontend |
| `wallet_links` routes (links existing table to WOR) | Backend |

---

## PHASE 2B — OTI VERIFIED BADGE (Cross-Platform Wallet Trust Badge)
**Owner: Backend Builder + Frontend Builder | Status: Architecture finalised (July 14, 2026) — open decisions closed. Ready for design into task prompts after Phase 2 (WOR) ships.**
**Sequencing:** Ships directly after Phase 2 (WOR). Reuses WOR's EIP-191 signature flow — not a replacement, an extension.

### What it is
A portable, cryptographically-signed proof of a wallet's OTI trust tier — attached to the wallet address itself, not just displayed on OTI's site. When a wallet is scored, OTI signs the result and stores it as a BAS attestation on BNB Chain. That attestation is then readable by any widget, extension, or third party — without trusting OTI's servers.

**Key architecture decisions (Ahmad, July 14, 2026):**
- ✅ **BAS (BNB Attestation Service) only** — attestation stored on BNB Chain via BAS. One attestation per wallet covers all chains (wallet address is chain-agnostic across EVM). BNB chosen because OTI token is on BSC, gas is cheap, and BAS is live with 3.5M+ attestations already.
- ✅ **No on-chain soulbound NFT** — removed entirely. Attestation via BAS is cleaner, cheaper, and avoids rescoring complexity.
- ✅ **No MetaMask Snap** — removed entirely. The widget and extension provide the same display coverage without requiring users to install a third-party plugin.
- ✅ **EAS/BAS is the sole trust record layer** — one standard, one chain, one place.

### How attestation works
```
OTI scores a wallet across all supported chains
→ Produces one unified tier result
→ OTI's signing key signs: "wallet 0xABC = TRUSTED tier, July 14 2026, expires August 13 2026"
→ Signed attestation stored in BAS registry on BNB Chain, indexed by wallet address
→ Anyone queries BAS: wallet address → tier + issue date + expiry
→ Widget reads BAS → displays badge on partner site
→ Extension reads BAS → displays badge on any site
→ Third party queries BAS directly → verifies without calling OTI's API
```

### Badge tiers — FINALIZED July 17, 2026

| Tier | Label | Score Band | Colour | Hex |
|---|---|---|---|---|
| 1 | HIGHLY TRUSTED | 75–100 | Mint green | `#00E5A0` |
| 2 | TRUSTED | 55–74 | Sky blue | `#4FC3F7` |
| 3 | NEUTRAL | 35–54 | Cool gray | `#90A4AE` |
| 4 | RISKY | 20–34 | Amber | `#FFB300` |
| 5 | HIGH RISK | 0–19 | Red | `#FF4444` |

**Compromised override:** Any wallet in `compromised_wallets` gets the Compromised badge regardless of its trust score. Colour: same red as HIGH RISK (`#FF4444`) with a warning icon (!) replacing the checkmark.

All five tiers get a badge. The badge doesn't mean "clean" — it means "OTI-verified at this tier."

### Visual anatomy (full card format — finalized July 17, 2026)
- Dark background, tinted to tier colour with a subtle ambient glow matching the tier
- **Shield icon:** pointed outline with checkmark inside. Fill: tier colour at 14% opacity. Stroke: tier colour at full opacity. Size: 44×44px in card form
- **Score number:** 28pt, tier colour, bold, below the shield
- **Tier label:** small caps, tier colour, below score number
- **Horizontal divider:** thin line at tier colour, 18% opacity
- **"OTI Verified" footer:** very small, subdued gray
- **Chain logo bubble:** 18×18px circle at bottom-right of the shield. Background: page background. Border: chain's brand colour. Text: chain ticker at chain's brand colour

**Chain bubble colours:** ETH `#627EEA` | BNB `#F0B90B` | POL `#8247E5` | SOL `#9945FF` | TRX `#EF0027` | All other EVM chains: their official brand colour, or neutral gray if none established.

### Badge interaction model — finalized July 17, 2026

**State 1 — Collapsed (always visible, no interaction required)**
Small horizontal pill next to the wallet address. Contains: shield icon (14px) with chain bubble at bottom-right + score number + tier label + ▼ arrow. Background: tier colour at 12% opacity. Border: tier colour at 33% opacity. The ambient trust signal — always present, zero friction.

**State 2 — Hover (quick summary, desktop only)**
Mouse-over triggers a floating tooltip panel above the pill. Contains: tier label + score (large) + all five signal bars (name | gradient bar | score) + wallet metadata chips (on-chain since, tx count) + "Click for full breakdown →" prompt. Disappears on mouse-out. Teaser format — enough to understand the score at a glance without committing to the full breakdown. Touch devices skip directly to State 3 on tap.

**State 3 — Expanded (full inline breakdown panel)**
Clicking the pill (or tapping) expands a panel directly below the wallet row in the partner's table or list. Nothing navigates away. Partner's page does not scroll. Contains: shield + tier + chain + metadata + score number (large, right-aligned) + all five signal bars with gradient fills + signal weight footnote + "OTI Verified ✓" tag + "Powered by OTI · otiscore.com" attribution. Clicking again collapses (▼/▲ toggle).

**Compromised expanded state:** Red warning shield with "!" → "⚠ Wallet Compromised" heading → explanation of WOR self-report + private key exposure warning + "Do not send funds" advisory + flag date.

**The no-redirect rule is absolute:** every interaction — hover tooltip, signal breakdown, WOR flow, attestation claim — happens inside the widget panel on the partner's page. The widget never navigates the user away from the partner's site.

### Pricing model (Ahmad, July 14, 2026)
- **First 10 million attestations: FREE** — deliberate network effect investment
- **Post-10M: one-time fee per attestation** — not recurring, not a subscription
- **OTI token discount** — users who pay in OTI token get a discount (Phase 3 integration)
- **Fee amount + token discount:** configured via admin panel, not hardcoded
- **Rescoring:** fully automated by OTI every 30 days — user never needs to do anything after initial attestation. OTI re-signs and updates the BAS attestation in background batches.

### Display layer — widget and extension
The badge is not displayed by the attestation itself. It is displayed by OTI's two distribution channels:

**Widget (partner-side):** Partners embed OTI's widget. Widget detects connected wallet → queries BAS → shows badge tier in partner's UI. Partners get full configuration control (which tiers to show, what action per tier, visual customisation). Initially free for partners; charged later once adoption is established.

**Extension (user-side):** Users install OTI's browser extension once. Extension auto-detects wallet addresses on every website — Etherscan, OpenSea, Twitter/X, any site — and overlays the OTI badge. No partner integration required. Works everywhere. The extension is the user's own widget.

### Phase 2B — Revenue Campaign (Build Now — Tasks 19–22)

**Philosophy:** Build the revenue-generating subset of Phase 2B first. Four components, one focused day, $7–25 total cost, $1,000–$5,000 projected return on first campaign. Fund Phase 2B Remaining with the proceeds.

**Build order is strict — each task unblocks the next:**

**Task 19 — Etherscan Key Rotation (Backend Builder, ~1 hour) — FIRST**
- Add `ETHERSCAN_API_KEYS` Railway env var (comma-separated, up to 10 keys)
- Round-robin counter in `chainRegistry.ts`'s `etherscanApiKey()` function
- Backward-compatible: falls back to single `ETHERSCAN_API_KEY` if array not set
- **10 keys = 1M calls/day = 200K–333K wallets scored per day → 3–5 days to build 1M target list**
- Key limit: maximum 10 free Etherscan keys (see D25 — ToS boundary). Never 100. Scale beyond 10 keys = Etherscan Standard plan ($199/mo) purchased from campaign revenue.

**Task 20 — BAS Schema Registration + Signing Endpoint (Backend Builder, ~2 hours) — SECOND**
- **Part A (do first):** Register OTI attestation schema on BAS — on-chain transaction, Ahmad signs and pays (~$0.01 gas). Fields: `address wallet, uint256 score, string tier, uint256 issuedAt, uint256 expiresAt`. Record the resulting schema UID — hardcoded into Task 21 smart contract.
- **Part B:** New file `src/routes/sign.ts`. Railway env var: `OTI_SIGNING_KEY`. Endpoint: `POST /api/sign/score` behind adminAuth. Input: `{ wallet_address, chain, expiry_timestamp }`. Logic: check compromised_wallets → check chain_scores (score ≥ 75) → sign with ethers.js → return `{ wallet_address, score, expiry_timestamp, signature }`.

**Task 21 — Smart Contract + XMTP Sender Script (Backend Builder, ~5 hours) — THIRD**
- **Part A:** Solidity smart contract on BNB Chain (chainId 56). Chainlink BNB/USD oracle (`0x0567F2323251f0Aab15c8dFb1967E4eaA47d42aEE`). Receives exactly $1 in BNB, verifies OTI signature via ecrecover, checks score ≥ 75 and expiry not passed, calls `BAS.attest()` with schema UID, emits `AttestationMinted`. `withdraw()` owner-only sweep. Deploy testnet → Ahmad confirms → mainnet deploy.
- **Part B:** Node.js XMTP sender script. Reads eligible wallets from `chain_scores` (chain='ethereum', score ≥ 75, scored within 30 days). Filters via `canMessage()`. For each: calls `/api/sign/score`, constructs `wallet_sendCalls` XMTP message (chainId 56, $1 BNB, signed payload as calldata). Rate: 3K messages/5-min window/sender wallet. Tracks sent wallets in local file to prevent re-sends. Ahmad runs manually after test confirms end-to-end.

**Task 22 — Conversion Dashboard (Frontend Builder, ~2 hours) — FOURTH**
- Moralis Streams webhook on `AttestationMinted` event → `campaign_conversions` table
- Admin-only React/Vite view: messages sent, attestations minted, conversion %, revenue in BNB + USD, live conversion table. Revenue happens without this — build last.

**Campaign targeting reality (documented July 19, 2026):**
- Ethereum wallets with score ≥ 75 in `chain_scores`: built via Dune Analytics SQL + OTI API pre-scoring
- Realistic universe of Ethereum wallets scoring ≥ 75: **2–4 million addresses** (active DeFi participants with strong on-chain history — casual wallets won't hit 75)
- XMTP penetration across all EVM wallets: **5–15%** — `canMessage()` filter cuts target list to this subset
- Real XMTP send list from 3M eligibles: ~150K–450K wallets
- Revenue at 0.25% conversion on 200K sends: ~500 attestations → ~$500
- Revenue at 0.25% conversion on 400K sends: ~1,000 attestations → ~$1,000
- Campaign 2 always outperforms Campaign 1 — first campaign generates conversion data (which score bands, which message copy, which time of day) that makes the second campaign 2× more effective

**The BSC blocker doesn't apply to this campaign (D26):**
Every EVM wallet address is the same person on Ethereum and BNB Chain. OTI scores on Ethereum (already live, no Etherscan Lite needed). Payment is collected on BNB Chain (cheap gas, no scoring needed). The $49/mo Etherscan Lite subscription is not required for the campaign.

---

### Phase 2B — Post-Campaign Remaining (After Campaign Revenue Is In)

Fund these with campaign proceeds:

**Backend:**
- `GET /v1/badge/:wallet` — widget API endpoint; reads Score Source setting from `system_settings` and routes to OTI DB / BAS / Auto accordingly
- `PATCH /admin/score-source` — admin endpoint to switch Score Source mode
- `POST /api/attestation/issue` — OTI backend calling BAS SDK directly (for in-widget attestation claim flow, separate from smart contract path)
- `GET /api/attestation/:address` — check attestation status
- `POST /api/attestation/revoke` — revoke on WOR compromise report
- Attestation scheduler — daily batch rescore of all wallets approaching 30-day expiry
- **Proactive background scoring pipeline** — scores wallets from Dune Analytics address lists in background batches. Feeds airdrop eligibility tracking list (D22 — tracking mandatory from day one of this pipeline, cannot be retroactively reconstructed).
- `wallet_attestations` DB table — track issued attestations, expiry, tier at issue time
- Full BAS SDK integration (BNB Chain)
- Upgrade to Etherscan Standard ($199/mo) if background scoring volume exceeds 10-key free-tier capacity

**Frontend:**
- Attestation claim UI embedded inside widget (no redirect)
- Public `/verify/:address` page
- Five-tier badge visuals (design finalized July 17, 2026 — see above)
- Admin panel: attestation stats, manual revoke, fee/discount settings, Score Source selector

### Open decisions
All architectural decisions are locked. No open design decisions remain. Task prompts for Tasks 19–22 are written and ready (see `TASKS.md`).

**Note on attestation fee:** amount, OTI token discount rate, and 10M free-tier cap managed via admin dashboard — not hardcoded. Ahmad sets them live.

**Privacy policy + Terms & Conditions:** deferred until full product is built (Ahmad, July 14, 2026).

### Risks logged
- OTI signing key compromise = fake badges possible — Ahmad holds the private key, it lives only in Railway env vars, never in code
- XMTP fees: currently $0 on mainnet. When fees activate (~$50–100 per 1M messages), Campaign 2 economics change. First campaign is free — run it before fees activate.
- BAS dependency — if BAS has an outage, attestation writes fail. Read path falls back to OTI's own DB record.
- Legal: "Verified" badge implies liability — clear disclaimer needed, consistent with WOR compromised-wallet reporting

---

## PHASE 3 — MONETIZATION INFRASTRUCTURE + TOKEN
**Owner: Both Builders | Status: Planned — infrastructure partially ready**

**Strategic decision (Ahmad, July 14, 2026):** Phase 3 now has two pillars:
1. **Fiat/crypto payment infrastructure** — paid API plans, self-serve portal, Stripe + Coinbase Commerce
2. **OTI token creation + ecosystem integration + presale** — token deployed on BSC, plugged into the platform (Rewards Pool, staking, ecosystem utility), presale runs during this phase to fund continued development

Exchange listing is a separate, later event — happens after Phase 3 revenue streams are working and the token has real utility behind it. Not scoped here; Ahmad decides timing.

| Feature | Notes |
|---|---|
| Developer self-serve portal | Sign up, get API key, choose plan — no Ahmad needed |
| Pro/Enterprise plan tiers | `plan_configs` table ready, needs new rows + payment checkout |
| Fiat payments | Stripe — easiest integration, industry standard |
| Crypto payments | Coinbase Commerce (hosted checkout, low setup) |
| BSC/Base/Optimism unlock | $49/mo Etherscan Lite — Ahmad's decision |
| **OTI token — create + deploy** | 30,000,000 fixed supply, BSC first, cross-chain later. See `TOKENOMICS.md`. Own independent token — NOT the OpenFlow "FLOW" ecosystem token. |
| **OTI token — ecosystem integration** | Plug token into the platform: Revenue-Backed Rewards Pool (buyback from API revenue), staking, and any other ecosystem utility Ahmad defines. |
| **OTI token — presale** | Pre-listing sale ($10k raise, 500k tokens, BNB/USDT on BSC). Runs during Phase 3 to fund continued development. Price/liquidity design deferred — do not reconstruct without Ahmad. |
| **Exchange listing** | Post-Phase 3 milestone — after revenue streams are live and generating real income. Ahmad decides timing. Not scoped into Phase 3 tasks. |
| **Partner attribution tracking — REQUIRED from day one** | Every attestation payment record must carry partner attribution metadata (which widget embed / partner site triggered the claim). Required for the Partner Revenue Share model (see BUSINESS_MODEL.md Layer 3). Cannot be retrofitted — must be in the payment record schema from the first day Phase 3 billing is built. |

---

## PHASE 4 — GROWTH FEATURES
**Status: Future**

| Feature | Notes |
|---|---|
| Score history UI | DB already accumulating data |
| Multi-chain wallet comparison | Same wallet scored across multiple chains |
| Wallet portfolio view | `wallet_links` table infrastructure already built |
| Webhook alerts | Notify integrators when a watched wallet is compromised |
| Enterprise exchange path | Compliance screening, withdrawal risk scoring — see Playbook Section 16 |
| **Etherscan key upgrade (Standard plan, $199/mo)** | After campaign revenue is in and proactive background scoring volume exceeds 10-key free-tier capacity (~1M calls/day), upgrade to Etherscan Standard. Key rotation infrastructure is already built (Task 19). Upgrade = swap env var to a single high-quota key — no code change. |
| **Bot / suspicious-wallet behavioral detection** | Deliberately deferred, Ahmad, July 13, 2026 — see note below. Not scoped, not assigned. |

**Note on bot/suspicious-wallet detection:** Ahmad identified this as arguably the system's primary intended use case — flagging bots, mixers/wash-trading patterns, and suspected-malicious wallets from on-chain behavior — but it currently does not exist at all; the system scores any syntactically valid address the same way regardless of behavioral red flags. This is a full new design (new signal(s) and/or a distinct classification layer), not a bug fix, and is explicitly set aside for now. Do not start scoping this until Ahmad reopens it — current priority is finishing real-data/signal-accuracy correctness work (`FIXES.md`) first.

---

## PHASE 5 — DISTRIBUTION CHANNELS (bots, widget, extension) 🔒 LAST — INTENTIONAL
**Owner: Backend Builder (bots + widget) + separate repo (extension) | Status: Not started — begins only after Phases 2, 3, and 4 are complete**

**Strategic decision (Ahmad, July 14, 2026):** Phase 5 is deliberately last. Distribution channels exist to bring people to the product — but the product needs to be fully built first (WOR, monetization, growth features). When mass adoption comes via Phase 5, the full stack will already be there to receive it. Build order: Phase 2 → 3 → 4 → 5.

Source: OTI Full Distribution & Technical Development Strategy (Founder's Playbook)

### Channel 1 — Telegram Bot
- Library: Telegraf (Node.js) — free
- Lives in: `/bots/telegram/` inside backend repo
- Railway deployment: second process alongside API server
- Commands: `/score [address] [chain]`, `/help`, `/about`
- Group mode = viral lever: bot added to a crypto group, replies are public
- **Railway free tier warning:** API + Telegram + Discord = ~720 hours/month. May need Railway Hobby ($5/mo).
- Env vars needed: `TELEGRAM_BOT_TOKEN`, `OTI_BOT_API_KEY`

### Channel 2 — Discord Bot
- Library: discord.js (Node.js) — free
- Lives in: `/bots/discord/` inside backend repo
- Railway deployment: third process
- Slash commands: `/score`, `/help`
- Rich embed responses (color, fields, thumbnail)
- Env vars needed: `DISCORD_BOT_TOKEN`, `DISCORD_CLIENT_ID`, `OTI_BOT_API_KEY`

### Channel 3 — Embeddable Widget
Confirmed embed specification (July 17, 2026 — see DECISIONS.md D24):
```html
<script src="https://widget.otiscore.com/oti.js" data-wallet="0x..." data-chain="ETH" data-key="partner-api-key"></script>
```
**Four hard constraints (non-negotiable at implementation time):**
1. **Zero external dependencies** — vanilla JS + Fetch API only. Works on any tech stack.
2. **Scoped styles** — all CSS scoped via unique class prefix or Shadow DOM. Must not leak or be overrideable by partner's global styles.
3. **Graceful empty state** — if no score exists, widget renders nothing silently. No error, no broken placeholder.
4. **No redirect on any interaction** — hover, click, WOR flow, attestation claim — all happen inside the widget. `window.location` is never called.

**WOR inside the widget:** The full WOR registration and compromise report flow is embedded as in-widget panels. A wallet owner on a partner site can register or file a compromise report without leaving the partner's page. Every partner becomes a WOR registration surface.

**Non-EVM chains — deferred from widget:** Non-EVM address formats (Solana, TON, Tron) are not linkable to EVM addresses without `wallet_links` populated via WOR. Widget shows EVM unified scores only until cross-chain identity infrastructure is in place. OTI continues scoring non-EVM chains independently; per-chain non-EVM badges are a future phase item.

**Data source:** Widget reads from OTI's `chain_scores` DB by default (Score Source = OTI Backend). Transitions to BAS-first once attestation adoption is meaningful — server-side setting, no partner action required (see DECISIONS.md D23).

- Uses shared widget API key internally
- Target sites: NFT marketplaces, on-chain portfolio trackers, P2P OTC platforms

### Channel 4 — Firefox Extension (then Chrome)
- Separate GitHub repo: `oti-firefox-extension`
- Content script: auto-detects `0x[a-fA-F0-9]{40}` addresses on Etherscan, OpenSea, BscScan
- Injects score badge next to each address
- Firefox first (free to publish) → Chrome after first revenue ($5 one-time fee)
- No backend changes required

**Distribution funnel:**
```
Community user sees bot reply → Shares score card (OTI branding)
→ Developer discovers OTI → Visits dev docs → Gets free API key → Integrates
→ Business sees API usage → Reaches out for enterprise contract
```

---

## Revenue Milestones (from Playbook)
| Milestone | Target |
|---|---|
| 1,000 active API users | 90 days after distribution launches |
| 3 paying business integrations | 6 months after distribution |
| First enterprise exchange contract | 12 months after distribution |
