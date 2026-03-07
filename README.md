# agtop

An htop-like TUI for monitoring AI coding agent sessions. Tracks Claude Code and Codex sessions running on your machine, showing real-time cost, token usage, tool invocations, and OS-level metrics.

![agtop](https://img.shields.io/badge/node-%3E%3D18-brightgreen)

## Features

- **Session discovery** -- automatically finds Claude Code (`~/.claude/projects/`) and Codex (`~/.codex/sessions/`) sessions
- **Cost estimation** -- computes per-session spend using LiteLLM pricing data, with plan-aware billing (retail, max, included)
- **Two list views** -- Summary (all sessions: duration, tokens, cost, tools, model) and Live (running sessions: CPU, memory, rates, incremental tool count)
- **Real-time rates** -- tokens/min and cost/min via EMA smoothing; incremental tool count since startup
- **OS process metrics** -- CPU%, memory, PID count for running sessions (macOS/Linux)
- **Last active tool** -- shows what a running agent is doing right now
- **Overview charts** -- sparkline charts for aggregate spend, tokens, and CPU
- **Detail view** -- full cost breakdown, token split, and per-model stats
- **Tabbed panels** -- Info (identity, cost, tokens), System (CPU/memory charts), Tool Activity, Config (CLAUDE.md/AGENTS.md, memories, skills, MCP servers, permissions)
- **Mouse support** -- click to select sessions, sort by column, switch tabs, clickable menu bar; hover tooltips on column headers
- **Non-interactive modes** -- list table and full JSON dump for scripting

## Requirements

- Node.js >= 18
- No dependencies (single-file, pure Node.js)

## Usage

```
# Interactive TUI (default)
agtop

# List sessions in a table
agtop -l

# Full JSON dump (pipe to jq for filtering)
agtop -j
agtop -j | jq '.[] | select(.cost.total > "1.00")'

# Set billing plan
agtop -p max

# Set refresh interval (seconds)
agtop -d 3
```

## Options

| Flag | Description |
|------|-------------|
| `-l`, `--list` | List sessions in a table and exit |
| `-j`, `--json` | Dump full session data as JSON and exit |
| `-p`, `--plan <plan>` | Billing plan for cost display (default: `retail`) |
| `-d`, `--delay <secs>` | Refresh interval in seconds (default: `2`) |
| `-h`, `--help` | Show help |

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `j`/`k` or arrows | Navigate sessions |
| `Enter` | Open detail view |
| `Tab` | Cycle bottom panel tabs |
| `Shift+Tab` or `` ` `` | Toggle Summary/Live view |
| `1`/`2`/`3`/`4` | Switch to Info/System/Tool Activity/Config panel |
| `F3` or `/` | Search/filter sessions |
| `F6` or `>` | Sort-by panel |
| `P`/`M`/`T` | Sort by status/memory/cost |
| `F5` or `r` | Force refresh |
| `q` or `F10` | Quit |

## Plans

The `-p` flag controls how costs are displayed:

- `retail` (default) -- standard API pricing
- `max` -- Claude Max subscription (Claude usage marked as "included")
- `included` -- all usage marked as included

## JSON Output

`agtop -j` dumps comprehensive session data including:

- Session identity (provider, ID, project, model)
- Cost breakdown (total, per-category, rates)
- Token counts (input, output, cached, detailed splits)
- Activity metrics (tool invocations by name, skills, web fetches/searches, MCP calls)

## How It Works

agtop reads JSONL transcript files written by Claude Code and Codex to extract token counts, tool invocations, and model information. It fetches current model pricing from LiteLLM (cached for 24 hours) to compute cost estimates. For running sessions, it uses `ps` and `lsof` to map OS processes back to sessions and collect CPU/memory metrics.
