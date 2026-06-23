---
type: user-story-template
description: 步骤1和步骤2产出的需求分析文档模板
---

# 需求分析文档

> 任务时间戳：{yyyyMMdd-HHmmss}
> 文档路径：`.dev-log/{yyyyMMdd-HHmmss}-userStory.md`

## 原始提示词

[用户输入原文，完整保留]

## 特性摘要

- 功能名称：
- 目标用户：
- 价值说明：

## 需求检索结果

### 需求终稿命中情况

- [ ] 已存在相同功能
- [ ] 已存在类似功能（需标注差异）
- [ ] 全新功能

### 相关章节

- 章节编号与标题：
- 关键内容摘要：
- 与本次需求的差异：

## 新增/变更需求

### 需求条目

1. **需求 1**：...
2. **需求 2**：...

### 验收标准

1. **AC-1**：GIVEN ... WHEN ... THEN ...
2. **AC-2**：GIVEN ... WHEN ... THEN ...

### 需求假设

- 假设 1：

### 需求歧义记录

- 歧义点：
- 当前处理方式：

## 关联文档

- 可行性分析：
- 需求终稿位置：`{{config.knowledge_dir}}/{{config.requirement_doc}}`
- 功能实现文档：`{{config.knowledge_dir}}/{{config.feature_impl_glob}}`（步骤4补充）

---

## 阻塞记录（步骤 1-2 发生阻塞时填写）

> 阻塞详情同时登记到 `.dev-log/{yyyyMMdd-HHmmss}-blockLog.md`

### 步骤 N 阻塞事件

- **阻塞时间**：{yyyyMMdd-HHmmss}
- **阻塞类型**：需求歧义 / 外部依赖缺失 / 用户决策待确认 / 其他
- **阻塞原因**：
- **阻塞详情**：
- **修复建议**：
- **是否已解决**：
