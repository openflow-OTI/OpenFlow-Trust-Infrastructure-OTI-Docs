# OTI Token — Tokenomics, Sale, Liquidity & Revenue Distribution

> Last updated: July 9, 2026 (session 6) | Maintained by: Development Manager
> **Status: Ahmad-approved design, locked.** This is OTI's own independent token — it is NOT the OpenFlow "FLOW" ecosystem token discussed in earlier planning. OTI can live with or without OpenFlow, so it has its own token, its own supply, and its own economics. Launch chain: BNB Smart Chain (BSC) first, with cross-chain expansion planned later.
> Regulatory note: Ahmad has explicitly decided this raise is small enough not to require formal securities/regulatory review at this stage. This decision was flagged to him once by the Manager and is his call to make.

---

## 1. Total Supply

**30,000,000 OTI — fixed supply, no inflation, no mint function after launch.**

Fixed supply is a deliberate trust signal: no bucket can be silently expanded, and total dilution is knowable from day one.

---

## 2. Allocation

| Bucket | % | Tokens | Purpose |
|---|---|---|---|
| Sale (Pre-Listing + Future Discount Sales) | 15% | 4,500,000 | Funds building OTI; sold in one pre-listing round plus later as-needed discounted rounds |
| Team & Founders | 15% | 4,500,000 | Long-term founder ownership (Ahmad + Musty) |
| Ecosystem & Partnerships | 15% | 4,500,000 | Developer grants, integration incentives, protocol/exchange partnerships that drive real API usage |
| Revenue-Backed Rewards Pool | 30% | 9,000,000 | Staking/holder rewards — funded by real OTI API revenue via buyback, not token inflation |
| Liquidity & Market Making | 10% | 3,000,000 | DEX trading pool depth, exchange listing support |
| Treasury / Reserve | 15% | 4,500,000 | Operating costs, security audits, contingency — multisig-controlled |
| **Total** | **100%** | **30,000,000** | |

---

## 3. Vesting

**Universal rule:** every bucket except Liquidity vests linearly over **36 months**, releasing 1/36th each month, starting from that bucket's (or that specific sale's) own release date. No cliff, no lump-sum unlock — a steady monthly drip for 3 years.

- **Liquidity & Market Making — the one exception.** Fully unlocked at listing, with no vesting. Liquidity has to be tradeable from day one; vesting it would leave the token illiquid at launch.
- **Sale round tokens** each get their own independent 36-month clock starting from their own purchase date — the pre-listing round starts at listing; each later discount sale starts its own clock from when that sale happens.
- **Team, Ecosystem, Rewards Pool, Treasury** — all 36-month monthly linear from their respective start dates.
- **Revenue-Backed Rewards Pool** has a second gate on top of vesting: even as tokens vest into availability, actual payout to stakers still depends on real API revenue funding the pool (see Section 6). Vesting caps the *maximum* release rate; revenue availability governs whether a payout actually happens that month.

---

## 4. Token Sale Structure

### 4.1 Pre-Listing Sale Round (the only round before launch)

| | |
|---|---|
| Target raise | $10,000 |
| Price | $0.02 per OTI |
| Tokens sold | 500,000 OTI (≈1.7% of total supply) |
| TGE unlock | **15% (75,000 OTI) unlocked immediately at listing** |
| Vesting (remainder) | Remaining 85% (425,000 OTI) — 36-month monthly linear, from listing date |
| Accepted currency | BNB / USDT on BSC |

**Why a small TGE unlock:** without it, the only tokens circulating at the instant of listing would be the Liquidity pool's own OTI side — leaving almost no real market beyond the pool itself. A 15% unlock gives presale buyers some immediate liquidity (rewarding early support) while keeping 85% locked to a 3-year drip, so there's no large day-one dump risk.

This is a deliberately small, single public round — there is no separate private/KYC'd round, since Ahmad does not currently have private investors lined up. All buyers go through the same public terms.

### 4.2 Post-Listing Discount Sales (as-needed, not scheduled)

These are **not** a second fixed round. They happen only when OTI genuinely needs additional capital, at any point after listing:

- **Price:** set at a discount to the live market price at the time of that sale (typically 20–30% below market — exact discount decided per sale, since market price varies)
- **Source:** drawn from the ≈4,000,000 OTI remaining in the Sale bucket after the pre-listing round
- **Vesting:** 36-month monthly linear, starting from that specific sale's own purchase date
- **Transparency:** every future discount sale is logged publicly with its date, amount raised, price, and vesting terms — no ambiguity about how much of the Sale bucket has been used vs. still reserved

**Why this shape:** committing to only one round now, and treating every later raise as need-based rather than calendar-based, avoids selling supply the business doesn't yet need — and the discount-for-lockup exchange means every future raise adds capital without creating instant sell pressure on the newly raised tokens.

---

## 5. Initial Liquidity & Circulating Market Cap

**Why this section matters:** a token's initial DEX liquidity pool determines how much can be bought or sold before the price swings sharply (slippage). What matters for a healthy launch is not FDV (a hypothetical "if all 30M tokens were unlocked" number) but the ratio of **liquidity pool value to actual circulating market cap** — the real, currently-tradeable tokens. The goal here is simple: **have enough liquidity relative to what's actually circulating that the pool isn't a shallow, easily-drained puddle.**

### 5.1 Initial Launch Pool (day one)

- **$4,000** of the $10,000 raised is allocated to seed liquidity on the BNB/USDT side (the remaining $6,000 goes to immediate operating costs — see Section 6).
- That **$4,000 in BNB/USDT** is paired with an equal **$4,000 worth of OTI** from the Liquidity bucket → 200,000 OTI (at $0.02/token).
- **Initial pool size: $8,000 total** ($4,000 BNB/USDT + $4,000 worth of OTI, 200,000 tokens).
- The remaining 2,800,000 OTI in the Liquidity bucket stays reserved, unpaired, ready to be added to the pool as capital becomes available.

### 5.2 The 10x Ratio — a Running Rule, Not a Fixed Launch Number

**Rule: market cap should track at roughly 10× the liquidity pool's value, on an ongoing basis — not a one-time target hit exactly on day one.** As the pool grows (Section 5.3) and circulating supply grows (via vesting unlocks each month), both sides of the ratio move together. Launch-day numbers are whatever they naturally are; the 10x rule is the benchmark used to judge if liquidity is keeping pace as supply unlocks, not a number to force via an artificial launch price or an oversized TGE unlock.

**Actual launch-day numbers, for reference:**

| Source | OTI |
|---|---|
| Liquidity pool (OTI side) | 200,000 |
| Pre-listing sale TGE unlock (15% of 500,000) | 75,000 |
| **Total circulating supply at launch** | **275,000** |

- **Circulating market cap at launch:** 275,000 × $0.02 = **$5,500**
- **Liquidity pool value:** $8,000
- At launch, the pool is actually worth *more* than circulating market cap (ratio favors liquidity, not the 10x-cap-to-pool direction) — this is expected and fine: it means day one is maximally safe from a slippage standpoint. The 10x rule becomes the operative check later, as vesting unlocks push circulating supply (and therefore market cap) up faster than the pool grows on its own.
- For reference, FDV (price × full 30,000,000 supply) is $600,000 — not the launch market cap; it's the ceiling market cap only reaches once all vesting completes in month 36.

### 5.3 Growing the Pool Over Time — Keeping Pace with the 10x Rule

As vesting unlocks more tokens each month (Section 3), circulating supply and market cap rise. The pool should be topped up to keep market cap from drifting past ~10x the pool's value:

- **From post-listing discount sale proceeds:** 30% of every future discount sale's raised capital is earmarked for liquidity top-ups (paired with unpaired OTI from the reserved Liquidity bucket).
- **From API revenue:** none of the revenue distribution (Section 6) is earmarked for liquidity by default — this keeps the two funding sources cleanly separated. If the 10x ratio starts slipping (market cap growing much faster than the pool), this should be revisited.
- **Monitoring:** Ahmad/Manager should recheck the ratio each time a discount sale happens or a significant vesting milestone passes, and adjust the liquidity top-up accordingly.

**Bottom line:** liquidity starts deep relative to launch-day market cap, and the 10x market-cap-to-pool ratio is the ongoing rule used to keep pace as vesting unlocks more circulating supply over the following 3 years — not a number engineered into the launch price itself.

---

## 6. Revenue Distribution

Once OTI's paid API tiers (Pro/Enterprise, per the existing Revenue Model in TASKS.md) begin generating real revenue, that revenue is split every month as follows — **starting from day one, immediately after the pre-listing sale round completes** (not gated behind any revenue threshold):

| Category | % | What it's for |
|---|---|---|
| Operating Costs | 40% | Hosting (Railway), any paid data providers/APIs, tools, infra — always covered first |
| Revenue-Backed Rewards Pool (buyback) | 25% | Buys OTI on the open market and routes it into the Rewards Pool for stakers — **only executes when the market is down**; held as cash reserve otherwise, deployed opportunistically |
| Team | 20% | Cash payout to the founders (Ahmad + Musty) — split decided internally between them. This is separate from the Team's fixed token allocation (Section 2), which is equity-like, not cash |
| Treasury/Reserve top-up | 15% | Cash reinvested into growth — marketing, audits, partnerships, legal. Separate from the fixed Treasury token allocation; this is a cash reserve, not tokens |

**Why the buyback is conditional, not automatic:** buying back tokens every month regardless of price would mean routinely overpaying when the market is already strong. Restricting the buyback to down-market conditions makes the Rewards Pool a genuine value-accrual mechanism (buy low) rather than a fixed monthly expense.

---

## 7. Chain & Cross-Chain Plan

- **Launch chain:** BNB Smart Chain (BSC) — chosen for low fees and fast settlement, matching the presale mechanics above (BNB/USDT accepted).
- **Cross-chain expansion:** planned for later, once OTI's token and liquidity are established on BSC. Not yet designed in detail — when this becomes active work, it should be scoped as its own task (likely via a standard bridging solution such as LayerZero or Wormhole, rather than a custom-built bridge, consistent with the "don't reinvent solved infrastructure" principle applied to vesting).

---

## 8. Explicitly Out of Scope For Now

- **Staking/derivative system (e.g., a "locked OTI" derivative used as collateral):** discussed as a future phase, not part of this design. To be scoped separately once OTI has real usage and revenue to build the mechanism around.
- **Governance:** not part of this design. No DAO or voting mechanism defined yet for OTI token holders.
- **Formal legal/securities review:** flagged once to Ahmad; his explicit decision is that the raise is small enough not to require this at this stage.
