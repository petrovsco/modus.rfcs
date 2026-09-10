---
title: "tekio — extract the specs into `tekio.rfcs`"
authors: [Peter Petrov]
created: 2026-09-10
last_updated: 2026-09-10
status: blocked
status_note: "Done on 2026-09-10 apart from the board: 71 RFCs and the five reference docs are in petrovsco/tekio.rfcs with their history, and the code repo is clean. The last box waits on RFC 0007."
label: infra
depends: [6, 7]
---

# RFC 0008: tekio — extract the specs into `tekio.rfcs`

**Kickoff:** start a fresh session with this file as the brief.
**Origin:** 2026-09-10, from "as tekio has worked most with the roadmap, the extraction there will be heavier". Parent decision: task 4.

## Goals

`petrovsco/tekio.rfcs` holds tekio's plan and its standing reference; the code
repo holds code. Roughly 72 briefs (18 active, 54 retired) plus `doctrine.md`,
`design-system.md`, `code-review.md`, `grounding-inventory.md` and `grounding/`
move — with their history.

## Approach

1. **Create the repo** — `gh repo create petrovsco/tekio.rfcs --private`,
   cloned to `Projects/tekio.rfcs`, beside the code repo where the board looks.
2. **Extract with history**, never a copy:
   `git filter-repo --path docs/roadmap/ --path docs/doctrine.md --path docs/design-system.md --path docs/code-review.md --path docs/grounding-inventory.md --path docs/grounding/`
   on a throwaway clone, then push that as `tekio.rfcs`. A status line's history
   is how its story is read back; a fresh-copy repo throws that away.
3. **Rearrange** — `docs/roadmap/*` → `rfcs/*`, `docs/roadmap/done/*` →
   `rfcs/done/*`, `releases.md` stays furniture in `rfcs/`, and the reference
   docs land at the repo root (`doctrine.md`, `design-system.md`, `grounding/`).
4. **Convert every brief** with a script that reads and writes whole files:
   `NNN-` → `NNNN-` (the number is the ID; only the padding changes), the four
   bold header lines → frontmatter, `**Status:** planned — <sentence>` →
   `status: planned` + `status_note: <sentence>`. `created` comes from each
   file's first commit, `last_updated` from its last. Do not rewrite what a
   brief argues — converting a header is not a licence to edit the content.
   `sed -i` under Git Bash strips `\r`; use a real script.
5. **Fill the gaps that the new shape exposes.** The old briefs have no
   `## Non-Goals` and often no `## Acceptance`. Add them where the brief already
   answers the question; leave a `## Unresolved questions` entry where it does
   not, rather than inventing scope.
6. **Delete `docs/roadmap/` and the moved reference docs from the code repo**
   in a commit of its own, and repoint every reference to them — links in other
   docs, in `CLAUDE.md`, and in source comments.
7. **Swap the CLAUDE.md import** from `pending-work-in-roadmap` to
   `rfc-convention`, in both repos.

## Non-Goals

- Renumbering. tekio's IDs are in the user's head and in commit messages; 71
  stays 71, only padded.
- Re-litigating retired briefs. `done/` moves as-is.
- Splitting `grounding/` into RFCs. It is reference; if it carries pending
  work, that becomes an RFC per the rule, but not in this pass.

## Acceptance

- [x] `tekio.rfcs` exists, cloned beside `tekio`, with the full history of every moved file — 195 commits
- [x] 71 briefs (not 72 — the count included `README.md`) are four-digit, frontmatter-headed, and keep their original numbers
- [x] `rfcs/releases.md` and the reference docs are in place
- [ ] `/roadmap` lists tekio from the new repo — waits on RFC 0007
- [x] The code repo has no `docs/roadmap/` and no dangling link to it — 264 links repaired, `check:docs` clean
- [x] Both repos import `rfc-convention` — the code repo's stale `pending-work-in-roadmap` import was swapped
- [ ] The board shows tekio from `tekio.rfcs` (needs RFC 0007 for the frontmatter parser)
