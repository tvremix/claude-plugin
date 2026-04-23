# tvremix Plugin for Claude Code

TradingView-powered market analysis inside Claude Code. Adds 8 slash commands backed by our hosted MCP server at `https://tvremix.xyz/api/mcp/v1` — no Python dependencies, no API keys to copy, no browser extension required.

## What you get

| Command | What it does | Example |
|---|---|---|
| `/tvremix:setup` | Verify connection + tour the commands | `/tvremix:setup` |
| `/tvremix:smc` | Smart Money Concepts — BOS/CHoCH, order blocks, FVGs, premium/discount bias | `/tvremix:smc NASDAQ:NVDA 4h` |
| `/tvremix:swing` | Swing setup — Fibonacci, pivots, pullback zones, R:R | `/tvremix:swing NASDAQ:AAPL` |
| `/tvremix:momentum` | Multi-timeframe RSI / MACD / ADX / volume scorecard | `/tvremix:momentum BINANCE:BTCUSDT` |
| `/tvremix:options` | Options chain, Greeks, IV skew, strategy suggestions with real bid/ask | `/tvremix:options NASDAQ:TSLA` |
| `/tvremix:levels` | Clean S/R ladder — pivots, OBs, FVGs, liquidity pools, Fib | `/tvremix:levels NASDAQ:META` |
| `/tvremix:confluence` | Do 15m / 1h / 4h / 1D / 1W all agree? | `/tvremix:confluence NASDAQ:MSFT` |
| `/tvremix:chart` | Fast price-action glance + pointer to the right deep-dive skill | `/tvremix:chart NYSE:JPM` |

You can also just ask Claude anything about a symbol — the slash commands are opinionated shortcuts for common workflows.

## Prerequisites

- **Claude Code** — `npm install -g @anthropic-ai/claude-code`
- **A tvremix account** — any Google or email OTP sign-in at [tvremix.xyz](https://tvremix.xyz). Free during the public beta (15 AI requests/day, 60 MCP calls/min).

## Install

```bash
claude plugin marketplace add tvremix/claude-plugin
claude plugin install tvremix
```

Restart Claude Code. The tvremix MCP server is wired in automatically via `.mcp.json` — no config editing.

## First run

```bash
/tvremix:setup
```

On the first tool call, a browser window opens at `tvremix.xyz/oauth/authorize`. Sign in, click **Allow**, and you're done. Claude Code stores a short-lived access token plus a refresh token — nothing to copy, nothing to rotate.

## How it works

```
Claude Code (slash command)
  └─> reads plugin/skills/<name>/SKILL.md
       └─> calls mcp__tvremix__* tools
            └─> hosted MCP at tvremix.xyz/api/mcp/v1
                 └─> OAuth 2.1 (DCR) on first request, refresh tokens after
                      └─> TradingView data + SMC/swing analyzers + options + web search
```

Every skill follows the same pattern: read the shared `skills/mcp-tools.md` for tool signatures, read `skills/presentation.md` for output conventions, then run the workflow. Skills are plain Markdown with YAML frontmatter — read, fork, or copy-paste them.

## Not in Claude Code?

The same MCP server works with Claude Desktop, Claude.ai web (OAuth), Cursor, OpenClaw, Hermes, MCP Inspector, Windsurf, Cline, and anything else that speaks MCP. See [tvremix.xyz/mcp](https://tvremix.xyz/mcp) for copy-paste snippets.

## Troubleshooting

**`401 Unauthorized` on the first call.** OAuth didn't complete. Run `/tvremix:setup` again — the browser should re-open.

**`429 Too Many Requests`.** Rate limit (60/min/account). Back off for the window in the `Retry-After` header.

**Slash commands don't appear.** Restart Claude Code after `claude plugin install`. If they still don't appear, `claude plugin list` should show `tvremix` — if missing, re-run the marketplace add.

**Plugin works but skills feel stale.** Skills are shipped as Markdown inside the plugin; `claude plugin update tvremix` pulls the latest.

**No exchange on a ticker.** You can always pass either `AAPL` or `NASDAQ:AAPL`. Skills resolve bare tickers via `search_symbols` first.

## Development

This plugin lives inside the main tvremix monorepo and is mirrored to a public `tvremix/claude-plugin` repo via `git subtree`. To ship a change:

```bash
# From the monorepo root, after merging your plugin change to main:
git subtree push --prefix=plugin git@github.com:tvremix/claude-plugin.git main
```

That's it — there's no GitHub Action in v1. Cut a tag on the public repo if you want to version-lock users.

To iterate locally:

```bash
cd /path/to/monorepo
claude --plugin-dir ./plugin
# Now /tvremix:* commands are available against your local skill files.
```

## License

Apache 2.0 — see [LICENSE](LICENSE).

## Issues

File against the public mirror: <https://github.com/tvremix/claude-plugin/issues>.
