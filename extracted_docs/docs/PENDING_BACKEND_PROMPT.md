# Pending Backend Builder Prompt
> Created: July 6, 2026
> Status: UNSENT — Backend Builder hit credit limit. Send this when he is back online.

---

## What to send:

```
New task added to your queue: TASK 12 — Signal Accuracy Audit & Cross-Chain Fix.
This is a pre-distribution task — do not start it until the Manager assigns it.
Add it to BACKEND_TASKS.md now so it is on record.

---

TASK 12 — Backend: Signal Accuracy Audit & Cross-Chain Fix
Phase: Pre-Distribution
Priority: CRITICAL
Depends on: All Phase 4 tasks complete before starting

Context:
The 5 scoring signals were built for EVM chains. Bitcoin, Solana, TON, Tron, and Sui have
different data models — no ERC-20 tokens, no internal transactions, no smart contracts.
These chains are currently being scored with EVM logic, producing wrong results.
Confirmed example: Satoshi genesis wallet scores 51 days wallet age (correct is ~5,700+ days),
"1 smart-contract tx" on a Bitcoin wallet (Bitcoin has no smart contracts),
0/20 Token Holding on BTC (no ERC-20 tokens exist on Bitcoin).

What to audit and fix — per signal:

SIGNAL 1 — Wallet Age (25%)
- Bitcoin: Task 7D attempted a fix. Re-verify against the Satoshi genesis wallet
  (1A1zP1eP5QGefi2DMPTfTL5SLmv7DivfNa) after flushing cache. Must return > 5,000 days.
  If still wrong, re-investigate the timestamp source from mempool.space.
- Solana, TON, Tron, Sui: verify each chain returns the correct first-tx timestamp.

SIGNAL 2 — Transaction Count (20%)
- Verify all non-EVM chains return real tx count, not a placeholder or default value.

SIGNAL 3 — Token Holding Behavior (20%)
- Bitcoin has NO tokens. Do not score 0/20 by default — it unfairly penalises BTC wallets.
  Fix: if chain is Bitcoin, score this signal as neutral (50/100) or redistribute its
  weight across the other 4 signals.
- Solana has SPL tokens — verify SPL token diversity is being scored, not defaulting to 0.
- TON, Tron, Sui: verify token data source and scoring logic for each.

SIGNAL 4 — Smart Contract Interactions (20%)
- Bitcoin has NO smart contracts. Do not fabricate data. Apply same fix as Signal 3
  (neutral score or weight redistribution).
- Solana: program interactions are the equivalent — verify this is being scored correctly.
- TON, Tron, Sui: verify each chain's equivalent of contract interactions.

SIGNAL 5 — Transaction Timing Patterns (15%)
- Do not use internal transaction counts as a proxy for non-EVM chains.
  Use actual transaction timestamps for timing analysis on all chains.

Definition of done:
- Satoshi genesis wallet returns wallet age > 5,000 days
- No signal fabricates data for chains where that data type does not exist
- Non-applicable signals are scored neutral, never 0 by default
- All 15 supported chains tested with at least one known wallet
- Short test log committed to the repo documenting results

---

Confirm you have added Task 12 to BACKEND_TASKS.md. Do not start any other work.
```
