---
name: implementation-checklist-rules
description: 新增聚合根实现 Checklist。定义 api/dao/domain/infrastructure/app 各层的实现顺序、检查项与测试要求。
---

# 新增聚合根实现 Checklist

> 新增聚合根时的实现顺序与检查清单。详细分层原则与领域模型设计见 `backend-ddd-layer-rules.md`，持久层与缓存规范见 `persistence-sql-cache-rules.md`，异常与错误码见 `error-handling-rules.md`。

---

## 1. api 层

- [ ] Request DTO（`CreateXxxRequest`、`UpdateXxxRequest`、`XxxPageRequest`）
- [ ] Response DTO（`XxxDto`）
- [ ] 枚举（`XxxStatusEnum`、`XxxTypeEnum`）
- [ ] 跨服务事件（`XxxCreatedEvent`）

---

## 2. dao 层

- [ ] PO（`XxxPo`）
- [ ] Mapper 接口（`XxxMapper extends BaseMapper<XxxPo>`）
- [ ] SQL 风格统一（优先 Wrapper API，复杂走 XML，禁止同一聚合根混用注解与 XML）
- [ ] XML（如有复杂 SQL）

---

## 3. domain 层

- [ ] 新增 `{aggr}` 子包
- [ ] `aggregate/{Aggr}.java` —— `create()` + `reconstitute()`，业务行为，事件注册
- [ ] `event/` —— 领域事件（实现 `DomainEvent`）
- [ ] `service/{Aggr}DomainService.java` —— 聚合根生命周期管理，PO ↔ Aggr 转换，领域事件发布

---

## 4. infrastructure 层

- [ ] `repository/{Aggr}Repository.java` —— 纯数据访问，返回 PO

---

## 5. app 层

- [ ] `application/{Aggr}ApplicationService.java` —— 用例编排，`@Transactional`，请求拆解/组装
- [ ] `controller/{Aggr}Controller.java` —— 全部 POST；路径 `/模块/资源/动作`（资源单数、动作用 `query`/`queryList`/`queryPage`/`create`/`update`/`delete`，禁 RESTful 复数/名词当动作）；返回 `DataResponse`；`@RequestBody` 直接 DTO/Map（禁自定义包装类）
- [ ] `domaineventhandler/{Aggr}EventHandler.java` —— 事件监听

---

## 6. 测试

- [ ] 单元测试覆盖聚合根核心行为（`create()`、`reconstitute()`、业务规则）
- [ ] 单元测试覆盖 ApplicationService 写用例主路径与异常路径
- [ ] 集成测试使用真实数据库验证 Repository 查询、保存、分页、批量操作
- [ ] 测试命名遵循 `shouldXxxWhenYyy` 或中文 `xx场景下应xx`
- [ ] 对 Repository / MQ / 第三方服务使用 Mock，不直接依赖外部系统跑单元测试
- [ ] 关键写操作覆盖幂等性测试

---

## 7. 全局

- [ ] 若新增领域或系统，在错误码段位文档中申请 `SS` / `D` 段位
- [ ] 若新增配置项，在 `application.yml` 中补充并给出默认值
- [ ] 若对外暴露接口，更新 API 文档（Swagger / Knife4j / README 视项目约定）
- [ ] 若新增或修改功能实现文档，在 `{{config.knowledge_dir}}/{{config.feature_impl_overview}}` 中登记或更新对应行；新增文档需记录新建原因

---