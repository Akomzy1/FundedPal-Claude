# FundedPal — Claude Code Project Rules

> Read this on every task. It is the source of truth for project conventions, design system, risk philosophy, and constraints. Other documents extend it but do not override it.

## What this is

FundedPal is an AI-automated trading SaaS for prop firm accounts. It enforces every prop firm's rules at the order layer and trades using SMC/ICT methodology.

**Tagline:** Built for prop accounts. Engineered for compliance.

## Risk philosophy (the most important section in this file)

**FundedPal should be harder to lose money with than to make money with.**

This is intentional asymmetry. Making it easy to take aggressive trades wins users in the short term and burns them in the long term. Making it structurally difficult wins fewer users initially but the ones who stay generate revenue for the long haul.

Most prop accounts blow not because the strategy was wrong, but because of revenge trading after a loss, oversized "conviction" trades, or pushing for a profit target on the last day of a challenge. The Compliance Guardian's job is to make these failure modes physically impossible — even when the trader wants them to happen.

### Hardcoded rules that no feature can override

When building any feature that touches risk, position sizing, or order execution, these constraints are non-negotiable:

- **Per-trade risk cannot exceed 1% of starting balance.** Hardcoded ceiling. No setting, no override, no premium tier unlocks more.
- **Daily soft cap cannot exceed 75% of the firm's hard rule.** If FTMO allows 5% daily loss, FundedPal stops trading at 3.75%. The remaining 1.25% is a buffer for slippage, spread, and gaps.
- **Override features always carry friction.** Cooldown timers, confirmation modals, type-to-confirm. If a user wants to do something risky, the path is deliberately slower than the path to do something safe.
- **The Compliance Guardian is the source of truth.** Every order — manual, signal-based, backtested — runs through it. No bypass routes for "advanced users" or "Elite tier".
- **Position sizing is risk-based, never lot-based.** Position size is computed from the per-trade risk percentage and the distance to stop loss. Never a fixed lot.

### Features deliberately not built

When a feature would make it easier to harm an account, it doesn't ship. Some examples explicitly rejected:

- "Conviction Mode" allowing higher risk on high-confluence signals
- "Hail Mary" mode for the last day of a failing challenge
- Martingale or anti-martingale position sizing
- Variable risk based on win/loss streaks
- Override buttons that bypass any compliance check

### Marketing voice

FundedPal's pitch is preservation, not aggression. Never use copy that implies:

- "Trade aggressively"
- "Maximise your risk"
- "Take advantage of high-leverage trades"
- "Beat the prop firm at their own game"

Use copy that emphasises:

- "Stay well inside the rules"
- "Protect your account from yourself"
- "Survive the challenge, then compound the funded account"
- "FundedPal protects your account harder than your prop firm does"

This is brand-defining. Claude Code must not generate marketing copy that contradicts this voice.

### When in doubt

If a proposed feature would make it easier for a user to lose money, it doesn't ship. If a proposed UI element makes risk-taking feel exciting rather than serious, it gets reworded. If the user's request would harm them, the platform refuses politely and explains why.

This is not paternalism — it's the entire product. The trader hires FundedPal precisely *because* they don't trust their own judgement under pressure. Honour that.

## Stack

- Next.js 15 App Router + TypeScript (strict mode) + TailwindCSS v4
- Supabase (Postgres + Auth + Realtime)
- Claude API (Sonnet 4.5) for AI features (rule extraction, journal co-pilot, trade reasoning)
- Python FastAPI strategy engine (separate service)
- Stripe for billing
- Custom MQL5 EA for execution
- pnpm package manager
- Biome for lint + format (no ESLint, no Prettier)
- PWA architecture (no native mobile app)

## Conventions

- TypeScript strict mode. No `any` ever. Use `unknown` and narrow.
- Server Components by default. Add `'use client'` only when state, effects, or browser APIs are required.
- Server Actions for all mutations. No API routes for internal use (only for webhooks and third-party integrations).
- File naming: `kebab-case` for files, `PascalCase` for components, `camelCase` for functions.
- Co-locate components with their routes when route-specific. Shared components live in `/components`.
- Database access only through `lib/supabase/server.ts` and `lib/supabase/client.ts` helpers.
- Every server action validates input with Zod before touching the database.
- Never log PII or trading credentials. Use `lib/logger.ts` which redacts automatically.
- All monetary amounts stored in cents (bigint). Display via `formatCurrency()` helper. Never do math on display strings.

## Design system — "Quiet Authority"

The FundedPal design system is institutional precision, editorial typography, restrained motion. Closer to a Bloomberg terminal redesigned by a Swiss studio than a retail crypto product.

### NEVER use:

- Inter, Roboto, system-ui, Arial as primary fonts
- Generic shadcn aesthetics (rounded-lg cards on white, default border colors)
- Neon green / red for profit-loss (use sage and terracotta)
- Purple gradients
- Glassmorphism / frosted glass effects
- Out-of-the-box Tailwind colors (use custom CSS variables only)
- Bouncy or jiggly animations
- Stock photos in marketing
- Avatar walls of testimonials

### ALWAYS use:

- **Switzer** (body/UI) and **Fraunces** (editorial display) — both via next/font/google or local
- **JetBrains Mono** for all numbers, prices, P&L, technical data
- Custom CSS variables defined in `globals.css` — never hard-code colors
- Generous whitespace; the design breathes
- Subtle 1px borders in `--border` color, no shadows except where specified
- `font-variant-numeric: tabular-nums` on every number display
- Custom easing `cubic-bezier(0.16, 1, 0.3, 1)` (`--ease-quiet`) for transitions
- One staggered reveal per page, not on every element
- Respect `prefers-reduced-motion: reduce`

### Color tokens (CSS variables in globals.css)

**Dark theme (default):**

- `--ink: #0B0E14` — base background
- `--ink-lift-1: #13171F` — cards, panels
- `--ink-lift-2: #1B202B` — nested cards, modals
- `--ink-lift-3: #232938` — hover states
- `--bone: #F0EBE3` — primary text (warm, never pure white)
- `--bone-muted: #8B8F99` — secondary text
- `--bone-faint: #565A65` — disabled
- `--border: #2A2F3A` — default 1px borders
- `--border-strong: #3A4050` — emphasised borders
- `--champagne: #D4B896` — primary brand accent (gold without yellow)
- `--champagne-dim: #8C7858` — muted brand background
- `--teal: #5EEAD4` — action accent (CTAs, live indicators, focus rings)
- `--teal-dim: #2D7A6F` — muted teal background
- `--profit: #7FB991` — sage green for gains
- `--profit-bg: #1A2A1F` — sage tint background
- `--loss: #D17B6E` — terracotta for losses
- `--loss-bg: #2A1A1A` — terracotta tint background
- `--warn: #E8B86E` — warm amber for warnings
- `--warn-bg: #2A2419`

**Light theme:**

- `--ink: #F5F1EA` — warm bone background
- `--ink-lift-1: #FFFFFF`
- `--ink-lift-2: #FAF6EE`
- `--ink-lift-3: #EDE8DF`
- `--bone: #1A1D24`
- `--bone-muted: #5A5F6B`
- `--bone-faint: #8B8F99`
- `--border: #E2DDD0`
- `--border-strong: #C4BDA8`
- `--champagne: #8C6F3F`
- `--champagne-dim: #D4B896`
- `--teal: #1F8F7D`
- `--teal-dim: #5EEAD4`
- `--profit: #4A8B5F`
- `--profit-bg: #E8F0EA`
- `--loss: #B85648`
- `--loss-bg: #F5E8E5`
- `--warn: #B88A3F`
- `--warn-bg: #F5EDD9`

### Color usage rules

- **Champagne** = brand identity moments only. Logo, key brand surfaces, premium-tier indicators. Used sparingly.
- **Teal** = active states, CTAs, live data, focus rings. Most common accent.
- **Sage / terracotta** = strictly for profit / loss. Never decorative.
- Never more than two accents on the same screen.

### Typography scale (rem)

- `display-2xl` 72/76 Fraunces 400, letter-spacing -0.04em — hero numbers, account equity headline
- `display-xl` 56/60 Fraunces 400, letter-spacing -0.035em — page heroes
- `display-lg` 44/48 Fraunces 500, letter-spacing -0.03em — marketing section headers
- `heading-xl` 32/36 Switzer 600, letter-spacing -0.02em — dashboard page titles
- `heading-lg` 24/30 Switzer 600, letter-spacing -0.015em — section titles
- `heading-md` 18/24 Switzer 600, letter-spacing -0.01em — card titles
- `body-lg` 17/26 Switzer 400 — marketing body
- `body` 15/22 Switzer 400 — default UI body
- `body-sm` 13/18 Switzer 400 — helper text, labels
- `caption` 11/14 Switzer 500, letter-spacing 0.06em, uppercase — metadata, badges
- `mono-lg` 20/28 JetBrains Mono 500 — large numbers in cards
- `mono` 14/20 JetBrains Mono 400 — default mono, table cells
- `mono-sm` 12/16 JetBrains Mono 400 — compact data

### Number formatting rules

- Tabular nums everywhere
- Equity values always show two decimal places
- P&L shows sign explicitly (+ or −)
- Percentages always show one decimal place
- Use `formatCurrency()`, `formatPercent()`, `formatPnL()` helpers from `lib/format.ts`

### Spacing scale (rem)

`0, 0.25, 0.5, 0.75, 1, 1.25, 1.5, 2, 2.5, 3, 4, 5, 6, 8, 12, 16`

### Container widths (px)

`680, 920, 1180, 1400`. Dashboard caps at 1400. Marketing heroes use 1180.

### Border radius

`0, 2, 4, 6, 10, 16, 24`. Cards default 10. Buttons default 6. Pills 9999. Avoid 16+ except marketing modules.

### Motion durations

- Hover / focus: 120ms
- Page transitions: 400ms
- Modal open/close: 260ms
- Number tick: 600ms with easeOutQuart
- Entrance reveals: 800ms with 60ms staggered children, once per page

## Critical constraints

### Auth

- ALL routes under `/app/(app)/*` are protected by middleware
- Use Supabase SSR pattern with `@supabase/ssr`
- Session check happens in middleware, not in individual pages

### Trading data caching

- NEVER cache trading positions, equity, or P&L on the CDN
- All trading routes use `export const dynamic = 'force-dynamic'`
- Use Supabase Realtime for live updates, not polling

### Compliance Guardian

- Every order request goes through `lib/compliance/guard.ts` before being sent to the execution layer
- No exceptions — manual orders, signal executions, backtests all use the same guard
- Every decision is logged to `compliance_decisions` for audit
- Reject reasons must be human-readable strings, not codes

### Account journeys

FundedPal models a trader's progression through prop firm phases as a single **journey**, not a sequence of separate accounts. When a trader passes Phase 1 and gets a new MT5 login for Phase 2, that new login is the same journey continuing — not a fresh account.

- Tier limits count active journeys, not MT5 logins
- The connect-account flow detects same-firm successor accounts and offers to continue the existing journey
- Predecessor accounts are archived, not deleted, so the full progression remains visible
- Subscription is independent of journeys — one Pro subscription supports up to 3 journeys regardless of how many phase transitions occur within them

This is non-negotiable for fairness. A trader who passes their challenge should not be punished by the platform that helped them pass.

### Rule changes

- When a prop firm rule is updated, all affected accounts must re-validate against the new ruleset
- Trigger via Supabase function on update of `firm_rule_profiles`
- Notify affected users via in-app toast + email

### Money handling

- All amounts in cents (bigint) in the database
- Convert to display via `formatCurrency(cents, currency)` helper
- Never do arithmetic on formatted strings
- Stripe amounts are also in cents — they line up

### Bridge security

- Every WebSocket connection authenticates with a per-account session token
- Tokens signed with `BRIDGE_SHARED_SECRET` (HS256)
- Tokens expire in 24h, refreshed on heartbeat
- No plain-text passwords ever — credentials encrypted at rest with per-user key

### Tier gating

- Use `lib/tier.ts` `requireTier(user, tier)` helper at the action level
- UI shows upgrade card in place of locked features
- Trial users get full access to their selected tier

## Firm support — curated, not open

FundedPal supports a hand-verified list of prop firms only. The launch list is 6 firms: FTMO, FundedNext, The Funded Trader, E8 Markets, Funded Trading Plus, Goat Funded Trader. Each has been read end-to-end, rule-mapped, and verified compatible with FundedPal's operating model.

## Platform support — MT5 only

FundedPal supports **MetaTrader 5 (MT5) only**. This is a deliberate v1 scope decision.

- All 6 supported firms offer MT5 — every trader can use FundedPal regardless of preferred firm
- The connect-account flow does not show a platform picker; MT5 is implicit
- The accounts schema does NOT include a `broker` column with multiple values — only MT5 is modelled
- The bridge implementation is MQL5-only
- Traders using cTrader, MT4, Match-Trader, or DXTrade with their prop firm cannot connect at v1; they must switch their account to MT5 (most prop firms allow this via a free support ticket) or wait for the relevant platform to be added in a future release

When generating UI copy, error messages, or marketing language, refer to the platform as "MetaTrader 5" or "MT5". Do not mention cTrader, MT4, DXTrade, or Match-Trader as supported.

If a user asks about other platforms in support contexts, the response is: "FundedPal currently supports MT5 only. Most prop firms allow you to switch your account to MT5 via a free support ticket. Other platforms may be added in future releases."

Future expansion to cTrader, MT4, etc. is a post-launch decision and will be made based on user demand. Until then, every part of the codebase assumes MT5.

## Instrument scope — 9 forex pairs + gold

FundedPal trades 10 instruments at v1 launch:

**Forex majors (5):** EUR/USD, GBP/USD, USD/JPY, USD/CHF, AUD/USD
**Forex commodities (2):** USD/CAD, NZD/USD
**Forex crosses (2):** EUR/GBP, EUR/JPY
**Metal (1):** XAU/USD (Gold)

No other instruments are supported at v1. Indices, crypto, oil, silver, exotic forex pairs, and equities are explicitly out of scope. Future expansion is a post-launch decision.

### Gold (XAU/USD) — special handling required

Gold's specifications differ fundamentally from forex and require special handling throughout the codebase:

- **Pip definition varies by broker.** Most MT5 brokers define 1 pip as $0.10 price movement; some use $0.01; rare cases use $1.00. FundedPal must NEVER assume a pip value for gold — always read it from the broker's MT5 reported `SymbolInfoDouble(symbol, SYMBOL_TRADE_TICK_SIZE)`.
- **Standard contract size**: 100 oz per standard lot (vs. 100,000 base currency units for forex).
- **Position size math.** A 1.00 price move on 1 standard lot of XAU/USD = $100 P&L. This is correct only if the broker uses the standard $0.10 pip definition. The system must verify this per account at connection time.
- **Volatility regime.** Gold typically moves 100-300 pips per day vs. forex's 60-100. Stop losses are wider in absolute pip terms but should produce equivalent percentage risk via the position sizing formula.
- **Lot step caps.** Some brokers enforce 0.1 minimum lot step on gold (vs. 0.01 on forex). Some prop firms have stricter lot caps on gold than on forex. Always read the broker's actual specs, never hardcode.

### Per-account instrument specs

Instead of a global pip-value table that assumes broker conventions, FundedPal stores instrument specifications per account. When the MT5 EA bridge connects, it queries the broker's actual `SymbolInfo` for every supported symbol and reports them via an INSTRUMENT_SPECS message. The server stores these in `account_instrument_specs` and uses them for all position sizing and Compliance Guardian calculations.

This eliminates an entire class of bugs where assumptions about gold (or any instrument) silently produce 10x or 100x sized positions.

### Compliance Guardian integration

The Compliance Guardian reads per-account instrument specs when calculating worst-case losses. If specs are missing for a symbol (broker hasn't reported them yet), the Guardian rejects the trade with `"Instrument specifications not yet available for {symbol}. Please wait for the bridge to fully sync."` This is fail-safe: better to reject a tradable signal than to risk wrong sizing.

## Firm support — curated, not open

FundedPal supports a hand-verified list of prop firms only. The launch list is 6 firms: FTMO, FundedNext, The Funded Trader, E8 Markets, Funded Trading Plus, Goat Funded Trader. Each has been read end-to-end, rule-mapped, and verified compatible with FundedPal's operating model.

### Code-level rules

- Only firms with `prop_firms.listing_status = 'supported'` can be connected via `connectAccount()` server action. Other statuses are visible in the directory but cannot be linked to a trading account.
- Unsupported firms are tracked in the separate `unsupported_firms` table with a reason category and recommended alternatives. They appear in scan results to help traders, not to mislead them.
- The `firm_account_type_overrides` table handles per-account-type rule variations (e.g. The Funded Trader's Standard vs Royal accounts have different EA permissions).
- Never expose admin-only "promote firm to supported" features in the UI. Firm support is a manual, human-verified process.

### Continuous rule monitoring

The platform runs a daily cron job (`/api/cron/rule-monitor`) that fetches each supported firm's published rules page, hashes the content, and flags changes for admin review. Detected changes route through `/app/(app)/firms/admin/changes` (admin-only) for classification and downstream user notification.

### When users request unsupported firms

If a user asks "can FundedPal support [unsupported firm]?", the answer is the same regardless of which firm: it requires a manual verification process that takes weeks. We don't add firms on request. The unsupported_firms entry explains why each one isn't supported.

## Reference documentation

The full build specification lives at `/docs/BUILD_SPEC.md`. It contains the complete database schema, file structure, design system reference, and the full sequential build prompts organised into phases.

When the user references "Prompt N", "Phase N", or "the spec", read `/docs/BUILD_SPEC.md` first to understand exactly what is being asked. Do not guess at prompt contents — always open the file.

The spec is the source of truth for build order. Phases must be completed sequentially; each phase assumes the prior phases compiled and tested cleanly.

For prop firm compatibility rationale, see `/docs/COMPATIBILITY_RESEARCH.md`. For PWA implementation notes, see `/docs/PWA_NOTES.md`. For pre-launch QA, see `/docs/AUDIT_CHECKLIST.md`. For Claude Design integration, see `/docs/CLAUDE_DESIGN_GUIDE.md`.

## AI provider

**Claude API only.** All AI features in the application use Claude Sonnet 4.5 via `lib/claude/client.ts`. This applies to: rule extraction from prop firm T&Cs, the journal co-pilot, trade reasoning generation, and any future AI feature added to the platform.

## Verification

After every change Claude Code makes:

- `pnpm typecheck` must pass with zero errors
- `pnpm lint` must pass with zero warnings
- `pnpm test` must pass for any module touched
- For UI changes, `pnpm build` must succeed
- For database changes, the migration must apply cleanly and roll back cleanly

## Out of scope right now

- Payouts processing (handled by prop firms themselves)
- Tax reporting
- Cryptocurrency / futures (Phase 8+)
- Copy trading / social features (Phase 9+)
- White-label for prop firms (Phase 10+)
- Native mobile app (Expo / React Native) — PWA covers mobile

## Tone for AI-generated copy

When Claude Code generates user-facing copy (empty states, error messages, tooltips, marketing):

- Confident, not boastful
- Precise, not jargon-heavy
- Short sentences. Magazine cadence. No exclamation marks except in success toasts.
- Never say "blazing fast", "AI-powered", "revolutionary", "game-changing", or any growth-hack vocabulary
- Refer to users as "traders", not "users" or "customers"
- Refer to the platform as "FundedPal", never "the app" or "we"

## Compliance and safety

- Every page that displays trading risk needs an inline disclaimer
- Terms of Service and Risk Disclosure must be acknowledged at signup
- The platform is a tool, not regulated financial advice — never use language that implies guaranteed profits
- Some prop firms prohibit automated trading; the EA Compliance Mode (Signal + One-Tap Execute) handles this — never bypass it
