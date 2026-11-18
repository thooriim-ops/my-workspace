# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project

Korean-language bucket list web app ("나의 버킷 리스트"). Pure static site: vanilla JS (ES6+), no framework, no build step, no package.json, no tests, no linter. Tailwind CSS is loaded via CDN (`cdn.tailwindcss.com`), so an internet connection is needed for styling.

## Running

Open `index.html` directly in a browser, or serve statically (e.g. `python3 -m http.server 8000`). Verify changes manually in the browser.

## Architecture

Two scripts loaded as plain `<script>` tags in `index.html` (order matters: `storage.js` before `app.js`; they share globals, no modules):

- `js/storage.js` — `BucketStorage` object literal; the only code that touches `localStorage` (key `bucketList`). Every method does a full `load()` → mutate → `save()` cycle on the JSON array; there is no in-memory state. Also provides `getStats()` and `getFilteredList(filter)` (`'all' | 'active' | 'completed'`).
- `js/app.js` — `BucketListApp` class, instantiated into the global `app` on `DOMContentLoaded`. Flow: `cacheElements()` → `bindEvents()` → `render()`. Every state change calls `render()`, which recomputes stats and rebuilds the whole list via `innerHTML` from `createBucketItemHTML()`.
- Per-item buttons use inline `onclick="app.handleToggle('id')"` etc., so `app` must remain a global and handler methods must stay public. Titles are interpolated into these attributes via `escapeHtml` plus a quote replace — be careful when changing item markup (quoting/XSS).
- `css/styles.css` supplements Tailwind: `.filter-btn.active` styling, animations, mobile layout (≤640px relies on `.bucket-item` child order — checkbox first, button group last), and a partial dark-mode override.

Item shape: `{ id (Date.now() string), title, completed, createdAt (ISO), completedAt (ISO|null) }`. Changing it affects existing users' stored data.

## Notes

- UI strings, comments and JSDoc are in Korean; keep that convention.
- README lists `assets/` in the structure, but the directory does not exist. Unimplemented roadmap items in the README: categories/tags, images, memos, due dates, priority, JSON export/import.
