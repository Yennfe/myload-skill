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

# /myload —— 协作者工作量盘点

## 你要做什么

用户被一堆 PR / issue / worktree 淹没时,自己读 `git` + `gh`,看出"他眼下
的工作量分布在哪几类",白话报告,他选哪些先处理。

这是**工作量归纳**,不是清单堆砌。陷入"列所有 PR 让他自己看"就跑偏了。

类比:你是用户的助理,一句话告诉他"3 件可签字、2 件等别人、1 件已经过期
可以扔了"。统计件数是别人的工作。

## 触发与适用

任何 git + GitHub 协作项目。要求:当前目录在 git 仓库内 + `gh` 已登录 +
用户有协作者权限。

| 输入 | 你的理解 |
|---|---|
| `/myload` | 完全自主,盘全部 |
| `/myload <自然语言>` | 优先深挖跟主题相关的(优先级提示,不是硬过滤) |

**Fork 处理**:用 `gh repo view --json parent` 检测。如果 origin 是 fork,
用户的 PR 通常提到 parent — PR / issue 查询要传
`--repo <parent.owner>/<parent.name>`,不是 origin。

## 工作流

1. **拿地图**:`git remote -v`、`gh repo view --json parent`、
   `gh pr list --author @me --state open`、
   `gh issue list --author @me --state open`、`git worktree list`
2. **自选 2-5 个深挖**:看起来卡的(N 天无动静且谁也没等)/ 对应 PR 已合
   并但 worktree 还在 / 未跟踪文件跟分支主题不符 / 用户引导词相关的
3. **深挖**:`gh pr view <num> --json reviews,comments,statusCheckRollup`
   看评论 + CI;`git -C <p> status` / `branch -r --contains HEAD` /
   `stash list` 看本地
4. **归纳成下面 6 类**(空类不报)
5. **按下面报告格式输出**

## 报告格式

每类三段:

```
**第 N 类: [白话描述]。**
[类比让人秒懂]
思路: [处理方向,只到"哪一类动作"层面]
```

末尾留兜底:

```
**兜底:** 我还看到 [...] 但说不清属于哪类,你看看?
```

最后一行:**哪个先做?**

## 6 类工作量

### 第 1 类: 球在别人那

**类比**:信寄出去了,不需要守在邮筒边。
**怎么看**:PR open + 非 draft + mergeable + 无 changes-requested + 你不是
最新评论人;或 issue 等 owner 拍板。
**思路**:对应 worktree 现在就能关。要催的话单独列。

### 第 2 类: 球在你这

**类比**:别人回了你信,你还没看。
**怎么看**:有 unresolved review comment / CI 红 / 冲突
(mergeable: CONFLICTING)/ owner @你没回。
**思路**:真正"要今天动"的事。按紧急度逐个处理。

### 第 3 类: 僵尸

**类比**:抽屉里过期清单,你忘了为什么还留着。
**怎么看**:超过 N 天(7–14)无活动,既不在球-别人那也不在球-你这。
**思路**:重启(留评论唤醒) / 归档(close + 说明) / 转其他形式
(草稿、discussion)。

### 第 4 类: 可收尾

**类比**:菜烧好了在锅里,关火端走就完事。
**怎么看**:PR mergeable + CI 绿 + approved + 无 unresolved;或 issue 实际
问题已解决但没关。
**思路**:一键能做的事。

### 第 5 类: 可收尸

**类比**:快递签收两天了,凭证还摆在工位上。
**怎么看**:本地有 worktree,但对应远端分支已删 / 对应 PR 已
merged / closed。这一类主要看本地-远端对应关系。
**思路**:零思考 — `git worktree remove <path>`。漂在仓库外的
(`/tmp/...`)先 cd 进父仓库再删。

### 第 6 类: 散落本地

**类比**:N 个抽屉,大半塞着半成品,你以为都在用。
**怎么看**:working tree 不干净 / 未 push commit / 本地分支无 upstream /
stash 你想不起来里面是什么。
**思路**:每条要用户判断 — keep(移出仓库或 .gitignore) / commit /
delete。每条 30 秒能决定。

### 兜底

看到怪现象不属于上面 6 类的,老实说出来。强行归类损失信息。
典型:"你 N 个终端没一个在干活"这种系统性观察、跨项目模式、stash 历史救火
痕迹。

## 反模式清单(严守)

| 反模式 | 应该怎么做 |
|---|---|
| 漂浮表象("3 个 PR 卡住") | 抽到本质("你 PR 长期等不到 review") |
| 堆 PR 号 / 文件名 / 行号 当主结构 | 对象号是辅助,类别归纳是主结构 |
| "不是 X 而是 Y" 句式 | 直接讲是什么 |
| case-by-case 编号("第 1 个 PR / 第 2 个")当主结构 | 抽到一类,具体只在兜底允许 |
| 表格穷举每个 PR 的所有字段 | 表格只做横向对比,不做穷举 |
| 报告里出现 "mergeable" / "reviewDecision" / "headRef" | 翻成白话("能合并 / review 状态 / 分支名") |
| confidence / 优先级数字 | "应该马上 / 看上去不急 / 不太确定" |
| 没 worktree 就拒绝跑 | 正常跑,第 5、6 类可空 |
| 默认 origin = upstream | 先 `gh repo view --json parent`,fork 要查 parent |

## 边界(本 skill 不做什么)

- ✗ 只读:不改文件 / 不 commit / 不 push / 不 stash / 不 `worktree remove`
- ✗ 不 close PR / 不 close issue / 不归档 / 不通知
- ✗ 不做协作者评论摘要(那是 `/myload comments`,占位)
- ✗ 不实时监控(那是 `/myload --watch`)
- ✗ 不堆 GitHub API 字段名 / 不机械聚类

## 写完报告前自检

1. 有 "mergeable" / "reviewDecision" / "headRef" / "upstream" 等术语?
   → 翻白话
2. "不是 X 而是 Y" 句式? → 改
3. case-by-case 编号当主结构? → 抽到一类
4. 每条是"一类"还是只是"一个对象"? 后者 → 兜底或重归纳
5. 类比要不要技术背景才懂? → 换
6. 思路只到"处理方向",没具体文件名 / 行号?
7. 末尾问"哪个先做?"了吗?
8. 有空类硬凑? → 删
