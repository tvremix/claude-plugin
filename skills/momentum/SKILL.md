---
name: momentum
description: Multi-timeframe momentum read — RSI, MACD, ADX, volume, divergences
argument-hint: EXCHANGE:TICKER
---

Assess momentum for: `$ARGUMENTS`

Before starting, read `../mcp-tools.md` and `../presentation.md`.

## 1. Resolve the symbol

If the user gave a bare ticker, use `mcp__tvremix__search_symbols` to get `EXCHANGE:SYMBOL`.

## 2. Gather data (batch these tool calls)

```
mcp__tvremix__analyze_multi_timeframe(symbol=<SYMBOL>, timeframes=["15m","1h","4h","1D","1W"])
mcp__tvremix__get_ohlcv(symbol=<SYMBOL>, interval="1D", count=200, summary=False)
mcp__tvremix__get_quote(symbol=<SYMBOL>)
```

`analyze_multi_timeframe` returns RSI / MACD / Stochastic / ADX / aggregated rating per TF. `get_ohlcv` gives you the bars you need for volume analysis. `get_quote` gives current price, change%, and volume-vs-average.

## 3. Build a momentum scorecard

For each of the five timeframes, score momentum as one of: `strong bullish` / `bullish` / `neutral` / `bearish` / `strong bearish`. The cues:
- RSI > 60 and rising → bullish; > 70 = overbought warning.
- RSI < 40 and falling → bearish; < 30 = oversold.
- MACD line above signal and rising histogram → bullish momentum.
- MACD cross-down or falling histogram → fading.
- ADX > 25 = trending (momentum has teeth); < 20 = chop.

Display as a compact table:

| TF | RSI | MACD | ADX | Rating | Read |
|---|---|---|---|---|---|
| 15m | … | … | … | … | … |
| …  | … | … | … | … | … |

## 4. Volume confirmation

From the daily `get_ohlcv` output, check the last 5–10 bars' volume vs the 20-bar average (`summary.avg_volume` from the tool output). Is recent volume supporting the price move, or fading?

## 5. Divergence check

Look at the last ~30 daily bars. Are new price highs being made without matching new RSI highs (bearish divergence) or new price lows without new RSI lows (bullish divergence)? Call it out if present. If no clear divergence, say "no notable divergence".

## 6. MA positioning

From the 1D technicals, note where current price sits relative to the 20 / 50 / 200 SMA. "Above all three, 50>200" → golden-cross territory. "Below 50, above 200" → pullback in uptrend. Etc.

## 7. Synthesise

End with a **single paragraph** that delivers:
- Overall momentum bias (bullish / bearish / mixed) with a 0–10 strength score.
- Timeframe alignment (all TFs aligned vs split vs reversing).
- The one or two pieces of evidence that carry the most weight.
- One actionable follow-up — a specific level to watch, a specific catalyst, or a clean delegation to `/tvremix:smc` or `/tvremix:swing` if structure work is the next step.

## Don't

- Don't dump the raw technicals object — extract what matters.
- Don't call every tool twice. If `analyze_multi_timeframe` already has RSI for 4h, don't re-fetch it via `get_technicals`.
- Don't invent momentum numbers the tools didn't return.
