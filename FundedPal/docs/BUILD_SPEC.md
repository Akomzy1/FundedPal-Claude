# FundedPal — Build Specification (Integrated)

> **Tagline:** Built for prop accounts. Engineered for compliance.

A complete build specification for FundedPal — an AI-driven SMC/ICT trading platform purpose-built for prop firm accounts, with auto-detected rule enforcement and prop-firm-compliant execution modes. Optimised for build with Claude Code.

This document integrates the original 35-prompt sequence plus three formerly-separate amendments (Risk Budget Layer, Account Journeys, Firm Curation & Rule Monitoring) into a single linear build path. Total: 38 prompts across 8 phases.

---

## How to use this document

This spec is the source of truth for the FundedPal build. Hand sections to Claude Code sequentially.

The flow:

1. CLAUDE.md content lives in the repo root from day one — Claude Code reads it on every task
2. This file lives at `/docs/BUILD_SPEC.md`
3. Run prompts in order through Phases 0 → 7. Don't skip phases; each one assumes the prior phase compiled and tested cleanly
4. After each prompt, run the verification commands listed and only proceed when green
5. Reference the design system in CLAUDE.md on every UI prompt — "follow the FundedPal design system in CLAUDE.md" is the standing instruction
6. For Claude Design integration, see `/docs/CLAUDE_DESIGN_GUIDE.md` — design surfaces are prototyped in Claude Design first, then exported as handoff bundles to inform Claude Code implementation

---

## 1. Project Overview

**Product:** FundedPal — AI-automated trading platform for prop firm accounts.

**Core promise:** Connect any supported prop firm account, the platform auto-detects every rule that account operates under (daily loss, max drawdown, consistency, news, EA permissions, weekend holding, lot caps), and runs an SMC/ICT strategy engine with a hard compliance layer that physically cannot place a rule-breaching trade.

**Differentiators (the moat):**

1. **Compliance Guardian** — hard-enforced rule engine at the order-routing layer
2. **Risk Budget Layer** — soft caps that keep accounts well clear of firm hard rules
3. **Account Journeys** — tracks a trader's progression through phases as a single journey, not separate accounts
4. **EA Compliance Mode** — Signal + One-Tap Execute for prop firms that ban automation
5. **Phase-Aware Strategy** — different aggression curves for Phase 1, Phase 2, and Funded
6. **AI Rule Detection** — Claude extracts rules from any prop firm's T&Cs
7. **Curated firm support** — 6 hand-verified firms, continuously monitored for rule changes
8. **Multi-Account Risk Allocator** — coordinates risk across simultaneous challenges

**Target user:** Retail prop trader running 1-5 challenges with FTMO, FundedNext, The Funded Trader, E8 Markets, Funded Trading Plus, or Goat Funded Trader.

**Pricing:**

- **Trader $39/mo** — 14-day trial, 1 journey, full Compliance Guardian, SMC/ICT engine
- **Pro $69/mo** — 14-day trial, 3 journeys, Multi-Account Risk Allocator, Drawdown Forecaster, journal co-pilot, advanced backtest
- **Elite $199/mo** — 14-day trial, unlimited journeys, API access, custom strategy plug-ins, white-glove onboarding

---

## 2. Tech Stack

```
Frontend       Next.js 15 (App Router) · TypeScript · TailwindCSS v4 · Framer Motion
UI Components  Custom-built. shadcn/ui as foundation only — every component restyled to FundedPal design system
Charts         TradingView Lightweight Charts (free, MIT) for in-app charts
Auth           Supabase Auth
Database       Supabase Postgres + Row Level Security
Realtime       Supabase Realtime for live position / equity updates
AI             Claude API (Sonnet 4.5)
Strategy       Python FastAPI service (separate worker)
Execution      Custom MT5 EA bridge (MQL5) over WebSocket — MT5 only at v1
Payments       Stripe Subscriptions + 14-day trial
PWA            @serwist/next for service worker management
Hosting        Vercel + Railway/Fly.io + Supabase
Monitoring     Sentry + PostHog + Logsnag
```

**Hard requirements:** Node 20+. Postgres 15+. Python 3.11+. MetaTrader 5 build 3815+ for the EA bridge.

**Platform scope:** v1 supports MT5 exclusively. cTrader, MT4, Match-Trader, and DXTrade are out of scope and will not be referenced in code, UI, or marketing.

---

## 3. Database Schema

The complete schema is built incrementally across multiple prompts. The full final state includes:

- `profiles` — extends Supabase auth.users with tier, trial state, admin flag
- `prop_firms` — supported firm catalogue with compatibility metadata
- `unsupported_firms` — visible-but-not-connectable firms with reason categories
- `firm_rule_profiles` — per-firm, per-account-size, per-phase rule sets
- `firm_account_type_overrides` — per-account-type rule variations
- `firm_rule_snapshots` — rule monitor history
- `firm_reviews` — community ratings
- `accounts` — connected MT5 accounts (with journey tracking)
- `account_phase_transitions` — journey progression history
- `account_snapshots` — daily account state
- `risk_setting_changes` — audit trail for risk settings
- `positions` — trading positions with SMC reasoning
- `compliance_decisions` — every Compliance Guardian decision
- `signals` — generated SMC/ICT signals
- `journal_entries` — auto + manual + AI reflections
- `backtests` — backtest configurations and results
- `push_subscriptions` — Web Push subscriptions

Schemas are defined inline in their respective prompts. See Prompts 4 (initial schema), 8 (rule profiles), 11 (accounts), 15.5 (risk budget), 22.5 (journeys), 29.5 (firm curation + monitoring), and others.

---

## 4. File Structure

```
fundedpal/
├── CLAUDE.md                            # Project rules (read every task)
├── README.md
├── package.json
├── pnpm-lock.yaml
├── tsconfig.json
├── next.config.ts
├── tailwind.config.ts
├── biome.json
├── .env.example
├── /app
│   ├── layout.tsx
│   ├── globals.css
│   ├── (marketing)/
│   ├── (auth)/
│   ├── (app)/
│   └── api/
├── /components
│   ├── ui/                              # custom-styled primitives
│   ├── branded/                         # FundedPal-specific components
│   ├── icons/                           # custom SVG
│   ├── charts/
│   └── compliance/
├── /lib
│   ├── supabase/
│   ├── compliance/
│   ├── claude/
│   ├── stripe/
│   ├── bridge/
│   ├── strategy/
│   ├── monitor/                         # rule monitor
│   ├── journey/                         # account journey logic
│   ├── format.ts
│   ├── logger.ts
│   └── utils.ts
├── /actions
├── /supabase
│   └── migrations/
├── /strategy-engine                     # Python FastAPI service
├── /ea-bridge                           # MQL5 EA source
├── /docs
│   ├── BUILD_SPEC.md                    # this file
│   ├── COMPATIBILITY_RESEARCH.md
│   ├── CLAUDE_DESIGN_GUIDE.md
│   ├── PWA_NOTES.md
│   └── AUDIT_CHECKLIST.md
└── /tests
```

---

## 5. Build Prompts

38 prompts across 8 phases. Run sequentially. Each prompt is self-contained and verifiable.

### Phase 0 — Foundation

**Prompt 1 — Initialise repo**

> Initialise a Next.js 15 App Router project with TypeScript strict mode, Tailwind v4, and pnpm. Add Biome for linting and formatting (replace ESLint and Prettier). Configure tsconfig with strict, noUncheckedIndexedAccess, isolatedModules enabled. Create the file structure from Section 4 of CLAUDE.md and BUILD_SPEC.md — empty placeholder files where appropriate. Add a comprehensive `.gitignore`. Set up package.json scripts: dev, build, start, typecheck, lint, format, test. Verify: `pnpm typecheck`, `pnpm lint`, `pnpm build` all pass on the empty project.

**Prompt 2 — PWA foundation**

> Configure FundedPal as a Progressive Web App. (1) Install `@serwist/next` and configure runtime caching strategies — NetworkOnly for `/api/*` and `/actions/*`, NetworkFirst with 3s timeout for `/app/*`, CacheFirst for static assets and fonts, StaleWhileRevalidate for marketing pages. (2) Create `/public/manifest.webmanifest` with name "FundedPal", short_name "FundedPal", description "Built for prop accounts. Engineered for compliance.", theme_color "#0B0E14", background_color "#0B0E14", display "standalone", orientation "any", start_url "/dashboard". Include icon entries at 192px, 512px, 512px maskable, plus Apple touch icon at 180px. (3) Generate the icon set — champagne F monogram in Fraunces on `--ink` background. (4) Add manifest link and Apple meta tags in app/layout.tsx head. (5) Build `<InstallPrompt />` component listening for `beforeinstallprompt`, showing a discreet install card after 3 visits. (6) Build `<ConnectionStatus />` indicator in the topbar. Verify: build succeeds, manifest validates, app installs on Chrome desktop and iOS Safari, Lighthouse PWA audit passes.

**Prompt 3 — Design system foundation**

> Set up the design system foundation. (1) Install fonts: Fraunces (variable, Google Fonts) and Switzer (download from Indian Type Foundry, store .woff2 in /public/fonts/). JetBrains Mono via Google Fonts. Configure all three with next/font for optimal loading. (2) Create `/app/globals.css` with the complete design token set from CLAUDE.md — both dark and light theme CSS variables, complete type scale as Tailwind utilities. (3) Configure tailwind.config.ts to consume CSS variables (not hardcoded colors). (4) Add a ThemeProvider that reads/writes `data-theme` on the html element with localStorage persistence and respects `prefers-color-scheme` on first visit. Default theme is dark. Verify: render a test page displaying all three font families and a swatch of every color token, both themes toggle cleanly.

**Prompt 4 — Core UI component library**

> Build the custom UI component library in `/components/ui` per CLAUDE.md design system. Use shadcn/ui as a base only — every component must be restyled, no shadcn defaults survive. Build: Button (primary/secondary/ghost variants), Input, Card, Badge, Tabs, Modal, Drawer, Tooltip, Toast, Table, Select, Switch, Skeleton, ProgressBar, RuleProgressBar (special: shows usage of a rule). Every component uses CSS variables only. Every component has working dark and light themes. Build a component playground at `/playground` (development-only, gated behind a flag) rendering every component with all variants and states. Verify: typecheck and build pass. Visually inspect playground in both themes.

### Phase 1 — Auth & Foundation

**Prompt 5 — Supabase setup and initial migrations**

> Set up Supabase. Configure the project with `.env.example` variables. Create `/lib/supabase/server.ts`, `/lib/supabase/client.ts`, and `/lib/supabase/middleware.ts` using the latest @supabase/ssr patterns for Next.js 15. Create the initial migration with `profiles` table extending Supabase auth.users — columns: id (uuid pk fk auth.users), email, full_name, avatar_url, tier ('trader'/'pro'/'elite'), trial_ends_at, stripe_customer_id, stripe_subscription_id, subscription_status, onboarded boolean, is_admin boolean (default false), created_at, updated_at. Add RLS policies: users can only access their own profiles row. Add a trigger that creates a profiles row when a new user signs up via auth.users. Verify: migrations apply cleanly to a local Supabase instance. RLS test: anonymous user cannot select from profiles.

**Prompt 6 — Auth flows**

> Build authentication pages in `/app/(auth)/`. Sign-up: email, password, full name. Sign-in: email + password and magic link. Include OAuth Google as secondary option. Forms follow the FundedPal design system — editorial layout, asymmetric, generous whitespace. Use Switzer for body and Fraunces for the page title (e.g. "Welcome back."). Implement auth callback at `/app/(auth)/callback/route.ts`. Add middleware in the project root that protects all `/app/(app)/*` routes and redirects unauthenticated users to `/sign-in?redirect=...`. Verify: full sign-up → email confirmation → sign-in → protected dashboard flow works end-to-end.

**Prompt 7 — Application shell**

> Build the protected application shell at `/app/(app)/layout.tsx`. Layout: 240px fixed sidebar left, top bar with breadcrumb + theme toggle + user menu, content area right. Sidebar nav items: Dashboard, Accounts, Strategies, Backtest, Firms, Journal, Billing, Settings. Sidebar uses FundedPal design system — `--ink-lift-1` background, 1px right border in `--border`, nav items in Switzer 14px, active state has 2px champagne left border and `--bone` text. Top bar has FundedPal wordmark in Fraunces on the left when sidebar is collapsed (mobile). User menu is a dropdown with avatar, email, Sign Out. Verify: shell renders correctly in both themes, sidebar collapses to hamburger drawer below 768px, all routes accessible from nav.

### Phase 2 — Prop Firm Rule Engine

**Prompt 8 — Seeded firms with curation status**

> Seed the prop firms catalogue with the 6 supported firms and 6 unsupported firms.
>
> Create the `prop_firms` table:
> ```sql
> create table public.prop_firms (
>   id uuid primary key default gen_random_uuid(),
>   slug text not null unique,
>   name text not null,
>   website text,
>   logo_url text,
>   health_score numeric(3,1),
>   payout_speed_days int,
>   supports_ea boolean not null default false,
>   supports_signals boolean not null default true,
>   jurisdiction text,
>   warnings text[],
>   listing_status text not null default 'supported' check (listing_status in ('supported','unsupported_visible','hidden')),
>   compatibility_status text not null default 'verified_green' check (compatibility_status in ('verified_green','verified_amber','verified_signal_only','verified_split','not_supported','pending_verification')),
>   ea_policy_summary text,
>   ea_policy_source_url text,
>   ea_policy_verified_at timestamptz,
>   auto_execute_allowed boolean not null default false,
>   signal_mode_allowed boolean not null default true,
>   requires_disclosure boolean not null default false,
>   requires_pre_approval boolean not null default false,
>   server_request_cap_per_day int,
>   max_strategy_capital_cents bigint,
>   min_trade_hold_seconds int,
>   created_at timestamptz not null default now()
> );
> ```
>
> Create `unsupported_firms` table:
> ```sql
> create table public.unsupported_firms (
>   id uuid primary key default gen_random_uuid(),
>   slug text not null unique,
>   name text not null,
>   reason text not null,
>   reason_category text not null check (reason_category in ('ea_prohibited','platform_not_supported','pre_approval_required','rules_unclear','compliance_risk','firm_under_investigation')),
>   alternative_firm_slugs text[],
>   rules_visible boolean not null default true,
>   public_rules_summary jsonb,
>   added_at timestamptz not null default now(),
>   last_reviewed_at timestamptz
> );
> ```
>
> Seed the 6 supported firms with full data per `/docs/COMPATIBILITY_RESEARCH.md`: FTMO (verified_green, ea_policy_source_url, server_request_cap_per_day=2000, max_strategy_capital_cents=$400k), FundedNext (verified_amber, requires_disclosure=true), The Funded Trader (verified_split — needs account type overrides), E8 Markets (verified_amber — block Signature Futures and Peak Scalping via overrides), Funded Trading Plus (verified_green), Goat Funded Trader (verified_green).
>
> Seed the 6 unsupported firms: Apex (ea_prohibited, alternatives: ftmo + funded-trading-plus), Topstep (platform_not_supported), MyFundedFX (rules_unclear), FundingPips (rules_unclear), Maven Trading (pre_approval_required), City Traders Imperium (rules_unclear).
>
> Both tables are publicly readable (no RLS restrictions). Verify: 6 supported + 6 unsupported rows exist; queries return them correctly.

**Prompt 9 — Firm rule profiles + account type overrides**

> Create `firm_rule_profiles` table:
> ```sql
> create table public.firm_rule_profiles (
>   id uuid primary key default gen_random_uuid(),
>   firm_id uuid not null references public.prop_firms(id) on delete cascade,
>   account_size_cents bigint not null,
>   phase text not null check (phase in ('phase_1','phase_2','funded','instant_funded')),
>   daily_loss_pct numeric(5,2) not null,
>   daily_loss_basis text not null check (daily_loss_basis in ('starting_balance','equity_high')),
>   max_drawdown_pct numeric(5,2) not null,
>   max_drawdown_type text not null check (max_drawdown_type in ('static','trailing')),
>   profit_target_pct numeric(5,2),
>   min_trading_days int,
>   max_trading_days int,
>   news_restriction jsonb,
>   ea_allowed boolean not null default false,
>   weekend_holding boolean not null default false,
>   hedging_allowed boolean not null default true,
>   consistency_rule_pct numeric(5,2),
>   stop_loss_required boolean not null default true,
>   max_lot_per_position numeric(8,2),
>   max_lots_total numeric(8,2),
>   inactivity_days int,
>   payout_cycle_days int,
>   source_url text,
>   source_extracted_at timestamptz,
>   source_extraction_method text check (source_extraction_method in ('manual','claude_extracted','user_confirmed')),
>   active boolean not null default true,
>   created_at timestamptz not null default now()
> );
> ```
>
> Create `firm_account_type_overrides` table:
> ```sql
> create table public.firm_account_type_overrides (
>   id uuid primary key default gen_random_uuid(),
>   firm_id uuid not null references public.prop_firms(id) on delete cascade,
>   account_type text not null,
>   auto_execute_allowed boolean not null,
>   signal_mode_allowed boolean not null,
>   notes text,
>   unique (firm_id, account_type)
> );
> ```
>
> For each of the 6 supported firms, manually curate rule profiles for the 4 most common account sizes ($10k, $25k, $100k, $200k) across all phases offered. Use only publicly published values from each firm's website. Set source_extraction_method = 'manual' and cite source_url.
>
> For The Funded Trader, populate firm_account_type_overrides for: standard (auto=false, signal=true), rapid (auto=false, signal=true), royal (auto=true, signal=true), knight (auto=true, signal=true).
>
> For E8 Markets, populate overrides for: e8_one (auto=true), e8_classic (auto=true), e8_track (auto=true), e8_signature_forex (auto=true), e8_static_crypto (auto=true), e8_signature_futures (auto=false, signal=false — hidden), peak_scalping (auto=false, signal=false — hidden).
>
> Tables are publicly readable. Verify: query for FTMO Phase 1 $100k returns one row with realistic values. Query for The Funded Trader returns 4 override rows.

**Prompt 10 — Claude rule extraction service**

> Build the AI Rule Detection Engine. Create `/lib/claude/extract-rules.ts` accepting a prop firm name and optional T&Cs URL(s). The function:
>
> (1) Fetches URL content (handles multiple URLs by concatenating with separators)
> (2) Constructs a structured prompt asking Claude Sonnet 4.5 to extract every rule into the firm_rule_profiles schema as strict JSON, with confidence score per field (0-100)
> (3) Parses and validates with Zod
> (4) Returns a candidate profile requiring user confirmation
>
> Add cost guardrails: max 50k input tokens per extraction (truncate intelligently if needed), rate limit 5 extractions/user/hour via Upstash Redis. Cache results by URL+content_hash for 7 days.
>
> Build the API route at `/app/api/claude/extract-rules/route.ts`. Build admin UI at `/app/(app)/firms/extract` (admin-only) where admin pastes URLs, sees extracted rules side-by-side with source text, edits any field, confirms to save. Low-confidence fields (<70) flagged with amber indicator requiring explicit toggle. Set source_extraction_method to 'claude_extracted' then 'user_confirmed' after edit.
>
> Verify: extracting from FTMO's published rules produces a profile that closely matches the manually-seeded one with confidence scores per field.

**Prompt 11 — Public firm directory + scan-but-filter UX**

> Build `/app/(marketing)/firms/page.tsx` (public, no auth) and `/app/(app)/firms/page.tsx` (logged in).
>
> Public version: editorial grid layout — Fraunces hero "Six firms. Hand-verified. Continuously monitored." Filter chips ("All firms", "Most popular", "Best for beginners", "Highest profit splits"). 3-column grid on desktop, single on mobile. Each firm card: logo top-left, name in Fraunces heading-lg, health score as Fraunces 56px display number with sage-to-terracotta color fade, three pill stats (profit split, max funding, payout speed), one-line summary, "View rules" ghost CTA opening drawer with rule profiles.
>
> Logged-in version: same layout + scan-but-filter feature. Search input at top, debounced 200ms, queries both prop_firms (where listing_status != 'hidden') AND unsupported_firms. Filter chips: "All" / "Supported" (default) / "Unsupported".
>
> When search query matches an unsupported_firms row, show special card variant:
> - Background --ink-lift-1, dashed 1px border in --border (vs solid for supported)
> - Caption "NOT CURRENTLY SUPPORTED" in Switzer caption uppercase --bone-muted
> - Firm name in Fraunces at heading-md (smaller than supported cards' heading-xl)
> - Reason in body-sm bone-muted, max 3 lines
> - "We recommend [alternative firm]" link, smooth scroll + highlight target
> - "View rules" button only if rules_visible, opens drawer with public_rules_summary, drawer header has prominent banner "Reference only — accounts cannot be connected"
> - No connect button anywhere on this card
>
> Use real Supabase data via Server Components. Verify: 6 supported firms render in default view; searching "Apex" shows the unsupported card; clicking a supported card opens drawer with all rule profiles.

### Phase 3 — Account Connection & Bridge

**Prompt 12 — Accounts schema + journey framework**

> Create the accounts schema with full journey tracking from day one:
>
> ```sql
> create table public.accounts (
>   id uuid primary key default gen_random_uuid(),
>   user_id uuid not null references public.profiles(id) on delete cascade,
>   firm_id uuid not null references public.prop_firms(id),
>   rule_profile_id uuid not null references public.firm_rule_profiles(id),
>   nickname text,
>   mt5_login text not null,
>   mt5_server text not null,
>   broker_server_timezone text not null default 'America/New_York',
>   starting_balance_cents bigint not null,
>   current_balance_cents bigint not null,
>   current_equity_cents bigint not null,
>   equity_high_cents bigint not null,
>   status text not null default 'pending_connection' check (status in ('pending_connection','live','paused','breached','suspended','closed')),
>   bridge_session_id text,
>   last_heartbeat_at timestamptz,
>
>   -- Journey framework (Account Journeys)
>   account_journey_id uuid not null default gen_random_uuid(),
>   predecessor_account_id uuid references public.accounts(id),
>   is_active_in_journey boolean not null default true,
>   journey_started_at timestamptz not null default now(),
>
>   -- Risk Budget Layer
>   per_trade_risk_pct numeric(5,2) not null default 0.5,
>   daily_soft_cap_pct numeric(5,2) not null default 1.4,
>   max_trades_per_day int not null default 3,
>   phase_aggression_locked boolean not null default true,
>
>   created_at timestamptz not null default now(),
>   updated_at timestamptz not null default now()
> );
>
> create table public.account_phase_transitions (
>   id uuid primary key default gen_random_uuid(),
>   account_id uuid not null references public.accounts(id),
>   account_journey_id uuid not null,
>   from_phase text,
>   to_phase text not null,
>   from_login text,
>   to_login text not null,
>   from_balance_cents bigint,
>   to_balance_cents bigint not null,
>   transition_type text not null check (transition_type in ('phase_1_to_phase_2','phase_2_to_funded','instant_to_funded','reset','breach_replacement','manual_swap')),
>   detected_at timestamptz not null default now(),
>   user_confirmed_at timestamptz,
>   notes text
> );
>
> create table public.account_snapshots (
>   id uuid primary key default gen_random_uuid(),
>   account_id uuid not null references public.accounts(id) on delete cascade,
>   trading_date date not null,
>   starting_balance_cents bigint not null,
>   starting_equity_cents bigint not null,
>   ending_balance_cents bigint,
>   ending_equity_cents bigint,
>   high_equity_cents bigint,
>   low_equity_cents bigint,
>   pnl_cents bigint,
>   trade_count int not null default 0,
>   rules_used jsonb,
>   unique (account_id, trading_date)
> );
>
> create table public.risk_setting_changes (
>   id uuid primary key default gen_random_uuid(),
>   account_id uuid not null references public.accounts(id) on delete cascade,
>   field text not null,
>   old_value numeric(5,2),
>   new_value numeric(5,2),
>   changed_at timestamptz not null default now(),
>   changed_by uuid references public.profiles(id)
> );
> ```
>
> Add indexes: idx_accounts_user, idx_journey_active (user_id, account_journey_id, is_active_in_journey) where is_active_in_journey, idx_snapshots_account_date.
>
> Add RLS: users access only their own accounts and related rows. Add database trigger that inserts risk_setting_changes row on every UPDATE to risk columns.
>
> Verify: schema applies cleanly, RLS enforced, indexes created.

**Prompt 13 — Account connection flow with journey detection**

> Build the account connection flow at `/app/(app)/accounts/connect`.
>
> Three-step wizard:
>
> Step 1 — Select firm: searchable list of supported firms only (firm_id where listing_status='supported'). Live filter as you type. Champagne 1px border on selection. "Don't see your firm?" link opens drawer explaining the curation policy.
>
> Step 2 — Phase + size: pill selector for account size ($10k, $25k, $50k, $100k, $200k), phase selector (Phase 1, Phase 2, Funded, Instant Funded). Below: dynamically populated rule preview card showing matching firm_rule_profile rules. For The Funded Trader and E8 Markets, additional account_type sub-selector reading firm_account_type_overrides; if auto_execute_allowed=false, force signal-only mode with clear explanation.
>
> Step 3 — MT5 credentials: nickname (optional), MT5 login (numeric validation), MT5 password, MT5 server (format validation). Step heading reads "Connect to MetaTrader 5". Helper text under server: "Find this in your prop firm welcome email — example: FTMO-Demo or FTMO-Server". Subtle caption above the form: "MetaTrader 5 only · Other platforms coming in future releases".
>
> **Journey detection** (between Step 2 and Step 3): After firm + phase selected, check if user has any existing accounts at the same firm. If yes, query for accounts where firm_id matches AND is_active_in_journey=true AND any phase transition is recent (passed Phase 1 or Phase 2 within 30 days, OR user has a Funded account already). Show a journey continuation card:
>
> > "Looks like this might be your new account from passing [previous phase]. Should we continue your existing FTMO journey?"
> >
> > Yes, continue journey · No, this is a new account
>
> If user confirms continuation:
> - Set predecessor_account_id to the existing account
> - Inherit account_journey_id from predecessor
> - Mark predecessor's is_active_in_journey=false
> - Create account_phase_transitions row with appropriate transition_type
> - Bypass tier limit checks (journey continuations don't count as new journeys)
> - Show journey timeline summary at confirmation
>
> If user says no, this is a new account:
> - Generate fresh account_journey_id
> - Apply tier limit check (count active journeys for user; reject if at cap)
>
> Compatibility summary panel at confirmation step lists what FundedPal will and won't do on this firm + account type.
>
> On submit, create accounts row with status 'pending_connection' and redirect to "waiting for bridge" screen.
>
> Verify: full flow creates database row with correct rule_profile_id; journey detection prompts for FTMO Phase 1 → Phase 2 → Funded sequence; tier limits enforce on truly new accounts.

**Prompt 14 — MT5 EA bridge protocol**

> Specify and build the MT5 EA bridge.
>
> Create `/lib/bridge/messages.ts` defining the WebSocket message protocol with strict TypeScript types and Zod schemas:
> - HELLO (auth with HS256-signed session token)
> - INSTRUMENT_SPECS (EA → server, sent immediately after HELLO; reports broker-specific pip value, tick size, contract size, min lot step for every supported symbol)
> - HEARTBEAT (every 5 seconds)
> - ACCOUNT_STATE (balance/equity/positions snapshot)
> - POSITION_OPEN_REQUEST (server → EA, includes client_request_id UUID for idempotency)
> - POSITION_OPEN_RESULT (EA → server)
> - POSITION_CLOSE_REQUEST
> - POSITION_CLOSED
> - ERROR
>
> Create the `account_instrument_specs` table to store broker-reported specifications:
>
> ```sql
> create table public.account_instrument_specs (
>   id uuid primary key default gen_random_uuid(),
>   account_id uuid not null references public.accounts(id) on delete cascade,
>   symbol text not null,
>   pip_value_account_currency_per_lot numeric(10,4) not null,
>   pip_decimal numeric(10,8) not null,
>   contract_size numeric(12,2) not null,
>   min_lot_step numeric(6,3) not null,
>   detected_at timestamptz not null default now(),
>   unique (account_id, symbol)
> );
> ```
>
> Build `/lib/bridge/ws-server.ts` as WebSocket server (run as separate Node service on Railway). Authenticate every connection via HELLO with valid HS256 token signed with BRIDGE_SHARED_SECRET, scoped to specific account_id, expires in 24h. Server stores recent client_request_ids for 24h to enforce idempotency.
>
> On INSTRUMENT_SPECS: persist all 10 instrument specs to account_instrument_specs (upsert by account_id + symbol). Mark account.status='live' only after specs received for all 10 supported instruments.
>
> On HEARTBEAT: update accounts.last_heartbeat_at; mark as paused if no heartbeat for 60 seconds.
>
> On ACCOUNT_STATE: persist account_snapshots, update accounts.current_equity_cents, update accounts.equity_high_cents only if exceeds (database trigger or app logic).
>
> On reconnect: server reconciles its known positions against EA's snapshot. If server thinks position open but EA says closed → mark closed with closed_at=reconnection_time, log reconciliation_event. If server has no record but EA reports open → create position with source='reconciled' flag. Reconciliation events fire Logsnag alert.
>
> Implement exponential backoff reconnection (1s → 2s → 4s → 8s → 16s → 30s max). Latency monitoring per trade: alert if 95th percentile RTT > 2 seconds.
>
> Verify: mock WebSocket client can authenticate, send INSTRUMENT_SPECS for all 10 symbols, heartbeat, have state persisted, handle reconnect with reconciliation.

**Prompt 15 — MQL5 EA implementation**

> Build the MetaTrader 5 EA bridge in `/ea-bridge/FundedPalBridge.mq5`.
>
> The EA: (1) reads FundedPal account session token from input field, (2) connects to FundedPal WebSocket server, (3) sends HELLO with token, (4) **immediately after HELLO acknowledgement, queries broker specs for all 10 supported symbols (EURUSD, GBPUSD, USDJPY, USDCHF, AUDUSD, USDCAD, NZDUSD, EURGBP, EURJPY, XAUUSD) using `SymbolInfoDouble(symbol, SYMBOL_TRADE_TICK_SIZE)`, `SymbolInfoDouble(symbol, SYMBOL_TRADE_TICK_VALUE)`, `SymbolInfoDouble(symbol, SYMBOL_TRADE_CONTRACT_SIZE)`, and `SymbolInfoDouble(symbol, SYMBOL_VOLUME_STEP)` and sends them as an INSTRUMENT_SPECS message**, (5) every 5 seconds sends ACCOUNT_STATE with balance/equity/free margin and array of open positions, (6) listens for POSITION_OPEN_REQUEST, validates client_request_id is not duplicate, executes via OrderSend(), replies with POSITION_OPEN_RESULT, (7) handles POSITION_CLOSE_REQUEST similarly.
>
> **Critical for gold (XAUUSD):** the EA must accurately report the broker's actual pip definition. Different brokers use different conventions ($0.10/pip, $0.01/pip, or $1.00/pip). Never assume — always query the broker. The INSTRUMENT_SPECS values for XAUUSD will determine all server-side position sizing for gold.
>
> Symbol mapping: handle broker-specific suffixes (.r, .pro, -Pro, .m, _PRO) via `symbol_aliases` config table. Function `mapSymbol(canonical)` returns broker-specific name. The EA queries broker specs using the broker's actual symbol but reports them to the server using the canonical symbol.
>
> Pip value handling: per symbol class — 4-decimal majors (0.0001), JPY pairs (0.01), gold (broker-dependent — query and report). Function `getPipValue(symbol)` returns precise value for fallback only; production always uses the broker-reported value from SYMBOL_TRADE_TICK_VALUE.
>
> Lot calculations: respect broker's minimum lot step (round DOWN, never up). Standard/mini/micro lot conversion uses broker's reported contract size, never hardcoded.
>
> Error handling: every TRADE_RETCODE_* maps to structured server error with human-readable message. Common errors handled gracefully (insufficient margin, market closed, invalid stops, requote).
>
> Include status indicator on chart showing connection state and instrument spec sync state. Provide install instructions in `/ea-bridge/README.md` including a section: "Why FundedPal queries broker specs at connection time" explaining the gold pip variation issue to traders who care.
>
> Verify: load EA on demo MT5 account; connects, reports correct INSTRUMENT_SPECS for all 10 symbols including gold's broker-specific pip value, sends heartbeats, accepts manual test order from server, handles disconnect with reconnect within 30 seconds. Compare reported gold pip value against the broker's published spec sheet — should match exactly.

**Prompt 16 — Single-account dashboard**

> Build `/app/(app)/accounts/[id]/page.tsx`.
>
> Layout: editorial hero showing nickname + firm + phase + account equity as Fraunces display-2xl in JetBrains Mono, day's P&L below in mono-lg (sage if positive, terracotta if negative, with explicit + or − sign). Three quick stats inline: Days trading · Best day · Win rate. Phase indicator pill: "Phase 1 · 8% target · 5% daily" in caption with champagne border.
>
> Below hero, 12-column grid:
>
> Row 1: Equity curve chart (full width, last 30 days) with sage profit zones, terracotta drawdown zones, subtle markers for trades.
>
> Row 2: Rules-usage card (8 columns) showing every active rule with RuleProgressBar (Daily Loss, Max Drawdown, Per-Trade Risk, Daily Soft Cap, Trades today, Server requests today for FTMO). Killzone clock (4 columns) — circular 24-hour indicator highlighting active sessions.
>
> Row 3: Open positions table (full width) — borderless rows, hover lifts row, columns: Symbol, Side, Lots, Entry, SL, TP, P&L, Time. All numbers in mono with tabular-nums.
>
> Row 4: Recent trades (8 columns) + Recent compliance decisions (4 columns) — list with sage check (approved) or terracotta x (rejected), time, rule that triggered.
>
> Real-time updates via Supabase Realtime; equity and P&L tick smoothly when ACCOUNT_STATE arrives. Number animations use easeOutQuart over 600ms.
>
> Add **journey timeline** as a subtle section between hero and grid: horizontal progression bar showing the journey's history (e.g. Phase 1 → Phase 2 → Funded), with current phase highlighted in champagne. Click expands to show full transition history with dates and balances.
>
> Verify: dashboard renders correctly, real-time updates work without flicker, journey timeline displays accurately for accounts with predecessors.

### Phase 4 — Compliance Guardian

**Prompt 17 — Compliance Guardian core**

> Build the Compliance Guardian core. Create `/lib/compliance/types.ts` defining RuleEvaluation, ComplianceDecision, OrderProposal types.
>
> Create `/lib/compliance/guard.ts` exporting async `evaluateOrder(account, proposedOrder)` returning `{ decision: 'approved' | 'rejected', reasons: string[], evaluations: RuleEvaluation[] }`.
>
> The function loads the account's active firm_rule_profile and evaluates rules by calling specialised checkers in `/lib/compliance/rule-checks.ts`:
>
> - checkInstrumentSpecsAvailable (NEW — verifies account_instrument_specs row exists for the proposed symbol; rejects with "Instrument specifications not yet available for {symbol}. Please wait for the bridge to fully sync." if missing)
> - checkStopLossRequired
> - checkLotSize
> - checkHedgingAllowed
> - checkWeekendHolding
> - checkNewsRestriction
> - checkConsistencyRule
> - checkDailyLoss (worst-case loss + realised + open exposure vs daily_loss_pct)
> - checkMaxDrawdown (current equity vs starting/equity_high based on type)
> - checkEAAllowed
> - checkInactivity
>
> **Position sizing math reads from `account_instrument_specs`, not from a global constants file.** This is critical for gold (XAUUSD) which has broker-specific pip values. Specifically, `worst_case_loss = (entry - stop_loss) × lot_size × account_instrument_specs.pip_value_account_currency_per_lot / account_instrument_specs.pip_decimal`. Currency conversion (if account currency != USD) handled via live OANDA rate cached 60s; reject orders if rate unavailable. All math in bigint cents — no floats.
>
> "Today" defined by accounts.broker_server_timezone, not UTC, not user local.
>
> Every evaluation logs to compliance_decisions (immutable). Add unit tests covering 20+ scenarios per checker, including: gold with $0.10/pip broker, gold with $0.01/pip broker, gold with $1.00/pip broker, JPY pairs, FX majors, timezone edges, missing instrument specs.
>
> Verify: tests pass; trade breaching daily loss rejected with clear reason; gold trade with broker-reported $0.10/pip produces correct worst-case calculation; gold trade attempted before INSTRUMENT_SPECS sync rejected with appropriate message.

**Prompt 18 — Risk Budget Layer**

> Implement the FundedPal risk budget layer that protects accounts from approaching prop firm hard rules. This sits inside the Compliance Guardian as soft-cap layer above the hard rules.
>
> Create `/lib/compliance/risk-defaults.ts` exporting `calculateDefaultRiskSettings(ruleProfile, phase)`:
>
> ```
> phase = 'phase_1':
>   daily_soft_cap_pct = ruleProfile.daily_loss_pct × 0.70
>   per_trade_risk_pct = min(daily_soft_cap_pct × 0.25, 1.0)
>   max_trades_per_day = 3
>
> phase = 'phase_2':
>   daily_soft_cap_pct = ruleProfile.daily_loss_pct × 0.65
>   per_trade_risk_pct = min(daily_soft_cap_pct × 0.20, 1.0)
>   max_trades_per_day = 2
>
> phase = 'funded' or 'instant_funded':
>   daily_soft_cap_pct = ruleProfile.daily_loss_pct × 0.60
>   per_trade_risk_pct = min(daily_soft_cap_pct × 0.15, 1.0)
>   max_trades_per_day = 2
> ```
>
> Add to `/lib/compliance/rule-checks.ts`:
> - checkPerTradeRisk: worst-case loss / starting_balance vs per_trade_risk_pct. Rejection includes suggested adjustment ("Reduce lot size to 0.6 or widen stop to 1.10180").
> - checkDailySoftCap: realised today + worst-case open + worst-case proposed vs daily_soft_cap_pct.
> - checkMaxTradesPerDay: positions opened today (any status) vs max_trades_per_day.
>
> Update `/lib/compliance/guard.ts` to run soft checks BEFORE hard checks:
> 1. checkPerTradeRisk (soft)
> 2. checkDailySoftCap (soft)
> 3. checkMaxTradesPerDay (soft)
> 4-13. existing hard checks
>
> Soft checks tagged severity:'soft', hard checks severity:'hard'. UI distinguishes visually but functionally both block.
>
> Update `/actions/accounts.ts` connectAccount() to call calculateDefaultRiskSettings() and populate risk columns when new account created. On phase transition (handled in Prompt 22.5 wiring), recalculate defaults.
>
> Build `/app/(app)/accounts/[id]/risk/page.tsx`:
> - Fraunces "Risk Budget" hero
> - Three sliders: per-trade (0.10%-1.00% bounded), daily soft cap (10%-75% of hard rule), max trades (1-10 stepper)
> - Live calculator card translating settings to dollars
> - Two-zone visualisation bar (sage zone → amber → terracotta hard limit)
> - Audit log table (last 10 changes from risk_setting_changes)
> - Explanatory copy: "These limits exist to protect your account. They cannot be raised beyond reasonable values."
>
> Server action updateRiskSettings() validates against hardcoded ceilings server-side: per-trade ≤ 1.0%, soft cap ≤ 75% of firm's daily_loss_pct. Reject with 422 if exceeded.
>
> Add Risk tab to per-account navigation. Tab shows sage check (healthy), amber dot (approaching soft cap), terracotta (soft cap hit today).
>
> Verify: 25+ test scenarios pass; FTMO Phase 1 $100k auto-defaults to per_trade ~0.875%, soft cap ~3.5%, max trades 3; settings UI works in both themes.

**Prompt 19 — Pre-trade simulation UI**

> Build `<PreTradeSimulation account={...} order={...} />` in `/components/compliance/`. Given proposed order, calls Compliance Guardian and renders structured result.
>
> Layout:
> - Status header: green-if-approved or terracotta-if-rejected
> - "Risk Budget" section (Switzer caption, sage accent) showing soft-cap evaluations:
>   - Each soft cap: name, current usage, projected usage after trade, sparkline progression
>   - Sage/amber/terracotta progression
> - "Prop Firm Rules" section (Switzer caption, champagne accent) showing hard-rule evaluations:
>   - Each rule: name, binary pass/fail
>   - Rule name and rejection reason if failed
> - Bottom: trade's worst-case impact summary on daily loss, max drawdown, consistency ratio in JetBrains Mono with sparkline projection bars
>
> Soft-cap rejections include "Suggested adjustment" line per rejection_reasons format.
>
> Component appears in manual order modal and signal review screen. Framer Motion staggered row reveal on first render (60ms intervals).
>
> Verify: synthetic over-leveraged order shows every rule evaluation correctly; rejection reasons human-readable.

**Prompt 20 — Manual order entry**

> Build manual order entry. From single-account dashboard, "Place Order" button opens modal with fields: symbol (search-select), side (buy/sell toggle), lot size, stop loss (price or pips), take profit (price or pips).
>
> On any field change, `<PreTradeSimulation>` re-runs Compliance Guardian. Submit button disabled while decision === 'rejected'.
>
> On submit, order goes through `/actions/positions.ts` placeOrder(): (1) re-evaluates compliance server-side (never trust client), (2) sends POSITION_OPEN_REQUEST via bridge with new client_request_id, (3) creates positions row with status 'pending'. On EA confirmation status flips to 'open'.
>
> Verify: valid order places successfully on demo MT5; position appears in dashboard.

### Phase 5 — SMC/ICT Strategy Engine

**Prompt 21 — Python strategy engine scaffolding**

> Set up the Python strategy engine in `/strategy-engine/`. FastAPI app with endpoints:
> - POST /signals/generate (input: symbol, timeframe, account_id; output: signal or null)
> - POST /signals/backtest (input: backtest config; output: backtest_id)
> - GET /signals/backtest/{id} (status + results)
>
> Use Pydantic v2 for validation. pandas-ta and polars for data handling. OANDA API for market data (free tier sufficient for development; production needs paid feed). Authenticate every request with STRATEGY_ENGINE_API_KEY.
>
> Provide Dockerfile. README with run instructions.
>
> Verify: uvicorn starts; /signals/generate for EURUSD 1h returns valid response shape (even if signal is null).

**Prompt 22 — SMC/ICT primitives**

> Implement SMC/ICT analysis primitives in `/strategy-engine/smc/` and `/strategy-engine/ict/`. Each primitive returns typed structured data.
>
> Centralise all parameters in `/strategy-engine/config/parameters.py` with comments citing sources:
>
> - detect_market_structure: HH/HL/LH/LL across swing-point window. Use 5-bar fractal-based swing detection.
> - detect_bos: break of structure on close beyond last swing
> - detect_choch: change of character — first opposite-direction structure shift
> - detect_order_block: last opposite candle before impulsive structure-breaking move with displacement ≥ 2× previous candle range. Mitigated when price closes through.
> - detect_fvg: 3-candle definition. Bullish FVG: gap between candle 1's high and candle 3's low. Bearish: candle 1's low and candle 3's high. Filled at midpoint (50%).
> - detect_breaker_block: failed order block becomes resistance/support
> - detect_liquidity_pools: equal highs/lows within tolerance. Tolerance: 0.1 ATR for FX majors, 0.15 ATR for metals, 0.2 ATR for indices.
> - detect_killzone: current time in Asian (20:00-00:00 EST), London (02:00-05:00 EST), NY AM (08:30-11:00 EST), NY PM (13:30-16:00 EST), with DST handling
>
> Add unit tests in `/strategy-engine/tests/test_known_patterns.py` with 10+ hand-curated historical examples (5 patterns × 2 symbols), verifying engine identifies each correctly within 1 candle.
>
> Verify: pytest passes; primitives correctly identify a known order block on EURUSD 1h on a date you can manually validate.

**Prompt 23 — Confluence scoring & signal generation**

> Build confluence scoring in `/strategy-engine/smc/confluence.py`. Given current chart state, factors weighted explicitly in `/strategy-engine/config/parameters.py`:
>
> - HTF bias (4h or daily structure direction): 20 points
> - Proximity to valid POI (order block / breaker / FVG): 20 points
> - Liquidity sweep just occurred: 15 points
> - In active killzone: 15 points
> - OTE level reached (62-79% retracement into POI): 15 points
> - Structure shift confirmed on entry timeframe: 15 points
>
> Total = confluence score (0-100). Generate signal only when score ≥ 70 (configurable per phase via phase_aggression_curve from Prompt 27).
>
> Signal includes: entry, stop_loss (just beyond POI), take_profit (next liquidity pool, minimum 2R). Position sizing risk-based per Risk Budget Layer — `(per_trade_risk_pct × starting_balance) / (entry-SL pips × pip_value)`, rounded DOWN to broker's lot step.
>
> Persist generated signals to Supabase signals table:
>
> ```sql
> create table public.signals (
>   id uuid primary key default gen_random_uuid(),
>   account_id uuid references public.accounts(id) on delete set null,
>   symbol text not null,
>   timeframe text not null,
>   side text not null check (side in ('buy','sell')),
>   entry_price numeric(12,5) not null,
>   stop_loss numeric(12,5) not null,
>   take_profit numeric(12,5) not null,
>   confluence_score int not null,
>   reasoning jsonb not null,
>   killzone text,
>   status text not null default 'generated' check (status in ('generated','executed','expired','rejected')),
>   expires_at timestamptz,
>   created_at timestamptz not null default now()
> );
> ```
>
> Lookahead bias check: every decision at time T uses only data ≤ T. Test by shifting backtest data forward by 1 candle; results must change.
>
> **Supported instruments at v1 (10 total):** EUR/USD, GBP/USD, USD/JPY, USD/CHF, AUD/USD, USD/CAD, NZD/USD, EUR/GBP, EUR/JPY, XAU/USD. The strategy engine generates signals only for these instruments. The instrument list is hardcoded in `/strategy-engine/config/instruments.py` and matches the canonical symbols used throughout the platform.
>
> **Gold (XAUUSD) specific handling in the strategy engine:** position sizing uses the per-account broker-reported pip value from `account_instrument_specs`, not a hardcoded value. Backtests for gold require verifying the historical data uses the same pip convention as the production broker — backtest config must specify the assumed pip definition explicitly.
>
> Backtest sanity criteria (per instrument, 3 months historical data):
> - Signal frequency: 5-25 signals
> - Win rate: 40-65%
> - Average R: ≥ 1.5
> - Max drawdown: < 15%
>
> Run sanity check on at least 3 different instruments before signing off Prompt 23 (recommend: EURUSD, GBPUSD, XAUUSD — covers FX major, FX-with-volatility, and metal cases).
>
> Verify: backtested over 3 months, engine produces sensible numbers across multiple symbols including gold.

**Prompt 24 — Signal review & one-tap execute**

> Build signal review for EA Compliance Mode. Create `/app/(app)/strategies/page.tsx`.
>
> Page hero Fraunces display-lg "Signals." Subhead: "Live SMC/ICT signals across your connected accounts. Each one passes the Compliance Guardian before reaching you."
>
> Filter bar: account, symbol, confluence threshold slider.
>
> Single-column signal feed. Each signal card:
> - Top: symbol + side (BUY/SELL pill) + ConfluenceMeter (0-100, sage gradient)
> - Mono-lg numbers: Entry · SL · TP · R:R
> - SMC/ICT reasoning structured list (HTF bias, POI type, liquidity sweep, killzone, OTE)
> - <PreTradeSimulation> panel
> - Two CTAs: "One-Tap Execute" (champagne, only enabled if compliance approves) · "Dismiss" (ghost)
>
> New signals slide in from top with 400ms cubic-bezier(0.16, 1, 0.3, 1). High-confluence (≥85) gets subtle teal pulse.
>
> Web Push integration:
> ```sql
> create table public.push_subscriptions (
>   id uuid primary key default gen_random_uuid(),
>   user_id uuid not null references public.profiles(id) on delete cascade,
>   endpoint text not null,
>   p256dh text not null,
>   auth text not null,
>   user_agent text,
>   platform text,
>   created_at timestamptz not null default now(),
>   last_seen_at timestamptz not null default now(),
>   unique (user_id, endpoint)
> );
> ```
>
> Register Web Push subscription on first dashboard load (after 30s of activity OR after 1st signal viewed) with explicit permission prompt. Send notification for signals with confluence ≥85 (Pro+ tier). Notification deep-links to signal in dashboard for one-tap execute. iOS support: requires PWA installed first.
>
> See `/docs/PWA_NOTES.md` for VAPID keys and Web Push setup details.
>
> Verify: mock signal inserted appears live; push notification delivers on Chrome/Android; one-tap execute places order through bridge.

### Phase 6 — Innovative Features

**Prompt 25 — Account journey transitions handler**

> Build the account journey transition system. This handles the case where a trader's existing account passes a phase and they get a new MT5 login for the next phase.
>
> Create `/lib/journey/transitions.ts`:
>
> Function `detectPhaseTransition(account)` runs on every ACCOUNT_STATE update from the bridge. Detects when:
> - Phase 1 account hits profit target → phase passed
> - Phase 2 account hits profit target → phase passed
> - Account balance goes to zero or negative → breach
> - Account is closed at the broker → trader received new credentials
>
> When phase pass detected: marks account.status='paused', shows banner in dashboard "Congratulations — looks like you passed Phase 1. Connect your new account to continue this journey."
>
> Function `executeTransition(predecessorAccount, newAccountData, transitionType)` handles the actual transition:
> 1. Creates new accounts row inheriting account_journey_id
> 2. Sets predecessor_account_id reference
> 3. Marks predecessor's is_active_in_journey=false
> 4. Inserts account_phase_transitions row
> 5. Recalculates risk settings via calculateDefaultRiskSettings() for new phase
> 6. Notifies user in-app + email summarising new rules
> 7. Pauses any open orders pending review against new ruleset
>
> Update `/actions/accounts.ts` connectAccount():
> - When user confirms "this is my new account from passing [phase]", calls executeTransition
> - When user says "this is a new account", checks tier limit on active journeys (count distinct account_journey_id where is_active_in_journey=true) and rejects if at cap
>
> Build journey view at `/app/(app)/accounts/[id]/journey/page.tsx`:
> - Editorial timeline showing the journey's full history
> - Each transition: from_login, to_login, from_balance, to_balance, transition_type, date
> - Cumulative P&L visualisation
> - Total payouts received (summed from withdrawal events when implemented)
>
> Add "Journey" tab to per-account navigation alongside Rules, Positions, Risk, Journal.
>
> Add tier limit calculation helper `getActiveJourneyCount(userId)` used by tier gating.
>
> Verify: connect Phase 1 → simulate phase pass → connect Phase 2 → verify journey continuity (same journey_id, predecessor reference, transition row created); journey timeline displays correctly; tier limit counts journeys not logins.

**Prompt 26 — Phase-aware strategy adaptation**

> Implement phase-aware strategy adaptation. In strategy engine, add phase_aggression_curve parameter to signal generation:
>
> - Phase 1: aggressive (confluence threshold 65, can target 1.5R, allow 2 trades per killzone)
> - Phase 2: balanced (threshold 75, 2R target, 1 trade per killzone)
> - Funded: preservation (threshold 85, 3R target, 1 trade per day max, position size halved)
>
> Wire account's current firm_rule_profile.phase into every signal request. Phase changes propagate via journey transitions (Prompt 25) — when transition recorded, signal generator switches aggression curve.
>
> In dashboard, show current aggression mode as small badge near account hero.
>
> When journey transitions to Funded, automatically halve per_trade_risk_pct via calculateDefaultRiskSettings (Funded uses 15% multiplier vs Phase 1's 25%).
>
> Verify: Phase 1 account generates more signals than same account configured as Funded over identical historical data.

**Prompt 27 — Multi-account risk allocator (Pro+)**

> Build the Multi-Account Risk Allocator (Pro and Elite only). Create `/lib/strategy/allocator.ts`.
>
> When multiple journeys are active for a user, before any signal executes, allocator:
>
> 1. Checks correlation — if similar position already open on another journey's active account, skip or reduce size
> 2. Enforces total portfolio risk cap (sum of worst-case losses across all active accounts ≤ configurable threshold, default 4% of total portfolio equity per day)
> 3. Optionally rotates between accounts so they don't all blow on the same losing day
>
> Settings panel at `/app/(app)/settings/allocator` to configure these.
>
> Tier-gate at action level — Trader-tier users see clear upsell when accessing.
>
> Verify: with 3 mock accounts (different journeys), opening correlated trade on account 1 causes allocator to skip same signal on accounts 2 and 3.

**Prompt 28 — Drawdown forecaster (Pro+)**

> Build the Drawdown Forecaster. Create `/lib/strategy/forecaster.ts` that, given proposed order and user's recent trade history (last 30 trades), uses Monte Carlo simulation of 1,000 paths with empirical win rate, average R-win, average R-loss, current sequence to estimate:
>
> (a) probability trade triggers >2% drawdown
> (b) expected drawdown depth
> (c) probability of breaching daily loss limit if this trade plus one more average loss occurred
>
> Display forecast as panel inside `<PreTradeSimulation>` for Pro and Elite tiers.
>
> Verify: with 30 mock trades and varying confluence score, forecaster returns sensible probabilities.

**Prompt 29 — News auto-pause**

> Build News Auto-Pause. Integrate ForexFactory's calendar (or Trading Economics if budget). Background worker every hour ingests next 24 hours of high-impact events into `news_events` table.
>
> Before any order placed, Compliance Guardian's checkNewsRestriction reads news_events for symbol's currencies within firm's blackout window (from firm_rule_profiles.news_restriction). Reject order with rejection_reason if blackout applies.
>
> Show upcoming news events on each account dashboard as small chips with countdown timers.
>
> Verify: with mock NFP event seeded 1 hour out, EURUSD order rejected with reason "News blackout: NFP at 13:30 UTC".

**Prompt 30 — Killzone scheduling**

> Build Killzone Scheduling. Strategy engine, by default, only generates signals during configured killzones.
>
> Create `/app/(app)/settings/killzones` showing four toggles (Asian, London, NY AM, NY PM) with timezone-aware time displays. Pro+ can customise window times within ±30 minutes of standard.
>
> Dashboard shows visual `<KillzoneClock>` — circular 24-hour indicator highlighting active sessions and current time.
>
> Verify: outside all enabled killzones, strategy engine returns no signals.

**Prompt 31 — Backtest module**

> Build Backtest module. Create backtests table:
>
> ```sql
> create table public.backtests (
>   id uuid primary key default gen_random_uuid(),
>   user_id uuid not null references public.profiles(id) on delete cascade,
>   firm_id uuid references public.prop_firms(id),
>   rule_profile_id uuid references public.firm_rule_profiles(id),
>   symbol text[] not null,
>   timeframe text not null,
>   date_from date not null,
>   date_to date not null,
>   config jsonb not null,
>   results jsonb,
>   status text not null default 'queued',
>   created_at timestamptz not null default now(),
>   completed_at timestamptz
> );
> ```
>
> UI at `/app/(app)/backtest`: configure (firm + rule profile + account size + symbols + timeframe + date range + risk profile: "FundedPal default" or "Custom"), submit creates backtests row and triggers Python engine.
>
> Engine runs strategy with Compliance Guardian active including Risk Budget Layer (so backtest shows what FundedPal would actually do, not maximum aggression).
>
> Results page: equity curve, drawdown curve, win rate, profit factor, average R, number of compliance rejections (and why), verdict on whether strategy would have passed Phase 1.
>
> Use Recharts with FundedPal theme.
>
> Verify: backtesting EURUSD 1h FTMO Phase 1 $100k over last 3 months completes and produces coherent verdict.

**Prompt 32 — Journal co-pilot (Pro+)**

> Build the AI Journal Co-pilot. Every closed position auto-creates journal_entries row with entry_type='auto_trade', SMC/ICT reasoning, market context, outcome.
>
> ```sql
> create table public.journal_entries (
>   id uuid primary key default gen_random_uuid(),
>   user_id uuid not null references public.profiles(id) on delete cascade,
>   account_id uuid references public.accounts(id) on delete set null,
>   position_id uuid references public.positions(id) on delete set null,
>   entry_type text not null check (entry_type in ('auto_trade','manual_note','ai_reflection','compliance_rejection')),
>   title text,
>   body text,
>   ai_analysis jsonb,
>   created_at timestamptz not null default now()
> );
> ```
>
> Build `/app/(app)/journal` showing entries in editorial timeline — Fraunces date headers, Switzer bodies, mono numbers.
>
> Chat sidebar where user asks questions. Pass recent journal entries and trades to Claude Sonnet 4.5 with system prompt explaining user's strategy. Returns structured answer citing specific entries with click-to-jump links.
>
> Compliance rejection journal entries auto-created when soft caps reject trades — using rejection message format from Prompt 18.
>
> Verify: 5 sample trades produce 5 auto entries; chat about one returns response referencing actual reasoning.

**Prompt 33 — Continuous rule monitoring**

> Build the Continuous Rule Monitoring system.
>
> Create firm_rule_snapshots table:
>
> ```sql
> create table public.firm_rule_snapshots (
>   id uuid primary key default gen_random_uuid(),
>   firm_id uuid not null references public.prop_firms(id) on delete cascade,
>   source_url text not null,
>   content_hash text not null,
>   raw_content text,
>   diff_from_previous jsonb,
>   detected_at timestamptz not null default now(),
>   reviewed_by uuid references public.profiles(id),
>   reviewed_at timestamptz,
>   review_action text check (review_action in ('no_change','minor_update','significant_change','breaking_change')),
>   review_notes text
> );
> ```
>
> Build `/lib/monitor/rule-watcher.ts` exporting `runRuleMonitor()`:
> - Iterates firms where listing_status='supported'
> - Fetches ea_policy_source_url, cleans HTML via cheerio (strip nav/footer/scripts)
> - Computes SHA256 of cleaned content
> - Compares against most recent firm_rule_snapshots row
> - Hash differs: stores new snapshot with review_action=null, fires Logsnag alert
> - Hash matches: stores snapshot with review_action='no_change'
> - Handles fetch errors via Sentry, doesn't false-positive alert
> - 5-second delay between firms, User-Agent identifies as FundedPal
>
> Schedule via Vercel cron in vercel.json: `{"path": "/api/cron/rule-monitor", "schedule": "0 6 * * *"}`. Cron endpoint verifies Authorization: Bearer ${CRON_SECRET}.
>
> Build admin review panel at `/app/(app)/firms/admin/changes` (gated by profiles.is_admin):
> - Editorial table of unreviewed snapshots, newest first
> - Click row navigates to detail with side-by-side diff (old left dimmed, new right highlighted) using react-diff-viewer-continued
> - Four action buttons with tooltips: No change / Minor update / Significant change / Breaking change
> - On significant_change or breaking_change: notify affected users (in-app + email); breaking_change auto-pauses affected accounts and forces re-acknowledgement
>
> Build public rule change feed at `/app/(marketing)/changelog/firms` showing last 90 days of detected changes. Editorial layout with Fraunces month headers. SEO-optimised. Sub-route `/firms/[slug]` shows per-firm history.
>
> Verify: identical hash produces no alert; different hash triggers alert and review row; cron endpoint rejects unauthorised requests.

**Prompt 34 — Firm Health Score**

> Build community feedback for firm health scores:
>
> ```sql
> create table public.firm_reviews (
>   id uuid primary key default gen_random_uuid(),
>   user_id uuid not null references public.profiles(id) on delete cascade,
>   firm_id uuid not null references public.prop_firms(id) on delete cascade,
>   payout_speed_rating int check (payout_speed_rating between 1 and 5),
>   support_rating int check (support_rating between 1 and 5),
>   rule_clarity_rating int check (rule_clarity_rating between 1 and 5),
>   would_recommend boolean,
>   comment text,
>   created_at timestamptz not null default now(),
>   unique (user_id, firm_id)
> );
> ```
>
> Build `/app/(app)/firms/[slug]/review` for users with at least one account at the firm to submit reviews.
>
> Health Score recalculated daily by Supabase function — weighted average of all rating fields, normalised to 0-10.
>
> Public firms page (Prompt 11) reads health_score from prop_firms table.
>
> Verify: submitting review changes firm's health score after function runs.

### Phase 7 — Billing, Polish, Launch

**Prompt 35 — Stripe billing**

> Wire up Stripe billing. Create `/lib/stripe/client.ts` and `/lib/stripe/plans.ts` defining three plans (Trader $39, Pro $69, Elite $199) with price IDs from env.
>
> Build `/app/(app)/billing/page.tsx`:
> - Current plan card with Fraunces tier name, status (Trial Day 7/14, Active, Canceled), next billing date and amount in mono, "Manage billing" CTA opening Stripe Customer Portal
> - Plan comparison with current tier highlighted, Upgrade/Downgrade buttons inline
> - Invoice history — borderless rows, columns date/amount/status/download
>
> New signups get 14-day trial on selected tier — card required at signup.
>
> Webhook handler at `/app/api/webhooks/stripe/route.ts` handling: subscription.created/updated/deleted, invoice.payment_succeeded/failed, subscription.trial_will_end (3 days before — send email). Updates profiles.tier, subscription_status, trial_ends_at.
>
> Verify: full signup → trial → upgrade → cancel flow works in Stripe test mode.

**Prompt 36 — Tier gating**

> Implement tier gating. Create `/lib/tier.ts` exporting:
> - `requireTier(user, requiredTier)` — server action helper
> - `canAccess(user, feature)` — UI helper
> - `getActiveJourneyCount(userId)` — counts active journeys
> - `canAddJourney(user)` — checks tier limit (Trader: 1, Pro: 3, Elite: unlimited)
>
> Decorate every Pro-only feature (Multi-Account Allocator, advanced backtest, journal co-pilot, drawdown forecaster) and Elite-only feature (unlimited journeys, API access, custom strategies).
>
> When Trader-tier user hits Pro feature, render styled upgrade card in place — Fraunces "Unlock with Pro", concise feature list, "Upgrade for $69/mo" CTA.
>
> Trial users get full access to selected tier.
>
> Critical: tier limit checks count active journeys, not MT5 logins. Phase transitions never trigger tier limit (handled by journey detection in Prompt 25).
>
> Verify: Trader-tier test user cannot access /settings/allocator; clicking shows upgrade card. Pro user with 3 journeys cannot connect 4th brand-new account but can transition any journey to next phase without limit.

**Prompt 37 — Marketing site & onboarding**

> Build marketing site at `/app/(marketing)/`. Pages: home, pricing, faq, changelog, about.
>
> Home hero: editorial layout, FundedPal wordmark in Fraunces, tagline "Built for prop accounts. Engineered for compliance." in Fraunces 56px on left, single CTA "Start 14-day trial" below body copy on left, bespoke product render on right at -2deg rotation with soft shadow.
>
> Below hero: 3-section feature breakdown (Compliance Guardian, Risk Budget Layer, Hand-verified Compatibility) — alternating left/right, each with real screenshot.
>
> Pull-quote section — single quote, no avatar wall.
>
> Pricing page: three large editorial cards horizontal, Pro card lifted slightly outlined in champagne. Marketing voice from CLAUDE.md — never "trade aggressively", always "stay well inside the rules".
>
> FAQ page renders content from `/content/faq.md` (provided separately in repo).
>
> Changelog uses real Markdown files in `/content/changelog/`.
>
> Build onboarding at `/onboarding`:
> 1. Welcome (Fraunces "Welcome to FundedPal", brief intro)
> 2. Pick prop firms (multi-select 6 supported firms)
> 3. Connect first account or skip
> 4. Choose tier and start trial
> 5. Killzone preferences
>
> Save progress to profiles.onboarded on completion. Single column flow, editorial type, slow staggered reveals.
>
> Verify: Lighthouse 95+ on marketing, full signup → onboarding → dashboard flow works.

**Prompt 38 — Observability and pre-launch hardening**

> (1) Observability: install Sentry on Next.js, Python engine, and bridge. Install PostHog. Define tracking taxonomy in `/lib/analytics.ts` with strongly typed events: account_connected, journey_continued, signal_generated, signal_executed, compliance_rejection (with rule), trial_started, trial_converted, subscription_cancelled, rule_change_detected, etc. Wire critical alerts to Logsnag: account breached, EA bridge disconnected >5min, Stripe webhook failed, rule monitor change detected.
>
> Add status page at `/status` showing real-time bridge connection counts and last-event timestamps.
>
> (2) Security: run audits, fix high-severity issues. Add rate limiting on every public API route via Upstash Redis.
>
> (3) Performance: Suspense boundaries on dashboard data fetches, audit bundle, lazy-load TradingView charts.
>
> (4) Legal: Terms of Service and Privacy Policy pages with disclaimers (automated trading risk, no profit guarantees, regulatory acknowledgements).
>
> (5) Mobile: audit every dashboard route on iPhone SE width (375px). Dashboard is desktop-first but must not break on mobile.
>
> (6) Empty states: every list page (accounts, signals, journal, backtest) needs designed empty state — Fraunces headline, brief instruction, CTA.
>
> (7) 404 and error pages with FundedPal design system.
>
> (8) Run `/docs/AUDIT_CHECKLIST.md` end-to-end. All four sections (MT5 bridge, SMC/ICT engine, rule extraction, Compliance Guardian) must hit green-light criteria.
>
> Verify: Lighthouse on dashboard 90+ all categories, no console errors any page, all flows tested end-to-end. AUDIT_CHECKLIST.md passes.

---

## 6. Definition of Done — v1 Launch Checklist

- [ ] All 38 prompts executed and verified
- [ ] Lighthouse 90+ on dashboard, 95+ on marketing
- [ ] Stripe live mode with all 3 plans configured
- [ ] All 6 supported firms with manually-curated rule profiles
- [ ] EA bridge tested on live MT5 demo for 7 consecutive days without disconnect
- [ ] Compliance Guardian unit tests at 90%+ coverage
- [ ] Account journey detection works end-to-end (Phase 1 → Phase 2 → Funded)
- [ ] Rule monitor running daily, detecting changes correctly
- [ ] Privacy policy, Terms of Service, Risk Disclosure live
- [ ] UK Ltd company set up, ICO registration complete
- [ ] First 25 beta users onboarded with manual support
- [ ] Sentry, PostHog, Logsnag all reporting
- [ ] Marketing site live with full pricing, signup, trial flow
- [ ] AUDIT_CHECKLIST.md fully green

---

*FundedPal — Built for prop accounts. Engineered for compliance.*
