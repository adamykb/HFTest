# Gold (XAUUSD) — Validation Plan

## Why validate before optimizing

The one gold result we have (PF 1.244, Sharpe 104.7, 14-second average hold,
83% tick quality) is such an outlier against every USDJPY run (PF capped
~0.97-0.98, Sharpe -5.0, minutes-long holds, 100% tick quality) that it needs
to survive a few checks before it's worth building a full sensitivity sweep
around, the way we did for USDJPY. If it's real, the checks below confirm
that cheaply. If it's a tick-modeling artifact, better to find out in 2-3
runs than after a full sweep built on top of it.

## Step 1 — Two checks, no backtest needed yet

1. **Tick quality across candidate weeks.** In Strategy Tester, check what
   "History Quality" XAUUSD actually achieves for both 2026.07.05–07.11
   (already used once, came back 83%) and 2026.07.12–07.19 (untouched). If
   neither reaches something close to 100%, try adjacent weeks until you find
   one that does — report back the quality % for whichever weeks you check.
   If XAUUSD genuinely can't get clean tick history from this broker for any
   nearby period, that's important information on its own (a strategy that
   can't be reliably backtested is also harder to trust live).
2. **Gold's actual point/spread scale.** Open XAUUSD in Market Watch (or the
   Symbols window) and note: `_Point` value (price precision — e.g. is
   2650.32 a 2-decimal quote with Point=0.01?), and the current live spread
   in points. `MaxSpread=18` (~$0.18) was reused from USDJPY without
   checking whether that's tight, loose, or nonsensical for gold — report
   these back so the filter can be set from real numbers instead of a guess.

## Step 2 — Three backtests, once Step 1 gives usable numbers

Same Strategy Tester setup as always (M1, Every tick based on real ticks,
$100 deposit) — but only proceed once you've found a week with high tick
quality for XAUUSD from Step 1. Call that week "the validated week" below.

| # | Purpose | Delta | Stop | TslPoints | MaxSpread | Window |
|---|---|---|---|---|---|---|
| G1 | Does the original result hold with clean ticks? | 3.5 | 25 | 20 | 18 (or recalibrated from Step 1) | The validated week |
| G2 | True gold-native baseline (not USDJPY's tuned endpoint) | 0.5 | 10 | 10 | recalibrated from Step 1 | The validated week |
| G3 | Reproducibility check — same combo, different data | 3.5 | 25 | 20 | 18 (or recalibrated) | Whichever candidate week G1 didn't use |

**G1** isolates the tick-quality question directly — if Profit Factor holds
up anywhere near 1.24 with real ticks, that's a strong signal this is a real
effect, not an artifact. If it collapses toward the 0.9-1.0 range we've seen
on USDJPY, that tells us the original number was mostly the missing 17% of
ticks.

**G2** gives us a genuine starting point for gold on its own terms, the same
way `Delta=0.5/Stop=10/TslPoints=10` was the real USDJPY baseline — useful
regardless of how G1 turns out, since a full sweep (if warranted) needs a
real anchor to sweep from, not USDJPY's already-tuned endpoint.

**G3** checks the result isn't specific to one week's price action — same
logic as the out-of-sample step in the USDJPY plan, just earlier in the
process this time since we're validating before optimizing rather than after.

## What happens next, depending on results

- **G1 holds up (PF still meaningfully > 1) and G3 reproduces it**: gold is
  a real lead. At that point I'd set up a full sensitivity sweep for gold
  from G2's baseline, the same process used for USDJPY (`Round 1/2/3`-style,
  isolating one parameter at a time).
- **G1 collapses toward the USDJPY-like range**: the original number was
  mostly the tick-quality artifact. Not necessarily a dead end — G2's clean
  baseline is still useful information — but expectations reset to "gold
  behaves similarly to USDJPY," not "gold is dramatically better."
- **Tick quality can't be improved past ~80% for XAUUSD on this broker at
  all**: worth pausing gold and asking whether a different broker/data
  source has better XAUUSD tick history before investing further here.
