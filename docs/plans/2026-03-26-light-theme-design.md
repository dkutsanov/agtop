# Light Theme Support Design

## Summary

Add a Catppuccin Latte light theme to agtop with automatic terminal background detection and a `--theme` CLI override.

## Decisions

| Decision | Choice | Rationale |
|----------|--------|-----------|
| Theme selection | Auto-detect via OSC 11 + `--theme` CLI override | "Just works" on supported terminals (ghostty), flag as escape hatch |
| Color palette | Catppuccin Latte | Matches user's TUI ecosystem (k9s, neovim); rich semantic palette |
| Backgrounds | Transparent base, explicit selection/header backgrounds | Matches k9s Catppuccin skin approach; avoids "wall of color" |
| Runtime toggle | Not included (startup only) | YAGNI; auto-detect covers the common case |
| Gradient hues | Same green/yellow/red progression, Catppuccin values | Universal semantic meaning for metrics |

## Architecture

Two static color palette objects (`DARK` and `LIGHT`) resolved once at startup into the existing `C` variable. All downstream rendering code continues referencing `C.border`, `C.hdrLabel`, etc. with zero changes.

### Detection Flow

1. If `--theme light` or `--theme dark` CLI flag is present, use it
2. Otherwise, query terminal background via OSC 11 (`\x1b]11;?\x07`)
3. Parse RGB response, compute luminance (`L = 0.2126*R + 0.7152*G + 0.0722*B`)
4. If `L > 0.5` -> light theme, else -> dark theme
5. Timeout (100ms) or parse failure -> fall back to dark (backward compatible)

Detection runs before entering raw mode / alt screen, as a one-shot async operation.

### What Changes in `index.js`

1. **Argument parser** (~5 lines) - add `--theme` option
2. **`detectTheme()` function** (~30 lines) - OSC 11 query with timeout
3. **`DARK` object** - rename existing `C` to `DARK`, no value changes
4. **`LIGHT` object** (~40 lines) - Catppuccin Latte ANSI 256 codes, same property names
5. **Light gradient functions** (~20 lines) - `sparkColorLight`, `sparkColorSpendLight`, `sparkColorAccentLight`
6. **Theme resolution** (~10 lines) - assign `C` and bind gradient functions based on theme
7. **`ageDimColor`** (~5 lines) - branch on theme for fade direction
8. **`modelColor`** (~3 lines) - branch on theme for model accent colors

**Total: ~120 lines added, ~10 lines modified. No new files. No new dependencies.**

## Color Mapping: Catppuccin Latte

### Borders & Panels

| Property | Dark (current) | Light (Catppuccin Latte) |
|----------|---------------|--------------------------|
| `border` | ANSI 60 (muted blue-gray) | ~247 Overlay0 `#9ca0b0` |
| `borderHi` | ANSI 75 (bright blue) | ~33 Blue `#1e66f5` |
| `panelTitle` | ANSI 179 bold (gold) | ~208 bold Peach `#fe640b` |

### Labels & Values

| Property | Dark (current) | Light (Catppuccin Latte) |
|----------|---------------|--------------------------|
| `hdrLabel` | ANSI 75 bold | ~33 bold Blue `#1e66f5` |
| `hdrValue` | Bold white | ~59 bold Text `#4c4f69` |
| `hdrCyan` | Bold cyan | ~30 bold Teal `#179299` |
| `hdrGreen` | Bold green | ~34 bold Green `#40a02b` |
| `hdrYellow` | Bold yellow | ~136 bold Yellow `#df8e1d` |
| `hdrDim` | ANSI 245 | ~243 Subtext0 `#6c6f85` |

### Column Headers & Selection

| Property | Dark (current) | Light (Catppuccin Latte) |
|----------|---------------|--------------------------|
| `colHdrBg` | Bold white on ANSI 236 | ~59 Text on Surface0 `#ccd0da` |
| `selBg` | ANSI 236 bg | Surface0 `#ccd0da` bg |
| `selFg` | Bold white | ~59 bold Text `#4c4f69` |

### Footer

| Property | Dark (current) | Light (Catppuccin Latte) |
|----------|---------------|--------------------------|
| `footerKey` | Black on ANSI 75 | Base `#eff1f5` on Blue `#1e66f5` |
| `footerLabel` | ANSI 245 on black | ~243 Subtext0 on default |
| `footerBg` | Black (40) | Default (transparent) |

### Semantic Colors

| Property | Dark (current) | Light (Catppuccin Latte) |
|----------|---------------|--------------------------|
| `costGreen` | ANSI 114 | ~34 Green `#40a02b` |
| `costYellow` | ANSI 221 | ~136 Yellow `#df8e1d` |
| `costRed` | ANSI 203 bold | ~160 bold Red `#d20f39` |
| `provClaude` | ANSI 141 (purple) | Mauve `#8839ef` |
| `provCodex` | ANSI 114 (green) | Green `#40a02b` |
| `dimText` | ANSI 245 | ~243 Subtext0 `#6c6f85` |
| `normalFg` | White (37) | ~59 Text `#4c4f69` |
| `searchFg` | Bold cyan | ~30 bold Teal `#179299` |
| `accent` | ANSI 75 | ~33 Blue `#1e66f5` |

## Gradient Functions (Light Variants)

### `sparkColor` / `sparkColorSpend` (CPU / Cost)

| Range | Dark (ANSI) | Light (ANSI) |
|-------|-------------|--------------|
| <1% | 22 (dark forest green) | 251 (light gray) |
| 1-30% | 71 (muted green) | 35 (dark green) |
| 30-50% | 114 (bright green) | 34 (green) |
| 50-70% | 186 (yellow-green) | 136 (yellow) |
| 70-85% | 221 (yellow) | 208 (peach) |
| >85% | 203 (red) | 160 (red) |

### `sparkColorAccent` (Non-CPU Metrics)

| Range | Dark (ANSI) | Light (ANSI) |
|-------|-------------|--------------|
| <1% | 238 (dark gray) | 252 (light gray) |
| 1-25% | 60 (muted blue) | 110 (light blue) |
| 25-50% | 68 (steel blue) | 68 (steel blue) |
| 50-75% | 75 (bright blue) | 33 (blue) |
| >75% | 117 (cyan) | 30 (teal) |

### `ageDimColor`

Sessions fade from Text `#4c4f69` (running) toward Overlay0 `#9ca0b0` (old) instead of white toward dark gray.

### `modelColor`

- Claude: Mauve `#8839ef` (instead of ANSI 173 orange)
- OpenAI: Sapphire `#209fb5` (instead of ANSI 110 blue)

## Out of Scope

- Runtime theme toggle / keybind
- Theme persistence in ui-prefs.json
- True color (24-bit) support - staying with ANSI 256
- Custom / user-defined themes
