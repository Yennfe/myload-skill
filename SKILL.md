---
name: myload
description: >
  Use when 用户在某个 git + GitHub 协作项目里同时有多个 open PR / open issue
  和多个本地 git worktree,感到"不知道哪些事在飞""不敢关终端怕丢东西""被
  review 评论压得喘不过气""worktree 不知道哪些该清";或用户说"盘点一下""我
  能关哪些终端""哪些 PR 卡住了""too many open PRs""don't know which
  terminals to close""which worktrees can I remove"。也适用于用户输入
  /myload 或 /myload <自然语言>。
---

# /myload —— 协作者工作量盘点工具

## 你要做什么

让用户在协作项目里被 PR / issue / worktree 淹没时,不靠他描述,你自己从 git + gh
看出"他眼下的工作量分布在哪几类",白话报给他,他选哪些先处理。

这件事的核心是"理解用户在这个仓库里有多少在飞的工作,从状态归纳出眼下负担",
不是"列出所有 PR 让他自己看"。陷入清单堆砌就跑偏了。

类比:你是用户的助理,在他出差回来时一句话告诉他"3 件你能马上签字的、2 件还在等
别人、1 件已经过期可以扔了"。统计件数是别人的工作。

---

## 适用范围

任何 git + GitHub 协作项目,要求:

- 当前工作目录在某个 git 仓库里
- `gh` 已经登录(`gh auth status` 通过)
- 用户有协作者身份(能 list 自己的 PR / issue)

**不限定特定仓库**。但当 origin 是个 fork 时(`gh repo view --json parent` 有
parent),用户的 PR 通常提到上游,要同时查上游的 PR 列表才完整 — 否则会漏。

---

## 你被触发的时刻

| 输入 | 你的理解 |
|---|---|
| `/myload` | 完全自主,盘全部 worktree + 全部 open PR + 全部 open issue |
| `/myload 今天` | 只看今天有活动的 |
| `/myload 文件预览那块` | 优先深挖跟某个主题相关的 PR / issue |
| `/myload 我能马上关哪些终端` | 复合诉求,重点关注"可收尸"和"散落本地"两类 |

引导是优先级提示,**不是**硬过滤 — 你看到值得报告的负担,即使不在引导范围内,
也报出来。

---

## 工作流

### Step 1: 看粗略地图

不要一头扎进单个 PR 的细节。先看整体。

```bash
# 仓库 + 上游(如果是 fork)
git remote -v
gh repo view --json owner,name,parent

# 用户名
gh api user --jq .login

# 全部 open PR(注意:fork 项目要查 parent repo,不是 origin)
gh pr list --repo <parent-or-origin> --author @me --state open \
  --json number,title,headRefName,isDraft,mergeable,reviewDecision,updatedAt,url

# 全部 open issue
gh issue list --repo <parent-or-origin> --author @me --state open \
  --json number,title,updatedAt,labels,url

# 全部 worktree
git worktree list
```

关键信息:

- PR:号、draft/ready、mergeable、review decision、距上次更新多久
- Issue:号、label 里有没有优先级、距上次更新多久
- Worktree:路径、分支、对应 PR(用分支名匹配)、工作区脏不脏、未 push commit、
  本地 commit 在不在远端

### Step 2: 自己决定哪几个深挖

判据:

- 看起来卡住的(N 天没动,状态既不是"等我"也不是"等别人")
- 对应 PR 已经合并/关闭但 worktree 还在的(典型"可收尸")
- 工作区有未跟踪文件且文件名跟当前分支主题不相关的
- 跟用户引导词相关的
- 跟过去几次 /myload 报过同类的

选 2-5 个深挖。多了你抓不住主线。

### Step 3: 深挖

对每个选中对象:

```bash
gh pr view <num> --repo <r> --json reviews,comments,statusCheckRollup,latestReviews
git -C <worktree> branch -r --contains HEAD   # 本地 commit 在不在远端
git -C <worktree> status --short              # 工作区细节
git -C <worktree> stash list                  # stash
```

需要时 `Read` 看本地笔记文件;`gh api repos/.../pulls/<num>/comments` 看内联评论。

### Step 4: 归纳工作量分布

从深挖中抽出**几类**眼下负担。

报告抽到本质("你有几个 PR 长期没人 review,卡了几天")。case-by-case 数据
("#178、#191、#209")是辅助,不是主结构。可以在每类后面附"具体对象"作为行动指引,
但**类别归纳本身要先于对象列举**。

6 类判据见下面 §6 类工作量。

### Step 5: 输出报告

按下面 §报告格式 写。

---

## 报告格式

每类三段式:

```
**第 N 类: [白话描述这是哪一类负担]。**
[一个类比让人秒懂]
思路: [处理方向,只到"应该怎么对付这一类"层面]
具体对象(可选): [如果列对象号能帮用户行动就列;只为列而列就删掉]
```

报告底部留**兜底**:

```
**兜底:** 我还看到 [...具体现象...] 但说不清是哪类,你看看?
```

允许不强行归类。空类不报。

最后一行问用户:**哪个先做?**

---

## 6 类工作量判据

### 第 1 类: 球在别人那

**类比**:信寄出去了,不需要守在邮筒边。

**怎么看出来**:

- PR open + 非 draft + mergeable + 没有 changes requested + 最新评论不是你
- 等 review 的(`reviewDecision` 为空或 `REVIEW_REQUIRED`)
- 等指派人/owner 拍板的 issue

**典型形态**:

- PR ready 但没人 review,卡了 N 天
- Issue 等 owner 给方向

**处理思路**:对应本地 worktree 现在就能关。要催的话单独列。

### 第 2 类: 球在你这

**类比**:别人回了你信,你还没看。

**怎么看出来**:

- PR 上有 unresolved review comment / changes requested 且最新评论不是你
- CI 失败的 PR
- mergeable: CONFLICTING
- Issue 里 owner @你 提问没回

**典型形态**:

- 协作方给了反馈你还没动
- CI 红了几天
- merge base 动了产生冲突

**处理思路**:这是真正"要你今天动"的事。按紧急度排序逐个处理。

### 第 3 类: 僵尸

**类比**:抽屉里的过期清单,你以为还在等什么但早就忘了。

**怎么看出来**:

- PR / Issue 超过 N 天没活动(N 自己判断,通常 7-14 天)
- 既不属于"球在别人那"也不属于"球在你这"(没人在等谁)
- 用户也想不起来在干嘛的

**典型形态**:

- 提了 PR 之后没人 review 你也没催
- 开了 issue 讨论着断了
- 自己开的探索分支没合也没关

**处理思路**:三选一 — 重启(留评论唤醒)/ 归档(close + comment 原因)/ 转其他
形式(降为草稿、转 discussion)。不要让它继续僵着。

### 第 4 类: 可收尾

**类比**:菜烧好了还在锅里,关火端走就完事。

**怎么看出来**:

- PR mergeable + CI 绿 + approved + 没有 unresolved comment
- Issue 实际问题已解决(对应 PR merged 或外部条件满足)只是没关

**典型形态**:

- 等 owner merge(你没合并权限)
- 自己能合的 PR 还没合
- 解决了忘记关的 issue

**处理思路**:一键能做的事。不让它继续占着 open 列表。

### 第 5 类: 可收尸

**类比**:快递签收两天了,凭证还摆在工位上。

**怎么看出来**:

- 本地有 worktree,但对应远端分支已被删 / 对应 PR 已 merged 或 closed
- `git worktree list` 列的路径,branch 在远端找不到,或对应 PR 状态 ≠ open
- 这一类**主要看本地 worktree 跟远端的对应关系**,不看 PR/issue 流程状态

**典型形态**:

- PR 已 merged 但 worktree 没清
- PR 被 closed(rejected)你也忘了清
- 当时 review 用的实验 worktree 早该删

**处理思路**:零思考 — `git worktree remove <path>`。如果路径漂在仓库外
(`/tmp/...` 或独立目录),先 cd 进父仓库再删。

### 第 6 类: 散落本地

**类比**:N 个抽屉,大半塞着不同程度的半成品,你以为都在用。

**怎么看出来**(主要看本地状态):

- worktree 工作区不干净(未跟踪 / 未提交)
- 本地 commit 没 push
- 本地分支没有 upstream
- stash 挂着但你想不起来里面是什么

**典型形态**:

- 主 worktree 里混了几个跟当前分支无关的笔记文件 — 切分支时容易被无声携带
- 临时分支 commit 了没 push,只活在这台机器上
- 历史 stash 没清,叠了好几层

**处理思路**:每条要用户判断 — keep(移出仓库或 .gitignore)/ commit(新分支或当前)
/ delete。不能批量处理,但每条 30 秒能决定。

### 兜底

**类比**:作业最后留一栏"我还想补充一点"。

看到些奇怪现象,但说不清属于上面 6 类的哪一类 — 老实说出来。强行归类损失信息。

**典型形态**:

- "N 个终端里没有 1 个是真的在干活的"这种系统性观察
- 跨项目的工作量观察
- stash 备注里看出来的历史救火痕迹

---

## 反模式清单(写报告时严守)

| 反模式 | 应该怎么做 |
|---|---|
| 漂浮在表象("3 个 PR 卡住") | 抽到本质("你提的 PR 长期等不到 review") |
| 堆 PR 号 / Issue 号 / 文件名 / 行号 | 主体用白话 + 类比;对象号是辅助,不是替代分类 |
| "不是 X 而是 Y" 句式 | 直接讲是什么,不用对比句式 |
| case-by-case 编号("第 1 个 PR / 第 2 个 PR")当主结构 | 抽到一类,只在兜底位置允许具体现象描述 |
| 表格罗列每个 PR 的所有字段 | 表格只用于横向对比,不用于穷举 |
| 报告里出现 "mergeable" / "reviewDecision" / "headRef" 等 GitHub 字段名 | 翻成白话:"能合并 / review 状态 / 分支名" |
| 自动 confidence / 优先级数字 | 用"应该马上 / 看上去不急 / 不太确定"等口语 |
| 没 worktree 就拒绝跑 | 没 worktree 也要正常跑,只是第 5、6 类可能为空 |
| 默认假设 origin 就是上游 | 先 `gh repo view --json parent`,fork 项目要查 parent |

---

## 工具栈

| 工具 | 用途 |
|---|---|
| `gh pr list --author @me --state open --json ...` | 自己的 open PR |
| `gh issue list --author @me --state open --json ...` | 自己的 open issue |
| `gh pr view <num> --json reviews,comments,statusCheckRollup` | 单 PR 深挖 |
| `gh api repos/<owner>/<r>/pulls/<num>/comments` | 内联评论 |
| `gh repo view --json parent` | fork 关系检测 |
| `git worktree list` | 所有 worktree |
| `git -C <p> status --short` | 工作区状态 |
| `git -C <p> log @{u}..HEAD --oneline` | 未 push commit |
| `git -C <p> branch -r --contains HEAD` | 本地 commit 在不在远端 |
| `git -C <p> stash list` | stash |
| `Read` | 看本地笔记文件 |

**fork 处理**: `gh repo view --json parent` 有 parent 字段 → 用户 PR 通常提到 parent。
PR/Issue 查询要传 `--repo <parent.owner>/<parent.name>`,不是 origin。

**漂在仓库外的 worktree**: `git worktree list` 会列出所有 worktree 不管它在哪。
即使用户在某个子 worktree 里运行 /myload,也要扫主仓库的所有 worktree。

---

## 边界(本 skill 不做什么)

- ✗ 不主动改文件 / commit / push / stash / worktree remove —— /myload 只读,清理由用户拍板
- ✗ 不主动 close PR / close issue
- ✗ 不归档 / 不发通知 —— session 文字输出就行
- ✗ 不做"协作者评论摘要" —— 那是 `/myload comments` 子命令的事(占位中,本版未实现)
- ✗ 不做实时监控 —— 第二版 `/myload --watch` 才做
- ✗ 不堆 GitHub API 字段名 / 不机械聚类

---

## 写完报告前自检

发出去之前过一遍:

1. 报告里有没有 "mergeable" / "reviewDecision" / "headRef" / "upstream" 这类术语?有 → 翻白话
2. 有没有 "不是 X 而是 Y" 句式?有 → 改
3. 有没有 case-by-case 编号当主结构("第 1 个 PR / 第 2 个 PR")?有 → 抽到一类
4. 每条是不是真的"一类",还是其实只是一个具体对象?后者 → 移到兜底或重新归纳
5. 类比是不是真的让人秒懂?还要技术背景才懂 → 换类比
6. 思路是不是只到"处理方向",没给具体改哪个文件 / 哪行?
7. 末尾问"哪个先做?"了吗?
8. 6 类里有没有"空类硬凑"?空就不报。
