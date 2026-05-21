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

When the user is buried under PRs, issues, and worktrees in a collaboration
project, don't ask them to describe it — read `git` and `gh` yourself,
identify "where their current workload actually concentrates", and report it
in plain language. Let them decide what to handle first.

The core of this skill is *understanding how much work is in flight and
grouping the load*, not *listing every PR for the user to read*. Falling into
"checklist sprawl" means you've drifted off mission.

Analogy: you're the user's assistant when they return from a trip — one
sentence: "3 things ready for your signature, 2 still waiting on others, 1
already stale and can be tossed." Counting individual items is someone else's
job.

---

## Applicability

Any git + GitHub collaboration project. Requirements:

- Current working directory is inside a git repo
- `gh` is authenticated (`gh auth status` passes)
- User has collaborator-level access (can list their own PRs / issues)

**Not tied to any specific repo**. But if `origin` is a fork
(`gh repo view --json parent` returns a `parent`), the user's PRs usually
target the upstream — query upstream too, or you'll miss everything.

---

## When you're triggered

| Input | Interpretation |
|---|---|
| `/myload` | Fully autonomous: scan all worktrees + all open PRs + all open issues |
| `/myload today` | Only look at items active today |
| `/myload the file-preview stuff` | Prioritize deep-diving items related to that topic |
| `/myload which terminals can I close right now` | Compound: focus on "ready to bury" and "scattered locally" |

User-given direction is a *priority hint*, not a hard filter. If you see
something worth reporting outside their hint, still report it.

---

## Workflow

### Step 1: Get the rough map

Don't dive into a single PR. See the whole.

```bash
# Repo + upstream (if fork)
git remote -v
gh repo view --json owner,name,parent

# Username
gh api user --jq .login

# All open PRs (for fork projects: query parent repo, not origin)
gh pr list --repo <parent-or-origin> --author @me --state open \
  --json number,title,headRefName,isDraft,mergeable,reviewDecision,updatedAt,url

# All open issues
gh issue list --repo <parent-or-origin> --author @me --state open \
  --json number,title,updatedAt,labels,url

# All worktrees
git worktree list
```

Key info per object:

- PR: number, draft/ready, mergeable, review decision, time since last update
- Issue: number, priority label if any, time since last update
- Worktree: path, branch, matching PR (by branch name), working tree clean/dirty,
  unpushed commits, whether local commit exists on remote

### Step 2: Choose 2–5 items to dig into

Criteria (not exhaustive):

- Looks stuck (N days idle, but state isn't "waiting on me" or "waiting on them")
- Corresponding PR is merged/closed but worktree still exists ("ready to bury")
- Working tree has untracked files whose names don't match the branch topic
- Matches the user's direction hint
- Same category came up in past `/myload` runs

Pick 2–5. More and you lose the thread.

### Step 3: Dig in

For each selected object:

```bash
gh pr view <num> --repo <r> --json reviews,comments,statusCheckRollup,latestReviews
git -C <worktree> branch -r --contains HEAD   # is local commit on remote?
git -C <worktree> status --short              # working tree detail
git -C <worktree> stash list                  # stash
```

When needed: `Read` for local note files; `gh api repos/.../pulls/<num>/comments`
for inline comments.

### Step 4: Categorize the workload

Extract a *handful of categories* from your deep-dive.

Report at the abstract level ("you have several PRs that have been waiting on
review for days"). Case-by-case data ("#178, #191, #209") is supporting
material, not the main structure. Listing object numbers is fine as actionable
detail *after* the category framing — not as a substitute for it.

See §The 6 Categories below.

### Step 5: Output the report

Per §Report Format below.

---

## Report Format

Each category as three parts:

```
**Category N: [plain-language description of this kind of load].**
[An analogy that makes it click instantly]
Path: [direction for handling — only to the "what kind of action" level]
Specific objects (optional): [list object numbers IF it helps the user act;
                              if listing for the sake of listing, drop it]
```

End with a **fallback** slot:

```
**Fallback:** I also noticed [...specific observation...] but can't cleanly
fit it into the categories — take a look?
```

You're allowed to not force-fit. Empty categories: don't report them.

Last line: **Which one first?**

---

## The 6 Categories

### Category 1: Ball in their court

**Analogy**: the letter is in the mail — no point waiting by the mailbox.

**How to see it**:

- PR is open + not draft + mergeable + no changes-requested + you weren't the last commenter
- Awaiting review (`reviewDecision` empty or `REVIEW_REQUIRED`)
- Issue waiting on assignee/owner decision

**Typical forms**:

- PR is ready but no one has reviewed, stuck for N days
- Issue submitted, awaiting owner direction

**Path**: the corresponding local worktree can be closed right now. List
nudges separately if any.

### Category 2: Ball in your court

**Analogy**: they wrote back. You haven't opened the letter yet.

**How to see it**:

- PR has unresolved review comments / changes-requested AND the last comment wasn't yours
- PR with failing CI
- mergeable: CONFLICTING
- Issue where the owner @-mentioned you with a question you haven't answered

**Typical forms**:

- Collaborator gave feedback you haven't acted on
- CI has been red for days
- Merge base moved and produced conflicts

**Path**: this is what you actually need to do today. Sort by urgency and
process one at a time.

### Category 3: Zombies

**Analogy**: an old to-do list at the bottom of a drawer — you forgot why
it's even there.

**How to see it**:

- PR/Issue with no activity for N days (N at your discretion, usually 7–14)
- Doesn't fit "ball in their court" *or* "ball in your court" (no one is waiting on anyone)
- User probably can't remember what it was for either

**Typical forms**:

- Submitted a PR, no one reviewed, you didn't nudge
- Opened an issue, discussion trailed off
- Your own exploration branch, neither merged nor closed

**Path**: three options — restart (leave a comment to revive) / archive
(close with a reason) / convert (downgrade to draft, move to discussion).
Don't let it keep dragging.

### Category 4: Ready to wrap up

**Analogy**: dinner is cooked and still in the pan — turn off the heat and plate it.

**How to see it**:

- PR mergeable + CI green + approved + no unresolved comments
- Issue actually resolved (corresponding PR merged or external condition met)
  but not yet closed

**Typical forms**:

- Waiting on owner to merge (you lack merge permission)
- Your own PR you can merge but haven't
- Issue solved but forgotten in the open list

**Path**: one-click actions. Don't let them keep taking up "open" slots.

### Category 5: Ready to bury

**Analogy**: the package was delivered two days ago — the slip is still on your desk.

**How to see it**:

- Local worktree exists, but the corresponding remote branch is gone / the
  corresponding PR is already merged or closed
- `git worktree list` shows the path, but its branch can't be found upstream
  OR its PR is no longer open
- **This category is about local-worktree-vs-remote correspondence**, not
  PR/issue workflow state

**Typical forms**:

- PR was merged; you forgot to clean the worktree
- PR was closed (rejected); you also forgot to clean
- An experimental worktree from a past review, should have been deleted

**Path**: zero-thought — `git worktree remove <path>`. If the path is outside
the main repo (`/tmp/...` or a sibling dir), `cd` into the parent repo first.

### Category 6: Scattered locally

**Analogy**: N drawers, most stuffed with half-finished things you've forgotten about.

**How to see it** (mostly local state):

- Worktree working tree not clean (untracked / uncommitted)
- Local commits not pushed
- Local branch with no upstream
- Stashes hanging around — you can't remember what's in them

**Typical forms**:

- Main worktree has unrelated note files mixed in — easy to silently carry
  across when switching branches
- Temp branch committed but not pushed, only exists on this machine
- Historical stashes never cleaned, stacked several deep

**Path**: each item needs a user decision — keep (move out of repo or
.gitignore) / commit (new branch or current) / delete. Can't batch, but each
one takes 30 seconds to decide.

### Fallback

**Analogy**: a blank line at the end of your homework for "anything else I want to add".

If you see something odd that doesn't cleanly fit into the 6 categories — say
so plainly. Force-fitting loses information.

**Typical forms**:

- "None of your N terminals are actually doing work right now" — a systems-
  level observation
- Cross-project workload patterns (if user works in multiple repos)
- Historical firefighting traces visible in stash messages

---

## Anti-patterns (strict during report writing)

| Anti-pattern | What to do instead |
|---|---|
| Floating at symptom level ("3 PRs are stuck") | Lift to root ("your PRs are routinely waiting days for review") |
| Piling on PR / issue numbers / file names / line numbers | Main body in plain language + analogy; object numbers are supporting, not structural |
| "Not X but Y" comparison construct | Just say what it is, no comparison framing |
| Case-by-case enumeration ("the 1st PR / the 2nd PR") as primary structure | Lift to a category; case detail only allowed in fallback |
| Tables listing every field of every PR | Tables only for side-by-side comparison, not for enumeration |
| Reports containing "mergeable" / "reviewDecision" / "headRef" GitHub field names | Translate to plain language: "can be merged / review state / branch name" |
| Auto confidence / priority numbers | Use natural language: "act on this now / not urgent / I'm unsure" |
| Refusing to run when no worktrees exist | Run normally — categories 5 & 6 just end up empty |
| Assuming origin = upstream | First check `gh repo view --json parent`; fork projects need parent lookup |

---

## Toolchain

| Tool | Purpose |
|---|---|
| `gh pr list --author @me --state open --json ...` | User's open PRs |
| `gh issue list --author @me --state open --json ...` | User's open issues |
| `gh pr view <num> --json reviews,comments,statusCheckRollup` | Deep-dive a PR |
| `gh api repos/<owner>/<r>/pulls/<num>/comments` | Inline comments |
| `gh repo view --json parent` | Detect fork relationship |
| `git worktree list` | All worktrees |
| `git -C <p> status --short` | Working tree state |
| `git -C <p> log @{u}..HEAD --oneline` | Unpushed commits |
| `git -C <p> branch -r --contains HEAD` | Is local commit on remote? |
| `git -C <p> stash list` | Stashes |
| `Read` | Local note files |

**Fork handling**: if `gh repo view --json parent` has a parent field, user's
PRs usually target the parent. Pass `--repo <parent.owner>/<parent.name>` to
PR/issue queries, not origin.

**Worktrees outside the repo**: `git worktree list` shows all of them
regardless of path. Even when running `/myload` from a child worktree, scan
the main repo's full worktree list — some may be floating in `/tmp` or
sibling directories.

---

## Boundaries (what this skill does NOT do)

- ✗ Doesn't modify files / commit / push / stash / `git worktree remove` —
  `/myload` is read-only; cleanup is the user's call
- ✗ Doesn't close PRs or issues
- ✗ Doesn't archive / notify — session text output is enough
- ✗ Doesn't do "collaborator comment summary" — that's `/myload comments`
  (planned, not in this version)
- ✗ Doesn't watch in real time — that's `/myload --watch` (v2)
- ✗ Doesn't dump GitHub API field names / doesn't bucket mechanically — LLM
  understands the workload

---

## Self-check before sending

Run through this before output:

1. Any "mergeable" / "reviewDecision" / "headRef" / "upstream" terminology in
   the report? → translate to plain language
2. Any "not X but Y" constructs? → rewrite
3. Any case-by-case numbering as primary structure ("the 1st PR / the 2nd
   PR")? → lift to category
4. Is each item really *a category*, or just *one specific object*? Latter →
   move to fallback or re-categorize
5. Does the analogy actually land instantly? Still need technical context to
   parse → swap analogy
6. Does the "path" stop at *direction* — not "edit which file / which line"?
7. Did you end with "Which one first?"
8. Any "empty category" being force-included? Empty → skip it.
