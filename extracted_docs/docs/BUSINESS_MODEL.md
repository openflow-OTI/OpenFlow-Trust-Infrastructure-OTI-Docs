# OTI — Business Model
> Last updated: July 17, 2026 (session 16 — in-widget attestation flow, confirmed API tier signal-depth boundaries, partner revenue share model, Score Source Switcher as default widget data source) | Maintained by: Development Manager
> This file captures the complete OTI business model — revenue, cost structure, growth strategy, and how every layer connects. It is the single source of truth for how OTI makes money and sustains itself. Update this file whenever Ahmad makes a strategic business decision that changes any part of the model.

---

## The One-Line Model

> OTI scores wallets, attests the result cryptographically, and earns revenue from the attestation, the data, and the infrastructure built around it — while the attestation itself creates the network effect that forces partners in.

---

## What OTI Sells

### Layer 1 — Attestation (B2C, direct to wallet owners)
OTI scores any wallet address across any supported chain and issues a cryptographically signed attestation of the result, stored on BNB Chain via BAS (BNB Attestation Service).

- **First 10 million attestations: FREE** — this is the network effect purchase. Not charity, not marketing spend — it is the deliberate act of building a critical mass of verified wallets before charging a cent. When 10 million wallets carry OTI attestations, partners have no choice but to integrate.
- **Post-10M: one-time fee per attestation** — not a subscription, not a recurring charge. One payment, permanent attestation record. Fee amount and OTI token discount configured via admin panel.
- **Rescoring: fully automated** — OTI rescores every attested wallet every 30 days in background batches. The user never has to do anything after the initial attestation. Score changes → attestation automatically updated.
- **Revenue from attestation fee scales with total wallet population.** There are already hundreds of millions of wallets and will be billions. Volume is the revenue engine, not margin per user.
- **The attestation claim flow is embedded inside the widget — no redirect to OTI's site required.** A wallet owner who encounters their badge on a partner site can initiate the full attestation claim flow (and the WOR registration/compromise report flow) without leaving that partner's page. This is a distribution decision: most users encounter OTI on a partner site, not at otiscore.com. Making the claim flow in-widget means every partner becomes an attestation conversion surface automatically. The widget never calls `window.location` or opens a new tab for any interaction.

### Layer 2 — API Access (B2B, developers and platforms)
Partners, developers, and enterprises pay for access to OTI's full scoring intelligence — not just the tier (which the attestation shows), but the raw signal breakdown, historical scores, chain-by-chain data, and webhook alerts.

**Confirmed tier signal-depth boundaries (July 17, 2026):**

| Tier | What the response includes |
|---|---|
| **Free / Anonymous** | Score number + tier label + compromised flag. Matches what the BAS attestation stores — neither competes with the other; they serve the same information through different channels (live API vs. blockchain record). |
| **Pro** | All five signal scores with weighted contributions (`score`, `weighted`, `maxWeight` per signal) + score history (last N scores for the wallet from `chain_scores`). The intelligence layer — explains *why* a wallet received its tier. Any developer who needs signal-level reasoning (risk-pricing a loan, adjusting collateral, auditing a wallet's history) must use the Pro API. |
| **Enterprise** | Everything in Pro + raw per-chain signal numbers (each individual chain the wallet is active on, not just the unified cross-chain EVM score) + webhook alerts for watched-wallet score changes + batch scoring endpoint + compliance-formatted export. The data infrastructure layer for exchanges, payment processors, and regulated entities who need OTI in their operational pipelines. |

**Why attestation and API are structurally complementary — and must stay that way:** BAS stores the tier only — never the signal breakdown. This is deliberate (see DECISIONS.md D17). It ensures the attestation and the Pro API are permanently complementary rather than redundant. The attestation creates the public, trustless proof; the Pro API unlocks the private intelligence behind it. Any change that puts signal-level data into the attestation record would destroy the API revenue model by giving away Pro-tier information for free via BAS. Never put signal breakdowns in attestation records.

### Layer 3 — Widget Access (B2B, charged later — Phase 5+)
Partners who embed OTI's widget get a pre-built trust display layer without building their own integration.

- **Default data source: OTI's own database** — the widget reads from `chain_scores` directly by default (Score Source = OTI Backend), not from BAS. This means the widget works from day one, before any wallet has an attestation. No BAS dependency in the early growth phase. Source transitions to Auto (BAS-first with DB fallback) once attestation adoption is meaningful — all server-side, invisible to the partner (see DECISIONS.md D23).
- **Initially free** — seeding partner adoption during Phase 5 rollout
- **Charged later** — once widget adoption is established, a subscription tier for commercial widget use
- **Partner Revenue Share (July 17, 2026):** Partners earn a percentage of every attestation fee generated through their embedded widget. When a user discovers their OTI badge on a partner site and claims a BAS attestation through the in-widget flow, a share of that attestation fee is credited to the partner. The percentage is Ahmad's decision — configurable via admin panel, not hardcoded. **Attribution tracking is non-optional:** every attestation payment record must carry partner attribution metadata from day one of Phase 3 billing infrastructure. An attestation triggered through a partner's widget must know its origin. This cannot be retrofitted — it must be built into the payment record schema from the first day Phase 3 is built. Without it, the revenue share model has no data to operate on.
- **Widget is the primary attestation sales channel.** Not a display layer bolted onto the product — it is the surface where most users will encounter OTI and where attestation conversion actually happens. Partners who earn from attestations have a financial incentive to surface the widget prominently and design UX that encourages attestation claims. OTI does not need to manage this — the revenue share structure manages it.

### Layer 4 — OTI Token (ecosystem utility, Phase 3+)
OTI token is not speculative — it is a functional payment rail within the OTI ecosystem from day one.

- **Pay for attestation in OTI token** — primary utility, creates immediate demand
- **Discount for token payment** — incentivises token acquisition over fiat/crypto payment
- **Early adopter rewards** — first 1 million users who register for attestation receive OTI tokens before the token launches. This is not a gimmick. It creates: real utility from launch day, token holders with skin in the game who promote OTI organically, and a launch story of "1 million verified wallets, tokens already in real hands."
- **Revenue buyback** — 25% of API/widget revenue buys OTI on the open market and routes into the Rewards Pool for stakers (buyback only executes in down-market conditions — see TOKENOMICS.md)

---

## The Cost Structure

### Ongoing costs OTI bears
| Cost | Nature |
|---|---|
| Blockchain API calls for rescoring | Per-wallet, monthly — scales with user count |
| BAS attestation writes on BNB Chain | Gas cost per attestation update — cheap on BNB but non-zero at scale |
| Railway hosting (backend) | Fixed monthly |
| Vercel hosting (frontend) | Fixed monthly |
| Etherscan / data provider subscriptions | Fixed or usage-based |

### Why costs are manageable
1. **BNB gas is cheap** — attestation writes on BNB Chain cost fractions of a cent each. 10 million monthly rescores is a manageable gas bill, not a prohibitive one.
2. **Rescoring is batched** — OTI rescores wallets daily in rolling batches as their 30-day windows expire, not all at once. Infrastructure load is spread flat across the month.
3. **Revenue scales with the same user base driving costs** — every paying attested user generates revenue that covers their own rescoring cost plus margin.
4. **OTI token treasury** — Phase 3 builds a token treasury funded by presale and revenue. Infrastructure costs can be drawn from treasury during early growth phases before revenue catches up.
5. **10M free users are an investment with a defined end** — after 10M, every new attestation is paid. Free users build the network effect that makes the paid tier valuable.

---

## The Network Effect Engine

This is the core strategic logic. It is a self-reinforcing loop:

```
Free attestations (first 10M)
         ↓
Millions of wallets carry OTI badges
         ↓
Partners see OTI badges everywhere → need to integrate
         ↓
Partners embed OTI widget (free initially, paid later)
         ↓
Widget shows badges on partner sites → more wallets discover OTI
         ↓
More wallets pay for attestation (post-10M)
         ↓
More revenue → covers rescoring costs + funds token buyback
         ↓
Token value rises → early adopters benefit → more organic promotion
         ↓
More wallets discover OTI → loop repeats
```

OTI doesn't need to convince partners to integrate. The network effect does it — when enough wallets have badges, a partner that doesn't show badges is the odd one out.

---

## The Two Distribution Channels

### Widget — Partner-side (B2B)
A partner embeds one script tag. OTI's widget detects the connected wallet, queries BAS for the attestation, and displays the badge tier right in the partner's UI. The partner has full configuration control:
- Which tiers to show or hide
- What action to trigger per tier (show, warn, block)
- Visual customisation to match their design system

Partners get a trust filtering system without building one. OTI gets distribution on every site that embeds the widget.

### Extension — User-side (B2C)
A user installs the OTI browser extension once. From that point, on every website they visit — Etherscan, OpenSea, Twitter/X, any site displaying a wallet address — the extension auto-detects the address and overlays the OTI badge. No partnership required. No site integration required.

**The extension is the user's own widget.** Same badge, same data, but controlled by the user not the platform. It works on every site on the internet regardless of whether that site has ever heard of OTI.

Together: widget covers partner surfaces (where the platform chooses to show trust), extension covers everywhere else (where the user carries trust with them).

---

## What the Attestation Actually Is

OTI scores a wallet across all supported chains. The result — a tier (HIGHLY TRUSTED / TRUSTED / NEUTRAL / RISKY / HIGH RISK) — is signed by OTI's private key and stored in BAS (BNB Attestation Service) on BNB Chain, indexed by wallet address.

**This is important:** the attestation stores the tier only — not the full signal breakdown. Anyone can query BAS and verify:
- That OTI attested this wallet
- What tier it received
- When it was issued and when it expires

But they cannot get the signal weights, the per-chain scores, or any other intelligence from the attestation alone. For that, they call OTI's API.

**Chain agnosticism:** the wallet address is the same across all EVM chains. OTI scores the wallet across every chain it has data on, produces one unified tier, and stores one attestation on BNB Chain. The attestation covers the wallet's full cross-chain picture. A user with activity on Ethereum, Polygon, and Arbitrum gets one attestation that reflects all of it.

**Trustless verification:** because the attestation is stored on BNB Chain via BAS, any developer can verify it directly from the blockchain — without calling OTI's API, without trusting OTI's servers, without any dependency on OTI being online. This is the property that makes the attestation genuinely portable and valuable to the ecosystem.

---

## Revenue Timeline

| Phase | Revenue source | Status |
|---|---|---|
| Now (post Phase 1) | Free API tier (anonymous, limited) | Live |
| Phase 2B | Attestation fee (post-10M free tier), admin panel managed | To build |
| Phase 3 | Paid API tiers (Pro/Enterprise), self-serve portal, Stripe + Coinbase Commerce | To build |
| Phase 3 | OTI token presale ($10k raise) | To build |
| Phase 5+ | Widget subscription (B2B, after adoption established) | Future |
| Phase 3+ | Token buyback from revenue → Rewards Pool | Ongoing after token launch |

---

## Why OTI Stays Healthy Long-Term

1. **Volume beats margin.** Billions of wallets exist and will continue to be created. A small one-time fee across that population is a large revenue base without requiring enterprise-scale B2B sales cycles.
2. **API revenue is stickier than attestation revenue.** A partner who integrates OTI's API into their product doesn't leave. Switching costs are high. API subscriptions compound.
3. **The token is a cost absorber.** Token treasury covers infrastructure costs during scaling. Revenue buyback creates token value. Token value increases treasury purchasing power. The model is self-hedging.
4. **Free tier wallets are not a cost — they are the product.** Without free attestation adoption, the network effect never fires. The 10M free tier is a deliberate strategic spend with a defined end and a clear expected return: partner integration.
5. **Partners eventually pay.** Widget starts free to seed adoption. Once embedded widely, pricing widget access is straightforward — partners are already dependent on it.

---

## What Is Not Part of This Model (Explicitly)

- **MetaMask Snap:** removed. The widget and extension cover the same ground without requiring user installation of a third-party plugin.
- **On-chain soulbound NFT:** removed. Attestation via BAS is cleaner, cheaper, chain-agnostic, and doesn't carry rescoring complexity.
- **Recurring attestation subscriptions:** removed. One-time fee reduces friction and maximises user volume, which is the correct lever at this scale.
- **Per-chain attestation contracts:** removed. One BAS attestation on BNB Chain covers all chains for that wallet address.
