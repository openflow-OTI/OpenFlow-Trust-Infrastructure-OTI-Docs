# OTI — Frontend Builder Task List
> Last updated: July 9, 2026 (session 6 — New Frontend Builder account onboarded. First task: Task 11 remediation (revert api.otiscore.io → Railway URL, fix baseUrl, coordinate Vercel deploy of oti-docs/, add vercel.json rewrite for /docs/). Task 11D (icons + copy tone pass) queued behind it — one task at a time, do not start 11D until Task 11 is confirmed live.) | Maintained by: Development Manager
> **This file contains your tasks only. Read BUILDER_ONBOARDING.md and ARCHITECTURE.md before starting anything here.**
> Build in the exact order listed. Some tasks have hard dependencies — do not start them until the dependency is confirmed merged and deployed.

---

## ✅ Your Completed Tasks

### Task 1 — UI Polish ✅
- Homepage title, back button style, chain icon sizing, dead CSS removed, `isKnownChain()` guard in `useScore.ts`

### Task 2 — Logo Fix ✅
- `Logo.tsx` → `<img src="/logo.jpg">` replacing spiral SVG math
- `generateScoreCard.ts` → uses `loadImage()` before canvas draw

### Task 2B — Logo: SVG Replace ✅
- Original SVG logo file used directly as `public/logo.svg` — no reconstruction needed
- Crisp at 34px on Retina/high-DPI screens, zero blur
- `generateScoreCard.ts` already pointed to `/logo.svg`

### Task 7 — Signal Bars → Weighted Display ✅
- `pnpm codegen` run — `src/api/schema.gen.ts` regenerated from live backend OpenAPI spec
- `SignalBar.tsx` updated: bar fill = `(weighted / maxWeight) × 100%`, label = "weighted/maxWeight" e.g. "25/25"
- Color logic updated to use `weighted / maxWeight` ratio
- `Home.tsx` updated to pass full signal object to SignalBar
- `generateScoreCard.ts` updated — score card PNG shows weighted labels and correct fills
- Verified live against real wallet on Vercel — all 5 bars show weighted values
- Also resolved black screen crash caused by Task 5 API shape mismatch

---

### Task 7B — txCount Cap Indicator ✅
- `src/lib/formatMetadata.ts` — shows "1,000+ transactions" when txCount >= 1000
- `src/lib/generateScoreCard.ts` — same logic in score card PNG
- Verified live on Vercel by Manager

### Task 7C — Dynamic Rate Limit Display ✅
- `src/hooks/useAnonymousLimit.ts` calls public `GET /api/config/anonymous-limit`
- Homepage shows live daily_limit value from DB — confirmed showing "20 per day" (July 7, 2026)
- Falls back gracefully when API returns null

### Task 10 — Navbar API Health Status Dot ✅
- `src/components/Navbar.tsx` — green/red dot connected to `useHealth` hook
- Verified live on Vercel by Manager

### Task 9 — Admin Panel UI ✅
- `/admin` route with password gate, Dashboard, API Keys, Query History, Cache, Plan Configs tabs
- All API calls use `x-admin-secret` from sessionStorage
- Verified login working July 7, 2026

### Plan Configs Tab ✅
- New tab in admin panel — table of all 4 plans, inline edit per row
- PATCH sends `daily_limit` (snake_case) correctly
- Invalidates anonymous-limit query on save so homepage updates immediately

### Admin Panel Desktop Layout Fix ✅
- CSS `:has()` selector removes `max-width: 720px` when admin shell is present

### useAnonymousLimit Hook Fix ✅
- Hook now calls correct public endpoint instead of admin-protected route
- Fallback to 3 when null returned

### Homepage Scrollbar + Layout Fix ✅
- html/body/root use `min-height`; browser scrollbar at far right edge

### API Keys Tab — UI Resilience Fix ✅
- "+ New Key" button always visible even when keys list fails to load
- Error shows inline with Retry button; table guarded behind isSuccess
- Verified working July 7, 2026

### Fix: API Key Reveal on Creation ✅
- POST /api/admin/keys response field corrected: `data.apiKey` → `data.api_key`
- TypeScript interface updated to match: `apiKey: string` → `api_key: string`
- Modal now displays full key after creation with copy button and "never shown again" warning
- Verified live on Vercel by Manager

### Task 8 — Professional Results Page Redesign ✅
- Score panel in its own bordered card — ring gauge color matches chain brand color (all 15 chains)
- Score number color reflects trust tier; tier label shown: HIGHLY TRUSTED / TRUSTED / CAUTION / SUSPICIOUS / HIGH RISK
- Trust Signals in separate bordered card with "Trust Signals" heading; each signal shows label, metadata, fraction, colored bar
- Wallet address truncated (0xAb58...eC9B) with copy button; chain icon + name displayed
- Share button — native OS share sheet (navigator.share) with clipboard fallback
- Save as Image — 3× scale PNG (1920×2580px), chain-color ring, tier label, matches live UI exactly
- "⚑ Report this wallet" ghost link in mint — placeholder for WOR system
- Footer: "© 2026 OpenFlow Labs · openflowlabs.io"
- Full color theme upgrade — see OTI Color System below
- Verified live on Vercel by Manager (July 7, 2026)

---

## 🎨 OTI Color System (Locked — July 7, 2026)

> This palette replaced the old flat black + mint system. Every future task must use these exact values. Do not revert to pure black (#000000) or the old surface colors.

| Token | Value | Usage |
|---|---|---|
| Background | `#05080f` | Page background — deep blue-black |
| Surface | `#0b0f1a` | Card backgrounds, panels |
| Surface-2 | `#0f1520` | Inner elements, signal bar tracks |
| Borders | `#1c2535` | All card/panel borders |
| Body text | `#e8f4ff` | Main readable text — slight blue tint |
| Dimmed text | `#7a8fa8` | Metadata, labels, secondary text |
| Mint (primary) | `#00e5a0` | CTAs, highlights, active states, mint accents |
| Mint (gradient end) | `#3EFFC1` | Gradient highlights |

**Chain brand colors (ring gauge + panel tint):**
| Chain | Hex |
|---|---|
| Ethereum | `#627EEA` |
| Bitcoin | `#F7931A` |
| Solana | `#9945FF` |
| BNB | `#F3BA2F` |
| (all 15 chains have their exact brand hex in the codebase) |

**Special effects (do not remove):**
- Navbar: `backdrop-filter: blur(14px)` frosted glass
- Submit button: green glow on hover `box-shadow: 0 0 24px rgba(0,229,160,0.40)`
- Score panel: border + background tint shifts toward selected chain's brand color via `color-mix()`
- Signal bars: 5px height, soft color glow on fill
- All `color-mix()` declarations have plain-value fallbacks above them for older browsers

---

## 🔴 Your Task Queue — Build In This Exact Order

---

### TASK 8 — Professional Results Page Redesign
**Phase:** 3 — Redesign
**Priority:** HIGH — MVP requirement, Ahmad's explicit request
**Depends on:** Task 7 ✅ done AND Task 2B ✅ done — this task is now unblocked

**Why you are doing this:**
The results page is the first thing a potential enterprise partner, developer, or investor sees after they score a wallet. Right now it doesn't look professional enough to represent OTI as the infrastructure business it is. Ahmad has explicitly flagged this as an MVP requirement — the product must be presentable before any distribution channel launches or any business conversation happens. This redesign does not change any logic — only the visual presentation and layout.

**What to build:**
Redesign the scoring results page (`src/pages/Home.tsx` results section and related components). Keep: the black background, the mint green color scheme, the chain selector (do not touch it), the score gauge color system. Keep the "← Score Another Wallet" button.

Required changes:

1. **Wallet address:** Truncate the middle — show `0xAb58...eC9B` format (first 6 + last 4 chars). Add a copy-to-clipboard icon button next to it.

2. **Score tier label:** Add a label below the score gauge:
   - 85–100 → "HIGHLY TRUSTED" (mint green)
   - 65–84 → "TRUSTED" (light green)
   - 45–64 → "CAUTION" (amber)
   - 25–44 → "SUSPICIOUS" (orange)
   - 0–24 → "HIGH RISK" (red)

3. **Signal section:** Add a section header "Trust Signals" above the signal bars. Each signal bar now shows `weighted/maxWeight` as the score label (from Task 7). Add visual card/panel separation between the score gauge section and the signals section.

4. **Share button:** Make it larger and more prominent — mint green border, full width or centered with min-width, not a tiny button buried at the bottom.

5. **Footer:** Add a minimal footer: "© 2026 OpenFlow Labs · openflowlabs.io"

6. **Report this wallet:** Add a small ghost link below the signals section: "⚑ Report this wallet" — it should do nothing for now (disabled or href="#") but must be present for the WOR system to connect to later.

7. **Empty space:** The large void below the share button must be eliminated. Use the footer to anchor the bottom.

Keep all CSS in `src/index.css`. Do not add a new component library. Follow existing class naming conventions.

**Definition of done:** Results page looks professional on both mobile (375px) and desktop. Wallet address is truncated. Score tier label is visible. Signal bars show weighted values. Share button is prominent. Footer exists. "Report this wallet" link is present.

---

### TASK 8B — Professional Wallet Input Page Redesign ✅
**Completed:** July 8, 2026 (initial build + one polish round)

- Logo (56px), CSS hover rotation, gradient wordmark, tagline, mint-glow form card, styled rate-limit pill badge, "Try an example →" link, WOR ghost links, footer, faint watermark — all shipped
- Polish round: main logo increased 1.5× and repositioned higher; zkSync + Linea chain logos fixed (were invisible against dark background); chain icon size and dropdown width increased; vertical spacing tightened so the whole page fits without scrolling on mobile; "Report a compromised wallet" link converted to a mint-bordered card for visibility
- Verified live by Manager via screenshot — July 8, 2026

---

### TASK 8C — Fix Anonymous Rate Limit Cache Sync Bug ✅
**Completed:** July 8, 2026. Root cause was `setEditId(null)` in `onSuccess` racing with React 18 batching, unmounting the edit row before the success banner (and its `invalidateQueries` call) could ever be seen/trusted. Fixed by keeping the row mounted until the user clicks "Done". Verified live on Vercel by Ahmad: anonymous limit set to 42 → homepage badge updated without refresh; enterprise set to unlimited (blank field) → homepage correctly showed "Unlimited". Confirmed working in production.

<details><summary>Original spec (for reference)</summary>

**Phase:** 2 — Operational
**Priority:** CRITICAL — admin control is currently broken; Ahmad cannot reliably change the homepage's displayed rate limit
**Depends on:** Nothing — start immediately

**Why you are doing this:**
When Ahmad changes the anonymous plan's `daily_limit` in the Admin Panel (Plan Configs tab), the homepage's "X free lookups / day" badge does not reliably update. A full audit (by the previous Frontend Builder) found the root cause: the entire sync mechanism is a single fragile `setQueryData` call with no fallback, no invalidation, and multiple silent-failure paths. This must be replaced with a mechanism that actually works every time.

**Files you will touch:**
- `src/components/admin/PlanConfigs.tsx`
- `src/hooks/useAnonymousLimit.ts`
- `src/pages/Home.tsx`

**Confirmed bugs from the audit — fix all of them:**

1. **No fallback path.** `setQueryData(['anonymous-limit'], ...)` in `PlanConfigs.tsx`'s per-mutate `onSuccess` is the ONLY update mechanism. There is zero `invalidateQueries(['anonymous-limit'])` anywhere. Fix: call `qc.invalidateQueries({ queryKey: ['anonymous-limit'] })` in the mutation's `onSuccess` (or `onSettled`) so the Home page is guaranteed to refetch the real value from the server, regardless of what the per-mutate callback does.

2. **Silent string-match failure.** The `'anonymous'` plan-name check (`getPlanName(cfg).trim().toLowerCase() === 'anonymous'`) can silently evaluate to `false` if the backend field names don't match what's expected (`plan_name` vs `planName` vs `plan`). Fix: make `getPlanName` robust, and if the check ever fails to match a known plan, log a `console.warn` — don't fail silently.

3. **Closure value instead of server response.** The per-mutate `onSuccess` uses the form's local `daily_limit` value, not the actual value returned by the PATCH response. Fix: use the value from the mutation's response object, not the form closure.

4. **React Query v5 unmount issue.** Per-mutate callbacks are not guaranteed to fire if the component unmounts before the mutation resolves (e.g. admin navigates away right after clicking Save). Fix: move the cache invalidation into the mutation's global `onSuccess`/`onSettled` (defined in `useMutation(...)`, not passed at call-time to `.mutate()`), since global callbacks are not affected by unmounting.

5. **60-second stale dead zone.** `useAnonymousLimit.ts` uses `staleTime: 60_000`. Combined with the invalidation fix above, this is fine — invalidation forces a refetch regardless of staleTime. Just confirm the fix in #1 actually forces a refetch and doesn't get skipped due to staleTime.

6. **Ordering race between global and per-mutate `onSuccess`.** Avoid relying on both firing in a specific order. Consolidate the cache-update logic into one place (the global mutation callback) rather than splitting it across global + per-call callbacks.

7. **"Unlimited" (null) displayed as "3".** When admin sets a plan to unlimited (`daily_limit = null`), the frontend currently stores/display "3" — a display lie. Fix: when `daily_limit` is `null`, the Home page should show something like "Unlimited free lookups / day" (or omit the badge), not silently show 3. Keep the fallback-to-3 behavior only for the case where the API call itself fails (network error / anonymous plan not found) — not for a legitimate null value.

8. **Wasted/harmful fetch on results page.** `useAnonymousLimit()` in `Home.tsx` fires unconditionally even when the user is viewing a score result (not the landing form). Fix: only enable the query when the landing view is showing (e.g. `enabled: !hasQuery` in the hook options), so cache freshness isn't churned by irrelevant fetches.

9. **No observability.** Add a `console.warn` (or equivalent lightweight logging) when: the plan-name match fails, the mutation fails, or invalidation is triggered — enough that a future debugging session doesn't require React Query DevTools to diagnose this again.

**Definition of done:**
- Changing the anonymous plan's `daily_limit` in Admin Panel → Plan Configs updates the Home page badge within a few seconds, every time, with no manual refresh needed
- Setting the plan to unlimited (null) shows an accurate "Unlimited" state, not "3"
- Cache invalidation happens via `invalidateQueries`, not solely via `setQueryData`
- `useAnonymousLimit` no longer fires while viewing a score result
- Manually test: set limit to 5 → confirm Home shows 5. Set to null (unlimited) → confirm Home shows "Unlimited". Set back to a number → confirm Home updates again. Test navigating away from Admin immediately after clicking Save (before the PATCH resolves) to confirm the fix still works.
- Report back to Manager with test results for each scenario above before marking done

</details>

---

### TASK 8D — Homepage Visual Polish: Contrast, Animated CTA, Spacing & Density ✅
**Completed:** July 8, 2026. Fixed placeholder contrast, rebuilt the "Try an example" moving border from a paint-triggering animated `@property`-driven conic-gradient to a GPU-cheap transform-based rotation (no jank), corrected oversized/zoom sizing, added breathing room between sections, and established a clear typographic hierarchy. Verified live by Ahmad via screen recording — confirmed working normally and looking good.

<details><summary>Original spec (for reference)</summary>

**Phase:** 3 — Redesign
**Priority:** HIGH — Ahmad wants this 100% professional before moving on
**Depends on:** Task 8B ✅ and Task 8C ✅ (both done — this task polishes what's already live)

**Context — read this first:**
A previous Builder started this exact task and got partway through before hitting their account's credit limit. Ahmad pushed their in-progress work to Vercel, so it is now live in production — but it is incomplete and has introduced a new problem (janky animation). Do not assume a blank slate: pull the current `src/index.css` and `src/pages/Home.tsx` and see exactly what's there before making changes. Some of what's below may already be partially done — verify, don't guess.

**Files you will touch:**
- `src/index.css`
- `src/pages/Home.tsx` (only if the animation markup needs restructuring — no logic changes)

**Do NOT touch:**
- `src/lib/scoring.ts`, `nixpacks.toml`, `vercel.json`
- The chain selector logic
- Anything related to Task 8C's rate-limit fetching/caching logic (already fixed and verified — don't touch `useAnonymousLimit.ts` or the admin mutation logic)

**What to fix — in order:**

1. **Placeholder text contrast.**
   The wallet address input's placeholder ("0x… or wallet address") currently renders as dim grey and is hard to read against the dark card. Set the `::placeholder` color to white (or near-white, consistent with the rest of the input text) while keeping it visually distinct from actual typed text (e.g. via opacity, not a different color).

2. **"Try an example →" — visible text + moving border, WITHOUT lag.**
   The text itself must be white (currently too dim). Ahmad wants a thin animated mint-green line that traces/moves around the text — always in motion, eye-catching, but it currently causes real, visible lag/jank on mobile Chrome. This must be fixed, not removed — Ahmad confirmed he wants to keep the moving effect, he just wants it smooth.
   - Root cause is almost certainly that the current implementation animates an expensive property (`box-shadow`, `filter`, `backdrop-filter`, or a background repaint) every frame.
   - Rebuild using only GPU-cheap properties: animate `transform` (e.g. `rotate()`) on a small pseudo-element with a `conic-gradient` background, clipped to a thin ring via `mask`/`-webkit-mask`, positioned behind/around the text. This is the standard performant "moving border" technique — it only triggers compositing, not layout or paint, on every frame.
   - Do not animate `width`, `height`, `top`/`left`, `box-shadow`, or `filter` on every frame — these cause jank on mobile.
   - Respect `prefers-reduced-motion: reduce` — pause the animation for users who have that OS setting on.
   - Test specifically on a throttled/mobile-simulated profile in Chrome DevTools (4x CPU slowdown) and confirm no visible stutter before reporting done.
   - Color: same mint green used everywhere else (`--mint` / whatever the existing CSS variable is called) — do not introduce a new green shade.

3. **Zoom / oversized content — verify and finish.**
   A previous attempt scaled things down but the current live screenshot still looks too large/zoomed on mobile (375–414px width). Audit every size value on this page — logo dimensions, wordmark font-size, tagline font-size, input height/font-size, button height/font-size, badge font-size, icon sizes, paddings, margins — at the mobile breakpoint, and reduce consistently so the full page (header through footer) comfortably fits a standard mobile viewport (375×812) without feeling oversized and without introducing scroll. Don't do a partial pass — go element by element and confirm each one against the live screenshot.

4. **Spacing — give everything room to breathe.**
   The current live layout is visually cramped, especially the gap between the "42 free lookups / day" badge and the buttons/links below it (WOR ghost links). Increase vertical spacing (margin/gap) between every major block on the page: header → input card → rate-limit badge → WOR links → footer. The page must still fit on one screen without scrolling on mobile — this means the extra spacing has to come from the size reductions in step 3, not by making the page taller. Treat steps 3 and 4 as one job: shrink what's oversized, then redistribute the reclaimed space as breathing room, don't just compress everything to fit.

5. **Typography hierarchy — not everything should be the same size.**
   Some text is currently too small across the board. Establish a clear hierarchy:
   - **Primary (largest, most weight):** the "OTI" wordmark, the "Check trust score" button label
   - **Secondary (medium):** the tagline, the wallet address input text
   - **Tertiary (small, muted):** field labels ("WALLET ADDRESS", "CHAIN"), the rate-limit badge, footer text, WOR ghost links
   Increase primary/secondary sizes where they're currently too small to command attention, while keeping tertiary elements compact. This should feel intentional, not uniform.

**Constraints:**
- All CSS in `src/index.css`, no new component libraries or animation libraries
- Must work on both mobile (375px) and desktop
- Black background, mint green accent color system — no new colors
- No performance regressions — the animation fix in step 2 must be verified smooth on a throttled mobile profile

**Definition of done:**
- Placeholder text is clearly readable (white/near-white)
- "Try an example →" text is white with a smooth, continuously moving mint-green border effect — zero visible jank on a throttled mobile CPU profile
- Page fits a 375×812 mobile viewport without feeling zoomed in and without introducing scroll
- Clear, comfortable spacing between every section, especially below the rate-limit badge
- Clear typographic hierarchy — primary elements stand out, secondary/tertiary elements stay compact
- Screenshot the final result on a simulated mobile viewport (375px) and report back to Manager before marking done

</details>

---

### TASK 8E — Disable Mobile Pinch/Double-Tap Zoom Across the App ✅
**Completed:** July 8, 2026. Confirmed working smoothly by Ahmad. Shipped as the accessibility-conscious compromise: `maximum-scale=5, minimum-scale=1` (not `user-scalable=no`), plus a `touch-action: manipulation` CSS backstop for iOS double-tap-zoom. Deliberate pinch-zoom still available; accidental zoom and desktop behavior unaffected.

<details><summary>Original spec (for reference)</summary>

**Phase:** 3 — Redesign
**Priority:** MEDIUM — polish/UX consistency issue, not a functional bug
**Depends on:** Task 8D ✅ (done)

**Why you are doing this:**
Ahmad noticed that on mobile, users can pinch-zoom and double-tap-zoom the page — on the homepage, the scoring/results page, and the admin dashboard. Since every page is carefully sized and spaced for mobile already (per Task 8D), letting users zoom in/out breaks the intended layout and feels unpolished. This must be disabled on mobile only — desktop zoom behavior (Ctrl +/-, Ctrl+scroll) must be completely unaffected.

**Files you will likely touch:**
- `index.html` (the `<meta name="viewport">` tag — this is almost certainly the single source of truth for this, since it's one HTML file serving the whole SPA)
- Possibly `src/index.css` (a global `touch-action` rule to curb iOS Safari's double-tap-to-zoom specifically — pinch-zoom must remain available for accessibility, only double-tap is being curbed)

**Do NOT touch:**
- `src/lib/scoring.ts`, `nixpacks.toml`, `vercel.json`
- Anything already fixed in Task 8C/8D — this is purely a zoom/viewport-behavior task

**Accessibility note (Ahmad's decision, already made):** Fully disabling zoom (`user-scalable=no`, `maximum-scale=1`) is a known WCAG 1.4.4 (Resize Text) accessibility anti-pattern — it blocks low-vision users from zooming to read content. Ahmad has chosen the compromise below: still prevent the *accidental* pinch-zoom-while-scrolling annoyance, but allow deliberate zoom up to a reasonable limit for users who need it.

**What to build:**

1. Update the viewport meta tag to:
   `<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=5, minimum-scale=1">`
   Do NOT set `user-scalable=no` and do NOT set `maximum-scale=1` — both fully disable zoom, which Ahmad has decided against. `maximum-scale=5` still allows deliberate zoom for accessibility while preventing runaway pinch-zoom.

2. This must apply everywhere in the app — homepage, scoring/results view, and the admin dashboard (`/admin`) — since they're all part of the same single-page app served from one `index.html`, one correct change should cover all of them. Confirm this is actually true for this codebase (check whether admin has any separate HTML entry point) rather than assuming.

3. Double-tap-to-zoom on iOS Safari is a separate gesture from pinch-zoom and can still be curbed for the "accidental zoom" annoyance without affecting deliberate pinch-zoom accessibility. Add a CSS backstop: `touch-action: manipulation;` on `html, body` (or a more targeted selector if a global rule causes any side effects with existing interactions like the chain selector dropdown or admin table scrolling — test those specifically).

4. **Desktop must be completely unaffected.** Viewport meta tags and `touch-action` only affect touch/mobile zoom gestures — desktop zoom (Ctrl+/-, Ctrl+scroll, browser zoom controls) is a separate browser-level feature and is not affected by these changes. Just confirm this is actually true after your changes — test zooming in a desktop browser and confirm it still works normally.

**Definition of done:**
- On a mobile device or mobile emulation (touch simulation on), double-tap-to-zoom no longer works, and accidental/runaway pinch-zoom is curbed, on the homepage, the results/scoring view, and the admin dashboard
- Deliberate pinch-zoom still works up to 5x on mobile (accessibility requirement — do not fully disable it)
- On desktop, Ctrl+/-, Ctrl+scroll, and browser zoom controls still work exactly as before — completely unaffected
- No regressions to existing touch interactions (chain selector dropdown, admin table scrolling, any swipeable elements)
- Report back to Manager confirming you tested all three views on mobile emulation and confirmed desktop zoom is unaffected, before marking done

</details>

---

### TASK 11A — Restructure Vercel App: Marketing Front Door + Scoring at /score ✅
**Completed:** July 8, 2026. Marketing homepage live at `/` (Hero, chain row, How It Works, Trust Signals, Use Cases, Get the API, Find Us/Integrations, footer), scoring tool live unchanged at `/score`. Brand consistency confirmed — locked OTI color system, shared components, logo matches `/score` exactly. Verified live by Manager via fresh cache-busted screenshot and JS bundle inspection, and by Ahmad directly.

<details><summary>Original spec (for reference)</summary>

**Phase:** 4 — Pre-Distribution
**Priority:** HIGH
**Depends on:** Task 8 (results redesign done) AND Task 2B (SVG logo done)

**Why you are doing this:**
Right now, anyone who visits `otiscore.vercel.app` lands directly on the scoring tool with no context about what OTI is, why it matters, or who built it. Every bot reply, every widget badge, every browser extension popup will link back to this URL. Without a proper front door, that traffic converts to nothing — developers and partners arrive, see a form, and leave. The marketing homepage gives OTI's story, explains the product, and converts curious visitors into API users and potential business partners. The scoring tool moves to `/score` — nothing about it changes visually. One Vercel project, one URL, two purposes.

**Part A — Move the scoring tool to `/score`**

Move the current homepage (wallet address form + results) to the `/score` route. Update `vercel.json` carefully — the SPA rewrite rule `{"source":"/(.*)", "destination":"/index.html"}` must stay intact; do not remove or break it. Update any internal links that currently point to `/`.

**Part B — Build the marketing homepage at `/`**

**⚠️ Brand consistency is non-negotiable.** This must look and feel like the exact same product as `/score` and the admin dashboard — a visitor moving between them should never feel like they landed on a different site. Before writing any code:
1. Open the live `/score` and results pages and note the real, current values — border radius, card padding, shadow/glow treatments, button hover/active states, font weights, spacing rhythm — don't guess.
2. Reuse existing shared components/CSS classes/tokens from the scoring app wherever possible (buttons, card containers, badges, chain icons) instead of rebuilding equivalents.
3. Any new component must still derive from the same tokens below — never introduce a one-off color, font, radius, or shadow value.

Brand system — use the locked OTI color system exactly (see Color System section above), zero deviation:
- Background: `#05080f` · Surface: `#0b0f1a` · Surface-2: `#0f1520` · Borders: `#1c2535`
- Mint primary: `#00e5a0` · Mint gradient: `#3EFFC1`
- Body text: `#e8f4ff` · Dimmed text: `#7a8fa8`
- Chain brand colors: use the same per-chain hex values already defined in the scoring app (all 15) — do not invent new ones for the homepage's chain row
- Typography: same font family already in use across the scoring app (Geist Sans or Inter) — match weights and sizing scale, not just the family name
- Same special effects already established: navbar frosted glass (`backdrop-filter: blur(14px)`), mint glow on primary buttons, `color-mix()` tint techniques where relevant — with plain-value fallbacks
- Micro-interaction: the spiral logo rotates subtly on hover (CSS `transform: rotate()`, 2–3s ease, infinite)
- All CTAs, highlights, active states: mint only
- Do NOT revert to pure black `#000000` anywhere

**Logo:** Use `logo.svg` (the current, non-blurry logo already live on the scoring app) at the same aspect ratio and visual treatment as it appears in the scoring app's navbar — same crispness, same proportions, same hover behavior if any exists there.

**Overall bar:** this must read as a fully polished, professional product website, not a placeholder or template — same level of care as Task 8/8B/8D. Screenshot both `/` and `/score` side by side before calling this done and confirm they feel like one coherent product.

Sections to build:

**1. Navbar**
- Left: OTI spiral logo (34px) + "OTI" wordmark in white
- Right: "Score a Wallet" button (mint outline) → `/score`; "API Docs" text link → placeholder `#`
- Mobile: hamburger with same links

**2. Hero**
- Headline: "Know Who You're Transacting With"
- Sub-headline: "OTI scores any blockchain wallet 0–100 using five on-chain trust signals. Instant. Free. API-ready."
- CTAs: "Try It Free" (mint filled → `/score`), "View API Docs" (ghost → `#`)
- Faint watermark spiral behind headline (CSS only, no JS library)
- Chain row below CTAs: small chain icons for ETH, SOL, BTC, TON, MATIC, ARB, SUI, TRX + "+7 more" badge

**3. How It Works (3 steps)**
- Step 1: Enter wallet address + select chain
- Step 2: OTI analyzes 5 on-chain behavioral signals
- Step 3: Get a 0–100 trust score with full signal breakdown
- Style: numbered cards, horizontal row on desktop, vertical stack on mobile

**4. Trust Signals (5 cards)**
- 🕐 Wallet Age (25%) — "How long this wallet has been active on-chain"
- 📊 Transaction Count (20%) — "Volume and frequency of transactions"
- 🪙 Token Holding Behavior (20%) — "Diversity and quality of tokens held"
- 🔗 Smart Contract Interactions (20%) — "Depth of DeFi and protocol engagement"
- ⏱ Transaction Timing (15%) — "Consistency and naturalness of activity patterns"

**5. Use Cases (tile grid — 3 cols desktop, 2 tablet, 1 mobile)**

| Tile | Sector | Body |
|---|---|---|
| 💱 | Exchanges & Gateways | Flag compromised wallets before processing withdrawals. |
| 🏦 | DeFi Protocols | Risk-adjust lending and collateral requirements based on wallet trust. |
| 🖼 | NFT Marketplaces | Display trust badges next to seller listings. |
| 💸 | Payment Processors | Require a minimum trust score before processing. |
| 🎮 | Web3 Gaming | Prevent fresh-wallet farming in Play-to-Earn. |
| 🗳 | DAO Governance | Weight votes by wallet trust alongside token balance. |
| 🔐 | Custody Services | Score source wallets before crediting accounts. |
| 📡 | Bridges & Cross-chain | Score the source wallet before allowing a bridge transaction. |
| 🛠 | Developer Tooling | One API call. Any chain. No blockchain infrastructure required. |

**6. Get the API**
- Headline: "Free API Key. No Credit Card."
- Sub-line: "Anonymous: 3 lookups/day. Register free for higher limits."
- cURL example block (copy-pasteable)
- CTA: "Read the Docs" → `#` placeholder

**7. Find Us / Integrations**
Row of icons with labels: Telegram, Discord, Chrome Extension, Firefox Extension — all `#` placeholder links

**8. Footer**
- Left: OTI spiral logo (small) + "© 2026 OpenFlow Labs"
- Center: Score a Wallet · API Docs · GitHub (public frontend repo link) · Privacy Policy (placeholder) · Terms (placeholder)
- Right: Social icons — Twitter/X, LinkedIn, Telegram, Discord (all `#`)

**Chat support — Crisp.chat:**
Add to `index.html` `<head>`:
```html
<script>window.$crisp=[];window.CRISP_WEBSITE_ID="AHMAD_PROVIDES_THIS";</script>
<script src="https://client.crisp.chat/l.js" async></script>
```
If Ahmad has not yet provided the ID, leave `CRISP_WEBSITE_ID=""` — the widget simply won't appear until filled.

**Feedback form — Tally.so:**
Ahmad creates a form at tally.so and provides the embed snippet. Place it in a modal triggered by "Send Feedback" in the footer. If not yet provided, use `mailto:` fallback.

**Definition of done:**
- `/` shows the full marketing homepage — all 8 sections present and styled
- `/score` shows the scoring tool exactly as before (no visual changes to the tool itself)
- Navbar "Score a Wallet" navigates correctly between the two views
- Crisp script is in `<head>` (even if ID is empty placeholder)
- Responsive on mobile (375px) and desktop (1440px)
- Brand system uses the locked OTI color system throughout, with zero one-off/invented colors, fonts, or component styles anywhere on the page
- Logo renders identically (crispness, proportions) to how it appears on `/score`
- Side-by-side screenshot of `/` and `/score` submitted to Manager showing visual consistency before marking done

---

### TASK 11B — Whitepaper Page
**Phase:** 4 — Pre-Distribution
**Priority:** HIGH
**Depends on:** Task 11A must be merged first
**Status: 🟡 LIVE BUT NOT DONE** — built and deployed at `/whitepaper`. Manager verified live (July 8, 2026, session 4) and found 3 issues Ahmad wants fixed before this is marked done:
1. All body/paragraph text must be white (`#e8f4ff`) — currently rendering in the dimmed grey token (`#7a8fa8`). Keep section numbers mint (`#00e5a0`).
2. Page can be scrolled/shifted horizontally on mobile — eliminate all horizontal overflow so everything fits one column at 375px.
3. Remove the "Roadmap" section entirely (no public dates being promised) — renumber every following section and the TOC sequentially.
Reply to Manager when the fixed version is live.

**Why you are doing this:**
OTI is positioning itself as infrastructure for enterprises — exchanges, custody platforms, DeFi protocols. Those buyers do not sign contracts based on a landing page alone. They need a technical document that explains what OTI is, how it works, who built it, and what the business model is. A whitepaper also builds credibility with investors and serious developers. The `/whitepaper` route lives inside the same Vercel project — no new deployment, no new domain needed.

**Route:** `/whitepaper`

**Design rules:**
- Use the locked OTI color system (Background `#05080f`, Surface `#0b0f1a`, Mint `#00e5a0`/`#3EFFC1` — see Color System section above) — must match the homepage (Task 11A) and scoring app exactly, same tokens, no deviation
- Reuse the shared navbar/footer components from Task 11A rather than rebuilding them — this page must feel like a continuation of the same site
- Typography: same font family as the rest of the site (Geist Sans or Inter). Body text 17–18px, line-height 1.8 — this is a long-form document
- Section numbers displayed in mint: `01`, `02`, `03`
- Sticky table of contents sidebar on desktop (collapses to "Jump to section" accordion on mobile)
- Faint spiral watermark behind page header — `opacity: 0.04`, CSS only
- "Download as PDF" button — uses `window.print()` with `@media print` stylesheet that hides navbar, sidebar, and footer

**Nav update required:**
- Desktop: `Logo | Score a Wallet | API Docs | Whitepaper | [Social icons]`
- Mobile hamburger: Score a Wallet → API Docs → Whitepaper

**Page header:**
- Title: `OpenFlow Trust Infrastructure — Whitepaper`
- Subtitle: `Version 1.0 · July 2026 · OpenFlow Labs`
- Two buttons: `[Download PDF]` and `[Score a Wallet →]`

**Content sections** (copy exactly as provided in TASKS.md — sections 01 through the end of the whitepaper content):
- 01 — Executive Summary
- 02 — About OpenFlow
- 03 — About OpenFlow Labs
- 04 — The Problem
- 05 — How OTI Works
- 06 — Supported Infrastructure
- 07 — Use Cases
- (remaining sections per TASKS.md)

**Definition of done:** `/whitepaper` renders all sections. Sticky sidebar works on desktop. Print-to-PDF hides nav and sidebar. "Score a Wallet" button navigates to `/score`.

---

### TASK 11 — Developer Docs Site
**Phase:** 4 — Pre-Distribution
**Priority:** HIGH — hard dependency before any bot or widget launches
**Status: 🟡 CODE COMPLETE — DEPLOYMENT + REMEDIATION PENDING**

**Context for the new account inheriting this task:**
Ahmad gave the previous Frontend Builder open-ended go-ahead on this task, so the scope grew well beyond the original spec across 8 self-directed rounds. Below is the FULL list of what was actually built, so you know exactly what exists and what's left broken. Read this whole section before touching anything.

**✅ What is already built and working (do not redo):**
1. Docusaurus site at `oti-docs/` in the frontend repo, with all 6 required sections, API values live-verified against production
2. An OG social-share image for the docs site
3. A "privacy audit" pass that removed internal-only doc folders and scrubbed leaked internal references from the public Whitepaper — this was a genuine security fix, keep it
4. A "Try It Live" interactive widget embedded in the docs that calls the real API

**🔴 What is broken right now and why:**
1. `otiscore.vercel.app/docs/` returns a 404 in production. Root cause: the docs were only ever proxied via Vite in the local Replit dev environment. On Vercel, the main site's `vercel.json` SPA catch-all (`{"source":"/(.*)", "destination":"/index.html"}`) intercepts `/docs/*` first and serves the React app instead, because the docs were never deployed as their own Vercel project.
2. "Try It Live" doesn't work in production. Root cause: during this scope creep, all API URLs across the codebase (MarketingNavbar.tsx, Landing.tsx, Whitepaper.tsx, oti-docs/docusaurus.config.js, and the Try It Live widget itself) were migrated from the working Railway backend URL to `https://api.otiscore.io` — a custom domain Ahmad has not set up. Manager confirmed by DNS check that `api.otiscore.io` does not currently resolve. Every API call in the docs and the widget is failing against a domain that doesn't exist yet.

**Ahmad's decision (July 9, 2026): do NOT wait on a custom domain purchase/DNS.** Instead, both the docs path and the API path get proxied through Vercel rewrites so everything stays under the single `otiscore.vercel.app` URL. This is free, requires no domain purchase, and if Ahmad buys a real domain later, only the Vercel project's domain settings need to change — no code changes.

**Your four jobs to complete this task:**

**Job 1 — Fix the baseUrl (do this first):**
Open `oti-docs/docusaurus.config.js`. Change `baseUrl: '/docs/'` to `baseUrl: '/'`. The `/docs/` base path was only correct for the local Replit proxy environment; on its own Vercel project the docs sit at the root.

**Job 2 — Revert the API domain migration:**
Search the entire repo for `api.otiscore.io` and replace every occurrence with the working Railway backend URL (the one in use before this migration). This immediately fixes "Try It Live" without needing any DNS work. Do this in code — do not touch `vercel.json` for this step, it's a plain string revert.

**Job 3 — Ahmad deploys `oti-docs/` as its own Vercel project (his action, not yours) — BLOCKED, needs your help:**
Ahmad set Root Directory to `oti-docs`, Vercel correctly auto-detected Docusaurus (v2+), but the build fails every time (including on retry, same commit) with:
```
npm error Exit handler never called! This is an error with npm itself.
```
This is a real, reproducible failure, not a transient Vercel hiccup (confirmed by identical failure on redeploy). It happens during `npm install` inside `oti-docs/`. Most likely cause: `oti-docs/package-lock.json` is out of sync with `oti-docs/package.json` (a dependency — e.g. the Try It Live widget — was added without regenerating the lockfile), or a `packageManager`/`engines` mismatch between the repo root and `oti-docs/`.

**Please do this before Ahmad tries deploying again:**
1. `cd oti-docs`, delete `package-lock.json` and `node_modules`, run `npm install` fresh, confirm it installs cleanly and `npm run build` succeeds locally
2. Commit the regenerated `package-lock.json`
3. Check `oti-docs/package.json` for an `engines` field that might conflict with the repo root's — Vercel logged a warning about `"engines": {"node": ">=20.0"}` before the error; confirm this isn't pointing at an unavailable Node version on Vercel's build image
4. If it still fails after a clean lockfile, try running `npm install` locally with the same Node major version Vercel uses (check Vercel project settings → Node.js Version) to reproduce the exact error
5. Tell Ahmad once you've confirmed a fix and pushed — he'll retry the Vercel import

**Job 4 — Add a `vercel.json` rewrite so `/docs/` keeps working under the main domain:**
This is the one approved exception to "never touch vercel.json" — Manager has pre-cleared it. Once Ahmad gives you the docs deployment URL, add a rewrite rule to the main site's `vercel.json`, placed BEFORE the existing SPA catch-all rule (order matters — first match wins):
```json
{"source": "/docs/:path*", "destination": "https://<oti-docs-deployment-url>/:path*"}
```
Do not remove or reorder the existing SPA catch-all rule below it. Test that `otiscore.vercel.app/docs/` loads the real docs site (not a 404, not the React app) after this change.

**Definition of done:**
- `baseUrl: '/'` set in `oti-docs/docusaurus.config.js`
- All `api.otiscore.io` references reverted to the Railway backend URL — "Try It Live" works again
- `oti-docs/` deployed as its own Vercel project (Ahmad's action)
- `vercel.json` rewrite added (docs rule before the SPA catch-all) — `otiscore.vercel.app/docs/` loads correctly, no 404
- Getting Started page works — a developer can read it and make their first successful API call within 5 minutes
- Report back to the Manager with a live, cache-busted screenshot of `/docs/` before marking done

---

### TASK 11D — Replace emoji icons with a real icon set + copy tone pass
**Phase:** 4 — Pre-Distribution
**Priority:** MEDIUM — do after Task 11 deployment fixes, before wider distribution
**Depends on:** none, can run independently of Task 11's deployment work

**Why you are doing this:**
Ahmad's read (and Manager agrees after review): the Trust Signals and Use Cases sections on the homepage use raw emoji as icons (🕐📊🪙🔗⏱ and 💱🏦🖼🎮🗳🔐📡🛠). This was actually in the Manager's original Task 11A spec, not a builder invention — but for a product selling into exchanges, custody platforms, and DeFi protocols, emoji icons read as consumer/hobby-project rather than enterprise infrastructure. Manager reviewed the live homepage and whitepaper Executive Summary directly and found the actual prose (headline, sub-headline, How It Works copy, whitepaper Section 01) reads clean and professional — the AI/emoji impression is concentrated in these two icon-grid sections specifically, not systemic across all copy.

**What to build:**
1. Replace all emoji icons in the Trust Signals (5-card) and Use Cases (9-tile) sections on the homepage with a consistent icon set (Lucide or Heroicons — pick one, use it everywhere) styled in mint (`#00e5a0`) on the locked dark theme. Do not introduce a new icon library if one is already installed in the repo — check first.
2. Do a full read-through of the Whitepaper (`/whitepaper`, all 13 sections) and the Docusaurus docs site copy specifically looking for generic AI-pattern phrasing (overly hedged corporate language, repetitive sentence structures, buzzword stacking without specifics). Flag anything found to the Manager with the exact sentence and section before rewriting — Manager will confirm which to change.
3. Do NOT touch the homepage hero, "How It Works," or whitepaper Executive Summary copy — Manager has already reviewed these live and confirmed they read fine as-is.

**Definition of done:**
- No raw emoji characters remain as icons anywhere on the marketing site, whitepaper, or docs
- Icon set is consistent and uses the locked mint color
- List of flagged AI-sounding sentences (if any) sent to the Manager before any docs/whitepaper copy is rewritten
- Screenshot of updated Trust Signals and Use Cases sections submitted to Manager

---

## ⏳ Future Tasks (Not Yet Active — Manager Will Assign When Ready)

These are coming after the queue above is complete. You do not need to read these in detail now.

- **Phase 5:** Browser extension (Firefox first, then Chrome) — separate repo, content script that detects wallet addresses on Etherscan/OpenSea and injects OTI score badges
- **Phase 6:** Wallet Ownership Registry (WOR) UI — registration flow, compromise report flow, passkey integration
- **Phase 7:** Self-serve developer portal — sign up, get API key, choose plan

---

## Keeping This File Updated

You never update either file on your own initiative. Every update — whether marking a task done or adding a new one — happens only when the Manager explicitly tells you to.

**When you complete a task:**
1. Notify the Manager — do not mark anything done yourself
2. Wait for the Manager to review and confirm your work
3. Only when the Manager explicitly tells you to mark it done:
   - Move it to the ✅ Completed Tasks section in this file
   - Mark it ✅ in `TASKS.md` as well
4. Both files must always match

**When the Manager assigns you a new task:**
1. The Manager will explicitly tell you to add the new task to your file
2. Only then: add it to the bottom of your Queue section in this file AND add it to `TASKS.md`
3. Read it fully before starting — every task has a "Why" and a "Definition of done"
4. Do not start a task that has a ⚠️ dependency warning until the Manager confirms it is cleared

**General rules:**
- Never touch Git, never push, never open a PR — Ahmad handles all of that himself
- Never touch `vercel.json` — ever
- Never manually edit `src/api/schema.gen.ts` — run `pnpm codegen` instead
- Ahmad loves the chain selector — do not touch it
- All CSS goes in `src/index.css` — no new component libraries
- Test on mobile (375px) — most users are on mobile
- If a task is blocked, tell the Manager immediately — do not sit on a blocker
- Never update any file without the Manager's explicit instruction
