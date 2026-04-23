---
name: levels
description: Key price levels — support/resistance, order blocks, FVGs, liquidity pools
argument-hint: EXCHANGE:TICKER [timeframe]
---

Produce a comprehensive levels map for: `$ARGUMENTS`

Before starting, read `../mcp-tools.md` and `../presentation.md`.

This skill is the "just give me the price levels" version — no full strategic read, no trade setup unless asked. The output should be a clean ladder of actionable price zones.

## 1. Parse arguments

- Symbol: resolve bare tickers with `mcp__tvremix__search_symbols`.
- Timeframe: default `1D`. Accept `4h` / `1W` / `1h` on user request.

## 2. Gather structural data

```
mcp__tvremix__analyze_swing_tool(symbol=<SYMBOL>, interval=<TF>, count=300, swing_lookback=10)
mcp__tvremix__analyze_smc_tool(symbol=<SYMBOL>, interval=<TF>, count=300, swing_lookback=5)
mcp__tvremix__get_quote(symbol=<SYMBOL>)
```

Swing gives you trendlines, channels, pivot-based S/R, Fib levels. SMC gives order blocks, FVGs, liquidity pools. Quote gives you the spot so you can sort by distance from price.

## 3. Build the level ladder

Merge every level from both tool outputs into one list. Annotate each with:
- **Price** (single number or tight range).
- **Type**: OB / FVG / EQH / EQL / swing pivot / Fib / trendline / round number.
- **Direction**: resistance (above spot) vs support (below spot).
- **Strength** (low / medium / high) — untested OBs and fresh liquidity pools rank high; touched-multiple-times pivots also rank high; once-touched Fibs rank medium; round numbers rank low.
- **Distance from spot** as a percentage.
- **State** (untested / mitigated / partial).

Sort by distance from spot, ascending. Drop mitigated levels entirely unless they're major HTF pivots.

## 4. Present

### Resistance ladder (above spot)

| Price | Type | Strength | Dist | Notes |
|---:|---|:---:|---:|---|
| $192.30 | Bearish OB | High | +3.8% | Untested, formed 2026-04-08 |
| $195.00 | Round + Fib 1.272 | Medium | +5.3% | Confluence |

### Support ladder (below spot)

| Price | Type | Strength | Dist | Notes |
|---:|---|:---:|---:|---|
| $181.40 | Bullish FVG | High | −2.1% | Unfilled, formed 2026-04-12 |
| $178.00 | Fib 0.618 + prior swing | High | −3.9% | Golden-pocket confluence |

### Nearest-to-watch
End with a one-line summary: "Watch $181.40 (bullish FVG) below and $192.30 (bearish OB) above — either break likely marks the next directional leg."

## 5. Optional extras

If the user asked for "key levels for a trade", add:
- Premium/discount context from the SMC output in one line.
- Nearest liquidity pool (EQH/EQL) that's likely to be swept first.

If they asked for "levels across timeframes", loop the tool calls over `["4h","1D","1W"]` and present one ladder per TF, compact.

## Don't

- Don't include every minor pivot — curate. A good levels map shows 4–8 levels each side, not 20.
- Don't list mitigated OBs unless the user specifically asked for historical structure.
- Don't draw — this MCP has no chart access. Point them at the Chrome extension if they want visuals.
