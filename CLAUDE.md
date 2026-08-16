# CLAUDE.md

Guidance for Claude Code (and other AI agents) working in this repository.

@BEHAVIORAL-GUIDELINES.md

## What this is

**GSD (Getting Stuff Done)** is a single, self-contained personal to-do app built around the
Getting Things Done framework. The entire application is one file — `gsd.html` — with **no build
step, no dependencies, and no server**. It runs offline over `file://`.

- **User-facing docs:** `README.md`
- **The app:** `gsd.html` (HTML + CSS + JS in one file)
- **Data store:** `data-gsd/` (JSON files the app reads/writes)

## Running & verifying

```
open gsd.html        # macOS — opens in the default browser
```

There are no tests, linters, or CI. To verify a change, open `gsd.html` in Chrome/Edge (the File
System Access API used for folder auto-save is unavailable in Safari/Firefox), link the `data-gsd/`
folder, and exercise the affected behavior directly in the browser.

## Architecture

Everything lives in `gsd.html`:

- **State** — two in-memory arrays (`todos`, `done`) plus `categories`. Awaiting/deferred tasks stay
  in `todos` with `awaiting: true`.
- **Persistence** — the `data-gsd/` folder is the source of truth. Every change writes straight to
  its JSON files via the File System Access API; `localStorage` is a secondary backup/cache. Writes
  are per-file (not atomic across files) — an accepted tradeoff for a single-user app.
- **Directory handle** is persisted in IndexedDB so the folder is remembered across sessions (one
  click to reconnect per browser session).

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
`YYYY-MM-DD` — the task auto-returns to the active list on that date).

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
- **Match the existing style** in `gsd.html` (vanilla JS, terse inline comments).
- `data-gsd/` in this repo is **demo seed data** (a Michael Scott to-do list). It is safe to edit for
  demo purposes but is not anyone's real list.
