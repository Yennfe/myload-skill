---
name: myload
description: >
  Use when the user says things like "give me a status report", "I don't know
  what's still in flight", "I'm afraid to close terminals", "the review backlog
  is crushing me", "which PRs are stuck", "which terminals can I close", "which
  worktrees should I clean up", "too many open PRs", "盘点一下", "我能关哪些终端";
  or has multiple open PRs / issues / local git worktrees on a git + GitHub
  project and feels overwhelmed; or types /myload or /myload <freeform direction>.
---

# /myload —— Collaborator Workload Triage

## What you do

When the user is buried under PRs, issues, and worktrees, read `git` and
`gh` yourself, identify **where their workload actually concentrates**, and
report in plain language. This is workload grouping, not checklist sprawl.

Analogy: you're the user's assistant when they return from a trip — one
sentence: *"3 ready for your signature, 2 waiting on others, 1 already
stale, toss it."*

## Triggers and applicability

| Input | Interpretation |
|---|---|
| `/myload` | Fully autonomous, scan everything |
| `/myload <freeform>` | Priority hint (NOT a hard filter) |

Requirements: current dir inside a git repo + `gh` authenticated + user has
collaborator access. **Fork handling**: check with
`gh repo view --json parent`; if `origin` is a fork, pass
`--repo <parent.owner>/<parent.name>` to PR / issue queries, not origin, or
you'll miss everything.

## Workflow

1. **Get the map**: `git remote -v` / `gh repo view --json parent` /
   `gh pr list --author @me --state open` /
   `gh issue list --author @me --state open` / `git worktree list`
2. **Pick 2–5 to dig into**: looks stuck (N days idle, no one waiting on
   anyone) / PR merged but worktree lingering / untracked files unrelated to
   branch topic / matches user's hint
3. **Dig in**: `gh pr view <num> --json reviews,comments,statusCheckRollup`;
   `git -C <p>` with `status` / `branch -r --contains HEAD` / `stash list`
4. **Group into the 6 categories below** (skip empty), output per format

## Report Format

Each category three parts: **plain-language description** + analogy +
path. End with a fallback slot, last line **"Which one first?"**.

## The 6 Categories

**1. Ball in their court** — letter is in the mail, no point waiting by the mailbox
See: PR open + not draft + mergeable + no changes-requested + you weren't the last commenter; or issue awaiting owner
Do: corresponding worktree can be closed now. Nudges go in a separate list.

**2. Ball in your court** — they wrote back, you haven't opened the letter
See: unresolved review comments / failing CI / merge conflicts (`CONFLICTING`) / owner @-mentioned you, no answer
Do: what you actually need to do today. Sort by urgency, one at a time.

**3. Zombies** — old to-do list in a drawer, you forgot why
See: N days idle (7–14), neither category 1 nor 2
Do: restart (revive with a comment) / archive (close with reason) / convert (downgrade to draft, move to discussion).

**4. Ready to wrap up** — dinner cooked, just turn off the heat and plate it
See: PR mergeable + CI green + approved + no unresolved; or issue actually resolved but never closed
Do: one-click actions.

**5. Ready to bury** — package delivered two days ago, slip still on your desk
See: local worktree exists, but its remote branch is gone / its PR is already merged or closed. Local-vs-remote correspondence
Do: `git worktree remove <path>` (for paths outside the repo, `cd` into parent first).

**6. Scattered locally** — N drawers, most stuffed with half-finished things
See: working tree not clean / unpushed commits / local branch with no upstream / stashes you can't remember
Do: each one keep (move out or `.gitignore`) / commit / delete, 30 seconds each.

**Fallback** — for odd observations that don't fit (typical: "none of your
N terminals are actually doing work" systems-level observations, cross-
project patterns, historical firefighting in stash messages). Force-fitting
loses information.

## Anti-patterns (strict during report writing)

| Anti-pattern | Do this instead |
|---|---|
| Floating at symptom level ("3 PRs are stuck") | Lift to root ("your PRs routinely wait days for review") |
| Piling on PR / issue numbers / file names as primary structure | Object numbers are supporting; category framing is primary |
| "Not X but Y" comparison construct | Just say what it is |
| Case-by-case enumeration as main structure | Lift to a category; specifics only in fallback |
| Tables enumerating every field of every PR | Tables for side-by-side comparison only |
| Reports with "mergeable" / "reviewDecision" / "headRef" terminology | Translate: "can be merged / review state / branch name" |
| Auto confidence / priority numbers | "act on this now / not urgent / I'm unsure" |
| Refusing to run when no worktrees exist | Run normally; categories 5 & 6 may be empty |
| Assuming origin = upstream | Check `gh repo view --json parent` first; fork projects need parent lookup |

## Boundaries (what this skill does NOT do)

**Read-only** — doesn't modify files / commit / push / stash / `worktree remove`;
doesn't close PRs or issues; doesn't archive / notify; doesn't do collaborator
comment summary (that's `/myload comments`, planned); doesn't watch in real
time (that's `/myload --watch`); doesn't dump GitHub API field names / doesn't
bucket mechanically.

## Self-check before sending

1. Any "mergeable" / "reviewDecision" / "headRef" terminology? → translate
2. "Not X but Y" constructs? → rewrite
3. Case-by-case enumeration as main structure? → lift to category
4. Is each item really *a category*, or just *one specific object*? Latter → fallback or re-categorize
5. Does the analogy need technical context to land? → swap
6. Is "path" at the *direction* level, no file paths or line numbers?
7. Did you end with "Which one first?"
8. Any empty category force-included? → drop it
