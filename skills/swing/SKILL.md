---
name: swing
description: Swing trading setup — Fibonacci levels, pivots, trend, pullback zones with R:R
argument-hint: EXCHANGE:TICKER [timeframe]
---

Produce a swing trading read on: `$ARGUMENTS`

Before starting, read `../mcp-tools.md` and `../presentation.md`.

## 1. Parse arguments

- Symbol: resolve bare tickers with `mcp__tvremix__search_symbols` if needed.
- Timeframe: default `1D` (swing trading's natural TF). Accept `4h` / `1W` on explicit user request.

## 2. Run the swing engine

```
mcp__tvremix__analyze_swing_tool(symbol=<SYMBOL>, interval=<TF>, count=300, swing_lookback=10)
```

Output includes: pivot sequence, trend (bullish/bearish/ranging), Fibonacci levels (retracement + extension), pullback depth + zone, trade setups, S/R ladder.

## 3. Multi-timeframe sanity check (recommended)

```
mcp__tvremix__analyze_multi_timeframe(symbol=<SYMBOL>, timeframes=["4h","1D","1W"])
```

Use this to confirm you're swinging WITH the higher-timeframe trend, not against it.

## 4. Present in this order

### a. Trend
Direction (bullish / bearish / ranging), strength (weak / moderate / strong), count of consecutive aligned swings, and whether the HTF trend (1W) agrees.

### b. Fibonacci Levels
Retracement ladder from the most recent major swing. Call out:
- **Golden pocket** (0.618 – 0.65) — highest-probability pullback entry zone.
- **0.382 / 0.5 / 0.786** — other levels of interest.
- **Extensions** — 1.272, 1.618, 2.0 as take-profit targets.

### c. Pullback Status
Is price currently pulling back? If yes: how deep, which Fib zone, healthy or overextended? If ranging/trending through, say so.

### d. Trade Setup (when conditions align)
Only present if there's a genuine asymmetric setup — don't force one.
- **Entry zone**: price range (usually the golden pocket or a pivot cluster).
- **Stop**: beyond the swing invalidation point, not a flat % stop.
- **Target 1**: previous swing extreme or 1.272 extension.
- **Target 2**: 1.618 extension or next major resistance.
- **R:R**: (T1 − entry) / (entry − stop). Also compute for T2.
- **Suggested size**: if user risk tolerance is unknown, just express as % of account per trade (e.g. "risk 1–2% per position").

### e. Key S/R Levels
Top 4–6 levels sorted by proximity, annotated as support or resistance with the source (Fib, swing pivot, trendline, round number). Include the trend/range context inline.

## 5. Risk caveats

Check and mention:
- Upcoming earnings within 14 days — derail any swing thesis. Call `mcp__tvremix__get_earnings_calendar(symbols=[<SYMBOL>])` if not already covered.
- Macro events within the hold window (`mcp__tvremix__get_economic_calendar`) if this is a macro-sensitive instrument.
- ATR context — if daily ATR is small relative to the proposed stop distance, flag it (your stop is too wide for the instrument).

## Notes

- Swing trading here = days-to-weeks horizon. If the user explicitly wants intraday, suggest `/tvremix:momentum` or `/tvremix:smc` with `4h`/`1h` instead.
- Do not draw on the chart — analysis only. Point them at the Chrome extension for chart markup.
- Never invent Fib levels or pivots the tool didn't output.
