# FundedPal — Claude Design Guide

> How to use Claude Design for FundedPal's visual layer, and how to hand off bundles to Claude Code for implementation.

---

## What this is

FundedPal's frontend is built using the Anthropic-native pipeline: **Claude Design** for prototyping the visual layer, **Claude Code** for production implementation. Claude Design produces interactive HTML prototypes that can be exported as handoff bundles for Claude Code to implement.

This guide is a companion to BUILD_SPEC.md, not a replacement. The build sequence in BUILD_SPEC.md is the authoritative order; this guide explains how Claude Design fits in.

---

## How the two tools relate

```
Claude Design (visual exploration)
    │
    │ Generates interactive HTML prototypes
    │ Refined via chat, inline comments, sliders
    │
    ▼
Handoff bundle exported
    │
    │ Bundle includes HTML/CSS/JS, design intent,
    │ component specs, asset references
    │
    ▼
Claude Code (production implementation)
    │
    │ Reads CLAUDE.md (project rules)
    │ Reads BUILD_SPEC.md (build sequence)
    │ Reads handoff bundle (visual reference)
    │
    ▼
Real Next.js + Supabase + Compliance Guardian etc.
```

The principle: **Claude Design owns the visual decisions, Claude Code owns the structural implementation.** Don't try to make Claude Design produce real React components with state management — it's a prototyper, not a production builder. Don't ask Claude Code to invent visual designs from scratch — use the Claude Design output as the reference.

---

## When to use Claude Design vs Claude Code

| Surface | Claude Design first? | Why |
|---|---|---|
| Marketing site (home, pricing, FAQ, changelog) | Yes | Visual win zone; iteration speed matters |
| Auth and onboarding pages | Yes | First impression; design-led works |
| Dashboard layouts | Yes | Information architecture decisions |
| Settings pages | Yes | Form patterns benefit from rapid iteration |
| Charts and data visualisation | Maybe | Claude Design can mock; Claude Code implements with real libs |
| Backend services (Supabase migrations, Compliance Guardian, MT5 bridge, strategy engine) | No | Pure code, no visual output |
| Component primitives (Button, Input, Card) | Optional | Could be done either way; recommend Claude Design first to lock visual style |
| Email templates | Yes | Design-led |
| Error pages, empty states | Yes | Editorial moments matter |

Rule of thumb: **if a human will look at it, design it in Claude Design first.**

---

## Setup

### Required subscriptions

- Claude Pro, Max, Team, or Enterprise (Claude Design access)
- Claude Code CLI installed locally per Anthropic's current install instructions

### First-time setup in Claude Design

Open Claude.ai → palette icon in left sidebar to access Claude Design. Start a new project named "FundedPal".

### The setup prompt (run once at the start)

This long prompt establishes the design system. Claude Design retains it across the project so subsequent prompts don't need to repeat it.

> I'm building FundedPal, a compliance-first trading platform for prop firm accounts. Tagline: "Built for prop accounts. Engineered for compliance."
>
> I need you to set up the design system before we begin work. The system is "Quiet Authority" — institutional precision, editorial typography, restrained motion. Closer to a Bloomberg terminal redesigned by a Swiss design studio than a retail crypto product.
>
> **Design tokens to apply:**
>
> Typography:
> - Display font: Fraunces (variable serif, weights 300-600) — use for headlines, hero numbers, editorial moments
> - Body font: Switzer (geometric sans-serif from Indian Type Foundry) — body, UI, default text
> - Mono font: JetBrains Mono — all numbers, prices, P&L, technical data
>
> Type scale (rem):
> - display-2xl: 72/76 Fraunces 400, letter-spacing -0.04em
> - display-xl: 56/60 Fraunces 400, letter-spacing -0.035em
> - display-lg: 44/48 Fraunces 500, letter-spacing -0.03em
> - heading-xl: 32/36 Switzer 600, letter-spacing -0.02em
> - heading-lg: 24/30 Switzer 600, letter-spacing -0.015em
> - heading-md: 18/24 Switzer 600, letter-spacing -0.01em
> - body-lg: 17/26 Switzer 400
> - body: 15/22 Switzer 400
> - body-sm: 13/18 Switzer 400
> - caption: 11/14 Switzer 500, letter-spacing 0.06em, uppercase
> - mono-lg: 20/28 JetBrains Mono 500
> - mono: 14/20 JetBrains Mono 400
> - mono-sm: 12/16 JetBrains Mono 400
>
> All numbers use tabular-nums everywhere.
>
> Color tokens (dark theme, default):
> - --ink: #0B0E14 (base background)
> - --ink-lift-1: #13171F (cards, panels)
> - --ink-lift-2: #1B202B (nested cards, modals)
> - --ink-lift-3: #232938 (hover states)
> - --bone: #F0EBE3 (primary text — warm, never pure white)
> - --bone-muted: #8B8F99 (secondary text)
> - --bone-faint: #565A65 (disabled)
> - --border: #2A2F3A (default 1px borders)
> - --border-strong: #3A4050 (emphasised borders)
> - --champagne: #D4B896 (primary brand accent — gold without yellow)
> - --teal: #5EEAD4 (action accent — CTAs, live indicators, focus rings)
> - --profit: #7FB991 (sage green for gains)
> - --loss: #D17B6E (terracotta for losses)
> - --warn: #E8B86E (warm amber for warnings)
>
> Light theme inverts these — warm bone background (#F5F1EA), ink text (#1A1D24), same accent palette adjusted for contrast.
>
> Color usage rules:
> - Champagne is brand identity moments only. Used sparingly. Logo, premium-tier indicators.
> - Teal is the action accent. Most common in the UI.
> - Sage and terracotta are strictly for profit and loss. Never decorative.
> - Never more than two accents on the same screen.
>
> Spacing scale (rem): 0, 0.25, 0.5, 0.75, 1, 1.25, 1.5, 2, 2.5, 3, 4, 5, 6, 8, 12, 16
>
> Container widths: 680, 920, 1180, 1400. Dashboard caps at 1400, marketing heroes use 1180.
>
> Border radius: 0, 2, 4, 6, 10, 16, 24. Cards default 10. Buttons default 6. Pills 9999.
>
> Motion:
> - Custom easing: cubic-bezier(0.16, 1, 0.3, 1) — feels deliberate
> - Hover/focus: 120ms
> - Page transitions: 400ms
> - Modal: 260ms
> - Number tick: 600ms with easeOutQuart
> - Entrance reveals: 800ms with 60ms staggered children, once per page
> - Always respect prefers-reduced-motion: reduce
>
> The aesthetic is restrained, editorial, and precise. No bouncing, no jiggle, no parallax. No glassmorphism. No purple gradients. No generic shadcn aesthetics. No neon green/red — sage and terracotta only.
>
> Save this as my design system for the FundedPal project. Apply it to every screen we generate.

After the setup prompt, every subsequent prompt automatically uses these tokens.

---

## Build sequence in Claude Design

The recommended order — build marketing first (visual win zone), then dashboards, then specialised surfaces.

### Stage 1 — Marketing surfaces

Map to BUILD_SPEC.md Prompt 37 for implementation.

- Homepage hero
- Feature breakdown sections (Compliance Guardian, Risk Budget Layer, Hand-verified Compatibility)
- Pricing page
- Public firm directory
- FAQ page
- Changelog page
- Global navigation and footer

### Stage 2 — Auth and onboarding

Maps to BUILD_SPEC.md Prompts 6 and 37.

- Sign-in page
- Sign-up page
- 5-step onboarding flow

### Stage 3 — Core dashboard

Maps to BUILD_SPEC.md Prompts 7 and 16.

- Application shell (sidebar, top bar)
- Multi-account dashboard
- Single-account dashboard with journey timeline

### Stage 4 — Specialised dashboards

Maps to BUILD_SPEC.md Prompts 11, 24, 31, 32.

- Logged-in firm directory with scan-but-filter UX
- Strategies / signals page
- Backtest configuration and results
- Journal with AI co-pilot sidebar

### Stage 5 — Settings, billing, admin

Maps to BUILD_SPEC.md Prompts 18 (risk settings), 30 (killzones), 35 (billing), 35.5 (referrals), 33 (admin rule changes).

- Settings (Profile, Risk, Killzones, Notifications, Connections, Security)
- Risk budget settings page
- Connect-account flow with journey detection
- Billing
- Referrals (user-facing share dashboard)
- Admin rule change review (admin role only)

---

## Specific prompts per surface

For each major surface, here's the Claude Design prompt to use. After the setup prompt establishes the design system, each prompt below produces one screen.

### Marketing — homepage hero

> Design the FundedPal homepage hero section. Editorial magazine layout, asymmetric, generous whitespace.
>
> Layout: Two-column desktop, single-column mobile. Left column 60% width, right column 40%.
>
> Left column content:
> - Tiny caption uppercase Switzer at top: "FOR PROP TRADERS"
> - Hero headline in Fraunces display-xl: "Built for prop accounts. Engineered for compliance."
> - Subhead in Switzer body-lg, max 600px wide: "FundedPal is the trading platform that enforces your prop firm's rules at the order layer — so you stay funded, not banned."
> - Primary CTA button in champagne: "Start 14-day trial"
> - Secondary text below CTA in body-sm bone-muted: "Card required. Cancel any time."
>
> Right column content:
> - Bespoke render of the FundedPal account dashboard, framed at -2 degree rotation with a soft shadow. Show a $100k account with sage profit indicator, an SMC/ICT signal card, and the kill zone clock.
>
> Below the hero: a subtle scroll indicator — small caret, no text, animates downward at 2-second intervals.
>
> Use the FundedPal design system. Dark theme by default with theme toggle in the top right of the navigation.

### Marketing — feature breakdown

> Design three feature breakdown sections for the FundedPal homepage. Each section is full-width with a 1180px content container, alternating left/right layout. 96px vertical spacing between sections on desktop, 64px on mobile.
>
> Section 1 — "The Compliance Guardian":
> - Visual on left, text on right
> - Visual: animated mockup showing an order being evaluated against rules, with each rule lighting up in sage as it passes, then a final "approved" state
> - Heading in Fraunces display-lg: "The trade your prop firm rejects, never gets placed."
> - Body: "Every order runs through the Compliance Guardian before it reaches your broker. Daily loss, drawdown, consistency, news blackouts, lot caps — checked and enforced in real time. If a trade would breach a rule, FundedPal physically cannot place it."
> - List of rules checked: "Daily loss limit · Maximum drawdown · Lot size caps · News restrictions · Consistency rule · Stop loss requirement · Weekend holding · EA permissions"
>
> Section 2 — "Risk Budget Layer" (text on left, visual on right):
> - Visual: two-zone bar showing sage zone (safe) → amber zone (caution) → terracotta line (firm hard limit) with a marker showing "your daily usage"
> - Heading: "Stay well inside the rules. Not skating along them."
> - Body: "Your prop firm allows 5% daily loss. FundedPal stops trading at 3.5%. The 1.5% gap is your buffer for slippage, spread, and the news event nobody saw coming."
>
> Section 3 — "Hand-verified Compatibility" (visual on left, text on right):
> - Visual: animation of a prop firm T&Cs page being scanned, with rules being extracted and populated into a structured form
> - Heading: "Six prop firms. Hand-verified. Continuously monitored."
> - Body: "Every supported firm has been read end-to-end and rule-mapped. A daily monitor watches each firm's published rules for changes — when something shifts, you'll know before your next trade."

### Marketing — pricing

> Design the FundedPal pricing page. Three tiers in a horizontal layout, with the middle tier (Pro) lifted slightly and outlined in champagne.
>
> Page hero: Fraunces display-xl headline "Pricing built for survival, not for growth hacks."
>
> Subhead in Switzer body-lg: "14-day trial on every tier. Card required. Cancel any time."
>
> Three pricing cards:
>
> Card 1 — Trader ($39/mo):
> - Tier name in Switzer caption uppercase at top
> - Price in Fraunces display-xl: $39
> - Period in body-sm: per month
> - Description: "Full Compliance Guardian. SMC/ICT engine. One journey."
> - Feature list (8 items): Compliance Guardian · Risk Budget Layer · Auto-execute mode · 1 active journey · SMC/ICT signal generation · Killzone scheduling · Basic backtest · Email support
> - CTA: "Start 14-day trial"
>
> Card 2 — Pro ($69/mo) — RECOMMENDED:
> - Lifted 8px above the others, 1px champagne border
> - Small badge above card: "RECOMMENDED" in caption uppercase, champagne
> - Description: "Multi-account coordination. Advanced analytics. Up to 3 journeys."
> - Feature list: Everything in Trader + Multi-Account Risk Allocator · Drawdown Forecaster · Advanced backtest · Payout Optimizer · Journal AI Co-pilot · Custom kill zones · 3 active journeys · Priority email support
> - CTA: "Start 14-day trial"
>
> Card 3 — Elite ($199/mo):
> - Description: "Unlimited journeys. API access. White-glove onboarding."
> - Feature list: Everything in Pro + Unlimited active journeys · REST API + webhooks · Custom strategy plug-ins · White-glove onboarding call · Priority execution queue · Direct support line
> - CTA: "Start 14-day trial"
>
> Below cards: a subtle FAQ teaser linking to /faq with the editorial line "More questions about why we're intentionally restrictive?"

### Marketing — public firm directory

> Design the public-facing firm directory at /firms.
>
> Page hero: Fraunces display-xl "Six firms. Hand-verified. Continuously monitored."
>
> Subhead: "Each firm in our directory has been read end-to-end and tested for compatibility with FundedPal's compliance-first model."
>
> Below hero: filter chips in Switzer caption uppercase: "All firms" / "Most popular" / "Best for beginners" / "Highest profit splits"
>
> Firm grid: 3 columns desktop, 1 column mobile. Each firm card:
> - Firm logo in top-left, height 32px
> - Firm name in Fraunces heading-lg
> - Health score as a Fraunces 56px number with sage-to-terracotta color fade based on score (e.g. 8.4)
> - Three pill stats: profit split, max funding, payout speed
> - One-line summary in body-sm bone-muted
> - "View rules" CTA at bottom in ghost button style
>
> Below grid: a subtle changelog teaser — "12 rule changes detected this quarter" with link to /changelog/firms.

### Auth — sign in / sign up

> Design FundedPal sign-in and sign-up pages. Editorial layout, single column, centered, max-width 480px.
>
> Sign-in page:
> - Page title in Fraunces display-lg: "Welcome back."
> - Subhead in body-sm: "Sign in to your FundedPal account."
> - Email input — bottom-border-only style, full border on focus
> - Password input with show/hide toggle
> - "Sign in" CTA in champagne primary, full width
> - Divider line with caption "or"
> - "Continue with Google" button in ghost style
> - "Sign in with magic link" text link below
> - Footer: "New to FundedPal? Create an account." with link
>
> Sign-up page:
> - Title: "Welcome to FundedPal."
> - Subhead: "Start your 14-day trial. Card required."
> - Full name, email, password fields (same input style)
> - Tier selector: three radio cards (Trader, Pro recommended, Elite) — same as pricing page but condensed
> - Stripe-style card input
> - "Start 14-day trial" CTA
> - Tiny disclaimer: "By creating an account, you agree to our Terms and Privacy. We charge nothing during the trial."
> - Footer: "Already have an account? Sign in."

### Onboarding — 5 steps

> Design a 5-step onboarding flow at /onboarding. Each step is a full-screen experience with editorial whitespace, single Fraunces hero element, and a minimal progress indicator at the top.
>
> Step 1 — Welcome:
> - Centered Fraunces display-2xl: "Welcome, [name]."
> - Subhead in body-lg: "FundedPal is set up to protect your account before it grows it. Three minutes to get you trading."
> - "Continue" button in champagne primary
>
> Step 2 — Pick your prop firms:
> - Title: "Which prop firms do you trade with?"
> - Multi-select grid of 6 supported firm logos, generous spacing, click to toggle
> - Toggle state: 1px champagne border around selected firm
> - "Continue" button enabled when at least 1 selected
>
> Step 3 — Connect your first account (optional):
> - Title: "Connect your first prop firm account."
> - Body: "Or skip this and connect later from the dashboard."
> - Visual flow showing 3 steps: Select firm → Choose phase → Enter MT5 credentials
> - "Connect now" CTA + "Skip for now" text link
>
> Step 4 — Risk preferences:
> - Title: "Set your risk preferences."
> - Three sliders for default settings:
>   - Per-trade risk: 0.10% to 1.00% (default 0.50%)
>   - Daily soft cap: shown as % of firm hard rule (default 70%)
>   - Max trades per day: 1 to 5 (default 3)
> - Live calculator below sliders showing dollar values on a hypothetical $100k account
> - Note: "These can be adjusted later. We've set conservative defaults to keep your account safe."
> - "Continue" button
>
> Step 5 — Killzone preferences:
> - Title: "When do you want FundedPal to trade?"
> - Visual circular 24-hour clock showing four kill zones (Asian, London, NY AM, NY PM)
> - Toggle each zone on/off
> - "Finish setup" CTA

### Dashboard — multi-account

> Design the FundedPal multi-account dashboard at /dashboard.
>
> Layout:
> - 240px fixed left sidebar (navigation)
> - Top bar with breadcrumb, theme toggle, user menu
> - Main content area, max-width 1400px
>
> Sidebar:
> - FundedPal wordmark in Fraunces at top
> - Nav items: Dashboard (active), Accounts, Strategies, Backtest, Firms, Journal, Billing, Settings
> - Each item: icon (Lucide, 20px) + label in Switzer body, 12px vertical padding
> - Active state: 2px champagne left border, --bone text, --ink-lift-1 background
> - Bottom of sidebar: connection status dot (sage when bridge connected) + small text
>
> Main content:
> - Hero section: Fraunces display-2xl showing total portfolio equity in JetBrains Mono ($247,832.50), with day's P&L below in mono (sage if positive, terracotta if negative, with explicit + or − sign)
> - Below hero: 12-column grid
>
> Grid contents:
> - Equity curve chart (full width, 4 columns x 2 rows tall) — showing all-account equity over last 30 days
> - Account cards (3 cards, 4 columns each, 1 row tall) — one per active journey, showing: firm logo, nickname, current equity, today's P&L, status indicator, rule usage progress bars, current phase badge
> - Active signals card (4 columns x 1 row) — showing latest 3 signals with confluence scores
> - Compliance decisions feed (4 columns x 1 row) — last 5 compliance decisions, approved/rejected status
> - Killzone clock (4 columns x 1 row) — circular 24-hour indicator
>
> All numbers in JetBrains Mono with tabular-nums. Real-time tick animations on equity changes (600ms easeOutQuart).

### Dashboard — single account with journey timeline

> Design the single-account dashboard at /accounts/[id].
>
> Top section — editorial hero:
> - Account nickname + firm name + phase as breadcrumb in body-sm bone-muted
> - Big editorial moment: account equity in Fraunces display-2xl, JetBrains Mono digits, e.g. "$104,832.50"
> - Day's P&L in mono-lg below, with sign and color
> - Three quick stats inline: Days trading · Best day · Win rate
> - Phase indicator pill: "Phase 1 · 8% target · 5% daily" in caption, champagne border
>
> Below hero: **Journey timeline section** (subtle, between hero and grid):
> - Horizontal progression bar showing journey history (e.g. Phase 1 → Phase 2 → Funded)
> - Current phase highlighted in champagne
> - Click expands to show full transition history with dates and balances
> - Editorial caption above: "Your FTMO journey · Started Sept 14, 2026"
>
> Below — 12-column grid:
>
> Row 1 — Equity curve:
> - Full-width chart showing equity over the trading days of this account
> - Sage profit zone, terracotta drawdown zone
> - Subtle markers for trades taken
>
> Row 2 — Rules usage (8 columns) + Killzone clock (4 columns):
> - Rules card shows every active rule with a RuleProgressBar:
>   - Daily Loss: usage in mono "1.2 / 5.0%" with progress bar (sage → amber → terracotta)
>   - Max Drawdown: usage with bar
>   - Per-Trade Risk: usage with bar
>   - Daily Soft Cap: usage with bar
>   - Trades today: counter
>   - Server requests today: counter (FTMO)
>
> Row 3 — Open positions (full width):
> - Tabular layout, no row borders, hover lifts row
> - Columns: Symbol, Side, Lots, Entry, SL, TP, P&L, Time
> - All numbers in mono with tabular-nums
>
> Row 4 — Recent trades (8 columns) + Recent compliance decisions (4 columns):
> - Trades: similar to open positions but closed
> - Compliance decisions: list with approved (sage check) or rejected (terracotta x) icon, time, rule that triggered

### Connect account flow with journey detection

> Design the 3-step account connection flow at /accounts/connect, with journey detection between steps 2 and 3.
>
> Layout: editorial single-column, max-width 680px, generous vertical spacing.
>
> Step 1 — Select firm:
> - Title in Fraunces display-lg: "Choose your prop firm."
> - Searchable list of 6 supported firms, each with logo, name, and a short tagline
> - Select state: 1px champagne border on selected firm
> - Below list: a subtle "Don't see your firm?" link opening a drawer that explains the curation policy
>
> Step 2 — Choose phase and account size:
> - Title: "Tell us about your account."
> - Account size selector: pills for $10k, $25k, $50k, $100k, $200k
> - Phase selector: Phase 1, Phase 2, Funded, Instant Funded
> - For The Funded Trader and E8 Markets, account_type sub-selector
> - Below: dynamically populated rule preview card showing the rules for the selected combination
>
> **Journey detection card** (appears after Step 2 if user has existing accounts at the same firm with recent phase progression):
> - Editorial card with subtle teal left border
> - Caption "ACCOUNT JOURNEY DETECTED" in Switzer caption uppercase
> - Body: "Looks like this might be your new account from passing [previous phase] on your existing FTMO journey. Should we continue your existing journey on this new login?"
> - Below body: a small visualisation showing the journey path (Phase 1 ✓ → Phase 2 [you are here])
> - Two buttons: "Yes, continue journey" (champagne) · "No, this is a new account" (ghost)
> - Helper text: "Continuing a journey doesn't count as a new connection toward your tier limit."
>
> Step 3 — Enter MT5 credentials:
> - Title: "Connect to MetaTrader 5."
> - Subtle caption above form in Switzer caption uppercase bone-muted: "METATRADER 5 ONLY · OTHER PLATFORMS COMING IN FUTURE RELEASES"
> - Form fields: nickname (optional), MT5 login (numeric), MT5 password, MT5 server
> - Helper text under server field: "Find this in your prop firm welcome email"
> - Visual diagram showing how FundedPal connects (small illustration)
> - Below form: compatibility summary panel showing what FundedPal will and won't do on this firm and account type
> - "Connect account" CTA in champagne
>
> After submission: a "Waiting for bridge connection" screen with animated indicator and instructions for installing the FundedPal MT5 EA on their MetaTrader 5 platform.

### Strategies — signals feed

> Design /strategies — the live signal feed where traders review SMC/ICT signals.
>
> Page hero in Fraunces display-lg: "Signals."
> Subhead body-sm: "Live SMC/ICT signals across your connected accounts. Each one passes the Compliance Guardian before reaching you."
>
> Filter bar: account filter, symbol filter, confluence threshold slider.
>
> Signal feed (single column):
> - Each signal is a card, full-width
> - Top of card: symbol + side (BUY/SELL) + confluence score visual bar (0-100, sage gradient)
> - Big numbers in mono-lg: Entry · Stop Loss · Take Profit · Risk:Reward
> - SMC/ICT reasoning in structured list:
>   - "HTF bias: Bullish (Daily structure)"
>   - "POI: Order block at 1.0850, mitigated 0% (untouched)"
>   - "Liquidity sweep: equal lows at 1.0832 swept"
>   - "Killzone: London Open"
>   - "OTE: 79% retracement reached"
> - Pre-trade simulation panel showing soft caps and hard rule evaluations
> - Two CTAs: "One-Tap Execute" (champagne, only enabled if compliance approves) · "Dismiss" (ghost)
>
> New signals slide in from top with 400ms cubic-bezier(0.16, 1, 0.3, 1). High-confluence signals (≥85) get a subtle teal pulse.

### Journal with AI co-pilot

> Design /journal — the editorial trading journal.
>
> Layout: two-pane. Left pane is the journal timeline (60% width). Right pane is the AI Co-pilot chat (40% width).
>
> Timeline:
> - Editorial design — Fraunces date headers (e.g. "Tuesday, October 21" in display-lg)
> - Each entry: type icon, title, body, optional embedded chart for trade entries
> - Auto-trade entries show: SMC/ICT reasoning, market context, outcome, P&L
> - Manual notes show: title, body
> - AI reflections show: prompt and analysis
> - Compliance rejection entries show: rejection reason, suggested adjustment, link back to signal
> - Entries fade in on scroll
>
> AI Co-pilot pane:
> - Header: "Ask about your trading" in heading-md
> - Suggested questions as chips: "Why did I take that EUR/USD short on Tuesday?" · "What's my best-performing setup?" · "Am I trading too aggressively?"
> - Message thread below
> - Input at bottom with send button
> - Responses cite specific journal entries with click-to-jump links

### Backtest

> Design /backtest — the strategy backtesting page.
>
> Two views: Configuration (initial) and Results (after run completes).
>
> Configuration view:
> - Page hero: "Backtest." in Fraunces display-lg
> - Form layout, single column max 680px:
>   - Firm selector (6 supported)
>   - Account size + phase
>   - Symbols multi-select
>   - Timeframe (1h, 4h, daily)
>   - Date range (defaults to last 3 months)
>   - Risk profile: "FundedPal default" / "Custom" — opens advanced sliders
> - "Run backtest" CTA in champagne
> - Below form: "What we'll simulate" explanation card listing every rule that will be enforced during the simulation
>
> Results view (after run completes):
> - Verdict hero: large Fraunces heading "PASSED Phase 1 in 14 days" or "FAILED — daily loss breach on day 8"
> - Verdict reasoning in body-sm
> - Equity curve chart with annotations on key trades and rule events
> - Stats grid: Win rate · Profit factor · Avg R · Total trades · Max drawdown · Days to target · Compliance rejections
> - Rejection breakdown: bar chart showing which rules caused rejections most often
> - "Run another" CTA

### Risk budget settings

> Design /accounts/[id]/risk — the risk budget settings page.
>
> Page hero: "Risk Budget" in Fraunces display-lg, account nickname subtitle in Switzer.
>
> Three setting controls in a single column max 680px:
>
> Per-trade risk slider:
> - Range 0.10% to 1.00% (hardcoded ceiling)
> - 0.05% steps
> - Current value displayed in mono next to slider
>
> Daily soft cap slider:
> - Range 10% to 75% of hard rule (dynamic ceiling based on connected firm)
> - 5% steps
> - Current value in mono
>
> Max trades per day stepper:
> - 1 to 10
>
> Live calculator card showing current settings translated into dollars on this account: "0.5% per trade = $500 max loss per trade. 1.4% daily = $1,400 max daily loss."
>
> Two-zone visualisation bar:
> - Sage zone (0 to soft cap)
> - Amber zone (soft cap to hard rule)
> - Terracotta line at hard rule
> - Animated marker showing current usage
>
> Audit log table: last 10 changes from risk_setting_changes, showing field, before, after, when.
>
> Explanatory copy: "These limits exist because most prop accounts blow not from the strategy being wrong, but from a single oversized trade. The bounds above cannot be raised — that's intentional."

### Referrals page

> Design /referrals — the user-facing referral dashboard where traders find their share link, track conversions, and see credits earned.
>
> This is a community-growth surface, not a marketing pitch. The user already pays for FundedPal — they're here to share it with others, not to be sold on it. Tone is utility-first with subtle warmth, not aggressive growth-hacking.
>
> Layout: editorial single-column, max-width 920px (slightly wider than dashboards because of the stats cards).
>
> **Hero section:**
> - Fraunces display-lg: "Your referrals."
> - Subhead in body-lg, max 600px: "Help fellow prop traders survive their challenges. Get paid for it."
> - No CTA in the hero — the action is below.
>
> **Section 1: Your code and link**
> Card with --ink-lift-1 background, generous padding (32px desktop).
> - Caption uppercase Switzer at top: "YOUR REFERRAL CODE"
> - Code displayed in JetBrains Mono mono-lg (e.g. "tokunbo-7k2x"), with copy-to-clipboard button to its right
> - Below code: full share link in body-sm bone-muted, also with copy button (e.g. "fundedpal.com/?ref=tokunbo-7k2x")
> - Below link: row of share buttons — X (Twitter), WhatsApp, Telegram, Discord, generic copy. Use Lucide icons at 20px in --bone-muted, hover lifts to --bone.
>
> **Section 2: Share message picker**
> Card showing 4 message templates the user can pick from:
> - Three pre-written variants (radio-card style — clicking selects that one)
> - One "Custom" option that expands to a textarea where they can write their own
> - Each variant shows the actual message in body in --bone, with subtle 1px border in --border around each card
> - Selected variant gets 1px champagne border
> - "Copy with link" button below — copies the selected message with the link substituted in
>
> **Section 3: Your stats**
> Two side-by-side cards (4 columns each in 8-column inner grid):
>
> *Rolling 12 months card:*
> - Caption: "ROLLING 12 MONTHS"
> - Big editorial number in Fraunces display-lg JetBrains Mono: total credits earned in dollars (e.g. "$117")
> - Below: 4 stat rows
>   - Signups: count
>   - Trial conversions: count
>   - Paid conversions: count
>   - Cap usage: "3 of 6" — small horizontal progress bar showing usage toward 6-referral cap
> - When cap is reached: subtle banner "You've maxed out for this year. Your link still works — friends still get 50% off."
>
> *Lifetime card:*
> - Caption: "LIFETIME"
> - Big editorial number: total credits earned all-time
> - Below: 3 stat rows (signups, trial conversions, paid conversions)
>
> **Section 4: Recent referrals**
> Editorial table, last 10 referrals.
> - Borderless rows, hover lifts row, columns:
>   - Referred user — first name + last initial only (privacy)
>   - Status — badge: "Trial active" / "Paid" / "Refunded" / "Cap exceeded" — sage/teal/terracotta/bone-muted respectively
>   - Date — body-sm bone-muted
>   - Credit earned — JetBrains Mono in sage if positive, bone-muted if zero
> - If empty: editorial empty state with Fraunces heading-lg "No referrals yet." and brief instructional body.
>
> **Section 5: How it works**
> Card with subtle --ink-lift-1 background. Editorial breakdown:
> - Fraunces heading-md: "How FundedPal referrals work"
> - 4-step explanation in body, no bullet points — written as flowing prose paragraphs
> - Step 1: Share your link with prop traders you know
> - Step 2: When they sign up and complete their first paid month, you both win
> - Step 3: They get 50% off their first paid month. You get a credit equal to your subscription tier.
> - Step 4: Refer up to 6 paid users in a 12-month rolling window. Past that, your link still works — your friends still get 50% off — but new referrals stop earning credits until older ones age out.
> - Below: small print in body-sm bone-muted noting the 30-day clawback rule and fraud detection.
>
> Use the FundedPal design system. Generous whitespace between sections (64px desktop, 48px mobile).
>
> **Important tone note:** This page is not a marketing pitch. The user is already paying for FundedPal. The page should feel like a useful internal dashboard, not a "refer friends, get rewards!" promotional page. Avoid growth-hack vocabulary. Use the same restrained editorial voice as the rest of the app.

### Referral banner component (dashboard)

> Design `<ReferralBanner />` — a discreet banner shown on the multi-account dashboard for the first 30 days post-signup, encouraging users to share their referral link.
>
> Layout: thin horizontal banner, full width of dashboard content area, sits below the dashboard hero and above the 12-column grid.
>
> Visual:
> - 1px subtle dashed border in --champagne-dim
> - --ink-lift-1 background
> - Padding 16px vertical, 24px horizontal
> - Subtle Lucide "share" icon at 18px in --champagne to the left
> - Body text in body-sm: "Tell other prop traders. Earn a free month for each one who joins."
> - Right side: "Get your link" ghost button linking to /referrals
> - Far right: small dismiss button (X icon, 16px, bone-muted, hover bone)
>
> Behaviour:
> - Shown only if profile.created_at < 30 days ago AND user has not dismissed it
> - Dismissal stored in localStorage AND profiles.referral_banner_dismissed_at
> - Once dismissed, never re-appears
> - Animates in with subtle fade + 4px lift (200ms cubic-bezier(0.16, 1, 0.3, 1))
>
> No urgency, no growth-hack pressure. The component should feel like a quiet nudge, not a popup demanding attention.

### Admin — rule changes review

> Design /firms/admin/changes — the admin rule change review panel (visible only to users with admin role).
>
> Page hero: "Rule changes pending review." in Fraunces display-lg.
>
> Below hero: editorial table of unreviewed firm_rule_snapshots, newest first.
> - Columns: Firm (with logo), Source URL, Detected at, Age (e.g. "2 days ago")
> - Click a row to navigate to detail page
>
> Detail page /firms/admin/changes/[id]:
> - Firm name + source URL at top
> - Side-by-side diff: old content (left, dimmed bone-muted) and new content (right, highlighted)
> - Use a code-style diff viewer with line numbers
> - Below diff: review_notes textarea
> - Four action buttons: "No change (false positive)" / "Minor update" / "Significant change" / "Breaking change"
> - Each button has a tooltip explaining the consequence

---

## Refinement prompts

---

## Workflow: how prompts map to conversations

A common point of confusion: should each prompt be its own Claude Design conversation, or should multiple be combined?

**Default rule: one prompt = one conversation = one surface.**

Don't combine unrelated surfaces in a single conversation. Claude Design works best when it can focus on one design context at a time. Combining a homepage hero with a settings page in the same conversation produces worse output for both.

### When to combine prompts in the same conversation

Only combine surfaces that are genuinely a unit:

- **Sign in + sign up** — paired in user flow, similar layout
- **Onboarding steps 1-5** — one continuous flow
- **Settings sub-sections** — shared sidebar, same layout pattern
- **Connect-account flow steps** — one continuous wizard
- **Feature breakdown sections** — stylistically a unit, three sections of same page

Everything else gets its own conversation.

### Recommended order

You'll have roughly 15-18 conversations across the project. Run them in this order:

**Stage 1 — Foundation (1 conversation)**
1. Design system setup (the long setup prompt establishes everything)

**Stage 2 — Marketing visual identity (5-6 conversations)**
2. Homepage hero
3. Feature breakdown sections
4. Pricing page
5. Public firm directory
6. Marketing consistency pass (review all marketing as a set)

**Stage 3 — Auth and onboarding (2 conversations)**
7. Sign in / sign up
8. Onboarding (5 steps)

**Stage 4 — Core dashboard (3 conversations)**
9. Application shell (sidebar + topbar)
10. Multi-account dashboard
11. Single-account dashboard with journey timeline

**Stage 5 — Trading surfaces (3 conversations)**
12. Connect-account flow
13. Strategies / signals feed
14. Journal with AI co-pilot

**Stage 6 — Power features (2 conversations)**
15. Backtest
16. Logged-in firm directory

**Stage 7 — Settings, billing, referrals, admin (5 conversations)**
17. Settings (all sub-sections)
18. Risk budget settings
19. Billing
20. Referrals (page + banner component)
21. Admin rule changes

### Iteration within a conversation

Once you start a conversation, the full prompt from this guide is your *starting* prompt. Refinement happens through Claude Design's native tools:

1. Paste the surface prompt → Claude Design generates first version
2. Use **chat** for broad changes ("make the headline larger")
3. Use **inline comments** to target specific elements ("this button feels too small")
4. Use **adjustment sliders** Claude Design auto-generates to tune spacing/color/sizing live
5. Use the **refinement prompts** below for common patterns

Don't re-paste the original prompt. Iterate within the conversation.

---

## Refinement prompts

After Claude Design generates a first version of any screen, refine with these patterns:

**Improve typography:**
> The headline feels generic. Tighten letter-spacing to -0.04em and consider Fraunces 500. Increase the visual weight relative to the body copy below.

**Add editorial moment:**
> This page needs one editorial moment. Pick the most important number or stat and render it in Fraunces display-2xl with tabular-nums, taking up significant visual space. The rest of the page should orbit around it.

**Fix spacing:**
> The vertical rhythm feels cramped. Increase section spacing to 96px on desktop, 64px on mobile. Add more breathing room between the header and the first content block.

**Polish motion:**
> Add subtle entrance animations using cubic-bezier(0.16, 1, 0.3, 1) easing. Hero element fades + lifts 8px, child elements stagger at 60ms intervals. One staggered reveal per page only — not on every element. Always respect prefers-reduced-motion: reduce.

**Mobile pass:**
> Audit this design at 375px width (iPhone SE). Fix anything that breaks: text overflow, cramped layouts, touch targets below 44px, horizontal scroll.

**Theme parity:**
> Show me this design in both dark and light themes. The light theme should be warm bone-on-ink (#F5F1EA background, #1A1D24 text), not generic white. Verify contrast ratios meet WCAG AA in both.

**Empty state:**
> This page has no designed empty state. Add one: Fraunces heading-lg, brief instructional body in body-sm, single CTA. Make it feel intentional, not like a placeholder.

---

## Handoff to Claude Code (standalone HTML workflow)

FundedPal exports Claude Design output as **standalone HTML files**, one per surface. Drop them in `/handoffs/` at the repo root.

### Folder structure

```
fundedpal/
└── /handoffs
    ├── README.md                          # bundle index (see below)
    ├── _design-system.html                # foundation — export first
    ├── home-hero.html
    ├── feature-breakdown.html
    ├── pricing.html
    ├── firm-directory-public.html
    ├── sign-in-up.html
    ├── onboarding.html
    ├── application-shell.html
    ├── multi-account-dashboard.html
    ├── single-account-dashboard.html
    ├── connect-account.html
    ├── strategies-feed.html
    ├── journal.html
    ├── backtest.html
    ├── firm-directory-app.html
    ├── settings.html
    ├── risk-budget.html
    ├── billing.html
    ├── referrals.html
    └── admin-rule-changes.html
```

The underscore prefix on `_design-system.html` sorts it to the top — Claude Code finds it as the foundational reference before any per-surface file.

Use kebab-case filenames that match the surface purpose, not the route. So `home-hero.html` not `index.html`, `connect-account.html` not `accounts-connect.html`. Routes might change during implementation; surface purposes stay stable.

### Add a header comment to each file

When you export from Claude Design, add this header at the top before saving:

```html
<!--
  FundedPal — [Surface name]
  Generated in Claude Design, [date]
  Build prompt: BUILD_SPEC.md Prompt [N]
  Status: approved | needs refinement | placeholder
-->
<!DOCTYPE html>
<html>
...
```

Five seconds per file. Massively useful when you have 18 files and need to remember which is which six weeks from now.

### Maintain a /handoffs/README.md index

Drop this in `/handoffs/README.md` so Claude Code (and future you) can find which file maps to which build prompt:

```markdown
# FundedPal Handoff HTML

Visual designs from Claude Design, used as reference by Claude Code during implementation.

## Foundation

| File | Purpose | Build prompt |
|---|---|---|
| _design-system.html | All tokens, primitives, motion | Prompts 3-4 |

## Marketing

| File | Surface | Build prompt |
|---|---|---|
| home-hero.html | Marketing homepage hero | Prompt 37 |
| feature-breakdown.html | Marketing feature sections | Prompt 37 |
| pricing.html | Marketing pricing page | Prompt 37 |
| firm-directory-public.html | Public /firms | Prompt 11 |

## Auth and onboarding

| File | Surface | Build prompt |
|---|---|---|
| sign-in-up.html | Auth pages | Prompt 6 |
| onboarding.html | 5-step onboarding flow | Prompt 37 |

## Dashboard

| File | Surface | Build prompt |
|---|---|---|
| application-shell.html | Sidebar + topbar | Prompt 7 |
| multi-account-dashboard.html | /dashboard | Prompt 16 |
| single-account-dashboard.html | /accounts/[id] | Prompt 16 |
| connect-account.html | /accounts/connect | Prompt 13 |

## Trading and journal

| File | Surface | Build prompt |
|---|---|---|
| strategies-feed.html | /strategies | Prompt 24 |
| journal.html | /journal | Prompt 32 |
| backtest.html | /backtest | Prompt 31 |
| firm-directory-app.html | /firms (logged in) | Prompt 11 |

## Settings, billing, referrals, admin

| File | Surface | Build prompt |
|---|---|---|
| settings.html | /settings | Prompt 18, 30 |
| risk-budget.html | /accounts/[id]/risk | Prompt 18 |
| billing.html | /billing | Prompt 35 |
| referrals.html | /referrals (page + banner component) | Prompt 35.5 |
| admin-rule-changes.html | /firms/admin/changes | Prompt 33 |
```

### How Claude Code consumes the HTML

When you reach a UI prompt in BUILD_SPEC.md, reference both the design system file and the surface file:

> Build the FundedPal homepage hero per BUILD_SPEC.md Prompt 37, using `/handoffs/home-hero.html` as the visual reference. Also reference `/handoffs/_design-system.html` for the canonical token values and component patterns.
>
> The HTML files are the authoritative source for visual design. Match their appearance closely, but translate appropriately:
>
> 1. Inline `<style>` blocks → Tailwind utility classes referencing CSS variables from `globals.css`
> 2. Inline base64 images → extracted assets in `/public/handoff-assets/[surface-name]/`
> 3. Inline SVG icons → use Lucide where equivalent exists, otherwise add to `/components/icons/`
> 4. Inline JavaScript (interactivity demos) → translate to React state and event handlers
> 5. Static markup with sample data → real components consuming Supabase data via Server Components or Server Actions
>
> The HTML shows what; you produce a React implementation that does the same thing using the project's actual stack. Use the components already built in `/components/ui/` where applicable.

This translation block is worth keeping as a snippet you paste before any UI build prompt. Saves repetition and ensures Claude Code translates deliberately rather than copy-pasting raw HTML.

### What to know about standalone HTML exports

A few practical notes about the format:

**Inline styles.** Standalone HTML usually has all CSS inline in a `<style>` tag at the top. Claude Code extracts these and translates to Tailwind classes referencing your CSS variables.

**Inline base64 assets.** Images and complex SVGs may be embedded as base64 data URIs. Tell Claude Code to extract these to `/public/handoff-assets/[surface-name]/` rather than leaving them inline.

**Self-contained dependencies.** Standalone HTML doesn't pull in fonts from Google Fonts or icons from Lucide automatically. Claude Code reconciles what's in the HTML against your `next/font` setup and Lucide imports.

**Sample data.** The HTML will contain placeholder data (e.g. "$104,832.50" as the equity number). Claude Code replaces these with real bindings to Supabase queries.

These translations are easy and well-understood — just flag them explicitly so Claude Code does them deliberately.

### Commit the handoffs folder to git

Yes, commit it. Two reasons:

**Reproducibility.** If you (or anyone else) needs to rebuild a surface six months from now, the handoff HTML is the design source of truth. It needs to be in version control alongside the code.

**Auditability.** When someone asks "why does the dashboard look this way?", being able to point to a specific HTML file is much cleaner than "we designed it in Claude Design back in May."

Add a `.gitignore` exception for any large temp files Claude Design might dump. Keep the HTML files themselves in git.

---

## Tips for working with Claude Design

**1. Generate broadly, refine narrowly.** First pass: get a working version. Refinement passes: target specific elements with inline comments rather than rewriting prompts.

**2. Use the web capture tool.** If a screen on a competing platform looks great, capture it and ask Claude Design to remix or evolve it. Useful for the firm directory page (look at how Topstep displays prop firm cards) or for the dashboard (look at Bloomberg-style information density).

**3. Save drafts before pivoting.** Tell Claude Design "save what we have and try a completely different approach" before exploring alternatives. You may want to come back.

**4. The handoff HTML is the goal.** Every design should eventually export as standalone HTML for Claude Code. Designs that are just visual demos won't help the implementation phase.

**5. Don't rebuild components for each surface.** Once Claude Design has a Button or Input it's happy with, ask it to use the same component on subsequent screens. This keeps the design system consistent.

**6. Validate the foundation early.** After generating just the design system + homepage hero, stop and look at them. Does the FundedPal aesthetic feel right? If yes, the foundation is solid. If no, refine before generating 15 more surfaces.

**7. Export incrementally.** Don't generate 18 surfaces and then try to export them all at once. Export each as you finish it — drop the HTML in `/handoffs/`, update the README index, then move to the next surface.

---

## Common pitfalls

**Trying to make Claude Design a Figma replacement.** It's not. It's a fast prototyper with code handoff. Don't expect pixel-perfect design system management or layered files.

**Generating in regular Claude chat instead of Claude Design.** Claude.ai chat can produce React artifacts but doesn't have the same UI polish loop as Claude Design proper. Use the dedicated Design tab.

**Forgetting to specify dark theme.** Claude Design defaults to light. The FundedPal design system is dark-first. Always include "dark theme by default" in the setup prompt.

**Skipping the consistency pass.** After all marketing surfaces are generated, run a consistency review prompt asking Claude Design to ensure same hero pattern, same section spacing, same color discipline across all pages.

**Designing screens in isolation.** Some surfaces only make sense alongside others. The dashboard hero needs to feel consistent with the marketing site's aesthetic. Reference earlier conversations when working on related surfaces.

**Combining unrelated surfaces in one conversation.** One prompt = one conversation = one surface (or one tightly-coupled group). Combining a homepage with a settings page produces worse output for both.

**Forgetting the file header.** Without the comment header noting purpose and build prompt, you'll have 18 HTML files in `/handoffs/` and won't remember which is which.

---

*Use this guide alongside BUILD_SPEC.md. The build sequence in BUILD_SPEC.md is authoritative; this guide explains how Claude Design fits into that sequence.*
