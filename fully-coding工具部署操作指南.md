# fully-coding 工具部署操作指南

> 将当前最新版 `fully-coding` 与 `fully-coding-batch` 部署到新项目的操作手册。两者共享 `fully-coding/project-config.md`，部署到新项目时优先修改该配置文件。

---

## 一、适用范围

本指南适用于部署以下文件：

- `.claude/skills/fully-coding/`
- `.claude/skills/fully-coding-batch/`
- `.claude/skills/QUICKSTART-fully-coding.md`
- `.claude/skills/QUICKSTART-fully-coding-batch.md`
- `.claude/skills/fully-coding工具使用指南.md`
- `.claude/skills/fully-coding工具部署操作指南.md`

`fully-coding` 负责单任务 9 步串行开发流程；`fully-coding-batch` 负责将多功能需求拆分为多个子任务，并串行调用 `fully-coding`。

---

## 二、部署前检查

| 检查项 | 说明 |
|---|---|
| Claude Code 已安装 | CLI / 桌面版 / IDE 插件均可 |
| 目标项目可被 Claude Code 访问 | 在目标项目根目录打开 Claude Code |
| 目标项目已有代码工程 | Java 后端项目优先，规则默认面向 Spring Boot + MyBatis-Plus + DDD 分层 |
| 目标项目已有或计划创建知识库目录 | 默认 `_knowledge`，可通过 `project-config.md` 调整 |
| 目标项目是否已有 `.claude/skills` | 已有时注意不要覆盖其他 Skill |
| 是否需要续跑旧任务 | 通常不迁移 `.dev-log`；只有要恢复旧任务时才迁移 |

---

## 三、复制文件清单

在新项目根目录下执行：

```bash
mkdir -p .claude/skills
cp -r {源项目}/.claude/skills/fully-coding .claude/skills/
cp -r {源项目}/.claude/skills/fully-coding-batch .claude/skills/
cp {源项目}/.claude/skills/QUICKSTART-fully-coding.md .claude/skills/
cp {源项目}/.claude/skills/QUICKSTART-fully-coding-batch.md .claude/skills/
cp {源项目}/.claude/skills/fully-coding工具使用指南.md .claude/skills/
cp {源项目}/.claude/skills/fully-coding工具部署操作指南.md .claude/skills/
```

如果目标项目已有同名 Skill，先备份旧目录后再覆盖：

```bash
cp -r .claude/skills/fully-coding .claude/skills/fully-coding.bak
cp -r .claude/skills/fully-coding-batch .claude/skills/fully-coding-batch.bak
```

---

## 四、当前目录结构

部署完成后，核心结构应为：

```text
{项目根目录}/
├── .claude/skills/
│   ├── QUICKSTART-fully-coding.md
│   ├── QUICKSTART-fully-coding-batch.md
│   ├── fully-coding工具使用指南.md
│   ├── fully-coding工具部署操作指南.md
│   ├── fully-coding/
│   │   ├── SKILL.md
│   │   ├── project-config.md
│   │   ├── agents/
│   │   │   ├── orchestrator.md
│   │   │   ├── requirement-analyst.md
│   │   │   ├── architect.md
│   │   │   ├── developer.md
│   │   │   ├── dba.md
│   │   │   └── reviewer.md
│   │   ├── rules/
│   │   │   ├── workflow-rules.md
│   │   │   ├── runtime-load-map.md
│   │   │   ├── review-checklist.md
│   │   │   ├── heartbeat-resume-rules.md
│   │   │   ├── git-rules.md
│   │   │   ├── backend-ddd-layer-rules.md
│   │   │   ├── backend-code-standard-rules.md
│   │   │   ├── frontend-code-standard-rules.md
│   │   │   ├── persistence-sql-cache-rules.md
│   │   │   ├── implementation-checklist-rules.md
│   │   │   ├── security-rules.md
│   │   │   └── error-handling-rules.md
│   │   ├── hooks/
│   │   │   └── pre-execution-hook.md
│   │   └── templates/
│   │       ├── user-story-template.md
│   │       ├── coding-log-template.md
│   │       ├── block-log-template.md
│   │       ├── auto-resume-log-template.md
│   │       └── suggestion-template.md
│   └── fully-coding-batch/
│       ├── SKILL.md
│       ├── agents/
│       │   └── batch-orchestrator.md
│       ├── rules/
│       │   ├── batch-runtime-load-map.md
│       │   ├── batch-workflow-rules.md
│       │   └── batch-resume-rules.md
│       └── templates/
│           └── batch-progress-template.md
├── _knowledge/
└── .dev-log/
```

`_knowledge/` 和 `.dev-log/` 的实际目录名由 `project-config.md` 的 `knowledge_dir` 与 `output_dir` 决定。

---

## 五、修改 project-config.md

打开：

```text
.claude/skills/fully-coding/project-config.md
```

这是唯一项目级配置入口。`fully-coding-batch` 共享该配置，不需要单独配置。

### 5.1 路径配置

| 配置项 | 说明 | 建议 |
|---|---|---|
| `knowledge_dir` | 知识库目录 | 默认 `_knowledge` |
| `output_dir` | 运行日志目录 | 默认 `.dev-log` |

### 5.2 知识库文档配置

| 配置项 | 说明 | 修改时机 |
|---|---|---|
| `requirement_doc` | 需求终稿文档 | 必改为目标项目需求文档名 |
| `error_code_doc` | 错误码段位文档 | 必改为目标项目错误码文档名 |
| `tech_design_glob` | 技术方案文档匹配规则 | 按目标项目命名调整 |
| `service_design_glob` | 服务详细设计文档匹配规则 | 按目标项目命名调整 |
| `feature_impl_glob` | 功能实现详情文档匹配规则 | 按目标项目命名调整 |
| `feature_impl_template` | 功能实现模板文档 | 按目标项目命名调整 |
| `feature_impl_overview` | 功能实现概览索引 | 按目标项目命名调整 |
| `feature_impl_menu_iteration` | 菜单/页面/路由/权限迭代索引 | 有前端入口时建议保留 |
| `menu_client_apps` | 菜单涉及端列表 | 按项目端类型调整 |
| `tech_architecture_doc` | 架构总览文档 | 按目标项目命名调整 |

### 5.3 心跳与续跑配置

| 配置项 | 说明 | 默认值 |
|---|---|---|
| `heartbeat_interval_seconds` | 心跳写入间隔 | `300` |
| `heartbeat_timeout_seconds` | 心跳超时时间 | `3600` |

### 5.4 后端框架配置

| 配置项 | 说明 | 示例 |
|---|---|---|
| `framework_type` | 框架类型标识 | `grace-ddd` / `xxx-ddd` / `non-ddd` |
| `response_wrapper_success` | 成功响应包装方法 | `DataResponse.success` / `R.ok` |
| `response_wrapper_fail` | 失败响应包装方法 | `DataResponse.fail` / `R.fail` |
| `base_response_class` | 基础响应类 | `BaseResponse` / `R` |

### 5.5 错误码配置

| 配置项 | 说明 | 示例 |
|---|---|---|
| `error_code_format` | 错误码结构 | `[L][CC][SS][D]` |
| `error_code_utility` | 错误码工具类 | `DomainSeq` |
| `error_code_utility_package` | 错误码工具类包名 | `com.xxx.common.error` |
| `error_code_enums` | 错误码枚举文件列表 | `["ErrCodeLevel.java", "ErrCodeCategory.java"]` |

### 5.6 维护原则

- `project-config.md` 是唯一配置来源，不支持运行时覆盖。
- 新增配置占位符时，必须先在 `project-config.md` 声明对应 key。
- 不要在其他 agent/rule/template 文件里硬编码项目路径、响应类、错误码类名。

---

## 六、创建知识库最小文档

先创建目录：

```bash
mkdir -p _knowledge
```

最小必备文档：

| 配置项 | 对应文档 | 最低要求 |
|---|---|---|
| `requirement_doc` | 需求终稿 | 至少包含项目目标、功能清单或需求骨架 |
| `error_code_doc` | 错误码段位文档 | 至少说明错误码结构和已有段位 |
| `feature_impl_template` | 功能实现模板 | 用于创建具体功能实现文档 |
| `feature_impl_overview` | 功能实现概览 | 至少包含最小表格结构 |
| `feature_impl_menu_iteration` | 菜单功能迭代索引 | 有菜单/页面/路由/权限入口时使用 |

可选但推荐准备：

| 配置项 | 用途 |
|---|---|
| `tech_design_glob` | 技术方案系列文档 |
| `service_design_glob` | 服务详细设计系列文档 |
| `tech_architecture_doc` | 架构总览文档 |
| `feature_impl_glob` | 已有具体功能实现文档 |

如果新项目暂时没有知识库文档，可以先创建空骨架；后续 Step 1/2 和 Step 8 会逐步补充。

---

## 七、部署后验证

### 7.1 验证 fully-coding 可加载

在 Claude Code 中执行：

```text
/fully-coding --status
```

预期结果：

- Skill 能正常触发；
- 如果没有未完成任务，提示无可恢复任务；
- 如果有未完成任务，显示当前任务状态。

### 7.2 验证一个最小任务

建议用低风险需求试跑：

```text
/fully-coding 检查当前项目的启动类和基础目录结构，生成开发范围说明，不修改业务代码
```

如果只想验证流程，不建议选择会大面积改代码的需求。

### 7.3 验证 batch 相关文档

batch 状态优先读取进度文档：

```text
Read .dev-log/BATCH-{batch-ts}-progress.md
```

`/fully-coding-batch --auto-resume` 是恢复/接管命令，不是纯状态查询命令；只有需要继续未完成批次时再使用。

---

## 八、常见部署问题

### Q1：提示找不到知识库文档怎么办？

检查 `project-config.md` 中的文档名是否与 `_knowledge/` 下文件名一致。至少先创建必备文档骨架。

### Q2：提示某个配置占位符缺失怎么办？

检查是否有 `{{config.xxx}}` 在 Skill 文档中被使用，但 `project-config.md` 没声明 `xxx`。补齐配置后再运行。

### Q3：batch 找不到 fully-coding 规则怎么办？

确认目录关系保持为：

```text
.claude/skills/fully-coding/
.claude/skills/fully-coding-batch/
```

两者必须是同级目录。`fully-coding-batch` 的 nested 文档会通过 `../../fully-coding/...` 引用单任务规则。

### Q4：新项目没有错误码体系怎么办？

先在 `project-config.md` 中填写计划采用的错误码格式、工具类和枚举文件名；再在 `error_code_doc` 中创建基础说明。后续 Step 4/5/8 会按该体系设计和补充。

### Q5：新项目不是 DDD 架构怎么办？

将 `framework_type` 改为项目实际类型，例如 `non-ddd`，并按项目分层修改 `rules/backend-ddd-layer-rules.md`。不建议直接删除规则文件，避免引用断裂。

### Q6：可以只部署 fully-coding 不部署 fully-coding-batch 吗？

可以。`fully-coding` 可独立运行；`fully-coding-batch` 只是批量拆分和串行调度扩展。

### Q7：部署后如何回滚？

删除或恢复备份目录即可：

```bash
rm -rf .claude/skills/fully-coding
rm -rf .claude/skills/fully-coding-batch
mv .claude/skills/fully-coding.bak .claude/skills/fully-coding
mv .claude/skills/fully-coding-batch.bak .claude/skills/fully-coding-batch
```

执行 `rm -rf` 前请确认目录无误。

---

## 九、升级已有项目

如果目标项目已部署旧版 Skill，按以下顺序升级：

1. 备份旧目录：
   ```bash
   cp -r .claude/skills/fully-coding .claude/skills/fully-coding.bak
   cp -r .claude/skills/fully-coding-batch .claude/skills/fully-coding-batch.bak
   ```
2. 覆盖 Skill 目录和根目录指南文档。
3. 保留目标项目原有 `project-config.md`，不要直接覆盖。
4. 对比新版 `project-config.md`，补齐新增 key。
5. 确认 `fully-coding-batch` 与 `fully-coding` 仍为同级目录。
6. 不迁移 `.dev-log/`，除非需要续跑旧任务。
7. 执行 `/fully-coding --status` 做加载验证。

升级时最容易遗漏的是新增配置项和知识库最小文档。若不确定，优先对照本指南第五、六节检查。