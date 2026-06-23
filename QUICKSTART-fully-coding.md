---
name: fully-coding-quickstart
description: fully-coding 快速开始指南。5 分钟上手，从第一次调用到查看任务产出的完整示例。
---

# fully-coding 快速开始

## 1. 前置准备

1. 已安装 Claude Code 并加载本 Skill。
2. 项目根目录已存在 `knowledge_dir`（默认 `_knowledge`）与代码工程。
3. 已根据项目情况填写 `.claude/skills/fully-coding/project-config.md`。

> 若首次部署，只需修改 `project-config.md` 中的路径与错误码相关配置。

## 2. 运行第一个任务

输入自然语言需求即可启动完整 9 步工作流：

```
/fully-coding 帮我实现管理员角色的批量导入功能，支持 Excel 上传和模板下载
```

启动后，Orchestrator 会自动：
1. 生成任务时间戳 `{ts}`（格式 `yyyyMMdd-HHmmss`）。
2. 在 `.dev-log/` 创建 `{ts}-userStory.md` 与 `{ts}-codingLog.md`。
3. 按 9 步串行执行：需求检索 → 需求生成 → 开发范围 → 开发方案 → 开发 → 评审 → 测试 → 文档更新 → 自主进化。

## 3. 执行模式

| 模式 | 命令示例 | 说明 |
|------|----------|------|
| 标准模式 | `/fully-coding 帮我实现管理员角色的批量导入功能` | 执行完整 Step 1-9，适合企业功能和需要完整闭环的任务 |
| 方案模式 | `/fully-coding 帮我设计收藏资讯功能 --plan-only` | 只执行 Step 1-4，生成 `userStory.md` 和 `codingLog.md` 中的开发范围/方案，不修改代码 |
| 轻量开发模式 | `/fully-coding 修复登录排序问题 --quick-dev` | 执行最小需求记录 → Step 3 → Step 5 → Step 6 → Step 7，适合小 bug 或明确小改动 |

`--plan-only` 完成后，任务状态为 `planned`；如需继续实现，可使用 `/fully-coding --start-step 5 --task-id {ts}`。

## 4. 流程图

```mermaid
flowchart TD
    A[用户输入自然语言需求] --> B[生成任务时间戳 {ts}<br/>角色：Orchestrator]
    B --> C[创建 .dev-log/{ts}-userStory.md<br/>角色：Orchestrator]
    B --> D[创建 .dev-log/{ts}-codingLog.md<br/>角色：Orchestrator]

    C --> S1[Step 1 需求检索<br/>角色：需求分析师]
    S1 --> S2[Step 2 需求生成<br/>角色：需求分析师]
    S2 --> S3[Step 3 开发范围<br/>角色：架构师]
    S3 --> S4[Step 4 开发方案<br/>角色：架构师]
    S4 --> S5[Step 5 开发实现<br/>角色：开发者]
    S5 --> S6[Step 6 代码评审<br/>角色：DBA + 代码评审员]
    S6 -->|auto-fix| S5
    S6 -->|通过 / 有条件通过| S7[Step 7 测试用例<br/>角色：开发者]
    S7 -->|测试失败且可修复| S5
    S7 -->|测试通过| S8[Step 8 文档更新<br/>角色：架构师]
    S8 --> S9[Step 9 自主进化<br/>角色：Orchestrator]
    S9 --> Z[任务完成 status=completed]

    S1 -.阻塞.-> X[登记 {ts}-blockLog.md 并等待用户确认<br/>角色：当前步骤角色 + Orchestrator]
    S2 -.阻塞.-> X
    S3 -.阻塞.-> X
    S4 -.阻塞.-> X
    S5 -.阻塞.-> X
    S6 -.3 次循环后仍阻塞.-> X
    S7 -.3 次循环后仍失败.-> X
    S8 -.阻塞.-> X
    X --> R[/fully-coding --auto-resume<br/>角色：Orchestrator/]
    R --> Y[读取 current-step<br/>恢复到中断步骤]
    Y --> N[继续串行执行后续步骤<br/>角色：当前步骤角色]
    N --> Z
```

## 5. 查看进度

执行过程中无需等待确认，状态自动写入 `.dev-log/`：

```
/fully-coding --status   # 查看最新未完成任务当前步骤
/fully-coding --list     # 列出所有未完成任务
```

也可直接读取日志文件：

```
Read .dev-log/{ts}-codingLog.md
```

## 6. 断点续跑

若任务因进程退出、电脑重启或阻塞而中断：

```
/fully-coding --auto-resume
```

系统会扫描 `.dev-log/` 下未完成的 `codingLog.md`，按心跳规则安全接管并继续执行。

## 7. 查看最终产出

任务完成后，`.dev-log/` 下会生成：

| 文件 | 内容 |
|------|------|
| `{ts}-userStory.md` | 需求分析、特性摘要、验收标准 |
| `{ts}-codingLog.md` | 开发范围、方案、代码变更、评审、测试、文档更新记录 |
| `{ts}-suggestion.md` | 步骤 9 生成的改进建议 |
| `{ts}-blockLog.md` | 若发生阻塞，记录阻塞原因与处理方案 |

## 8. 批量开发

若需求包含 ≥3 个独立功能，推荐用 batch 模式自动拆分：

```
/fully-coding-batch 实现用户管理、订单系统、支付集成三个模块
```

batch 模式会创建 `BATCH-{batch-ts}-progress.md` 统一监控所有子任务进度。

## 9. 常用参数速查

| 参数 | 作用 |
|------|------|
| `--start-step N` | 从第 N 步开始执行 |
| `--task-id {ts}` | 指定任务时间戳，用于续跑或重跑 |
| `--plan-only` | 只执行 Step 1-4，生成方案后以 `planned` 状态停止 |
| `--quick-dev` | 轻量开发模式，执行最小需求记录 → Step 3 → Step 5 → Step 6 → Step 7 |
| `--auto-resume` | 自动续跑最新未完成任务 |
| `--new-task` | 强制启动新任务，忽略自动恢复 |

## 10. 下一步

- 了解完整 9 步流程：阅读 `fully-coding/SKILL.md`。
- 了解心跳与续跑机制：阅读 `fully-coding/rules/heartbeat-resume-rules.md`。
- 了解编码规范：阅读 `fully-coding/rules/backend-ddd-layer-rules.md` 与 `fully-coding/rules/backend-code-standard-rules.md`。
