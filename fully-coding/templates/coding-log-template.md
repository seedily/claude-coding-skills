---
type: coding-log-template
description: 步骤3-9产出的开发日志文档模板，支持标准模式、plan-only 与 quick-dev
status: running
execution-mode: standard
last-heartbeat: {yyyyMMdd-HHmmss}
current-step: 3
current-role: architect
pid: {process-id}
batch-subtask: false
---

# 开发日志

> 任务时间戳：{yyyyMMdd-HHmmss}
> 文档路径：`.dev-log/{yyyyMMdd-HHmmss}-codingLog.md`
> 心跳字段：frontmatter 中的 `status` / `last-heartbeat` / `current-step` / `current-role` / `pid`
> 执行模式：`execution-mode` 可取 `standard` / `plan-only` / `quick-dev`；`plan-only` 完成后 `status=planned`，`quick-dev` 完成 Step 7 后 `status=completed`

## 步骤 3：开发范围

- 影响服务：
- 影响模块：
- 改动文件清单：
  - 新增：
  - 修改：
  - 删除：

## 步骤 4：开发方案

- 方案摘要：
- 新增/更新的功能实现文档：
- 文档处理方式：复用既有文档 / 更新既有文档 / 新建文档（新建原因：缺少可承载文档 / 核心大功能）
- 关键设计决策：
- 错误码使用情况：
- 幂等性设计说明：

## 步骤 5：开发实现

- 代码变更：
- 编译结果：
- 错误码使用情况：
  - 使用已有枚举：
  - 新增枚举（取值/含义/使用场景）：
- 幂等性实现情况：
- 删除/废弃类变更（如适用）：
  - 残留引用检查结果：
  - 父 POM / dependencyManagement / 其他工程依赖清理情况：
- 关键实现说明：

## 步骤 6：代码评审

- 评审人：DBA、架构师/代码评审员

### DBA 意见

- 问题清单：
- 修复结果：

### 代码评审员意见

- 问题清单：
- 评审结论：通过 / 有条件通过 / 阻塞
- 修复结果：

## 步骤 7：测试用例

- 测试覆盖：
- 测试结果：
- 未覆盖项说明：

## 步骤 8：文档更新

- 更新文档清单：
- 功能实现文档更新：
  - 新增/修改功能文档：
  - 概览文档同步：
  - 菜单功能迭代同步：
- 错误码文档更新：
  - 新增 [L][CC] 枚举：
  - 新增系统/领域分配：
- 代码与文档偏差说明：
- 公共资源变更摘要：

## 步骤 9：自主进化

- 自检结果：
  - 文档覆盖检查：
  - 代码一致性检查：
  - 功能实现概览一致性检查：
  - 菜单功能迭代一致性检查：
  - 功能文档路径有效性检查：
- 自动补充更新：
- 工具改进建议：

## 任务状态

- 最终状态：completed / planned / blocked / failed
- 模式说明：标准模式 Step 9 后 completed；plan-only Step 4 后 planned；quick-dev Step 7 后 completed

---

## 阻塞记录（本任务发生阻塞时填写）

> 阻塞详情同时登记到 `.dev-log/{yyyyMMdd-HHmmss}-blockLog.md`

### 步骤 N 阻塞事件

- **阻塞时间**：{yyyyMMdd-HHmmss}
- **阻塞类型**：评审阻塞 / 测试失败 / 编译错误 / 外部依赖缺失 / 用户决策待确认 / 其他
- **阻塞原因**：
- **阻塞详情**：
- **修复建议**：
- **是否已解决**：
