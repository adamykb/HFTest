# Gold (XAUUSD) — Findings Log

Running log of notable backtest results and what they mean. Newest entries
first. See `VALIDATION_PLAN.md` for the checks this is working through, and
`NOTES.md` for the original preliminary result.

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
