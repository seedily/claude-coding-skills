---
name: batch-resume-rules
description: fully-coding-batch 断点续跑规则。仅定义批次接管、子任务恢复和 --start-from 逻辑，心跳状态机复用 fully-coding。
---

# Batch Resume Rules

> 心跳字段、超时判断、`running/resuming/blocked/stalled/failed/completed` 状态语义复用 `../../fully-coding/rules/heartbeat-resume-rules.md`。本文件只补充 batch 特有接管逻辑。

## 1. `--auto-resume`

执行：

```text
/fully-coding-batch --auto-resume
```

### 接管流程

1. 扫描 `{{config.output_dir}}/BATCH-*-progress.md`，选择最新批次进度文档。
2. 读取 BATCH frontmatter：校验 `type == batch-progress`（防同名前缀的非 batch 文件误匹配），再读 `status`、`last-heartbeat`、`current-subtask`、`pid`。
3. 按 `../../fully-coding/rules/heartbeat-resume-rules.md` 判断当前 batch Orchestrator 是否仍活跃：
   - 活跃：跳过本轮 resume。
   - 超时或失败：允许接管。
   - `blocked`：检查当前子任务 blockLog。
4. 检查是否存在其他活跃的 batch 或非 batch fully-coding 任务；若存在且心跳未超时，跳过本轮 resume。
5. 将 BATCH frontmatter 更新为 `status=resuming`、当前 `last-heartbeat`、当前 `pid`；再次读取确认未被抢占后改为 `running`。
6. 定位第一个状态非「已完成」的子任务。
7. 读取共享分支 `{batch-branch-name}`。
8. 恢复当前子任务：
   - 若 `{tsN}-codingLog.md` 存在且未完成：
     ```text
     /fully-coding --auto-resume --task-id {tsN} --batch-subtask --git-branch {batch-branch-name}
     ```
   - 若 `{tsN}-codingLog.md` 不存在：
     ```text
     /fully-coding --start-step 3 --task-id {tsN} --batch-subtask --git-branch {batch-branch-name}
     ```
9. 当前子任务完成后，继续按执行顺序启动后续子任务。

## 2. 阻塞恢复

当当前子任务状态为「阻塞」：

1. 检查 `{tsN}-blockLog.md`。
2. 若仍包含「用户确认状态：待确认」，保持 BATCH `status=blocked`，报告阻塞步骤、原因、建议和 blockLog 路径后停止。
3. 若已标记「已确认」，按 `--auto-resume` 的子任务恢复逻辑继续执行。

## 3. `--start-from N`

执行：

```text
/fully-coding-batch --start-from N
```

适用于用户明确知道从第 N 个子任务继续的场景。

### 执行流程

1. 读取最新或用户指定的 `BATCH-{batch-ts}-progress.md`。
2. 提取共享分支 `{batch-branch-name}`。
3. 确认目标代码工程处于共享分支；切换失败时记录到 BATCH 进度文档并暂停。
4. 将 BATCH frontmatter 更新为 `status=running`、当前 `last-heartbeat`、第 N 个 `current-subtask`、当前 `pid`。
5. 从第 N 个子任务开始，逐个调用：
   ```text
   /fully-coding --start-step 3 --task-id {tsN} --batch-subtask --git-branch {batch-branch-name}
   ```

## 4. 安全边界

- `--start-from N` 不自动验证前序子任务的业务完整性；使用前需确保前序子任务和公共资源已正确落盘。
- 不允许跳过仍处于待确认阻塞状态的子任务。
- 不允许重建、删除或强制覆盖共享分支。
