---
title: "Marketplace from GitHub: the latest plugin in every session"
authors: [Peter Petrov]
created: 2026-09-10
last_updated: 2026-09-11
status: in progress
status_note: Repo half landed 2026-09-11 (aa477fd) — the version-pin question is answered, the README carries the one-command bootstrap and the recovery, and `/modus:init` plus the catalog now write committed settings. This repo's own committed `.claude/settings.json` followed in 1318a91. Pushed 2026-09-11 and shipped in `modus 1.4.0`; WSL is on the GitHub source. Two of five acceptance items pass. The Windows premise was dropped as unreal (it is dormant, and was never a separate source). What is left is two live questions: whether an interactive session start auto-updates at all — the machine is parked one version behind to answer it — and whether a cloud session installs the plugin from committed settings.
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

- **2026-09-11 — `modus 1.4.0` released**, on the owner's call, with two
  acceptance items here still unverified. Deliberate and recorded in
  `releases.md`: the remaining checks need a second machine and a cloud
  sandbox, and the second machine has nothing to pull until the release is
  out. Plugin version 1.3.6 → 1.4.0.

- **2026-09-11 — the cloud check ran, and answered RFC 0011's half of it only.**
  A cloud session on `modus` itself named both committed rules and quoted one
  verbatim, so the rules reach a sandbox. **The plugin half was not tested** —
  the session was not asked whether `/modus:init` was available to it, so
  whether a repo's committed `.claude/settings.json` actually installs the
  plugin at cloud-session start remains unknown. That is the open half of the
  third acceptance item, and the question below it.

- **2026-09-11 — the "both installs" premise was checked and dropped.** A Windows
  install exists but has been idle since 2026-08-25 (settings last touched
  2026-08-29, every session record under the superseded `topal` account path),
  while WSL is in daily use. **And the two were never separate sources:**
  `C:\Users\peter\Projects\modus` and `/mnt/c/Users/peter/Projects/modus` are
  one folder spelled two ways, so they could not diverge on content — only on
  which cached plugin version each had pulled. The motivation's "the two installs
  can drift apart" was therefore the weakest of its three arguments; the other
  two — a machine that is not this one, and a cloud sandbox with no folder at
  all — are untouched and remain the reason for this RFC. The first two
  acceptance items were rewritten from "both installs" to "every install in use",
  on the owner's call. Reviving Windows means the three-command switch in the
  README's *Switching a machine that already had the folder source*.

- **2026-09-11 — the delivery test ran, and a push does *not* reach a fresh
  session on its own.** Test payload: a real documentation fix plus
  `modus 1.4.0 → 1.4.1`, pushed (1371812), with the installed plugin sitting at
  1.3.6. Four observations, in order:
  1. `autoUpdate: true` written into the user-scope `extraKnownMarketplaces`
     entry **does** reach `known_marketplaces.json`, so the settings field is
     not ignored — it feeds the registry that drives the behaviour.
  2. A fresh non-interactive session (`claude -p`) started with that flag on
     refreshed **nothing**: same installed version, same marketplace commit,
     same `lastUpdated` timestamp.
  3. `claude plugin marketplace update modus` moved the marketplace clone to
     the new commit — but left the **installed plugin at 1.3.6**. Refreshing
     the catalog and updating a plugin are two separate steps, and the README's
     "the push is the whole delivery" elides that.
  4. A second fresh `-p` session, now with a current catalog showing 1.4.1,
     still did not update.
  **Untested and the one thing that would settle it:** whether an *interactive*
  session start does what `-p` does not. Deliberately left in that state — the
  machine is parked at 1.3.6 against a pushed 1.4.1, so the next interactive
  start is a free, decisive test. Updating by hand would have destroyed it.

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

- [ ] Every install in use takes the plugin from the GitHub source with auto-update on
- [ ] A push to modus reaches a fresh session with no manual step
- [ ] One repo's committed `.claude/settings.json` declares marketplace and plugins; a cloud session on it shows the plugin loaded (the session-start hook fired)
- [x] The recovery for a bad update is written in the README, five lines or fewer — aa477fd
- [x] `/modus:init` writes committed settings, not local — aa477fd (catalog entries and the README passage in the same commit)

## Unresolved questions

- **Does auto-update ever fire without a manual step?** Partly answered
  2026-09-11, and the answer so far is no. The settings field is honoured as
  far as the registry, but two fresh `-p` sessions refreshed nothing, and a
  manual marketplace refresh updates the catalog without touching the installed
  plugin. What remains is whether an interactive session start behaves
  differently; the machine is parked one version behind on purpose so the next
  one answers it. If interactive sessions also do nothing, the README's
  "the push is the whole delivery" is wrong and this RFC needs a step that
  runs `claude plugin update` — which is a smaller promise than the one made
  here, and worth saying out loud rather than quietly tolerating.
- **Does a repo's own marketplace entry override a user-scope one of the same
  name?** The README's "hold one repo back" recovery — pin
  `extraKnownMarketplaces.modus.source` to a `ref` — assumes a project-scope
  entry wins over the user-scope `modus` entry, which follows from settings
  precedence but was not exercised.
- **Will a cloud session install these plugins without a manual step?** Still
  open after the 2026-09-11 cloud run, which tested the rules and not the
  plugin. One question to a cloud session on this repo settles it: *are the
  modus slash commands available to you?* The docs
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
