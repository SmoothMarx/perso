# Perso — catalog entry (draft, not submitted)

Status: **blocked on one asset** — the only screenshots in `assets/` are Personae's
(old roster layout, Oct 11–19). A catalog card showing a different plugin is worse
than no card, so the entry waits for a real capture of the Perso pane.

Everything else is ready: repo public, default branch `main`, HEAD `033d0bc`,
`tests/portability.sh` PASS, harness 894/894 + 155/155 class tokens at that commit.

## Entry

```yaml
name: perso
repo: https://github.com/SmoothMarx/perso
sha: 033d0bc7a1e91c19c64dc10eed661aee5447ce02
description: "Personae's little brother: instead of a roster to expand profile by profile, just the list — every profile's sessions in one stream, ordered by activity, each row carrying its profile's picture and name. Sessions arrive sorted into Unread, Running and Read, and any of them can be grouped by topic. Search, an archived section, colour, a row menu that acts on the whole selection, and Apropos — a new session that already knows the ones you picked, one of them or a whole group — come along. Disclosure — a pane over all your local profiles: it reads their sessions, groups and settings through the app's own requests and, only when you click, creates, opens, deletes or archives sessions (deletes ask first) and files them into groups; Apropos starts a session carrying a hidden pointer message naming those you picked; the toolbar's + creates a real session so the empty chat can introduce the profile; no network calls or telemetry."
maintainer: SmoothMarx
tier: community
category: desktop
version: "0.1.15"
image: https://raw.githubusercontent.com/SmoothMarx/perso/033d0bc7a1e91c19c64dc10eed661aee5447ce02/assets/perso-pane.png
screenshots:
  - https://raw.githubusercontent.com/SmoothMarx/perso/033d0bc7a1e91c19c64dc10eed661aee5447ce02/assets/perso-pane.png
platforms: [linux, macos, windows]
capabilities:
  provides_tools: []
  provides_hooks: []
  provides_middleware: []
  requires_env: []
requires_hermes: ">=0.21.6"
```

## Why each field

- **`sha`** — the loader wants a full 40-character commit, and the pin IS the trust
  model: installs check out exactly that commit. Bumps are new PRs.
- **`requires_hermes: >=0.21.6`** — the pane registers on the `chat.empty` SDK area
  (the empty-chat hero), which first shipped in 0.21.6 (`45176219a2`). On an older
  app the import is `undefined` and the plugin would register into nothing.
- **`version: 0.1.15`** — 0.x on purpose. It is a fork being iterated nightly; the
  Personae line is the settled 1.x.
- **`capabilities:` all empty** — no tools, hooks, middleware or env vars; the pane
  only speaks the app's own request surface. This block must match reality at the
  pinned commit, or validation fails the entry.

## The description's three jobs

1. **Say what it is in one clause** — "one flat list of every profile's sessions,
   ordered by activity". That is the whole difference from Personae, and the
   Personae entry already explains the roster flavour.
2. **Say it is a fork**, so a reader does not think two plugins independently
   invented the same pane chrome.
3. **Disclose the writes**, in the same shape Personae's entry does: what it reads,
   what it writes, that deletes ask, that `+` creates a session. `+` creating a
   session is new behaviour and the easiest thing to be surprised by, so it is
   named rather than implied by "creates".

## Still needed

1. **`assets/perso-pane.png`** — a capture of the app window with the Perso tab
   active, list populated, ideally with a group box and a divider visible. Same
   framing as `personae-pane.png` (app window, 1916×1008, uncropped).
2. **Optional: `assets/perso-icon.png` / `perso-logo.png`** — the current files are
   Personae's. A fork with the parent's face is confusing in a catalog grid.
3. Rename the borrowed assets once the new ones exist; nothing in the plugin reads
   them (the pane inlines its own marks), so this is repo tidiness only.

## Sequence when the asset lands

1. Commit the capture to `SmoothMarx/perso`, re-run `tests/portability.sh` and the
   harness at that commit.
2. Fork-sync `SmoothMarx/hermes-agent` to upstream `main`.
3. Add `plugin-catalog/perso.yaml` on a branch, open the PR (Personae's re-pin went
   through as NousResearch/hermes-agent#154209, so the account is no longer
   blocked).
4. Reviewer looks at the pinned commit range; `hermes plugins validate` runs the
   desktop-surface and capability checks.
