# AAHFT — Changelog

## v2 — `AAHFT v2 (Session PL Lock).mq5`

Adds a **session profit/loss lock**: two new inputs, `ProfitTargetBalance`
(default 5000) and `MaxLossBalance` (default 50). Once the account balance
reaches the profit target or falls to the loss floor, the EA immediately:

1. Cancels every pending order for this symbol/magic number.
2. Closes every open position for this symbol/magic number.
3. Stops opening any new orders for the rest of the run — enforced via
   `IsTradingAllowed()`, so it can't be bypassed by any other code path.

The lock resets automatically whenever the EA re-initializes (a new
backtest run, or reattaching live with a fresh deposit) — it's intentionally
per-session, not persistent across restarts, matching a daily
reset-to-$100-and-go-again workflow. Set either input to `0` to disable it.

**Why this exists**: backtesting on gold found that `RiskPercent`-based
compounding can turn a near-breakeven edge into a severe boom-then-crash
equity swing within a single week — one test peaked at ~210x the starting
balance before crashing to near-total ruin in the same run (see
`EA/gold-xauusd-exploration/findings.md`, the G1/G2/G2b entries). "Stop once
we're up $X" or "cap the loss at $Y" isn't something a human can reliably
enforce by watching a screen at this EA's trading frequency (positions can
close in single-digit seconds) — this makes that rule mechanical instead of
aspirational.

Also changes the **default** `LotType` from `Fixed_Lots` to `Pct_of_Balance`
(`RiskPercent=1`), since fixed-lot sizing was found to produce single trades
costing 20%+ of a $100 account (`gold_4.xlsx` in the same findings doc) —
the old default no longer reflects what testing has shown to be safe.

**Not yet tested**: this file hasn't been backtested itself. Compile and
verify in MetaEditor/Strategy Tester before relying on it — in particular,
confirm the lock actually fires at the expected balance level and that
`trade.PositionClose()`/`trade.OrderDelete()` succeed reliably under the
same tick conditions used elsewhere in this repo's testing.

## v1 — `AAHFT.mq5`

Symbol-agnostic rebrand of `EA/tuned-usdjpy-24-5/HF EA - USDJPY 24-5
Tuned.mq5` — no functional changes, only USDJPY-specific naming/comments
removed. See that file's `NOTES.md` for the tuning history this inherits.
