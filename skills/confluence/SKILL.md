---
name: confluence
description: Multi-timeframe trend + structure confluence check
argument-hint: EXCHANGE:TICKER
---

Assess multi-timeframe confluence for: `$ARGUMENTS`

Before starting, read `../mcp-tools.md` and `../presentation.md`.

Purpose: tell the user, in one read, whether all timeframes agree or disagree — so they can decide whether this is a high-probability directional trade or a chop zone to avoid.

## 1. Resolve symbol

Use `mcp__tvremix__search_symbols` for bare tickers.

## 2. Gather cross-TF data

```
mcp__tvremix__analyze_multi_timeframe(symbol=<SYMBOL>, timeframes=["15m","1h","4h","1D","1W"])
```

Single call returns indicators + aggregated bias per TF.

Then, for the two most decision-relevant TFs (default: `4h` and `1D`), also run SMC:

```
mcp__tvremix__analyze_smc_tool(symbol=<SYMBOL>, interval="4h", count=300, swing_lookback=5)
mcp__tvremix__analyze_smc_tool(symbol=<SYMBOL>, interval="1D", count=300, swing_lookback=10)
```

This gives you structural bias (not just indicator bias).

## 3. Build the confluence scorecard

Present a compact table:

| TF | Trend | RSI | MACD | Structure (SMC) | Verdict |
|---|:---:|:---:|:---:|:---:|:---:|
| 15m | up | 62 | bull | — | bullish |
| 1h | up | 65 | bull | — | bullish |
| 4h | up | 58 | bull | BOS up, in discount | **strong bullish** |
| 1D | up | 55 | neutral | BOS up, mid-range | bullish |
| 1W | sideways | 50 | flat | ranging | neutral |

Verdict per row combines indicator direction + structure (where available).

## 4. Compute the alignment score

- **5/5 aligned** → "strong confluence, trade with trend".
- **4/5 aligned (including 1D and 4h)** → "good confluence".
- **3/5 with 1D/1W disagreeing** → "intraday signal, fading HTF — low-conviction".
- **Mixed (2–3 each way)** → "chop / rotation, no directional edge".
- **HTF reversing (1W CHoCH, lower TFs still trending old direction)** → "warning: reversal in progress, avoid with-trend entries".

State the score explicitly: "4/5 aligned bullish" or similar.

## 5. Actionable follow-up

One paragraph ending with either:
- **If aligned**: "Best entry TF is [4h / 1D depending on user's holding horizon]. Run `/tvremix:smc <SYMBOL> <TF>` for entry zones."
- **If mixed**: "No clean trade here — wait for [specific TF] to align, then re-check."
- **If HTF reversing**: "Reduce risk / flip bias. Watch [specific level] for confirmation."

## Don't

- Don't overload the user with every indicator reading — the table is the summary.
- Don't call `get_technicals` or `get_ohlcv` separately per TF. `analyze_multi_timeframe` is a single call that covers all of them.
- Don't recommend a trade — this skill's job is only to answer "do the timeframes agree?". Direct them to `/tvremix:smc` or `/tvremix:swing` for the setup work.
