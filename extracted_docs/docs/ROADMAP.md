# OTI — Product Roadmap
> Last updated: July 10, 2026 (session 8 — full rewrite for consistency with the new `FIXES.md` file; no strategic changes, phase structure unchanged from the July 5 reorganization) | Maintained by: Development Manager
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

## PHASE 3 — MONETIZATION INFRASTRUCTURE
**Owner: Both Builders | Status: Planned — infrastructure partially ready**

| Feature | Notes |
|---|---|
| Developer self-serve portal | Sign up, get API key, choose plan — no Ahmad needed |
| Pro/Enterprise plan tiers | `plan_configs` table ready, needs new rows + payment checkout |
| Fiat payments | Stripe — easiest integration, industry standard |
| Crypto payments | Coinbase Commerce (hosted checkout, low setup) |
| BSC/Base/Optimism unlock | $49/mo Etherscan Lite — Ahmad's decision |
| **OTI token + presale** | Finalized design — see `TOKENOMICS.md`. Own independent token (not the OpenFlow "FLOW" ecosystem token), 30,000,000 fixed supply, launching on BSC first, cross-chain later. Price/liquidity design intentionally deferred — do not reconstruct. Not yet scoped into Builder tasks. |

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
- Vanilla JS, zero dependencies, served from Railway at `GET /widget.js`
- One-line integration for site owners: `<div data-oti-wallet="0x..." data-oti-chain="ethereum"></div><script src="...widget.js" async></script>`
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
