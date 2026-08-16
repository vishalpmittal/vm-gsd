# GSD — Getting Stuff Done

A single, self-contained to-do app for managing **one personal todo list**, built around the
[Getting Things Done](https://en.wikipedia.org/wiki/Getting_Things_Done) framework. It helps you
navigate daily tasks by their attributes — priority, due date, category, size — rather than staring
at a flat list.

It's one HTML file. **No install, no accounts, no dependencies, works offline.**

> The name is **Getting Stuff Done** — though if you read the "S" a little more colourfully, that's
> on you. 😏

## Running it

Open `gsd.html` in a browser — from the terminal: `open gsd.html` (macOS).

On first launch (Chrome/Edge) it asks you to **choose a `data-gsd/` folder** to store your tasks.
Pick the included `data-gsd/` folder to explore the demo list, or an empty folder of your own to
start fresh. From then on every change auto-saves there, and the folder is remembered across
sessions (one click to reconnect each new browser session).

> Live folder auto-save needs the File System Access API (Chrome/Edge). On Safari/Firefox you can
> still use the app with in-browser storage and export a backup manually.

The included `data-gsd/` is a **demo dataset** — Michael Scott's (Dunder Mifflin, Scranton) to-do
list — so you have something to click around before pointing the app at your own folder.

## Task fields

Each task has:

- **title** — short name of the task
- **description** — optional detail / next action
- **priority** — High · Medium · Low
- **due date** — optional
- **category** — one of the managed categories (defaults: work, career, family, hobbies, essentials,
  investments, savings), a **custom** value entered inline, or **none**
- **size** — xs, s, m, l, xl (effort estimate)

## Features

- **Add** — the floating **+** button (bottom-right) opens a right-side pane with the task fields.
- **Edit / delete** — click any task row to open the same pane pre-filled; edit, toggle done, or
  delete from there.
- **Duplicate** — hover a task row to reveal a **⧉** icon; click it to copy that task (titled
  "copy of - <original title>", reset to not-done).
- **Awaiting (defer / waiting-on)** — hover a task and click **⏳** to move it out of the active list.
  You note *what you're waiting on* and a *bring-back date*; the task lives in a separate **Awaiting**
  section and **returns to the main list automatically on that date**. Use **↩** to pull it back
  immediately.
- **Search / filter / sort** — search text; filter by status, category, priority; sort by:
  - **✨ Smart (auto)** — a computed score blending urgency (due date) + priority + a quick-win boost
    for smaller tasks, so the most important-and-urgent tasks rise to the top.
  - or Due date · Priority · Size · Category · Created.
- **Manage categories** — the **🗂 Categories** pane (in the ☰ menu) lets you add, rename, and delete
  categories. Deleting one that has tasks leaves those tasks uncategorized rather than removing them.
- **Menu (☰ top-right)** — data-folder status / auto-save indicator, **📂 Switch data folder**,
  **⬇ Export backup (.zip)**, a **🎨 Legend** (colour key), and the **🗂 Categories** and **ℹ️ About**
  panes.
- **Color coding** (full key under **🎨 Legend**)
  - Left border + tinted •dot chip = **priority**: 🔴 High · 🟠 Medium · 🟢 Low
  - Solid pill = **category**; grey badge = **size** (xs–xl)
  - 📅 Due chip = **urgency**: red = overdue, amber = due within 2 days
  - ⏳ Purple chip = **awaiting** (auto-returns on its bring-back date)
  - Completed tasks are dimmed with a strikethrough

## Your data

Your tasks live in the `data-gsd/` folder as plain JSON — the app treats that folder as its database
and auto-saves on every change. Because it's just files, your list is easy to back up, diff, or move.

- **⬇ Export backup (.zip)** (☰ menu) downloads a timestamped zip of your data; unzip to restore.
- **📂 Switch data folder** points the app at a different folder.
- Choosing a folder never wipes your tasks: if the folder has tasks they win; if it's empty, the
  app re-seeds it with your current list.

---

Working on the app itself? See [`CLAUDE.md`](./CLAUDE.md) for architecture, the data model, and
schema-versioning notes.
