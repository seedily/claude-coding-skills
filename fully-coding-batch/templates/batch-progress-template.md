---
name: batch-progress-template
description: fully-coding-batch 批次进度文档模板。
---

---
type: batch-progress
batch-timestamp: {yyyyMMdd-HHmmss}
status: running
last-heartbeat: {yyyyMMdd-HHmmss}
current-subtask: {tsN}
pid: {process-id}
---

# 批量开发进度

> 批次时间戳：{yyyyMMdd-HHmmss}
> 任务描述：{用户原始需求}

## 子任务清单

### 子任务 1：{功能名称}
- 时间戳：{ts1}
- 依赖关系：无依赖
- 文档路径：`{{config.output_dir}}/{ts1}-userStory.md`
- 开发日志：`{{config.output_dir}}/{ts1}-codingLog.md`
- 当前步骤：待启动 / 步骤 N（{角色名}）
- 状态：待启动 / 进行中 / 阻塞 / 已完成
- 完成时间：
- 产出摘要：

### 子任务 2：{功能名称}
- 时间戳：{ts2}
- 依赖关系：依赖子任务 1 / 无依赖
- 文档路径：`{{config.output_dir}}/{ts2}-userStory.md`
- 开发日志：`{{config.output_dir}}/{ts2}-codingLog.md`
- 当前步骤：待启动
- 状态：待启动
- 完成时间：
- 产出摘要：

## 执行顺序

- 子任务 1 → 子任务 2 → 子任务 N

## 执行日志

- {yyyyMMdd-HHmmss}：创建批次任务

## Git 分支信息

- 共享分支名：`{type}/{batch-ts}-{batch-brief-desc}`
- 基于分支：`main` / `master`
- 子任务 commit 数：

## 阻塞记录

- 当前阻塞子任务：无
- blockLog 路径：无
- 阻塞原因：无

## 汇总产出

- 新增文件：
- 修改文件：
- 新增错误码：
- 公共资源变更摘要：
- 总耗时：

## 公共资源状态快照（批次完成后填写）

- `{{config.knowledge_dir}}/{{config.error_code_doc}}`：
- 公共枚举变更：

## 任务状态

- 当前状态：running / blocked / completed
- 完成时间：
