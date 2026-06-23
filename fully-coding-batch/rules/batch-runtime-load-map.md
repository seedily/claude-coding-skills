---
name: batch-runtime-load-map
description: fully-coding-batch 每步运行时最小上下文加载索引，用于减少默认读取文档和 token 消耗。
---

# Batch Runtime Load Map

> 原则：batch 入口只负责调度；单个子任务实现细节交给 `fully-coding` 的 `runtime-load-map.md` 和 `workflow-rules.md`。

| 场景 | 角色 | 必读 | 按需读取 |
|---|---|---|---|
| 初始化 | batch Orchestrator | `../../fully-coding/project-config.md`; `agents/batch-orchestrator.md`; `rules/batch-runtime-load-map.md` | `../../fully-coding/rules/heartbeat-resume-rules.md` |
| 需求拆分 | Requirement Analyst + batch Orchestrator | `agents/batch-orchestrator.md` Step 1; `rules/batch-workflow-rules.md` §2; `../../fully-coding/templates/user-story-template.md`; `templates/batch-progress-template.md` | 需求终稿文档 |
| 串行启动 | batch Orchestrator | `agents/batch-orchestrator.md` Step 2; `rules/batch-workflow-rules.md` §3; `BATCH-{batch-ts}-progress.md` | `../../fully-coding/rules/git-rules.md` |
| 进度监控 | batch Orchestrator | `agents/batch-orchestrator.md` Step 3; `rules/batch-workflow-rules.md` §4; `BATCH-{batch-ts}-progress.md` | 子任务 `{ts}-codingLog.md`; `{ts}-blockLog.md` |
| 完成汇总 | batch Orchestrator | `agents/batch-orchestrator.md` Step 4; `rules/batch-workflow-rules.md` §5; `BATCH-{batch-ts}-progress.md` | 子任务 codingLog Step 8；错误码文档；公共枚举 |
| auto-resume | batch Orchestrator | `rules/batch-resume-rules.md`; `../../fully-coding/rules/heartbeat-resume-rules.md`; 最新 `BATCH-*-progress.md` | 当前子任务 codingLog/blockLog；`../../fully-coding/rules/git-rules.md` |

## 轻量加载规则

- 不在 batch 层加载 fully-coding 的 DDD、编码、持久层、安全、异常、评审规则；这些由子任务自身按 `../../fully-coding/rules/runtime-load-map.md` 加载。
- `../../QUICKSTART-fully-coding-batch.md` 只供人类阅读，运行时不默认读取。
- 只有执行 git 操作时才读取 `../../fully-coding/rules/git-rules.md`。
- 只有恢复或接管时才读取 `rules/batch-resume-rules.md`。
