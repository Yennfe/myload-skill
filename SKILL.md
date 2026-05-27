---
name: myload
description: >
  Use when 用户说"盘点一下""不知道哪些事在飞""不敢关终端怕丢东西""被 review
  评论压得喘不过气""哪些 PR 卡住了""我能关哪些终端""哪些 worktree 该清""too
  many open PRs""which terminals can I close""which worktrees can I remove";
  或用户在 git + GitHub 协作项目里同时挂着多个 open PR / open issue / 本地
  worktree、感到被淹没;或输入 /myload 或 /myload <自然语言>。
---

# /myload —— 协作者工作量盘点

## 你要做什么

被 PR / issue / worktree 淹没时,自己读 `git` + `gh`,看出用户**眼下工作量
分布在哪几类**,白话报告。这是工作量归纳,不是清单堆砌。

类比:用户出差回来,你一句话告诉他"3 件可签字、2 件等别人、1 件已经过期可以扔了"。

## 触发与适用

| 输入 | 含义 |
|---|---|
| `/myload` | 完全自主,盘全部 |
| `/myload <自然语言>` | 优先级提示(不是硬过滤) |

要求:当前目录在 git 仓库 + `gh` 已登录 + 用户协作者权限。**Fork 处理**:
`gh repo view --json parent` 检测;若是 fork,PR / issue 查询传
`--repo <parent.owner>/<parent.name>`,不是 origin,否则全漏。

## 工作流

1. **拿地图**:`git remote -v` / `gh repo view --json parent` /
   `gh pr list --author @me --state open` /
   `gh issue list --author @me --state open` / `git worktree list`
2. **自选 2-5 深挖**:看起来卡的(N 天无动静且谁也没等)/ PR 已合但 worktree 在 /
   未跟踪文件跟分支主题不符 / 用户引导相关
3. **深挖**:`gh pr view <num> --json reviews,comments,statusCheckRollup`;
   `git -C <p>` 的 `status` / `branch -r --contains HEAD` / `stash list`
4. **归 6 类**(空类不报),按下面格式输出

## 报告格式

每类三段:**白话描述** + 类比 + 思路。末尾留兜底 + 最后一行"**哪个先做?**"。

## 6 类工作量

**1. 球在别人那** — 信寄出去,不需守邮筒
看:PR open + 非 draft + mergeable + 无 changes-requested + 你不是最新评论人;或 issue 等 owner
做:对应 worktree 可关。要催的话单独列。

**2. 球在你这** — 别人回信,你没看
看:unresolved review comment / CI 红 / 冲突(mergeable: CONFLICTING) / owner @你没回
做:真正"今天动"的事。按紧急度逐个处理。

**3. 僵尸** — 抽屉里过期清单,忘了为啥还留
看:超过 N 天(7-14)无活动,既不属第 1 也不属第 2
做:重启(评论唤醒)/ 归档(close + 说明)/ 转草稿、discussion。

**4. 可收尾** — 菜烧好了关火端走
看:PR mergeable + CI 绿 + approved + 无 unresolved;或 issue 已解决但没关
做:一键能做的事。

**5. 可收尸** — 快递签收两天,凭证还在桌上
看:本地有 worktree,但对应远端分支已删 / 对应 PR 已 merged / closed。看本地-远端对应关系
做:`git worktree remove <path>`(漂在仓库外的先 cd 父仓库)。

**6. 散落本地** — N 抽屉,大半半成品
看:working tree 不干净 / 未 push commit / 本地分支无 upstream / stash 想不起来内容
做:每条 keep(移出或 .gitignore)/ commit / delete,各 30 秒。

**兜底** — 看到怪现象不属于上面 6 类的,老实说出来(典型:"N 个终端没一个在干活"
这种系统性观察、跨项目模式、stash 历史救火痕迹)。强行归类损失信息。

## 反模式清单(严守)

| 反模式 | 怎么做 |
|---|---|
| 漂浮表象("3 个 PR 卡住") | 抽到本质("你 PR 长期等不到 review") |
| 堆 PR 号 / 文件名 / 行号当主结构 | 对象号是辅助,类别归纳是主结构 |
| "不是 X 而是 Y" 句式 | 直接讲是什么 |
| case-by-case 编号当主结构 | 抽到一类,具体只在兜底允许 |
| 表格穷举每个 PR 所有字段 | 表格只做横向对比 |
| 出现 "mergeable" / "reviewDecision" / "headRef" | 翻白话("能合并 / review 状态 / 分支名") |
| confidence / 优先级数字 | "应该马上 / 不太确定 / 看上去不急" |
| 没 worktree 就拒绝跑 | 正常跑,第 5、6 类可空 |
| 默认 origin = upstream | 先 `gh repo view --json parent`,fork 要查 parent |

## 边界(本 skill 不做)

**只读** — 不改文件 / commit / push / stash / `worktree remove`;不 close PR/issue;
不归档 / 不通知;不做协作者评论摘要(那是 `/myload comments`,占位);不实时监控
(那是 `/myload --watch`);不堆 GitHub API 字段名 / 不机械聚类。

## 写完报告前自检

1. 有 "mergeable" / "reviewDecision" / "headRef" 等术语? → 翻白话
2. "不是 X 而是 Y" 句式? → 改
3. case-by-case 编号当主结构? → 抽到一类
4. 每条是"一类"还是"一个对象"? 后者 → 兜底或重归
5. 类比要技术背景才懂? → 换
6. 思路在"处理方向"层面,没具体文件 / 行号?
7. 末尾问"哪个先做?"了吗?
8. 有空类硬凑? → 删
