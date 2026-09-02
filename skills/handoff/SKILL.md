---
name: handoff
description: Use when shipit artifacts must be delivered to git and the PR — branch, commit, push, and the pull request body. Runs after `plan`, `implement`, or `pr-fix`. It writes nothing to your tracker and posts no review-thread replies. Do not use to write plans, tickets, or product code.
metadata:
  writes_product_code: false
---

# shipit handoff

Owns every external side effect, so `plan`, `implement`, and `pr-fix` own none.
They produce artifacts; this skill delivers them — to git and to the pull request,
and nowhere else.

**The delivery surface is exactly four things: branch, commit, push, PR body.**
No tracker comment, no tracker status transition, no issue creation, no
review-thread reply, no marking a PR ready for review. Those are human moves. A
mode that has nothing but those to do has nothing to do.

Three modes: `plan` (after a plan is written), `implementation` (after a change is
verified), `review` (after PR feedback is resolved). Ask which one when it is not
given and the branch does not make it obvious.

`implement` invokes this skill itself once validation is green, passing its report.
`pr-fix` does **not** — the user runs `/shipit:handoff` by hand after reviewing the
fixes. Being invoked either way changes nothing: run the same preflight, and trust
the report for **content** only — never for whether the side effects
already happened. They have not. This skill is the only thing that performs them.

## Context budget

- Required: `.sdd/config.json` (for `tracker` and `repo`), and the report or plan
  being delivered.
- Required: `references/tracker-adapters.md` — only the section for the configured
  adapter. It is consulted for **branch naming and PR issue-linking only**.
- Optional: `references/worktree-lifecycle.md` when the branch lives in a worktree.
- Never read the product diff to decide content. The report is the manifest.

## Hard rules

- Never write product code. Never edit a plan or a task draft. Delivery only.
- **Never write to the tracker.** No comment, no status transition, no issue
  created, no label. The adapter reference is read for a branch name and an issue
  reference in the PR body; nothing else about the tracker is touched. Report the
  tracker line as `n/a (handoff does not write to the tracker)`.
- **Never reply to or resolve a review thread.** `pr-fix` hands you replies and
  pushback justifications as text in its report — they stay in the report for the
  user to post. Copy them nowhere.
- **Never mark a PR ready for review.** A PR this skill creates stays a draft; one
  that is already ready stays ready. The transition out of draft is the user's.
- **Prose language.** Human-facing output follows `language` in `.sdd/config.json` (absent → `en`). Code, identifiers, commit subjects and branch names stay English.
- **Stage by explicit path.** Never `git add -A`, never `git add .`.
- **`sdd_tracking: local` excludes every `.sdd/*` path from staging**, in every
  mode, even when the report's `Files Changed` lists one. Drop it from the manifest
  and note it in the report as informational — never as an error or a stop
  condition.
- `plan` mode stages the plan file only. `implementation` and `review` modes use the
  report's `Files Changed` as the manifest: every staged path must appear there
  **and** in the current diff. A listed path absent from the diff, or an intended
  path in the diff missing from the manifest, stops the handoff.
- A dirty worktree with unrelated user changes blocks the commit unless the shipit
  artifacts isolate cleanly. Stop and report. Never stash, reset, or checkout over
  user work.
- Never fake success. Host unavailable → report `blocked` or `partial` with the
  exact failing call. Never a summary that implies it worked.
- Never merge. Never push to the default branch. Never force-push.
- Never remove a worktree. That is `/shipit:status --prune`.
- **One line per side effect.** This skill reports what it did, never what the
  change was — the report it was handed already said that, and a PR body is not
  improved by a second telling. Copy sections from the report verbatim or leave
  them out; never expand them.

## Preflight — all modes

1. `git status`, current branch, remote. `gh auth status` when the host is GitHub.
2. Read `sdd_tracking` from `.sdd/config.json` (absent → `committed`). Local →
   no path under `.sdd/` is ever staged, in any mode; see Hard rules.
3. Confirm which checkout you are in. The main checkout is the default and is fine.
   Only when the plan names a worktree must you be in it — see
   `references/worktree-lifecycle.md`.
4. Confirm the artifacts to deliver exist on disk.
5. Any check fails → stop and report what is missing. Produce **no** partial side
   effects. Half a handoff is worse than none.

## Mode: plan

- **Branch** — from the tracker adapter when it supplies one; otherwise the repo's
  branch convention from `.sdd/conventions.md`; otherwise
  `<user>/<issue-id>-<slug>`. Never invent a prefix. Usually `plan` already created
  it with the worktree; verify rather than re-create.
- **Commit** — the plan file only. `sdd_tracking: local` → nothing to commit
  (the plan file is untracked by design): report `Commit: skipped (.sdd is
  local-only per config)`, never an empty commit.
- **Push** — normally, setting upstream when missing.
- **PR** — Draft. Title `<ISSUE-ID> Plan: <short title>`. Body states this is
  planning only and execution belongs to `/shipit:implement`. `sdd_tracking:
  local` → the body also carries the plan's content verbatim (Goal,
  Acceptance, Files, Notes) — with no commit to show it, the PR would otherwise
  arrive empty of everything a reviewer needs.

## Mode: implementation

- **Commit** — every path in the report's `Files Changed`.
- **Push** — normally.
- **PR** — replace the plan PR's body on this branch with the report's
  `PR Preparation` body verbatim. Verbatim means no re-wording, no added preamble,
  and no re-adding a section the template dropped. The PR's draft state is left
  exactly as it is. Create a new PR only when none exists, as a draft.
- **QA steps** — they stay in the report. They used to be copied to the ticket;
  now the user posts them wherever they belong. Do not append them to the PR body
  unless the report's `PR Preparation` section already contains them.

## Mode: review

Input: the `pr-fix` report — item table, `Files Changed`, validation results,
pushback justifications.

- **Commit** — every path in the report's `Files Changed`. Same manifest rule. An
  empty `Files Changed` is valid here: a run of `resolved`, `pushback` or `question`
  items touches no code. Then there is nothing to deliver — say so and stop.
- **Push** — normally, to the PR's existing branch.
- **PR** — never create one; a missing PR is a preflight failure. Update the body
  only when the report supplies a new one. Thread replies, resolutions, and the
  draft state are untouched.

## Report

One line per side effect, each marked `completed`, `partial`, or `blocked`, with the
exact reason for anything short of completed. No preamble, no summary paragraph, no
recap of the change — a `completed` line needs no explanation:

- Branch.
- Commit hash, and the files staged — plus any `.sdd/*` path excluded because
  tracking is local.
- Push.
- PR url, and whether the body was created or replaced.
- Tracker — always `n/a (handoff does not write to the tracker)`.
- Worktree and preflight issues encountered.

Then, when the input report carried them, one line naming what the user still has
to do by hand: post the QA steps, reply to and resolve the review threads, move the
ticket, mark the PR ready. Name them; never do them.

This report is appended verbatim to the caller's report. Write it so it survives
being pasted without edits.
