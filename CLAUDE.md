# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a static, single-file HTML web application — a GPTs resource gallery for Bicome's Weapon Research Center (武器研究中心). It lists 31 curated ChatGPT custom GPTs organized by business function for community/group management workflows.

**There is no build process, package manager, test suite, or linting toolchain.** The entire application lives in one file: `武器研究中心_提示詞資源庫_v2.html`. Open it directly in any modern browser.

## Architecture

The file is self-contained with inline CSS (lines 8–127), HTML markup (lines 129–186), and JavaScript (lines 188–403). No external dependencies or CDNs.

### Data model

Each GPT entry in the `GPTS` array has:
- `name` — tool name (Traditional Chinese)
- `link` — ChatGPT custom GPT URL
- `role` — one of 8 personas (Paul, 規劃者, 分析者, 執行者, 溝通者, 創意者, 監督者, 策略者)
- `module` — category key used for filtering (maps via `MODULE_MAP`)
- `timing` — frequency tag (日/週/月/季)
- `input` / `output` — expected content types
- `desc` — short description

### Mapping objects (top of the JS block)

| Object | Purpose |
|---|---|
| `ROLE_COLORS` | Maps role name → hex color for card badges |
| `MODULE_MAP` | Normalizes raw module strings → canonical category keys |
| `MODULE_GROUPS` | Canonical key → display label with emoji |
| `TIMING_LABELS` | Timing shorthand → human-readable label |

### State & rendering

- `currentUser` — persisted in `localStorage` (`wrc_user`); a name-input modal blocks the UI until set
- `currentFilter` — active module category or `"all"`; set by filter bar buttons
- `currentSearch` — live search string; filters across name, desc, and role
- `renderCards()` — main render: applies filter + search, groups by module when filter is `"all"`, calls `makeCard()` per item
- `logClick(name)` — appends `{user, name, time}` to `localStorage` (`wrc_log`) on each GPT open

### UI regions

- **Name modal** — shown on first visit; persists username
- **Filter bar** — one button per module + "全部"; updates `currentFilter` and re-renders
- **Search input** — live filtering on every keystroke
- **Card grid** — responsive CSS grid (`repeat(auto-fill, minmax(320px, 1fr))`); cards fade in via CSS animation

## Conventions

- All text content is Traditional Chinese; keep new entries in Traditional Chinese.
- Card styling is driven entirely by inline styles computed in `makeCard()` using `ROLE_COLORS`. Add new roles there before using them in data.
- To add a new GPT: append an object to the `GPTS` array following the existing shape, and ensure its `module` value is handled by `MODULE_MAP` and `MODULE_GROUPS`.
- To add a new module category: add an entry to both `MODULE_MAP` and `MODULE_GROUPS`, then add a filter button in the HTML filter bar.
- localStorage keys: `wrc_user` (username string), `wrc_log` (JSON array of click events).
