# Gold (XAUUSD) — Validation Plan

## Where things stand (updated 2026-08-12)

Four backtests in (`gold.xlsx`, `gold_2.xlsx`, `gold_3.xlsx`, `gold_4.xlsx`
— full detail in `findings.md`), here's what's actually been established:

- **Tick quality is solved**: 2026.07.19–07.25 gets **100% real ticks** for
  XAUUSD on this broker (confirmed in `gold_2.xlsx`). Use this as "the
  validated week" for everything below — no need to keep hunting for a good
  week.
- **A serious, blocking risk was found**: `gold_4.xlsx` produced a **98.94%
  drawdown** (near-total account wipeout) and a single trade losing 27% of
  a $100 account, using `LotType=0` (Fixed_Lots). This is a structural
  problem, not a bad parameter choice — fixed lot size doesn't scale to
  account size at all. **No further gold testing should use `LotType=0`.**
- **Every test so far has changed multiple variables at once** (tick
  quality, trading hours, and the parameter set, across `gold_2/3/4`), so
  none of the original Step 2 checks (G1/G2/G3 below) have actually been
  completed cleanly yet. Restarting them properly, with sizing fixed first,
  is what this plan now covers.

## Step 0 (new, blocking) — switch to risk-based position sizing

Before any other test: change `LotType` from `0` to `1` (Pct_of_Balance)
with `RiskPercent = 1`. This caps a single trade's maximum loss at roughly
1% of current balance by design (the EA's `calcLots()` already implements
this — it's just never been turned on in any test run, USDJPY or gold).
Every gold test from here on should use this, not a fixed lot.

## Step 1 — status

1. ~~Tick quality across candidate weeks~~ — **done**. 2026.07.19–07.25 =
   100% real ticks. Use this week going forward.
2. **Gold's actual tick size / tick value / live spread — still outstanding.**
   Open XAUUSD's Symbol Specification in MT5 and send: Tick Size, Tick
   Value, current Spread. Every gold input so far (`MaxSpread=50`,
   `Delta=30`, `Stop=250`, etc.) is still a reasoned estimate, not something
   calculated from real numbers. Not blocking Step 2 below, but needed
   before trusting any of these values as calibrated rather than guessed.

## Step 2 — controlled test sequence (all four conditions fixed except the parameter set)

Every run below uses: **Window = 2026.07.19–07.25**, **StartHour=1,
EndHour=23**, **Deposit = $100**, **LotType=1, RiskPercent=1** (Step 0).
Only the trade-setting parameters vary between rows — this is the
single-variable control that `gold_2/3/4` didn't have.

| # | Purpose | Delta | Stop | MaxDistance | MaxSpread | TslTriggerPoints | TslPoints |
|---|---|---|---|---|---|---|---|
| G1 | Does the original (small-scale) result hold under full control? | 3.5 | 25 | 7 | 18 | 15 | 20 |
| G2 | True gold-native baseline | 0.5 | 10 | 7 | 50 | 15 | 20 |
| G2b | The widened set from gold_3/gold_4, now risk-sized | 30 | 250 | 100 | 50 | 100 | 150 |

**G1** re-does what `gold_2.xlsx` almost was, minus its narrow-hours
confound — the cleanest read yet on whether the original 1.24 Profit
Factor reflects anything real.

**G2** still hasn't been run at all — the actual gold-native starting point,
same role `Delta=0.5/Stop=10/TslPoints=10` played for USDJPY.

**G2b** re-tests the exact parameters that caused the near-wipeout in
`gold_4`, but now with risk-based sizing — this directly tests whether that
was purely a position-sizing problem (in which case it should now show a
much smaller, bounded drawdown) or whether the parameters themselves are
bad regardless of sizing (in which case Profit Factor should still look
weak even with drawdown under control).

## What happens next, depending on results

- **Any row clears Profit Factor > 1 with a sane drawdown (well under the
  10% bar) under full control**: that's a real candidate — reproduce it on
  a second, different week before trusting it further (the G3-style check
  from the original plan, still valid, just deferred until something here
  looks worth reproducing).
- **All three still show weak Profit Factor once sizing is fixed**: the
  earlier "gold looks great" signal was mostly noise/confounds after all —
  reset expectations to "gold behaves similarly to USDJPY" rather than
  "gold is dramatically better."
- **G2b still shows a large drawdown even with RiskPercent=1**: that means
  the parameters themselves (not just the sizing) are the problem — Stop=250
  combined with this EA's straddle mechanics may simply be too volatile for
  gold regardless of position size, and would need its own sensitivity
  sweep from G2's baseline rather than reusing the gold_3/gold_4 numbers.
