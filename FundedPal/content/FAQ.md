# FundedPal — Frequently Asked Questions

> Honest answers about why FundedPal works the way it works.

---

## Why does FundedPal stop me trading before I hit my prop firm's daily loss limit?

Because the line is the wrong place to defend.

If your prop firm allows 5% daily loss, FundedPal stops you at around 3.5%. That 1.5% gap is deliberate. It's the buffer for slippage, spread, overnight gaps, and the news event nobody saw coming. Trading right up to the line means a single piece of bad luck blows the account.

Real prop firm survival is about staying well inside the rules, not skating along them. The traders who get funded — and stay funded — are the ones who never come close to the limits.

---

## Can I increase my per-trade risk above 1%?

No. The 1% per-trade ceiling is hardcoded and cannot be raised by any setting, any premium tier, or any support request.

This isn't a paternalism thing — it's structural. Here's the maths: a single 2% loss requires a 2.04% gain to recover. A 5% loss requires 5.26%. A 10% loss requires 11.11%. By the time you're risking 2%+ per trade, your recovery maths is already working against you, and one bad sequence puts the account in a hole you can't climb out of.

Capping at 1% means even three losing trades in a row leaves you at 97% of your starting balance. The strategy has room to recover. The trader has room to think.

---

## I'm an experienced trader. Why am I treated the same as a beginner?

Because experience is not the variable that blows accounts. **Discipline under pressure** is.

The most common pattern we've observed in prop trading: experienced traders take a small loss, identify the "perfect" setup to make it back, double their normal size, and lose 2-3x what they originally lost. Then they double again. The market doesn't care that they have ten years of screen time.

FundedPal's hard ceilings exist precisely because experienced traders are *more* likely to override their own rules under stress, not less. The platform protects you from the version of yourself that turns up after a losing morning.

If you're a discretionary trader who genuinely wants no constraints, FundedPal isn't the right tool. Trade through your broker directly. We'd rather refund you than ship a feature that hurts you.

---

## Why don't you have a "Conviction Mode" for high-confluence setups?

Because every trader believes their losing trades were high-confluence setups. Look back at your last ten losing trades — at the moment you took each one, did you think it was a low-quality signal? Of course not. You took it because it looked good.

A "Conviction Mode" override would be used most often in exactly the moments traders shouldn't be using it: tilted, behind on target, chasing a missed move. The data on prop firm failures is unambiguous on this point.

We may revisit this in 2027 once we have years of production data showing which users could safely use such a feature. Until then, it doesn't ship. Not as a free feature, not as an Elite-tier perk, not as an override behind a confirmation modal. The honest answer is no.

---

## What if I'm on the last day of my challenge and need to push for the target?

Then the challenge is already lost.

If you're at day 30 of 30 needing 4% with a 2% daily limit, the maths says you'd need to risk every dollar of your daily budget on a single trade with at least 2:1 reward — and you'd need to be right. Forced trades to hit a deadline have an empirical win rate well below 50%. The expected value is negative.

The right move on the last day of a failing challenge is to take only A+ setups within your normal risk parameters. If they don't appear, the challenge dies on schedule. The wrong move is to abandon the discipline that's the entire reason you're using FundedPal in the first place.

We deliberately don't build a "last day push" mode because it would mostly be used by people who shouldn't use it.

---

## Why does FundedPal limit me to 2-3 trades per day?

Because most prop accounts blow on the third or fourth trade of a losing day, not the first.

Trade one is taken with normal discipline. Trade two is sometimes a revenge trade. Trade three is usually emotional. Trade four is almost always damage. Capping the daily count breaks the sequence before it gets dangerous.

If your strategy genuinely produces more than 3 high-quality setups per day, you're probably overtrading. The SMC/ICT methodology FundedPal uses tends to produce 1-2 institutional-grade setups per session at most. Three is generous.

---

## Why is the Compliance Guardian stricter than my actual prop firm?

Because your prop firm wants you to fail.

This sounds harsh but the economics are clear: most prop firms make money primarily from challenge fees, not from profit splits with funded traders. They have no incentive to keep you in challenges. The rules are designed to be tripable — daily loss caps that are easy to breach in volatile markets, consistency rules that punish your best days, news restrictions that catch you off-guard.

FundedPal sits on your side of that table. Our incentive is to keep you trading because you only stay subscribed if you survive.

---

## Doesn't restricting users hurt your business?

Short term, yes. Long term, no — and the gap is the whole point.

Platforms that let traders take maximum risk see explosive growth in month one and 70%+ churn by month three because their users keep blowing accounts and blaming the tool. Platforms that protect users from themselves grow more slowly and retain at much higher rates.

FundedPal is built for the second curve. We'd rather sign up 100 traders who stay subscribed for 18 months than 1,000 who churn in 6 weeks. The pricing reflects this — there's no free tier because we don't want users we can't afford to support properly.

---

## Why do I have to wait through a confirmation modal to change my risk settings?

Because the changes you make to your risk settings while calm should not be easily reversible while tilted.

The 30-second cooldown isn't there to inconvenience you. It's there for the version of you who's down 1.5% on the day, watching another setup form, fingers hovering over the slider. That version of you needs friction. The version who first set up the account didn't.

We optimise for the trader you are at your worst, not your best. That's where the protection actually matters.

---

## Can I use FundedPal with multiple prop firms simultaneously?

Yes — Pro tier supports 3 accounts, Elite supports unlimited. The Multi-Account Risk Allocator coordinates across them so you can't blow them all on the same correlated trade.

But honestly, most successful funded traders run 1-2 accounts, not 5. Spreading thin across many challenges multiplies fees without proportionally improving expected outcomes. If you're tempted to run 6 challenges to "increase your odds", consider whether you'd be better served by running 2 properly.

---

## What happens if my prop firm changes their rules mid-challenge?

FundedPal monitors rule changes for the firms in our database. When we detect a change, we re-validate every affected account against the new ruleset and notify you in-app and via email. Your account stays paused until you confirm you've reviewed the changes.

This is one of the genuine advantages of using FundedPal versus a generic trading bot — the bot has no idea your firm just tightened the consistency rule. We do.

---

## What does FundedPal do that I can't do myself with discipline?

Honest answer: in theory, nothing. A perfectly disciplined human trader following an ironclad risk plan, executing SMC/ICT setups flawlessly during kill zones, never deviating, never tilting, never overtrading — that trader doesn't need us.

That trader also doesn't exist. Twenty years of trading research is unanimous on this: humans are bad at executing their own rules under pressure, especially with money at stake, especially after losses.

FundedPal's value isn't the strategy or the technology. It's the structural inability to break the rules. It's automation as a commitment device — you decide your risk parameters when you're calm, and we make them physically enforceable when you're not.

---

## Is FundedPal regulated?

FundedPal is software tooling, not a regulated investment service. We don't manage your money, take custody of funds, or provide financial advice. Your prop firm holds the trading capital and your relationship is with them.

We're a UK-registered company with proper data protection registration and operate within UK consumer software law. Trading involves substantial risk of loss; past performance does not predict future results; you are solely responsible for your trading decisions.

---

## What about regulatory risk to the prop firm industry itself?

Real and worth knowing about. The CFTC sued MyForexFunds in 2023 for $310m in alleged misappropriation; the firm shut down. Regulators in the US, UK, and EU are scrutinising the prop firm model.

Some prop firms may not exist in 18 months. FundedPal mitigates this by:

1. Supporting multiple firms — you can move accounts between them
2. Publishing health scores so you can avoid firms with red flags
3. Building tooling that's useful across the industry, not tied to any single firm's survival

If you're considering a prop firm, check our public Health Score directory before committing the challenge fee.

---

## Why don't you offer a free tier?

Because trading software with consequences as serious as breaching a prop firm rule deserves to be paid for. A free tier means cutting corners on infrastructure, support, or features — none of which we're willing to do.

The 14-day trial gives you full access to your chosen tier. If FundedPal doesn't pay for itself in saved challenge fees and preserved accounts, cancel before the trial ends. Most users find the cost trivial relative to a single saved challenge.

---

## What's the catch?

The catch is that FundedPal will frustrate you sometimes. You'll see a setup the system rejects, and the next candle will be a 2% mover, and you'll wonder why you're paying us to miss profits.

That's not a bug — it's the product. For every setup we cause you to miss, we prevent multiple account-ending mistakes. The maths work in your favour over months and years, not days.

If you can accept that trade-off, FundedPal is built for you. If you can't, we're the wrong tool — and we'd genuinely rather you knew that before subscribing than after.

---

*Have a question that isn't answered here? Email questions@fundedpal.com — we read every one and update this FAQ regularly.*
