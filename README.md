# myload — collaborator workload triage skill for Claude Code

A Claude Code skill that helps GitHub collaborators triage their own workload
across open PRs, open issues, and local git worktrees — in plain language,
with analogies, ending with **"which one first?"**.

Born from one specific exhaustion: 8 terminals open, can't remember what's
in each, afraid to close any, getting buried under GitHub notifications.

## What it does

When you type `/myload` (or `/myload <some freeform direction>`) in Claude
Code, the skill walks `git` + `gh`, looks at your open PRs, open issues,
and all your local worktrees, and reports back grouped into **6 categories**:

| # | Category | What it captures |
|---|---|---|
| 1 | **Ball in their court** | PRs waiting on review, issues waiting on a decision. The corresponding worktree can be closed. |
| 2 | **Ball in your court** | Review comments to address, failed CI, merge conflicts, @-mentions you haven't answered. |
| 3 | **Zombies** | No activity for N days; no one is actually waiting on anyone. Restart, archive, or convert. |
| 4 | **Ready to wrap up** | CI green + approved + no unresolved comments. One-click merge or close. |
| 5 | **Ready to bury** | Worktrees whose PR is already merged or closed, lingering on your disk. Zero-thought `git worktree remove`. |
| 6 | **Scattered locally** | Untracked files, unpushed commits, dangling stashes. Each needs a quick keep/commit/delete call. |

Plus a **fallback** slot for things that don't cleanly fit.

## Why a skill instead of a script

A script could list these categories mechanically. The skill format lets
Claude *read your situation* and report in plain language — including subtle
calls like *"your main worktree has 4 untracked .md files whose names don't
match the current branch's topic; those are at risk of being silently carried
across when you switch branches."* That kind of judgment doesn't fit a
CLI flag.

The skill is **read-only**. It never modifies files, never commits, never
removes worktrees. It tells you what's safe to close and what needs action;
the actual cleanup is your call.

## Installation

Clone into `~/.claude/skills/myload/` (Claude Code's user skills directory):

```bash
git clone https://github.com/<your-username>/myload-skill.git ~/.claude/skills/myload
```

Then in any Claude Code session inside a git + GitHub project where `gh`
is authenticated, type `/myload`.

## Requirements

- Claude Code
- `gh` CLI, authenticated (`gh auth status`)
- A git repository as the current working directory
- Collaborator-level access on the GitHub repo (so you can list your own PRs/issues)

## Language

`SKILL.md` is in Chinese — the author's working language; the analogies and
the anti-pattern list are calibrated for Chinese-native LLM behavior.
`SKILL.en.md` is the English port. Claude Code loads `SKILL.md`; if your
team works primarily in English, swap the files:

```bash
mv SKILL.md SKILL.zh.md
mv SKILL.en.md SKILL.md
```

## Status

Field-tested once on the author's own Sedna project workload (7 open PRs,
10 open issues, 6 worktrees) and produced useful, accurate output. It has
*not* yet been through formal RED-GREEN-REFACTOR skill testing
(`superpowers:writing-skills` methodology). PRs welcome.

## License

MIT — see [`LICENSE`](LICENSE).

---

# myload —— Claude Code 协作者工作量盘点 skill(中文)

一个给 Claude Code 用的 skill,帮 GitHub 协作者盘自己手上飞着的工作 ——
跨 open PR、open issue、本地 git worktree 三种状态,白话报告 + 类比 +
末尾问**"哪个先做?"**。

起源于一个具体的疲惫:8 个终端开着,记不清每个在干嘛,不敢关任何一个,
还被 GitHub 通知埋住。

## 它做什么

你在 Claude Code 里输入 `/myload`(或者 `/myload <任意自然语言引导>`),
skill 会自己跑 `git` + `gh`,看你的 open PR、open issue、所有本地 worktree,
归纳成 **6 类**:

| # | 类别 | 抓什么 |
|---|---|---|
| 1 | **球在别人那** | PR 等 review、issue 等拍板。对应 worktree 可以关。 |
| 2 | **球在你这** | 有 review 评论要回、CI 红、有冲突、@你的问题没回。 |
| 3 | **僵尸** | N 天没动,谁也没在等谁。重启 / 归档 / 转其他形式。 |
| 4 | **可收尾** | CI 绿 + approved + 没 unresolved。一键合并/关闭。 |
| 5 | **可收尸** | 对应 PR 已 merged/closed 但 worktree 还在。零思考 `git worktree remove`。 |
| 6 | **散落本地** | 未跟踪文件、未 push commit、挂着的 stash。每条要 keep/commit/delete。 |

外加一个**兜底位置**,放不进 6 类的现象老实说出来。

## 为什么用 skill 而不是脚本

脚本能机械列出 6 类。skill 的形式让 Claude *看你的实际情况*并白话报告 ——
包括"主 worktree 里有 4 个未跟踪 .md 文件,文件名跟当前分支主题不相关,
切分支时容易被无声携带过去"这种判断。这种事塞不进 CLI 工具。

skill 是**只读的**。不会改文件、不会 commit、不会删 worktree。它告诉你
什么能放心关、什么要动手,实际清理是你的事。

## 安装

clone 到 `~/.claude/skills/myload/`(Claude Code 的用户 skill 目录):

```bash
git clone https://github.com/<your-username>/myload-skill.git ~/.claude/skills/myload
```

之后在任何 `gh` 已登录的 git + GitHub 项目里,在 Claude Code 里输入 `/myload`。

## 前置条件

- Claude Code
- `gh` CLI 已登录(`gh auth status` 通过)
- 当前目录在某个 git 仓库里
- 你在该 GitHub 仓库有协作者权限(能 list 自己的 PR/issue)

## 语言

`SKILL.md` 是中文版(作者主要工作语言;类比和反模式清单按中文母语 LLM 行为
校准过)。`SKILL.en.md` 是英文移植版。Claude Code 加载 skill 时读
`SKILL.md`;如果你们团队主要用英文,可以 swap 一下:

```bash
mv SKILL.md SKILL.zh.md
mv SKILL.en.md SKILL.md
```

## 状态

在作者的 Sedna 项目工作场景下(7 个 open PR、10 个 open issue、6 个 worktree)
实地跑过一次,产出有用且准确。**没有**走过完整的 RED-GREEN-REFACTOR
skill 测试(`superpowers:writing-skills` 方法论)。欢迎 PR。

## License

MIT —— 见 [`LICENSE`](LICENSE) 文件。
