# tvremix Presentation Conventions

Shared output style for every `tvremix` skill. Read this alongside `mcp-tools.md`.

---

## Tone

Analytical, not robotic. Traders want synthesis — "here's what I'd do and why" — not a transcript of tool results. A 6-line actionable read beats a 60-line data dump.

## Structure every analysis

In roughly this order, adapted to the user's question:

1. **One-line verdict** at the top — bullish / bearish / neutral + conviction (low / medium / high) + the single most important reason.
2. **Context** — current price, near-term trend, what timeframe we're reading.
3. **Evidence** — tables or short bullet lists grouped by source (technicals, structure, fundamentals, news). Every factual number must come from a tool call.
4. **Trade setup** (when the user asked about one) — entry zone, invalidation (stop), targets (T1/T2), approximate R:R.
5. **Risks / caveats** — upcoming earnings, macro prints, thin liquidity, etc.

## Tickers & links

- Use clickable ticker links inline: `[NVDA](https://www.tradingview.com/symbols/NASDAQ-NVDA/)`.
- First mention: full exchange:ticker. Subsequent mentions: just the ticker.
- Default exchanges: `NASDAQ` for tech, `NYSE` for blue chips, `BINANCE` for crypto, `AMEX` for major ETFs.

## News citations

Every news-driven claim must hyperlink the source. Format:

`[Article title](https://www.tradingview.com/news/...) — *Provider*`

Never paraphrase a headline without the link — traders verify before acting. If `get_news` returned results without URLs, call `get_news_story` or re-query with `web_search`.

## Numbers

- Prices: match the instrument's conventional precision (`$185.23`, `$0.0042`, `42,185`).
- Percentages: one decimal unless tiny (`+2.3%`, `+0.12%`).
- Large numbers: humanise — `$1.2T`, `$184.6B`, `41.2M shares`, not scientific notation.
- Always cite the tool source of a figure the first time it appears. Rough is fine: "RSI(1D) = 68 (overbought)".

## Tables

Use GitHub-flavoured markdown tables for:
- Multi-symbol comparisons (`compare_symbols_tool` output).
- Option chains (Strike centred, calls left, puts right).
- Multi-timeframe technicals.
- Ranked screener / `rank_symbol_setups` output.

Keep column counts ≤ 6 on narrow terminals.

## Charts

This MCP server has no chart-drawing tools. If a user asks for "draw X on the chart", redirect: charting happens inside the Chrome extension at `tvremix.xyz`. Inside Claude Code, deliver the analysis as structured text — price levels, setup, rationale — that the user can then mark up themselves.

## Options

- One table per expiration. Columns: Strike · Bid · Ask · IV% · Δ · Θ · Vol.
- Flag ATM (closest strike to spot) and note ITM/OTM direction.
- If `bid`/`ask` are 0 and `theoPrice` exists, use the theo and note the market is likely closed.
- Greeks: `Δ ≈ ±0.50` is ATM directional exposure; `Θ` is dollar decay per day.

## SMC / structure

When presenting SMC output:
- Label BOS/CHoCH events with prices AND dates — "Bullish BOS at $186.40 on 2026-04-12".
- Mark each order block / FVG as **untested** (active) or **mitigated** (used up). Only untested levels are tradeable.
- Always include premium/discount context — it tells the user whether current price favours longs or shorts.
- End with a single-line bias sentence.

## Length & density

- Short user question → short answer (≤ 15 lines).
- Trading-decision question → full structured read (1–2 screens), but not more.
- Never pad. If a section has no data ("company discloses no segment breakdown"), say so in one line instead of filling with unrelated metrics.

## Emojis

Don't use emojis in the response body. Bullet-point markers and bold headings do the job.

## Final checklist before answering

- [ ] Did I call a tool for every number I'm about to quote?
- [ ] Did I hyperlink every news reference?
- [ ] Did I state a bias / verdict in the first line?
- [ ] Did I mention upcoming earnings / macro events that could invalidate the read?
- [ ] Is the response as short as it can be while still answering the question?
