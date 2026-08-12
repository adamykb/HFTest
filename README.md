# HFTest

MetaTrader 5 Expert Advisor sources and tuning work for a high-frequency
straddle/grid scalper — **AAHFT** — plus per-symbol tuning work (USD/JPY,
gold/XAUUSD) built on top of it.

## What this EA does

The base strategy places a `BUY_STOP` above price and a `SELL_STOP` below it,
continuously repositioning both pending orders to stay a small, spread-relative
distance from the current price. Whichever side gets triggered by a breakout
becomes a live position, managed with a tight stop-loss and a choice of trailing-
stop methods (fixed scalp trail, previous-candle, fast MA, or Ichimoku Tenkan-sen).
Because entry/exit distances scale with live spread rather than fixed pips, it's
very sensitive to transaction costs — spread, commission, and slippage can dominate
the outcome if the filters around it aren't calibrated correctly.

## Repo layout

```
EA/
  original/                     Unmodified source as received
    ...v1 - Cleaned up by Mr CapFree.mq5      (single trailing-stop method)
    ...v2 - Including new Trailing Stops.mq5  (adds selectable TSL methods)
  AAHFT/                        Symbol-agnostic build (current, no pair name baked in)
    AAHFT.mq5
  tuned-usdjpy-24-5/             USD/JPY-specific tuning history (built from v2)
    HF EA - USDJPY 24-5 Tuned.mq5
    NOTES.md                    What changed and why
    OPTIMIZATION_PLAN.md        Parameter sensitivity sweep, round by round
  gold-xauusd-exploration/       Gold/XAUUSD exploration, currently unvalidated
    NOTES.md
    VALIDATION_PLAN.md          Checks required before trusting the initial result
```

`AAHFT.mq5` carries the same input defaults as the USD/JPY tuned variant, but
with the symbol-specific naming and comments stripped out — inputs like
`MaxSpread` still need calibrating per symbol/broker before trusting them on
whatever pair you attach it to. Per-symbol tuning work and its reasoning live
in the `tuned-usdjpy-24-5/` and `gold-xauusd-exploration/` folders.

## Status

Actively backtested via MetaTrader 5 Strategy Tester (M1, Every tick based on
real ticks). USD/JPY has been through several rounds of a manual parameter
sensitivity sweep — see `EA/tuned-usdjpy-24-5/OPTIMIZATION_PLAN.md` for the
full history and current best result. Gold/XAUUSD produced one promising but
unvalidated result — see `EA/gold-xauusd-exploration/VALIDATION_PLAN.md`
before trusting it. **Nothing here has been run live** — forward-test on a
demo account once a symbol clears its validation/acceptance bar, before
considering real money.
