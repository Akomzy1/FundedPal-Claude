# How FundedPal Trades

> A complete, honest explanation of FundedPal's trading methodology — written for the traders who use it.

We believe you should know exactly what's happening with your account. This document explains every part of how FundedPal generates signals, sizes positions, and decides when to trade.

If you read it carefully, you'll understand FundedPal better than most traders understand their own strategies.

---

## The thirty-second version

FundedPal trades using **Smart Money Concepts and Inner Circle Trader methodology** — a school of technical analysis that looks for spots where institutional traders likely have unfilled orders, waits for retail traders to push price into those spots, then takes trades expecting the institutions to defend their levels.

The platform is **selective by design**. A typical week produces 1-3 high-quality setups per instrument. We don't trade for action. We trade when the market regime is favourable AND when five specific factors align.

Position sizing is **risk-based**, not lot-based. Every trade risks a fixed percentage of your *current equity* — naturally protecting accounts during drawdown and compounding during winning streaks. The Compliance Guardian rejects any trade that would breach your prop firm's rules — or your own risk budget.

If that sounds restrained, that's the point. **FundedPal is built for traders who'd rather be approved than apologise.**

---

## The philosophy

Most automated trading systems try to predict the market. They use indicators, machine learning, neural networks — all looking for an edge in price prediction.

FundedPal doesn't predict. It **reads institutional footprints** and trades alongside them.

Three assumptions sit at the foundation:

**Markets are not random.** Price moves are caused by orders being filled. Large institutions place orders in identifiable patterns, and those patterns leave visible traces on charts.

**Retail traders are predictable liquidity.** When price approaches a level where retail traders have obviously placed stop losses (just below recent lows, just above recent highs), institutions use those stops as liquidity to fill their own large orders. The result is a fast move that takes out retail stops, then reverses.

**Specific patterns mark institutional activity.** Order blocks, fair value gaps, and structure shifts are visible signatures of where institutional positioning happened.

The strategy doesn't try to be right about market direction broadly. It tries to identify specific, high-probability setups where the institutional thesis is clear, then take those trades with tight stops and asymmetric reward.

---

## The technical building blocks

These are the patterns FundedPal looks for. Each has a specific definition built into the engine.

### Market Structure

The skeleton of every analysis. Markets move in waves of higher highs and higher lows (uptrend), or lower highs and lower lows (downtrend). FundedPal identifies these swing points using a 5-bar fractal detection method — meaning a swing high is a candle whose high exceeds the two candles before and after it.

When price closes beyond the previous swing in the opposite direction, the structure has changed. Two specific patterns matter:

- **Break of Structure (BOS)**: price breaks the previous swing in the trend direction, confirming continuation
- **Change of Character (CHoCH)**: the first opposite-direction structure shift, suggesting a potential reversal

### Order Block (OB)

The most important pattern. An order block is **the last opposite-direction candle before an impulsive move that breaks structure**.

The thesis: institutions accumulated their positions in that candle's range. When price returns to that zone later, they'll defend it because they want their orders to remain profitable.

Specific rules:
- Must be the last opposite candle before the impulsive move
- The impulsive move must show displacement of at least 2× the previous candle's range
- The block is "valid" until price returns to it; once price closes through it, it's invalidated

You can think of an order block as **unfinished business** — institutions began something there and are likely to come back.

### Fair Value Gap (FVG)

A three-candle imbalance. When price moves so quickly that the candles don't overlap, it leaves a gap that markets often return to fill.

- **Bullish FVG**: gap between the high of candle 1 and the low of candle 3
- **Bearish FVG**: gap between the low of candle 1 and the high of candle 3
- "Filled" when price returns to the midpoint (50%) of the gap

FVGs represent **emotional moves** — too much enthusiasm in one direction, too quickly. Markets tend to come back and rebalance.

### Liquidity Pools

Areas where many traders likely have stop losses clustered. FundedPal identifies several types:

- **Equal highs / equal lows**: when price tests the same level multiple times within a small tolerance (0.1 ATR for major forex pairs, 0.15 for metals, 0.2 for indices), traders' stops accumulate just beyond
- **Session highs and lows**: the Asian range, London Open level, NY Open level — natural reference points
- **Swing point clusters**: where multiple recent swings sit close together

When price sweeps through a liquidity pool — taking out the stops — and immediately reverses, that's a signature of institutional accumulation.

### Optimal Trade Entry (OTE)

Once price has returned to a valid order block or FVG, the entry trigger considers **how deep the retracement has gone into the zone**. Rather than a binary trigger (e.g. only between 62-79%), FundedPal uses a continuous retracement score that peaks around 70% and tapers off toward 50% (too shallow) and 90% (too deep, likely to fail). This avoids artificial cliff-edges and reflects the reality that "good retracement" is a smooth gradient, not a hard zone.

The OTE zone is where high-probability entries happen. Outside it, the trade has worse risk-to-reward.

### Killzones

Specific time windows when institutional activity is highest. By default, FundedPal **only generates signals during these periods**:

- **Asian session**: 20:00–00:00 EST (range establishment)
- **London Open killzone**: 02:00–05:00 EST (highest manipulation)
- **New York AM killzone**: 08:30–11:00 EST (largest moves)
- **New York PM killzone**: 13:30–16:00 EST (close positioning)

Outside these windows, the engine is silent. This is deliberate — most failed prop accounts are blown by trading during low-liquidity periods where retail patterns work poorly.

---

## How signals form: the confluence engine

A trade doesn't trigger because one pattern appears. It triggers when **multiple patterns align** — and only when the market regime is favourable.

### The regime pre-condition

Before any confluence scoring, FundedPal checks the current market regime using ADX (a standard trend-strength indicator). The classification:

- **Trending** (ADX > 25) — normal scoring applies
- **Transitional** (ADX 20-25) — confluence threshold raised by 5 points (more selective)
- **Ranging** (ADX < 20) — no signals generated regardless of confluence

This is important because SMC/ICT patterns work much better in trending conditions than in chop. By filtering out ranging markets entirely, FundedPal avoids one of the biggest sources of false signals.

### The 5 confluence factors

When the regime allows, FundedPal scores 5 orthogonal factors totalling 100 points:

| Factor | Points | What it tests |
|---|---|---|
| Higher timeframe bias | 20 | Is the daily/4h structure aligned with the trade direction? |
| Location quality | 25 | Is price at a fresh point of interest (order block, breaker, FVG) AND at a high-quality retracement depth? Combined score from both. |
| Liquidity event | 15 | Did price just sweep equal highs/lows or run stops with a strong rejection wick? |
| Active killzone | 15 | Is the current session a high-probability time? |
| Structure shift | 25 | Has BOS or CHoCH printed on the entry timeframe? Downweighted to 10 if a liquidity sweep already scored (because sweeps frequently cause structure shifts). |

**The threshold for generating a signal is 70 out of 100** by default. This adjusts based on which phase your account is in (more on that below).

Most setups score in the 50-65 range. Those don't trigger. Only the cleanest setups — where the institutional thesis is unambiguous — make it through.

### Why these 5 factors and not more

Earlier versions of the confluence engine used 6 factors that turned out to be partially correlated. Liquidity sweeps frequently cause structure shifts. Order block proximity and OTE retracement are mathematically overlapping conditions. Counting these separately overstated confidence.

The current 5-factor model:
- **Merged** location quality (POI proximity + retracement depth) into a single score
- **Replaced** binary OTE detection (62-79%) with a continuous retracement score that peaks at 70% and tapers at the edges
- **Added** the regime pre-condition to suppress signals in choppy markets
- **Downweights** structure shift when a liquidity sweep already scored, to avoid double-counting correlated events

This is more rigorous than the earlier model, while remaining a rule-based system you can audit and reason about.

---

## A worked example

Let's walk through a complete signal so you can see exactly how this plays out.

**Setup on EUR/USD, 1-hour chart, London Open killzone (3:15 EST):**

**Step 0: Regime check.**
ADX(14) on the 1-hour reads 28 — the market is trending. Signals are enabled.
*Threshold: 70 (no transitional adjustment needed)*

**Step 1: Higher timeframe bias check.**
Daily structure shows higher highs and higher lows over the past two weeks. Bias is bullish.
*Score: +20 points*

**Step 2: Location quality (POI + retracement).**
Yesterday at 14:00 EST, an aggressive bullish move broke structure to the upside. The last bearish candle before that move ranged from 1.0820 to 1.0835 — that's a fresh bullish order block, and it sits at the 70% retracement of the previous bullish leg (the peak of the retracement quality curve). Price is currently at 1.0828, inside the order block.
*Score: +25 points (full marks — fresh POI at peak retracement)*

**Step 3: Liquidity event.**
During the Asian session, price drifted lower toward the order block. Just before London open, price spiked down to 1.0815 — taking out equal lows that had formed at 1.0820, then closed back inside the range with a strong rejection wick.
*Score: +15 points*

**Step 4: Active killzone.**
Time is 3:15 EST. London Open killzone is active.
*Score: +15 points*

**Step 5: Structure shift on entry timeframe.**
A small bearish move just shifted to bullish — the first higher low formed at 1.0825. Mini-CHoCH on the 1-hour.
*Score: +10 points (downweighted from 25 because liquidity event already scored — sweeps frequently cause structure shifts; counting both fully would double-count)*

**Total confluence: 85 out of 100. Above threshold (70). Signal generated.**

The signal:
- **Side:** Buy
- **Entry:** 1.0828 (current price within retracement zone)
- **Stop loss:** 1.0810 (just below the liquidity sweep low — 18 pips)
- **Take profit:** 1.0900 (next major liquidity pool above — 72 pips)
- **Risk:Reward:** 1:4

If your account is FTMO Phase 1 $100k currently sitting at $98,500 equity (had a small loss earlier this week) with FundedPal default risk settings (0.875% per trade), the position size is calculated as:

```
position size = (0.875% × $98,500) / (18 pips × $10 per pip)
              = $861.88 / $180
              = 4.79 lots
```

Rounded down to your broker's lot step (0.01) = **4.79 lots**.

Notice the sizing is calculated against your *current equity* of $98,500, not the original starting balance of $100,000. This is intentional — your position is automatically smaller because you're already slightly drawn down. As your equity recovers and grows, position sizes will scale up naturally.

This trade risks $862 and targets $3,447. If it hits, your equity moves from $98,500 to $101,947. If it loses, your equity moves to $97,638 — still well clear of FTMO's daily loss limit.

---

## Phase-aware aggression

The same setup is treated differently depending on which phase your account is in. This is one of the most important aspects of FundedPal.

### Phase 1 — Evaluation

You've paid the challenge fee. You need to hit the profit target. The strategy is **most aggressive** here:

- Confluence threshold: 65 (lower bar, more signals)
- Minimum reward target: 1.5R
- Up to 3 trades per killzone allowed
- Default per-trade risk: 0.875% on most firms

The thinking: the cost of failing Phase 1 is already locked in (the challenge fee). The marginal cost of one more trade is small. Better to take more shots and pass.

### Phase 2 — Verification

You're close to funding. Survival matters more than profit. The strategy becomes **balanced**:

- Confluence threshold: 75 (higher bar)
- Minimum reward target: 2R
- Only 1 trade per killzone
- Default per-trade risk: ~0.5%

The thinking: Phase 2 typically has the same drawdown rules as Phase 1 but a smaller profit target. You can afford to wait for A+ setups.

### Funded — Capital preservation

You're trading real funded capital. The cost of losing this account is months of grinding to replace it. The strategy becomes **defensive**:

- Confluence threshold: 85 (very high bar)
- Minimum reward target: 3R
- Maximum 1 trade per day
- Default per-trade risk: ~0.25%
- Position sizes effectively halved

The thinking: a funded account is worth far more than the marginal profit from one extra trade. Protect it.

You can override these defaults within hardcoded ceilings (per-trade risk can never exceed 1%, daily soft cap can never exceed 75% of your firm's hard rule), but the defaults are calibrated to keep accounts alive long-term.

---

## Position sizing — risk-based, equity-aware

Most retail bots use fixed lot sizes ("always 0.5 lots"). FundedPal doesn't.

Every trade is sized so that, if your stop loss hits, you lose **a fixed percentage of your current equity** — regardless of where the stop sits.

The formula:

```
position size = (per_trade_risk_pct × current_equity) / (stop_distance_in_pips × pip_value_in_account_currency)
```

This means:

- A trade with a tight stop produces a **larger** position
- A trade with a wide stop produces a **smaller** position
- The dollar risk per trade is a constant percentage of your *current* account size

### Why equity-based, not starting-balance-based

Most platforms calculate risk against your starting account balance. FundedPal calculates against your current equity. This sounds minor; it's actually one of the most important protective decisions in the system.

When equity equals starting balance (day one of the challenge), the math is identical. But when equity diverges:

- **During drawdown**, position sizes naturally shrink. You're trading a smaller account, so the system risks a smaller dollar amount per trade. Drawdown can't compound on itself the way it does with starting-balance sizing.
- **During winning streaks**, position sizes naturally grow. The strategy's edge compounds against the larger equity base.

This is how disciplined institutional desks size. It's mathematically the right approach for asymmetric risk-reward strategies trading inside drawdown constraints.

The Compliance Guardian also caps position size at your firm's lot limits — if a calculation would exceed FTMO's 30 lot per position cap, the trade is automatically reduced or rejected.

---

## The Risk Budget Layer

Even with equity-based sizing, FundedPal adds a second layer of protection: **soft caps**.

Your prop firm allows a 5% daily loss. FundedPal stops trading at 3.5%. The 1.5% gap is your buffer for slippage, spread, and the news event nobody saw coming.

Every trade is evaluated against three soft caps before reaching your account:

**Per-trade risk** — the worst-case loss of this single trade as a percentage of current equity. Hardcoded ceiling: 1%. Default: 0.875% on Phase 1, 0.5% on Phase 2, 0.25% on Funded.

**Daily soft cap** — your total daily exposure (realised losses today + worst-case from open positions + worst-case from this proposed trade). Hardcoded ceiling: 75% of your firm's hard rule. Default: 70%.

**Max trades per day** — number of trades opened today. Default: 3 on Phase 1, 2 on Phase 2 and Funded.

If any soft cap would be breached, the trade is rejected with a clear reason and a suggested adjustment. You can lower these caps further but you cannot raise them above the hardcoded ceilings.

This is intentional. **Most prop accounts blow not from the strategy being wrong, but from a single oversized trade.** The soft caps make that structurally impossible.

---

## The Compliance Guardian

Every signal — even high-confluence ones — must pass through the Compliance Guardian before reaching your account. The Guardian evaluates 13+ rules in sequence:

1. Per-trade risk check (soft cap)
2. Daily soft cap check (soft cap)
3. Max trades per day check (soft cap)
4. Stop loss required (hard rule — many firms require SL on every trade)
5. Lot size limits (hard rule)
6. Hedging restrictions (hard rule, varies by firm)
7. Weekend holding rules (hard rule)
8. News blackout windows (hard rule)
9. Consistency rule (hard rule, varies by firm)
10. Daily loss limit (hard rule — your firm's actual ceiling)
11. Maximum drawdown limit (hard rule)
12. EA permission check (hard rule, varies by firm and account type)
13. Inactivity rules (hard rule)

If any check fails, the trade is rejected with a clear, human-readable reason. Examples:

> *"Soft cap: Per-trade risk*
> *This trade would risk 0.82% — your per-trade limit is 0.50%.*
> *Reduce lot size to 0.6 or widen stop to 1.10180."*

> *"Soft cap: Daily risk budget*
> *You've used 1.10% of your 1.40% daily budget. This trade would put you at 1.85%.*
> *Wait for tomorrow or reduce the trade to 0.30%."*

> *"Hard rule: News blackout*
> *NFP releases at 13:30 UTC (in 23 minutes). Your firm prohibits trading EUR/USD within 2 minutes of high-impact USD events.*
> *Trade can resume at 13:32 UTC."*

The Guardian operates server-side. It cannot be bypassed — not by you, not by any premium tier, not by any override. This is how FundedPal protects your account from your worst impulses.

---

## Realistic expectations

Honesty matters here. Let's be precise about what this strategy can and can't do.

### What the strategy does well

**Pass prop firm challenges with materially better odds than self-directed trading.** The industry pass rate for self-directed prop traders is roughly 10-15%. FundedPal's structural risk discipline alone — the soft caps, max-trades-per-day enforcement, killzone scheduling, and regime filtering — meaningfully shifts those odds in your favour, before any consideration of strategy edge. Specific pass-rate numbers depend heavily on instrument, market conditions, and friction modelling assumptions, so we don't quote them as guarantees. What we do guarantee is that the platform refuses to let you blow accounts the obvious ways.

**Compound funded accounts steadily** in preservation mode. The Funded-phase aggression curve targets ~3-5% monthly returns with drawdowns capped at 4-5%. This is the profile prop firms want to see continue indefinitely.

**Avoid catastrophic drawdowns** because the Compliance Guardian and Risk Budget Layer make them structurally hard to produce.

**Eliminate emotional trading** — the engine doesn't tilt, doesn't revenge trade, doesn't FOMO into setups. It executes the same way after a 5-loss streak as after a 5-win streak.

### What the strategy can't do

**Beat the market in absolute return terms.** It's not trying to. It deliberately underperforms maximum-return strategies in exchange for better drawdown control.

**Work in all market conditions.** SMC/ICT is historically weakest during low-volatility, range-bound periods. FundedPal addresses this directly with the regime filter — when ADX shows the market is ranging, no signals are generated at all. Some weeks, this means no trades trigger. This is the engine being correct, not broken.

**Replace your judgment entirely.** A trader who watches the engine generate only 2 signals per week will eventually want to take their own ideas. FundedPal supports manual orders — but those orders go through the same Compliance Guardian. The engine is the disciplined floor, not the only path.

### What might surprise you

**Win rate is moderate** — roughly 40-55% in backtests with realistic friction modelling. This isn't a 90% win rate system. The edge comes from asymmetric risk-reward (typically 1:2 to 1:4), not from being right most of the time. Backtests without friction modelling can show inflated win rates that don't survive contact with real markets — FundedPal's backtest module always includes slippage, spread variation, and execution latency by default.

**Profit factor is the metric that matters.** Default backtests show profit factors of 1.6-2.2. That's healthy but not extraordinary. You're paying for consistency, not magic.

**Drawdowns happen.** Even with the Risk Budget Layer, monthly drawdowns of 3-5% are normal during losing streaks. Be prepared for these or you'll panic during what's actually expected variance.

**The strategy is rule-based, not adaptive.** It doesn't "learn" from losing trades. This is intentional — interpretable behaviour is more important than marginal performance gains for a compliance-first product.

---

## Parameter table

For traders who want to see exactly how decisions are made. These are the engine's default values; some are configurable in your settings.

### Regime filter (pre-condition)

| ADX value | Regime | Effect on signals |
|---|---|---|
| ADX < 20 | Ranging | No signals generated |
| ADX 20-25 | Transitional | Confluence threshold raised by 5 points |
| ADX > 25 | Trending | Normal scoring applies |

### Confluence scoring weights (5 factors)

| Factor | Weight | Notes |
|---|---|---|
| Higher timeframe bias | 20 | Daily/4h structure direction |
| Location quality | 25 | Merged: 60% POI presence + 40% retracement score (continuous, peaks at 70%) |
| Liquidity event | 15 | Sweep with strong rejection wick |
| Active killzone | 15 | Within enabled session window |
| Structure shift | 25 (or 10) | BOS/CHoCH on entry timeframe; downweighted to 10 if liquidity event already scored |

### Confluence thresholds by phase

| Phase | Threshold | Min target | Trades per killzone |
|---|---|---|---|
| Phase 1 | 65 | 1.5R | 3 |
| Phase 2 | 75 | 2R | 1 |
| Funded | 85 | 3R | 1 (per day) |

(Thresholds raised by 5 in transitional regime. No signals at all in ranging regime.)

### Default risk settings by phase

| Phase | Per-trade risk | Daily soft cap (% of firm hard rule) | Max trades/day |
|---|---|---|---|
| Phase 1 | 25% of soft cap (max 1%) | 70% | 3 |
| Phase 2 | 20% of soft cap (max 1%) | 65% | 2 |
| Funded | 15% of soft cap (max 1%) | 60% | 2 |

(All percentages are of *current equity*, not starting balance.)

### Pattern detection parameters

| Parameter | Value |
|---|---|
| Swing detection method | 5-bar fractal |
| Order block displacement minimum | 2× previous candle range |
| Order block compression filter | ≥3 candles below 1× ATR before displacement |
| FVG fill threshold | 50% (midpoint, configurable per instrument) |
| Liquidity tolerance — FX majors | 0.1 ATR |
| Liquidity tolerance — metals | 0.15 ATR |
| Retracement score peak | 70% (continuous curve, score declines toward 50% and 90%) |
| Regime classifier | ADX(14) on entry timeframe |

### Instrument specifications

| Instrument | Pip value source | Notes |
|---|---|---|
| EUR/USD, GBP/USD, AUD/USD, USD/CAD, NZD/USD, EUR/GBP | Standard 4-decimal (0.0001 = 1 pip) | Pip values consistent across brokers |
| USD/JPY, EUR/JPY | 2-decimal (0.01 = 1 pip) | JPY pair convention |
| USD/CHF | Standard 4-decimal | Standard |
| XAU/USD (Gold) | **Broker-reported, queried at connection** | Most brokers: $0.10/pip. Some: $0.01/pip. System never assumes — always reads from broker. |

### Killzone times (EST, DST-aware)

| Session | Window |
|---|---|
| Asian | 20:00–00:00 |
| London Open | 02:00–05:00 |
| NY AM | 08:30–11:00 |
| NY PM | 13:30–16:00 |

### Backtest friction modelling (always applied)

| Parameter | Value |
|---|---|
| Slippage — FX majors | 0.5 pips |
| Slippage — JPY pairs | 1.0 pips |
| Slippage — FX cross | 1.5 pips |
| Slippage — gold | 3.0 pips |
| Spread variation | 0.8× to 1.5× historical average |
| News spread spike | 3× during high-impact events |
| Execution latency | 200-500ms simulated |
| Re-quote probability | 2% |

### Hardcoded ceilings (cannot be overridden)

| Limit | Value |
|---|---|
| Maximum per-trade risk | 1% of current equity |
| Maximum daily soft cap | 75% of firm's hard rule |
| Minimum stop distance | 5 pips (FX) / 0.5 ATR (others) |
| No adaptive parameter tuning | Strategy parameters are stable; changes are deliberate version updates |

---

## Frequently asked questions about the strategy

### Why doesn't FundedPal use indicators like RSI or moving averages?

Indicators describe price; they don't explain why price moves. SMC/ICT focuses on the cause (institutional order flow) rather than effects (overbought/oversold readings). A 50-period moving average doesn't tell you where institutions placed orders; an order block does.

This isn't a value judgment about indicator-based trading — many strategies work. SMC/ICT is the methodology FundedPal commits to because it's interpretable, testable, and rule-based.

### Why so few trades?

Selectivity is the strategy. The win rate of a 70+ confluence setup is meaningfully higher than a 50-confluence setup. Trading low-confluence setups more frequently doesn't increase profit — it increases drawdown variance and emotional fatigue.

You're paying FundedPal partly for the discipline to wait. If we generated 30 signals a week, we'd be a different (and worse) product.

### What if I see a great setup the engine ignored?

You can take it manually. The "Place Order" button on your account dashboard lets you input any trade. The Compliance Guardian still evaluates it the same way it evaluates engine signals. If your manual trade passes, it's executed. If it breaches a soft cap or hard rule, it's rejected with a reason.

Many traders find that overrides eventually align with engine output anyway. The engine is wrong sometimes; you'll be wrong more often.

### Why doesn't the strategy use AI or machine learning?

Three reasons:

1. **Interpretability.** When a trade is rejected or generated, we can tell you exactly why. ML systems often can't.
2. **Compliance auditability.** Prop firms (and regulators) prefer rule-based systems they can review. Black-box ML raises red flags.
3. **Stability.** ML systems can develop unexpected behaviours after deployment. Rule-based systems behave predictably.

We may add ML-based components in the future for specific tasks (e.g. drawdown prediction). The core decision engine will remain rule-based.

### Does FundedPal trade during major news events?

The Compliance Guardian enforces your firm's news blackout window automatically. For FTMO Funded accounts, that's 2 minutes before and after Tier 1 events on the affected currencies. The strategy engine also doesn't generate signals during these windows.

You can configure additional news avoidance in your settings — for example, refusing to trade EUR pairs during ECB rate decisions even if your firm allows it.

### What instruments does FundedPal trade?

At launch, the engine generates signals for major forex pairs (EUR/USD, GBP/USD, USD/JPY, USD/CHF, AUD/USD, USD/CAD, NZD/USD, EUR/GBP, EUR/JPY) and gold (XAU/USD). Other instruments may be added based on validation results.

We deliberately avoid exotic pairs and crypto at launch — the SMC/ICT methodology is most validated on the most liquid instruments.

### How does FundedPal handle gold's different pip values?

This deserves a specific answer because gold is genuinely different from forex.

Gold's pip definition varies by broker. Most MT5 brokers define 1 pip as a $0.10 price movement; some use $0.01; rare cases use $1.00. If FundedPal assumed one definition and your broker used another, position sizing would be wrong by a factor of 10 or 100 — that's catastrophic.

To prevent this, FundedPal queries your broker's actual specifications when your account first connects. The MT5 EA reads the broker-reported tick size, tick value, contract size, and minimum lot step for every supported instrument and reports them back to FundedPal. The Compliance Guardian then uses your broker's actual values when calculating position sizes — never assumptions.

Practically, this means:
- Your first 30 seconds after connecting an account, gold trades cannot be placed (the system is still receiving instrument specs)
- Once specs sync, gold position sizing is calculated against your broker's exact pip definition
- If you change brokers, the system re-queries on reconnect — old specs don't carry over

This isn't unique to gold; the same applies to every supported instrument. We just emphasise gold because it's the most common source of pip-related mistakes among prop traders.

### Can I customise the strategy?

Some parameters are user-configurable: your risk settings, your killzones, which instruments to monitor, and your phase aggression curve (within ceilings).

The pattern definitions (order block, FVG, OTE) are not configurable — these are the heart of the methodology and changing them would mean trading a different strategy entirely.

If you want a fully customisable strategy engine, FundedPal isn't the right tool. If you want the discipline of a rule-based methodology with prop firm rule enforcement, you're in the right place.

### What happens during a losing streak?

The same thing that happens during a winning streak. The engine evaluates each setup on its own merits regardless of recent history. There's no martingale, no anti-martingale, no "make it back" logic.

If your daily soft cap is hit during a losing streak, no further trades are placed that day — that's the protection working as intended.

If the strategy enters an extended drawdown (over 5% on the journey), the dashboard surfaces a warning and suggests reviewing your settings or pausing the engine. This is rare but possible.

### How is the strategy validated?

Pre-launch, the engine is backtested against 3+ years of historical data on each supported instrument under each supported firm's rules. Every backtest applies realistic friction (slippage, spread variation, execution latency, news blackouts) — without these, results are inflated and meaningless.

Sanity criteria for green-light launch:

- Signal frequency: 5-25 signals per instrument per quarter
- Win rate: 40-55% (after friction modelling)
- Average reward: ≥1.5R
- Maximum drawdown: <15%
- Backtest must include slippage, spread, latency, and news blackout simulation

We don't quote a specific Phase 1 pass rate as a marketing number because it depends heavily on instrument, market regime, and friction assumptions, and would mislead more than inform. What we commit to is honest backtesting infrastructure that produces results survivable in live trading.

Post-launch, real-world performance is tracked and visible in your dashboard. If aggregate user metrics significantly diverge from backtested expectations, parameters will be reviewed.

We don't claim historical performance guarantees future results. We do claim that the methodology is rigorously defined, the parameters are honestly published, and our backtests model the friction that real markets will apply.

---

## A final note

FundedPal's strategy will frustrate you sometimes.

You'll see a setup the engine rejected, and the next candle will be a 2% mover. You'll be in a losing streak and want to override the soft caps. You'll think you know better than the system — and sometimes you'll be right.

That's not a bug. It's the product.

For every setup we cause you to miss, we prevent multiple account-ending mistakes. The maths works in your favour over months and years, not days. Most retail traders never reach the months and years because they blow accounts in the first weeks.

If you can accept that trade-off, FundedPal is built for you.

If you can't, we're the wrong tool — and we'd genuinely rather you knew that now than after subscribing.

---

*This document describes FundedPal's trading methodology as of v1 launch. Parameters and weights may be adjusted based on real-world performance data. Material changes will be announced in the changelog and notified to active users.*

*Last updated: pre-launch.*
