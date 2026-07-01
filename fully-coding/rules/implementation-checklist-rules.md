---
name: implementation-checklist-rules
description: 全栈实现 Checklist。定义 api/dao/domain/infrastructure/app + 前端 + 第三方集成各层的实现顺序、检查项与测试要求。
---

# 全栈实现 Checklist

> 新增功能的实现顺序与检查清单。本 checklist 覆盖后端（api/dao/domain/infrastructure/app）、前端（页面/组件/API对接）、第三方集成（外部API/数据源/中间件）。
>
> 实现开始时，必须先确认本功能涉及哪些层（前端/后端/数据/第三方），涉及的全都要完成，禁止只实现前端或只实现后端。
>
> 详细分层原则与领域模型设计见 `backend-ddd-layer-rules.md`，持久层与缓存规范见 `persistence-sql-cache-rules.md`，异常与错误码见 `error-handling-rules.md`，前端规范见 `frontend-code-standard-rules.md`。

---

## 0. 全栈覆盖检查（强制）

**实现前必须填写**：本功能涉及哪些层？涉及的全都要实现。

| 层 | 涉及？ | 实现内容 | 完成状态 |
|----|--------|----------|----------|
| 后端: Controller (REST API) | 是/否 | 新增/修改接口路径： | 未完成 / 已完成 |
| 后端: Application Service | 是/否 | 用例列表： | 未完成 / 已完成 |
| 后端: Domain Service | 是/否 | 领域逻辑： | 未完成 / 已完成 |
| 后端: Repository/DAO | 是/否 | 数据访问： | 未完成 / 已完成 |
| 后端: PO/DTO | 是/否 | 数据模型： | 未完成 / 已完成 |
| 后端: Mapper/XML | 是/否 | SQL定义： | 未完成 / 已完成 |
| 后端: 事件/消息 | 是/否 | 事件类型： | 未完成 / 已完成 |
| 前端: 页面/组件(.vue) | 是/否 | 页面路径： | 未完成 / 已完成 |
| 前端: API 对接 | 是/否 | API 调用函数： | 未完成 / 已完成 |
| 前端: 路由/菜单 | 是/否 | 路由配置： | 未完成 / 已完成 |
| 第三方集成 | 是/否 | 接口/中间件： | 未完成 / 已完成 |
| 数据库迁移 (DDL) | 是/否 | 变更SQL： | 未完成 / 已完成 |

> **红线**：任一标记为"是"的层未完成，不得进入 Step 6（代码评审）。

---

## 1. api 层（后端）

- [ ] Request DTO（`CreateXxxRequest`、`UpdateXxxRequest`、`XxxPageRequest`）
- [ ] Response DTO（`XxxDto`）
- [ ] 枚举（`XxxStatusEnum`、`XxxTypeEnum`）
- [ ] 跨服务事件（`XxxCreatedEvent`）

---

## 2. dao 层（后端）

- [ ] PO（`XxxPo`）
- [ ] Mapper 接口（`XxxMapper extends BaseMapper<XxxPo>`）
- [ ] SQL 风格统一（优先 Wrapper API，复杂走 XML，禁止同一聚合根混用注解与 XML）
- [ ] XML（如有复杂 SQL）

---

## 3. domain 层（后端）

- [ ] 新增 `{aggr}` 子包
- [ ] `aggregate/{Aggr}.java` —— `create()` + `reconstitute()`，业务行为，事件注册
- [ ] `event/` —— 领域事件（实现 `DomainEvent`）
- [ ] `service/{Aggr}DomainService.java` —— 聚合根生命周期管理，PO ↔ Aggr 转换，领域事件发布

---

## 4. infrastructure 层（后端）

- [ ] `repository/{Aggr}Repository.java` —— 纯数据访问，返回 PO

---

## 5. app 层（后端）

- [ ] `application/{Aggr}ApplicationService.java` —— 用例编排，`@Transactional`，请求拆解/组装
- [ ] `controller/{Aggr}Controller.java` —— 全部 POST；路径 `/模块/资源/动作`（资源单数、动作用 `query`/`queryList`/`queryPage`/`create`/`update`/`delete`，禁 RESTful 复数/名词当动作）；返回 `DataResponse`；`@RequestBody` 直接 DTO/Map（禁自定义包装类）
- [ ] `domaineventhandler/{Aggr}EventHandler.java` —— 事件监听

---

## 6. 前端实现

- [ ] 页面/组件（.vue 文件）已创建或修改
- [ ] API 调用函数已对接（`src/api/` 下对应的函数）
- [ ] 路由配置已添加（`router/index.ts`）
- [ ] 菜单/权限配置已更新（如需）
- [ ] 前端编译通过（`npm run build` / `vite build`）
- [ ] 前后端数据格式一致（字段名 camelCase，响应结构 DataResponse）
- [ ] 空态/错误态/加载态已覆盖

---

## 7. 第三方集成

- [ ] 外部 API 调用已封装在独立的 `xxxClient.java` 中
- [ ] Feign 配置已添加（FeignClient + contextId）
- [ ] API 密钥/Token 已配置在 `application.yml` 的配置中心/环境变量（禁止硬编码）
- [ ] 调用超时和重试策略已配置
- [ ] 外部接口异常已捕获并转化为业务异常（防泄漏）
- [ ] 第三方回调已实现（如果涉及异步回调）

---

## 8. 数据库迁移

- [ ] 建表/改表 SQL 已验证（`init.sql` 或 Flyway migration）
- [ ] 索引已添加
- [ ] 数据迁移/初始化脚本已准备（如需要）
- [ ] 字段类型、默认值、非空约束已核对

---

## 9. 测试

- [ ] 单元测试覆盖聚合根核心行为（`create()`、`reconstitute()`、业务规则）
- [ ] 单元测试覆盖 ApplicationService 写用例主路径与异常路径
- [ ] **前端功能已人工验证**（页面加载、数据展示、交互操作、前端编译通过）
- [ ] 集成测试使用真实数据库验证 Repository 查询、保存、分页、批量操作
- [ ] **端到端数据流已验证**：前端页面→调用API→后端处理→返回数据→前端展示，全链路通畅
- [ ] 测试命名遵循 `shouldXxxWhenYyy` 或中文 `xx场景下应xx`
- [ ] 对 Repository / MQ / 第三方服务使用 Mock，不直接依赖外部系统跑单元测试
- [ ] 关键写操作覆盖幂等性测试

---

## 10. 全局

- [ ] 若新增领域或系统，在错误码段位文档中申请 `SS` / `D` 段位
- [ ] 若新增配置项，在 `application.yml` 中补充并给出默认值
- [ ] 若对外暴露接口，更新 API 文档（Swagger / Knife4j / README 视项目约定）
- [ ] 若新增或修改功能实现文档，在 `{{config.knowledge_dir}}/{{config.feature_impl_overview}}` 中登记或更新对应行；新增文档需记录新建原因
- [ ] **全栈关闭确认**：上述 [[#0. 全栈覆盖检查（强制）]] 中的「是否涉及」列已全部覆核，所有标记为"是"的层均已完成