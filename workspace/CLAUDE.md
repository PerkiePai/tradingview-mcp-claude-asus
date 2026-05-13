# Claude + TradingView MCP — Project Overview

This project is a YouTube video production about using the TradingView MCP server with Claude Code for AI-assisted crypto trading analysis. The video demos chart reading, Pine Script generation, alert setting, and multi-symbol scanning — all via natural language prompts.

---

## ⚡ Use the TradingView MCP By Default

**This project is built around the `mcp__tradingview__*` toolset. For ANY trading, chart, market data, Pine Script, or indicator question in this directory, use the MCP tools — not web search, not guessing, not Bash workarounds.**

**On session start, do this automatically (no need to ask the user):**
1. Call `mcp__tradingview__tv_health_check` immediately.
2. If `cdp_connected: false` (or the tool isn't available because TradingView wasn't running), launch TradingView yourself via Bash/PowerShell:
   ```powershell
   Start-Process "C:\Program Files\WindowsApps\TradingView.Desktop_3.1.0.7818_x64__n534cwy3pjxzj\TradingView.exe" -ArgumentList "--remote-debugging-port=9222"
   ```
   Then wait ~5 seconds (`Start-Sleep -Seconds 5`) and retry `tv_health_check`. If the MCP tool itself is missing from the session (TV wasn't running when Claude Code started), launch TV with the command above and tell the user to restart Claude Code so the MCP server attaches.
3. Once connected, call `mcp__tradingview__chart_get_state` to see the current symbol, timeframe, and loaded indicators before answering anything chart-related.

**Tool-picking shortcuts** (full guide in the MCP server's instructions):
- Price snapshot → `quote_get`
- Bars / OHLCV → `data_get_ohlcv` (always pass `summary=true` unless you need raw bars)
- Indicator values (RSI/MACD/EMA/etc.) → `data_get_study_values`
- Custom Pine output (lines/labels/tables/boxes) → `data_get_pine_*` with `study_filter="<indicator name>"`
- Add/remove studies → `chart_manage_indicator` with the **full** indicator name (e.g. "Relative Strength Index", not "RSI")
- Change ticker / timeframe → `chart_set_symbol`, `chart_set_timeframe`
- Pine dev → `pine_set_source` → `pine_smart_compile` → `pine_get_errors`
- Screenshot → `capture_screenshot` (regions: `full`, `chart`, `strategy_tester`)

**Do NOT** call `pine_get_source` unless actively editing — it can return 200KB+.

---

## Setup State (as of 2026-05-12) — Complete

Everything below is already done. Do not redo any of it.

| Component | Status | Location |
|---|---|---|
| MCP server | Installed | `C:\Users\bower\tradingview-mcp\` |
| MCP config | Configured | `~/.claude/mcp.json` |
| Tool permissions | Pre-approved | `~/.claude/settings.json` |
| Trading rules | Created | `C:\Users\bower\tradingview-mcp\rules.json` |
| TradingView Desktop | Installed (v3.1.0) | `C:\Program Files\WindowsApps\TradingView.Desktop_3.1.0.7818_x64__n534cwy3pjxzj\` |

**MCP tools load on session start** — TradingView must be running with `--remote-debugging-port=9222` before Claude Code is opened, otherwise the tools won't appear.

---

## Launching TradingView

TradingView was installed from a `.msix` package (not a standard `.exe`). Launch command:

```powershell
Start-Process "C:\Program Files\WindowsApps\TradingView.Desktop_3.1.0.7818_x64__n534cwy3pjxzj\TradingView.exe" -ArgumentList "--remote-debugging-port=9222"
```

Verify CDP is up: `http://localhost:9222/json/version` should return a JSON response with `TradingView/3.1.0` in the User-Agent.

---

## Session Startup Checklist

1. Launch TradingView with `--remote-debugging-port=9222` (above)
2. Open Claude Code
3. Run `tv_health_check` — expect `cdp_connected: true`
4. If `cdp_connected: false`, TradingView didn't expose port — relaunch it and retry

---

## MCP Server

- **Repo:** https://github.com/tradesdontlie/tradingview-mcp (by @tradesdontlie)
- **Entry point:** `C:\Users\bower\tradingview-mcp\src\server.js`
- **Transport:** stdio (Claude Code spawns it via `node`)
- **Protocol:** Chrome DevTools Protocol on `localhost:9222`
- **78 tools** covering: chart state, quotes, OHLCV, Pine Script, indicators, alerts, drawings, watchlists, replay, screenshots, tabs, panes

Key tools:
- `tv_health_check` — verify CDP connection
- `chart_get_state` — get current symbol, timeframe, indicator list
- `quote_get` — real-time price (OHLC, volume)
- `chart_set_symbol` — change ticker
- `pine_set_source` + `pine_smart_compile` — inject and compile Pine Script
- `data_get_study_values` — read indicator values (RSI, MACD, etc.)

---

## Trading Rules (`rules.json`)

```json
{
  "watchlist": ["EIGHTCAP:XAUUSD", "EIGHTCAP:BTCUSD", "EIGHTCAP:EURUSD"],
  "timeframes_to_check": ["1W", "1D", "4H"],
  "risk_rules": {
    "max_risk_per_trade": "1% of portfolio",
    "min_rr_ratio": 2,
    "no_trades_during": ["major US CPI", "FOMC", "weekend thin liquidity"]
  },
  "indicators_i_care_about": ["RSI (14)", "MACD (12, 26, 9)", "50 EMA", "200 EMA", "Volume"]
}
```

---

## Video Structure

The video has 10 sections. Key demo sections:

| Section | Demo | Prompt focus |
|---|---|---|
| 4 | Setup | Master install prompt |
| 5 | The Analyst | Read BTC chart, find setups, draw on chart |
| 6 | The Builder | Generate + inject Pine Script indicator |
| 7 | The Assistant | Entry / stop / target from live chart |
| 8 | The Automator | Scan 10 majors, set alerts, screenshot setups |
| 10 | Phone finale | Telegram voice prompts ("Oscar, read BTC for me") |

Demo prompts and backup prompts are in `Claude + TradingView Prompt.md`.

The AI assistant in the video is referred to as **"Oscar"**.

---

## Key Files

| File | Purpose |
|---|---|
| `CLAUDE.md` | This file — project context for Claude |
| `progress.md` | Step-by-step setup log |
| `Claude + TradingView Prompt.md` | Video script prompts, demo prompts, troubleshooting lines |
| `~/.claude/mcp.json` | MCP server registration |
| `~/.claude/settings.json` | Tool permissions + Stop hook |
| `C:\Users\bower\tradingview-mcp\rules.json` | Crypto trading rules |

---

## Troubleshooting

- **MCP tools missing from session:** TradingView wasn't running when Claude Code started. Launch TV with debug port, then restart Claude Code.
- **`cdp_connected: false`:** Port 9222 not open. Relaunch TradingView with `--remote-debugging-port=9222`.
- **Tools not pre-approved:** Check `~/.claude/settings.json` has `mcp__tradingview__*` in `permissions.allow`.
- **MSIX launch fails:** The exe path contains the version number — if TradingView updates, the path changes. Re-run `Get-AppxPackage -Name "TradingView.Desktop"` to get the new `InstallLocation`.