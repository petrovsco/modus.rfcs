---
title: What actually reaches a session
authors: [Peter Petrov]
created: 2026-09-11
last_updated: 2026-09-11
status: planned
status_note: Split out of RFC 0012, which delivered one source of the plugin but left two delivery questions unanswered — whether an interactive session start auto-updates at all, and why a cloud session installs nothing from a committed declaration. One half is a cheap local test; the other may resolve upstream without any work here.
label: infra
depends: [12]
---

# RFC 0013: What actually reaches a session

**Kickoff:** start a fresh session in the modus repo with this file as the brief.
**Origin:** 2026-09-11 — RFC 0012 moved the marketplace to GitHub on the premise
that "pushing is the delivery". Two tests that afternoon showed the premise is
at best unproven and in the cloud simply false.

## Summary

RFC 0012 made the *declaration* correct everywhere — one marketplace source on
GitHub, every repo naming its own plugins in committed settings. This RFC is
about the step after the declaration: whether a session actually ends up
running the plugin that the declaration names. Two environments, two different
answers, neither of them the one the README assumed.

## Motivation

A source of truth nothing reads is decoration. Two findings, both from
2026-09-11, both recorded in RFC 0012's progress log:

- **Locally, a push did not reach a fresh session.** With `autoUpdate: true`
  reaching the marketplace registry, two fresh non-interactive sessions
  refreshed nothing at all — same installed version, same marketplace commit,
  same timestamp. A manual `claude plugin marketplace update` then moved the
  catalog clone forward while leaving the **installed plugin** where it was.
  Refreshing a catalog and updating a plugin are two operations, and only the
  first was ever observed happening on its own.
- **In a cloud sandbox, nothing installed at all.** A session on a repo whose
  committed `.claude/settings.json` declares the marketplace and
  `enabledPlugins` reported an empty plugin list, an empty
  `plugins/synced/<id>/`, no modus skills, and no SessionStart hook. It could
  read the declaration — it quoted it back — and still had nothing.

The cost of leaving this is a false sense of reach. Every rule edit, every
command fix, every hook change is written believing it lands somewhere; if
delivery needs a manual step nobody remembers, the whole marketplace quietly
serves an old version, and the failure is invisible because everything still
works — just not the new version.

## Goals

- Know, with evidence, what makes a pushed change reach a local session.
- Have the README's delivery promise match that, whatever it turns out to be.
- Know whether the cloud gap is ours to close or upstream's, and stop guessing
  either way.

## Non-Goals

- Building a substitute plugin-loader for cloud sessions. If plugins do not
  load there, the answer is to depend on committed files instead — which is
  what RFC 0011 already does for rules, and what makes a cloud session usable
  today.
- Changing anything about how the marketplace is sourced. RFC 0012 settled
  that and it is not in question here.
- Chasing a fix inside Claude Code. This RFC observes and documents; an
  upstream change is watched for, not built.

## Proposal

Three steps, smallest first.

1. **The parked local test.** As of 2026-09-11 this machine sits at modus
   1.3.6 while `petrovsco/modus` is at 1.4.2, deliberately, so that the next
   *interactive* session start is a clean experiment. `claude plugin list`
   before touching anything: a version above 1.3.6 means auto-update fires on
   interactive starts and the README is right; 1.3.6 means it does not and the
   README is wrong.
2. **Make the README say the true thing.** If delivery needs
   `claude plugin update <plugin>@modus`, that step joins the "Changing
   things" procedure as an explicit line rather than living in the gap between
   "commit, push" and reality. If auto-update does fire interactively, say
   *interactively* — a non-interactive session demonstrably does not.
3. **Pin down the cloud gap to one sentence, and retest on a schedule.**
   Record what a cloud sandbox does and does not do with a committed
   declaration, and check it again after any Claude Code release that mentions
   plugins or sandboxes. Until it changes, the standing guidance is the one
   RFC 0011 already proves: **anything a cloud session must have, commit as a
   file** — rules in `.claude/rules/`, commands as repo skills in
   `.claude/skills/`. Plugins are a convenience for local machines.

## Rationale

*Keep this inside RFC 0012* — rejected. 0012 achieved something real and
self-contained, and holding it open against an upstream behaviour nobody here
controls would have made "in progress" mean "waiting indefinitely", which is
exactly the state the status keywords exist to distinguish.

*Fold it into the cloud-session work instead* — rejected for scope: the local
delivery half has nothing to do with cloud sessions, and it is the half that
affects every rule edit made from this machine.

*Treat the cloud gap as a bug to fix here* — rejected on evidence. The
declaration is correct and readable in the sandbox; what did not happen is the
install, which is not code this repo owns.

## Acceptance

- [ ] The local delivery question is answered by observation: `claude plugin list` at a fresh interactive session start, against a version known to be newer upstream
- [ ] The README's "Changing things" procedure states the real delivery path, including a manual update step if one is needed
- [ ] The cloud behaviour is recorded in one paragraph, with the date it was last checked
- [ ] The standing guidance — commit what a cloud session must have — is written where a reader of this repo will meet it

## Unresolved questions

None. The first acceptance item is the first step of the work, not a blocker:
it needs an interactive session start, which happens by itself.
