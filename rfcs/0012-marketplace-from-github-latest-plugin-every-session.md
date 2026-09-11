---
title: "Marketplace from GitHub: the latest plugin in every session"
authors: [Peter Petrov]
created: 2026-09-10
last_updated: 2026-09-11
status: in progress
status_note: Repo half landed 2026-09-11 (aa477fd) — the version-pin question is answered, the README carries the one-command bootstrap and the recovery, and `/modus:init` plus the catalog now write committed settings. This repo's own committed `.claude/settings.json` followed in 1318a91. Two of five acceptance items pass; Pushed 2026-09-11, and WSL is on the GitHub source; the rest waits on the Windows install, a push reaching it unaided, and a cloud session proving the declaration works.
label: infra
release: modus 1.4.0
---

# RFC 0012: Marketplace from GitHub: the latest plugin in every session

**Kickoff:** start a fresh session in the modus repo with this file as the brief.
**Origin:** 2026-09-10 — "every next session should have the latest modus plugin available, with some safeguards — not too tight, to save gas".

## Progress log

- **2026-09-11 — the version-pin check, answered from the plugin documentation.**
  A repo cannot pin an enabled plugin to a version. `enabledPlugins` takes
  `true`/`false` and nothing else, and `claude plugin install`, `update` and
  `marketplace add` expose no version argument. Version pins are
  producer-side: a plugin entry's `ref`/`sha` inside `marketplace.json`, or the
  manifest's `version`. The one consumer-side hold-back is a **marketplace**
  source's `ref` — a branch or tag, never a sha — which freezes the whole
  marketplace for one repo. So the README's recovery is revert-the-push plus
  restart, with the `ref` pin named as the way to hold a single repo back.
- **2026-09-11 — the repo half landed (aa477fd).** README: bootstrap is one
  command against `petrovsco/modus` (the clone URL still named the previous
  GitHub owner; corrected), auto-update named as what actually delivers a push,
  and the five-line recovery section. `/modus:init`: step 0 no longer needs a
  clone — plugin root, else the catalog straight from the repository of record
  over https, else ask — and step 3 writes the committed `.claude/settings.json`
  carrying `extraKnownMarketplaces` plus `enabledPlugins`. Both plugin entries
  in `catalog.json` say the same. `modus` 1.3.5 → 1.3.6.
- **2026-09-11 — this repo's own committed settings (1318a91).**
  `extraKnownMarketplaces.modus.source` pointing at `petrovsco/modus`, plus
  `enabledPlugins`. `autoUpdate` deliberately omitted while the question below
  is open. Written outside the working session: the one that did the rest was
  refused that path by a settings-file guard.
- **2026-09-11 — left for the outside world.** Both user-scope installs, the
  push, and the cloud check. Nothing is pushed in either repository.

- **2026-09-11 — one of the two installs is on the GitHub source.** WSL swapped
  over and verified: the marketplace resolves to `petrovsco/modus`, cloned into
  the plugin cache rather than read from a working tree, and `modus@modus` 1.3.6
  is enabled at user scope. The swap is not clean, and the README now says so:
  `/plugin marketplace remove` **also deletes the plugin from
  `enabledPlugins`**, and `marketplace add` rewrites the settings entry
  **without** `autoUpdate`. Both were restored by hand. The Windows install is
  still on the folder source, so the first acceptance item stays open.

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
- [x] The recovery for a bad update is written in the README, five lines or fewer — aa477fd
- [x] `/modus:init` writes committed settings, not local — aa477fd (catalog entries and the README passage in the same commit)

## Unresolved questions

- **Is `"autoUpdate": true` honoured on an `extraKnownMarketplaces` entry
  outside managed settings?** The docs describe that field for administrators
  in managed settings only, and say a third-party marketplace has auto-update
  off by default. So the committed settings file omits it and the README tells
  each machine to toggle auto-update in `/plugin`. Still open, with one
  observation from the 2026-09-11 swap: `claude plugin marketplace add` rewrote
  the user-scope entry and dropped an `autoUpdate` key that was already there,
  which says the CLI does not maintain the field — not that the runtime ignores
  it. What settles it is a push landing in a session with no manual update.
- **Does a repo's own marketplace entry override a user-scope one of the same
  name?** The README's "hold one repo back" recovery — pin
  `extraKnownMarketplaces.modus.source` to a `ref` — assumes a project-scope
  entry wins over the user-scope `modus` entry, which follows from settings
  precedence but was not exercised.
- **Will a cloud session install these plugins without a manual step?** The docs
  say a plugin that only a project's `.claude/settings.json` enables, and that
  comes from an *external* source, does not load until someone runs
  `claude plugin install`. The modus entries use marketplace-relative sources
  (`./plugins/modus`), which should not count as external — the third
  acceptance item is what proves it.
- **Denied, and settled another way.** Listing `~/.claude/plugins/` to see the
  install layout was refused; the documentation answered it instead (a
  marketplace install copies only the plugin directory into
  `~/.claude/plugins/cache/`), which is why `/modus:init` falls back to reading
  the catalog over https rather than walking up from the plugin root.
