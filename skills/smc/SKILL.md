---
name: smc
description: Smart Money Concepts analysis — BOS/CHoCH, order blocks, FVGs, liquidity, premium/discount bias
argument-hint: EXCHANGE:TICKER [timeframe]
---

Run a full Smart Money Concepts (ICT-style) read on: `$ARGUMENTS`

Before starting, read `../mcp-tools.md` and `../presentation.md`.

## 1. Parse the arguments

- First token is the symbol. If the user gave a bare ticker (no colon), resolve it with `mcp__tvremix__search_symbols(query=<ticker>, limit=5)` and use the top exchange-qualified match (default exchanges: `NASDAQ` for tech, `NYSE` for blue chips, `BINANCE` for crypto).
- Second token is the timeframe. Default to `4h` if omitted. Accept any of `5m`, `15m`, `30m`, `1h`, `4h`, `1D`, `1W`.

## 2. Run the SMC engine

```
mcp__tvremix__analyze_smc_tool(symbol=<SYMBOL>, interval=<TF>, count=300, swing_lookback=5)
```

300 bars gives enough history for meaningful structure detection. `swing_lookback=5` captures internal (short-term) structure — the actionable stuff.

Optionally, for a bigger-picture read, also call it once with `swing_lookback=25` to get swing-level structure. Compare the two to see whether the internal move is with or against the higher-timeframe trend.

## 3. Layer in multi-timeframe context (optional but recommended)

```
mcp__tvremix__analyze_multi_timeframe(symbol=<SYMBOL>, timeframes=["1h","4h","1D","1W"])
```

This gives HTF bias you can cross-reference with the SMC read.

## 4. Present the analysis

Order matters. Use this exact section sequence:

### a. Market Structure
Current trend direction. Most recent BOS (continuation) and CHoCH (reversal) events with their prices and dates. Explain in one line what the structure tells us.

### b. Key Levels
- **Untested Order Blocks** — list each bullish / bearish OB with its price range and formation date. Flag which are mitigated vs untested.
- **Open Fair Value Gaps** — unfilled FVGs with their price ranges. Note direction (bullish/bearish gap).
- **Liquidity Pools** — equal highs (EQH = buy-side liquidity, short stops stacked) and equal lows (EQL = sell-side liquidity, long stops stacked).

Only list **untested** / **unfilled** levels — mitigated/filled levels are spent and waste the reader's attention.

### c. Premium / Discount
Where does current price sit relative to the latest swing range? Discount zone (below equilibrium) favours longs, premium zone favours shorts. State the exact equilibrium midpoint.

### d. Bias
One-sentence overall bias with confidence (low / medium / high) synthesising structure + premium/discount + HTF context.

### e. Trade Setup (only if conditions align)
- **Entry**: at an untested OB or unfilled FVG on the bias-aligned side.
- **Stop**: beyond the relevant structural level (below the OB for longs, above it for shorts).
- **Targets**: T1 = next opposing liquidity pool; T2 = next Fib extension / major swing level.
- **R:R**: compute (T1 − entry) / (entry − stop). Anything under 1.5 isn't worth taking.
- **Invalidation**: what specific price or close would void the read.

## 5. Reference cheatsheet (for the user)

At the very end, include a collapsible note for unfamiliar readers — a 5-line glossary:

> BOS = break of structure (continuation). CHoCH = change of character (first reversal signal). OB = last opposing candle before an impulsive break. FVG = three-candle price imbalance; ~70% fill on 1h+. Premium/Discount = upper/lower half of the swing range.

Skip the glossary if the user's message makes clear they already know the terms.

## Notes

- This MCP can **not** draw on the chart. If the user wants visuals, tell them they can run the same analysis inside the tvremix Chrome extension (`https://tvremix.xyz`) which drives TradingView directly.
- If `analyze_smc_tool` returns `{"success": false}`, surface the error and stop — don't pad with generic technicals.
- Never quote a price or level that isn't in the tool output.
