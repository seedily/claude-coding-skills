---
name: runtime-load-map
description: fully-coding 每步运行时最小上下文加载索引，用于减少默认读取文档和 token 消耗。
---

# Runtime Load Map

> 原则：先读必读文件；只有当前步骤需要、或检查发现问题时，才读取按需文件。流程规则以 `workflow-rules.md` 为准。

## 执行模式加载

| 模式 | 参数 | 加载步骤 |
|---|---|---|
| 标准模式 | 无 | Step 0-9 |
| 方案模式 | `--plan-only` | Step 0-4，Step 4 后停止 |
| 轻量开发模式 | `--quick-dev` | Step 0、最小需求记录、Step 3、Step 5、Step 6、Step 7 |

| Step | 角色 | 必读 | 按需读取 |
|---|---|---|---|
| 0 初始化/恢复 | Orchestrator | `project-config.md`; `agents/orchestrator.md`; `rules/workflow-rules.md` §1/§3.1; `rules/heartbeat-resume-rules.md` | `templates/auto-resume-log-template.md`; `{ts}-blockLog.md` |
| 1 需求检索 | 需求分析师 | `agents/requirement-analyst.md`; `rules/workflow-rules.md` §3.2; `templates/user-story-template.md` | `{{config.knowledge_dir}}/{{config.requirement_doc}}` |
| 2 需求生成 | 需求分析师 | `agents/requirement-analyst.md`; `rules/workflow-rules.md` §3.3; `{ts}-userStory.md` | 需求终稿相关章节 |
| 3 开发范围 | 架构师 | `agents/architect.md` Step 3; `rules/workflow-rules.md` §3.4; `{ts}-userStory.md`; `templates/coding-log-template.md` | `{{config.tech_design_glob}}`; `{{config.service_design_glob}}`; `{{config.error_code_doc}}`; `rules/git-rules.md` |
| 4 开发方案 | 架构师 | `agents/architect.md` Step 4; `rules/workflow-rules.md` §3.5/§7; `{{config.feature_impl_overview}}`; `{{config.feature_impl_template}}`; `{{config.error_code_doc}}` | DDD/编码/持久层/安全/异常规则；概览命中的既有 `{{config.feature_impl_glob}}`；仅缺失或核心大功能时新建功能实现文档 |
| 5 开发实现 | 开发者 | `agents/developer.md`; `rules/workflow-rules.md` §3.6; `rules/implementation-checklist-rules.md`; `codingLog` Step 4 | DDD/编码/持久层/安全/异常规则；错误码文档；现有代码 |
| 6 代码评审 | DBA + Reviewer | `agents/dba.md`; `agents/reviewer.md`; `rules/review-checklist.md`; `rules/workflow-rules.md` §3.7/§6; 本次 diff; `codingLog` Step 5 | 发现问题时读取对应完整规则文件；`{{config.error_code_doc}}`; `{{config.feature_impl_glob}}` |
| 7 测试用例 | 开发者 | `agents/developer.md`; `rules/workflow-rules.md` §3.8; `codingLog` Step 6 | 现有测试；测试规范；失败日志 |
| 8 文档更新 | 架构师 | `agents/architect.md` Step 8; `rules/workflow-rules.md` §3.9/§7; `codingLog` Step 3-7 | 触发矩阵涉及的知识库文档；`rules/git-rules.md` |
| 9 自主进化 | Orchestrator | `agents/orchestrator.md` Step 9; `rules/workflow-rules.md` §3.10; `templates/suggestion-template.md`; `codingLog`; `userStory` | 需要核对的知识库文档；`{{config.error_code_doc}}` |

## 轻量加载规则

- 不因“可能会用到”而提前读取完整规则文件。
- Step 6 默认使用 `review-checklist.md` 覆盖评审维度；只有定位到具体风险时才读取对应完整规则。
- Step 8 只读取触发矩阵命中的知识库文档。
- `../../QUICKSTART-fully-coding.md`、FAQ、示例文档只供人类阅读，运行时不默认读取。
