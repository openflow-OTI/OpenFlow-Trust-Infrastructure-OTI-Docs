# Whitepaper Additions Draft
_Manager workspace — content ready for Frontend Builder to implement_

---

## ⚠️ RECOMMENDATION: REMOVE SECTION 11 — THE OPENFLOW VISION

Section 11 does not belong in this whitepaper. The OTI whitepaper is a technical and product document read by developers, integrators, and enterprise buyers deciding whether to use the API. Section 11 talks about FLOW tokens, the attention economy, private and public token offerings, and OpenFlow's broader mission — none of which is relevant to someone evaluating OTI as infrastructure. It makes the document read like a hybrid investor pitch, which undermines the credibility of the technical sections around it.

**Recommendation:** Remove Section 11 from the whitepaper entirely. That content belongs in a separate OpenFlow ecosystem or investor brief. The OpenFlow Vision can be referenced at the end of the document with a single line — "OTI is the first product in the OpenFlow ecosystem. Learn more at openflow.xyz" — without embedding the full pitch here.

Everything below is additive. Nothing in the existing whitepaper needs to be rewritten or removed except Section 11.

---

## ADDITION 1 — New subsections inside Section 05: How OTI Works

These go directly after the existing Trust Tiers table in Section 05, before Section 06.

---

### How a Score Request Is Processed

When a wallet address is submitted to OTI, the following sequence executes in full before a score is returned. Every step is deterministic — the same address on the same chain, at the same point in time, will always produce the same result.

**1. Address Validation**

The submitted address is validated against the structural rules of the specified chain before any data is fetched. Each chain has its own format rules:

- **EVM chains (Ethereum, Polygon, Arbitrum, and all other EVM networks):** must be a valid 42-character hex address beginning with `0x`, either checksummed or lowercase.
- **Bitcoin:** accepts four formats — Legacy addresses beginning with `1` (P2PKH), Script addresses beginning with `3` (P2SH), native SegWit addresses beginning with `bc1q` (bech32), and Taproot addresses beginning with `bc1p` (bech32m).
- **Solana:** must be a valid base58-encoded public key, 32–44 characters.
- **TON:** accepts user-friendly format addresses beginning with `EQ` or `UQ` (48 characters total), as well as raw format addresses beginning with `0:` followed by 64 hex characters.
- **Tron:** must begin with `T`, 34 characters, base58-encoded.
- **Sui:** must begin with `0x` followed by 64 hex characters, 66 characters total.

If the address fails validation, the request is rejected immediately with a structured error. No external call is made.

**2. Compromised Wallet Check**

Before any scoring begins, the address is checked against OTI's compromised wallet registry — a database of addresses that have been flagged as stolen, drained, or associated with known attacks. This check runs against OTI's own database and requires no external API call.

If the address is found in the registry, a score of `0` is returned immediately, with `"compromised": true` in the response. The request ends here. No signal computation occurs.

**3. Cache Check**

OTI queries its own database for an existing score for this address and chain combination. Scores are valid for 30 days from the time they were computed. If a valid score exists within that window, it is returned immediately — `"cached": true` in the response — and the request completes without any external blockchain data fetch. This is the typical path for any address that has been scored before.

**4. Chain Routing and Data Fetching**

If no valid cached score exists, the request is routed to the appropriate data layer for the specified chain. OTI maintains separate data fetching pipelines for each chain family:

- **EVM chains** share a unified pipeline backed by the Etherscan V2 API, which gives OTI access to full transaction history, ERC-20 token holdings, and contract interaction records across all supported EVM networks through a single consistent interface.
- **Bitcoin** connects to a UTXO ledger API. Transaction history is paginated — OTI does not apply an arbitrary cap on how many transactions it fetches. Full history is retrieved regardless of wallet age or transaction volume.
- **Solana** connects via Solana RPC. Both SPL tokens and Token-2022 standard tokens are included. Transaction history is paginated across all signature pages.
- **TON** connects via Toncenter, covering Jetton holdings and full message history.
- **Tron** connects via TronScan, covering TRC-20 and TRC-10 token holdings and full contract interaction history.
- **Sui** connects via Sui RPC, covering Move object ownership and transaction history.

Where the chain's API allows it, the data required for all five signals is fetched in parallel to minimize response time. All fetched data reflects the live on-chain state at the moment of the request.

**5. Chain-Aware Signal Computation**

Each of the five behavioral signals is computed independently from the raw chain data. The computation is chain-aware: signals that are structurally impossible on a given chain — because that chain was not designed to support the behavior the signal measures — are excluded from the calculation rather than scored as zero.

This matters because penalizing a wallet for what its chain cannot do would produce dishonest scores. A Bitcoin wallet cannot hold ERC-20 tokens — not because the wallet has bad behavior, but because Bitcoin has no token standard. Scoring it zero on Token Holding would artificially reduce its score for a reason unrelated to the wallet's trustworthiness.

Instead, when a signal cannot apply to a chain, its weight is redistributed proportionally across the signals that do apply. The total always sums to 100%. The wallet is only judged on what is measurable.

Bitcoin is the most significant example: it has no native token standard and no smart contract layer. Token Holding Behavior and Smart Contract Interactions are excluded entirely. Their combined 40% weight is redistributed across the three remaining signals — Wallet Age, Transaction Count, and Transaction Timing Patterns — in proportion to their original weights.

**6. Score Aggregation**

Once all applicable signals are computed and weights are adjusted, the overall score is calculated:

```
Overall Score = Σ (signal_score × adjusted_weight)
```

The result is a single integer between 0 and 100. Because the weights always sum to 1.0 regardless of how many signals apply, the score always sits within the same 0–100 range across all chains. Scores are directly comparable across chains without any chain-specific interpretation.

**7. Response and Asynchronous Storage**

The score and full signal breakdown are assembled into a JSON response and returned to the client immediately. Storage does not block the response.

After the response is delivered, the computed score is written to OTI's database asynchronously. It becomes the cache source for all future requests for the same address on the same chain within the 30-day freshness window. If the new score is the highest score this address has ever received, the highest-ever record is updated in the same write.

---

### What the Response Contains

Every OTI score response returns the same structure, regardless of chain:

```json
{
  "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
  "chain": "ethereum",
  "cached": false,
  "compromised": false,
  "score": 87,
  "signals": {
    "walletAge":                 { "score": 100, "weighted": 25, "maxWeight": 25 },
    "transactionCount":          { "score": 100, "weighted": 20, "maxWeight": 20 },
    "tokenHoldingBehavior":      { "score": 90,  "weighted": 18, "maxWeight": 20 },
    "smartContractInteractions": { "score": 85,  "weighted": 17, "maxWeight": 20 },
    "transactionTimingPatterns": { "score": 46,  "weighted": 7,  "maxWeight": 15 }
  }
}
```

- `score` — the overall trust score, 0–100
- `cached` — `true` if the score was returned from the database without a fresh chain fetch
- `compromised` — `true` if the address is in the compromised wallet registry; score will be 0
- `signals` — each signal's raw `score` (0–100), how many points it `weighted` toward the overall score, and its `maxWeight` cap (what it would contribute at a perfect 100)

The signal breakdown makes every score fully auditable. Any integrator can verify exactly how the overall score was reached by inspecting each signal's contribution.

For chains where a signal does not apply, that signal's entry will show `score: 0`, `weighted: 0`, and `maxWeight: 0` — indicating it was excluded from this chain's calculation, not that the wallet scored zero on it.

---

### Score Validity and the 30-Day Freshness Model

OTI scores are valid for 30 days from the time they are computed. This window exists because on-chain behavior is not static — a wallet that scores 75 today may score differently in a month if it has been compromised, drained, or used for wash trading. Fresh scores better reflect current reality.

When a score expires, the next request for that address triggers a full fresh computation from live chain data. There is no manual action required from the API consumer — the cache check in step 3 handles this transparently.

OTI also maintains a highest-ever score record for every address it has scored. This record is not returned in the standard API response but is used internally to support trust trend analysis and the future self-reporting compromise system.

---

## ADDITION 2 — Expand Section 06: Supported Infrastructure

Add the following after the existing chain list in Section 06.

---

### Signal Applicability by Chain

Not all five behavioral signals are available on every blockchain. Some chains were architecturally designed without token standards or programmable contracts — and OTI scores those chains honestly rather than penalizing wallets for their chain's design.

The table below shows which signals apply natively per chain and how their absence affects weight distribution. When a signal is marked as not applicable, its weight is redistributed proportionally across the signals that do apply. The total weight always sums to 100%.

| Chain | Wallet Age | Tx Count | Token Holding | Smart Contract | Timing Patterns |
| --- | --- | --- | --- | --- | --- |
| Ethereum | ✅ | ✅ | ✅ | ✅ | ✅ |
| Polygon | ✅ | ✅ | ✅ | ✅ | ✅ |
| Arbitrum | ✅ | ✅ | ✅ | ✅ | ✅ |
| Avalanche | ✅ | ✅ | ✅ | ✅ | ✅ |
| Fantom | ✅ | ✅ | ✅ | ✅ | ✅ |
| Linea | ✅ | ✅ | ✅ | ✅ | ✅ |
| Scroll | ✅ | ✅ | ✅ | ✅ | ✅ |
| zkSync | ✅ | ✅ | ✅ | ✅ | ✅ |
| Sepolia | ✅ | ✅ | ✅ | ✅ | ✅ |
| Holesky | ✅ | ✅ | ✅ | ✅ | ✅ |
| Solana | ✅ | ✅ | ✅ | ✅ | ✅ |
| Tron | ✅ | ✅ | ✅ | ✅ | ✅ |
| TON | ✅ | ✅ | ✅ | ✅ | ✅ |
| Sui | ✅ | ✅ | ✅ | ✅ | ✅ |
| Bitcoin | ✅ | ✅ | ❌ No token standard | ❌ No smart contract layer | ✅ |

**Bitcoin weight redistribution:**
Bitcoin has no native token standard and no programmable smart contract layer. Token Holding Behavior and Smart Contract Interactions are excluded from the Bitcoin calculation. Their combined 40% weight is redistributed across the three applicable signals:

| Signal | Standard Weight | Bitcoin Weight |
| --- | --- | --- |
| Wallet Age | 25% | 41.7% |
| Transaction Count | 20% | 33.3% |
| Token Holding Behavior | 20% | — |
| Smart Contract Interactions | 20% | — |
| Transaction Timing Patterns | 15% | 25.0% |

_Note: The non-EVM signal data in this table (Solana, Tron, TON, Sui) will be updated following a full chain diagnostic audit currently in progress. Bitcoin and all EVM chains are confirmed._

---

## ADDITION 3 — New Section: Integrating OTI

**Placement:** Between Section 07 (Use Cases) and Section 08 (WOR). This becomes the new Section 08, and WOR becomes Section 09 (all subsequent sections shift by one).

---

### Integrating OTI

OTI exposes a single REST endpoint. No SDK, no client library, no blockchain infrastructure required. Any application that can make an HTTP GET request can integrate OTI in minutes.

**Base URL**

```
https://workspaceapi-server-production-5c0c.up.railway.app/api
```

**Score a wallet**

```
GET /score/{address}?chain={chain}
```

**Parameters**

| Parameter | Location | Required | Description |
| --- | --- | --- | --- |
| `address` | Path | Yes | The wallet address to score |
| `chain` | Query | Yes | The chain identifier (e.g. `ethereum`, `bitcoin`, `solana`) |
| `x-api-key` | Header | No | API key for keyed-tier access. Omit for anonymous tier. |

**Authentication tiers**

| Tier | How to use | Daily limit |
| --- | --- | --- |
| Anonymous | No key, no header | 100 requests/day per IP |
| Developer | `x-api-key: oti_<key>` header | Higher daily limit |
| Pro | `x-api-key: oti_<key>` header | High volume (paid) |
| Enterprise | Direct contract | Unlimited + SLA |

API keys follow the format `oti_<32 hex characters>`. Keys are available on request from the OTI team — a self-serve request flow is coming soon.

**CORS**

The API accepts requests from any origin. No proxy required. OTI can be called directly from frontend JavaScript in the browser.

**Full example**

```bash
curl "https://workspaceapi-server-production-5c0c.up.railway.app/api/score/0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045?chain=ethereum" \
  -H "x-api-key: oti_your_key_here"
```

```json
{
  "address": "0xd8da6bf26964af9d7eed9e03e53415d37aa96045",
  "chain": "ethereum",
  "cached": false,
  "compromised": false,
  "score": 87,
  "signals": {
    "walletAge":                 { "score": 100, "weighted": 25, "maxWeight": 25 },
    "transactionCount":          { "score": 100, "weighted": 20, "maxWeight": 20 },
    "tokenHoldingBehavior":      { "score": 90,  "weighted": 18, "maxWeight": 20 },
    "smartContractInteractions": { "score": 85,  "weighted": 17, "maxWeight": 20 },
    "transactionTimingPatterns": { "score": 46,  "weighted": 7,  "maxWeight": 15 }
  }
}
```

Every `signals` entry is auditable: `score` is the raw signal score (0–100), `weighted` is its point contribution to the overall score, and `maxWeight` is the maximum it could have contributed. For chains where a signal does not apply, all three fields return `0`.

The full API reference — all endpoints, error codes, chain identifiers, and code examples in JavaScript, Python, and cURL — is available at the API documentation site.

---

## ADDITION 4 — New Section: Frequently Asked Questions

**Placement:** Final section, after Contact & Links. Or immediately before Contact & Links if a closing section feels better to Ahmad.

---

### Frequently Asked Questions

**What happens between submitting a wallet and receiving a score?**

The request goes through five stages in order: address format validation, compromised wallet check, database cache lookup, live chain data fetching (if no valid cache exists), and signal computation with chain-aware weight distribution. Each stage can terminate the request early — invalid addresses are rejected at stage one, compromised wallets are returned at stage two, cached addresses are returned at stage three. A full fresh computation only runs for addresses that pass all three checks and have no valid cached score.

**How long does scoring take?**

Cached responses return in milliseconds — they require no external API call. Fresh computations depend on the chain's API response times, but typically complete within one to three seconds. High-throughput use cases should factor in that most repeat requests on scored addresses return from cache.

**How does OTI know which blockchain a wallet is on?**

You specify the chain explicitly in the request. A wallet address alone is not enough — the same string of characters can be a valid address on multiple chains simultaneously (a `0x` address is valid on every EVM chain, for example). OTI uses the chain parameter to route the request to the correct data source and apply the correct address validation rules.

**What external data sources does OTI use?**

EVM chains are sourced from the Etherscan V2 API, which gives OTI consistent access to transaction history, token holdings, and contract interactions across all supported EVM networks. Non-EVM chains each have their own dedicated data source: Solana uses Solana RPC, Bitcoin uses a UTXO ledger API, TON uses Toncenter, Tron uses TronScan, and Sui uses the Sui RPC. All data sourced by OTI is publicly verifiable on-chain by anyone.

**What does `"cached": true` mean in the response?**

It means OTI found a valid score for this address in its database that was computed within the last 30 days, and returned that instead of fetching fresh data. The score is still accurate — it reflects the wallet's on-chain state at the time it was last computed. If you need a score that reflects state within the last few seconds, the cached response will note when it was originally computed.

**Why might the same wallet score differently on different days?**

OTI scores reflect the wallet's on-chain state at the time of computation. Within the 30-day freshness window, the same cached score is returned on every request. After 30 days, the next request triggers a fresh computation from current chain data — and if the wallet's behavior has changed (more transactions, new token holdings, different timing patterns), the score will reflect that. Wallets that become more active over time will generally score higher. Wallets that are drained or used for suspicious activity will score lower.

**Can OTI score a wallet that has never sent a transaction?**

Yes. OTI computes scores on any address that exists on a chain, regardless of whether it has sent anything. A wallet that has only received funds will score low on Transaction Count and Transaction Timing Patterns because those signals reflect outgoing activity, but it will not cause an error. Wallet Age for receive-only wallets is calculated from the first time the address appeared on-chain — whether inbound or outbound.

**Why does my Bitcoin wallet score 0 for Token Holding and Smart Contract Interactions?**

Bitcoin has no native token standard and no programmable smart contract layer. These two signals are structurally inapplicable on Bitcoin — they show as 0 not because the wallet scored zero, but because the signals were excluded from the calculation. Their weight was redistributed to the three signals that do apply on Bitcoin: Wallet Age, Transaction Count, and Transaction Timing Patterns. The wallet is not penalized for this. See the Signal Applicability by Chain table in Section 06.

**What chains does OTI support?**

15 chains are live: Ethereum, Polygon, Arbitrum, Avalanche, Fantom, Linea, Scroll, zkSync, Sepolia, Holesky (all EVM), and Bitcoin, Solana, TON, Tron, and Sui (non-EVM). Three chains — BNB Smart Chain, Base, and Optimism — are temporarily unavailable pending an infrastructure upgrade and will be added in a near-term release.

**Does OTI score the same address across multiple chains?**

Yes, but each chain produces a separate score. A `0x` address that is active on Ethereum and Polygon will have a distinct score on each chain — because it may have different transaction histories, token holdings, and contract interactions across them. Specify the chain in each request to get the score for that chain.

**What makes a wallet "Highly Trusted" versus "High Risk"?**

Trust tiers reflect the combination of all five signals. A Highly Trusted wallet (85–100) is typically one that has been active for years, has a high transaction volume, holds a diverse range of tokens over time, regularly interacts with smart contracts, and has transaction timing that reflects organic human behavior rather than bot patterns. A High Risk wallet (0–24) is typically brand new, has minimal or no transaction history, holds nothing meaningful, and shows no contract engagement. A Trusted score does not mean a wallet is safe in all contexts — it means OTI's behavioral analysis found no indicators of risk in the on-chain data.

**Can a compromised wallet ever score high?**

It can score high based on its historical on-chain behavior — long-established wallets that get compromised will have accumulated strong behavioral signals. That is exactly why OTI maintains a separate compromised wallet registry. If a wallet is flagged as compromised — either through OTI's Wallet Ownership Registry (when it launches) or via the OTI team — it returns `"compromised": true` and a score of 0 regardless of its historical signals. A high behavioral score and a compromised flag are not contradictory; they measure different things.

**How do I get an API key?**

Keys are available on request from the OTI team. Contact details are in the Contact & Links section. A self-serve key request flow is coming soon. The anonymous tier (100 requests/day, no key required) is sufficient to fully build and test an integration before requesting a key.

**What does the full API reference cover?**

All endpoints, request parameters, response fields, chain identifier strings, error codes and their meanings, and copy-paste code examples in JavaScript, Python, and cURL are available in the API documentation.

**Does OTI store wallet data?**

OTI stores the computed score, signal breakdown, and computation timestamp for each address it has scored. No personal information is collected — OTI does not know who submitted a request, only that a request was made. Wallet addresses are public on-chain identifiers; scoring them does not constitute collection of personal data.

**Can I request my data be deleted?**

Wallet address scores can be removed from OTI's database on request. Contact the OTI team via the details in the Contact & Links section.

---

## PLACEMENT SUMMARY

| Addition | Where it goes |
| --- | --- |
| Remove Section 11 (OpenFlow Vision) | Delete the section; optionally leave a one-line pointer to the OpenFlow ecosystem site at the end of Section 13 |
| "How a Score Request Is Processed" | Subsection inside Section 05, after the Trust Tiers table |
| "What the Response Contains" | Subsection inside Section 05, after the lifecycle |
| "Score Validity and the 30-Day Freshness Model" | Subsection inside Section 05, after the response section |
| "Signal Applicability by Chain" table | Subsection inside Section 06, after the chain list |
| "Integrating OTI" | New Section 08 (current 08 WOR shifts to 09, all following sections shift by 1) |
| "Frequently Asked Questions" | New final section before Contact & Links |
