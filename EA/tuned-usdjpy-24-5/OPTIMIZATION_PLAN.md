# Optimization Plan — USDJPY 24/5 Tuned EA

## Objective

Find a parameter set with a genuinely positive edge (Profit Factor consistently
> 1) before considering live capital, position-size scaling, or session timing.
Current defaults are **not** there yet — see baseline below.

## Baseline (already measured — don't re-run this, it's the reference point)

Source: `usdjpy.xlsx` backtest report, `HF EA - USDJPY 24-5 Tuned`, USDJPY, M1,
ICMarketsSC-Demo, **2026.07.05–2026.07.11**, defaults (`Delta=0.5`, `Stop=10`,
`MaxDistance=7`, `MaxTrailing=4`, `TslTriggerPoints=15`, `TslPoints=10`,
`TrailType=1`, fixed `0.01` lot).

| Metric | Value |
|---|---|
| Trades | 9,281 |
| Win rate | 37.2% (breakeven win rate given avg win/loss size = 39.7%) |
| Profit Factor | 0.90 |
| Net P&L | -$43.61 |
| Sharpe | -5.0 |

Diagnosis from the deal log: win rate sits in the same 33–41% band in **every**
hour of the day — this is not a session-timing problem, it's the entry/exit
distances themselves (order too easily triggered by noise, stop too tight,
winners cut short). That's why this plan sweeps `Delta`, `Stop`, and `TslPoints`
rather than trading hours.

## Data windows (walk-forward — do not skip the out-of-sample step)

| Window | Dates | Status |
|---|---|---|
| Originally-planned in-sample | 2026.05.25 – 2026.07.04 | Not used — the sweep ended up running on the window below instead |
| **Sweep window (used for everything so far)** | 2026.07.05 – 2026.07.11 | Baseline through Round 2 all run here — see the correction below |
| **Out-of-sample (the only remaining clean holdout)** | 2026.07.12 – 2026.07.19 | Untouched — reserve this for the final check, run once |

A parameter set only counts as validated once it clears the acceptance bar
(below) on the 2026.07.12–07.19 holdout — not on the sweep window itself,
which has now been tuned against directly (see the correction further down).

## Strategy Tester settings

- Symbol: USDJPY
- Period: M1
- Model: **Every tick based on real ticks**
- Deposit: **$100**, matching the intended live account size — use this for
  every run (optimization + both out-of-sample validations) so results are
  comparable. Doesn't change $ P&L at fixed 0.01 lot, but keeps margin/lot
  behavior representative of the real account.
- Optimization criterion: **Profit Factor** (if only "Balance/Complex Criterion"
  are offered, use Complex Criterion but manually re-sort the results table by
  Profit Factor afterward — don't trust the default sort alone, see pitfalls below)
- Algorithm: **Slow complete algorithm** — the grid below is only 140
  combinations, small enough for exhaustive coverage rather than the genetic
  approximation

## ⚠ Data window correction

Every run logged below (baseline through Round 1) was actually run on
**2026.07.05–07.11**, not the intended in-sample window (2026.05.25–07.04).
That date range was originally marked "Out-of-sample #1" — a reserved
validation week. It no longer serves that purpose, since the sweep has now
been tuned against it directly. **2026.07.12–07.19 (Out-of-sample #2) is the
only untouched week left** — treat it as the single true holdout: run it once
on whatever final parameter set the sweep converges on, and don't iterate
against it. If that result looks bad, that's the answer, not a cue to keep
adjusting and re-testing on that same week.

## Results logged so far (window: 2026.07.05–07.11 — see correction above)

Running a full 140-combination grid by hand isn't practical, so this plan now
runs as a **manual local sensitivity sweep** instead (next section) — a small,
targeted set of single backtests around the best point found so far, rather
than the full grid. The full grid is kept further below as an alternative,
only worth it if MT5's Optimizer can run all combinations unattended on your
machine.

| Run | Delta | Stop | TslPoints | Trades | Win rate | Profit Factor | Net P&L |
|---|---|---|---|---|---|---|---|
| Baseline | 0.5 | 10 | 10 | 9,281 | 37.2% | 0.902 | -$43.61 |
| Run A | 1.0 | 15 | 15 | 6,678 | 43.7% | 0.936 | -$24.94 |
| Round 1 #2 | 1.5 | 15 | 20 | 5,505 | 38.0% | 0.933 | -$23.02 |
| Round 1 #4 | 1.5 | 25 | 15 | 5,092 | 49.1% | 0.940 | -$19.79 |
| Round 1 #6 | 1.0 | 25 | 20 | 4,180 | 44.2% | 0.946 | -$16.14 |
| Run B | 1.5 | 25 | 20 | 4,199 | 43.9% | 0.949 | -$15.34 |
| Round 1 #1 | 1.5 | 35 | 20 | 3,285 | 47.8% | 0.951 | -$12.48 |
| Round 1 #3 | 1.5 | 25 | 25 | 3,357 | 41.1% | 0.957 | -$11.40 |
| Round 1 #5 | 2.0 | 25 | 20 | 4,176 | 44.1% | 0.959 | -$12.11 |
| Round 2 #9 (combined) | 2.0 | 25 | 25 | 3,374 | 40.8% | 0.955 | -$12.13 |
| Round 2 #8 | 1.5 | 25 | 30 | 2,745 | 39.2% | 0.945 | -$13.59 |
| **Round 2 #7 (best so far)** | **2.5** | **25** | **20** | 4,257 | 43.6% | **0.968** | -$9.49 |

**Per-axis read** (isolating each parameter, holding the other two at
Delta=1.5/Stop=25/TslPoints=20 unless noted):

- **Stop** (15→25→35): PF 0.933 → 0.949 → 0.951 — flattened, gains shrank to
  near zero by 35. Not worth pushing further.
- **TslPoints** (15→20→25→**30**): PF 0.940 → 0.949 → 0.957 → **0.945** —
  peaked around 25 and reversed at 30. Stop pushing this axis; hold at 20
  (the value in the current best point) going forward.
- **Delta** (1.0→1.5→2.0→**2.5**): PF 0.946 → 0.949 → 0.959 → **0.968** —
  still climbing, no flattening yet, and by far the strongest lever. Trade
  count has stayed flat (~4,180-4,260) across the entire range tested —
  unlike Stop/TslPoints, widening Delta isn't costing trade frequency.
- **Combining Delta=2.0 with TslPoints=25 (Round 2 #9) underperformed both
  of them individually** (0.955 vs 0.959 and 0.957) — the two axes aren't
  simply additive. Don't assume the best individual values automatically
  combine into the best joint result; re-check combinations once Delta's
  peak is found rather than stacking blindly.

Still below both targets (PF > 1.0, let alone the 1.15 bar) even at the best
point found, but climbing steadily.

## Round 3 (2 runs) — keep pushing the one direction still working

| # | Delta | Stop | TslPoints | Why |
|---|---|---|---|---|
| 10 | **3.0** | 25 | 20 | Does Delta keep climbing past 2.5? |
| 11 | **3.5** | 25 | 20 | Bracket further out in one round rather than waiting on #10 first |

**Watch `MaxDistance` (fixed at 7) as Delta keeps climbing.** `Delta` sets how
far the pending order sits from price; `MaxDistance` is the outer bound that
triggers a full repositioning if price moves too far away — `Delta` needs to
stay meaningfully below it for that logic to behave sensibly. At Delta=2.5
we're using about a third of that headroom; 3.0-3.5 is still fine, but if
Delta keeps climbing past there, `MaxDistance` will need to be raised in
tandem rather than left fixed (this was already earmarked as a Phase 2
parameter — may end up needed sooner than expected).

Same Strategy Tester settings as before (USDJPY, M1, Every tick based on real
ticks) — run these on **2026.07.05–07.11** for consistency with everything
logged above, not the out-of-sample week.

**Correction:** despite being noted here, Rounds 1-3 were actually all run at
$100,000 deposit with `Fixed_Lots`, not the real $100 target — same gap that
was caught and fixed on the gold side. Round 4 below addresses this.

### Round 3 results

| # | Delta | Stop | TslPoints | Trades | Win rate | Profit Factor | Net P&L |
|---|---|---|---|---|---|---|---|
| **10 (best so far)** | **3.0** | 25 | 20 | 4,309 | 43.4% | **0.975** | -$7.46 |
| 11 | 3.5 | 25 | 20 | 4,358 | 43.6% | 0.972 ↓ | -$8.32 |

Delta peaked around 3.0 and dipped slightly at 3.5 — same peak-then-reverse
shape seen on the TslPoints axis, though at 3.5 Delta is now half of
`MaxDistance` (7), so this dip may be a genuine price-action peak or may be
an artifact of crowding that headroom — can't fully separate the two without
also varying MaxDistance. **Current best point: Delta=3.0, Stop=25,
TslPoints=20, PF=0.975.** Hold here rather than pushing Delta further without
opening up MaxDistance too.

Still below Profit Factor 1.0 even at the best point across all 11 runs. This
parameter family (Delta/Stop/TslPoints) is topping out somewhere around
0.97-0.98 — a real signal to open up Phase 2
(`MaxDistance`/`MaxTrailing`/`TslTriggerPoints`) or revisit the hour-of-day
pattern noticed earlier, rather than continuing to fine-tune the same three
inputs for diminishing returns.

## Round 4 — real-conditions risk check (before any Phase 2 work)

The gold exploration (`EA/gold-xauusd-exploration/`) found that `RiskPercent`-
based position sizing can turn a near-breakeven edge into a severe boom-bust
equity swing (one gold test peaked at ~210x the starting balance before
crashing to near-total ruin in the same week). USDJPY's edge is similarly
close to breakeven (PF 0.975 at best) and has never actually been tested at
the real $100 deposit with risk-based sizing — every prior round used
$100,000 and `Fixed_Lots`. This needs checking before any further parameter
tuning is worth trusting.

Run the current best point (Round 3 #10) unchanged, except:

| Input | Value |
|---|---|
| Deposit | **$100** |
| `LotType` | **Pct_of_Balance** |
| `RiskPercent` | **1** |
| `Delta` / `Stop` / `MaxDistance` / `MaxTrailing` | 3.0 / 25 / 7 / 4 (unchanged) |
| `MaxSpread` | 18 (unchanged) |
| `TslTriggerPoints` / `TslPoints` | 15 / 20 (unchanged) |

Window and hours unchanged (2026.07.05–07.11, StartHour=1/EndHour=23).

**What to watch for**: Profit Factor should be roughly similar to 0.975
(sizing scheme doesn't change the underlying win/loss pattern much), but the
**path** might look completely different — check Balance Drawdown Maximal
and the implied peak balance vs. final balance for the same boom-bust shape
seen on gold. If it shows up here too, that's a general risk of
`RiskPercent`-based compounding on any near-flat edge, not something
specific to gold — worth addressing (e.g. a non-compounding, fixed-dollar-
risk-per-session scheme) before Phase 2 or any live use, on either pair.

## A note on the Inputs tab labels

MT5 displays each input's trailing `//` comment as its row label in the Inputs
tab — it **replaces** the variable name, it doesn't add to it. Several inputs
picked up long descriptive comments during tuning, so their on-screen label is
no longer the short variable name. Use the "Label shown in Inputs tab" column
below to find the right row — the code variable name is there for reference
against `NOTES.md`, but it won't be what you see on screen.

## Alternative: full grid via MT5 Optimizer (only if it can run unattended)

Skip this section and use the manual sweep above unless MT5's Optimizer tab
is genuinely usable on your setup to run all combinations in one unattended
pass — in which case this is more thorough than the manual sweep since it
also captures interaction effects between the three parameters, not just
their individual effects.

Check the optimize checkbox for exactly these three rows:

| Code variable | Label shown in Inputs tab | Start | Step | Stop |
|---|---|---|---|---|
| `Delta` | "ORDER DISTANCE" | 0.5 | 0.5 | 2.5 |
| `Stop` | "Stop Loss size" | 10 | 5 | 40 |
| `TslPoints` | "Trailing distance in points (10 points = 1 pip)" | 10 | 5 | 25 |

5 × 7 × 4 = **140 combinations**.

## Inputs to hold fixed (Phase 1)

Leave every other row's optimize checkbox **unchecked**, with Value set as below:

| Code variable | Label shown in Inputs tab | Value |
|---|---|---|
| `InpMagic` | "Magic Number" | 12345 |
| `Slippage` | "Widened from 1 for 24/5 volatility variance" | 2 |
| `StartHour` | "START TRADING HOUR (server time; blocks only the midnight rollover hour)" | 1 |
| `EndHour` | "END TRADING HOUR (server time)" | 23 |
| `Secs` | "ORDER MODIFICATIONS (Should be same as TF)" | 60 |
| `AvoidWeekendGap` | "Block NEW pending orders near weekly open/close" | true |
| `FridayCutoffHour` | "Server hour (Fri) after which no new orders are placed" | 21 |
| `SundayResumeHour` | "Server hour (Sun) before which no new orders are placed" | 1 |
| `LotType` | "Type of Lotsize calculation" | Fixed_Lots |
| `FixedLot` | "Fixed Lots 0.0 = MM" | 0.01 |
| `RiskPercent` | "Risk MM%" | 0.0 |
| `MaxDistance` | "THETA (Max order distance)" | 7.0 |
| `MaxTrailing` | "COS (Start of Trailing Stop)" | 4.0 |
| `MaxSpread` | "Max Spread Limit in points (~1.8 pips starting ceiling for USDJ..." | 18 |
| `TrailType` | "Type of Trailing StopLoss" | Scalp_Trail |
| `TslTriggerPoints` | "Points in profit before trailing starts (10 points = 1 pip)" | 15 |

Below the "If Trailing by..." groups (`PrvCandleN`, `FMAperiod`, `MA_Mode`,
`MA_AppPrice`) don't matter this run — `TrailType=Scalp_Trail` means those
inputs aren't used, leave them at whatever MT5 shows by default.

*(Phase 2, only if Phase 1 doesn't clear the bar: widen the sweep to
`MaxDistance`, `MaxTrailing`, and `TslTriggerPoints` — hold off on this until
Phase 1 results are in, to avoid combinatorial explosion and overfitting on a
single pass.)*

## Acceptance criteria — a result only counts if ALL of these hold

1. **Profit Factor > 1.15** on the sweep window (2026.07.05–07.11) — extra
   margin above 1.0, since live spread/slippage will be worse than backtest
   and eats into any edge.
2. **Profit Factor > 1.0 on the 2026.07.12–07.19 holdout**, run once, not
   iterated on.
3. **Trade count ≥ 500** in both windows. A parameter set that "wins" mainly
   by barely trading (e.g. `Delta=2.5` producing 40 trades total) is not a
   real result — it's too small a sample to trust, discard it even if PF
   looks great.
4. **Max relative drawdown < 10%** in both windows.
5. Win rate and average-win/average-loss ratio should both be reported, not just
   PF — two very different parameter sets can produce the same PF for different
   (and differently reliable) reasons.

## Pitfalls to watch for

- **Don't pick the single best in-sample row and stop.** Optimizers reliably
  find noise that looks like edge on 140 combinations tested against one window.
  The out-of-sample step is not optional.
- **Watch trade count on every candidate**, not just the top-line PF — see
  criterion 3 above.
- **Re-verify `MaxSpread=18` is still appropriate** across whatever date range
  you test — if spread conditions were different back in May/June, some
  combinations may be getting filtered differently than they would be now.

## Reporting back

For the manual sweep: export each single-test Strategy Tester report the same
way as `usdjpy.xlsx`/`usdjpy1.xlsx`/`usdjpy2.xlsx` (the standard report, not a
screenshot) for each of the 6 runs — that lets me re-run the same
hour-by-hour / breakeven-win-rate analysis on each one. Once the sweep
converges on a best point, export the same report for that final parameter
set on both out-of-sample windows.

If using the full-grid Optimizer instead: export the Optimization Results tab
to CSV (right-click the results grid → "Export to CSV") for each window.
