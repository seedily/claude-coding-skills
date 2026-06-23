# Claude Code Skills 目录说明

> 本目录用于维护项目内的 Claude Code Skill。当前核心能力是 `fully-coding` 与 `fully-coding-batch`：前者面向单任务 9 步串行交付，后者面向多功能需求的批次拆分与串行调度。

## 简介

`.claude/skills` 是项目级 Skill 目录，用来放置可被 Claude Code 调用的工作流、角色提示词、规则文件、模板和使用说明。

当前目录主要围绕两个工具组织：

| Skill | 定位 | 适合场景 |
|---|---|---|
| `fully-coding` | 单任务全流程开发 Orchestrator | 1 个 feature、1 个 bug、需要需求、方案、开发、评审、测试、文档闭环的任务 |
| `fully-coding-batch` | 超长需求批次串行开发工具 | 多个 feature、多个 CRUD 模块、存在依赖关系的批量开发任务 |

核心原则是：**严格串行、文档驱动、规则优先、可恢复、可审计**。

## fully-coding 简介

`fully-coding` 是面向 Claude Code 的多角色串行流转工具，用自然语言需求驱动完整 9 步开发流程。

```text
用户输入
  → 需求检索
  → 需求生成
  → 开发范围
  → 开发方案
  → 开发实现
  → 代码评审
  → 测试用例
  → 文档更新
  → 自主进化
```

它的目标不是“快速生成一段代码”，而是把一次开发任务变成可追踪、可恢复、可评审、可测试、可同步知识库的交付流水线。

### 常用命令

```text
/fully-coding 帮我实现管理员角色的批量导入功能
/fully-coding 帮我设计收藏资讯功能 --plan-only
/fully-coding 修复登录排序问题 --quick-dev
/fully-coding --start-step 5 --task-id {ts}
/fully-coding --auto-resume
```

### 执行模式

| 模式 | 参数 | 执行范围 | 适合场景 |
|---|---|---|---|
| 标准模式 | 无 | Step 1-9 | 企业功能、复杂后端、需要完整审计和知识库闭环 |
| 方案模式 | `--plan-only` | Step 1-4 | 只生成需求、开发范围和开发方案，不修改代码 |
| 轻量开发模式 | `--quick-dev` | 最小需求记录 → Step 3 → Step 5 → Step 6 → Step 7 | 小 bug、明确小改动、无需完整知识库同步 |

### 主要产出

输出目录由 `fully-coding/project-config.md` 中的 `output_dir` 指定，默认是 `.dev-log/`。

| 文件 | 说明 |
|---|---|
| `{ts}-userStory.md` | 需求检索、需求生成、验收标准、假设和歧义 |
| `{ts}-codingLog.md` | 开发范围、方案、实现、评审、测试、文档同步和状态 |
| `{ts}-blockLog.md` | 仅阻塞时创建，记录阻塞原因和用户确认状态 |
| `{ts}-suggestion.md` | Step 9 生成的工具改进建议 |
| `AUTO-resume-{date}.log` | 自动续跑记录 |

## fully-coding-batch 简介

`fully-coding-batch` 用于处理超长、多功能需求。它先拆分子任务，再按顺序调用 `fully-coding` 执行每个子任务。

```text
多功能需求
  → 需求拆分
  → 共享分支
  → 子任务 1 Step 3-9
  → 子任务 2 Step 3-9
  → ...
  → 批次汇总
```

它适合一次需求包含多个功能、多个模块或多个服务改造，并且这些子任务之间存在先后依赖的情况。

### 常用命令

```text
/fully-coding-batch 实现用户管理、订单系统、支付集成三个模块
/fully-coding-batch --auto-resume
/fully-coding-batch --start-from 2
```

### 核心规则

| 规则 | 说明 |
|---|---|
| 严格串行 | 不并行启动多个子任务 |
| 共享分支 | 所有子任务共用同一个 git 分支 |
| 复用 fully-coding | 子任务复用 `fully-coding` Step 3-9 规则 |
| 阻塞暂停 | 任一子任务阻塞时，批次暂停，等待确认 |
| 统一进度 | 使用 `BATCH-{batch-ts}-progress.md` 记录批次状态 |

### 主要产出

| 文件 | 说明 |
|---|---|
| `BATCH-{batch-ts}-progress.md` | 批次总控进度、子任务清单、执行顺序和共享分支 |
| `{tsN}-userStory.md` | 子任务需求文档 |
| `{tsN}-codingLog.md` | 子任务开发日志 |
| `{tsN}-blockLog.md` | 子任务阻塞记录 |
| `{tsN}-suggestion.md` | 子任务自主进化建议 |

## 当前目录结构

```text
.claude/skills/
├── README.md
├── QUICKSTART-fully-coding.md
├── QUICKSTART-fully-coding-batch.md
├── fully-coding工具使用指南.md
├── fully-coding工具部署操作指南.md
├── fully-coding竞品对比报告.md
├── fully-coding/
│   ├── SKILL.md
│   ├── project-config.md
│   ├── agents/
│   ├── rules/
│   ├── templates/
│   └── hooks/
└── fully-coding-batch/
    ├── SKILL.md
    ├── agents/
    ├── rules/
    └── templates/
```

## 顶层文件说明

| 文件 | 说明 |
|---|---|
| `README.md` | 当前目录说明文档 |
| `QUICKSTART-fully-coding.md` | `fully-coding` 快速开始指南，包含首次运行、执行模式、流程图、进度查看和断点续跑 |
| `QUICKSTART-fully-coding-batch.md` | `fully-coding-batch` 快速开始指南，包含批次任务启动、流程图、进度查看和续跑说明 |
| `fully-coding工具使用指南.md` | 两个工具的综合使用指南，说明适用场景、参数、阻塞处理、续跑和最佳实践 |
| `fully-coding工具部署操作指南.md` | 部署与迁移说明，用于将 Skill 配置到新项目 |
| `fully-coding竞品对比报告.md` | 与 OpenSpec、Spec Kit、BMAD、SuperClaude 等工具的对比分析 |

## fully-coding 目录说明

`fully-coding/` 是单任务流水线的主体目录。

| 路径 | 说明 |
|---|---|
| `fully-coding/SKILL.md` | Skill 入口文件，定义触发说明、目标、启动命令、执行模式和权威文档索引 |
| `fully-coding/project-config.md` | `fully-coding` 与 `fully-coding-batch` 共享配置，包含知识库路径、输出目录、错误码、框架类型、心跳续跑等配置 |
| `fully-coding/agents/` | 角色提示词目录，定义 Orchestrator、需求分析师、架构师、开发者、Reviewer、DBA 等角色行为 |
| `fully-coding/rules/` | 规则目录，定义工作流契约、运行时加载、心跳续跑、代码规范、安全、持久层、错误处理、Git、评审清单等规则 |
| `fully-coding/templates/` | 文档模板目录，包含 userStory、codingLog、blockLog、suggestion、auto-resume log、功能实现文档模板等 |
| `fully-coding/hooks/` | 执行前检查或扩展钩子，目前用于预执行规则说明 |

### agents 文件说明

| 文件 | 角色 |
|---|---|
| `orchestrator.md` | 主调度角色，负责模式解析、步骤推进、状态更新、阻塞处理和 Step 9 自检 |
| `requirement-analyst.md` | 需求分析师，负责 Step 1 需求检索和 Step 2 需求生成 |
| `architect.md` | 架构师，负责 Step 3 开发范围、Step 4 开发方案、Step 8 文档更新 |
| `developer.md` | 开发者，负责 Step 5 开发实现和 Step 7 测试用例 |
| `reviewer.md` | 代码评审员，负责代码质量、安全、规范和实现一致性评审 |
| `dba.md` | DBA 评审角色，负责数据库、SQL、索引、事务、迁移、幂等和数据风险评审 |

### rules 文件说明

| 文件 | 说明 |
|---|---|
| `workflow-rules.md` | 核心工作流规则，定义 9 步契约、模式差异、auto-fix、block、文档同步和退出条件 |
| `runtime-load-map.md` | 运行时加载图，定义每个步骤最小读取哪些角色、规则和模板，用于节省 token |
| `heartbeat-resume-rules.md` | 心跳、状态机、stalled/resuming 接管、planned 状态和 auto-resume 规则 |
| `review-checklist.md` | Step 6 轻量评审清单，覆盖 DBA 与 Reviewer 的基础检查维度 |
| `implementation-checklist-rules.md` | 开发实现检查清单，约束 Step 5 的实现完整性 |
| `backend-ddd-layer-rules.md` | 后端 DDD 分层、模块边界和依赖方向规则 |
| `backend-code-standard-rules.md` | 后端编码规范 |
| `frontend-code-standard-rules.md` | 前端编码规范 |
| `persistence-sql-cache-rules.md` | 持久层、SQL、缓存、索引、事务、幂等和数据访问规则 |
| `security-rules.md` | 安全规则，覆盖输入校验、敏感数据、权限、注入风险等 |
| `error-handling-rules.md` | 异常处理、错误码和响应包装规则 |
| `git-rules.md` | Git 分支、提交、推送和危险操作确认规则 |

### templates 文件说明

| 文件 | 说明 |
|---|---|
| `user-story-template.md` | `{ts}-userStory.md` 模板，用于记录需求检索、需求生成、验收标准和歧义 |
| `coding-log-template.md` | `{ts}-codingLog.md` 模板，用于记录 Step 3-9 的开发过程、状态、评审、测试和文档同步 |
| `block-log-template.md` | `{ts}-blockLog.md` 模板，仅在必须暂停等待用户确认时生成 |
| `suggestion-template.md` | `{ts}-suggestion.md` 模板，用于 Step 9 自主进化建议 |
| `auto-resume-log-template.md` | 自动续跑日志模板 |
| `feature_impl_template.md` | 功能实现文档模板，用于新建或补充 `6.功能实现-*.md` 类知识库文档 |

## fully-coding-batch 目录说明

`fully-coding-batch/` 是批量串行调度工具目录。它不重写单个子任务的开发规则，而是负责拆分、排序、共享分支、监控和批次汇总。

| 路径 | 说明 |
|---|---|
| `fully-coding-batch/SKILL.md` | Skill 入口文件，定义 batch 目标、启动命令、执行流程、核心红线和权威文档索引 |
| `fully-coding-batch/agents/` | batch 调度角色目录 |
| `fully-coding-batch/rules/` | batch 工作流、续跑和运行时加载规则目录 |
| `fully-coding-batch/templates/` | batch 进度文档模板目录 |

### batch agents 文件说明

| 文件 | 说明 |
|---|---|
| `batch-orchestrator.md` | 批次主调度角色，负责需求拆分、共享分支、子任务串行启动、进度监控和完成汇总 |

### batch rules 文件说明

| 文件 | 说明 |
|---|---|
| `batch-workflow-rules.md` | 批次流程规则，定义需求拆分、串行执行、共享分支、阻塞处理和批次汇总 |
| `batch-resume-rules.md` | 批次续跑规则，定义 `--auto-resume`、`--start-from N` 和阻塞恢复逻辑 |
| `batch-runtime-load-map.md` | batch 模式运行时加载图，定义不同场景下需要读取的最小上下文 |

### batch templates 文件说明

| 文件 | 说明 |
|---|---|
| `batch-progress-template.md` | `BATCH-{batch-ts}-progress.md` 模板，用于记录子任务清单、执行顺序、状态、共享分支和汇总结果 |

## 配置说明

两个 Skill 共享 `fully-coding/project-config.md`。部署到新项目时，通常只需要调整该文件。

常用配置包括：

| 配置 | 说明 |
|---|---|
| `knowledge_dir` | 项目知识库目录，默认 `_knowledge` |
| `output_dir` | 开发日志输出目录，默认 `.dev-log` |
| `requirement_doc` | 需求文档匹配规则 |
| `error_code_doc` | 错误码文档 |
| `tech_design_glob` | 技术方案文档匹配规则 |
| `service_design_glob` | 服务详细设计文档匹配规则 |
| `feature_impl_glob` | 功能实现文档匹配规则 |
| `feature_impl_overview` | 功能实现概览文档 |
| `feature_impl_menu_iteration` | 菜单功能迭代文档 |
| `feature_impl_template` | 功能实现文档模板 |
| `heartbeat_interval_seconds` | 心跳写入间隔 |
| `heartbeat_timeout_seconds` | 心跳超时阈值 |
| `framework_type` | 项目后端框架类型 |
| `error_code_format` | 错误码格式 |

维护原则：`project-config.md` 是唯一配置来源。新增 `{{config.*}}` 占位符时，必须同步在该文件声明对应 key。

## 推荐阅读顺序

首次了解建议按下面顺序阅读：

```text
README.md
  → QUICKSTART-fully-coding.md
  → fully-coding/SKILL.md
  → fully-coding工具使用指南.md
  → fully-coding/project-config.md
  → fully-coding/rules/workflow-rules.md
  → fully-coding/rules/runtime-load-map.md
  → QUICKSTART-fully-coding-batch.md
  → fully-coding-batch/SKILL.md
```

如果只是使用工具，优先看 Quickstart 和使用指南。如果需要维护或改造 Skill，再看 `agents/`、`rules/`、`templates/`。

## 初始化、首次使用注意

1. 本系列工具，是基于 SDD 思想实现，要求有足够的知识文档的沉淀，尤其包括：需求文档、整体架构。
2. 请重点理解 `fully-coding/project-config.md` 文档的配置项，并结合自身情况修改配置值。强烈建议使用 claude code 做一轮本地化调整、适配。

## 维护建议

1. 修改流程规则时，优先修改 `rules/workflow-rules.md`，再同步 Quickstart 和使用指南。
2. 修改执行模式或参数时，需要同步 `fully-coding/SKILL.md`、`QUICKSTART-fully-coding.md`、`fully-coding工具使用指南.md`。
3. 修改 batch 逻辑时，需要同步 `fully-coding-batch/SKILL.md`、`QUICKSTART-fully-coding-batch.md` 和 batch rules。
4. 新增 `{{config.*}}` 占位符时，必须更新 `fully-coding/project-config.md`。
5. 新增模板时，要说明产出文件名、使用步骤和是否参与 auto-resume。
6. 保持单任务与 batch 的边界：`fully-coding` 负责单任务交付，`fully-coding-batch` 负责多任务拆分和串行调度。
