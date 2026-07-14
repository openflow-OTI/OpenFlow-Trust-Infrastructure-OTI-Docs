# Backend Builder Prompt — Phase 2: Wallet Ownership Registry (WOR)
> Created: July 14, 2026 | Status: READY TO SEND — awaiting Ahmad to paste to Backend Builder

---

## The prompt (copy this entire block):

```
You have a new task: Phase 2 — Wallet Ownership Registry (WOR). Read this prompt fully before writing a single line of code.

─────────────────────────────────────────────────────────────
WHAT YOU'RE BUILDING AND WHY
─────────────────────────────────────────────────────────────

WOR lets a wallet owner pre-register their ownership via off-chain cryptographic signing + a secret passkey phrase. If the wallet is ever compromised — even if the attacker has the private keys — the original owner can connect the wallet, enter the passkey, and instantly flag it as compromised with a 0-score warning. No admin review. No blockchain write. Fully automated.

The passkey is the differentiator: an attacker who has the private key can replicate a wallet signature, but they cannot know a passkey phrase that was registered before the attack. That combination — wallet signature (key control) + passkey (pre-registration knowledge) — is what makes WOR a genuine trust primitive, not just another "flag a wallet" button.

This is Ahmad's flagship feature for OTI.

─────────────────────────────────────────────────────────────
BEFORE YOU WRITE ANY CODE — READ THESE FILES
─────────────────────────────────────────────────────────────

In order:
1. BUILDER_ONBOARDING.md — your role, the team, sacred files
2. ARCHITECTURE.md — every DB table and what it does
3. DECISIONS.md — why things exist the way they do.
   Read D7 (WOR automated, no admin review — non-negotiable),
   D16 (evidence standard — applies to everything you verify).

─────────────────────────────────────────────────────────────
SACRED FILES — NEVER TOUCH
─────────────────────────────────────────────────────────────

src/lib/scoring.ts — NEVER MODIFY (core IP, protected algorithm)
nixpacks.toml — NEVER TOUCH (Railway build config)
railway.json — do not touch unless explicitly told to

─────────────────────────────────────────────────────────────
D16 — EVIDENCE STANDARD (non-negotiable)
─────────────────────────────────────────────────────────────

Every claim of "verified" or "tested" in your completion report must be backed by real evidence — a real wallet address you tested against, a real raw API response showing the result, not code-inspection alone. This applies to every endpoint in this task. The Manager will ask "what specific wallet, what specific raw response?" before accepting any close. Prepare for that question.

─────────────────────────────────────────────────────────────
subscriptions TABLE — CRITICAL REMINDER
─────────────────────────────────────────────────────────────

The subscriptions table in Railway does NOT match the Drizzle schema in the repo (real columns: id, api_key, plan, owner_address, created_at, expires_at, updated_at — no status, no email). All handlers that touch it use raw SQL. Apply the same discipline to any new table: inspect actual Railway column names before writing queries. Do not trust the Drizzle schema file for tables that were created before the current codebase.

─────────────────────────────────────────────────────────────
PHASE 2 SCOPE — BACKEND ONLY
─────────────────────────────────────────────────────────────

Your task is backend only. The Frontend Builder has a separate task that cannot start until your endpoints are live on Railway. Do not start on any frontend code.

Phase 2 signing scope: EVM chains only (EIP-191). The schema and code must be extensible so non-EVM chains can be added later — but in this phase, if a non-EVM address is submitted to /register, return 400 "Non-EVM chain registration not yet supported — EVM wallets only in Phase 2". No stub implementations for non-EVM — just a clean 400 with that message.

─────────────────────────────────────────────────────────────
NEW DB TABLE: wallet_ownership
─────────────────────────────────────────────────────────────

Create this table via Drizzle schema. Column definitions:

  id               UUID        PRIMARY KEY DEFAULT gen_random_uuid()
  wallet_address   TEXT        NOT NULL UNIQUE
                               (normalized: EVM = lowercase;
                                others = canonical form per chain)
  chain_family     TEXT        NOT NULL
                               ('evm' | 'solana' | 'bitcoin' |
                                'ton' | 'tron' | 'sui')
  passkey_hash     TEXT        NOT NULL
                               (bcrypt hash, cost factor 12 —
                                NEVER return this in any API response)
  reg_message      TEXT        NOT NULL
                               (exact challenge message that was signed —
                                stored for audit trail only)
  signature        TEXT        NOT NULL
                               (the wallet's registration signature —
                                stored for audit trail only)
  registered_at    TIMESTAMPTZ NOT NULL DEFAULT now()
  last_verified_at TIMESTAMPTZ (updated on each successful passkey action)
  status           TEXT        NOT NULL DEFAULT 'active'
                               ('active' | 'compromised' | 'revoked')

wallet_address UNIQUE constraint — one registration per address.
passkey_hash is bcrypt (never stored in plain text, never returned in any response).
reg_message and signature are stored for audit purposes only.

⚠ After pushing schema changes, Ahmad must manually run:
  drizzle-kit push against the Railway production DATABASE_URL
Tell him this in your completion report. The schema change is not live until he runs it.

─────────────────────────────────────────────────────────────
EXISTING TABLES YOU WILL WRITE TO
─────────────────────────────────────────────────────────────

compromised_wallets — already in DB, already powering the red banner.
  Inspect its real column names in Railway before writing any INSERT.
  Use raw SQL, not Drizzle ORM, same as subscriptions.

wallet_links — already in DB. Your POST /api/wallet/links and
  GET /api/wallet/links/:address activate it.
  Inspect real column names in Railway before writing queries.

─────────────────────────────────────────────────────────────
EIP-191 SIGNATURE VERIFICATION
─────────────────────────────────────────────────────────────

Use ethers.verifyMessage(message, signature) (ethers v6) or
viem's verifyMessage. Check existing package.json first — do not
add a new library if one already does the job. If neither ethers
nor viem is installed, add ethers@6.

Pattern:
  const recovered = ethers.verifyMessage(message, signature)
  const match = recovered.toLowerCase() === address.toLowerCase()

Do not implement the signing math manually.

─────────────────────────────────────────────────────────────
PASSKEY — WHAT IT IS
─────────────────────────────────────────────────────────────

This is NOT WebAuthn / FIDO2 / biometric passkeys.
It is a user-chosen text phrase, bcrypt-hashed server-side (cost 12),
stored in wallet_ownership.passkey_hash.
Use bcrypt for hashing. No SMS, no OTP, no browser credential API.

─────────────────────────────────────────────────────────────
REPLAY PROTECTION — NO NONCES TABLE NEEDED
─────────────────────────────────────────────────────────────

Embed a UTC timestamp (ISO 8601) in every challenge message.
On submission, parse that timestamp and reject anything older
than 15 minutes (return 400 "Challenge expired").
The message text itself is the nonce. No separate DB table needed.

─────────────────────────────────────────────────────────────
ENDPOINTS TO BUILD (10 total)
─────────────────────────────────────────────────────────────

════════════════════════════════════
PUBLIC ENDPOINTS (no auth)
════════════════════════════════════

── 1. GET /api/wallet/register/challenge ──────────────────────

Query params: ?address={addr}&chain_family={evm|...}
Purpose: Returns the exact message the wallet must sign.

Challenge message format (use exactly this template):
  "OTI Wallet Ownership Registration
  Address: {address}
  Timestamp: {ISO-8601 UTC timestamp}
  Do not sign this message on any other platform."

Response:
  {
    "message": "<full message text as above>",
    "address": "0x...",
    "expires_at": "<timestamp + 15 minutes, ISO-8601>"
  }

Rules:
- Validate address via existing validateAddress utility first.
  Return 400 if invalid.
- No state stored server-side. The message itself contains all
  replay-protection information.
- The message text is returned in full so the frontend can display
  it to the user before they sign. Never hide what the user is signing.

── 2. POST /api/wallet/register ───────────────────────────────

Body:
  {
    "address": "0x...",
    "chain_family": "evm",
    "message": "<the exact challenge message from /challenge>",
    "signature": "<wallet signature of that message>",
    "passkey": "<user's passkey phrase, plaintext>"
  }

Processing (execute in this exact order — stop on first failure):
  1. Validate address format via validateAddress → 400 if invalid
  2. If chain_family != 'evm' → 400 "Non-EVM chain registration
     not yet supported — EVM wallets only in Phase 2"
  3. Parse "Timestamp: " line from message. If message is older
     than 15 minutes → 400 "Challenge expired"
  4. Verify message text matches expected template for this
     address and timestamp (prevent crafted-message attacks)
  5. EIP-191: recover signer from signature. If recovered address
     (lowercase) != submitted address (lowercase) → 401 "Signature
     does not match address"
  6. Query wallet_ownership WHERE wallet_address = address.
     If found and status = 'active' → 409 "Address already registered"
  7. Validate passkey: min 8 chars, non-empty/whitespace → 400
     "Passkey must be at least 8 characters"
  8. bcrypt.hash(passkey, 12) → passkey_hash
  9. INSERT into wallet_ownership:
     (wallet_address, chain_family, passkey_hash, reg_message,
      signature, registered_at, status)
     reg_message = the full message string
 10. Return 201:
     { "registered": true, "address": "0x...",
       "registered_at": "<ISO timestamp>" }

Error codes summary: 400 / 401 / 409 / 500

── 3. GET /api/wallet/registration-status/:address ───────────

Purpose: Let anyone check whether an address is registered.
No auth required.

Response when registered:
  { "address": "0x...", "registered": true, "status": "active" }

Response when not registered:
  { "address": "0x...", "registered": false }

⚠ NEVER return passkey_hash, signature, reg_message, or
  last_verified_at in this response. Status only.

── 4. POST /api/report/compromised ────────────────────────────

Purpose: Instantly flag a registered wallet as compromised.
Proves ownership via wallet signature + passkey. No admin review.
(This is D7 — the whole point of WOR.)

Body:
  {
    "address": "0x...",
    "message": "<fresh challenge message, signed right now>",
    "signature": "<wallet signature of that fresh message>",
    "passkey": "<passkey set during registration>"
  }

Processing (in this order):
  1. Validate address format → 400 if invalid
  2. Query wallet_ownership WHERE wallet_address = address.
     If not found → 404 "Address not registered with OTI.
     Register first to enable self-reporting."
  3. If status = 'compromised' → 409 "Wallet is already flagged"
  4. Parse timestamp from message. If >15 min old → 400
     "Challenge expired. Request a fresh challenge and try again."
  5. EIP-191: recover signer. If mismatch → 401 "Verification failed"
  6. bcrypt.compare(passkey, stored passkey_hash).
     If mismatch → 401 "Verification failed"

  ⚠ CRITICAL SECURITY RULE:
  If step 5 OR step 6 fails, ALWAYS return 401 "Verification failed"
  with NO indication of which factor was wrong. If the error reveals
  "wrong passkey" vs "wrong signature", an attacker with the stolen
  key can enumerate passkeys. Generic message, always.

  7. Both factors verified. Execute in a DB transaction:
     a. UPDATE wallet_ownership
        SET status = 'compromised', last_verified_at = now()
        WHERE wallet_address = address
     b. INSERT into compromised_wallets (use raw SQL, inspect
        real Railway column names first)
        Use INSERT ... ON CONFLICT DO NOTHING if address already there
     c. Invalidate score cache for this address — use the same
        cache-invalidation code path as
        DELETE /api/admin/cache/:address

  8. Return 200:
     {
       "flagged": true,
       "address": "0x...",
       "flagged_at": "<ISO timestamp>",
       "message": "Wallet flagged as compromised. All future OTI
                   score queries will show a compromised warning."
     }

Error codes: 400 / 401 / 404 / 409 / 500

── 5. GET /api/wallet/links/:address ──────────────────────────

Purpose: Return all wallets linked to the same owner identity
as the given address.

Response:
  {
    "address": "0x...",
    "linked_wallets": ["0x...", "0x..."],
    "count": 2
  }

If not registered or no links: always return the shape with an
empty array. Never 404.

── 6. POST /api/wallet/links ──────────────────────────────────

Purpose: Link two wallets as belonging to the same owner.
Both must be registered. Proves ownership of both.

Body:
  {
    "primary_address": "0x...",
    "secondary_address": "0x...",
    "primary_message": "<fresh challenge for primary>",
    "primary_signature": "<primary wallet signature>",
    "secondary_message": "<fresh challenge for secondary>",
    "secondary_signature": "<secondary wallet signature>",
    "passkey": "<passkey for primary address>"
  }

Processing:
  1. Verify both addresses are in wallet_ownership, both
     status = 'active' → 404 if either is missing, 409 if
     either is compromised
  2. Verify both message timestamps within 15 minutes
  3. Verify primary_signature → primary_address (EIP-191)
  4. Verify secondary_signature → secondary_address (EIP-191)
  5. Verify passkey against primary address's passkey_hash
  6. Check wallet_links for existing link between these two
     addresses (either direction) → 409 if already linked
  7. INSERT into wallet_links (inspect real Railway column
     names first)
  8. Return 201:
     { "linked": true,
       "primary": "0x...", "secondary": "0x..." }

════════════════════════════════════
ADMIN ENDPOINTS (x-admin-secret header required)
Apply existing adminAuth.ts middleware — same as all other
/api/admin/* routes.
════════════════════════════════════

── 7. GET /api/admin/wor/registrations ────────────────────────

Query params:
  ?page=1&limit=50&search={address_prefix}&status=active|compromised

Response:
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

⚠ Never return passkey_hash, signature, or reg_message — even to admin.

── 8. GET /api/admin/wor/registrations/:address ───────────────

Returns single registration record. Same field exclusion rule.

── 9. POST /api/admin/wor/flag ────────────────────────────────

Purpose: Admin force-flag any address as compromised.
No passkey needed. Works for ANY address — even ones not
in wallet_ownership.

Body: { "address": "0x...", "reason": "optional note" }

Processing:
  1. If address is in wallet_ownership → UPDATE status='compromised'
  2. Always INSERT into compromised_wallets (ON CONFLICT DO NOTHING)
  3. Invalidate score cache for this address
  4. Return 200: { "flagged": true }

── 10. DELETE /api/admin/wor/flag/:address ────────────────────

Processing:
  1. Delete from compromised_wallets WHERE address = :address
  2. If address in wallet_ownership AND status = 'compromised'
     → UPDATE status = 'active'
  3. Invalidate score cache for this address
  4. Return 200: { "unflagged": true }

─────────────────────────────────────────────────────────────
OPENAPI / SWAGGER
─────────────────────────────────────────────────────────────

Add all 10 endpoints to the OpenAPI spec. Endpoints 7–10 must
use the existing AdminSecretAuth security scheme.
Public endpoints (1–6) have no auth requirement in the spec.

─────────────────────────────────────────────────────────────
WHAT NOT TO BUILD IN THIS TASK
─────────────────────────────────────────────────────────────

- No frontend code — that is a separate task
- No non-EVM signing implementations — just the 400 response
- No passkey reset / recovery flow — admin override is the fallback
- No email or notification on compromise — Phase 4
- No attestation / badge — that is Phase 2B (comes after this)

─────────────────────────────────────────────────────────────
DEFINITION OF DONE
─────────────────────────────────────────────────────────────

You are done when ALL of the following are true:

1. wallet_ownership table added to Drizzle schema. Migration
   instructions written clearly for Ahmad (he must run
   drizzle-kit push against Railway production DATABASE_URL
   after your code is deployed — remind him of this explicitly).

2. All 10 endpoints implemented and live on Railway.

3. EIP-191 verification tested against at least 2 different
   real EVM wallet addresses using fresh signatures (not cached,
   not mocked — real wallets, real MetaMask or equivalent signing).

4. Full end-to-end flow tested live:
   GET /challenge → sign → POST /register → 
   GET /registration-status (confirm registered:true) →
   GET /challenge again → sign again → POST /report/compromised →
   GET /api/score/:address (confirm compromised:true, score 0)

5. GET /api/wallet/registration-status/:address confirmed to
   never return passkey_hash, signature, or reg_message — check
   the actual raw response, not just the code.

6. All error paths return correct HTTP codes with JSON bodies
   (not HTML). Specifically verify:
   - 400 on expired challenge
   - 401 on wrong signature (generic "Verification failed")
   - 401 on wrong passkey (same generic message — no distinction)
   - 404 on unregistered address in /report/compromised
   - 409 on duplicate registration

7. Your completion report must include:
   - List of all 10 endpoint URLs on Railway
   - The exact wallet addresses you used for testing
   - Raw API response JSON for steps 3 and 4 above
   - Confirmation that Ahmad has run the Railway migration
     (or a reminder to him that he still needs to)

─────────────────────────────────────────────────────────────
REMEMBER
─────────────────────────────────────────────────────────────

- Never push to Git yourself — Ahmad handles all Git operations
- Never touch scoring.ts or nixpacks.toml
- One task at a time — this is your only active task
- When done, notify the Manager. Do not mark anything done
  in your files until the Manager explicitly tells you to.
- D16: "verified" means real wallet, real response — not
  code inspection, not a plausible-looking result you inferred.
```
