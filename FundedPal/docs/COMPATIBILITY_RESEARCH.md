# FundedPal — Prop Firm Compatibility Research

> **Update — Launch curation decision (April 2026):** Following this research, the launch firm list has been narrowed from 12 to **6 supported firms**. The other 6 firms remain in the system as `unsupported_firms` — visible in scan results with explanatory reasons, but not connectable. See `/docs/PROMPT_29_5_AMENDMENT.md` for the implementation specification.
>
> **Launch firm list:** FTMO, FundedNext, The Funded Trader, E8 Markets, Funded Trading Plus, Goat Funded Trader.
>
> **Unsupported at launch:** Apex Trader Funding, Topstep, MyFundedFX, FundingPips, Maven Trading, City Traders Imperium. Each has a defined reason and alternative firm recommendations in the `unsupported_firms` table.

> **Purpose:** Document the current EA / automated trading rules of every prop firm in FundedPal's seeded database, classify each firm's compatibility with FundedPal's operating model, and define how the platform should behave for each one.
>
> **Status:** Initial research conducted April 2026 based on publicly published rules. Re-verify quarterly.
>
> **Source disclaimer:** Rules summarised from each firm's official help center and T&Cs as of the date listed. Always verify current rules before launch — prop firm policies change frequently.

---

## How to read this document

Each firm is classified into one of four compatibility tiers:

- **🟢 GREEN — Full Compatible**: EAs explicitly allowed. FundedPal can run in full auto-execute mode.
- **🟡 AMBER — Conditional**: EAs allowed with conditions (uniqueness, capital caps, platform restrictions). FundedPal must enforce these conditions in code.
- **🟠 ORANGE — Signal Only**: Automated execution prohibited; signal generation + manual one-tap execute is acceptable.
- **🔴 RED — Not Supported**: EAs and signal services prohibited. FundedPal cannot operate; firm appears in the directory for reference rules only.

---

## 1. FTMO 🟢 GREEN

**Rules verified:** April 2026. Source: [ftmo.com/forbidden-trading-practices](https://ftmo.com/en/forbidden-trading-practices/) and [FTMO Help](https://ftmo.com/en/faq/which-instruments-can-i-trade-and-what-strategies-am-i-allowed-to-use/).

**EA policy:** EAs explicitly allowed on Challenge, Verification, and Funded accounts on MT4, MT5, and cTrader. No pre-approval needed. No source code submission.

**Conditions:**
- Strategy must be replicable on real markets (no exploitation of platform inefficiencies)
- Server request limit: 2,000 per day per account (orders, modifications, TP/SL adjustments)
- Maximum capital allocation per strategy: $400,000 across all FTMO accounts owned by the trader
- HFT, latency arbitrage, and tick scalping are prohibited
- Third-party EAs run a risk if multiple traders use identical configurations (capital allocation breach)

**News restrictions on Funded accounts:** No new orders within 2 minutes before/after high-impact events.

**FundedPal compatibility:**
- Full auto-execute mode is acceptable
- Compliance Guardian must enforce 2,000 server request/day cap
- Compliance Guardian must enforce 2-minute news blackout on Funded phase
- Each FundedPal user's trades on FTMO must be unique (already covered by per-user signal generation)
- Track total capital deployed per strategy across user's FTMO accounts; cap at $400k

**Status:** ✅ Recommended for launch. Strong fit for FundedPal.

---

## 2. FundedNext 🟡 AMBER

**Rules verified:** April 2026. Source: [help.fundednext.com](https://help.fundednext.com/en/articles/8020763-is-ea-allowed-in-fundednext) and [FundedNext rules guide](https://tradingfinder.com/props/fundednext/rules/).

**EA policy (CFD/Forex accounts):** EAs allowed on **MT4 and MT5 only** with an additional EA usage fee paid at checkout. Not allowed on cTrader or Match-Trader.

**EA policy (Futures accounts):** EAs and bots allowed on both Challenge and Funded phases without uniqueness restrictions.

**Conditions (CFD):**
- Each EA/bot must use a unique strategy — identical trades across user accounts trigger soft breach
- Maximum allocation per strategy: $300,000 across all FundedNext accounts
- EA usage fee must be paid (additional cost)
- "EAs designed specifically to pass prop firm challenges" are explicitly banned (named ban list)
- Cannot switch between EA and manual trading mid-challenge or post-funding
- HFT prohibited
- Tools that only modify SL/TP/lot size are also classified as EAs

**FundedPal compatibility:**
- Full auto-execute acceptable on MT4/MT5 CFD accounts
- User must be informed that EA usage fee applies at checkout (not collected by FundedPal)
- Compliance Guardian must enforce method consistency: if user trades manually during challenge, system blocks auto-execute on funded; vice versa
- Per-strategy capital cap of $300k must be tracked
- Critical: FundedPal must NOT brand itself as "an EA designed to pass prop firm challenges" — that exact framing is on FundedNext's banned list. Position as compliance tool that happens to execute, not pass-the-challenge automation
- cTrader and Match-Trader users on FundedNext: signal-only mode

**Status:** ✅ Acceptable with conditions. Code must enforce uniqueness, capital cap, and method consistency.

---

## 3. The Funded Trader 🟠 ORANGE

**Rules verified:** April 2026. Source: [help.thefundedtraderprogram.com](https://help.thefundedtraderprogram.com/en/articles/6082908) and [proptraders.club](https://proptraders.club/does-the-funded-trader-allow-expert-advisors-eas-answered/).

**EA policy varies sharply by account type:**

- **Standard accounts (Normal + Swing):** EAs prohibited
- **Rapid accounts (Normal + Swing):** EAs prohibited
- **Royal accounts:** EAs allowed with rule compliance
- **Knight accounts:** EAs allowed with rule compliance

**Indicators:** Allowed on all accounts if trades are executed manually.

**FundedPal compatibility:**
- For Standard and Rapid accounts: signal-only mode (manual one-tap execute)
- For Royal and Knight accounts: full auto-execute acceptable
- The connect-account flow must detect account type at connection and configure execute mode automatically
- Third-party EA usage on Royal/Knight requires unique strategy settings to avoid copy-trading-pattern detection

**Status:** ✅ Supported with split mode. Code must auto-detect account type.

---

## 4. E8 Markets 🟡 AMBER

**Rules verified:** April 2026. Source: [help.e8markets.com](https://help.e8markets.com/en/articles/5515409) and [proptradingvibes.com](https://www.proptradingvibes.com/blog/e8-markets-rules-overview).

**EA policy:** EAs allowed on most account types (E8 One, E8 Classic, E8 Track, E8 Signature Forex, E8 Static Crypto). Prohibited on **E8 Signature Futures** and **Peak Scalping**.

**Conditions:**
- One trading strategy per user — multiple users running identical EA = termination
- Server request limit: 2,000 per day
- Position limit: 2,000 per day
- Over 50% of trades must remain open at least 1 minute (anti-HFT)
- HFT and latency arbitrage prohibited
- Third-party EAs allowed but proprietary EAs strongly recommended
- News restriction: 5 minutes before/after Tier 1 events on Funded (E8 Trader) accounts only

**Account-specific rules:**
- E8 Signature Futures: EAs prohibited entirely
- Peak Scalping: only capital management EAs (SL/TP automation), no trading EAs

**FundedPal compatibility:**
- Full auto-execute acceptable for E8 One, Classic, Track, Signature Forex, Static Crypto
- Connect flow must detect account type; block FundedPal connection for E8 Signature Futures and Peak Scalping with clear explanation
- Compliance Guardian must enforce: 1-minute minimum hold on >50% of trades, 5-minute news blackout on funded, 2,000 server requests/day, 2,000 positions/day
- Each user's signal must be unique (already covered by per-user signal generation)

**Status:** ✅ Supported with account-type filtering at connection.

---

## 5. Apex Trader Funding 🔴 RED → 🟠 SIGNAL ONLY

**Rules verified:** April 2026. Source: [Apex Prohibited Activities](https://support.apextraderfunding.com/hc/en-us/articles/40463668243099-Prohibited-Activities).

**EA policy:** Apex's published Prohibited Activities page states: "**No Automation or Algorithm Usage allowed**: Rewards are intended to recognize human traders actively participating in the learning process, not to reward automated systems."

However, ecosystem signals are mixed — third-party platforms (TradersPost, QuantVPS) market Apex automation integrations, and some traders run automated strategies via Rithmic API. The published rule is the source of truth, and it's strict.

**FundedPal compatibility:**
- Full auto-execute is **prohibited** by published rule
- Signal-only mode (signal generation + manual one-tap execute) is the only safe operating mode
- Even signal-only carries some risk if Apex interprets one-tap execute as algorithmic
- Recommend: at connect time, show user a clear disclosure: "Apex prohibits automated execution. FundedPal will only show you signals — you must tap to execute each one manually. Even with this safeguard, Apex may flag unusual execution patterns. Use at your own risk."

**Critical warning to traders:**
- No hedging (Apex requires directional trading only)
- No bracket trading on both sides of market
- All positions must close before market close (no overnight)
- 50% consistency rule

**Status:** ⚠️ Signal-only mode at launch. Reach out to Apex for explicit clarification before marketing the firm. Consider deferring to Phase 8 if Apex doesn't approve.

---

## 6. Topstep 🟠 SIGNAL ONLY (TopstepX) / 🟡 AMBER (legacy platforms)

**Rules verified:** April 2026. Source: [help.topstep.com](https://help.topstep.com/en/articles/10296582-prohibited-conduct) and [proptradingvibes.com Topstep rules](https://www.proptradingvibes.com/blog/topstep-rules-overview).

**EA policy:** Differs by platform.

- **TopstepX (their proprietary platform):** Automation explicitly prohibited. All trades must be manually executed. EAs, bots, algorithmic systems, and auto-executing signal services are banned. Includes auto-executing signals from external platforms.
- **TopstepX API:** Available for traders who want to build custom automation. Allowed.
- **Other supported platforms (NinjaTrader, Tradovate, Quantower, R|Trader Pro):** Automation allowed if compliant with Prohibited Conduct.

**FundedPal compatibility:**
- Topstep is a **futures-only** firm trading via NinjaTrader/Tradovate/TopstepX, not MT4/MT5
- FundedPal's MQL5 EA bridge does not connect to these platforms
- Initial launch: **defer Topstep support to Phase 8** when futures platform integration is built
- For v1: include Topstep in the firms directory as reference-only (rule database visible; cannot connect account)

**Status:** ⏸️ Deferred to Phase 8. Mark as "Coming Soon" in firms directory.

---

## 7. MyFundedFX 🟡 AMBER (CFD) / 🟢 GREEN (Futures via MyFundedFutures)

**Rules verified:** April 2026 — limited public detail; recommend re-verification before launch.

**EA policy (MyFundedFX CFD):** EAs allowed on supported MT4/MT5 platforms; some plan-specific restrictions on copy trading and HFT. Recent T&Cs (post-2025 update) include language about "AI-generated signals" requiring disclosure on certain plans.

**EA policy (MyFundedFutures):** As of July 2025 policy update, algo trading permitted on both evaluation and funded accounts. Tools must not exploit simulation advantages. HFT prohibited.

**FundedPal compatibility:**
- CFD accounts: full auto-execute acceptable on MT4/MT5
- Futures accounts: deferred to Phase 8 (platform integration required)
- Disclosure flow: at connect time, FundedPal must inform user of "AI-generated signal service" status and have user acknowledge the firm's disclosure requirements

**Status:** ✅ CFD supported at launch with disclosure flow. Futures deferred.

**Re-verification note:** MyFundedFX rules have changed multiple times in 2024–2025. Verify current T&Cs within 30 days of launch.

---

## 8. Funded Trader Plus / Funded Trading Plus 🟢 GREEN

**Rules verified:** April 2026. Source: [eafunded.com/firms/funded-trading-plus](https://www.eafunded.com/firms/funded-trading-plus) and [Funded Trading Plus / Myfxbook review](https://www.myfxbook.com/prop-firms/funded-trading-plus).

**EA policy:** EAs allowed on MT4 and MT5 with no pre-approval and no news restrictions (rare in industry).

**Conditions:**
- Standard trend-following, breakout, swing, and scalping EAs permitted
- Grid EAs explicitly prohibited
- Cross-account copy trading banned
- HFT and latency arbitrage banned
- No minimum trade hold times
- No consistency rules on most plans (Prestige Static is the exception — requires 3 trading days, 0.5% min profit/day)

**FundedPal compatibility:**
- Full auto-execute acceptable
- Compliance Guardian must NOT generate grid-style strategies (FundedPal's SMC/ICT engine doesn't anyway, so this is naturally compliant)
- Friendly to FundedPal's model — one of the simpler firms to support

**Status:** ✅ Recommended for launch. Strong fit.

---

## 9. FundingPips 🟡 AMBER

**Rules verified:** April 2026 — public rules less comprehensive; recommend direct outreach.

**EA policy:** EAs allowed on supported platforms (MT4 primarily, recently added MT5 and cTrader options). HFT prohibited. Copy trading restrictions apply.

**Conditions inferred from public sources:**
- 5% daily / 8-10% max drawdown on standard plans
- Profit target 8% (Phase 1), 5% (Phase 2)
- News trading allowed
- Minimum trading days requirement on most plans

**FundedPal compatibility:**
- Full auto-execute likely acceptable on MT4/MT5
- cTrader: signal-only mode (cTrader users at FundingPips have stricter automation rules)
- Recommend: send confirmation email to FundingPips support before launch

**Status:** ⚠️ Likely compatible but requires direct verification. Treat as Amber until firm confirms.

---

## 10. Goat Funded Trader 🟢 GREEN

**Rules verified:** April 2026. Source: [help.goatfundedtrader.com](https://help.goatfundedtrader.com/en/articles/10749630-can-i-use-expert-advisors-eas).

**EA policy:** EAs explicitly allowed. Direct quote from their FAQ: EAs are permitted as long as they comply with prohibited trading practices.

**Conditions:**
- HFT prohibited
- Gold Arbitrage EAs explicitly banned
- Standard EA strategies welcome
- No specific capital allocation cap mentioned in public docs

**FundedPal compatibility:**
- Full auto-execute acceptable
- Compliance Guardian's anti-HFT measures (1-minute minimum hold majority of trades) align with their rules
- SMC/ICT methodology is well outside the prohibited "Gold Arbitrage" category

**Status:** ✅ Recommended for launch. Friendly to automation.

---

## 11. Maven Trading 🟡 AMBER (with pre-approval requirement)

**Rules verified:** April 2026. Source: [proptrusted.com/prop-firm-rules](https://proptrusted.com/prop-firm-rules/).

**EA policy:** EAs and copy trading **permitted** on MT5 only. Not available on cTrader or DXTrade. **EAs require pre-approval before use.**

**Conditions:**
- Two-step pre-approval process:
  1. Email source code (.mq5) and compiled file (.ex5) to support
  2. Select "Enable EA" option at checkout
  3. Contact support to finalise activation
- For Qualified Analyst funded accounts: same EA must have been used during evaluation

**FundedPal compatibility:**
- Full auto-execute acceptable **only after Maven pre-approval**
- This is a significant barrier — FundedPal's MQL5 EA bridge would need to be submitted to Maven for review
- Recommend: defer Maven integration to Phase 8. Submit FundedPal's EA bridge to Maven for review post-launch; if approved, add Maven as a fully-supported firm. Until then, signal-only mode.
- News restrictions vary by account type (4-min window on Qualified Analyst Swing, 4–10 min on Alpha accounts)

**Status:** ⏸️ Signal-only at launch. Pursue Maven pre-approval as Phase 8 priority.

---

## 12. City Traders Imperium (CTI) 🟡 AMBER

**Rules verified:** April 2026 — limited recent public detail; recommend direct outreach.

**EA policy (per public sources):** EAs permitted on supported MT4/MT5 platforms. CTI's positioning emphasises long-term swing trading and discretionary methods.

**Conditions inferred:**
- HFT prohibited
- Up to 100% profit split on certain accounts
- Focus on consistency over aggressive trading
- Specific EA conditions not extensively documented publicly

**FundedPal compatibility:**
- Full auto-execute likely acceptable on MT4/MT5
- CTI's swing-trading focus aligns well with FundedPal's SMC/ICT methodology (which targets longer holds than scalping)
- Recommend: direct verification before launch to confirm

**Status:** ⚠️ Likely compatible but requires direct verification. Treat as Amber until firm confirms.

---

## Summary Matrix

| Firm | Tier | Auto-execute | Signal mode | At launch | Notes |
|---|---|---|---|---|---|
| FTMO | 🟢 GREEN | ✅ MT4/MT5/cTrader | ✅ | ✅ Yes | Enforce 2k req/day, $400k strategy cap |
| FundedNext | 🟡 AMBER | ✅ MT4/MT5 only | ✅ MT4/MT5 | ✅ Yes | EA fee + uniqueness + method consistency |
| The Funded Trader | 🟠 SPLIT | ✅ Royal/Knight only | ✅ All accounts | ✅ Yes | Auto-detect account type |
| E8 Markets | 🟡 AMBER | ✅ Most accounts | ✅ | ✅ Yes | Block Signature Futures + Peak Scalping |
| Apex | 🔴 RED → 🟠 | ❌ | ✅ With disclosure | ⚠️ Cautiously | Pursue clarification; consider deferral |
| Topstep | 🟠 SPLIT | ❌ Futures-only firm | n/a | ⏸️ Phase 8 | Reference-only at launch |
| MyFundedFX | 🟡 AMBER | ✅ CFD MT4/MT5 | ✅ | ✅ Yes | Disclosure flow required |
| Funded Trading Plus | 🟢 GREEN | ✅ MT4/MT5 | ✅ | ✅ Yes | Excellent fit |
| FundingPips | 🟡 AMBER | Likely ✅ | ✅ | ⚠️ Verify | Direct outreach needed |
| Goat Funded Trader | 🟢 GREEN | ✅ MT4/MT5 | ✅ | ✅ Yes | Excellent fit |
| Maven Trading | 🟡 AMBER | ⏸️ Needs approval | ✅ At launch | ⚠️ Signal only initially | Pursue Maven pre-approval |
| City Traders Imperium | 🟡 AMBER | Likely ✅ | ✅ | ⚠️ Verify | Direct outreach needed |

**Launch recommendation:** Open with **6 firms in Green/clear-Amber status** — FTMO, FundedNext, The Funded Trader, E8 Markets, Funded Trading Plus, Goat Funded Trader. These have clearest published EA permissions and strongest fit. Add the remaining 6 firms post-launch as verifications complete.

---

## Required Schema Changes

Update the `prop_firms` table to capture compatibility status:

```sql
alter table public.prop_firms add column compatibility_status text not null default 'pending_verification'
  check (compatibility_status in (
    'verified_green',          -- EAs explicitly allowed, full auto-execute
    'verified_amber',           -- Allowed with conditions enforced in code
    'verified_signal_only',     -- Auto-execute prohibited; signal-only mode
    'verified_split',           -- Per-account-type variation (e.g. The Funded Trader)
    'not_supported',            -- Reference rules only; cannot connect accounts
    'pending_verification'      -- Default — directory listing, no connect option
  ));

alter table public.prop_firms add column ea_policy_summary text;
alter table public.prop_firms add column ea_policy_source_url text;
alter table public.prop_firms add column ea_policy_verified_at timestamptz;
alter table public.prop_firms add column auto_execute_allowed boolean not null default false;
alter table public.prop_firms add column signal_mode_allowed boolean not null default true;
alter table public.prop_firms add column requires_disclosure boolean not null default false;
alter table public.prop_firms add column requires_pre_approval boolean not null default false;
alter table public.prop_firms add column server_request_cap_per_day int;
alter table public.prop_firms add column max_strategy_capital_cents bigint;
alter table public.prop_firms add column min_trade_hold_seconds int;
```

Also add per-account-type variation for firms like The Funded Trader and E8 Markets:

```sql
create table public.firm_account_type_overrides (
  id uuid primary key default gen_random_uuid(),
  firm_id uuid not null references public.prop_firms(id) on delete cascade,
  account_type text not null,                    -- 'standard','rapid','royal','knight','signature_futures', etc.
  auto_execute_allowed boolean not null,
  signal_mode_allowed boolean not null,
  notes text,
  unique (firm_id, account_type)
);
```

When a user connects an account, the connect flow:
1. Selects the firm from the directory
2. Selects account type (if firm has overrides)
3. System reads `firm.auto_execute_allowed` (or override) and configures the account's execute mode automatically
4. If auto-execute is not allowed, signal-only mode is forced and the user is informed clearly

---

## Suggested Connect-Flow UX Per Tier

**🟢 GREEN firms:**
> "FundedPal supports auto-execute on this firm. Trades will be placed automatically on signals that pass compliance checks. Auto-execute on. [Toggle to switch to signal-only mode]"

**🟡 AMBER firms:**
> "FundedPal supports this firm with conditions: [list conditions e.g. uniqueness rule, capital cap, EA fee at firm checkout]. Auto-execute is enabled. We'll enforce these conditions automatically."

**🟠 SIGNAL ONLY firms:**
> "This firm prohibits automated execution. FundedPal will show you signals you can execute with one tap — but each trade must be confirmed by you. Auto-execute is disabled. [Cannot toggle on]"

**🔴 NOT SUPPORTED firms:**
> "FundedPal cannot connect to accounts at this firm. Their rules prohibit our model. You can still view this firm's rule profile in our directory for reference. [No connect option]"

---

## Direct-Outreach Email Templates

For firms requiring verification (Apex, FundingPips, Maven, CTI, plus re-verification of any others), send this email from your client's official AkomzyAi Consulting / FundedPal address:

> **Subject:** FundedPal — Compliance-First Signal Service Integration Inquiry
>
> Hello [Firm Name] team,
>
> I'm reaching out from FundedPal, a new compliance-first trading platform built specifically to help traders survive prop firm challenges within their rules — not exploit them.
>
> Our platform's core feature is the Compliance Guardian, which enforces every prop firm's rules at the order layer and physically blocks trades that would breach drawdown, lot size, news, or consistency rules. We support both auto-execute (where firm rules permit) and signal-only modes (signal + manual one-tap execute) to accommodate firms that prohibit automated trading.
>
> Before recommending [Firm Name] to our users, I wanted to:
>
> 1. Confirm our understanding of your current EA / automated trading policy
> 2. Verify that signal-only mode (where the trader manually clicks to execute each trade based on a generated signal) is acceptable under your rules
> 3. Discuss whether full auto-execute would be acceptable for FundedPal users, given that our system enforces all your rules in real-time
>
> Our positioning is fundamentally compliance-first — we've designed FundedPal to make rule breaches harder, not easier. Happy to share technical details, our rule enforcement layer, or arrange a call.
>
> Thank you,
> [Name]
> FundedPal — *Built for prop accounts. Engineered for compliance.*
> [website] | [email]

Track responses in a spreadsheet. Non-responses after 14 days = default to most-restrictive interpretation in code.

---

## Re-verification Schedule

Prop firm rules change frequently. Schedule:

- **Quarterly automated check:** Re-fetch each firm's published rules page and diff against stored version. Auto-flag firms with detected changes.
- **Quarterly manual review:** Read each firm's current rules in full; update `ea_policy_verified_at` and any changed fields.
- **Pre-launch:** Within 30 days of going live, re-verify all 12 firms.
- **On firm-initiated rule change:** Affected accounts notified in-app and via email; re-validation required before next trade.

---

## Reality Check

Two honest things to flag:

**Not all firms in this seed are equally good fits.** FTMO, Funded Trading Plus, and Goat Funded Trader are the clearest wins — friendly EA policies, established reputations, healthy ecosystems. Topstep and Apex are futures-focused and harder to integrate cleanly. Maven requires pre-approval. Some smaller firms have less-documented policies.

**Launch with fewer firms, well-supported.** It's tempting to launch with all 12 to maximise market coverage. The wiser play is to launch with the 5–6 cleanest fits, get your first paying users, prove the model, then expand. Each additional firm carries integration cost (rules, account types, edge cases) and legal risk (incorrect rule encoding can blow user accounts).

**The prop firm industry is regulatory-fragile.** The CFTC's MyForexFunds case in 2023 ($310m alleged misappropriation) was a reminder that firms can disappear overnight. Firms most likely to survive long-term: FTMO, Topstep, Apex (longer track records, clearer compliance posture). Newer firms with aggressive growth may not be around in 24 months. Weight this in your prioritisation.

---

*Re-verify before each FundedPal release. Public T&Cs change frequently. This document was compiled from publicly available rules pages as of April 2026.*
