---
name: myload
description: >
  Use when a collaborator on a git + GitHub project has multiple open PRs /
  issues plus multiple local git worktrees, and feels overwhelmed: "I don't
  know what's still in flight", "I'm afraid to close terminals", "the review
  backlog is crushing me", "I don't know which worktrees to clean up"; or
  says things like "give me a status report", "which terminals can I close",
  "which PRs are stuck", "should I clean up worktrees", "盘点一下", "我能关
  哪些终端". Also applies when the user types /myload or /myload <freeform
  direction>.
---

# /myload —— Collaborator Workload Triage

## What you do

When the user is buried under PRs, issues, and worktrees, read `git` and
`gh` yourself, identify "where their workload actually concentrates", and
report in plain language. Let them pick what to handle first.

This is **workload grouping**, not listing every PR. Drifting into
checklist sprawl means you've lost the mission.

Analogy: you're the user's assistant when they return from a trip — one
sentence: *"3 ready for your signature, 2 waiting on others, 1 already
stale, toss it."* Counting individual items is someone else's job.

## Triggers and applicability

Any git + GitHub collaboration project. Requirements: current working
directory inside a git repo + `gh` authenticated + user has collaborator
access.

| Input | Interpretation |
|---|---|
| `/myload` | Fully autonomous: scan everything |
| `/myload <freeform>` | Prioritize topics matching the hint (priority hint, NOT a hard filter) |

**Fork handling**: check with `gh repo view --json parent`. If origin is a
fork, the user's PRs usually target the upstream — pass
`--repo <parent.owner>/<parent.name>` to PR / issue queries, not origin.

## Workflow

1. **Get the map**: `git remote -v`, `gh repo view --json parent`,
   `gh pr list --author @me --state open`,
   `gh issue list --author @me --state open`, `git worktree list`
2. **Pick 2–5 to dig into**: looks stuck (N days idle, no one waiting on
   anyone) / PR merged but worktree lingering / untracked files unrelated
   to branch topic / matches user's direction hint
3. **Dig in**: `gh pr view <num> --json reviews,comments,statusCheckRollup`
   for comments + CI; `git -C <p> status` / `branch -r --contains HEAD` /
   `stash list` for local
4. **Group into the 6 categories below** (skip empty categories)
5. **Output per Report Format below**

## Report Format

Each category as three parts:

```
**Category N: [plain-language description].**
[Analogy that lands instantly]
Path: [direction for handling — only "what kind of action" level]
```

Then a fallback slot:

```
**Fallback:** I also noticed [...] but can't cleanly fit it — take a look?
```

Last line: **Which one first?**

## The 6 Categories

### Category 1: Ball in their court

**Analogy**: the letter is in the mail — no point waiting by the mailbox.
**How to see it**: PR open + not draft + mergeable + no changes-requested
+ you weren't the last commenter; or issue awaiting owner.
**Path**: corresponding worktree can be closed now. Nudges go in a separate list.

### Category 2: Ball in your court

**Analogy**: they wrote back. You haven't opened the letter.
**How to see it**: unresolved review comments / failing CI / merge
conflicts (`CONFLICTING`) / owner @-mentioned you, no answer.
**Path**: what you actually need to do today. Sort by urgency, process one
at a time.

### Category 3: Zombies

**Analogy**: an old to-do list at the bottom of a drawer — you forgot why
it's there.
**How to see it**: N days idle (7–14), neither "ball in their court" nor
"ball in your court".
**Path**: restart (revive with a comment) / archive (close with reason) /
convert (downgrade to draft, move to discussion).

### Category 4: Ready to wrap up

**Analogy**: dinner is cooked and still in the pan — turn off the heat
and plate it.
**How to see it**: PR mergeable + CI green + approved + no unresolved
comments; or issue actually resolved but never closed.
**Path**: one-click actions. Don't let them squat on the open list.

### Category 5: Ready to bury

**Analogy**: the package was delivered two days ago — the slip is still
on your desk.
**How to see it**: local worktree exists, but its remote branch is gone
or its PR is already merged / closed. This category is about
local-vs-remote correspondence, not workflow state.
**Path**: zero-thought — `git worktree remove <path>`. For paths outside
the repo (`/tmp/...`), `cd` into the parent repo first.

### Category 6: Scattered locally

**Analogy**: N drawers, most stuffed with half-finished things you've
forgotten about.
**How to see it**: working tree not clean / unpushed commits / local
branch with no upstream / stashes you can't remember.
**Path**: each one needs a user decision — keep (move out of repo or
`.gitignore`) / commit / delete. 30 seconds each.

### Fallback

For odd observations that don't fit. Force-fitting loses information.
Typical: *"none of your N terminals are actually doing work"* systems-
level observations, cross-project patterns, historical firefighting in
stash messages.

## Anti-patterns (strict during report writing)

| Anti-pattern | Do this instead |
|---|---|
| Floating at symptom level ("3 PRs are stuck") | Lift to root ("your PRs are routinely waiting days for review") |
| Piling on PR / issue numbers / file names as primary structure | Object numbers are supporting; category framing is primary |
| "Not X but Y" comparison construct | Just say what it is |
| Case-by-case enumeration ("the 1st PR / the 2nd") as main structure | Lift to a category; specifics only in fallback |
| Tables enumerating every field of every PR | Tables for side-by-side comparison only |
| Reports with "mergeable" / "reviewDecision" / "headRef" terminology | Translate: "can be merged / review state / branch name" |
| Auto confidence / priority numbers | "act on this now / not urgent / I'm unsure" |
| Refusing to run when no worktrees exist | Run normally; categories 5 & 6 may be empty |
| Assuming origin = upstream | Check `gh repo view --json parent` first; fork projects need parent lookup |

## Boundaries (what this skill does NOT do)

- ✗ Read-only: doesn't modify files / commit / push / stash / `worktree remove`
- ✗ Doesn't close PRs or issues / doesn't archive / doesn't notify
- ✗ Doesn't do collaborator comment summary (that's `/myload comments`, planned)
- ✗ Doesn't watch in real time (that's `/myload --watch`)
- ✗ Doesn't dump GitHub API field names / doesn't bucket mechanically

## Self-check before sending

1. Any "mergeable" / "reviewDecision" / "headRef" / "upstream" terminology?
   → translate
2. "Not X but Y" constructs? → rewrite
3. Case-by-case enumeration as main structure? → lift to category
4. Is each item really *a category*, or just *one specific object*?
   Latter → fallback or re-categorize
5. Does the analogy need technical context to land? → swap
6. Is "path" at the *direction* level — no file paths or line numbers?
7. Did you end with "Which one first?"
8. Any empty category force-included? → drop it
