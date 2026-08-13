# Gold (XAUUSD) exploration — preliminary, NOT validated

**Status (2026-08-12):** this is the only result so far, and it's flagged as
unvalidated below. Active work has moved to
[`VALIDATION_PLAN.md`](./VALIDATION_PLAN.md) — checking tick quality and
gold's real spread scale, then three confirmation backtests, before any full
optimization sweep is built for this pair. Later, higher-signal results
(including a run that doubled a $100 account but with a 57%+ drawdown along
the way) are logged in [`findings.md`](./findings.md) as they come in.

## What was run

Same file as `EA/tuned-usdjpy-24-5/HF EA - USDJPY 24-5 Tuned.mq5`, no code
changes — just the Symbol switched to XAUUSD in Strategy Tester. Inputs
reused as-is from the USDJPY sweep's Round 3 #11 (Delta=3.5, Stop=25,
TslPoints=20, MaxSpread=18, same window 2026.07.05–07.11, M1, $100 deposit).

## Result

| Metric | Value |
|---|---|
| Trades | 43,979 |
| Win rate | 58.9% |
| Profit Factor | **1.244** |
| Net P&L | **+$2,301.25** |
| Sharpe Ratio | 104.7 |
| Average hold time | 0:00:14 |
| History quality | **83% real ticks** |

On paper this clears every acceptance bar set for the USDJPY sweep, by a wide
margin. **Do not treat this as a validated result yet** — see below.

## Why this needs verification before acting on it, not celebration

1. **83% tick quality, not 100%.** Every USDJPY test run so far has been
   100% real ticks. Here, 17% of the price path was synthetically generated
   by MT5's tick modeler rather than real broker data — exactly the failure
   mode that inflates results for a tick-sensitive strategy like this one.
2. **14-second average hold time.** The USDJPY runs held for minutes. A
   strategy resolving in 14 seconds is reacting almost entirely to
   fine-grained intra-tick movement — the timescale where the missing 17%
   of real ticks (now interpolated) has the most influence and the least
   resemblance to real fills.
3. **Sharpe 104.7 / Z-score 40 / Recovery Factor 57**, against a Sharpe of
   -5.0 on every single USDJPY run. A jump of this scale on one week of data
   is a textbook signature of a backtest artifact, not a sudden real edge.
4. **None of the point-based inputs were calibrated for gold.** `MaxSpread=18`
   (~$0.18) and the trailing-stop point values were tuned for USDJPY's price
   scale and typical spread. Getting 43,979 trades through an $0.18 spread
   filter on gold means either the broker's live gold spread really is that
   tight, or the filter isn't behaving as expected on this symbol's point
   scale — unverified either way.

## Before trusting this result

- [ ] Re-run with 100% real tick quality if the broker's history supports it
      for this period. If it doesn't, that's itself an important constraint
      on how reliable *any* gold backtest can be here.
- [ ] Check XAUUSD's actual live spread in Market Watch and recalibrate
      `MaxSpread` and the trailing-stop point inputs from scratch for gold's
      real scale, rather than reusing USDJPY's tuned numbers.
- [ ] Re-run on a different, untouched week — everything above is on
      2026.07.05–07.11 like every other test so far; a result this large
      deserves a check on unfamiliar data before drawing any conclusion.

Only after those checks would this be worth building out into a full
sweep the way USDJPY got one.
