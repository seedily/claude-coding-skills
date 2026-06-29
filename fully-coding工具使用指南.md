# Claude 编程工具使用指南

本指南说明 `fully-coding` 与 `fully-coding-batch` 两个 Claude Code Skill 的适用场景、典型用法与最佳实践。

> **路径约定**：本指南中的 `.dev-log/`、`_knowledge/` 等路径来自 `project-config.md` 配置。部署到其他项目时，这些路径会随配置变化。详见 `fully-coding工具部署操作指南.md`。

---

## 一、工具总览

| 工具 | 定位 | 执行方式 | 文档产出 |
|------|------|---------|---------|
| `fully-coding` | 单任务全流程开发 | 单 Claude 实例串行执行 9 步工作流 | `{{config.output_dir}}/{ts}-userStory.md` + `{ts}-codingLog.md` |
| `fully-coding-batch` | 超长需求批次开发 | 自动拆分为多个子任务，串行调用 `fully-coding` | `{{config.output_dir}}/BATCH-{batch-ts}-progress.md` + 多个子任务文档 |

核心原则：**严格串行**、**文档驱动**、**规范优先**。

---

## 二、fully-coding

### 2.1 适用场景

| 适合使用 | 不适合使用 |
|---------|-----------|
| 单一功能或 Bug 修复（1 个 feature） | 一次要改多个独立功能（≥3 个 feature） |
| 需求明确、边界清晰 | 需求高度模糊，需反复协商 |
| 需要完整走需求 → 方案 → 开发 → 评审 → 测试 → 文档 | 只需要无审计、无评审、无测试的临时代码改动 |

执行流程：

```
步骤 1：需求检索  → 需求分析师
步骤 2：需求生成  → 需求分析师
步骤 3：开发范围  → 架构师
步骤 4：开发方案  → 架构师
步骤 5：开发实现  → 开发者
步骤 6：代码评审  → DBA + 代码评审员（同一响应中完成）
步骤 7：测试用例  → 开发者
步骤 8：文档更新  → 架构师
步骤 9：自主进化  → Orchestrator
```

### 2.2 执行模式

| 模式 | 命令示例 | 执行范围 | 典型场景 |
|------|----------|----------|---------|
| 标准模式 | `/fully-coding 帮我实现管理员角色的批量导入功能` | Step 1-9 | 企业功能、复杂后端、需要完整审计和知识库闭环 |
| 方案模式 | `/fully-coding 帮我设计收藏资讯功能 --plan-only` | Step 1-4 | 先生成需求、开发范围和开发方案，用户审方案后再决定是否实现 |
| 轻量开发模式 | `/fully-coding 修复登录排序问题 --quick-dev` | 最小需求记录 → Step 3 → Step 5 → Step 6 → Step 7 | 小 bug、明确小改动，需要评审和测试但不需要完整知识库同步 |

`--plan-only` 完成后任务状态为 `planned`；如需继续实现，使用 `/fully-coding --start-step 5 --task-id {ts}` 进入后续开发闭环。

### 2.3 使用场景

#### 常用参数

| 参数 | 说明 | 典型场景 |
|------|------|---------|
| `--plan-only` | 只执行 Step 1-4，生成方案后停止 | 先审方案、不修改代码 |
| `--quick-dev` | 轻量开发模式，执行最小需求记录、范围、实现、评审、测试 | 小 bug 或明确小改动 |
| `--start-step N` | 从第 N 步开始执行 | 需求已确认，直接开发 |
| `--task-id {ts}` | 指定任务时间戳 | batch 子任务或 resume 时 |
| `--batch-subtask` | 标记为批次子任务 | 由 `fully-coding-batch` 自动传入 |
| `--git-branch <name>` | 指定复用的 git 分支名 | batch 模式专用 |
| `--status` | 查看当前任务进度 | 随时查看执行到哪一步 |
| `--list` | 列出所有未完成任务 | 存在多个未完成任务时 |

#### 基础用法

```
/fully-coding 帮我实现管理员角色的批量导入功能，支持 Excel 上传和模板下载
```

#### 方案模式

```
/fully-coding 帮我设计收藏资讯功能 --plan-only
```

#### 轻量开发模式

```
/fully-coding 修复登录排序问题 --quick-dev
```

#### 从指定步骤开始（断点续跑）

```
/fully-coding 修复登录bug --start-step 5
```

### 2.4 阻塞处理与循环修复

步骤 6（代码评审）和步骤 7（测试用例）支持自动循环修复：

| 场景 | 处理方式 | 最大循环次数 |
|------|---------|-------------|
| 评审发现 `auto-fix` 级别问题 | 自动切换到 Developer 修复，修复后重新评审 | 3 次 |
| 评审发现 `block` 级别问题（仅限致命） | 3 次循环后仍无法解决则暂停，登记 blockLog.md | 3 次 |
| 测试失败 | 自动分析失败原因并修复后重跑 | 3 次 |

**问题分级定义**：
- `auto-fix`（默认）：代码规范、类型安全、业务校验缺失、数据库问题、性能优化、测试不足等可通过修改代码解决的问题
- `block`（仅限致命）：安全漏洞、数据损坏风险、接口契约破坏、需求歧义——必须同时满足「致命且无法自动修复」才标记

超过最大循环次数后，流程中断并登记到 `blockLog.md`（按 `templates/block-log-template.md` 格式），向用户报告阻塞原因和修复建议，等待用户确认处理后再恢复。

**blockLog.md 登记规则**：
- 仅在流程中断、必须用户确认时创建
- 循环修复期内（如评审阻塞但仍在 3 次循环内）不登记
- 用户确认后更新 blockLog.md 对应事件为「已确认」

### 2.5 自动续跑

推荐每 4 小时检查一次：

```
/schedule "0 */4 * * *" /fully-coding --auto-resume
```

**隔离原则**：独立任务的 `auto-resume` 仅扫描独立任务（排除含 `batch-subtask: true` 的日志），batch 子任务由 `fully-coding-batch --auto-resume` 统一调度。两者互不干扰。

### 2.6 git 操作规范

详细 git 规范（分支创建、commit message、提交流程、步骤限制等）参见 `fully-coding/rules/git-rules.md`，以及 `fully-coding/rules/workflow-rules.md` 中的 Git 与危险操作约束。

**核心规则**：
- 分支创建时机：步骤 3 确定后、步骤 5 之前
- 分支命名：`{type}/{yyyyMM}`（月度大版本分支，同类型同月多任务共用，如 `feature/202506`）
- 禁止在 `main`/`master` 上直接提交
- 禁止 force push

### 2.7 任务总结与改进建议

步骤 9（自主进化）会自动生成 `.dev-log/{ts}-suggestion.md`，包含三方面建议：

1. **提示词优化**：回顾原始需求，指出模糊/遗漏之处，给出更清晰表达方式
2. **技术债务**：识别本次开发涉及的系统是否存在重复代码、硬编码、测试覆盖不足等问题
3. **工具改进**：基于本次执行经验，反思流程/模板/角色 Prompt 的优化空间

### 2.8 最佳实践

1. **先确认需求边界再启动**：需求模糊时先用步骤 1/2 输出 userStory.md 确认，再进入步骤 3
2. **不要跳过文档落盘**：每步结束后必须更新对应文档
3. **善用 `--start-step` 断点续跑**：步骤 5 因编译错误中断，修复后可继续
4. **避免多任务并行**：一个未完成任务未完成前，不要启动新的独立任务

---

## 三、fully-coding-batch

### 3.1 适用场景

| 适合使用 | 不适合使用 |
|---------|-----------|
| 用户输入包含 3 个以上独立功能 | 只需改 1-2 个文件的小改动 |
| 多个 CRUD 模块需按顺序实现 | 需求间耦合极强、无法拆分 |
| 多个微服务接口需批量落地 | 时间极紧、不允许走完整流程 |

执行流程：

```
步骤 1：需求拆分       → 生成多个 userStory.md + BATCH 进度文档
步骤 2：串行开发启动   → 逐个调用 fully-coding（从步骤 3 开始）
步骤 3：进度监控       → 实时更新 BATCH 进度文档
步骤 4：任务完成与汇总 → 汇总错误码、公共资源变更
```

### 3.2 使用场景

#### 常用参数

| 参数 | 说明 | 典型场景 |
|------|------|---------|
| `--start-from N` | 从第 N 个子任务开始 | 前序子任务已完成 |
| `--auto-resume` | 自动读取进度文档续跑 | 中断后恢复 |

#### 基础用法

```
/fully-coding-batch 实现用户管理、订单系统、支付集成三个模块
```

#### 断点自动续跑

```
/fully-coding-batch --auto-resume
```

恢复时 Orchestrator 自动从 `BATCH-*-progress.md` 读取共享分支名，通过 `--git-branch` 传入子任务。

#### 从第 N 个子任务手动续跑

```
/fully-coding-batch --start-from 2
```

#### 定时触发

```
/schedule "0 */4 * * *" /fully-coding-batch --auto-resume
```

### 3.3 最佳实践

1. **需求拆分时明确依赖关系**：支付依赖订单、订单依赖商品，拆分必须标注依赖
2. **中断后优先使用 `--auto-resume`**：比 `--start-from` 更安全
3. **不要手动修改 BATCH 进度文档中的状态**：状态由 Orchestrator 自动更新
4. **每个子任务完成后可查看 BATCH 进度文档**：确认方向与公共资源变更是否符合预期
5. **错误码重复时按“前序保持、后序重分配”策略修复**：前序已落盘的保持不变，后序改用未占用值
6. **批次内所有子任务共享同一个 git 分支**：Orchestrator 统一创建，后序子任务自动继承前序代码变更

---

## 四、如何选择工具

```
需求规模判断：

1 个 feature / 1 个 Bug          →  fully-coding
2-3 个弱相关 feature              →  fully-coding（分多次调用）
2 个 feature 但强相关              →  fully-coding-batch（依赖管理更可靠）
≥3 个 feature 或存在依赖关系       →  fully-coding-batch
```

| 维度 | fully-coding | fully-coding-batch |
|------|-------------|-------------------|
| 启动成本 | 低 | 中（需先拆分） |
| 单任务控制力 | 高 | 中（Orchestrator 调度） |
| 适合的需求复杂度 | 单一明确 | 多模块、有依赖 |
| 公共资源管理 | 单任务内完成 | 串行实时追加 + 批次汇总校验 |
| 恢复方式 | `--auto-resume` / `--start-step` | `--auto-resume` / `--start-from` |

---

## 五、配置与可移植性

### 5.1 project-config.md

Skill 的所有项目特有配置集中在 `.claude/skills/fully-coding/project-config.md`。包括：

- 知识库路径和文档名
- 功能实现文档名和菜单端配置
- 心跳与续跑配置
- 框架类型、响应包装类和错误码配置

`fully-coding-batch` 共享同一份配置，无需单独配置。

### 5.2 占位符约定

Skill 内部文件使用 `{{config.*}}` 占位符，由 LLM 运行时从 `project-config.md` 解析。用户无需关心这些占位符——它们是 Skill 内部机制。

### 5.3 部署到其他项目

如需将 Skill 部署到其他 Java 项目，只需：

1. 复制 `.claude/skills/fully-coding/` 和 `fully-coding-batch/`
2. 修改 `project-config.md`
3. 创建知识库目录和初始文档

详见 `fully-coding工具部署操作指南.md`。

---

## 六、常见问题

### Q1：任务中断了怎么恢复？

- **fully-coding（独立任务）**：`/fully-coding --auto-resume`
- **fully-coding-batch（批次）**：`/fully-coding-batch --auto-resume`（自动定位未完成子任务并恢复）

### Q2：为什么 batch 模式也需要串行？

多个子任务可能修改同一公共资源（如错误码段位文档）。串行执行保证后序子任务能看到前序子任务的实时变更，避免覆盖冲突。

### Q3：可以跳过步骤 1/2 吗？

- 只想先出方案：使用 `--plan-only`，完整执行 Step 1-4 后以 `planned` 状态停止
- 小 bug / 明确小改动：使用 `--quick-dev`，写入最小需求记录后进入 Step 3/5/6/7
- 已有完整需求文档：可用 `--start-step 3`，但不建议手工跳步，避免出现步骤跳跃或步骤重复
- `fully-coding-batch`：步骤 1 由 batch 完成，不可跳过

### Q4：batch 子任务发现错误码重复怎么办？

按“前序保持、后序重分配”策略修复：前序子任务已落盘的段位保持不变，后序子任务改用未占用值，同步更新相关文档和代码。

### Q5：auto-resume 什么情况下会跳过？

- 最近 60 分钟内有活跃任务正在执行
- 存在未确认的 `blockLog.md` 阻塞事件
- batch 子任务由 `fully-coding-batch` 统一调度，独立任务 `auto-resume` 自动排除
