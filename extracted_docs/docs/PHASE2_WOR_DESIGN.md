# OTI — Phase 2: Wallet Ownership Registry (WOR) — Full Design Brief
> Created: July 14, 2026 (session 16) | Author: Development Manager
> Status: Design complete — awaiting Ahmad review before Builder prompts are written.
> This doc is the single source of truth for all Phase 2 design decisions. Builder prompts will be derived directly from it. Nothing here is implementation detail for Builders to decide — it is specification.

---

## What WOR Is (One Paragraph)

WOR lets a wallet owner pre-register their ownership via off-chain cryptographic signing + a secret passkey phrase. If the wallet is later compromised — even if the attacker has the private key — the original owner can connect the wallet, enter the passkey, and instantly flag it as compromised with a 0-score. The passkey is the differentiator: an attacker can replicate a wallet signature but cannot know a passkey registered before the attack. No admin review. No blockchain write. Fully automated. This is Ahmad's flagship trust feature.

---

## What Needs to Be Built (Component Map)

| Component | Builder | Depends On |
|---|---|---|
| `wallet_ownership` DB table | Backend | Nothing — first thing to build |
| `GET /api/wallet/register/challenge` | Backend | DB table |
| `POST /api/wallet/register` | Backend | DB table + challenge |
| `GET /api/wallet/registration-status/:address` | Backend | DB table |
| `POST /api/report/compromised` | Backend | DB table + `compromised_wallets` (exists) |
| `wallet_links` routes (activates existing table) | Backend | `wallet_ownership` table |
| Admin WOR endpoints (4 endpoints) | Backend | DB table |
| Registration UI (`/register`) | Frontend | All backend endpoints live |
| Report Compromise UI (`/report`) | Frontend | All backend endpoints live |
| Admin WOR tab (2 sub-views + override) | Frontend | Admin WOR endpoints live |
| "Report this wallet" link activation | Frontend | Report UI page exists |

**Sequencing rule:** Backend ships and deploys first. Frontend task only starts after Backend confirms all endpoints are live on Railway. Two separate sequential task prompts.

---

## DB Tables

### NEW: `wallet_ownership`

```
id                UUID        PRIMARY KEY DEFAULT gen_random_uuid()
wallet_address    TEXT        NOT NULL UNIQUE   -- normalized: EVM lowercase; others: canonical form
chain_family      TEXT        NOT NULL          -- 'evm' | 'solana' | 'bitcoin' | 'ton' | 'tron' | 'sui'
passkey_hash      TEXT        NOT NULL          -- bcrypt hash of user's passkey phrase (never store raw)
reg_message       TEXT        NOT NULL          -- exact challenge message that was signed (audit trail)
signature         TEXT        NOT NULL          -- wallet signature proving ownership at registration time
registered_at     TIMESTAMPTZ NOT NULL DEFAULT now()
last_verified_at  TIMESTAMPTZ                   -- updated on each successful passkey-verified action
status            TEXT        NOT NULL DEFAULT 'active'  -- 'active' | 'compromised' | 'revoked'
```

**Notes:**
- `wallet_address` is the primary lookup key. UNIQUE constraint — one registration per address.
- `passkey_hash` uses bcrypt (cost factor 12). Never returned in any API response.
- `reg_message` and `signature` are stored for audit purposes only — to prove the registration was legitimate if ever disputed.
- `chain_family` is a validation hint and determines which signature verification function to use.
- Phase 2 implements EVM (`evm`) chain family fully. The schema and code structure must support other chain families so they can be added in future without a schema change. Non-EVM signing functions are stubs in Phase 2 — they throw "not yet supported" rather than silently failing.

### EXISTING: `wallet_links`

Already in the DB. Phase 2 activates it by building the routes that read/write it. No schema change needed. The Backend Builder must inspect the actual table schema in Railway before writing any queries — do not assume columns from the Drizzle schema file (same raw-SQL rule as `subscriptions`).

### EXISTING: `compromised_wallets`

Already in the DB and powering the red banner. WOR's `POST /api/report/compromised` writes into this table. The Backend Builder must inspect its actual column names in Railway before writing queries.

### No `wor_nonces` table

Replay protection is handled by embedding a timestamp in the challenge message and rejecting anything signed more than 15 minutes ago. This avoids a third new table and is stateless on the backend.

---

## Backend Endpoints — Full Specification

### 1. `GET /api/wallet/register/challenge`

**Purpose:** Generate the exact message the wallet must sign to prove ownership. Public endpoint, no auth.

**Query params:** `?address={wallet_address}&chain_family={evm|solana|...}`

**Response:**
```json
{
  "message": "OTI Wallet Ownership Registration\nAddress: 0x...\nTimestamp: 2026-07-14T10:30:00.000Z\nDo not sign this message on any other platform.",
  "address": "0x...",
  "expires_at": "2026-07-14T10:45:00.000Z"
}
```

**Rules:**
- Backend generates this message deterministically from address + current UTC timestamp (ISO 8601).
- The message is returned to the frontend so the user can read it before signing. Never hide what the user is signing.
- No state stored on the backend — the message content itself is the nonce. Backend verifies the embedded timestamp on submission.
- `expires_at` = 15 minutes from `Timestamp`.
- Address validation: run through the existing `validateAddress` chain-aware validator. Return 400 if invalid.

---

### 2. `POST /api/wallet/register`

**Purpose:** Complete registration — verifies the signature, hashes the passkey, stores the record.

**Body:**
```json
{
  "address": "0x...",
  "chain_family": "evm",
  "message": "<the exact challenge message returned by /challenge>",
  "signature": "<wallet signature of that message>",
  "passkey": "<user's chosen passkey phrase, plaintext — hashed server-side immediately>"
}
```

**Backend processing (in order):**
1. Validate `address` format via existing `validateAddress`.
2. Parse `Timestamp:` from `message`. Reject if older than 15 minutes (return 400 "Challenge expired").
3. Verify the message text matches the expected template for the given address and timestamp (prevent signature reuse on a crafted message).
4. Recover the signer from `signature` using EIP-191 verification (`ethers.verifyMessage(message, signature)` or equivalent). Compare recovered address to submitted `address` (both lowercase). Reject if mismatch (return 401 "Signature does not match address").
5. Check `wallet_ownership` for an existing record with this `address`. If found and `status = 'active'`, return 409 "Address already registered".
6. Validate `passkey`: min 8 characters, reject if empty or whitespace-only.
7. `bcrypt.hash(passkey, 12)` → `passkey_hash`.
8. INSERT into `wallet_ownership`: address, chain_family, passkey_hash, reg_message (= message), signature, registered_at = now(), status = 'active'.
9. Return 201:

```json
{
  "registered": true,
  "address": "0x...",
  "registered_at": "2026-07-14T10:30:00.000Z"
}
```

**Error cases:** 400 (invalid address / expired challenge / weak passkey), 401 (signature mismatch), 409 (already registered), 500 (DB failure).

---

### 3. `GET /api/wallet/registration-status/:address`

**Purpose:** Let the frontend (and any public caller) check whether an address is registered. Public endpoint.

**Response:**
```json
{
  "address": "0x...",
  "registered": true,
  "status": "active"
}
```
or
```json
{
  "address": "0x...",
  "registered": false
}
```

**Rules:** Never return `passkey_hash`, `signature`, `reg_message`, or any sensitive field. `status` only returned when `registered: true`.

---

### 4. `POST /api/report/compromised`

**Purpose:** Instantly flag a wallet as compromised. The owner proves they are the original registrant via signature + passkey. No admin review.

**Body:**
```json
{
  "address": "0x...",
  "message": "<fresh challenge message from /challenge, signed right now>",
  "signature": "<wallet signature of that fresh message>",
  "passkey": "<the passkey set during registration>"
}
```

**Backend processing (in order):**
1. Validate `address` format.
2. Look up `wallet_ownership` by `address`. If not found → return 404 "Address not registered with OTI — register first to enable self-reporting".
3. If `status = 'compromised'` already → return 409 "This wallet is already flagged".
4. Parse and verify timestamp in `message` (15-min window, same as registration).
5. Verify signature → recover address → confirm match.
6. `bcrypt.compare(passkey, stored_passkey_hash)`. If mismatch → return 401 "Incorrect passkey". **Do not reveal which factor failed in the error body — return a generic "Verification failed" to the frontend.**
7. Both factors verified. In a transaction:
   a. UPDATE `wallet_ownership` SET status = 'compromised', last_verified_at = now() WHERE address = ...
   b. INSERT OR IGNORE into `compromised_wallets` (address) — use existing table structure (inspect real Railway columns first)
   c. Invalidate any cached score for this address (call the same cache-invalidation path used by `DELETE /api/admin/cache/:address`)
8. Return 200:
```json
{
  "flagged": true,
  "address": "0x...",
  "flagged_at": "2026-07-14T10:30:00.000Z",
  "message": "Wallet flagged as compromised. All future OTI score queries will show a compromised warning."
}
```

**Error cases:** 400, 401 (generic "Verification failed" — never expose which factor), 404, 409.

**Security note:** The generic 401 body is intentional. If the error reveals "wrong passkey" vs "wrong signature", an attacker who has the key can enumerate passkeys more effectively. "Verification failed" reveals nothing about which factor failed.

---

### 5. `GET /api/wallet/links/:address`

**Purpose:** Return all wallets linked to the same owner identity as the given address.

**Response:**
```json
{
  "address": "0x...",
  "linked_wallets": ["0x...", "0x..."],
  "count": 2
}
```

If address not registered or no links exist: `{ "address": "0x...", "linked_wallets": [], "count": 0 }`. Never 404 — always return the shape.

---

### 6. `POST /api/wallet/links`

**Purpose:** Link two wallets as belonging to the same owner. Both must be registered. Proves ownership of both via signatures + passkey of one.

**Body:**
```json
{
  "primary_address": "0x...",
  "secondary_address": "0x...",
  "primary_message": "<fresh challenge for primary address>",
  "primary_signature": "<primary wallet's signature>",
  "secondary_message": "<fresh challenge for secondary address>",
  "secondary_signature": "<secondary wallet's signature>",
  "passkey": "<passkey for primary address>"
}
```

**Backend processing:**
1. Verify both addresses are registered and `status = 'active'`.
2. Verify both challenge timestamps within 15 minutes.
3. Verify `primary_signature` recovers to `primary_address`.
4. Verify `secondary_signature` recovers to `secondary_address`.
5. Verify `passkey` against `primary_address`'s `passkey_hash`.
6. Check that a link between these two addresses doesn't already exist.
7. INSERT into `wallet_links`.
8. Return 201 `{ "linked": true, "primary": "0x...", "secondary": "0x..." }`.

---

### 7. `GET /api/admin/wor/registrations` (admin auth required)

**Query params:** `?page=1&limit=50&search={address_prefix}&status=active|compromised`

**Response:** Paginated list.
```json
{
  "registrations": [
    {
      "address": "0x...",
      "chain_family": "evm",
      "registered_at": "...",
      "last_verified_at": "...",
      "status": "active"
    }
  ],
  "total": 142,
  "page": 1,
  "limit": 50
}
```

Never return `passkey_hash`, `signature`, or `reg_message` even to admin.

---

### 8. `GET /api/admin/wor/registrations/:address` (admin auth required)

Returns single registration record. Same field exclusion rule — no passkey_hash, no raw signature.

---

### 9. `POST /api/admin/wor/flag` (admin auth required)

**Purpose:** Admin force-compromise — bypass passkey, flag any address.

**Body:** `{ "address": "0x...", "reason": "optional admin note" }`

**Processing:** If address is in `wallet_ownership`, UPDATE status = 'compromised'. Always INSERT OR IGNORE into `compromised_wallets`. Invalidate score cache for address. Return 200 `{ "flagged": true }`.

**Note:** This works for ANY address — even unregistered ones. An address doesn't need to be in WOR for the admin to add it to `compromised_wallets`. Same behavior as the existing admin panel's "flag wallet" function — WOR just adds the additional `wallet_ownership` status update.

---

### 10. `DELETE /api/admin/wor/flag/:address` (admin auth required)

**Processing:** Remove from `compromised_wallets`. If in `wallet_ownership` and status = 'compromised', UPDATE status = 'active'. Invalidate score cache. Return 200 `{ "unflagged": true }`.

---

## EIP-191 Signature Verification — Implementation Notes for Backend Builder

Use `ethers.verifyMessage(message, signature)` (ethers v6) or `viem`'s `verifyMessage`. Do not implement the signing math manually.

The pattern:
```
recovered = ethers.verifyMessage(message, signature)  // returns the signer's address
match = recovered.toLowerCase() === submitted_address.toLowerCase()
```

**Which library to use:** Check the existing backend's `package.json`. If `ethers` or `viem` is already installed, use it. Do not add a new library when one already does the job. If neither is present, add `ethers@6` — it's the standard.

**Non-EVM chains:** For Phase 2, only `chain_family = 'evm'` is fully implemented. If a non-EVM address is submitted to `/register`, return 400 "Non-EVM chain registration not yet supported — EVM wallets only in Phase 2". The `chain_family` column and code structure must be extensible so Solana/TON/etc. can be added later without a schema rewrite.

---

## Frontend Flows — Full Specification

### Registration Page — `/register`

**Route:** `/register`
**Access:** Public. Linked from the "⚑ Report this wallet" ghost link on the results page AND from a new standalone entry point (footer or homepage "Find Us/Integrations" section — to be determined by Frontend Builder based on what fits the existing layout).

**Mobile-first, 375px, same black/mint design system as the rest of the app.**

#### Step 1 — Enter Address

- Single text input: "Your wallet address"
- Chain selector: same component as the score page (EVM chains only in Phase 2; non-EVM chains are greyed out with a tooltip "Coming soon")
- Helper text: "You'll be asked to sign a message with this wallet to prove ownership."
- "Continue →" button

Validation: run client-side `validateAddress` before allowing Continue. If invalid format, show inline error.

#### Step 2 — Sign to Prove Ownership

- Page shows the exact challenge message text (fetched from `GET /api/wallet/register/challenge`) in a readable block. User can read it before signing.
- "Sign with wallet" button → calls `window.ethereum.request({ method: 'eth_personal_sign', params: [message, address] })` (MetaMask / any injected provider).
- If no `window.ethereum` detected: show "Please install MetaMask or another Ethereum wallet to continue." Do not crash.
- On signature success: move to step 3. Show the address and a ✓ "Ownership verified" confirmation.
- On user rejection (user clicks "Cancel" in MetaMask): show "Signature cancelled. Click below to try again." Do not navigate away.

#### Step 3 — Set Your Passkey

- Label: "Create your OTI passkey"
- Helper text: "This is a secret phrase only you know. If your wallet is ever compromised, you'll need it to instantly flag it. OTI never stores this in plain text — we only store a hash."
- Input 1: "Passkey phrase" (type=password, min 8 chars)
- Input 2: "Confirm passkey" (must match)
- Strength hint: character count only (no password-strength scoring library needed — just show green text when ≥ 8 chars)
- "Register Wallet →" button → calls `POST /api/wallet/register`
- On success: show success screen (below). On error: inline error message per the API response.

**⚠️ Important:** The passkey is submitted once and never stored in localStorage, sessionStorage, or any browser storage. No "remember passkey" feature. The user is responsible for remembering it.

#### Success Screen

> ✓ Wallet Registered
> Your wallet `0x...` is now registered with OTI.
> 
> If your wallet is ever compromised, visit [/report] and use your passkey to instantly flag it.
> 
> [Report a Compromise →] [Score Another Wallet →]

---

### Report Compromise Page — `/report`

**Route:** `/report`
**Access:** Public. The "⚑ Report this wallet" link on the results page links here with `?address={current_address}` pre-filled. Users can also navigate here directly.

**Activate the existing placeholder:** Task 8 already built a ghost "⚑ Report this wallet" link on the results page. Phase 2 is when this link becomes real. The Frontend Builder updates it to point to `/report?address={currentAddress}` — it should no longer be a dead placeholder.

#### Step 1 — Identify Wallet

- Wallet address input (pre-filled from `?address=` URL param if present — validate and show, but allow editing)
- Chain selector (EVM only for Phase 2)
- Check registration status via `GET /api/wallet/registration-status/:address`. Show inline:
  - If registered and active: ✓ "This wallet is registered with OTI. Continue to prove ownership."
  - If not registered: ⚠ "This wallet is not registered with OTI. You must register before you can self-report a compromise." + [Register →] link to `/register`.
  - If already compromised: ℹ "This wallet is already flagged as compromised."
- "Continue →" button (only enabled if wallet is registered and active)

#### Step 2 — Sign to Prove Ownership

Same as registration step 2 — fetch a fresh challenge from `GET /api/wallet/register/challenge`, display the message, request MetaMask signature.

Label difference: The helper text should say "This proves you control the wallet — even if an attacker also controls it, your passkey is the second proof only you have."

#### Step 3 — Enter Your Passkey

- Label: "Enter your OTI passkey"
- Single input (type=password) — no confirm field, no strength hint
- Helper text: "The passkey you set when you registered. If you've forgotten it, contact OTI support."
- "Flag as Compromised" button → calls `POST /api/report/compromised`
- Warning: show a confirmation dialog before submitting: "This will immediately flag your wallet with a 0-score trust warning visible to everyone who queries it. Are you sure?" [Confirm] [Cancel]

#### Success Screen

> ⚠ Wallet Flagged as Compromised
> `0x...` is now flagged. Any OTI score query for this wallet will show a compromised warning.
>
> [Score This Wallet to Verify →] [Register Another Wallet →]

#### Error states:
- 401 from API: "Verification failed. Check that you're signing from the correct wallet and entering the exact passkey you registered with."
- 404 from API: redirect to Step 1 with the "not registered" message.
- Generic 500: "Something went wrong. Please try again."

---

### Admin Panel — WOR Tab

**New tab:** "WOR" — added to the existing admin panel tab strip, between existing tabs (exact position: after "Cache", before a future "Phase 2B" tab if applicable — for now just append it).

Uses the exact same tab/table/card design patterns as the existing admin panel. No new component patterns.

#### Sub-view 1: Registry

Table columns:
| Address | Chain | Status | Registered | Last Verified | Actions |
|---|---|---|---|---|---|
| 0x...abc (truncated, copy button) | EVM | 🟢 Active / 🔴 Compromised | Jul 14 2026 | Jul 14 2026 | [Flag] |

- Search input: filter by address prefix (client-side on the loaded page, or server-side via `?search=` param — server-side preferred for large datasets)
- Status filter: All / Active / Compromised
- Pagination: 50 per page
- "Flag" action button: triggers the manual override (calls `POST /api/admin/wor/flag`) — shows confirmation dialog first

#### Sub-view 2: Compromised Wallets

Shows ALL addresses currently in `compromised_wallets` table — both WOR self-reported and admin-flagged.

Table columns:
| Address | Source | Flagged At | Actions |
|---|---|---|---|
| 0x...abc | WOR Self-Report / Admin Override | Jul 14 2026 | [Remove Flag] |

"Source" column: if address exists in `wallet_ownership` with status = 'compromised', source = "WOR Self-Report". Otherwise, source = "Admin Override".

"Remove Flag" → calls `DELETE /api/admin/wor/flag/:address` — shows confirmation dialog.

#### Manual Override Panel

Below both sub-views, a simple card:

> **Manual Flag**
> Enter any wallet address to immediately flag it as compromised (no passkey required).
> [Address input] [Chain selector] [Flag Wallet] button
> → calls `POST /api/admin/wor/flag`

---

## Passkey Design — Clarification for Builders

This is NOT WebAuthn/FIDO2/biometric ("passkeys" in the Apple/Google sense). It is Ahmad's custom concept: a user-chosen secret text phrase, stored as a bcrypt hash on the OTI backend. It functions like a second password — the first factor is wallet signature (proves key control), the second factor is the passkey (proves pre-registration knowledge that an attacker can't infer from the private key alone).

Do not implement WebAuthn. Do not implement SMS/OTP. Just a text phrase, bcrypt-hashed, stored in `wallet_ownership.passkey_hash`.

---

## wallet_links Table — Scope for Phase 2

The routes are built in Phase 2 but the UI for wallet portfolio management is Phase 4. In Phase 2, wallet linking is an API-only capability. No frontend UI for linking wallets is needed in Phase 2 — just the backend endpoints (`POST /api/wallet/links` and `GET /api/wallet/links/:address`). The admin WOR tab does not need to display wallet links. This scoping keeps Phase 2 tight.

---

## Sacred Files — Reminder for Builders

Backend: `src/lib/scoring.ts` (never touch), `nixpacks.toml` (never touch), `railway.json` (only touch if explicitly told to).
Frontend: `vercel.json` (never touch), `src/api/schema.gen.ts` (run `pnpm codegen` to regenerate, never manually edit).

---

## Railway Migration Note

After the Backend Builder creates the `wallet_ownership` table via Drizzle schema and pushes the code, Ahmad must manually run `drizzle-kit push` against the Railway production `DATABASE_URL` to apply the migration. This is standard procedure (BF12 automates part of it, but confirmation is still needed). Backend Builder must remind Ahmad of this in their completion report — no schema change is live until Ahmad runs the migration.

---

## Definition of Done — Backend Task

1. `wallet_ownership` table schema committed and migration instructions written clearly for Ahmad.
2. All 10 endpoints (1–10 above) implemented and live on Railway.
3. EIP-191 verification tested against at least 2 real EVM wallets (fresh signatures, not cached).
4. `POST /api/report/compromised` tested end-to-end: register → report → verify `compromised_wallets` has the address and `GET /api/score/:address` returns `"compromised": true` with score 0.
5. `GET /api/wallet/registration-status/:address` verified to never return `passkey_hash` or `signature`.
6. All error cases return correct HTTP status codes (400/401/404/409) with JSON bodies (not HTML).
7. D16 applies: all verification steps must be tested with real wallets and real signatures — not code-inspection-only or mocked data.
8. Report to Manager with: endpoint URLs, Railway deployment link, test wallet addresses used, raw API responses showing each major flow working.

## Definition of Done — Frontend Task

1. `/register` page live on Vercel, mobile-verified at 375px, full 3-step flow working end-to-end with a real MetaMask signature.
2. `/report` page live, pre-fill from `?address=` param working, registration status check working (all 3 states visible).
3. "⚑ Report this wallet" link on results page now points to `/report?address={currentAddress}` (no longer a dead placeholder).
4. Admin WOR tab live: Registry sub-view with real registered wallets visible, Compromised sub-view accurate, Manual Override functional.
5. No new component libraries introduced. Color system tokens used exactly.
6. Report to Manager with: Vercel preview link, screenshots of each step on both pages (mobile 375px + desktop), admin WOR tab screenshot.

---

## Open Questions — Resolved

- **Do non-EVM chains register in Phase 2?** No. EVM-only. Return 400 for non-EVM. Design is extensible.
- **Can an address register more than once?** No. UNIQUE constraint. If already registered active, return 409.
- **Can a compromised wallet be re-registered?** No. Compromised status is permanent via self-report. Admin can override (unflag via DELETE endpoint), after which the original registration record is reactivated.
- **Is the passkey recoverable?** No. bcrypt is one-way. If forgotten, the owner cannot self-report. They can contact OTI (Ahmad) for admin override. This is by design — the passkey's value comes from its non-recoverability by attackers.
- **Does the score API change?** No. `compromised_wallets` already powers `"compromised": true` in score responses. WOR just adds a new way to write to that table.

---

## What Is NOT in Phase 2

- No UI for wallet portfolio / aggregated trust (Phase 4)
- No non-EVM chain signing (Phase 2 EVM only — stubs for later)
- No passkey reset / recovery flow (by design — admin override is the fallback)
- No email or notification on compromise flag (Phase 4 webhook work)
- No badge / attestation (Phase 2B — comes after Phase 2 ships)
