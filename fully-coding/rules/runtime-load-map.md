---
name: runtime-load-map
description: fully-coding 每步运行时最小上下文加载索引，用于减少默认读取文档和 token 消耗。
---

# Runtime Load Map

> 原则：先读必读文件；只有当前步骤需要、或检查发现问题时，才读取按需文件。流程规则以 `workflow-rules.md` 为准。

## 执行模式加载

| 模式 | 参数 | 加载步骤 |
|---|---|---|
| 标准模式 | 无 | 初始化/恢复 + Step 1-9 |
| 方案模式 | `--plan-only` | 初始化/恢复 + Step 1-4，Step 4 后停止 |
| 轻量开发模式 | `--quick-dev` | 初始化/恢复、最小需求记录、Step 3、Step 5、Step 6、Step 7 |

> "初始化/恢复" = Orchestrator 加载 project-config、runtime-load-map、workflow-rules 与心跳规则等运行时上下文的阶段，**非交付步骤，不计入 `workflow-rules.md §3.0` 的执行范围**；各模式权威执行范围以 `workflow-rules.md §3.0` 为准。

> 本表是**加载范围**视角（决定每步读哪些上下文）。模式的**执行范围、完成状态、互斥约束**以 `workflow-rules.md §3.0` 为唯一权威源；若两处 step 列表出现不一致，以 `workflow-rules.md §3.0` 为准并回头修正本表。

| Step | 角色 | 必读 | 按需读取 |
|---|---|---|---|
| 初始化/恢复 | Orchestrator | `project-config.md`; `agents/orchestrator.md`; `rules/workflow-rules.md` §1/§3.1; `rules/heartbeat-resume-rules.md` | `templates/auto-resume-log-template.md`; `{ts}-blockLog.md` |
| 1 需求检索 | 需求分析师 | `agents/requirement-analyst.md`; `rules/workflow-rules.md` §3.2; `templates/user-story-template.md` | `{{config.knowledge_dir}}/{{config.requirement_doc}}` |
| 2 需求生成 | 需求分析师 | `agents/requirement-analyst.md`; `rules/workflow-rules.md` §3.3; `{ts}-userStory.md` | 需求终稿相关章节 |
| 3 开发范围 | 架构师 | `agents/architect.md` Step 3; `rules/workflow-rules.md` §3.4; `{ts}-userStory.md`; `templates/coding-log-template.md` | `{{config.tech_design_glob}}`; `{{config.service_design_glob}}`; `{{config.error_code_doc}}`; `rules/git-rules.md` |
| 4 开发方案 | 架构师 | `agents/architect.md` Step 4; `rules/workflow-rules.md` §3.5/§7; `{{config.feature_impl_overview}}`; `{{config.feature_impl_template}}`; `{{config.error_code_doc}}`; `rules/backend-code-standard-rules.md`; `rules/frontend-code-standard-rules.md` | `rules/backend-ddd-layer-rules.md`; `rules/persistence-sql-cache-rules.md`; `rules/security-rules.md`; `rules/error-handling-rules.md`; 概览命中的既有 `{{config.feature_impl_glob}}`；仅缺失或核心大功能时新建功能实现文档 |
| 5 开发实现 | 开发者 | `agents/developer.md`; `rules/workflow-rules.md` §3.6; `rules/implementation-checklist-rules.md`; `codingLog` Step 4; `rules/backend-code-standard-rules.md`; `rules/frontend-code-standard-rules.md` | `rules/backend-ddd-layer-rules.md`; `rules/persistence-sql-cache-rules.md`; `rules/security-rules.md`; `rules/error-handling-rules.md`; `{{config.error_code_doc}}`; 现有代码 |
| 6 代码评审 | DBA + Reviewer | `agents/dba.md`; `agents/reviewer.md`; `rules/review-checklist.md`; `rules/workflow-rules.md` §3.7/§6; 本次 diff; `codingLog` Step 5; `rules/backend-code-standard-rules.md`; `rules/frontend-code-standard-rules.md` | 发现问题时读取 `rules/backend-ddd-layer-rules.md`/`persistence-sql-cache-rules.md`/`security-rules.md`/`error-handling-rules.md` 对应完整规则；`{{config.error_code_doc}}`; `{{config.feature_impl_glob}}` |
| 7 测试用例 | 开发者 | `agents/developer.md`; `rules/workflow-rules.md` §3.8; `codingLog` Step 6 | 现有测试；测试规范；失败日志 |
| 8 文档更新 | 架构师 | `agents/architect.md` Step 8; `rules/workflow-rules.md` §3.9/§7; `codingLog` Step 3-7 | 触发矩阵涉及的知识库文档；`rules/git-rules.md` |
| 9 自主进化 | Orchestrator | `agents/orchestrator.md` Step 9; `rules/workflow-rules.md` §3.10; `templates/suggestion-template.md`; `codingLog`; `userStory` | 需要核对的知识库文档；`{{config.error_code_doc}}` |

## 轻量加载规则

- **命名与接口契约规则为硬性必读**（历史高频违反项，不适用懒加载）：`rules/backend-code-standard-rules.md` 与 `rules/frontend-code-standard-rules.md` 在 Step 4/5/6 必须读取。禁止项：前端 `get`/`list` 命名、RESTful URL；后端自定义响应/请求包装类、`@JsonNaming(SnakeCaseStrategy)`/`@JsonProperty(snake_case)`、`new ObjectMapper()` 默认 strict。
- 不因”可能会用到”而提前读取完整规则文件。
- Step 6 默认使用 `review-checklist.md` 覆盖评审维度；只有定位到具体风险时才读取对应完整规则。
- Step 8 只读取触发矩阵命中的知识库文档。
- `../../QUICKSTART-fully-coding.md`、FAQ、示例文档只供人类阅读，运行时不默认读取。

## batch 子任务额外必读（防公共文档覆盖）

> 触发条件：codingLog frontmatter `batch-subtask: true`（由 `--batch-subtask` 写入）。非 batch 单任务不适用本节，仍按上方"按需读取"。

batch 多个子任务串行写同一批公共知识库文档（功能实现概览、菜单功能迭代表）。若后续子任务只"按需"读取，可能基于陈旧版本**整表覆盖前序子任务刚写入的行**。因此 batch 子任务在以下步骤把公共文档**升级为必读，且必须读磁盘最新版本**：

| Step | 额外必读（batch 子任务） | 原因 |
|---|---|---|
| Step 4 | `{{config.feature_impl_overview}}`（主表已必读，此处确认）；若涉菜单/页面/路由/权限入口，追加 `{{config.feature_impl_menu_iteration}}` | 方案判定与"是否新建功能文档"必须基于前序子任务已登记的最新索引与入口 |
| Step 8 | `{{config.feature_impl_overview}}`；若本次或任一前序子任务涉菜单变更，追加 `{{config.feature_impl_menu_iteration}}` | 写入前必须读到前序子任务 Step 8 刚落盘的最新行，**按行 merge，禁止整表重写** |

写入约定：概览表与菜单迭代表均以「功能名称 / 路由入口」为主键做行级 upsert，不得用本地缓存的旧表整体覆盖。

> 注：fully-coding 标准 Step 4/8 默认把 `feature_impl_menu_iteration` 列为按需；batch 串行场景下，若前序子任务已写过菜单迭代表，必须前置读到最新版本，故此处对 batch 子任务强制升级为必读（fully-coding Step 8 默认按需，batch 串行场景必须前置读到前序行）。
