---
name: setup
description: Verify the tvremix MCP connection and tour the available commands
---

Walk the user through confirming their `tvremix` plugin is working and introduce them to the available commands. Keep the tone conversational.

## Step 1 — Confirm the environment

Tell the user you'll run a harmless smoke test. If they're seeing this skill, Claude Code is clearly running, so just confirm briefly.

## Step 2 — Authenticate (first run only)

This plugin connects to one MCP server:

- **tvremix** — `https://tvremix.xyz/api/mcp/v1` (TradingView data + SMC/swing analyzers + options + web search)

On the very first tool call, a browser window will open at `tvremix.xyz/oauth/authorize`. Tell the user:

1. Sign in with Google (or email OTP).
2. Click **Allow**.
3. You'll be redirected back and the tool will complete automatically.

If they've done this before, nothing opens — the stored refresh token is used silently.

## Step 3 — Smoke test

Call `mcp__tvremix__get_quote` with `symbol="NASDAQ:AAPL"`. This is the cheapest, safest call — public data, no side effects. Show the user the resulting price + change% + volume as a single line, e.g.:

> **AAPL** — $185.23 (+1.2%) on 41.8M shares — tvremix MCP is live.

If the call fails:
- `401 Unauthorized` → OAuth didn't complete. Ask the user to re-run the command to re-trigger the browser flow.
- `429 Too Many Requests` → rate limit (60/min/account). Wait a minute and retry.
- Any other error → report the full message and suggest they file an issue at `https://github.com/tvremix/claude-plugin/issues`.

## Step 4 — Quick tour

Tell the user about the available slash commands. All are prefixed with `/tvremix:`:

| Command | What it does | Example |
|---|---|---|
| `/tvremix:setup` | This verification skill | `/tvremix:setup` |
| `/tvremix:smc` | Smart Money Concepts analysis (BOS, CHoCH, OBs, FVGs, bias) | `/tvremix:smc NASDAQ:NVDA 4h` |
| `/tvremix:swing` | Swing trading setup with Fibonacci, pivots, R:R | `/tvremix:swing NASDAQ:AAPL` |
| `/tvremix:momentum` | Multi-timeframe RSI/MACD/volume momentum read | `/tvremix:momentum BINANCE:BTCUSDT` |
| `/tvremix:options` | Options chain, Greeks, strategy suggestions | `/tvremix:options NASDAQ:TSLA` |
| `/tvremix:levels` | Key S/R, order blocks, FVGs, liquidity levels | `/tvremix:levels NASDAQ:META` |
| `/tvremix:confluence` | Multi-timeframe trend alignment check | `/tvremix:confluence NASDAQ:MSFT` |
| `/tvremix:chart` | Quick price-action read (quote + bars + trend) | `/tvremix:chart NYSE:JPM` |

They can also just ask Claude anything about a symbol — the commands are shortcuts for common workflows.

## Step 5 — What's next

Suggest they try `/tvremix:smc NASDAQ:NVDA 4h` as a first real test — it exercises the most sophisticated tool (`analyze_smc_tool`) and produces a visually interesting structured read.

Remind them:
- Not a Claude Code user? The same tools are available to Claude Desktop, Claude.ai web, Cursor, OpenClaw, Hermes, and any other MCP client — see `https://tvremix.xyz/mcp`.
- Need API keys for CLI tools? `https://tvremix.xyz/account#api-keys`.
