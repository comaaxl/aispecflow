[English](./README.md) | [中文](./README-zh.md)

# aispecflow

一个面向 AI 编码 agent 的完整开发生命周期插件——从项目探察到文档刷新，一个插件搞定。

## 为什么做这个

各个开发阶段都有好工具。[Grill Me](https://github.com/mattpocock/skills) 用 relentless 访谈打磨需求。[OpenSpec](https://github.com/Fission-AI/openspec) 把规格变成可追踪的产物。[Superpowers](https://github.com/obra/superpowers) 加入了 TDD 纪律和子代理代码审查。它们单独都很强。

但没有一个把从头到尾的流程串起来。你 grill 完需求，然后呢？你写完 spec，然后呢？你写完代码——谁来审查？你归档了变更，但 README 还在描述去年的架构。

Superpowers 最接近完整工作流，但它很重——自带规划系统、worktree 管理，假设你全盘接受它的方法论。有时候你只想要 OpenSpec 的 spec 驱动流程，前面加个 grill 阶段、后面加个代码审查，而不必 adopting 一整个框架。

**aispecflow** 填的就是这个空。它把这些工具最好的想法缝合进一个轻量插件：

- Grill 式需求访谈
- OpenSpec 的 spec 驱动产物（proposal -> specs -> design -> tasks）
- TDD 实施纪律
- 独立子代理代码审查
- 每次变更后同步文档

没有自定义规划系统，没有 lock-in。每个技能可独立使用。一起用是完整流程，挑着用也行。对既有代码库（跑 terrain-scan 了解现状）和从零开始的新项目都同样适用。

## 技能

| # | 技能 | 阶段 | 做什么 |
|---|-------|-------|-------------|
| 0 | `terrain-scan` | 探察 | 扫描代码库 -> `docs/project-overview.md` |
| 1 | `seed-grill` | Grill | Relentless 需求访谈 -> `docs/requirements.md`、`CONTEXT.md`、ADR |
| 2 | `bloom-spec` | Spec | OpenSpec propose -> proposal、specs、design、tasks |
| 3 | `grow-apply` | 实施 | 用 TDD 或直接方式实施任务（用户选择）+ git checkpoint 提交 + 可选任务级审查 + 可选的 review 前项目校验（lint/类型检查/集成测试，按项目配置驱动） |
| 4 | `prune-review` | 审查 | 三级审查：task / change / project，由独立子代理执行 |
| 5 | `harvest-archive` | 归档 | 同步 specs、归档变更、可选归档需求文档 |
| 6 | `renew-docs` | 刷新 | 保持 README + project-overview 最新（基于变更或全量刷新） |
| O | `axl-dev-flow` | 编排器 | 完整生命周期：探察 -> grill -> spec -> 实施 -> 审查 -> 归档 -> 刷新 |

每个技能都可独立调用。它们通过约定路径共享状态——没有技能间 RPC，没有隐藏耦合。

技能名遵循一个生长的生命周期隐喻——从 terrain 到 seed 到 bloom 到 growth 到 pruning 到 harvest 到 renewal：

```
terrain-scan -> seed-grill -> bloom-spec -> grow-apply -> prune-review -> harvest-archive -> renew-docs
```

每个阶段喂养下一个，像花园里的四季。但和真花园不同，你可以随时跳进任何一个季节。

## 完整流程

```
┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐   ┌──────────┐
│ 探察     │──▶│  GRILL   │──▶│   SPEC   │──▶│  实施    │──▶│  审查    │──▶│  归档    │──▶│  刷新    │
│terrain-scan│   │seed-grill│   │bloom-spec│  │grow-apply │   │prune-review│  │archive-  │   │refresh-  │
└──────────┘   └──────────┘   └──────────┘   └──────────┘   └──────────┘   │change    │   │docs      │
      │              │              │              │              │         └──────────┘   └──────────┘
      ▼              ▼              ▼              ▼              ▼              │              │
  project-       requirements   proposal +      代码 +         审查            变更          README +
  overview.md    .md            specs +         测试           报告            已归档        project-
                CONTEXT.md      design +                       (+ 需求       (specs        overview.md
                ADR             tasks                           可选)         已同步)       已更新
```

跑 `/axl-dev-flow` 走完整流水线，或单独用任一技能。

## 使用后的项目结构

```
your-project/
├── openspec/                          # OpenSpec spec 驱动产物
│   ├── specs/                         # 主 specs（归档后同步）
│   │   └── user-auth/
│   │       └── spec.md
│   └── changes/
│       ├── add-user-auth/             # 活跃变更（归档前）
│       │   ├── proposal.md            # 做什么 & 为什么
│       │   ├── specs/
│       │   │   └── user-auth/
│       │   │       └── spec.md        # Delta spec（需求 + 场景）
│       │   ├── design.md              # 技术决策
│       │   └── tasks.md               # 实施清单
│       └── archive/                   # 已归档变更
│           └── 2026-07-05-add-user-auth/
│               ├── proposal.md
│               ├── specs/
│               ├── design.md
│               ├── tasks.md
│               └── requirements.md    #（如果用户选择归档）
├── docs/
│   ├── project-overview.md            # terrain-scan 生成，renew-docs 刷新
│   ├── requirements.md                # seed-grill 生成
│   ├── adr/                           # 架构决策记录
│   │   └── 0001-use-postgres-for-sessions.md
│   └── .archive/                      # 修改前的备份文件
│       └── README.md.20260705-143000.bak
├── CONTEXT.md                         # 领域词汇表（统一语言）
├── README.md                          # 项目 README（renew-docs 刷新）
└── src/                               # 你的实际代码
```

## 安装

### Claude Code

```bash
claude plugin marketplace add comaaxl/aispecflow
claude plugin install aispecflow@aispecflow
```

### Codex

```bash
codex plugin marketplace add comaaxl/aispecflow
codex plugin add aispecflow@aispecflow
```

## 前置条件

- [OpenSpec CLI](https://github.com/Fission-AI/openspec)：`npm install -g @fission-ai/openspec`
- Git：任务级和变更级审查范围需要它。项目级审查无需 git。没有 git 时，grow-apply 仍能实施任务，但任务级子代理审查不可用，变更级审查退化为审查整个工作树。

## grow-apply 如何使用 git

grow-apply 与 git 集成，让审查范围清晰、进度能抗中断：

- **启动检查**——实施前检查工作树。脏工作区会被标记（未提交的改动会污染审查 diff）；你选择提交、stash 还是中止。没有 git 时，提示 `git init` 或降级继续。
- **base + 每任务 checkpoint**——启动时记录 base commit，每个任务作为一个 checkpoint 提交（`task(X.Y): ...`）。变更级审查范围是 `base..HEAD`；任务级审查范围是 `TASK_BASE..HEAD`。
- **每次提交前 self-review**——任务测试全过后，主对话先做一次快速 self-review（inline 修复、重跑测试），*然后*才 checkpoint 提交，让提交反映已审查的状态。
- **apply 状态文件**——`openspec/changes/<change>/.aispecflow-apply-state.md` 记录 base、head、最后完成的任务、你的审查/修复选择。中断后重启时，grow-apply 把它和 tasks.md、git log 交叉对账，并询问如何 resume，而不是盲目继续。
- **只追加提交**——永远只新增提交（`task(...)`、`fix(review-task-X): ...`）；默认不 amend、不 squash。合回 main 用 merge，不用 rebase。

## 可选：用 worktree 隔离变更

本插件自己不管 git worktree，但对 worktree 完全透明。你可以自己建 worktree，在里面跑完整流程——所有 git 命令都作用在 worktree 的分支上（因为相对当前目录）。合回 main 前记得先提交你的工作。

## License

MIT
