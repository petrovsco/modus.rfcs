---
title: "Marketplace from GitHub: the latest plugin in every session"
authors: [Peter Petrov]
created: 2026-09-10
last_updated: 2026-09-10
status: planned
status_note: Kickoff-ready. One open check at kickoff — whether an enabled plugin can be pinned to a version — decides the shape of the safeguard, not whether to proceed.
label: infra
release: modus 1.4.0
---

# RFC 0012: Marketplace from GitHub: the latest plugin in every session

**Kickoff:** start a fresh session in the modus repo with this file as the brief.
**Origin:** 2026-09-10 — "every next session should have the latest modus plugin available, with some safeguards — not too tight, to save gas".

## Summary

The modus marketplace is registered from its GitHub repository instead of a
local directory, with auto-update on, so every session — either local install,
a second machine, a cloud sandbox — starts with the latest pushed plugin. Repos
declare the marketplace and their enabled plugins in a committed
`.claude/settings.json`, so cloud sessions install them at start. The
safeguard is light: the existing release rule, plus a written five-line
recovery when an update misbehaves.

## Motivation

The marketplace source is a directory path on one machine. Consequences: the
Windows and WSL installs can drift apart; a second machine needs a clone at the
same path before anything works; and a cloud session has no plugin at all,
because user-scope settings do not carry into the sandbox (docs, 2026-09-10).
The docs also say plugins declared in a repo's `.claude/settings.json` are
installed at cloud session start from the declared marketplace. The repo is
public, so the GitHub source is available.

## Goals

- One source of the plugin for every session, everywhere.
- A change reaches sessions by being pushed — no manual step.
- Cloud sessions get the hooks and commands, not only the rules.

## Non-Goals

- A staging channel or version matrix. The request was explicitly lean.
- Changing how the plugin is developed — the local clone stays the workbench.

## Proposal

- **User scope, both installs:** the `modus` marketplace source becomes the
  GitHub repo, `autoUpdate` on; the directory source is dropped. A change
  reaches sessions after push plus the auto-update at the next start (or an
  explicit plugin update).
- **Each repo commits `.claude/settings.json`** with the same marketplace block
  and `enabledPlugins` for what it uses (`modus`, `session-flow`,
  `visual-iteration` where relevant). `/modus:init` writes this instead of
  `settings.local.json`.
- **Safeguard.** Minor and major releases are already declared and shipped on
  purpose; patch bumps are automatic. If a plugin can be pinned per repo by
  version in `enabledPlugins`, that is the recovery; if not, the recovery is
  "revert the push, restart the session". Either way it is written in the
  README in five lines or fewer.
- **Bootstrap for a new machine** reduces to one command: add the marketplace
  from GitHub. The README says so.

## Rationale

Keeping the directory source and adding a GitHub one for the cloud would mean
two sources for one plugin, drifting on the first day. One source, the pushed
one, is the lean option.

## Acceptance

- [ ] Both local installs use the GitHub source with auto-update; the plugin list shows the same versions on WSL and Windows
- [ ] A push to modus reaches a fresh session on the other install with no manual step
- [ ] One repo's committed `.claude/settings.json` declares marketplace and plugins; a cloud session on it shows the plugin loaded (the session-start hook fired)
- [ ] The recovery for a bad update is written in the README, five lines or fewer
- [ ] `/modus:init` writes committed settings, not local

## Unresolved questions

None — the version-pin check is the first step of the work, not a blocker.
