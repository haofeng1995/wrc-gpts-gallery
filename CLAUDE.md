# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a zero-dependency, single-file HTML application — a GPTs tool gallery for the "武器研究中心" (WRC / Weapon Research Center) team at Bicome. There is no build step, no package manager, and no framework. Open `武器研究中心_提示詞資源庫_v2.html` directly in a browser to run the app.

## Architecture

Everything lives in one file: `武器研究中心_提示詞資源庫_v2.html`. It is structured as embedded CSS → HTML → JavaScript.

### Data Model

The `GPTS` array (line ~222) is the single source of truth — an array of objects with these fields:

```js
{
  name: string,    // Tool name (Chinese)
  link: string,    // ChatGPT GPT URL
  role: string,    // One of 8 roles (maps to a color via ROLE_COLORS)
  module: string,  // Raw module string (normalized via MODULE_MAP)
  timing: string,  // '日' | '週' | '月' | '季' | ''
  input: string,   // Input description shown on card
  output: string,  // Output description shown on card
  desc: string,    // Short description shown on card
}
```

### Module System

Raw `module` values in `GPTS` are non-canonical — multiple raw values map to the same display group. The mapping chain is:

1. `MODULE_MAP` — normalizes raw module strings (e.g., `'livetalk模組'` → `'主題模組'`, `'全局詢問'` → `'AI輔助'`)
2. `MODULE_GROUPS` — maps canonical group keys to emoji display labels
3. Filter buttons use the canonical group keys as `data-filter` attributes

When adding a new tool with a new raw `module` value, you must add it to `MODULE_MAP`; otherwise it silently falls back to `'AI輔助'`.

### Rendering Flow

State is held in two module-level variables: `currentFilter` and `currentSearch`. Every filter click or search keystroke calls `renderCards()`, which wipes `#cardGrid` and rebuilds from scratch. `makeCard()` generates card HTML strings via template literals and sets `--role-color` as a CSS custom property directly on the element — this drives the top border stripe, badge color, and hover border.

### User Persistence

- `localStorage.wrc_user` — user's name; controls whether the name modal is shown on load
- `localStorage.wrc_log` — JSON array of `{ user, tool, time }` click events appended by `logClick()`

Clicking a user's name tag in the header calls `resetUser()`, which clears both localStorage keys after a `confirm()`.

## Conventions

- The "last updated" date in the header (`最後更新：2025/06/06`) is hardcoded at line ~151 and must be updated manually when tools are added or changed.
- All text is Traditional Chinese (zh-TW). Keep new UI strings in Traditional Chinese.
- Role names in `GPTS` entries must exactly match a key in `ROLE_COLORS`; unmatched roles silently fall back to `--accent` blue.
- The canonical module display order in the "全部" view is hardcoded in the `order` array inside `renderCards()` — add new canonical groups there if introducing a new module.
- Card HTML is built via innerHTML template literals in `makeCard()`; single quotes in tool names are escaped with `\'` in the `onclick` attribute.
