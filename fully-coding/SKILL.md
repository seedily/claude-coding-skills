---
name: fully-coding
description: 从自然语言需求到代码交付的全自动开发流水线。适用于"实现某功能 / 开发某模块 / 从需求到交付 / 需求分析到代码实现"等场景。多角色串行 9 步闭环（需求→方案→实现→评审→测试→文档→自检），支持 --plan-only 方案模式、--quick-dev 轻量修复、断点续跑
---

# Fully Coding

> 新用户先看 [`QUICKSTART-fully-coding.md`](../QUICKSTART-fully-coding.md)。详细规则以 `rules/workflow-rules.md` 为唯一权威源。

## 目标

`fully-coding` 将用户自然语言需求串行为 9 步文档驱动开发流程：

```text
用户输入 → 需求检索 → 需求生成 → 开发范围 → 开发方案 → 开发实现 → 代码评审 → 测试用例 → 文档更新 → 自主进化
```

## 启动命令

```text
/fully-coding 帮我实现管理员角色的批量导入功能，支持 Excel 上传和模板下载
```

常用参数：

| 参数 | 说明 |
|---|---|
| `--plan-only` | 只执行 Step 1-4，生成需求、开发范围和开发方案，不修改代码、不进入评审测试文档闭环 |
| `--quick-dev` | 轻量开发模式，执行最小需求记录 → Step 3 → Step 5 → Step 6 → Step 7，用于小 bug 或明确小改动 |
| `--start-step N` | 从第 N 步开始执行 |
| `--task-id {ts}` | 指定任务时间戳，用于续跑、重跑或 batch 子任务定位 |
| `--batch-subtask` | 标记为批次子任务 |
| `--git-branch <name>` | 指定复用分支，batch 模式常用 |
| `--status` | 查看当前任务进度 |
| `--list` | 列出未完成任务 |
| `--new-task` | 忽略恢复扫描，强制启动新任务 |
| `--auto-resume` | 自动续跑最新可接管任务 |

## 运行必读顺序

1. `project-config.md`：解析所有 `{{config.*}}`。
2. `agents/orchestrator.md`：执行调度。
3. `rules/runtime-load-map.md`：按步骤懒加载角色、规则和模板。
4. `rules/workflow-rules.md`：流程契约、红线、阻塞、auto-fix、文档同步的权威规则。

## 执行模式

| 模式 | 触发参数 | 执行范围 | 适用场景 |
|---|---|---|---|
| 标准模式 | 无 | Step 1-9 | 企业功能、复杂后端、需要完整审计和文档闭环 |
| 方案模式 | `--plan-only` | Step 1-4 | 只需要生成需求、开发范围和开发方案，先审方案再开发 |
| 轻量开发模式 | `--quick-dev` | 最小需求记录 → Step 3 → Step 5 → Step 6 → Step 7 | 小 bug、明确小改动、无需完整知识库同步的快速修复 |

> 各模式的**执行范围、完成状态与互斥约束**以 `rules/workflow-rules.md §3.0` 为唯一权威源；本表仅作入口概览，勿在此维护权威定义。

## 9 步流程

| Step | 角色 | 输出 | 关键动作 |
|---|---|---|---|
| 1 需求检索 | 需求分析师 | `{ts}-userStory.md` | 检索需求终稿并记录命中情况 |
| 2 需求生成 | 需求分析师 | `{ts}-userStory.md` | 写入新增/变更需求、AC、假设和歧义 |
| 3 开发范围 | 架构师 | `{ts}-codingLog.md` | 定位服务、模块、文件清单 |
| 4 开发方案 | 架构师 | `codingLog` + 功能实现文档 | 先检索 `{{config.feature_impl_overview}}`，优先复用/追加既有文档；仅缺失或核心大功能才按命名规范新增 `{{config.feature_impl_glob}}` |
| 5 开发实现 | 开发者 | 代码 + `codingLog` | 按 DDD、编码、安全、错误码、幂等规则实现 |
| 6 代码评审 | DBA + 代码评审员 | `codingLog` | 使用 `rules/review-checklist.md` 轻量评审，必要时展开完整规则；auto-fix 最多 3 轮 |
| 7 测试用例 | 开发者 | 测试 + `codingLog` | 补充/运行测试，失败最多自动修复 3 轮 |
| 8 文档更新 | 架构师 | 知识库 + `codingLog` | 按 `workflow-rules.md §7.4` 同步服务设计、功能实现、概览、菜单迭代、错误码 |
| 9 自主进化 | Orchestrator | `{ts}-suggestion.md` | 文档一致性自检，记录工具改进建议，标记 completed |

## 核心红线

- 单 Claude 实例串行执行；禁止 `TeamCreate`、禁止 spawn 多 Agent、禁止 `run_in_background`。
- 上一步输出文档落盘后，才能进入下一步。
- Step 6 的 DBA 与 Reviewer 必须在同一响应中合并评审，不得拆成两个 Agent。
- 除真正阻塞、破坏性/不可逆操作、用户明确打断外，不为状态汇报暂停。
- Git push、不可逆 DDL、删除已有功能、CI/CD、密钥相关操作必须先确认。
- 不调用 Apifox、webhook、TAPD；仅允许按规则执行 git-sync。

## 文档与目录

输出目录由 `{{config.output_dir}}` 指定，默认 `.dev-log/`：

```text
{ts}-userStory.md
{ts}-codingLog.md
{ts}-blockLog.md      # 仅阻塞时创建
{ts}-suggestion.md
AUTO-resume-{date}.log
```

知识库路径由 `project-config.md` 统一配置：

```text
{{config.knowledge_dir}}/{{config.requirement_doc}}
{{config.knowledge_dir}}/{{config.tech_design_glob}}
{{config.knowledge_dir}}/{{config.service_design_glob}}
{{config.knowledge_dir}}/{{config.error_code_doc}}
{{config.feature_impl_template}}
{{config.knowledge_dir}}/{{config.feature_impl_overview}}
{{config.knowledge_dir}}/{{config.feature_impl_menu_iteration}}
{{config.knowledge_dir}}/{{config.feature_impl_glob}}
```

## 权威文档索引

| 文件 | 职责 |
|---|---|
| `project-config.md` | 配置与路径占位符 |
| `agents/orchestrator.md` | 串行调度、恢复、Step 9 自检 |
| `rules/runtime-load-map.md` | 每步最小加载上下文 |
| `rules/workflow-rules.md` | 流程红线、步骤契约、阻塞、auto-fix、文档同步 |
| `rules/heartbeat-resume-rules.md` | 心跳、状态机、自动续跑 |
| `rules/review-checklist.md` | Step 6 轻量评审清单 |
| `rules/git-rules.md` | Git 分支、commit、push 规则 |
| `agents/*.md` | 各角色执行提示 |
| `templates/*.md` | userStory / codingLog / blockLog / suggestion 模板 |

## Batch 模式

超长需求（≥3 个 feature）使用 `fully-coding-batch` 串行拆分子任务；子任务通过 `/fully-coding --start-step 3 --task-id {ts} --batch-subtask --git-branch {branch}` 进入本流程。
