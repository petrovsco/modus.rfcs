---
title: House rules travel with the repo
authors: [Peter Petrov]
created: 2026-09-10
last_updated: 2026-09-11
status: in progress
status_note: Mechanism landed in `modus` 2026-09-10 (2e70fb6); twelve repos plus the lumi workspace wrapper migrated 2026-09-11, one commit each, all pushed. `modus 1.4.0` released 2026-09-11. Five of six acceptance items pass; what is left is the cloud-session check, which needs a live sandbox rather than any work in a repository.
label: infra
release: modus 1.4.0
---

# RFC 0011: House rules travel with the repo

**Kickoff:** start a fresh session in the modus repo with this file as the brief.
**Origin:** 2026-09-10, from checking what a cloud session actually loads: nothing under `~/.claude/` carries into the sandbox, so every `@~/.claude/modus/rules/…` import resolves to nothing there.

## Summary

A repo's opted-in house rules live as committed files in
`.claude/rules/modus/`, which Claude Code loads automatically wherever the repo
is checked out — a cloud sandbox, a collaborator's machine, a machine without
the plugin. The `sync-rules` hook, which today publishes rules to
`~/.claude/modus/rules/`, refreshes those in-repo copies at session start. The
`@~/.claude/modus/rules/<rule>.md` import lines go away.

## Motivation

The import mechanism assumes the plugin is installed on the machine that reads
the repo. Two readers break that assumption:

- **Cloud sessions.** Per the Claude Code docs (checked 2026-09-10), user-scope
  settings and the `~/.claude/` tree do not carry into a cloud sandbox; what
  does is the repo itself — its `.claude/settings.json`, its `.claude/rules/`,
  and plugins declared in its settings. A cloud session on any repo using
  modus today runs with **no house rules at all**.
- **Outside readers of a public repo.** The catalog already warns that the
  import "resolves to nothing for a reader without modus installed". The rules
  that matter most for a public repo — `no-personal-context`,
  `rfc-convention` — are exactly the ones a stranger has to see.

`.claude/rules/*.md` is documented as auto-loaded, recursively, with optional
path scoping, and listed as carried into cloud sessions as part of the clone.

## Goals

- The same rule text everywhere the repo is.
- Opt-in stays per repo; edit once in modus, reaches every repo at its next
  local session.
- No rule loaded twice.
- A reader with nothing installed sees the rules.

## Non-Goals

- Hooks and commands in the cloud — RFC 0012.
- Changing the content of any rule.
- Auto-committing the refreshed copies; the session's ordinary commits carry
  them.

## Proposal

- **Opt-in = the file exists.** `/modus:init` installing a rule writes
  `<repo>/.claude/rules/modus/<slug>.md`; removing a rule deletes the file. No
  manifest, no import line.
- **`sync-rules.mjs` gains a second target.** For each `.md` already present
  in `<cwd>/.claude/rules/modus/`, overwrite it with the plugin's current
  version when they differ. Never add a file that is not there. A rule retired
  upstream is not deleted silently: its body is replaced by a two-line note
  saying so, and the human removes the file.
- **A first line in every copy:**
  `<!-- managed by modus — edit the rule in the modus repo, not here -->`.
- **`~/.claude/modus/rules/` keeps being published for one release**, for
  repos not yet migrated, then stops.
- **Migration**, one commit per repo, for every repo carrying import lines
  today: copy each imported rule into `.claude/rules/modus/`, delete the import
  lines, leave the `## House rules` section as one sentence pointing at the
  folder. Update the catalog's install steps for every `rule` entry and the
  `configs/rfcs/README.md` skeleton.
- **Ordering caveat, accepted.** The hook runs at session start; if the rules
  are read before it rewrites them, the first session after an upstream edit
  sees the previous text and the next one sees the new. Good enough — the
  request was "not too tight".

## Rationale

- *Keep imports, add copies* — loads every rule twice. Rejected.
- *Install the plugin in the cloud (0012) and rely on its hook to recreate
  `~/.claude/modus/rules/`* — brings the hooks, but a stranger still sees
  nothing, and an external-path import triggers an approval dialog the first
  time, which a non-interactive start may not answer. Complementary, not
  sufficient.
- *A cloud setup script that clones modus* — not documented, and it is
  per-environment where the rule is per-repo.
- *Committed copies refreshed by the hook* — the only form every reader gets.
  Chosen.

## Acceptance

- [x] `sync-rules.mjs` refreshes `<cwd>/.claude/rules/modus/*.md`, adds nothing on its own, and marks retired rules instead of deleting them  *(2e70fb6)*
- [x] `/modus:init` writes and removes rule files there; every `rule` entry's install steps in the catalog say so  *(2e70fb6)*
- [x] Every repo that imported rules is migrated: no `@~/.claude/modus/rules/` line remains anywhere  *(twelve repos plus the lumi workspace wrapper, one commit each, 2026-09-11)*
- [ ] A cloud session on one migrated public repo quotes a house rule when asked
- [x] A local session shows each rule once in its context, not twice  *(imports removed in the same commit that added the file, so no repo ever carries both)*
- [x] modus 1.4.0 released per the release rule  *(2026-09-11, on the owner's call; plugin version bumped and `releases.md` marked released)*

## Unresolved questions

None blocking. Two facts the migration turned up, recorded because the next
reader will hit them:

- **`lumi.rfcs` ignored `.claude/` wholesale**, so a committed rule copy could
  never have travelled with it. Fixed in that repo (`.claude/*` with
  `!.claude/rules/`). Any repo adopting a rule needs the same check — the
  ignore fails silently and looks exactly like success.
- **The `lumi-workspace` wrapper is not a git repo**, so its copies are
  untracked. They still load for a session started there, but "travels with the
  repo" does not apply to a folder that is not one. It is the last workspace
  wrapper left and a flattening candidate.
