# HFTest

MetaTrader 5 Expert Advisor sources and tuning work for a high-frequency
straddle/grid scalper, plus a variant tuned for running USD/JPY around the clock.

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
  tuned-usdjpy-24-5/             USD/JPY, 24/5 tuned variant (built from v2)
    HF EA - USDJPY 24-5 Tuned.mq5
    NOTES.md                    What changed and why, plus open items
```

## Status

**Not compiled or backtested.** This work was done without a Windows/MT5
environment available, so nothing here has been run through MetaEditor or the
Strategy Tester. Before using any of these files:

1. Open the `.mq5` in MetaEditor and compile — resolve any errors that surface.
2. Backtest in Strategy Tester using **"Every tick based on real ticks"** mode with
   realistic spread/commission modeling — this EA's edge is thin enough that a
   lower-fidelity tick model can show a fake profit or a fake loss.
3. Forward-test on a demo account before considering real money.

See `EA/tuned-usdjpy-24-5/NOTES.md` for the specific changes made to the USD/JPY
variant, the reasoning behind them, and what's still uncalibrated (spread filter
threshold, TSL point values, weekend-gap hours) — those all need checking against
your own broker's actual behavior, not just taken as-is.

## Why USD/JPY, 24/5

Chosen over a single-session build (e.g. AUD/JPY during Asian hours only) because
it has several distinct volatility catalysts spread across the full day — Tokyo
fix, London open, US data/Fed-driven moves — rather than depending on one region's
session, which suits a breakout-straddle EA that needs recurring real moves rather
than one active window and a long quiet tail. Full reasoning in `NOTES.md`.
