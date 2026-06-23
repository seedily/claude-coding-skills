# fully-coding 竞品对比报告

> 本报告用于评估 `fully-coding` 与 OpenSpec、GitHub Spec Kit、BMAD-METHOD、SuperClaude Framework 等开发流程工具的差异，并参考 Claude Code 官方 `/plan`、`/code-review`、Anthropic `webapp-testing`、`skill-creator` 等局部能力，辅助判断 `fully-coding` 的产品定位、优化方向和后续评测方案。
>
> **报告性质说明**：本报告是面向 `fully-coding` 后续演进的内部竞品分析与产品规划参考，不是严格实测 benchmark 报告。除明确标注为“已实现 / 已验证”的内容外，性能、准确性和综合评分均为基于公开资料、工具定位和项目经验的定性判断，后续仍需通过统一测试任务集验证。

---

## 一、报告范围、证据等级与适用边界

### 1.1 核心对比对象

| 工具 | 类型 | 纳入原因 | 证据等级 |
|---|---|---|---|
| OpenSpec | Spec-driven development Skill / workflow | 与 `fully-coding` 的“需求 → 计划 → 开发推进”高度重叠 | 低 |
| GitHub Spec Kit | Spec-driven development toolkit | 支持 Claude Code，流程覆盖 spec / plan / tasks / implement | 中 |
| BMAD-METHOD | AI Agile / 多角色开发框架 | 与 `fully-coding` 的角色流、需求到开发流程接近 | 中 |
| SuperClaude Framework | Claude Code 增强框架 | 覆盖计划、实现、测试、文档、研究等多个开发环节 | 中 |
| fully-coding | 项目级 9 步串行交付 Skill | 当前自研对象，作为主评估基准 | 高 |

### 1.2 补充基准对象

| 工具 | 类型 | 用途 | 说明 |
|---|---|---|---|
| Claude Code `/plan` | 官方规划能力 | 对标 Step 3 开发范围、Step 4 开发方案 | 局部能力基准，不是完整 workflow |
| Claude Code `/code-review` | 官方代码评审能力 | 对标 Step 6 代码评审 | 局部能力基准，不覆盖需求、测试、文档闭环 |
| Anthropic `webapp-testing` | 官方 Web 测试 Skill | 对标 Step 7 测试用例 | 主要用于 Web UI 验证 |
| Anthropic `skill-creator` | 官方 Skill 创建与评测 Skill | 用于构建 benchmark 和迭代评估 | 不是业务开发竞品 |

### 1.3 证据等级定义

| 证据等级 | 含义 | 适用说明 |
|---|---|---|
| 高 | 已在 `fully-coding` 当前规则、模板或流程中实现，或公开文档明确支持 | 可作为近期产品决策依据 |
| 中 | 基于公开定位、常见使用方式和工具设计特征判断 | 可作为方向参考，但需结合实际版本复核 |
| 低 | 基于有限资料或能力模型推断，尚未完成同任务实测 | 只能作为待验证假设 |

### 1.4 适用边界

- 本报告适合用于 `fully-coding` 的定位讨论、路线图规划、优化优先级排序。
- 本报告不适合直接作为对外宣传材料、采购选型最终依据或严格性能结论。
- OpenSpec 相关判断置信度相对较低，需结合实际仓库版本和同任务执行结果进一步验证。
- SuperClaude 相关判断依赖具体命令、Agent 配置和用户组合方式，不能视为单一固定工作流能力。
- 官方 `/plan`、`/code-review`、`webapp-testing`、`skill-creator` 只作为局部能力基准，不与完整端到端交付工具做一一等价比较。

---

## 二、核心对象定位与流程差异

整体上，这几类工具并不是完全同型：

- OpenSpec / Spec Kit 更偏 **spec-driven development**。
- BMAD-METHOD 更偏 **AI Agile 多角色开发方法论**。
- SuperClaude 更偏 **Claude Code 增强型命令 / Agent 工具箱**。
- `fully-coding` 更偏 **项目级 9 步串行交付 Orchestrator**。

### 2.1 fully-coding

`fully-coding` 是一个面向单任务开发的 **9 步串行交付 Orchestrator**。

```text
用户输入
  → Step 1 需求检索
  → Step 2 需求生成
  → Step 3 开发范围
  → Step 4 开发方案
  → Step 5 开发实现
  → Step 6 代码评审
  → Step 7 测试用例
  → Step 8 文档更新
  → Step 9 自主进化
```

核心特点：

- 支持完整 9 步标准模式、`--plan-only` 方案模式与 `--quick-dev` 轻量开发模式。
- 单 Claude 实例串行执行。
- 文档驱动开发。
- 每步有输入/输出契约。
- `.dev-log` 状态落盘。
- 支持心跳与 auto-resume。
- Step 6 / Step 7 支持最多 3 轮 auto-fix。
- DBA + Reviewer 合并评审。
- Step 8 强制同步项目知识库。
- `fully-coding-batch` 支持多个子任务串行开发。
- 项目规则强约束：DDD、编码规范、安全、错误码、幂等性、Git 规范等。

适合：

- 企业项目。
- 规范化开发。
- 需要需求追踪。
- 需要代码评审。
- 需要测试闭环。
- 需要知识库同步。
- 需要中断恢复。
- 需要批次任务串行执行。

核心短板：

- 标准 9 步模式较重。
- 简单任务 token 成本偏高。
- 首次理解成本高。
- 任务拆解结构不如 OpenSpec / Spec Kit 显式。
- 命令灵活性不如 SuperClaude。
- 产品 / PRD / UX 层角色表达不如 BMAD-METHOD 丰富。

一句话定位：

> `fully-coding` 更像“项目内规范化交付流水线”，不是单点 coding assistant。

### 2.2 OpenSpec

> 注：OpenSpec 此处按“根据需求生成开发计划，并进一步推进开发”的能力建模；具体实现细节需结合实际仓库版本进一步验证。

典型流程：

```text
需求输入 → 生成 spec → 生成开发计划 → 拆分任务 → 推进实现
```

优势通常集中在：

- 需求结构化。
- 开发计划生成。
- 任务拆解。
- 计划驱动实现。

与 `fully-coding` 的关系：

- 是 Step 2-5 的直接竞品。
- 通常不一定完整覆盖评审、测试 auto-fix、知识库同步、心跳续跑、blockLog、batch 调度和自主进化。

### 2.3 GitHub Spec Kit

典型流程：

```text
constitution → spec → clarify → plan → tasks → implement
```

主要特点：

- 支持 Claude Code 工作流。
- 有 CLI / slash command 形式的操作体验。
- 产物包括 constitution、spec、plan、tasks、checklist。
- 技术栈中立。
- 适合 greenfield 和 brownfield 增量开发。

与 `fully-coding` 的关系：

- 在需求结构化、技术方案、任务拆解、实现前治理方面竞争力强。
- 不一定提供项目知识库同步、DDD / 错误码 / 幂等强规则、DBA + Reviewer 合并评审、测试失败自动修复、`.dev-log` 审计日志和 auto-resume 状态机。

### 2.4 BMAD-METHOD

典型流程：

```text
brainstorming → product brief → PRD → architecture → implementation → test / deployment support
```

主要特点：

- 强调 PM、Architect、Developer、UX、Test Architect 等角色分工。
- 更偏产品、PRD、架构和多角色协作方法论。
- 与 `fully-coding` 一样重视阶段化流转，但 `fully-coding` 更偏项目规则、落盘、续跑和交付闭环。

### 2.5 SuperClaude Framework

典型能力方向：

```text
/sc:implement
/sc:test
/sc:research
/sc:pm
/sc:analyze
/sc:document
```

主要特点：

- 更像增强型命令和 Agent 工具箱。
- 优势在灵活、命令丰富、能力覆盖广、开发体验好。
- 劣势是不一定强制落盘、不一定有固定步骤契约、不一定有统一审计日志、不一定有知识库同步和 auto-resume。

---

## 三、能力矩阵与场景适配

### 3.1 9 步流程覆盖对比

| fully-coding 步骤 | OpenSpec | Spec Kit | BMAD-METHOD | SuperClaude | fully-coding |
|---|---:|---:|---:|---:|---:|
| Step 1 需求检索 | 中 | 中 | 中 | 中 | 高 |
| Step 2 需求生成 | 高 | 高 | 高 | 中高 | 高 |
| Step 3 开发范围 | 高 | 高 | 高 | 高 | 高 |
| Step 4 开发方案 | 高 | 高 | 高 | 高 | 高 |
| Step 5 开发实现 | 中高 | 高 | 高 | 高 | 高 |
| Step 6 代码评审 | 中 | 中 | 中 | 中高 | 高 |
| Step 7 测试用例 | 中 | 中高 | 中高 | 高 | 高 |
| Step 8 文档更新 | 中 | 中 | 中 | 中高 | 高 |
| Step 9 自主进化 | 低 | 中 | 中 | 中 | 高 |

结论：

- OpenSpec / Spec Kit 强在 Step 2-5。
- BMAD-METHOD 强在 Step 1-5，尤其产品、PRD、架构。
- SuperClaude 强在 Step 3-7，尤其开发体验和工具命令覆盖。
- `fully-coding` 覆盖 Step 1-9，尤其后半程闭环最强。

### 3.2 核心能力矩阵

| 能力 | OpenSpec | Spec Kit | BMAD-METHOD | SuperClaude | fully-coding |
|---|---:|---:|---:|---:|---:|
| 自然语言需求结构化 | 高 | 高 | 高 | 中 | 高 |
| PRD / spec 产物 | 高 | 高 | 高 | 中 | 中高 |
| AC 验收标准 | 高 | 高 | 高 | 中 | 高 |
| 歧义识别 | 中高 | 高 | 高 | 中 | 高 |
| 开发计划 | 高 | 高 | 高 | 高 | 高 |
| 任务拆解 | 高 | 高 | 高 | 中高 | 中高 |
| 按计划实现 | 中高 | 高 | 高 | 高 | 高 |
| 项目规则遵循 | 中 | 中 | 中 | 中高 | 高 |
| DDD / 错误码 / 幂等 | 取决配置 | 取决配置 | 取决配置 | 取决配置 | 高 |
| DBA 视角评审 | 低 | 低中 | 中 | 中 | 高 |
| auto-fix 闭环 | 低中 | 低中 | 中 | 中 | 高 |
| 测试失败分析与修复 | 低中 | 低中 | 中 | 中高 | 高 |
| 项目知识库同步 | 低中 | 低中 | 中 | 中 | 高 |
| 错误码文档同步 | 低 | 低 | 低 | 低 | 高 |
| 功能实现概览同步 | 低 | 低 | 低 | 低 | 高 |
| 状态机与 auto-resume | 低中 | 中 | 中 | 中 | 高 |
| blockLog / 审计追踪 | 低 | 低 | 低 | 低 | 高 |

### 3.3 企业级关键要求对比

| 要求 | OpenSpec | Spec Kit | BMAD-METHOD | SuperClaude | fully-coding |
|---|---:|---:|---:|---:|---:|
| 需求追踪 | 中 | 高 | 高 | 中 | 高 |
| 代码审计 | 中 | 中 | 中 | 中 | 高 |
| 测试闭环 | 中 | 中 | 中 | 中高 | 高 |
| 文档同步 | 中 | 高 | 高 | 中 | 高 |
| 失败恢复 | 中 | 中 | 中 | 中 | 高 |
| 项目规范 | 中 | 中 | 中 | 中 | 高 |
| 多任务管理 | 低中 | 中 | 中 | 中 | 高 |
| Git 安全约束 | 低 | 中 | 中 | 中 | 高 |

### 3.4 典型场景推荐

| 场景 | 推荐工具 | 原因 |
|---|---|---|
| 快速把需求变成开发计划 | OpenSpec / Spec Kit / `fully-coding --plan-only` | spec / plan / tasks 结构强，`--plan-only` 可保留项目上下文和审计日志 |
| 复杂产品功能从 0 到 1 | BMAD-METHOD / Spec Kit / fully-coding | BMAD 产品和 PRD 强，Spec Kit 任务治理强，fully-coding 交付闭环强 |
| 企业 Java 后端功能交付 | fully-coding | DDD、错误码、幂等、数据库、评审、测试、知识库同步、续跑 |
| 前端页面 / Web App 测试 | SuperClaude / webapp-testing / fully-coding + webapp-testing 思路 | Playwright、截图、DOM、浏览器日志更关键 |
| 小 bug 快速修复 | SuperClaude / Claude Code `/plan` + `/code-review` / `fully-coding --quick-dev` | 单点命令更轻，`--quick-dev` 适合仍需评审和测试的小改动 |
| 需要审计和断点续跑的长任务 | fully-coding / fully-coding-batch | 状态机、heartbeat、blockLog、batch progress、共享分支保护 |

---

## 四、性能、准确性与可靠性评估

### 4.1 性能对比（定性预估）

| 工具 | Token 成本 | 执行耗时 | 说明 | 证据等级 |
|---|---:|---:|---|---|
| OpenSpec | 中 | 快-中 | 主要消耗在 spec / plan / tasks | 低 |
| Spec Kit | 中-高 | 中-中高 | constitution、spec、clarify、plan、tasks 都会产生上下文 | 中 |
| BMAD-METHOD | 高 | 高 | 多角色、多文档、PRD、架构、实现流程较长 | 中 |
| SuperClaude | 中 | 快-中 | 按命令使用时较轻，但 MCP / 多 Agent 会增加成本 | 中 |
| fully-coding 标准模式 | 中-高 | 高 | 9 步完整流程、评审、测试、文档同步都会消耗 token | 高 |
| fully-coding `--plan-only` | 中 | 中 | 只执行 Step 1-4，保留需求检索、范围、方案和审计日志 | 高 |
| fully-coding `--quick-dev` | 中 | 中 | 跳过完整方案和文档闭环，但保留范围、实现、评审和测试 | 高 |

定性排序：

```text
从需求到开发计划：OpenSpec ≈ Spec Kit > fully-coding --plan-only > fully-coding 标准模式 > BMAD-METHOD
完整交付闭环：fully-coding 标准模式 > Spec Kit > BMAD-METHOD ≈ SuperClaude > OpenSpec
低 token 小任务：SuperClaude > OpenSpec > Spec Kit > fully-coding --quick-dev > fully-coding 标准模式 > BMAD-METHOD
```

说明：这里的排序不是实测速度排名，而是基于流程完整度、上下文加载量和默认执行路径的预估。

### 4.2 准确性对比

| 工具 | 需求准确性 | 方案准确性 | 代码准确性 | 文档一致性 | 说明 | 证据等级 |
|---|---:|---:|---:|---:|---|---|
| OpenSpec | 4.0 | 4.0 | 3.5 | 3.0 | spec / plan 强，但项目知识库和后半程闭环待验证 | 低 |
| Spec Kit | 4.5 | 4.5 | 4.0 | 4.0 | spec → plan → tasks 结构清晰，checklist 能提高准确性 | 中 |
| BMAD-METHOD | 4.5 | 4.5 | 4.0 | 4.0 | PRD、PM、Architect 流程强 | 中 |
| SuperClaude | 3.5 | 4.0 | 4.0 | 3.0 | 取决于命令和上下文，不强制固定文档闭环 | 中 |
| fully-coding | 4.5 | 4.5 | 4.5 | 4.5 | 有需求检索、开发方案、实现规则、评审、测试和 Step 8 文档同步 | 高 |

### 4.3 可靠性对比

| 工具 | 可续跑 | 可审计 | 可控失败 | 适合长任务 | 综合可靠性 | 说明 |
|---|---:|---:|---:|---:|---:|---|
| OpenSpec | 中 | 中 | 中 | 中 | 3.5 | 需结合具体实现验证 |
| Spec Kit | 中 | 高 | 中 | 高 | 4.0 | spec / plan / tasks 文档化较强 |
| BMAD-METHOD | 中 | 高 | 中 | 高 | 4.0 | 多角色产物较完整，但运行时状态机取决实现 |
| SuperClaude | 中 | 中 | 中 | 中 | 3.5 | 灵活但不一定有固定审计和续跑模型 |
| fully-coding | 高 | 高 | 高 | 高 | 4.5 | status、current-step、current-role、heartbeat、blockLog、codingLog、auto-resume 明确 |

### 4.4 综合评分（定性版）

> 评分范围：1-5，5 为最好。当前评分是基于工具定位和能力覆盖的定性评估，不代表实测 benchmark 结果。

| 工具 | 需求计划 | 开发实现 | 评审测试 | 文档同步 | 可靠性 | 性能 | 灵活性 | 综合 | 证据等级 |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---|
| OpenSpec | 4.5 | 3.8 | 3.0 | 3.0 | 3.5 | 4.2 | 4.0 | 3.7 | 低 |
| Spec Kit | 4.7 | 4.0 | 3.5 | 4.0 | 4.0 | 3.8 | 3.8 | 4.0 | 中 |
| BMAD-METHOD | 4.8 | 4.0 | 3.8 | 4.0 | 4.0 | 3.2 | 3.5 | 4.0 | 中 |
| SuperClaude | 3.8 | 4.2 | 4.0 | 3.5 | 3.5 | 4.3 | 4.8 | 4.0 | 中 |
| fully-coding | 4.5 | 4.5 | 4.7 | 4.8 | 4.8 | 3.3 | 3.2 | 4.4 | 高 |

评分解读：

- `fully-coding` 综合分高，原因是闭环完整、可靠性强、文档同步和审计能力明确。
- Spec Kit 是最值得对标的 spec-driven 竞品。
- BMAD-METHOD 在产品和角色流程上很强。
- SuperClaude 在灵活性和开发体验上强。
- OpenSpec 适合作为轻量计划驱动开发竞品，但需要实测确认能力边界。

---

## 五、补充基准与评测方案

### 5.1 补充基准对象分析

| 工具 | 对标 fully-coding 步骤 | 优势 | 局限 |
|---|---|---|---|
| Claude Code `/plan` | Step 3 开发范围、Step 4 开发方案 | 官方内置、轻量、适合改动前规划 | 不是完整交付流程，不强制文档落盘，不负责测试和知识库同步 |
| Claude Code `/code-review` | Step 6 代码评审 | 官方内置、成本低、常规 review 能力强 | 不一定理解 userStory / codingLog，不一定包含 DBA 视角和 auto-fix 循环 |
| Anthropic `webapp-testing` | Step 7 测试用例 | Playwright、截图、DOM 检查、浏览器日志能力成熟 | 主要覆盖 Web UI，不覆盖后端单测、集成测试、数据库测试 |
| Anthropic `skill-creator` | benchmark / eval | 支持 with-skill / baseline 对比、assertions、timing、tokens、review viewer | 不是业务开发工具，而是评测工具 |

### 5.2 建议评测指标体系

| 类型 | 指标 | 说明 |
|---|---|---|
| 性能指标 | 首次计划耗时 | 从输入需求到输出 plan 的时间 |
| 性能指标 | 完整交付耗时 | 从输入需求到代码 + 测试 + 文档完成 |
| 性能指标 | total tokens | 总 token 消耗 |
| 性能指标 | 每步 tokens | 观察是否有 token 热点 |
| 性能指标 | 失败重试成本 | auto-fix / retry 消耗 |
| 准确性指标 | 需求覆盖率 | 是否覆盖所有用户需求 |
| 准确性指标 | AC 完整率 | 验收标准是否完整 |
| 准确性指标 | 影响范围准确率 | 文件、模块、服务定位是否准确 |
| 准确性指标 | 代码一次通过率 | 编译 / lint / test 首次通过比例 |
| 准确性指标 | 文档同步完整率 | 应更新文档是否全部更新 |
| 可靠性指标 | 中断恢复成功率 | 模拟中断后能否恢复 |
| 可靠性指标 | 状态一致性 | 状态文档与真实代码是否一致 |
| 可靠性指标 | block 判定准确率 | 是否过度阻塞或漏阻塞 |
| 工程适配指标 | DDD / 数据库 / 错误码 / 安全 / Git / 知识库规范 | 是否遵循项目规则 |

### 5.3 建议测试任务集

| 场景 | 输入示例 | 评估重点 |
|---|---|---|
| 简单查询接口 | 新增资讯分类列表查询接口，支持启用状态过滤，返回按 sortOrder 排序的树形结构。 | plan 速度、affected files 准确率、代码一次通过率、测试覆盖 |
| 复杂写操作 | 新增用户收藏资讯功能，要求防重复收藏、支持取消收藏、记录操作时间，并保证并发下幂等。 | 幂等性、唯一约束、事务、错误码、并发测试 |
| 后台管理页面 | 后台管理端新增资讯标签管理页面，支持分页、搜索、新增、编辑、禁用。 | 前后端联动、菜单 / 路由 / 权限、文档同步、UI 测试 |
| 明确 Bug 修复 | 修复资讯列表按发布时间倒序排序不生效的问题。 | 定位速度、是否过度设计、token 消耗、测试回归 |
| 跨模块会员权限 | 新增会员订阅策略，支持免费、试用、付费三种状态，并影响资讯访问权限。 | 领域边界、权限影响范围、数据模型、测试覆盖、文档一致性 |
| 中断恢复测试 | 执行到开发实现后模拟中断，再恢复任务继续完成评审、测试和文档更新。 | 是否能恢复、是否重复修改、状态是否一致、文档是否完整 |

### 5.4 Benchmark 落地建议

建议新增：

```text
.claude/skills/fully-coding/evals/
  evals.json
  scenarios/
  benchmark-template.md
```

评测对象：

```text
fully-coding
OpenSpec
Spec Kit
BMAD-METHOD
SuperClaude
Claude Code baseline
```

记录：

```text
tokens
duration
plan completeness
compile success
test success
review findings
doc sync completeness
human score
```

价值：

- 从主观“好不好”变成可度量。
- 后续优化 Skill 时有依据。
- 可验证 token 优化是否有效。
- 可验证 `--plan-only`、`--quick-dev` 是否真正降低成本。

---

## 六、fully-coding 当前状态与优化路线

### 6.1 已完成优化

| 优化项 | 状态 | 对标对象 | 价值 |
|---|---|---|---|
| `--plan-only` 方案模式 | 已完成 | OpenSpec / Spec Kit | 只执行 Step 1-4，生成需求、开发范围和开发方案，不修改代码 |
| `--quick-dev` 轻量开发模式 | 已完成 | SuperClaude / `/plan` / `/code-review` | 执行最小需求记录 → Step 3 → Step 5 → Step 6 → Step 7，降低小任务流程成本 |
| 功能实现文档复用机制 | 已完成 | 项目知识库治理 | 先检索 `6.功能实现（概览）.md`，优先复用既有功能文档，仅缺失或核心大功能时新建 |
| 功能实现文档命名规范 | 已完成 | 项目知识库治理 | 新文档命名为 `6.功能实现-[管理web|会员web|会员app]-[一级菜单|核心功能]-[二级菜单].md` |

### 6.2 后续优先优化方向

| 优先级 | 优化项 | 对标对象 | 建议动作 |
|---|---|---|---|
| P0 | Step 4 增加 Implementation Tasks | OpenSpec / Spec Kit | 在开发方案中增加可执行任务清单，方便 Step 5 按任务推进 |
| P0 | 建立 benchmark | skill-creator | 用固定任务集验证 tokens、duration、compile success、test success、doc sync completeness |
| P1 | 强化 Web 测试流程 | webapp-testing / SuperClaude | 吸收“启动服务 → 页面稳定 → 截图 → DOM 探测 → 操作 → 断言 → 收集日志”的流程 |
| P1 | 优化 `--quick-dev` 日志粒度 | SuperClaude / `/code-review` | 保留审计信息，同时减少不必要的长文档输出 |
| P1 | 增加竞品模式说明 | OpenSpec / Spec Kit | 在使用指南中说明何时用标准模式、`--plan-only`、`--quick-dev`、`--start-step 5` |
| P2 | 增强 PRD / 产品分析模板 | BMAD-METHOD | 补充产品、UX、用户旅程、业务假设等前置分析能力 |

### 6.3 建议的执行模式说明

建议在使用指南中补充以下说明：

```markdown
【与 OpenSpec / Spec Kit 的关系】

- 需要快速生成 plan：使用 `--plan-only`
- 需要完整交付：使用完整 9 步标准模式
- 已有 plan：使用 `--start-step 5`
- 小 bug / 明确小改动：使用 `--quick-dev`
```

### 6.4 fully-coding 的竞争优势

`fully-coding` 的核心优势不是“计划最快”，而是：

```text
完整交付闭环
项目规则遵循
评审测试自动修复
知识库同步
中断恢复
审计追踪
batch 串行调度
```

它最适合：

- 企业项目。
- 复杂后端功能。
- 需要文档一致性。
- 需要严格开发规范。
- 需要过程审计。
- 需要续跑和自动恢复。

### 6.5 fully-coding 的主要风险

```text
流程偏重
token 成本较高
简单任务体验不如轻量工具
任务拆解显式度不如 Spec Kit / OpenSpec
命令灵活性不如 SuperClaude
产品 / PRD 角色丰富度不如 BMAD-METHOD
```

应对方向不是删除 9 步标准流程，而是继续强化分层执行模式：

```text
完整交付：标准模式 Step 1-9
只出方案：--plan-only
快速修复：--quick-dev
已有方案继续实现：--start-step 5
超长需求拆分：fully-coding-batch
```

---

## 七、最终结论与参考来源

### 7.1 总体判断

五个核心工具可以这样定位：

```text
OpenSpec：轻量 spec-driven 开发推进工具
Spec Kit：结构化、治理更强的 spec-driven toolkit
BMAD-METHOD：AI Agile 多角色开发方法论
SuperClaude：Claude Code 增强型命令 / Agent 工具箱
fully-coding：项目级 9 步串行交付 Orchestrator
```

### 7.2 最终推荐

如果要做后续产品化或工具演进，建议将 `fully-coding` 定位为：

> 面向企业项目的 Claude Code 规范化交付 Orchestrator，强调文档驱动、项目规则、评审测试闭环、知识库同步、状态可恢复和过程可审计。

同时吸收竞品优势：

- 吸收 OpenSpec / Spec Kit 的 tasks 显式拆解能力。
- 吸收 BMAD-METHOD 的产品 / PRD / 多角色表达能力。
- 吸收 SuperClaude 的轻量命令与开发体验。
- 吸收 `webapp-testing` 的前端验证模式。
- 吸收 `skill-creator` 的 benchmark 方法。

### 7.3 结论适用边界

本报告当前最适合作为 `fully-coding` 的内部定位、功能规划和优化优先级判断材料。关于竞品能力、性能和准确性的判断，主要基于公开文档、工具定位和项目经验推断；除非后续 benchmark 明确验证，否则不应视为严格实测结论。

### 7.4 参考来源

- [GitHub Spec Kit](https://github.com/github/spec-kit)
- [BMAD-METHOD](https://github.com/bmad-code-org/BMAD-METHOD)
- [SuperClaude Framework](https://github.com/SuperClaude-Org/SuperClaude_Framework)
- [Anthropic Skills](https://github.com/anthropics/skills)
- [Claude Code Skills documentation](https://code.claude.com/docs/en/skills)
- [Claude Code Commands documentation](https://code.claude.com/docs/en/commands)
