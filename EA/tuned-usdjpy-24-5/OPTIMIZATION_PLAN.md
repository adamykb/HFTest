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

| Window | Dates | Purpose |
|---|---|---|
| **In-sample (optimize here)** | 2026.05.25 – 2026.07.04 | Run the optimizer on this range only |
| **Out-of-sample #1** | 2026.07.05 – 2026.07.11 | Same week as the baseline above — direct comparison against PF 0.90 |
| **Out-of-sample #2** | 2026.07.12 – 2026.07.19 | Fully unseen week, confirms #1 wasn't luck |

A parameter set only counts as "fixed" if it clears the bar (below) on **both**
out-of-sample windows independently — not just on average, and never on the
in-sample window alone. If your broker's history doesn't reach back to
2026.05.25, shift the whole three-window block back in time but keep the same
relative structure and lengths.

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

## A note on the Inputs tab labels

MT5 displays each input's trailing `//` comment as its row label in the Inputs
tab — it **replaces** the variable name, it doesn't add to it. Several inputs
picked up long descriptive comments during tuning, so their on-screen label is
no longer the short variable name. Use the "Label shown in Inputs tab" column
below to find the right row — the code variable name is there for reference
against `NOTES.md`, but it won't be what you see on screen.

## Inputs to optimize (Phase 1)

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

1. **Profit Factor > 1.15** in-sample (extra margin above 1.0, since live spread/
   slippage will be worse than backtest and eats into any edge).
2. **Profit Factor > 1.0 on both out-of-sample windows independently.**
3. **Trade count ≥ 500** in each window. A parameter set that "wins" mainly by
   barely trading (e.g. `Delta=2.5` producing 40 trades total) is not a real
   result — it's too small a sample to trust, discard it even if PF looks great.
4. **Max relative drawdown < 10%** in every window.
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

Export the Optimization Results tab to CSV (right-click the results grid →
"Export to CSV") for each window rather than a screenshot — that lets me re-run
the same hour-by-hour / breakeven-win-rate style analysis on whatever you bring
back. Also export the standard Strategy Tester report (like `usdjpy.xlsx`) for
whichever single parameter set ends up best, for both out-of-sample windows.
