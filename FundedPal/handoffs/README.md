# FundedPal

> Built for prop accounts. Engineered for compliance.

An AI-driven SMC/ICT trading platform purpose-built for prop firm accounts. Connects to supported prop firm accounts, auto-detects every rule that account operates under, and trades within a hard compliance layer that physically cannot place a rule-breaching order.

Built with the Anthropic-native pipeline: **Claude Design** for visual prototyping, **Claude Code** for production implementation.

---

## What it does

- **Compliance Guardian** — every order evaluated against the prop firm's active rules before execution. No breaches.
- **Risk Budget Layer** — soft caps that keep accounts well clear of firm hard rules.
- **Account Journeys** — tracks a trader's full progression (Phase 1 → Phase 2 → Funded) as a single journey.
- **EA Compliance Mode** — Signal + One-Tap Execute for prop firms that ban automation.
- **Phase-Aware Strategy** — different aggression curves for Phase 1, Phase 2, and Funded accounts.
- **AI Rule Detection** — Claude extracts rules from any prop firm's T&Cs.
- **Curated firm support** — 6 hand-verified prop firms with continuous rule monitoring.
- **Multi-Account Risk Allocator** — coordinates risk across simultaneous challenges.

**Platform support:** MetaTrader 5 only at v1. All 6 supported firms offer MT5. Other platforms (cTrader, MT4, Match-Trader, DXTrade) are out of scope for v1 and may be added in future releases based on user demand.

---

## Stack

```
Frontend       Next.js 15 · TypeScript · Tailwind v4 · Framer Motion · PWA via @serwist/next
Database       Supabase Postgres + RLS + Realtime
Auth           Supabase Auth
AI             Claude API (Sonnet 4.5)
Strategy       Python FastAPI service
Execution      Custom MQL5 EA bridge over WebSocket
Payments       Stripe
Hosting        Vercel + Railway + Supabase
Tooling        Built with Claude Design + Claude Code
```

---

## Documentation

The repo follows a clean documentation structure:

```
fundedpal/
├── CLAUDE.md                            # Project rules, design system, conventions
├── README.md                            # This file
└── /docs
    ├── BUILD_SPEC.md                    # Linear 38-prompt build sequence
    ├── CLAUDE_DESIGN_GUIDE.md           # How to use Claude Design with the build
    ├── COMPATIBILITY_RESEARCH.md        # Why each prop firm is or isn't supported
    ├── PWA_NOTES.md                     # PWA implementation, iOS quirks, Web Push
    └── AUDIT_CHECKLIST.md               # Pre-launch QA for high-risk areas

/content
    └── faq.md                           # Public FAQ
```

For Claude Code: read CLAUDE.md on every task. Read BUILD_SPEC.md when working on a specific prompt. Other docs are reference.

---

## Getting started

### Prerequisites

- Node 20+
- pnpm 9+
- Python 3.11+ (for the strategy engine)
- A Supabase project
- A Stripe account (test mode for development)
- An Anthropic API key
- A Claude Pro/Max/Team subscription (for Claude Design access)
- Claude Code CLI installed locally
- MetaTrader 5 build 3815+ (for testing the EA bridge)

### Setup

```bash
# Clone and install
git clone <repo-url> fundedpal
cd fundedpal
pnpm install

# Environment
cp .env.example .env.local
# Fill in Supabase, Claude, Stripe, and bridge values

# Database
pnpm supabase db push

# Develop
pnpm dev
```

The app runs on `http://localhost:3000`. The strategy engine and EA bridge run as separate services — see their respective READMEs in `/strategy-engine/` and `/ea-bridge/`.

### Common commands

```bash
pnpm dev              # Next.js dev server
pnpm typecheck        # TypeScript check
pnpm lint             # Biome lint
pnpm format           # Biome format
pnpm test             # Unit tests
pnpm build            # Production build
```

---

## Build flow

The project is built sequentially through 38 prompts in BUILD_SPEC.md, organised into 8 phases:

- **Phase 0** — Foundation (repo setup, PWA, design system, UI primitives)
- **Phase 1** — Auth and application shell
- **Phase 2** — Prop firm rule engine and curated directory
- **Phase 3** — Account connection, MT5 bridge, journey framework
- **Phase 4** — Compliance Guardian and Risk Budget Layer
- **Phase 5** — SMC/ICT strategy engine
- **Phase 6** — Innovative features (allocator, forecaster, news pause, journal, rule monitor, health score)
- **Phase 7** — Billing, marketing site, launch hardening

Visual surfaces are designed in Claude Design first, then handed off to Claude Code via export bundles. See CLAUDE_DESIGN_GUIDE.md.

---

## Design system — "Quiet Authority"

FundedPal uses a deliberate, restrained design language closer to a Bloomberg terminal redesigned by a Swiss design studio than a retail crypto product.

- **Typography:** Switzer (UI) · Fraunces (editorial display) · JetBrains Mono (numbers)
- **Palette:** Warm bone on deep ink, with champagne and teal accents
- **Motion:** Custom cubic-bezier easing, 60ms staggered reveals, respects prefers-reduced-motion

Full design tokens documented in CLAUDE.md.

---

## Risk philosophy

FundedPal is intentionally restrictive. The platform is harder to lose money with than to make money with — by design.

- Per-trade risk caps cannot exceed 1% of starting balance
- Daily soft cap cannot exceed 75% of the firm's hard rule
- Override features always carry friction
- The Compliance Guardian is the source of truth — no bypass routes
- Position sizing is risk-based, never lot-based

Read the full philosophy in CLAUDE.md. Read the trader-facing version in `/content/faq.md`.

---

## Disclaimers

FundedPal is software tooling, not regulated financial advice. Automated trading involves substantial risk of loss. Past performance does not guarantee future results. Users are responsible for compliance with their prop firm's terms and any applicable local regulations.

Some prop firms prohibit automated trading. FundedPal's EA Compliance Mode (Signal + One-Tap Execute) handles these cases — never bypass it.

---

## Operator

Built and operated by **AkomzyAi Consulting**.

---

*FundedPal — Built for prop accounts. Engineered for compliance.*
