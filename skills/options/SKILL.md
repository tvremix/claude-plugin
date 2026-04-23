---
name: options
description: Options chain analysis with Greeks, IV skew, and strategy suggestions
argument-hint: EXCHANGE:TICKER [expiration]
---

Analyse the options chain for: `$ARGUMENTS`

Before starting, read `../mcp-tools.md` and `../presentation.md`.

## 1. Parse arguments

- Symbol: resolve bare tickers with `mcp__tvremix__search_symbols`.
- Optional expiration: if the user gave a date (e.g. `2026-05-16`), use it. Otherwise, pick the nearest weekly/monthly expiration that's at least 7 days out.

## 2. Get expirations and spot

```
mcp__tvremix__get_option_expirations(symbol=<SYMBOL>)
mcp__tvremix__get_quote(symbol=<SYMBOL>)
```

`get_quote` gives you the underlying price you need for ATM / ITM / OTM classification and for `min_strike`/`max_strike` framing.

## 3. Fetch the chain

```
mcp__tvremix__get_option_chain(symbol=<SYMBOL>, expiration=<PICKED_EXPIRATION>)
```

Don't pass `option_type` unless the user explicitly asked for just calls or just puts — the full chain lets you show the calls/puts table side-by-side.

If the returned chain is huge (hundreds of strikes), filter in-code to strikes within ±15% of spot. Keep ≈ 10 calls + 10 puts bracketing ATM.

## 4. Present the chain

### Table format
One table per expiration, strikes in the centre, calls on the left, puts on the right:

| Call Vol | Call Δ | Call IV% | Call Ask | Strike | Put Ask | Put IV% | Put Δ | Put Vol |
|---:|---:|---:|---:|:---:|---:|---:|---:|---:|
| … | … | … | … | **(ATM)** $185 | … | … | … | … |

- Bold the ATM row (closest strike to spot).
- Note ITM / OTM above/below ATM.
- If `bid`/`ask` are 0 and `theoPrice` is populated, use `theoPrice` and add a footnote: "market likely closed — theo prices shown".

### IV context
After the table, in 2–3 lines:
- ATM IV and whether it's elevated vs historical norms for the name.
- Skew: are OTM puts materially higher IV than OTM calls (fear premium)? Vice versa?
- Call ATM Δ and Put ATM Δ — should be close to ±0.50; anything skewed tells you where the chain thinks the underlying is heading.

## 5. Strategy suggestion (when the user asked for one)

Pick one or two strategies from this list that match the stated view. Always quote the real bid/ask from the chain — never estimate:

- **Bullish & want leverage** → long ATM call or slight OTM call-debit spread.
- **Bullish & want income** → cash-secured put at a strike you'd happily own.
- **Bearish & want leverage** → long ATM put or put-debit spread.
- **Neutral / range-bound** → iron condor or short strangle (warn about unlimited-risk legs).
- **Holding shares, want income** → covered call just OTM.
- **High IV, expecting a big move** → long straddle / strangle.
- **Low IV, expecting a move** → same, but note cost of theta decay.

For each suggested strategy, give:
- Exact contract(s), strike(s), expiration.
- Net debit/credit using actual bid/ask.
- Max profit, max loss, breakeven(s) — compute from contract prices.
- Theta exposure (dollars-per-day) if meaningful.
- One-line risk call-out (earnings? binary events? liquidity?).

## 6. Catalysts

Run `mcp__tvremix__get_earnings_calendar(symbols=[<SYMBOL>])` to check whether earnings fall inside the option window. If yes, flag it — the option premium probably already prices in the move.

## Don't

- Don't estimate option prices — always use the chain output.
- Don't recommend complex multi-leg strategies without quoting each leg's real price.
- Don't touch 0-DTE unless the user explicitly asked for it (mention it's high-risk).
- Don't forget to include IV% — it's the single most informative column.
