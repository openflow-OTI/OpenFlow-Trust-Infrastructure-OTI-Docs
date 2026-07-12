# OTI — Architectural Decisions Log
> Last updated: July 12, 2026 (session 10 — file created. Ahmad's direction: before treating any observed behavior as a bug, understand whether it was a deliberate decision with a reason. This file records why things exist the way they do — confirmed decisions, known technical limitations, and items still pending an answer from the Builder.) | Maintained by: Development Manager

---

## What This File Is

This file records the reasoning behind how OTI was built — not what the code does (that's `ARCHITECTURE.md`) but **why** it does it that way. It exists because:

- Some behavior looks like a bug but was a deliberate tradeoff
- Some behavior is a known technical limitation, not an oversight
- Without this record, future Builders fix things that were intentional and break something else in the process

**Access:**
- **Manager** — reads and writes this file. Only the Manager adds or updates entries.
- **Builders** — read this file. If your task or fix touches something documented here, read the relevant entry before writing a single line of code. Never update this file yourself — report your findings to the Manager.
- **Ahmad** — reads this file. If a decision needs reversing, tell the Manager and they will update the entry.

**Entry statuses:**
- `INTENTIONAL` — a deliberate design choice, confirmed by Ahmad or the Manager. Do not change without Ahmad's explicit approval.
- `TECHNICAL LIMITATION` — the API, platform, or data source at the time didn't support a better approach. Watch for newer APIs or approaches that remove the constraint.
- `REVISIT` — was intentional or constrained at the time, but there is now a better path. Still do not change without Manager instruction — it is flagged for future work, not immediate reversal.
- `PENDING ANSWER` — behavior was observed in the diagnostic audit but the reason is unknown. The Builder has been asked. Do not treat as a bug until the answer comes back and the Manager updates the status.

---

## Confirmed Decisions

---

### D1 — CORS Is Fully Open (No Origin Restrictions)
**Status:** INTENTIONAL
**What the code does:** `app.use(cors())` with no origin restrictions. Any domain can call the OTI API from a browser.
**Why:** OTI is a public developer API. Exchanges, DeFi frontends, and wallet apps need to call it directly from their browser-based dashboards. CORS restrictions would break those integrations. API key quotas are the correct tool for access control — CORS is not.
**Confirmed by:** Ahmad (original architecture decision) + Manager (documented July 5, 2026 in ARCHITECTURE.md)
**Implications for fixes:** Never add origin restrictions without Ahmad's explicit instruction. The open CORS + API key auth combination is correct and intentional.
**Future:** Once paid plans exist, per-key origin allowlisting may be added as an opt-in enterprise feature. Low priority, does not change the default.

---

### D2 — `scoring.ts` Is Never Modified
**Status:** INTENTIONAL
**What the code does:** The backend `src/lib/scoring.ts` contains the core trust algorithm. It is treated as a sacred, unmodifiable file across all Builder work.
**Why:** This is OTI's protected intellectual property — the core competitive differentiator. Changes to the algorithm would directly affect every score ever computed and could constitute liability if a change degraded results for a paying enterprise customer. The algorithm's correctness is also non-trivial to verify without thorough testing. All adaptation work (chain-aware weighting, signal transformers) is done in wrapper files that sit alongside it, never inside it.
**Confirmed by:** Ahmad (explicit, repeated instruction across all sessions)
**Implications for fixes:** Any fix that touches scoring behavior — weight redistribution, signal computation, tier cutoffs — must be implemented in an adapter or transformer layer. Never edit `scoring.ts` itself.

---

### D3 — BSC, Base, and Optimism Return 503 Intentionally
**Status:** INTENTIONAL
**What the code does:** Requests to `?chain=bnb`, `?chain=base`, and `?chain=optimism` return HTTP 503 "This chain requires an Etherscan plan upgrade."
**Why:** These three chains require the Etherscan Lite plan ($49/month) to access their API. The code to support them is already implemented — it is gated behind a 503 at the route level pending Ahmad's decision to subscribe.
**Confirmed by:** Ahmad (explicit decision, documented across ARCHITECTURE.md and MANAGER_HANDOVER.md)
**Implications:** These chains are NOT bugs, missing implementations, or oversights. Do not attempt to "fix" the 503. The gate is intentional. When Ahmad decides to subscribe, a Builder removes the gate — that is the only change needed.

---

### D4 — Admin Page Is URL-Only (No Navigation Link)
**Status:** INTENTIONAL
**What the code does:** The admin panel at `/admin` has no link from any public page. It is accessible only if you know the URL.
**Why:** Ahmad's deliberate decision. The admin panel is for his eyes only — no public visibility, no discoverability by ordinary users or crawlers. Security by obscurity is a secondary layer on top of the `x-admin-secret` header requirement.
**Confirmed by:** Ahmad (explicit decision, documented July 7, 2026)
**Implications:** Do not add a navbar link, a footer link, or any discoverable reference to `/admin` on any public-facing page.

---

### D5 — Bitcoin Weight Redistribution (40% Across 3 Remaining Signals)
**Status:** INTENTIONAL
**What the code does:** On Bitcoin, Token Holding Behavior (20%) and Smart Contract Interactions (20%) are excluded. Their combined 40% weight is redistributed proportionally to the three remaining signals: Wallet Age (25% → 41.7%), Transaction Count (20% → 33.3%), Transaction Timing Patterns (15% → 25%).
**Why:** Bitcoin has no native token standard and no smart contract layer. Scoring a Bitcoin wallet as 0/20 on Token Holding and 0/20 on Smart Contract Interactions would penalize the wallet for what its chain was architecturally not designed to support — not for any bad behavior by the wallet. The correct behavior is to exclude the inapplicable signals and redistribute their weight so the score remains on a 0–100 scale and measures only what is genuinely measurable. The wallet is scored honestly.
**Confirmed by:** Ahmad (explicit decision made during BF10 architectural discussion, July 11, 2026)
**Implications for whitepaper:** The whitepaper must explain this redistribution principle openly. The weights table shown (25/20/20/20/15) applies to EVM chains and chains where all 5 signals are available. Bitcoin is the primary exception. The explanation belongs in the whitepaper and must NOT name any internal files.
**Implications for other chains:** If any other chain is found to genuinely lack one or more signal types, the same redistribution principle applies — confirmed by Ahmad as the general policy, not a Bitcoin-only special case.

---

### D6 — 30-Day Score Validity Window
**Status:** INTENTIONAL
**What the code does:** A computed score is valid for 30 days. Requests within that window return the cached score from the database without fetching fresh chain data.
**Why:** Ahmad's decision made during the BF13 architectural discussion (July 11, 2026). Two purposes: (1) Scale — most repeat requests on any scored address return from the database instantly, with zero external API calls. (2) Trust decay — on-chain behavior is not static. A wallet that scored 75 today might be compromised or used for wash trading in a month. Monthly rescoring catches wallets that turn malicious over time without requiring constant re-evaluation.
**Confirmed by:** Ahmad (explicit decision, July 11, 2026)
**Implications for BF13:** The DB cache fix (BF13) must use this 30-day window. Admin panel controls a rolling daily rescore period so not all wallets expire at the same time.

---

### D7 — WOR Compromise Reports Are Fully Automated (No Admin Review)
**Status:** INTENTIONAL
**What the code does (planned):** When the Wallet Ownership Registry (WOR) is built, a verified wallet owner will be able to submit a compromise report by combining a wallet signature with a pre-registered passkey. On success, the wallet is immediately flagged with `compromised: true` and score 0 — no admin queue, no review step.
**Why:** Ahmad's flagship design decision. The passkey pre-registration is the differentiator — an attacker who has the private key cannot replicate it because the passkey was registered before the compromise. The combination of wallet signature + passkey hash match is sufficient proof that the requester is the original owner, not the attacker. Manual review would introduce delay and a human error surface that the passkey system eliminates.
**Confirmed by:** Ahmad (explicit decision, documented across ROADMAP.md and MANAGER_HANDOVER.md)
**Implications:** When building WOR (Phase 2), do not add any admin review queue or approval step. The automated path is the product.

---

### D8 — Scroll, Sepolia, and Holesky Are Not Implemented
**Status:** INTENTIONAL (with documentation correction needed)
**What the code does:** Requests to `?chain=scroll`, `?chain=sepolia`, and `?chain=holesky` return HTTP 400 "Invalid enum value." These chains do not exist in the codebase.
**Why:** These chains were listed in early documentation as planned/supported but were never implemented. Sepolia and Holesky are Ethereum testnets — supporting testnets in a production trust-scoring product is of limited value. Scroll is an L2 that would require the same Etherscan Lite plan as BSC/Base/Optimism.
**Confirmed by:** Diagnostic audit, July 12, 2026 (BF23)
**Status of documentation:** All public-facing materials (docs site, whitepaper, UI chain selector) incorrectly list these as supported. This is tracked as BF23 in FIXES.md. The chain count across all materials must drop from 15 to 12 until these are genuinely implemented. Do not implement them to avoid the fix — remove them from documentation instead.

---

### D9 — Fantom Chain ID Points to Sonic (146 Instead of 250)
**Status:** REVISIT — confirmed as a bug, not an intentional decision
**What the code does:** `CHAIN_ID.fantom = "146"` — queries Sonic Mainnet data and labels it as Fantom Opera.
**Why this happened:** Unknown — likely a data entry error at the time the chain was added. Chain ID 146 is Sonic Mainnet; Fantom Opera is chain ID 250. No intentional reason to query Sonic under the Fantom label has been identified.
**Confirmed as bug by:** Diagnostic audit, July 12, 2026 (BF22)
**Fix:** Change `CHAIN_ID.fantom` from `"146"` to `"250"`. Single-line change. Tracked in FIXES.md as BF22.

---

## Pending Answers — Awaiting Builder Response

The following behaviors were observed in the July 12, 2026 diagnostic audit. Before treating them as bugs, the Builder has been asked to explain the reasoning. Do not assign fix prompts for these items until the Manager has reviewed the answers and updated the entries below.

---

### D10 — EVM Transaction Fetch Is Capped at a Single Page (1,000 / 500 / 500)
**Status:** PENDING ANSWER
**What the code does:** EVM chains fetch a single page: 1,000 native transactions, 500 token transfers, 500 internal transactions, oldest-first, no pagination loop. Active wallets (e.g. vitalik.eth) are silently truncated.
**Question sent to Builder:** Was this a deliberate decision (rate-limit protection, API cost control) or the first thing that worked?
**Why this matters before fixing:** If it was deliberate rate-limit protection, fixing it needs a pagination strategy with rate-limit awareness — not just a loop. If it was an oversight, the BF10-style pagination fix applies directly.

---

### D11 — EVM Token Holdings Computed From Transfer History, Not Real Balance
**Status:** PENDING ANSWER
**What the code does:** Token holding diversity on EVM chains counts distinct contract addresses in the `tokentx` transfer event log. A wallet that received and fully sent away 100 tokens still shows high token diversity.
**Question sent to Builder:** Was this a deliberate simplification (one API call vs. many balance calls), an API limitation, or an oversight?
**Why this matters before fixing:** Real balance queries require either individual `tokenbalance` calls per token (expensive) or a multicall contract (complex). If the Builder knows a cleaner approach exists, that shapes the fix. If it was purely an oversight, we scope the replacement method.

---

### D12 — Tron and Solana Transactions Use the Wallet's Own Address as `to` Field
**Status:** PENDING ANSWER
**What the code does:** For Tron and Solana, the transaction shaping code sets `to: walletAddress` (self-referential) rather than the actual contract/program address. This collapses smart contract diversity to ≤1 for any wallet.
**Question sent to Builder:** Was this a deliberate simplification to keep a consistent data shape across chains, or was there no straightforward way to extract the real counterpart address at the time?
**Why this matters before fixing:** If the real program ID / contract address is accessible in the raw API response, this is a straightforward data extraction fix. If it requires a separate API call per transaction, the approach changes significantly.

---

### D13 — TON Jetton Holdings Inferred From Outgoing Messages
**Status:** PENDING ANSWER (likely Technical Limitation)
**What the code does:** TON token holdings are inferred by examining outgoing Jetton-related message destinations rather than querying actual current balances.
**Question sent to Builder:** Did Toncenter v2 have no direct owner→Jetton-holdings endpoint at the time? Is Toncenter v3 or tonapi.io now available and usable?
**Note:** The diagnostic audit report already states "Toncenter v2 has no such direct endpoint" — this is likely a TECHNICAL LIMITATION that is now resolved by newer API versions. Status will likely move to REVISIT once confirmed.

---

### D14 — EVM Rate-Limit Errors Cached as Legitimate Zero-Activity Scores
**Status:** PENDING ANSWER
**What the code does:** When Etherscan returns a `NOTOK` (rate-limit) response, the fetcher treats it as "no data" and the result — score 0, txCount 0 — gets written to cache as a real score for up to 5 minutes. Confirmed live on Polygon and Linea.
**Question sent to Builder:** Was caching error states a deliberate "fail fast, don't hammer a down API" design choice, or was this an oversight in the non-array result check?
**Why this matters:** If deliberate, the fix needs to preserve the intent while not writing false data (e.g. cache a "try again" state, not a zero score). If an oversight, the fix is simply: throw on non-array / NOTOK responses, never cache them.

---

### D15 — Sui Token Diversity Counts All Owned Object Types, Not Just Coins
**Status:** PENDING ANSWER
**What the code does:** `suix_getOwnedObjects` returns all owned Move objects. The code counts distinct object types as "token diversity" — including NFTs and capability objects, not just `Coin<T>` types.
**Question sent to Builder:** Was this a deliberate "total asset diversity" measure, or was the intent always to count coins specifically and the all-objects approach was an approximation?
**Why this matters:** If deliberate, the signal measures something real (total portfolio diversity). If an oversight, switching to `suix_getAllCoins` gives a cleaner coin-specific count. Either answer is defensible — we just need to know which was intended.
