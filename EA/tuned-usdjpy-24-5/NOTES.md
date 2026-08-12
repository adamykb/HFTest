# USD/JPY 24/5 tuning — notes

Base: `EA/original/High Frequency EA v2 - Including new Trailing Stops.mq5`
(straddle scalper — pending BUY_STOP/SELL_STOP kept close to price, spread-relative
distances, selectable trailing-stop methods).

## What changed and why

- **`MaxSpread` 5555 → 18 points.** The original default effectively disabled the
  spread filter (5555 points is enormous for any FX pair). 18 points (~1.8 pips) is
  a starting ceiling for USD/JPY on a typical ECN account — **watch your broker's
  live Market Watch spread on USDJPY for a session or two and tighten this to just
  above what you actually observe.**
- **`Slippage` 1 → 2.** Slightly more tolerance for fast-moving conditions across a
  24/5 schedule than a single narrow session.
- **`StartHour`/`EndHour` 16/22 → 1/23.** Runs nearly the full day; only server hour
  `0` is blocked, which is the daily rollover / spread-spike window. This is
  anchored to the broker's own midnight, so it works correctly **without needing to
  know your broker's GMT offset** — unlike a session-specific window would.
- **New: weekend gap guard** (`AvoidWeekendGap`, `FridayCutoffHour`, `SundayResumeHour`).
  Blocks *new* pending-order placement in the window before Friday close and after
  Sunday open, where gap risk is highest for a straddle strategy. Existing open
  positions are still managed as normal by `TrailStop()` — this only stops fresh
  entries into the gap window.

## Why USD/JPY for 24/5

Picked over EUR/USD because it has multiple separate volatility catalysts spread
across the day (Tokyo fix, London open, US data/Fed-driven moves) rather than being
dependent on one region's session — better fit for a breakout-straddle EA that needs
recurring, real directional moves rather than one big window and a long quiet tail.
EUR/USD has the tightest spread of any pair at any hour, but noticeably less to
trade on during quiet Asian-session stretches under one static parameter set.

## Known limitation, not fixed here

`TrailStop()` in the original v2 code has a latent double-modification issue: the
v2 "Scalp Trail" block runs **unconditionally for every position**, regardless of
`TrailType`. If `TrailType` is ever set to `0` (`Default_Trail`), that position gets
both the old formula-based trail *and* the v2 breakeven-nudge logic calling
`PositionModify` independently on the same tick. Not a problem with the default
`TrailType=1` used here — flagging it so it isn't a surprise if you switch trail
types later.

## Still open / needs your input before this is trustworthy

1. **This has not been compiled or backtested.** No MT5/MetaEditor available in this
   environment — compile in MetaEditor and run Strategy Tester ("Every tick based on
   real ticks" mode) before trusting any of this.
2. **`TslTriggerPoints`/`TslPoints` (15/10) are untouched** — these are raw points,
   not spread-relative, and were likely tuned for a different pair originally.
   Worth checking against USD/JPY's actual recent intraday range and adjusting.
3. **`FridayCutoffHour`/`SundayResumeHour` (21/1) are reasonable starting guesses**,
   not calibrated to your specific broker's actual weekly close/open behavior —
   confirm against your broker's quote history around the weekend.
4. Optimize only within `Delta`/`MaxDistance`/`Stop`/`MaxTrailing`/`TslTriggerPoints`/
   `TslPoints` in the Strategy Tester, then validate out-of-sample (a different date
   range than you optimized on) — this style of EA curve-fits easily to one window.

## Recommended Strategy Tester settings

- **Symbol Period: M1.** The EA is tick-driven (`OnTick`), so this doesn't change
  how often it trades — but it does control what `PERIOD_CURRENT` resolves to for
  `TrailType=2/3/4` (previous-candle / fast-MA / Ichimoku trailing), and matches the
  `Secs=60` input, whose own comment ("Should be same as TF") implies M1 was the
  timeframe the EA was designed around. `TrailType=1` (Scalp_Trail, our default) is
  purely point-based and unaffected by this setting either way. Movement character
  genuinely differs by timeframe — an M1 candle reflects only the last minute,
  while M5/M15 smooths and lags behind it — so M1 keeps trailing reference values
  close to current price for a strategy meant to react fast.
- **Tick model: "Every tick based on real ticks."** This is a separate setting from
  Symbol Period — selecting M1 alone does not give tick-level fill fidelity, you
  need both set correctly.
