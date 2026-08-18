---
name: handoff
description: Use when shipit artifacts must be delivered — branch, commit, push, pull request, review-thread replies, tracker comment, status transition. Runs after `plan`, `implement`, or `pr-fix`. Do not use to write plans or product code.
metadata:
  writes_product_code: false
---

# shipit handoff

Owns every external side effect, so `plan`, `implement`, and `pr-fix` own none.
They produce artifacts; this skill delivers them.

Three modes: `plan` (after a plan is written), `implementation` (after a change is
verified), `review` (after PR feedback is resolved). Ask which one when it is not
given and the branch does not make it obvious.

`implement` and `pr-fix` invoke this skill themselves once validation is green,
passing their report. Being invoked that way changes nothing: run the same
preflight, and trust the report for **content** only — never for whether the side
effects already happened. They have not. This skill is the only thing that performs
them.

## Context budget

- Required: `.sdd/config.json` (for `tracker` and `repo`), and the report or plan
  being delivered.
- Required: `references/tracker-adapters.md` — only the section for the configured
  adapter.
- Optional: `references/worktree-lifecycle.md` when the branch lives in a worktree.
- Never read the product diff to decide content. The report is the manifest.

## Hard rules

- Never write product code. Never edit a plan. Delivery only.
- **Stage by explicit path.** Never `git add -A`, never `git add .`.
- **`sdd_tracking: gitignored` excludes every `.sdd/*` path from staging**, in every
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
- Never fake success. Tracker or host unavailable → report `blocked` or `partial`
  with the exact failing call. Never a summary that implies it worked.
- Never move a tracker issue backwards. See the adapter reference for which states
  are terminal.
- Never merge. Never push to the default branch. Never force-push.
- Never remove a worktree. That is `/shipit:status --prune`.
- **One line per side effect.** This skill reports what it did, never what the
  change was — the report it was handed already said that, and a PR body or tracker
  comment is not improved by a second telling. Copy sections from the report
  verbatim or leave them out; never expand them.

## Preflight — all modes

1. `git status`, current branch, remote. `gh auth status` when the host is GitHub.
2. Read `sdd_tracking` from `.sdd/config.json` (absent → `committed`). Gitignored →
   no path under `.sdd/` is ever staged, in any mode; see Hard rules.
3. Confirm which checkout you are in. The main checkout is the default and is fine.
   Only when the plan names a worktree must you be in it — see
   `references/worktree-lifecycle.md`.
4. Tracker issue id present in the branch or slug? Read its **current** status
   before deciding any transition.
5. Confirm the artifacts to deliver exist on disk.
6. Any check fails → stop and report what is missing. Produce **no** partial side
   effects. Half a handoff is worse than none.

## Mode: plan

- **Branch** — from the tracker adapter when it supplies one; otherwise the repo's
  branch convention from `.sdd/conventions.md`; otherwise
  `<user>/<issue-id>-<slug>`. Never invent a prefix. Usually `plan` already created
  it with the worktree; verify rather than re-create.
- **Commit** — the plan file only. `sdd_tracking: gitignored` → nothing to commit
  (the plan file is untracked by design): report `Commit: skipped (.sdd is
  local-only per config)`, never an empty commit.
- **Push** — normally, setting upstream when missing.
- **PR** — Draft. Title `<ISSUE-ID> Plan: <short title>`. Body states this is
  planning only and execution belongs to `/shipit:implement`. `sdd_tracking:
  gitignored` → the body also carries the plan's content verbatim (Goal,
  Acceptance, Files, Notes) — with no commit to show it, the PR would otherwise
  arrive empty of everything a reviewer needs.
- **Tracker** — the plan comment from the adapter reference.
- **Status** — earliest-state to next-state only (e.g. `Backlog` → `Todo`). Leave
  started, completed, cancelled, and duplicate states untouched.

## Mode: implementation

- **Commit** — every path in the report's `Files Changed`.
- **Push** — normally.
- **PR** — replace the plan PR's body on this branch with the report's
  `PR Preparation` body verbatim, then mark it ready for review. Verbatim means no
  re-wording, no added preamble, and no re-adding a section the template dropped.
  Create a new PR only when none exists.
- **Tracker** — the report's summary, the validation result, the PR link, and the
  QA steps **copied verbatim**. Nothing else: no file list, no diff narration. The
  QA steps were written for a non-developer, so they are the one part never
  shortened.
- **Status** — move to the review state. Leave QA-ready, completed, cancelled, and
  duplicate states untouched.

## Mode: review

Input: the `pr-fix` report — item table, `Files Changed`, validation results,
pushback justifications.

- **Commit** — every path in the report's `Files Changed`. Same manifest rule.
- **Push** — normally, to the PR's existing branch.
- **PR** — never create one; a missing PR is a preflight failure. Then, per item in
  the report's table:
  - `fix` → reply on the thread with one line (what changed, in which file), then
    resolve the thread. On GitHub: `gh api graphql` with the `resolveReviewThread`
    mutation, thread id from `pullRequest.reviewThreads`.
  - `pushback` → reply with the justification **verbatim** from the report. Do
    **not** resolve. The decision is human.
  - `question` → reply with the question verbatim. Do **not** resolve.
- **Tracker** — fix summary per item plus validation results.
- **Status** — no transition. The issue is already in review or later.

## Report

One line per side effect, each marked `completed`, `partial`, or `blocked`, with the
exact reason for anything short of completed. No preamble, no summary paragraph, no
recap of the change — a `completed` line needs no explanation:

- Branch.
- Commit hash, and the files staged — plus any `.sdd/*` path excluded because
  tracking is gitignored.
- Push.
- PR url.
- Review thread replies and resolutions — `review` mode, one line per thread.
- Tracker comment.
- Tracker status transition.
- Worktree and preflight issues encountered.

This report is appended verbatim to the caller's report. Write it so it survives
being pasted without edits.
