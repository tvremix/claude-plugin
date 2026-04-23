# tvremix MCP Tool Reference

Shared cheat sheet for every `tvremix` skill. Read this first, then return to the calling skill.

---

## Connection & Auth

- **Server**: `https://tvremix.xyz/api/mcp/v1` (Streamable HTTP).
- **Transport**: Claude Code reads the server from `.mcp.json` in this plugin — nothing to configure.
- **Auth**: OAuth 2.1 with Dynamic Client Registration. On the first tool call a browser window opens at `tvremix.xyz/oauth/authorize`; sign in with Google (or email OTP) and click **Allow**. Tokens refresh automatically.
- **If OAuth is blocked** (CI, headless env): the user can generate a `tvr_…` API key at `https://tvremix.xyz/account#api-keys` and set it via their MCP client config. OAuth is strongly preferred.

## Rate limits

60 requests / minute / account. Hitting the cap returns HTTP 429 with a `Retry-After` header. Back off for that window and retry.

## Tool catalog

All tools are exposed as `mcp__tvremix__<name>`. Categories:

### Symbol identity & quotes
| Tool | Args | Returns |
|---|---|---|
| `search_symbols` | `query` (str), `limit` (int=10) | Fuzzy-matched symbols with `EXCHANGE:SYMBOL`. Use this first when the user gives a ticker without an exchange. |
| `get_quote` | `symbol` (str) | Snapshot: price, change, volume, O/H/L/C, market cap. |
| `get_symbol_data` | `symbol`, `columns` (list[str]) | Escape hatch for raw scanner columns. |

### Technicals & ratings
| Tool | Args | Returns |
|---|---|---|
| `get_technicals` | `symbol`, `interval` (default `1D`) | RSI, MACD, SMA/EMA, Bollinger, etc. at one interval. |
| `get_technicals_rating` | `symbol`, `interval` | Aggregated buy/sell/neutral score. |
| `get_full_technicals` | `symbol`, `timeframes` (list, default `["15","60","240","1D","1W"]`) | Same indicators across many intervals in one call. |
| `analyze_multi_timeframe` | `symbol`, `timeframes` | Trend bias per TF + cross-TF alignment score. |

### Fundamentals & forecasts
| Tool | Args | Returns |
|---|---|---|
| `get_financials` | `symbol` | P/E, EPS, market cap, margins, sector, industry. |
| `get_forecasts` | `symbol` | Analyst price targets, ratings distribution, EPS estimates. |
| `get_documents` | `symbol`, `doc_type?`, `limit` | SEC filings (10-K/10-Q/8-K) listings. |
| `get_document_view` | `view_id` | Full document text. |

### News & calendars
| Tool | Args | Returns |
|---|---|---|
| `get_news` | `symbol`, `limit=10` | Headlines with `link`, `provider`, `published`. |
| `get_news_story` | `story_id` | Full article body. |
| `get_economic_calendar` | `countries?`, `importance?`, `date_from?`, `date_to?` | Macro events. |
| `get_earnings_calendar` | `symbols?`, `market`, `date_from?`, `date_to?`, `limit` | Upcoming/past earnings. |
| `get_dividends_calendar` | `market`, `date_from?`, `date_to?`, `limit` | Upcoming dividend payments. |

### Bars & market structure
| Tool | Args | Returns |
|---|---|---|
| `get_ohlcv` | `symbol`, `interval`, `count` (≤5000), `summary` (bool) | Historical bars straight from TV's WebSocket feed. |
| `analyze_smc_tool` | `symbol`, `interval`, `count`, `swing_lookback` | Smart Money Concepts: BOS/CHoCH, order blocks, FVGs, liquidity, premium/discount, bias. Internally fetches bars. |
| `analyze_swing_tool` | `symbol`, `interval`, `count`, `swing_lookback` | Swing structure: pivots, trendlines, channels, range zones, bias. Internally fetches bars. |

### Options
| Tool | Args | Returns |
|---|---|---|
| `get_option_expirations` | `symbol` | Available expiries. |
| `get_option_chain` | `symbol`, `expiration?`, `option_type?` | Calls/puts with IV, Greeks, OI, volume. |

### Screeners & cross-symbol
| Tool | Args | Returns |
|---|---|---|
| `run_screener` | `market`, `filters`, `sort_by`, `sort_order`, `limit`, `columns?`, `symbol_types?`, `filter_preset?` | Full TV screener access. |
| `rank_symbol_setups` | `symbols` (list), `focus` (`balanced`/`momentum`/`mean_reversion`/`breakout`), `top_n`, `side` | Score a watchlist for tradable setups. |
| `compare_symbols_tool` | `symbols`, `metrics?` | Side-by-side price/fundamentals/technicals with rankings. |
| `analyze_sector_tool` | `sector_name`, `metric`, `limit`, `market` | Scan + rank a sector/industry. |
| `calculate_correlation_tool` | `symbols`, `period=30`, `interval=1D` | Pearson correlation matrix from log returns over real bars. |

### Web research
| Tool | Args | Returns |
|---|---|---|
| `web_search` | `query`, `num_results=3` | General-purpose web search (Exa-backed). Use for news context, earnings commentary, macro. |

## Symbol format

Always `EXCHANGE:SYMBOL`:
- `NASDAQ:AAPL`, `NASDAQ:MSFT`, `NASDAQ:NVDA`
- `NYSE:JPM`, `NYSE:BRK.B`
- `BINANCE:BTCUSDT`, `BINANCE:ETHUSDT`
- `AMEX:SPY`, `AMEX:QQQ`
- `FX:EURUSD`, `OANDA:XAUUSD`

If the user gives a bare ticker, call `search_symbols` first to resolve the exchange.

## Interval format

Plain-text codes: `1m`, `5m`, `15m`, `30m`, `1h`, `4h`, `1D`, `1W`, `1M`. A few tools accept minute counts (`240` = 4h); prefer the plain-text codes when available.

## Defaults that matter

- `analyze_smc_tool` / `analyze_swing_tool`: **`count=300`** gives enough history for meaningful structure. Don't drop below ~100 or pivots collapse.
- `swing_lookback=5` → internal structure (short-term). `swing_lookback=20-50` → swing-level structure. Call twice with different values if you want both.
- `get_full_technicals` default TFs (15m / 1h / 4h / 1D / 1W) cover most multi-TF needs in one call — prefer it over five separate `get_technicals` calls.

## Timestamps

Most tool responses return ISO 8601 strings already (server-side safety net). `get_ohlcv` bars use Unix seconds in `t` for compactness — convert only if you need to quote dates to the user.

## Failure modes

- `{"success": false, "error": "..."}` on tool failures — surface the error to the user, don't retry blindly.
- Unknown symbol → empty/null fields. Verify with `search_symbols` first.
- Market closed → options `bid`/`ask` may be 0; use `theoPrice` + IV.

## What this MCP does NOT do

No authenticated TradingView surface (alerts, watchlists, drawings, paper trading). That lives only in the Chrome extension (`tvremix.xyz`). Don't promise features outside this catalog.
