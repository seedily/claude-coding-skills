---
name: pre-execution-hook
description: fully-coding 每次步骤开始前的自检钩子。检查输入文档、模板、规范引用、输出目录，确保串行条件满足。
---

# Pre-Execution Hook（步骤前自检）

## 触发时机
每次进入新步骤前，Orchestrator 必须执行本 Hook 中的检查项。

## 自检清单

### A. 任务与心跳检查

> 详细规则参见 `rules/heartbeat-resume-rules.md`。

- [ ] 当前任务 `{{config.output_dir}}/{ts}-codingLog.md` 已存在或已创建
- [ ] 当前任务的 `codingLog.md` 已存在，且 frontmatter 中 `status` 为 `running`（续跑时已完成接管）
- [ ] 若为续跑：已按 `rules/heartbeat-resume-rules.md` 完成心跳校验（`last-heartbeat` 未超时 / 已超时）
- [ ] 同一任务无其他 Claude 实例在并发执行（通过 `pid` 进程存活、`last-heartbeat` 和 `heartbeat_timeout_seconds` 判断）

### B. 文档就绪检查
- [ ] 若当前步骤 > 1，当前任务的 `userStory.md` 已存在
- [ ] 若当前步骤 > 3，当前任务的 `codingLog.md` 已存在
- [ ] 上一步骤在对应文档中的章节已填写且非空

### C. 输出目录检查
- [ ] `{{config.output_dir}}/` 目录存在，不存在则创建
- [ ] 时间戳已确定：`{yyyyMMdd-HHmmss}` 格式
- [ ] 输出文件路径已确定：当前任务的 `{ts}-userStory.md` 和 `{ts}-codingLog.md`

### D. 角色 Prompt 检查
- [ ] 已读取 `agents/orchestrator.md`（调度中枢）
- [ ] 已读取当前步骤对应角色的 Prompt 文件：
  - 步骤 1/2 → `agents/requirement-analyst.md`
  - 步骤 3/4/8 → `agents/architect.md`
  - 步骤 5/7 → `agents/developer.md`
  - 步骤 6 数据库评审 → `agents/dba.md`
  - 步骤 6 代码评审 → `agents/reviewer.md`

### E. 规范引用检查
- [ ] 已读取 `rules/runtime-load-map.md`，并按当前步骤加载必读文件
- [ ] 当前步骤的 `workflow-rules.md` 步骤契约已检查
- [ ] Step 6 默认读取 `rules/review-checklist.md`；仅在命中具体风险时读取对应完整规则文件
- [ ] Step 3/4/5/7/8 仅在命中 DDD、编码、持久层、安全、异常、Git、错误码等具体场景时，按 `rules/runtime-load-map.md` 的「按需读取」列加载对应规则或知识库文档
- [ ] 若当前步骤为 3/4/5/6/8，已按需读取 `{{config.knowledge_dir}}/{{config.error_code_doc}}`

### F. 并行约束检查
- [ ] 当前未运行任何 Agent Team
- [ ] 当前未 spawn 任何子 Agent
- [ ] 本步骤不会触发 TeamCreate

## 心跳更新

自检通过后、进入角色执行前，必须更新当前任务 `codingLog.md` 的 frontmatter：
- `last-heartbeat`：当前时间（`yyyyMMdd-HHmmss`）
- `current-step`：当前步骤编号
- `current-role`：当前角色名称
- `pid`：当前进程 PID（保持与当前进程一致）

## 失败处理

- 任一检查项失败，立即停止当前步骤
- 向用户报告失败项和补救方案
- 不得在未修复的情况下继续流程

## 快速通过话术

自检全部通过后，用一句话开场：

> 步骤 N（角色名）开始，任务 `{ts}`，心跳校验通过，输入文档已就绪，输出目录已确认，规范已引用，开始执行。
