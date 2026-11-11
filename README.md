# Perso — one list, newest first

**Every profile's sessions, in one list. Sorted by when, not by who.**

Perso is a copy of [Personae](https://github.com/SmoothMarx/personae) with the
roster view replaced. Instead of a row per profile that expands into its
sessions, the pane draws **one flat list of every profile's sessions, ordered by
activity**, and each row carries the profile it belongs to — picture and name —
so a single stream is still readable.

It is a desktop plugin for the **Hermes Desktop** app. It ships no build step:
one plain-ESM `desktop/plugin.js` the app loads at runtime, plus a small
dashboard backend for the two reads the SDK cannot reach.

---

## What it does

| Capability | What it gives you |
| --- | --- |
| 🗒️ **One flat list** | Every profile's sessions merged, newest activity first — no expanding, no per-profile rows |
| 🖼️ **Who it belongs to** | Status dot → the profile's picture → the title → its name, on every row |
| 🟡 **Status lights** | Per session: amber waiting on you, blue working, emerald unread — the pane paints the session's own dot |
| 📄 **A page, then a door** | 50 rows, then *Show more (n)* — the row just past the page is always reachable |
| 🗄️ **Archived section** | A collapsible section at the bottom of the same list, read from the pane's own backend (the gateway cannot list archived sessions at all) |
| 🗂️ **Groups** | Named session groups still draw as boxes, anchored where their **newest** member would have been |
| ◆ **Apropos** *(beta)* | A fresh session seeded with the background of the ones you picked — same verbs, same digest |
| ✅ **Multi-select** | Tick a set, then archive, close, delete or Apropos them in one go |
| 🖱️ **The same menus** | Session, Group, Appearance, Assign to project — the roster's verbs, unchanged |
| 🧩 **Shared state** | Groups, Apropos links, hidden flags and sections are **Personae's own document**, so a change in one pane (or on the phone) shows in the other |
| 🔍 **Search & filter** | Server-side title + message-body search; filter by activity, kind, or a profile set |

## What it deliberately does not do

- **No profile rows, no sections, no per-profile drill-downs.** The unit here is
  the session; a profile is an attribute of a row, not a container.
- **No profile aggregate dot.** A flat list shows each session's own light
  (Personae rolls them up per profile).
- **No drag-and-drop.** A session dragged across a flat list could file an id
  into a group that belongs to another profile, so the gesture is off; *Move to
  group ▸* in the row menu is the same verb, scoped correctly and unable to fail
  silently.
- **No Scheduled-jobs tile.** That pane belongs to Personae's layout.

## One list has one consequence

A group box holds its members together, so a grouped session is not ordered by
its own activity — it rides with its box, at the box's newest member. Everything
ungrouped is strictly by activity, with a total tiebreak (profile, then id) so
two rows at the same second cannot swap places between polls.

## Install

Two halves, two hosts — they drift silently if you ship one:

1. **Desktop half** — copy `desktop/plugin.js` into the app's own home:
   `<app home>/desktop-plugins/perso/plugin.js`
   (`%LOCALAPPDATA%\hermes\desktop-plugins\perso\` on Windows). `deploy.sh` does
   the `scp` and prints both hashes; the app hot-reloads, ⌘K → **Reload desktop
   plugins** is the fallback.
2. **Backend half** — copy this package's `dashboard/*.py` onto the gateway box
   as `<hermes home>/plugins/perso/dashboard/` (`deploy-backend.sh`), then
   **restart** the dashboard process: plugin API routers mount at import, so the
   file on disk is not what answers until it re-imports.

A user plugin is **opt-in per profile**: the loader skips it unless `perso` is in
that profile's `plugins.enabled`, and the API gate answers
`404 {"detail": "Plugin not found"}` for a profile that has not consented —
which reads exactly like a missing install. Until the backend mounts, Perso says
the archive is unavailable rather than drawing an empty one.

## State

`dashboard/ui_state.py` resolves the store to `plugins/personae/state/`
**when that directory exists**, so both panes and Personae Mobile write one
document (entry-wise last-writer-wins by rev, tombstones, a file lock). On a
machine without Personae it falls back to `plugins/perso/state/`.
`MESSENGER_STATE_DIR` overrides either.

## Tests

```bash
./tests/run.sh
```

Three suites: `run.mjs` renders the pane headlessly with `react-dom/server` and
asserts on the tree and on the pure decision layer; `classes.mjs` proves every
Tailwind token exists in the app's already-compiled stylesheet; `icons.mjs` does
the same for codicon names. It needs a Hermes Desktop checkout with
`node_modules` installed:

```bash
HERMES_DESKTOP_REPO=~/path/to/checkout ./tests/run.sh
```

The ordering rules — activity, the tiebreak, a box anchored at its newest
member, an archived row never in the live list — are pinned as pure functions, so
a refactor cannot quietly re-sort the list.

## Layout

```
desktop/plugin.js        the pane (the whole UI half)
dashboard/plugin_api.py  /archived, /resolve, /search, /title, /state
dashboard/ui_state.py    the shared store (Personae's document)
tests/                   headless harness + live E2E
shared/                  the vendored markdown-to-text module
```

## License

GPL-3.0-or-later, inherited from Personae.
