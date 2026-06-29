# 前端编程规范

本规范适用于 Vue3 + TypeScript + Vite + Pinia 前端工程。

---

## 1. 技术栈与目录约定

- **框架**：Vue 3（Composition API + `<script setup>`）
- **语言**：TypeScript（严格模式）
- **构建**：Vite
- **状态管理**：Pinia
- **UI 组件库**：Element Plus
- **样式**：SCSS，组件级 `scoped` 样式
- **HTTP 库**：axios（统一封装为 `request.ts`）

目录结构应保持：

```
src/
├── api/              # API 接口封装
├── assets/           # 静态资源
├── components/       # 公共组件
├── router/           # 路由配置
├── store/            # Pinia 状态
├── utils/            # 工具函数
└── views/            # 页面级组件
```

---

## 2. 接口调用规范（RPC 风格）

前后端统一采用 RPC 风格，所有请求均使用 **POST**。

### 2.1 Request 封装

- `src/utils/request.ts` 仅暴露 `post` 方法，禁止暴露 `get` / `put` / `delete`。
- `baseURL` 统一为 `/api/v1`。
- 请求拦截器自动注入 `Authorization` 与业务身份头（如 `X-Member-Id`）。
- 响应拦截器统一按 `code === '000000'` 判断成功，失败时自动 `ElMessage.error` 并返回 rejected Promise。

```ts
const request = {
  post<T = any>(url: string, data?: any, config?: AxiosRequestConfig): Promise<T> {
    return instance.post(url, data, config) as unknown as Promise<T>
  },
}
```

### 2.2 API 函数命名

| 操作类型 | 命名前缀 | 示例 |
|---|---|---|
| 查询单个 | `query` | `queryUser`、`queryNewsDetail` |
| 查询列表 | `queryXxxList` | `queryMenuList`、`queryRoleList` |
| 分页查询 | `queryXxxPage` | `queryUserPage`、`queryNewsPage` |
| 创建 | `create` | `createMenu`、`createSchedulerJob` |
| 更新 | `update` | `updateMenu`、`updateMemberProfile` |
| 删除 | `delete` | `deleteMenu`、`deleteChatSession` |

**禁止 RESTful / 资源复数风格命名**（本工程 RPC 风格、全 POST，不用 HTTP 动词语义）：

| ❌ 错误 | ✅ 正确 | 说明 |
|---|---|---|
| `getXxx` | `queryXxx` | 禁止 `get` 前缀，查询统一 `query` |
| `listXxx` | `queryXxxList` | 禁止 `list` 前缀 |
| `queryUsers`（复数 s） | `queryUserList` | 返回列表必须 `List` 后缀，禁止裸复数 |
| `queryUser`（实际返回数组/分页） | `queryUserList` / `queryUserPage` | 集合结果加 `List`，分页加 `Page`，禁止用单数名承载集合 |

### 2.3 API 路径

路径以 `/模块/资源/动作` 组织，**资源用单数**，动作用 `query` / `queryList` / `queryPage` / `queryDetail` / `create` / `update` / `delete` 等。例如：

```ts
'/system/menu/queryTree'
'/system/user/queryPage'
'/portal/news/queryList'
'/portal/profile/update'
```

**禁止 RESTful 风格路径**（不用资源复数、不用名词当动作、不用路径参数表达动作）：

| ❌ 错误 | ✅ 正确 | 说明 |
|---|---|---|
| `/xxx/item/groups` | `/xxx/item/group/queryList` | 列表必须有 `queryList`，资源单数 |
| `/xxx/catalog/list` | `/xxx/catalog/queryList` | 禁止 `list`，用 `queryList` |
| `/xxx/order/detail` | `/xxx/order/queryDetail` | 名词当动作禁止，用 `queryDetail` |
| `/xxx/profile/get` | `/xxx/profile/query` | 禁止 `get`，用 `query` |
| `/xxx/news/:id` | `/xxx/news/queryDetail` | 禁止 RESTful 路径参数 |

### 2.4 DTO / VO 字段命名

- 全部使用 **camelCase**，禁止使用 snake_case。
- 接口类型命名使用 `Request` / `Response` / `Item` 等语义化后缀。

```ts
export interface NewsItem {
  id: string
  title: string
  summary: string
  source: string
  publishTime: string
  sentiment: 'positive' | 'neutral' | 'negative'
  stockCodes?: string[]
}

export interface NewsPageRequest {
  category?: string
  keyword?: string
  page: number
  pageSize: number
}
```

---

## 3. Vue 组件规范

### 3.1 组件定义

- 页面组件放在 `views/` 下，公共组件放在 `components/` 下。
- 统一使用 `<script setup lang="ts">` + `<style lang="scss" scoped>`。
- 组件名使用 PascalCase，文件名为对应的小写或语义化命名（如 `index.vue`、`user-detail.vue`）。

### 3.2 响应式数据

- 简单状态使用 `ref`，表单/复杂对象使用 `reactive`。
- 类型标注优先使用泛型 `ref<T>(...)`。

### 3.3 模板规范

- 指令简写：`v-bind` 用 `:`，`v-on` 用 `@`。
- 循环必须提供稳定 `:key`。
- 避免在模板中写复杂表达式，优先使用 `computed`。

### 3.4 样式规范

- 组件根元素使用 `-page` 或对应语义类名。
- 颜色、字号等视觉 token 优先复用 Element Plus 变量或项目内统一色值。
- 移动端优先，桌面端通过 `@media (min-width: 1025px)` 等断点扩展。

---

## 4. 类型与命名规范

### 4.1 文件/目录命名

- 目录、Vue/TS 文件：小写，多单词用短横线连接（kebab-case）。
- 类型/接口：PascalCase。
- 变量/函数/属性：camelCase。
- 常量：全大写 + 下划线（如 `MAX_PAGE_SIZE`）。

### 4.2 函数命名

- 事件处理：以 `handle` 开头，如 `handleSearch`、`handlePageChange`。
- 跳转/导航：以 `goTo` 开头，如 `goToDetail`、`goToNewsDetail`。
- 纯工具函数：动词或动词短语，如 `formatDate`、`sentimentText`。

### 4.3 禁止事项

- 禁止 API 函数使用 `get` / `list` 前缀（查询统一 `query`，列表 `queryXxxList`，分页 `queryXxxPage`）。
- 禁止 URL 使用 RESTful 风格（资源复数、`list`/`get`/`detail` 名词当动作、路径参数表达动作）。
- 禁止在 API 类型与页面数据中使用 snake_case 字段。
- 禁止在 `request.ts` 中暴露非 POST 方法。
- 禁止在组件内直接使用 `axios` 实例，统一通过 `@/utils/request`。
- 禁止在 `views/` 下沉淀可复用业务组件。

---

## 5. 状态管理规范

- 使用 Pinia，store 文件统一放在 `src/store/`。
- 用户登录态、token、profile 等全局状态放入 `useUserStore`。
- `localStorage` 键名使用下划线小写（如 `member_token`、`member_id`），仅用于本地持久化，不作为 DTO 字段。

---

## 6. 路由规范

- 路由路径统一小写，多单词用短横线连接。
- 详情页参数使用 `:id`，如 `/news-detail/:id`。
- 路由守卫在 `router/index.ts` 中统一处理登录校验。

---

## 7. 代码质量

- 启用 TypeScript 严格模式与 ESLint，提交前运行类型检查。
- 优先使用 Element Plus 组件，避免重复造轮子。
- 复杂页面按功能拆分为小组件，保持单文件行数可控。
- 删除不再使用的 import、变量与函数。
