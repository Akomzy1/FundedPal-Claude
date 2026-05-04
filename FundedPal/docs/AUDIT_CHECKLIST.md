# FundedPal — Pre-Launch Audit Checklist

> **Purpose:** Verify the four highest-risk areas of the FundedPal codebase before paying users connect real prop firm accounts. Built after Prompt 25 to catch gaps in earlier-built systems without unnecessary refactoring.
>
> **When to run:** Once Prompts 1–35 are complete and before the first paying customer connects an account. Ideally during the Prompt 35 hardening phase.
>
> **How to use:** Walk each section. For every checklist item, verify in the codebase or by running the listed test. Mark as ✅ pass, ⚠️ partial, or ❌ fail. Run the remediation prompts only for items that fail or are partial.

---

## How this document works

Each of the four sections follows the same structure:

1. **Why this area matters** — the production failure mode if unaudited
2. **Verification checklist** — specific things to check, with test cases
3. **Green-light criteria** — what "good enough for v1 launch" looks like
4. **Remediation prompt** — drop-in Codex prompt to fix gaps if found

The discipline: **don't run remediation prompts speculatively.** Only run them for items that genuinely fail. Working code that's slightly different from the spec is fine.

---

## Section 1 — MT5 EA Bridge (Prompts 12 + 13)

### Why this matters

The bridge is the centrepiece of FundedPal's technical risk. A bridge that drops connections, mishandles reconnects, or fails to confirm orders means traders see ghost positions, miss fills, or — worst case — get charged for trades that never executed. This is the area most likely to cause a refund-demanding bug in production.

### Verification checklist

**1.1 Heartbeat resilience**
- [ ] EA sends heartbeat every 5 seconds (verify in MQL5 source)
- [ ] Server marks account as `paused` after 60 seconds without heartbeat
- [ ] Account auto-recovers to `live` when heartbeat resumes
- [ ] Test: kill the EA mid-session; verify status shifts to `paused` within 90 seconds; restart EA; verify status shifts back to `live` within 15 seconds

**1.2 Reconnection state machine**
- [ ] EA implements exponential backoff on disconnect (1s → 2s → 4s → 8s → 16s → 30s max)
- [ ] On reconnect, EA sends a full `ACCOUNT_STATE` snapshot (not delta)
- [ ] Server reconciles its known positions against EA's snapshot
- [ ] Test: drop network on the EA's machine for 2 minutes; verify reconnection succeeds and position state matches between EA and server

**1.3 Order idempotency**
- [ ] Every `POSITION_OPEN_REQUEST` carries a server-generated `client_request_id` (UUID)
- [ ] EA stores recent `client_request_id`s and refuses to execute the same one twice
- [ ] If the WS connection drops between order placement and confirmation, on reconnect the server queries the EA for the status of unconfirmed `client_request_id`s
- [ ] Test: place an order, kill the WS during the round trip; verify on reconnect the order is correctly attributed (executed once OR not at all, never twice)

**1.4 Symbol mapping**
- [ ] System handles broker-specific symbol suffixes (`.r`, `.pro`, `-Pro`, `.m`, `_PRO`, etc.)
- [ ] A `symbol_aliases` table or config maps FundedPal's canonical symbol names to broker-specific names
- [ ] Test: connect a demo MT5 account from a broker using `EURUSD.r` notation; verify orders for "EURUSD" route correctly

**1.5 Pip value & lot calculation**
- [ ] EA correctly calculates pip value for JPY pairs (0.01 not 0.0001)
- [ ] EA correctly handles gold/XAUUSD (pip value differs from FX pairs)
- [ ] EA correctly handles indices (US30, NAS100, etc — variable pip values)
- [ ] EA respects broker's minimum lot step (0.01 standard, but some brokers use 0.1)
- [ ] Test: place identical 0.5% risk orders on EURUSD, USDJPY, XAUUSD; verify lot sizes differ correctly per pip value

**1.6 Position state reconciliation**
- [ ] On reconnect, if server thinks a position is open but EA reports it closed → server marks it closed with `closed_at = reconnection_time`, `exit_price = current_market`, logs a `reconciliation_event`
- [ ] On reconnect, if server has no record but EA reports a position open → server creates a position record with `source = 'reconciled'` flag
- [ ] Reconciliation events trigger an alert to Logsnag (these should be rare)
- [ ] Test: manually close a position in MT5 while bridge is disconnected; verify on reconnect the server detects the closure

**1.7 Multi-account isolation**
- [ ] Each MT5 account runs its own EA instance with its own session token
- [ ] One account's WS messages cannot affect another account's state
- [ ] Account-level rate limiting (no single account can flood the bridge)
- [ ] Test: connect two MT5 accounts simultaneously; place orders on both; verify no cross-contamination

**1.8 Authentication**
- [ ] WS connection requires HELLO message with valid HS256-signed token
- [ ] Tokens are scoped to a specific `account_id` and expire in 24h
- [ ] Server rejects HELLO from any token that doesn't match the account being requested
- [ ] Test: try connecting with an expired token; verify connection rejected. Try connecting account A's token to claim account B; verify rejected.

**1.9 Error code mapping**
- [ ] Every MQL5 trade return code (`TRADE_RETCODE_*`) maps to a structured server error
- [ ] User-facing error messages are human-readable (not "TRADE_RETCODE_TIMEOUT")
- [ ] Common errors handled gracefully: insufficient margin, market closed, invalid stops, requote
- [ ] Test: place an order with a stop loss too close to entry; verify user sees clear "Stop loss too close to current price" not "TRADE_RETCODE_INVALID_STOPS"

**1.10 Latency monitoring**
- [ ] Round-trip time from order request to EA confirmation is logged per trade
- [ ] Alert fires if 95th percentile RTT exceeds 2 seconds
- [ ] Latency is shown to user in the position details (helpful for trust)

### Green-light criteria

The bridge is ready for v1 launch when:

- Items 1.1, 1.2, 1.3, 1.5, 1.7, 1.8 all pass (these are critical)
- Items 1.4, 1.6, 1.9, 1.10 are at least partial (degraded but not broken)
- Bridge has run continuously on a demo MT5 account for **7 consecutive days** without manual intervention
- Manual disconnect-during-trade test passes 5 times in a row

### Remediation prompt (if bridge audit fails)

> Audit the FundedPal MT5 EA bridge implementation against the verification checklist in `/docs/AUDIT_CHECKLIST.md` Section 1. For each failing item, implement the missing functionality:
>
> 1. Read `/lib/bridge/messages.ts`, `/lib/bridge/ws-server.ts`, and `/ea-bridge/FundedPalBridge.mq5`
> 2. Identify which checklist items are not currently enforced
> 3. For each gap, implement the fix following the FundedPal design system and TypeScript strict mode
> 4. Pay particular attention to: (a) idempotency via `client_request_id`, (b) reconnection state reconciliation, (c) symbol alias handling for broker-specific notations, (d) JPY/gold/indices pip value calculations
> 5. Add unit tests in `/tests/unit/bridge.test.ts` covering each fixed scenario
> 6. Update the EA source with corresponding fixes
> 7. Add a `bridge_events` table if not present: `id, account_id, event_type, payload, created_at` for audit trail
>
> Do NOT refactor working functionality. Only patch gaps identified by the audit. Verify every change with `pnpm test` and a manual test on a demo MT5 account before declaring complete.

---

## Section 2 — SMC/ICT Strategy Engine (Prompts 19 + 20)

### Why this matters

The strategy engine generates signals that traders execute. Wrong definitions of order blocks, FVGs, or BOS produce signals that look like SMC/ICT but aren't — and the difference shows up only after enough live trades to lose money. Codex picked specific definitions during the build; you need to verify they're correct ones.

### Verification checklist

**2.1 Backtest sanity check**
- [ ] Run a 3-month backtest on EURUSD 1h with default settings
- [ ] Signal frequency: between 5 and 25 signals over 3 months (less = engine is too strict; more = engine is too loose)
- [ ] Win rate: between 40% and 65% (anything outside is suspicious — too low = bad logic, too high = lookahead bias)
- [ ] Average R: at least 1.5 (lower = TP placement wrong)
- [ ] Maximum drawdown: less than 15% (higher = sizing or stop placement wrong)
- [ ] Repeat for GBPUSD, XAUUSD, USDJPY — verify similar order of magnitude

**2.2 Pattern detection accuracy — manual spot check**
- [ ] Pick 5 specific historical setups on EURUSD 1h (use TradingView to identify them by eye)
- [ ] For each: a clear order block, a clear FVG, a clear BOS, a clear CHoCH, a clear liquidity sweep
- [ ] Run the engine over the same data and verify it correctly identifies each pattern at approximately the right timestamp (within 1 candle)
- [ ] If 4+ out of 5 match → pass. If 3 or fewer → primitive definitions need adjustment

**2.3 Order block definition**
- [ ] Verify the implementation uses the standard ICT definition: the last opposite-direction candle before an impulsive move that breaks structure
- [ ] Verify "impulsive move" requires displacement of at least 2× the previous candle's range (or another committed-to threshold)
- [ ] Verify mitigation logic: order block is "valid" until price returns to it; once price closes through it, it's invalidated
- [ ] Test: examine 3 known order blocks in the codebase; verify each was identified correctly

**2.4 Fair Value Gap definition**
- [ ] Verify 3-candle definition: gap between candle 1's high and candle 3's low (for bullish FVG) or candle 1's low and candle 3's high (bearish FVG)
- [ ] Verify minimum gap size threshold (commits to either ATR-based or fixed pip — both defensible, but it must be one)
- [ ] Verify fill logic: FVG is "filled" when price returns to and trades through the midpoint (or 100% — commit to one)
- [ ] Verify inversion logic: a filled FVG can become an inverse FVG (acts as opposing force on retest)

**2.5 Liquidity equivalence tolerance**
- [ ] Verify "equal highs" / "equal lows" use a defined tolerance (e.g. within 0.1 ATR or within 3 pips on majors)
- [ ] Tolerance is configurable, not hardcoded magic numbers
- [ ] Tolerance differs for FX vs metals vs indices (gold's volatility ≠ EURUSD's)

**2.6 Higher timeframe bias**
- [ ] HTF bias methodology is committed: e.g. daily structure direction OR weekly OR a specific MA filter
- [ ] HTF bias correctly aligns or filters trades on entry timeframe
- [ ] When HTF and entry timeframe disagree, engine defaults to HTF (or commits to a specific rule)
- [ ] Test: in a known down-trending daily, verify the engine doesn't generate buy signals on the 1h regardless of confluence

**2.7 Confluence scoring**
- [ ] Scoring weights are explicit in code, not hidden in magic numbers scattered across files
- [ ] All factors that contribute are documented in a single config (HTF bias, POI proximity, liquidity sweep, killzone, OTE level, structure shift)
- [ ] Threshold for signal generation (default 70/100) is configurable per phase
- [ ] Scoring is deterministic — same inputs always produce same score

**2.8 Killzone definitions**
- [ ] Times use trader's selected timezone, with broker server time as tiebreaker
- [ ] Asian: 20:00–00:00 EST (or as committed)
- [ ] London: 02:00–05:00 EST
- [ ] NY AM: 08:30–11:00 EST
- [ ] NY PM: 13:30–16:00 EST
- [ ] Engine does NOT generate signals outside enabled killzones (when killzone filtering is on)
- [ ] DST handling: killzones shift correctly with DST changes

**2.9 Risk-based position sizing**
- [ ] Position size = (account starting balance × per_trade_risk_pct) / (entry_price - stop_loss in pips × pip_value)
- [ ] Lot size rounds DOWN to broker's minimum lot step (never up — undersizing is safe, oversizing breaks compliance)
- [ ] Calculation handles account currency != quote currency correctly
- [ ] Test: same setup on $10k vs $100k accounts produces lot sizes proportional to balance

**2.10 Lookahead bias check**
- [ ] Backtest results don't use future data — every decision at time T uses only data ≤ T
- [ ] No moving averages or indicators that "see" candle close before close happens
- [ ] Test: artificially shift the backtest data forward by 1 candle; verify results change (if they don't, you've got lookahead bias)

### Green-light criteria

Strategy engine ready for v1 launch when:

- Items 2.1, 2.2, 2.3, 2.4, 2.6, 2.9, 2.10 pass (critical correctness items)
- Items 2.5, 2.7, 2.8 are at least partial
- Backtest on at least 3 different symbols across 3 months produces sensible numbers (item 2.1 criteria met)
- Manual spot-check of 5 known patterns matches engine output (item 2.2)

### Remediation prompt (if strategy audit fails)

> Audit the FundedPal SMC/ICT strategy engine against the verification checklist in `/docs/AUDIT_CHECKLIST.md` Section 2. The Python service lives in `/strategy-engine/`.
>
> 1. Read all files under `/strategy-engine/smc/` and `/strategy-engine/ict/`
> 2. Identify which checklist items are not currently met
> 3. For each gap, write the correct implementation following standard ICT definitions:
>    - Order block: last opposite candle before impulsive structure-breaking move; displacement ≥ 2× previous candle range; mitigated when price closes through
>    - FVG: 3-candle imbalance; gap defined by candle 1 wick to candle 3 wick; filled at midpoint
>    - Liquidity equivalence: configurable tolerance (default 0.1 ATR for FX, 0.15 ATR for metals, 0.2 ATR for indices)
>    - HTF bias: daily structure direction (last 3 swing points)
> 4. Centralise all magic numbers in a single `/strategy-engine/config/parameters.py` file with comments explaining each
> 5. Add a `/strategy-engine/tests/test_known_patterns.py` test file with at least 10 hand-curated historical examples (5 patterns × 2 symbols) and verify the engine identifies each correctly
> 6. Run a 3-month backtest on EURUSD 1h, GBPUSD 1h, XAUUSD 1h after fixes; signal frequency, win rate, and avg R should fall within the ranges defined in checklist 2.1
>
> Do NOT change the high-level architecture. Only correct definitions and parameters where the audit found gaps. Document every parameter change with a comment explaining the source (e.g. "ICT definition per ICT Mentorship 2022 Episode 14").

---

## Section 3 — Claude Rule Extraction (Prompt 9)

### Why this matters

Wrong rules in the database mean traders blow accounts thinking they're compliant when they're not. This is the rule database moat at risk. A single wrong consistency-rule extraction could cost a user thousands.

### Verification checklist

**3.1 Real-world extraction test**
- [ ] Run rule extraction on a firm NOT in the seed data (e.g. a smaller firm: Goat Funded Trader, Maven Trading, or City Traders Imperium)
- [ ] Compare extracted rules against the firm's published T&Cs manually
- [ ] At minimum these fields must be correct: daily_loss_pct, max_drawdown_pct, profit_target_pct, ea_allowed, weekend_holding, consistency_rule_pct
- [ ] Acceptable: 90%+ of extractable fields correct; disputed fields (e.g. ambiguous wording) flagged for user review

**3.2 Confidence scoring**
- [ ] Extraction returns a confidence score per field (0–100)
- [ ] Fields with confidence < 70 are flagged in the UI for explicit user confirmation
- [ ] Fields with confidence < 40 are not auto-saved at all — user must manually enter

**3.3 Multi-page handling**
- [ ] If the firm publishes rules across multiple URLs (rules page + FAQ + scaling plan), all are fetched
- [ ] Conflicting information between pages is flagged, not silently merged
- [ ] Test: try a firm with split rules (FundedNext has separate rule pages); verify all are processed

**3.4 Cost guardrails**
- [ ] Each extraction has a max token budget (e.g. 50k input tokens)
- [ ] Budget is enforced before the Claude call, not after
- [ ] If a single page exceeds budget, content is truncated to most-relevant sections (using rough heuristics — search for keywords like "daily loss", "drawdown", "consistency", etc.)
- [ ] Per-user rate limit: max 5 extractions per hour
- [ ] Test: try to trigger 10 extractions in quick succession; verify rate limiter blocks the 6th

**3.5 Schema validation**
- [ ] Claude response is parsed with Zod against `firm_rule_profiles` schema
- [ ] Malformed responses retry once with a corrective prompt ("Your previous response was not valid JSON. Try again.")
- [ ] After 2 failed attempts, return error to user with option to enter rules manually
- [ ] Test: simulate a malformed response; verify retry behaviour and graceful failure

**3.6 User confirmation flow**
- [ ] Extracted rules are NEVER auto-saved as `active = true`
- [ ] User must review every field in the confirmation UI
- [ ] Differences between extracted values and seeded firms (if applicable) are highlighted
- [ ] Source URL and `source_extracted_at` timestamp are stored on every confirmed profile

**3.7 Caching**
- [ ] Extractions are cached by URL + content hash for 7 days
- [ ] Cache is invalidated if the source page's content hash changes
- [ ] Cache hit returns instantly, no Claude call

**3.8 PII safety**
- [ ] T&Cs scraping does not leak any user identifying information into the prompt
- [ ] Extraction prompts are logged but not user data
- [ ] Rate limits keyed on user_id, not IP

### Green-light criteria

Rule extraction ready for v1 launch when:

- Items 3.1, 3.2, 3.4, 3.5, 3.6 pass
- At least 3 different firms tested end-to-end with manual verification
- Cost per extraction averages under $0.30
- User confirmation flow works smoothly in both themes

### Remediation prompt (if rule extraction audit fails)

> Audit the FundedPal Claude rule extraction service against the verification checklist in `/docs/AUDIT_CHECKLIST.md` Section 3.
>
> 1. Read `/lib/claude/extract-rules.ts` and `/app/api/claude/extract-rules/route.ts` and `/app/(app)/firms/extract/page.tsx`
> 2. For each failing checklist item, implement the fix:
>    - Add per-field confidence scoring (Claude returns confidence 0-100 alongside each field; low-confidence fields flagged in UI)
>    - Add cost guardrails: max 50k input tokens per extraction, rate limit 5 extractions/user/hour via Upstash Redis
>    - Add multi-page support: accept array of URLs, fetch all, concatenate with clear separators before sending to Claude
>    - Add caching keyed on URL + SHA256 of content, 7-day TTL via Supabase
>    - Add malformed-response retry logic: 1 corrective retry, then graceful failure
> 3. Update the prompt sent to Claude to require structured JSON output with confidence per field
> 4. Update the confirmation UI to highlight low-confidence fields with an amber indicator and require explicit user toggle to confirm them
> 5. Add unit tests covering: malformed responses, rate limit hits, cache hits, multi-page extraction
>
> Do NOT change the database schema unless adding columns. Verify with `pnpm test` and an end-to-end extraction on a firm not currently in the database.

---

## Section 4 — Compliance Guardian Calculations (Prompt 15)

### Why this matters

The Compliance Guardian's evaluator math determines whether trades are approved or rejected. A pip value bug on JPY pairs means traders trading USDJPY get wrong risk calculations — they think they're risking 0.5%, they're actually risking 5%. This is exactly the failure mode that causes catastrophic account blow-ups.

### Verification checklist

**4.1 Pip value per symbol class**
- [ ] EURUSD, GBPUSD (4-decimal majors): pip = 0.0001
- [ ] USDJPY, EURJPY (2-decimal JPY pairs): pip = 0.01
- [ ] XAUUSD (gold): pip = 0.01 OR 0.1 (depends on broker convention — must be tested per broker)
- [ ] US30, NAS100 (indices): pip = 1.0
- [ ] BTCUSD (crypto, if supported): pip = 1.0 typically
- [ ] Test: 0.5% risk calculation produces correct lot sizes across all symbol classes

**4.2 Lot-to-units conversion**
- [ ] Standard lot = 100,000 base currency units
- [ ] Mini lot = 10,000 units
- [ ] Micro lot = 1,000 units
- [ ] Code uses the broker's reported lot size, never hardcodes 100,000
- [ ] Test: connect a broker that defaults to mini-lots; verify lot calculations adjust

**4.3 Account currency conversion**
- [ ] Account is USD, trade is EUR/JPY: worst-case loss is calculated in USD using current EUR/USD rate
- [ ] Conversion rate is fetched at order placement, not cached
- [ ] If conversion rate fetch fails, order is rejected (not silently miscalculated)
- [ ] Test: account in GBP trading XAUUSD; verify P&L is correctly calculated and displayed in GBP

**4.4 Worst-case loss calculation**
- [ ] For a single trade: worst-case = (entry - SL in pips) × pip value × lot size, converted to account currency
- [ ] For multiple open positions: worst-case is sum of individual worst cases (not netted, even for hedged positions — prop firms typically calculate gross drawdown)
- [ ] Stop loss must be set; if not, trade is rejected with `STOP_LOSS_REQUIRED`
- [ ] Verify against the firm's actual rule wording — some firms net hedged positions, most don't

**4.5 Timezone handling for "today"**
- [ ] "Today" is defined by the broker server's reset time (typically 00:00 server time, often EST)
- [ ] NOT user local time, NOT UTC, NOT account creation timezone
- [ ] Daily snapshots and daily loss calculations use this server-time-based "today"
- [ ] Test: trade at 23:55 server time and 00:05 server time the next day; verify they're recorded against different `trading_date` values

**4.6 Floating point precision**
- [ ] All monetary values stored as bigint cents
- [ ] Lot sizes stored as numeric(8,2) — never float
- [ ] Pip values used in calculation are stored or computed precisely, never truncated
- [ ] Comparisons against rule thresholds use cents-vs-cents, never float-vs-float
- [ ] Test: a trade that should be exactly at the per-trade limit is approved; a trade 1 cent over is rejected

**4.7 Equity high tracking**
- [ ] `accounts.equity_high_cents` updates whenever equity exceeds previous high
- [ ] Trailing drawdown calculations reference equity_high, not balance
- [ ] Static drawdown calculations reference starting_balance, not equity_high
- [ ] Test: trade up to a profit, then verify trailing drawdown threshold has moved up
- [ ] Test: trade down to a loss, then verify trailing drawdown threshold has NOT moved down

**4.8 Daily loss calculation**
- [ ] Includes realised P&L from positions closed today
- [ ] Includes worst-case unrealised P&L from currently open positions (using their stop losses)
- [ ] Does NOT double-count partial closes
- [ ] Excludes commissions and swap on the unrealised side (commissions only counted when realised)
- [ ] Test: open a 1-lot position with a defined SL; verify daily loss usage updates immediately

**4.9 Rule profile activation**
- [ ] When an account changes phase (Phase 1 → Phase 2 → Funded), the active `firm_rule_profile` switches
- [ ] All in-flight orders re-evaluate against the new ruleset
- [ ] User is notified of the change with the new rules summarised
- [ ] Old rule evaluations remain in `compliance_decisions` for audit

**4.10 Audit completeness**
- [ ] Every order decision (approved or rejected) creates a `compliance_decisions` row
- [ ] Decision row includes: full proposed order, every rule evaluated, result of each, final decision, rejection reasons (if any)
- [ ] Decisions are immutable — once written, never updated
- [ ] Test: query `compliance_decisions` after 10 trades; every trade has exactly one corresponding decision

### Green-light criteria

Compliance Guardian ready for v1 launch when:

- Items 4.1, 4.4, 4.5, 4.6, 4.10 all pass (these are the money-bug items)
- Items 4.2, 4.3, 4.7, 4.8, 4.9 are at least partial
- Test coverage on `/lib/compliance/` is at least 85%
- A test suite specifically for JPY pairs, gold, and indices passes (item 4.1)

### Remediation prompt (if Compliance Guardian audit fails)

> Audit the FundedPal Compliance Guardian against the verification checklist in `/docs/AUDIT_CHECKLIST.md` Section 4.
>
> 1. Read `/lib/compliance/guard.ts`, `/lib/compliance/rule-checks.ts`, and `/lib/compliance/types.ts`
> 2. For each failing item, implement the fix following these specifications:
>    - **Pip values**: create `/lib/compliance/pip-values.ts` with explicit per-symbol-class definitions (FX majors, FX JPY, FX minors, gold, silver, US indices, EU indices, crypto). Function `getPipValue(symbol, broker)` returns precise value.
>    - **Currency conversion**: fetch live FX rate at order time via the OANDA API or a fallback rate provider. Cache for max 60 seconds. Reject orders if rate unavailable.
>    - **Timezone**: standardise "today" using each account's broker server timezone, stored on `accounts.broker_server_timezone`. Add to schema if missing.
>    - **Money math**: audit every line that does arithmetic on dollar amounts; ensure all are bigint cents. Convert any floats to cents at boundary.
>    - **Equity high**: ensure `equity_high_cents` updates only on new highs, never decreases. Use database trigger to enforce.
> 3. Add `/tests/unit/compliance-symbols.test.ts` with at least 30 scenarios covering: EURUSD, USDJPY, EURJPY, XAUUSD, XAGUSD, US30, NAS100, BTCUSD. For each: verify lot calculation, pip value, worst-case loss, and rule evaluation.
> 4. Add `/tests/unit/compliance-timezones.test.ts` covering: trade at 23:59 server time, trade at 00:01 server time, DST transitions, accounts with non-EST broker servers.
> 5. Run full compliance test suite; coverage must be ≥85% on `/lib/compliance/`.
>
> Do NOT change the high-level evaluation order. Only correct calculation and precision issues.

---

## Final pre-launch acceptance test

Once all four sections pass their green-light criteria, run this end-to-end acceptance test:

**The 7-day live demo test:**

1. Connect a real demo MT5 account from a real prop firm (FTMO Free Trial works)
2. Configure FundedPal in EA Compliance Mode (signals + one-tap)
3. Run for 7 consecutive days, taking signals as they come
4. At end of week, verify:
   - [ ] No bridge disconnections lasted >2 minutes
   - [ ] Every trade taken has a complete `compliance_decisions` audit row
   - [ ] Every signal generated has correct SMC/ICT reasoning logged
   - [ ] No unexplained equity discrepancies between MT5 and FundedPal dashboard
   - [ ] No rule profile mismatches between FundedPal and FTMO's actual rules
   - [ ] Daily loss calculations match FTMO's own dashboard within $0.50

If all 6 acceptance items pass, FundedPal is ready for paying users.

If any fail, run the corresponding remediation prompt and re-test the failed item before launch.

---

## A note on discipline

The temptation when reaching this audit will be to fix everything you spot, even small cosmetic issues. Resist this. The goal is not perfection — it's ensuring no production bug can lose a user money. Fix what fails the green-light criteria; document the rest as Phase 8 polish.

The bridge, the strategy, the rule extraction, and the Guardian math are the four places where bugs cost money. Everything else is recoverable.

---

*Run this audit after Prompt 35 completes. Don't skip it. Don't rush it.*
