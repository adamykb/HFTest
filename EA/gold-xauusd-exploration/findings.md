# Gold (XAUUSD) — Findings Log

Running log of notable backtest results and what they mean. Newest entries
first. See `VALIDATION_PLAN.md` for the checks this is working through, and
`NOTES.md` for the original preliminary result.

**⚠ Do not run `Delta=30/Stop=250/MaxSpread=50/TslTriggerPoints=100/TslPoints=150`
with `LotType=0` (Fixed_Lots) on a real account — see the 2026-08-12
`gold_4.xlsx` entry below. This combination produced a 98.94% drawdown and a
single trade losing 27% of a $100 account.**

**⚠ `LotType=1` (Pct_of_Balance) fixes the single-trade-too-big problem but
introduces a different risk: compounding position size on a near-zero-edge
strategy can still produce huge boom-bust swings — see the 2026-08-12
`G1.xlsx` entry below (peak balance >$537 from a $100 start, then crashed to
-$66.75 net). Watch for this pattern on every future risk-sized run, not
just this specific parameter set.**

---

## 2026-08-12 — `G1.xlsx`: near-zero edge confirmed, but compounding turned it into a boom-bust cycle

### Settings

First cleanly controlled run per the rewritten `VALIDATION_PLAN.md`: window
2026.07.19–07.25 (100% real ticks), StartHour=1/EndHour=23, $100 deposit,
`LotType=1` (Pct_of_Balance), `RiskPercent=1` — the small-scale parameters
from the original gold result (Delta=3.5, Stop=25, MaxDistance=7,
MaxTrailing=4, MaxSpread=18, TslTriggerPoints=15, TslPoints=20, Slippage=2).

### Result

| Metric | Value |
|---|---|
| Total Net Profit | **-$66.75** |
| Balance Drawdown Maximal | **$513.37 (95.53%)** |
| Implied peak balance | ~$537 (from $100 start) |
| Profit Factor | 0.993 |
| Trades | 36,785 |
| Average hold time | 0:00:17 |
| Largest single loss | -$7.80 |
| Margin Level (end) | 150.20% |

### Two things this confirms

1. **Profit Factor is essentially exactly breakeven again** (0.993 here,
   1.0036 in `gold_2.xlsx` — same Delta/Stop/TslPoints, completely different
   position-sizing and hours setup in each). Two independent tests
   converging on "coin flip" is a real signal: these specific parameter
   values likely have no real edge on gold, positive or negative.
2. **`RiskPercent`-based sizing does not prevent wild equity swings on a
   thin edge — it can amplify them.** It fixed the single-trade-too-big
   problem from `gold_4` (largest loss here: -$7.80, contained relative to
   how large the balance had grown by then), but because lot size scales
   with *current* balance, a winning stretch compounds into bigger bets,
   which then compounds losses just as much when the stretch reverses. The
   account grew over 5x, then gave almost all of it back, netting a loss —
   not because of a few catastrophic trades (max losing streak was only
   -$4.44 across 9 trades) but because of prolonged multi-thousand-trade
   drift at a constantly-changing position size.

### Why this matters beyond this one test

This is a concrete demonstration of the risk flagged early in the
conversation about scaling lot size as capital grows: **compounding
position size only works safely on top of a validated, robustly positive
edge.** On a near-zero-edge strategy, it doesn't reliably grow an account —
it turns ordinary drift into much larger swings in both directions. Watch
for this same boom-bust shape on G2 and G2b, not just this parameter set —
if it shows up regardless of Delta/Stop/TslPoints, that points to the
compounding mechanism itself (or the `RiskPercent` value) as the thing to
address, separate from parameter tuning.

---

## 2026-08-12 — `gold_4.xlsx`: 98.94% drawdown, account nearly wiped out

### Settings

Same as `gold_3.xlsx` below, except `StartHour=1, EndHour=23` (standard
hours, correctly reset this time). Window still 2026.07.05–07.11, **History
Quality still 83% real ticks** — the clean 100%-quality week (2026.07.19–07.25)
still hasn't been used for a full-day gold test.

### Result — the most severe finding in this entire exploration

| Metric | `gold_3` (StartHour=15/EndHour=17) | `gold_4` (StartHour=1/EndHour=23) |
|---|---|---|
| Net P&L | +$100.15 | **-$98.37** |
| Balance Drawdown Maximal | $77.02 (57.56%) | **$152.85 (98.94%)** |
| Largest single loss | -$22.02 | **-$27.61** |
| Margin Level | 409.80% | **40.25%** |
| Profit Factor | 1.248 | 0.890 |

**98.94% drawdown** means the account came within roughly a dollar of zero
at its worst point. **Margin Level 40.25%** is low enough that most brokers'
automatic stop-out (commonly triggered somewhere in the 20-50% range) would
very plausibly have force-liquidated everything at the worst possible moment
in a live account — not a scary-but-recoverable dip, a real margin-call
scenario.

### What changed between gold_3 and gold_4, and what that means

The *only* difference is trading hours — narrow 15-17 window vs. the full
1-23 day, same parameters, same week. That means gold_3's +$100.15 wasn't a
real edge; it was a narrow window that happened to avoid whatever happened
during the rest of the day. Full-day exposure reveals the actual risk
profile, and it's severe.

### The structural cause, independent of week/tick-quality confounds

`LotType=0` (Fixed_Lots) at `0.01` does not scale to account size.
`Stop=250` (a wide multiplier of live spread) combined with that fixed lot
produced a single trade that lost $27.61 — over a quarter of a $100
account, in one trade. No choice of week or trading hours fixes this; the
position size itself is too large for the account once the stop is this
wide.

### Required before any further gold testing

**Switch `LotType` from `0` to `1` (Pct_of_Balance) with a conservative
`RiskPercent`, e.g. 1-2%.** The EA already supports this (`calcLots()`), it
has just never been enabled in any test run so far (USDJPY or gold). This
caps a single trade's maximum loss as a percentage of the account by
design, rather than leaving it to whatever a fixed lot happens to produce
against whatever the stop distance turns out to be. Parameter sweeping
should not resume until this is in place — the risk here isn't specific to
these particular Delta/Stop values, it's inherent to fixed-lot sizing on a
$100 account with any sufficiently wide stop.

---

## 2026-08-12 — `gold_3.xlsx`: +$100.15 net, but with a 57.56% drawdown and a -$22 single trade

### Settings

| Input | Value |
|---|---|
| Deposit | $100 |
| Window | 2026.07.05–07.11 |
| StartHour / EndHour | 15 / 17 (narrow, arbitrary — not the standard 1-23) |
| Delta / MaxDistance / Stop / MaxTrailing | 30 / 100 / 250 / 4.0 |
| MaxSpread | 50 |
| Slippage | 20 |
| TslTriggerPoints / TslPoints | 100 / 150 |
| History Quality | **83% real ticks** |

### Headline result

- **Total Net Profit: +$100.15** on a $100 deposit — the account slightly
  more than doubled over the week.
- Profit Factor 1.248, win rate 56.8%, 498 trades.

### The numbers that can't be separated from that headline

- **Balance Drawdown Maximal: $77.02 (57.56%).** At its worst point during
  this same week, the account was down to roughly $23 before recovering to
  end at +$100.15. Profit Factor and net profit only describe where the week
  ended, not the path — and the path here came within a hair of wiping the
  account out.
- **Largest loss trade: -$22.02** — a single trade costing over a fifth of
  a $100 starting balance in one shot.
- Equity Drawdown Maximal: $69.91 (54.53%), consistent with the balance
  figure.

### Structural issue this exposes, independent of which parameters are used

`LotType=0` (Fixed_Lots) at `0.01` does not scale with account size at all.
The same lot size that was invisible risk on the earlier $100,000-deposit
tests (see `tuned-usdjpy-24-5/`) becomes a 20%+ single-trade swing on a real
$100 account. This is a position-sizing problem, not a parameter-tuning one
— fixed lot size and a $100 account don't mix safely regardless of how
Delta/Stop/TslPoints end up tuned. Worth revisiting `LotType`
(Pct_of_Balance/Pct_of_Equity) with a conservative `RiskPercent` before any
live use.

### Why this specific result isn't trustworthy as a forward estimate yet

Three things changed at once relative to the one clean-data reference point
we have (`gold_2.xlsx`, 100% real ticks, window 2026.07.19–07.25, standard
hours):

1. **Tick quality is back down to 83%** — same artifact-risk concern as the
   very first gold test (`NOTES.md`), not the 100%-quality week.
2. **`StartHour=15/EndHour=17`** — the narrow window from the previous test
   was never reset to the standard 1-23.
3. The wider Delta/Stop/TslPoints values are new on top of both of the
   above.

Because tick quality, trading hours, and the parameter set all changed
together, it's not possible to tell from this result alone whether the
+$100 profit (or the 57.56% drawdown) comes from the parameter change, the
lower-quality data, the narrow window, or some combination. A clean read
requires isolating the parameter change as the only variable.

### Recommended next step

Re-run the same widened parameters (Delta=30/MaxDistance=100/Stop=250/
MaxSpread=50/Slippage=20/TslTriggerPoints=100/TslPoints=150) on:

- **Window: 2026.07.19–07.25** (the one confirmed 100%-real-tick week)
- **StartHour=1, EndHour=23** (standard, not the narrow arbitrary window)
- Deposit: $100 (keep — this is what exposed the drawdown risk above and
  should stay the default for every future run)

If both the profit *and* the 57%+ drawdown persist under clean, controlled
conditions, that's real information about this parameter set's risk profile
on gold. If either changes substantially, that tells us how much of this
result was the confounds rather than the parameters.
