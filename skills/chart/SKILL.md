---
name: chart
description: Quick price-action read — quote, trend, recent bars, immediate context
argument-hint: EXCHANGE:TICKER [timeframe]
---

Give a concise price-action read on: `$ARGUMENTS`

Before starting, read `../mcp-tools.md` and `../presentation.md`.

This is the lightweight "glance at the chart and tell me where we are" skill. Target output: ≤ 12 lines, no tables unless the user asks.

## 1. Parse arguments

- Symbol: resolve bare tickers with `mcp__tvremix__search_symbols`.
- Timeframe: default `1D`. Accept `1h` / `4h` / `1W`.

## 2. Pull a minimal data set

```
mcp__tvremix__get_quote(symbol=<SYMBOL>)
mcp__tvremix__get_ohlcv(symbol=<SYMBOL>, interval=<TF>, count=60, summary=False)
mcp__tvremix__get_technicals(symbol=<SYMBOL>, interval=<TF>)
```

60 bars is enough to describe recent structure without drowning in data.

## 3. Describe, don't dump

Produce a paragraph (or a few short bullets) covering:

- **Spot**: current price, today's change %, volume vs average from the quote.
- **Trend**: up / down / sideways. Use the last ~20 bars. Confirm with SMA(20) vs SMA(50) positioning from technicals.
- **Recent action**: what happened in the last 5–10 bars? Breakout? Pullback? Range? Big impulsive candle?
- **Nearest structural levels**: the single most important S and R from the bars (the recent swing high + swing low usually suffices).
- **Indicator tone**: RSI, MACD state in one phrase each ("RSI 68 overbought", "MACD above signal, rising").
- **Heads-up**: anything immediate — approaching a major round level, big gap, earnings in < 7 days.

If earnings timing might matter, call `mcp__tvremix__get_earnings_calendar(symbols=[<SYMBOL>])` — but only if a trade discussion seems likely.

## 4. Upsell to deeper skills

Finish with one line pointing at the right deep-dive skill:
- Structure question → "Run `/tvremix:smc <SYMBOL> <TF>` for order blocks + FVG detail."
- Setup question → "Run `/tvremix:swing <SYMBOL>` for Fib + R:R."
- Multiple TFs → "Run `/tvremix:confluence <SYMBOL>` to see if all TFs agree."
- Levels only → "Run `/tvremix:levels <SYMBOL>`."

Don't list all four — pick the most relevant one based on what the user just asked.

## Don't

- Don't call the SMC or swing analyzers here — this is the fast lane. If the user wants depth, point them at the dedicated skill.
- Don't output a full table of bars — the user can see them on TradingView. Describe the shape, not the data.
- Don't speculate beyond what the data supports. "Could break out" needs a specific level; if you can't point to one, don't say it.
