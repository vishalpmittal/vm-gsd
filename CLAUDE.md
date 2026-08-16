# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

@BEHAVIORAL-GUIDELINES.md

## What this is

**GSD (Getting Stuff Done)** is a single, self-contained personal to-do app built around the
Getting Things Done framework. The entire application is one file — `gsd.html` — with **no build
step, no dependencies, no server, and no package manager**. It runs offline over `file://`.

- **The app:** `gsd.html` (HTML + CSS + JS in one file, ~1400 lines)
- **Data store:** `data-gsd/` (JSON files the app reads/writes)
- **User-facing docs:** `README.md`

## Running & verifying

```
open gsd.html        # macOS — opens in the default browser
```

There are no tests, linters, or CI — verification is manual. Open `gsd.html` in **Chrome or Edge**
(the File System Access API used for folder auto-save is unavailable in Safari/Firefox), pick the
`data-gsd/` folder at the startup gate, and exercise the affected behavior directly in the browser.
On `file://`, the startup gate always requires choosing a folder before the app is usable.

## Architecture

Everything lives in `gsd.html` as one IIFE. Key ideas that span the file:

- **Single source array.** All tasks live in one in-memory `todos` array; completed tasks are just
  flagged `done: true`. The split into `to-do-tasks.json` / `done-tasks.json` happens **only at write
  time** (`writeToDir`) and is merged back into one array on read (`ingestDir`). Awaiting/deferred
  tasks also stay in `todos`, flagged `awaiting: true`.
- **The `data-gsd/` folder is the database.** Every mutation calls `saveLocal()` →  `persist()`,
  which write-throughs to both `localStorage` (backup/cache) and, if linked, the folder via the File
  System Access API. Writes are per-file (not atomic across files) — an accepted single-user tradeoff.
- **Folder handle persistence.** The chosen directory handle is stored in IndexedDB (`idbGet`/`idbSet`)
  so the folder is remembered across sessions; each new browser session needs one click to re-grant
  permission (the "reconnect" startup-gate mode).
- **Startup gate** (`showGate`) blocks the app until a folder is selected/reconnected, with distinct
  modes: `loading` / `select` / `reconnect` / `unsupported` (non-Chromium browsers fall back to
  in-browser storage + manual export).
- **Rendering** is full re-render on change: `render()` rebuilds both task lists from `todos`;
  `rowHtml()` renders a row for both the active and awaiting lists. Menus/panes are built as HTML
  strings (`renderMainMenu`, etc.).
- **Theming** uses CSS custom properties on `:root`. A `data-theme` attribute on `<html>`
  (`system`/`light`/`dark`, persisted in `localStorage`) selects the palette; `system` and the
  pre-JS default fall through to the `prefers-color-scheme` media query.
- **Backup export** is a dependency-free STORE-only ZIP writer (`makeZip`/`crc32`) so the app stays a
  single offline file with no library.

## Data model

Files in `data-gsd/`:

| File | Contents |
|------|----------|
| `to-do-tasks.json` | Active (not-done) tasks, including `awaiting` ones. |
| `done-tasks.json` | Completed tasks. |
| `categories.json` | Managed category list (array of strings). |
| `meta.json` | `{ app, schemaVersion, updatedAt }` — schema version marker. |

A **task** object:

```json
{
  "id": "string", "created": 0, "done": false,
  "title": "string", "description": "string",
  "priority": "High|Medium|Low", "due": "YYYY-MM-DD or ''",
  "category": "one of categories, custom string, or ''",
  "size": "xs|s|m|l|xl"
}
```

Awaiting tasks additionally carry `awaiting: true`, `awaitNote`, and `awaitUntil` (a future
`YYYY-MM-DD` — the task auto-returns to the active list on that date via `reactivateDue()`).

## Schema versioning

`gsd.html` owns the current `SCHEMA_VERSION` and an ordered `MIGRATIONS[n]` set (each upgrades the
`{ todos, categories }` model from v`n-1` → v`n`). On load, an older folder is migrated up to the
latest version (a folder with no `meta.json` is treated as v1); before overwriting, the app snapshots
the current files into `data-gsd/backups/` (git-ignored). Newer folders load as-is with a warning and
are never downgraded.

**To evolve the schema:** bump `SCHEMA_VERSION` and add the matching `MIGRATIONS[n]` step — nothing
else is required. Migrations must preserve unknown/custom fields (spread the original, then coerce
known fields).

## Conventions

- **Keep it dependency-free and single-file.** Do not introduce a build step, framework, or npm
  packages without being asked.
- **Match the existing style** in `gsd.html` (vanilla JS, `const $ = …` DOM helpers, terse inline
  comments explaining the "why").
- `data-gsd/` in this repo is **demo seed data** (a Michael Scott to-do list). It is safe to edit for
  demo purposes but is not anyone's real list.
