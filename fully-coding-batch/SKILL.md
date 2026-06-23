---
name: fully-coding-batch
description: 针对超长需求的批次串行开发工具。自动拆分大需求为多个子任务，按顺序逐个调用 fully-coding 流水线，监控批量进度。共享 fully-coding/project-config.md 配置。
---

# fully-coding-batch

> 新用户先看 [`QUICKSTART-fully-coding-batch.md`](../QUICKSTART-fully-coding-batch.md)。详细规则以 `rules/batch-workflow-rules.md`、`rules/batch-resume-rules.md` 为准；单个子任务规则复用 `../fully-coding/rules/workflow-rules.md`。

## 目标

`fully-coding-batch` 用于将一个超长、多功能需求拆分为多个子任务，并串行调用 `fully-coding` 完成每个子任务的 Step 3-9。

```text
多功能需求 → 需求拆分 → 共享分支 → 子任务 1 Step 3-9 → 子任务 2 Step 3-9 → ... → 批次汇总
```

## 启动命令

```text
/fully-coding-batch 实现用户管理、订单系统、支付集成三个模块
/fully-coding-batch --auto-resume
/fully-coding-batch --start-from 2
```

## 运行必读顺序

1. `../fully-coding/project-config.md`：共享配置，解析所有 `{{config.*}}`。
2. `rules/batch-runtime-load-map.md`：按场景加载最小上下文。
3. `agents/batch-orchestrator.md`：执行 batch 调度。
4. `rules/batch-workflow-rules.md`：批次拆分、串行启动、进度监控、完成汇总。
5. `rules/batch-resume-rules.md`：仅在 `--auto-resume` 或 `--start-from N` 时读取。

## 执行流程

| 阶段 | 角色 | 输入 | 输出 | 说明 |
|---|---|---|---|---|
| 1 需求拆分 | Requirement Analyst + batch Orchestrator | 用户原始需求 | `BATCH-{batch-ts}-progress.md` + 多个 `{ts}-userStory.md` | 拆分子任务，分析依赖，确定执行顺序 |
| 2 串行启动 | batch Orchestrator | 批次进度文档 | 共享 git 分支 + 子任务执行 | 逐个调用 `/fully-coding --start-step 3 --batch-subtask --git-branch ...` |
| 3 进度监控 | batch Orchestrator | 子任务 codingLog/blockLog | 更新 BATCH 进度 | 完成后继续下一个；阻塞时暂停 |
| 4 完成汇总 | batch Orchestrator | 所有子任务 codingLog Step 8 | 批次汇总与 completed 状态 | 汇总公共资源变更，不重复落盘 |

## 核心红线

- 严格串行执行，禁止并行启动多个 Agent、Team 或 background subagent。
- 每个子任务复用 `fully-coding` 完整 Step 3-9 流程，batch 层不重写子任务实现规则。
- 所有子任务共享同一 git 分支，必须通过 `--git-branch` 传入。
- 调用 fully-coding 时必须传入 `--batch-subtask`。
- 阻塞子任务未确认前，不得跳过执行后续子任务。
- git push、force push、删除分支、reset hard 等危险操作必须先向用户确认。

## 文件产出

| 文件 | 用途 |
|---|---|
| `{{config.output_dir}}/BATCH-{batch-ts}-progress.md` | 批次总控进度 |
| `{{config.output_dir}}/{tsN}-userStory.md` | 子任务需求文档 |
| `{{config.output_dir}}/{tsN}-codingLog.md` | 子任务开发日志 |
| `{{config.output_dir}}/{tsN}-blockLog.md` | 子任务阻塞记录（如有） |
| `{{config.output_dir}}/{tsN}-suggestion.md` | 子任务自主进化建议 |

## 权威文档索引

| 文档 | 用途 |
|---|---|
| `rules/batch-runtime-load-map.md` | batch 每个场景的最小加载上下文 |
| `agents/batch-orchestrator.md` | batch Orchestrator 动作清单 |
| `rules/batch-workflow-rules.md` | batch 串行流程、共享分支、阻塞与汇总规则 |
| `rules/batch-resume-rules.md` | batch 断点续跑和 `--start-from` 规则 |
| `templates/batch-progress-template.md` | BATCH 进度文档模板 |
| `../fully-coding/rules/workflow-rules.md` | 子任务 Step 3-9 权威规则 |
| `../fully-coding/rules/heartbeat-resume-rules.md` | 心跳、续跑、进程接管通用规则 |
| `../fully-coding/rules/git-rules.md` | git 分支、commit、push 规则 |
