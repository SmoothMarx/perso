# Perso — one list, newest first

**Personae's little brother. Only the session list — with all the goodies.**

Where [Personae](https://github.com/SmoothMarx/personae) hands you a full roster
to open profile by profile, Perso keeps just the list: **every profile's sessions
in one stream, ordered by activity**. Each row carries the profile it belongs to —
picture and name — so one stream across twenty profiles still reads.

Everything Personae is known for comes along: sessions arrive sorted into
**Unread**, **Running** and **Read**, and any of them can be **grouped together
by topic** (or whichever line you want to draw), with colour, search, an archived
section and a row menu that acts on the whole selection.

It is a desktop plugin for the **Hermes Desktop** app. It ships no build step:
one plain-ESM `desktop/plugin.js` the app loads at runtime, plus a small
dashboard backend for the two reads the SDK cannot reach.

---

## What it does

| Capability | What it gives you |
| --- | --- |
| 🗒️ **One flat list** | Every profile's sessions merged, newest activity first — no expanding, no per-profile rows |
| 🖼️ **Who it belongs to** | Status dot → the profile's picture → the title → its name, on every row |
| 💬 **Two-line rows** | The title, and under it the last line of that conversation — a row tall enough to read at a glance |
| 🗂️ **Dividers** | The list is cut into **Unread**, **Running** and **Read** — what needs your eyes, what is working, what you have already seen — and each group is split the same way inside its box |
| 🙈 **Hide all up to here** | Right-click a session → both it and everything **created before it, in every profile**, leave the Perso view (reversible from the hidden toggle) |
| 🟡 **Status lights** | Per session: amber waiting on you, blue working, emerald unread — the pane paints the session's own dot |
| 📄 **A page, then a door** | 50 rows, then *Show more (n)* — the row just past the page is always reachable |
| 🗄️ **Archived section** | A collapsible section at the bottom of the same list, read from the pane's own backend (the gateway cannot list archived sessions at all) |
| 🗂️ **Groups** | Named session groups draw as boxes **at the top** of the list, above the loose sessions. A box holds one profile's sessions, so it wears the profile's picture on its heading and its rows drop the repeated picture and name |
| ◆ **Apropos** *(beta)* | A fresh session seeded with the background of the ones you picked — same verbs, same digest |
| ✅ **Multi-select** | Tick a set, then archive, close, delete or Apropos them in one go |
| 🖱️ **The same menus** | Session, Group, Appearance, Assign to project — the roster's verbs, unchanged |
| 🧩 **Shared state** | Groups, Apropos links, hidden flags and sections are **Personae's own document**, so a change in one pane (or on the phone) shows in the other |
| 🖼️ **Empty-chat hero** | A session you open or start here begins blank — the profile's picture and its name take the empty screen, the way Bot Mode introduces a bot chat. `+` creates the session up front (a draft has no id, so nothing could introduce it), which means a real "New session" row exists until you type in it or delete it |
| 🔍 **Search & filter** | Server-side title + message-body search; filter by activity, kind, or a profile set |

## What it deliberately does not do

- **No profile rows, no sections, no per-profile drill-downs.** The unit here is
  the session; a profile is an attribute of a row, not a container.
- **No profile aggregate dot.** A flat list shows each session's own light
  (Personae rolls them up per profile).
- **No session drag, but groups drag.** A session dragged across a flat list
  could file an id into a group that belongs to another profile, so that gesture
  is off — *Move to group ▸* in the row menu is the same verb, scoped correctly
  and unable to fail silently. **Group boxes** do drag: grab a heading and drop
  it on another box (or its heading) to re-order them.
- **No tile of its own.** The **Scheduled jobs** pane rides along as the same
  tile Bots puts beside the chat: registered only while Perso is on screen, and
  collapsed by default so it takes no width off the chat.

## The order

**Group boxes come first.** A box is a place you put things, so it is not buried
under a busy morning's traffic: every box sits above the loose list, and the boxes
order themselves by their newest member so they do not jump around.

Below the boxes the list is cut into three **dividers**:

| Divider | What lands there |
| --- | --- |
| **Unread** | finished running and **not opened since** — the pane remembers when you opened a row, so a stale backend watermark cannot trap a session here. Anything **waiting on your answer** lands here too, which is neither running nor read |
| **Running** | working right now |
| **Read** | everything else |

Inside a divider the order is strictly by activity — newest first — with a total
tiebreak (profile, then id) so two rows at the same second cannot swap places
between polls.

A box gets the same three dividers inside it: a grouped session is still unread,
running or read, and a box is not a place to lose that. It is the one place a
box's own filing order gives way — inside, the question is again "what needs my
eyes". A box never repeats the profile picture or name its heading already
shows.

## Install

Two halves, two hosts — they drift silently if you ship one:

1. **Desktop half** — copy `desktop/plugin.js` into the app's own home:
   `<app home>/desktop-plugins/perso/plugin.js`
   (`%LOCALAPPDATA%\hermes\desktop-plugins\perso\` on Windows). `deploy.sh` does
   the `scp` and prints both hashes — it knows no hosts, so you name yours:

   ```bash
   HOST=user@box \
   WIN_DIR='C:/Users/you/AppData/Local/hermes/desktop-plugins/perso' \
   ./deploy.sh
   ```

   The app hot-reloads in seconds; ⌘K → **Reload desktop plugins** is the
   fallback.
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
the same for codicon names. Before any of them, `tests/portability.sh` scans the
tracked tree and fails the run if a file names a private host, a real home
directory or a LAN/tailnet address — an artefact that only runs on its author's
machine is tested by the first stranger instead. It needs a Hermes Desktop
checkout with `node_modules` installed:

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
