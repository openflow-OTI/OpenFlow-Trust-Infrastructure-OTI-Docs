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

### D9 — "Fantom" Chain Entry Rebranded to Sonic (Chain ID Stays 146)
**Status:** INTENTIONAL (resolved) — was a mislabeling bug, now fixed by rebranding the entry rather than repointing the chain ID
**What the code did before the fix:** `CHAIN_ID.fantom = "146"` — queried Sonic Mainnet data and labeled it Fantom Opera.
**Why this happened:** Data entry error when the chain was added — 146 is Sonic Mainnet, not Fantom Opera (250).
**Live-verification finding (July 12, 2026, BF22):** Switching to the "correct" Fantom Opera chain ID (250) does not work — Etherscan V2's chainlist (64 chains) no longer includes Fantom Opera under any chain ID; Fantom migrated/rebranded to Sonic and Etherscan V2 only serves the Sonic entry (146). Fantom's legacy standalone explorer domain (ftmscan.com) no longer resolves at all. Real, distinct Fantom Opera data is not obtainable from our data source.
**Ahmad's decision (July 12, 2026):** Rename the chain entry itself from `"fantom"` to `"sonic"`, keep chain ID 146. This scores real data (Sonic Mainnet, which is what Fantom Opera became) under its real, current name instead of either lying about the label (old bug) or dropping the chain entirely.
**Fix:** In `etherscan.ts`, rename the `fantom` key throughout `CHAIN_ID` and the reverse lookup table to `sonic` (chain ID stays `"146"`). Update any chain name strings/labels/docs surfaced to API callers (e.g. `/api/chains`, response `chain` field) from "Fantom"/"Fantom Opera" to "Sonic"/"Sonic Mainnet". Tracked in FIXES.md as BF22.

---

## Pending Answers — Awaiting Builder Response

The following behaviors were observed in the July 12, 2026 diagnostic audit. Before treating them as bugs, the Builder has been asked to explain the reasoning. Do not assign fix prompts for these items until the Manager has reviewed the answers and updated the entries below.

---

### D10 — EVM Transaction Fetch Is Capped at a Single Page (1,000 / 500 / 500)
**Status:** REVISIT — confirmed as a bug, not a deliberate rate-limit safeguard
**What the code does:** EVM chains fetch a single page: 1,000 native transactions, 500 token transfers, 500 internal transactions, oldest-first, no pagination loop. Active wallets (e.g. vitalik.eth) are silently truncated.
**Live test evidence (July 12, 2026):** Sequential pagination (8 requests, no delay) completed cleanly at ~1.2 req/s. Concurrent fire of 15 requests hit Etherscan's real throttle — a hard **3 requests/second** cap — not the single-page cap currently coded. Separately, Etherscan enforces its own hard ceiling of **10,000 total records** reachable via `page × offset` on this endpoint, regardless of pacing. There is no evidence the 1,000-cap was chosen for rate-limit protection — the real constraints (3 req/s, 10k-record ceiling) are far more generous than what the code currently uses.
**Fix scope:** Paginate up to Etherscan's real ceiling (10 pages at offset=1000), respecting the 3 req/s throttle. For vitalik.eth specifically, no signal score changed when the cap was lifted (all affected signals were already saturated), but this cannot be assumed for less active wallets sitting mid-tier — the current cap risks under-scoring them. Tracked for a fix prompt in `FIXES.md`.

---

### D11 — EVM Token Holdings Computed From Transfer History, Not Real Balance
**Status:** REVISIT — confirmed as a bug via live counterfactual
**What the code does:** Token holding diversity on EVM chains counts distinct contract addresses in the `tokentx` transfer event log. A wallet that received and fully sent away 100 tokens still shows high token diversity.
**Live test evidence (July 12, 2026):** On vitalik.eth, 11 of 236 transfer-log tokens (including MKR, TheDAO) were verified via Etherscan's `tokenbalance` action to have a real current balance of **0**, yet still count as "held." For this specific wallet the tier didn't shift (236 vs. 225 both land in the same `≥20` tier), but a constructed counterfactual using the same 11 confirmed-zero tokens showed an **80-point delta on the signal (16 points overall)** between the current transfer-log method and a real-balance check. This is a genuine, quantified inaccuracy, not a hypothetical one.
**Fix scope:** Etherscan's `tokenbalance` action gives a real balance per token/address — confirmed working live. Per-token calls at scale need a batching/rate-limit-aware approach (see D10's confirmed 3 req/s ceiling). Tracked for a fix prompt in `FIXES.md`.

---

### D12 — Tron and Solana Transactions Use the Wallet's Own Address as `to` Field
**Status:** REVISIT — confirmed as a straightforward bug
**What the code does:** For Tron and Solana, the transaction shaping code sets `to: walletAddress` (self-referential) rather than the actual contract/program address. This collapses smart contract diversity to ≤1 for any wallet.
**Live test evidence (July 12, 2026):** Solana's raw `getTransaction` response exposes real, distinct `programId` values directly in `transaction.message.instructions[]`. Tron's raw transaction detail exposes `toAddress`, `contractData.contract_address`, and `trigger_info.parameter._to` — all real, distinct from the wallet's own address. The real counterparty data is present and directly extractable in both cases; the current code simply never reads these fields.
**Fix scope:** Pure data-extraction fix — no additional API calls needed for either chain. Tracked for a fix prompt in `FIXES.md`.

---

### D13 — TON Jetton Holdings Inferred From Outgoing Messages
**Status:** TECHNICAL LIMITATION (confirmed) — resolution path under active discussion, do not close
**What the code does:** TON token holdings are inferred by examining outgoing Jetton-related message destinations rather than querying actual current balances.
**Confirmed:** Toncenter v2 genuinely has no direct owner→Jetton-holdings endpoint — this is a real technical limitation, not an oversight.
**Why this is not simply closed:** Ahmad's explicit direction — inferring holdings from outgoing messages only can still produce inaccurate ("lying") scores (e.g. a wallet that sent away all its Jettons may still read as holding them, or vice versa), so this cannot just be filed away as an accepted limitation. Next step: evaluate Toncenter v3 and tonapi.io, both of which expose real Jetton balance endpoints, as a replacement data source before deciding how to proceed. Manager + Ahmad to decide the replacement approach once the current bug-fix round (D9-D12, D14) is complete.

---

### D14 — EVM Rate-Limit Errors Cached as Legitimate Zero-Activity Scores
**Status:** REVISIT — confirmed as a bug, reproducible on demand
**What the code does:** When Etherscan returns a `NOTOK` (rate-limit) response, the fetcher treats it as "no data" and the result — score 0, txCount 0 — gets written to cache as a real score.
**Live test evidence (July 12, 2026):** A single concurrent burst across 7 chains hit the false-zero pattern on 4 of 6 Etherscan-backed chains (ethereum, polygon, arbitrum, fantom, linea all affected; avalanche threw an error instead of swallowing). Re-querying the affected chains returned `"cached":true` with the identical degraded score — the response shape is indistinguishable from a real cache hit (no `error`/`stale`/`degraded` flag). A client cannot tell a real zero-activity wallet from a rate-limit-corrupted one. No evidence of a deliberate "fail fast" design anywhere in the response handling.
**Fix scope:** Never cache a `NOTOK`/non-array response. Throw or retry instead of treating it as legitimate zero-activity data. Tracked for a fix prompt in `FIXES.md`.

---

### D15 — Sui Token Diversity Counts All Owned Object Types, Not Just Coins
**Status:** REVISIT (proposed) — recommend product call from Ahmad, not a pure bug
**What the code does:** `suix_getOwnedObjects` returns all owned Move objects. The code counts distinct object types as "token diversity" — including NFTs and capability objects, not just `Coin<T>` types.
**Live test evidence (July 12, 2026):** On a real mixed-holdings wallet, the current method (single 50-item page of `suix_getOwnedObjects`) scored diversity 100 (9 distinct types incl. NFTs, hit the page cap). A coin-only method (`suix_getAllCoins`, fully paginated) found 19 real distinct coin types → tier 80, a 20-point signal delta. But at matched shallow pagination depth, coin-only collapsed to just 1 type (many individual SUI coin objects filled the single page before any other coin type appeared) → tier 20, an 80-point delta in the *opposite* direction. **The shallow single-page cap is a bug regardless of which scope is chosen** — it needs full pagination either way.
**Why this needs Ahmad, not just a fix:** The signal is named "Token Holding Behavior," which argues for coins-only (NFTs are not tokens in the fungible sense). But no evidence anywhere suggests the all-objects approach was deliberate "total portfolio diversity" design either — it reads as an approximation, not a documented choice. Recommend Ahmad confirm scope (coins-only vs. total-asset diversity) before a fix prompt is written; the pagination-depth bug is unconditionally fixable in either case.
