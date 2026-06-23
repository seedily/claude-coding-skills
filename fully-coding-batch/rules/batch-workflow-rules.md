---
name: batch-workflow-rules
description: fully-coding-batch 批次串行流程规则。定义需求拆分、共享分支、子任务调度、阻塞处理和完成汇总。
---

# Batch Workflow Rules

> 本文件只定义 batch 特有规则。单个子任务的 Step 3-9 执行、代码评审、测试、文档更新、阻塞登记遵循 `../../fully-coding/rules/workflow-rules.md`。

## 1. 执行原则

- 批次内子任务严格串行执行，不得并行启动多个 Agent、Team 或 background subagent。
- 每个子任务从 `/fully-coding --start-step 3` 开始执行，跳过单任务 Step 1/2。
- 所有子任务共享同一个 git 分支，通过 `--git-branch {batch-branch-name}` 传入。
- 所有子任务必须传入 `--batch-subtask`，由 fully-coding 在 codingLog frontmatter 写入 `batch-subtask: true`。
- 完成一个子任务后直接启动下一个子任务，不为“是否继续”暂停；仅在真实阻塞、危险操作或用户打断时暂停。

## 2. Step 1：需求拆分

**角色**：Requirement Analyst + batch Orchestrator

**输入**：用户原始多功能需求；`../../fully-coding/project-config.md`；`../../fully-coding/templates/user-story-template.md`。

**动作**：
1. 识别功能边界，将需求拆分为多个独立子任务。
2. 分析子任务依赖关系，按依赖顺序编号；发现循环依赖时，优先通过合并或重新拆分解决。
3. 创建 `{{config.output_dir}}/BATCH-{batch-ts}-progress.md`，格式参考 `templates/batch-progress-template.md`。
4. 为每个子任务创建 `{ts}-userStory.md` 和 `{ts}-codingLog.md`。
5. 在 BATCH frontmatter 写入 `status=running`、`last-heartbeat`、`current-subtask`、`pid`。

**输出**：批次进度文档；子任务 userStory/codingLog；执行顺序。

## 3. Step 2：串行开发启动

**角色**：batch Orchestrator

### 3.1 共享分支

1. 根据批次需求判断类型：`feature` / `fix` / `refactor` / `docs`。
2. 生成分支名：`{type}/{batch-ts}-{batch-brief-desc}`。
3. 若共享分支已存在，复用；否则按 `../../fully-coding/rules/git-rules.md` 创建。
4. 将共享分支名写入 `BATCH-{batch-ts}-progress.md`。

### 3.2 子任务启动

按执行顺序逐个调用：

```text
/fully-coding --start-step 3 --task-id {tsN} --batch-subtask --git-branch {batch-branch-name}
```

子任务完成后更新 BATCH 进度文档中的状态、完成时间、产出摘要、`last-heartbeat` 和 `current-subtask`。

## 4. Step 3：进度监控

**角色**：batch Orchestrator

- 每完成一个子任务，更新 `BATCH-{batch-ts}-progress.md` 的子任务状态与执行日志。
- 若发现子任务 `{tsN}-blockLog.md` 存在且含「用户确认状态：待确认」，将 BATCH 状态改为 `blocked`，记录阻塞原因、阻塞步骤和 blockLog 路径。
- 阻塞时停止启动后续子任务，等待用户处理后通过 `/fully-coding-batch --auto-resume` 恢复。
- 可发送简短通知，但通知仅包含：子任务编号、阻塞步骤、阻塞原因、修复建议、blockLog 路径。

## 5. Step 4：完成汇总

**角色**：batch Orchestrator

批次完成时只汇总，不重复落盘已经由子任务 Step 8 完成的公共资源变更。

**动作**：
1. 收集所有子任务 codingLog Step 8 的「公共资源变更摘要」。
2. 汇总新增/复用错误码、公共枚举、知识库文档变更、各子任务耗时。
3. 检查错误码段位是否重复。若重复，按“前序保持、后序重分配”为默认策略修正，并重新编译验证；无法安全修正时登记阻塞。
4. 更新 BATCH 进度文档为 `status=completed`，写入完成时间和汇总产出。

## 6. 子任务阻塞处理

- 不得跳过阻塞子任务直接执行后续子任务。
- 阻塞期间共享分支保持不变，不得切换、删除或重建。
- 用户处理阻塞后，必须将对应 `{tsN}-blockLog.md` 更新为「用户确认状态：已确认」。
- 恢复时调用：

```text
/fully-coding --auto-resume --task-id {tsN} --batch-subtask --git-branch {batch-branch-name}
```

## 7. Git 与危险操作

- git 规则遵循 `../../fully-coding/rules/git-rules.md`。
- 创建本地分支可自动执行；push 必须先向用户确认。
- 不执行 force push、reset hard、删除分支、删除共享分支等破坏性操作，除非用户明确要求。
