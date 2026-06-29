---
name: fully-coding-batch-quickstart
description: fully-coding-batch 快速开始指南。5 分钟上手，自动拆分大需求并串行调度多个子任务。
---

# fully-coding-batch 快速开始

## 1. 前置准备

1. 已安装 Claude Code 并加载 `fully-coding` 与 `fully-coding-batch` 两个 Skill。
2. 项目根目录已存在 `knowledge_dir`（默认 `_knowledge`）与代码工程。
3. 已根据项目情况填写 `.claude/skills/fully-coding/project-config.md`（batch 模式共享此配置）。

> 若首次使用 `fully-coding`，建议先阅读 `QUICKSTART-fully-coding.md` 完成单任务试跑。

## 2. 运行第一个批次任务

当需求包含 ≥3 个独立功能时，推荐用 batch 模式自动拆分并串行执行：

```
/fully-coding-batch 实现用户管理、订单系统、支付集成三个模块
```

启动后，batch Orchestrator 会自动：
1. 生成批次时间戳 `{batch-ts}`（格式 `yyyyMMdd-HHmmss`）。
2. 分析子任务依赖关系，确定串行执行顺序。
3. 创建 `.dev-log/BATCH-{batch-ts}-progress.md` 作为批次总控文档。
4. 为每个子任务生成 `{tsN}-userStory.md` 与 `{tsN}-codingLog.md`。
5. 创建或复用共享 git 分支 `{type}/{yyyyMM}`（月度大版本分支，同类型同月多任务共用）。
6. 逐个调用 `fully-coding` 完成每个子任务（步骤 3–9），并实时更新批次进度。

## 3. 流程图

```mermaid
flowchart TD
    A[用户输入多功能需求] --> B[生成批次时间戳 {batch-ts}<br/>角色：batch Orchestrator]
    B --> C[需求拆分与依赖分析<br/>角色：需求分析师]
    C --> D[创建 BATCH-{batch-ts}-progress.md<br/>角色：batch Orchestrator]
    D --> E[为每个子任务创建 userStory.md / codingLog.md<br/>角色：batch Orchestrator]
    E --> F[创建或复用共享 git 分支<br/>角色：batch Orchestrator]
    F --> G[确定串行执行顺序<br/>角色：batch Orchestrator]

    G --> T1[子任务 1: fully-coding Step 3-9<br/>角色链：架构师 → 开发者 → DBA + 代码评审员 → 开发者 → 架构师 → Orchestrator]
    T1 --> U1{子任务 1 完成?}
    U1 -->|是| T2[子任务 2: fully-coding Step 3-9<br/>角色链：架构师 → 开发者 → DBA + 代码评审员 → 开发者 → 架构师 → Orchestrator]
    U1 -->|阻塞| X[登记 {ts1}-blockLog.md 并暂停批次<br/>角色：当前子任务角色 + batch Orchestrator]

    T2 --> U2{子任务 2 完成?}
    U2 -->|是| TN[子任务 N: fully-coding Step 3-9<br/>角色链：架构师 → 开发者 → DBA + 代码评审员 → 开发者 → 架构师 → Orchestrator]
    U2 -->|阻塞| X

    TN --> UN{全部子任务完成?}
    UN -->|是| Z[汇总产出并标记 batch completed<br/>角色：batch Orchestrator]
    UN -->|阻塞| X

    X --> R[/fully-coding-batch --auto-resume<br/>角色：batch Orchestrator/]
    R --> C1{阻塞是否已确认?}
    C1 -->|否| X
    C1 -->|是| G
```

单个子任务内部的 `fully-coding Step 3-9` 角色链为：Step 3-4 架构师 → Step 5 开发者 → Step 6 DBA + 代码评审员 → Step 7 开发者 → Step 8 架构师 → Step 9 Orchestrator。

## 4. 查看进度

执行过程中无需等待确认，所有状态写入 `.dev-log/`。查看进度时优先直接读取批次进度文档：

```
Read .dev-log/BATCH-{batch-ts}-progress.md
```

`/fully-coding-batch --auto-resume` 是恢复/接管命令，不是纯状态查询命令；只有需要继续未完成批次时再使用。

进度文档包含：
- 子任务清单与当前状态
- 执行顺序与依赖关系
- 每个子任务的 `codingLog.md` 路径
- 共享 git 分支名

## 5. 断点续跑

若批次因进程退出、电脑重启、子任务阻塞而中断：

```
/fully-coding-batch --auto-resume
```

系统会：
1. 扫描最新 `BATCH-{batch-ts}-progress.md`。
2. 检查 batch Orchestrator 自身心跳与进程存活。
3. 检查是否有运行中的子任务或其他独立任务。
4. 定位第一个未完成的子任务，恢复执行。

若某个子任务阻塞，需先处理阻塞问题并更新 `{ts}-blockLog.md` 为「已确认」，再调用 `--auto-resume`。

## 6. 手动从指定子任务开始

若明确知道要从第 N 个子任务开始，跳过已完成的子任务：

```
/fully-coding-batch --start-from 2
```

> 使用前需确保前序子任务已完成，公共资源（如错误码段位）已正确落盘。

## 7. 查看最终产出

批次完成后，`.dev-log/` 下会生成：

| 文件 | 内容 |
|------|------|
| `BATCH-{batch-ts}-progress.md` | 批次总控文档：子任务清单、执行顺序、进度、Git 分支、汇总产出 |
| `{ts1}-userStory.md` ~ `{tsN}-userStory.md` | 各子任务需求分析 |
| `{ts1}-codingLog.md` ~ `{tsN}-codingLog.md` | 各子任务开发日志 |
| `{ts1}-suggestion.md` ~ `{tsN}-suggestion.md` | 各子任务改进建议 |
| `{ts1}-blockLog.md` ~ `{tsN}-blockLog.md` | 若发生阻塞，记录阻塞原因与处理方案 |

## 8. 常用参数速查

| 参数 | 作用 |
|------|------|
| `--start-from N` | 从第 N 个子任务开始执行 |
| `--auto-resume` | 自动读取批次进度文档，从上次中断处继续 |

## 9. 关键约束

- **严格串行**：子任务按依赖顺序逐个执行，禁止并行启动多个 Agent。
- **共享分支**：所有子任务复用同一个 git 分支，后序子任务继承前序代码变更。
- **公共资源实时修改**：每个子任务在步骤 8 中可直接修改错误码文档、公共枚举等，后序子任务基于最新状态继续。
- **batch 子任务标识**：调用 `fully-coding` 时自动传入 `--batch-subtask`，在 `codingLog.md` 中标记 `batch-subtask: true`。
- **阻塞不跳过**：存在待确认阻塞子任务时，不启动后续子任务。
- **保护共享分支**：不重建、删除或强制覆盖共享分支。
- **push 需确认**：`git push` 必须先向用户确认。

## 10. 下一步

- 了解 batch 完整流程：阅读 `fully-coding-batch/SKILL.md`。
- 了解单任务 9 步流程：阅读 `fully-coding/SKILL.md`。
- 了解心跳与续跑机制：阅读 `fully-coding/rules/heartbeat-resume-rules.md`。
- 了解错误码规范：阅读 `fully-coding/rules/error-handling-rules.md`。
