---
name: git-rules
description: fully-coding 工作流的 Git 操作规范。包含分支创建、commit message、提交流程、步骤限制等完整规则。
---

# Git 操作规范

## 1. 前提条件检查

步骤 8 执行 git 操作前，必须先验证以下前提条件：

1. **工程已初始化 git**：存在 `.git/` 目录
2. **已配置 remote**：`git remote -v` 有至少一个 remote 地址
3. **工作区可控**：当前变更仅限于本次任务产生的文件，不得包含无关变更

若任一条件不满足，**不得执行 git 操作**，记录原因到 codingLog.md，并提示用户手动处理。

---

## 2. 分支规范

### 2.1 创建时机

- **fully-coding 单任务**：步骤 3（开发范围）确定后、步骤 5（开发实现）之前
- **fully-coding-batch 批次**：批次步骤 2-B（串行开发启动）统一创建，所有子任务共享

### 2.2 命名格式

```
{type}/{task-id}-{brief-desc}
```

- `type`：feature / fix / refactor / docs / test / chore
- `task-id`：
  - 单任务：本次任务时间戳 `{yyyyMMdd-HHmmss}`
  - 批次：批次时间戳（即 BATCH 进度文档的时间戳）
- `brief-desc`：2-5 个英文单词简述，kebab-case
- **示例**：
  - 单任务：`feature/20250610-143000-admin-batch-import`
  - 批次：`feature/20250610-143000-user-order-pay`

### 2.3 约束

- 禁止在 `main` / `master` 分支上直接提交
- 新分支基于当前分支最新提交创建：先 `git fetch` 获取远程状态，确认本地与远程一致后，再 `git checkout -b`
- 若当前已在功能分支上且与本次任务类型一致，可复用现有分支（需记录到 codingLog.md）

### 2.4 batch 模式特殊规则

- 所有子任务共享同一个分支，由 `fully-coding-batch` 的 Orchestrator 在步骤 2-B 统一创建
- `fully-coding` 通过 `--git-branch {name}` 参数接收共享分支名，跳过分支创建，直接复用
- 子任务启动时自动 `git checkout {branch-name}` 切换到共享分支，确保继承前序子任务的代码变更
- 每个子任务的 commit message 仍使用子任务自己的时间戳，便于追溯
- 批次完成后，所有子任务的代码变更都在同一分支上，用户只需合并该分支即可

---

## 3. Commit Message 规范

```
[{type}] {brief description}

- 任务时间戳：{ts}
- 影响服务：{service}
- 变更摘要：{summary}

Co-Authored-By: Claude <noreply@anthropic.com>
```

- `type`：feat / fix / refactor / docs / test / chore
  - 对应分支命名中的 `feature` / `fix` / `refactor` / `docs` / `test` / `chore`
- `brief description`：50 字以内，说明本次变更核心内容
- 必须包含任务时间戳，便于与 `.dev-log/` 下的文档关联追溯
- 必须包含 `Co-Authored-By` 标记

---

## 4. 提交流程

步骤 8 中的 git 操作必须按以下顺序执行：

1. `git status` 检查变更范围，确认无无关文件
2. `git add` **逐个指定文件**（禁止 `git add -A` 或 `git add .`）
3. `git commit -m` 按 Commit Message 规范填写
4. `git push -u origin {branch}` **必须先向用户确认**

### 推送确认规则

- 推送前向用户说明：分支名、commit 数、变更文件清单
- 用户确认后执行 push
- 用户 1 分钟内未响应：记录到 codingLog.md，完结任务并提示用户手动推送
- **禁止 force push**

---

## 5. 步骤限制

| 步骤 | git 操作限制 |
|------|-------------|
| 步骤 1-5 | 禁止任何 git commit / push（允许本地 `git status` / `git diff` 查看） |
| 步骤 6-7 | 禁止 git push（允许本地 commit 作为评审参考，由 Orchestrator 执行） |
| 步骤 8 | 允许 git add / commit / push（push 需用户确认） |

---

## 6. 异常处理

### 分支切换失败（batch 模式）

若 `--start-from` 或 `--git-branch` 切换分支失败：

1. 记录错误原因到 BATCH 进度文档或 codingLog.md
2. **暂停流程**，向用户报告失败原因和修复建议
3. 等待用户手动处理（提交/储藏变更、创建分支等）后再继续

### 推送被拒绝

若 `git push` 因远程分支冲突被拒绝：

1. 不要执行 `git push --force`
2. 向用户报告冲突情况
3. 建议用户手动处理：`git pull --rebase` 或合并远程变更后再推送