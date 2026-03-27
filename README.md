# agtop

Run with:
```
npx github:dkutsanov/agtop
```

Your window into what your AI coding agents are actually doing. agtop is a top-style terminal dashboard that tracks every Claude Code and Codex session on your machine -- spend, token usage, context pressure, CPU load, tool invocations, and more -- all in one place, live.

![agtop](https://img.shields.io/badge/node-%3E%3D18-brightgreen)
![License: GPL v2](https://img.shields.io/badge/License-GPL%20v2-blue.svg)

<img src="screenshots/main-list.png" width="700" alt="Session list with live CPU and cost metrics">

## Features

- **Session discovery** -- automatically finds Claude Code (`~/.claude/projects/`) and Codex (`~/.codex/sessions/`) sessions
- **Cost tracking** -- per-session spend with plan-aware billing (retail, Max, included)
- **Context pressure** -- %CONTEXT shows how full each agent's context window is, with [1M] model annotation
- **Live toggle** -- filter to running sessions with real-time %CPU, cost rates, and incremental tool counts
- **Panel focus navigation** -- press `0` to focus the session table, `1` to focus the detail panel; arrow keys navigate the active panel
- **Tool Activity panel** -- scrollable per-tool invocation history with timestamps; see exactly what each agent has been doing
- **OS process metrics** -- %CPU and memory for running sessions (macOS/Linux)
- **Overview header** -- k9s-style borderless header with metrics (CPU, memory, spend, tokens), account limits, and keyboard shortcut reference
- **Detail panels** -- Info, Performance, Processes, Tool Activity, Cost, and Config tabs with scrollable content
- **Config panel** -- browse Instructions, Permissions, Memory, Commands, Agents, Skills, and MCP servers per session
- **Light and dark themes** -- auto-detects terminal background via OSC 11; Catppuccin Latte palette for light terminals, btop-style dark palette; override with `--theme light|dark`
- **Mouse support** -- click to select, sort by column, switch tabs; hover tooltips on column headers
- **Non-interactive modes** -- table and full JSON dump for scripting

## Requirements

- Node.js >= 18
- No dependencies (single-file, pure Node.js)

## Usage

```
# Run the TUI (no install needed)
npx github:dkutsanov/agtop

# Or install globally
npm install -g github:dkutsanov/agtop

# Run the installed version
agtop

# Set refresh interval (seconds)
agtop -d 3

# Force a specific theme
agtop --theme light
agtop --theme dark
```

## Options

| Flag | Description |
|------|-------------|
| `-l`, `--list` | List sessions in a table and exit |
| `-j`, `--json` | Dump full session data as JSON and exit |
| `-p`, `--plan <plan>` | Billing plan for cost display (default: `retail`) |
| `-d`, `--delay <secs>` | Refresh interval in seconds (default: `2`) |
| `--theme <light\|dark>` | Color theme (default: auto-detect from terminal) |
| `-h`, `--help` | Show help |

## Keyboard Shortcuts

| Key | Action |
|-----|--------|
| `0` | Focus session table |
| `1` | Focus detail panel |
| `j`/`k` or arrows | Navigate focused panel |
| Left/Right (detail) | Switch detail panel tab |
| `Tab` | Cycle detail panel tabs |
| `` ` `` | Toggle Live filter |
| `F3` or `/` | Search/filter sessions |
| `F5` or `r` | Force refresh |
| `F6` | Sort-by panel |
| `F7` | Filter by age |
| `d` | Delete selected session |
| `q` or `F10` | Quit |

## Themes

agtop automatically detects your terminal background color and picks the appropriate theme:

- **Dark theme** -- btop-style palette with vibrant accents on dark backgrounds
- **Light theme** -- Catppuccin Latte palette designed for light terminals (e.g. Ghostty with Atom One Light)

Use `--theme light` or `--theme dark` to override auto-detection.

## JSON Output

`agtop -j` dumps comprehensive session data including:

- Session identity (provider, ID, project, model)
- Cost breakdown (total, per-category, hourly/daily rates)
- Token counts (input, output, cached, detailed splits)
- Activity metrics (tool invocations by name, skills, web fetches/searches, MCP calls)

## Screenshots

**Info panel** -- session identity, wall/API time, context pressure

<img src="screenshots/panel-info.png" width="600" alt="Info panel">

**Tool Activity** -- per-tool invocation counts and live feed with timestamps

<img src="screenshots/panel-tool-activity.png" width="600" alt="Tool Activity panel">

**Cost breakdown** -- total spend by time window, per-model token and cost split

<img src="screenshots/panel-cost.png" width="600" alt="Cost panel">

**Config** -- browse Instructions, Permissions, Memory, Commands, Agents, Skills, and MCP servers

<img src="screenshots/panel-config.png" width="600" alt="Config panel">

## How It Works

agtop reads JSONL transcript files written by Claude Code and Codex to extract token counts, tool invocations, and model information. It fetches current model pricing from LiteLLM (cached for 24 hours) to compute cost estimates. For running sessions, it uses `ps` and `lsof` to map OS processes back to sessions and collect CPU/memory metrics.
