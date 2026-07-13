# 代码规范库

> 本文件整合了项目中所有编码规范，每条规则具有唯一 ID，供 CR（Code Review）报告引用。
>
> 每条规则的检查方式、严重等级、违反范例与正确范例均在一处明确定义。

***

## 违规规则项速查表

| 分类      | 规则 ID       | 规则简述                                        | 等级 |
| ------- | ----------- | ------------------------------------------- | -- |
| 基础编码    | CR-RULE-002 | 禁止 `any` 类型                                 | S  |
| 基础编码    | CR-RULE-003 | `async` 配套 `try/catch/finally`              | S  |
| 基础编码    | CR-RULE-004 | Props 禁止 `Record<string, any>`              | S  |
| 基础编码    | CR-RULE-005 | 魔法值定义为具名常量                                | M  |
| 基础编码    | CR-RULE-006 | 生命周期钩子内只调方法，不写业务逻辑                          | M  |
| 基础编码    | CR-RULE-007 | 方法注释含 `@function` / `@param` / `@return`    | M  |
| 基础编码    | CR-RULE-008 | HTML 区块用 `<!-- S -->` / `<!-- E -->` 标记     | M  |
| 基础编码    | CR-RULE-009 | 非直观逻辑必须补注释                                  | M  |
| 基础编码    | CR-RULE-010 | 禁止 Options API（`data` / `methods` 选项）       | S  |
| 基础编码    | CR-RULE-011 | 禁止使用拼音和中文命名                                 | M  |
| 基础编码    | CR-RULE-012 | `v-for` 必须设置 `key`                          | S  |
| 基础编码    | CR-RULE-013 | 模板中使用简单表达式，复杂逻辑用计算属性/方法                     | M  |
| 基础编码    | CR-RULE-014 | 标签顺序保持 `template` → `script` → `style`      | M  |
| 基础编码    | CR-RULE-015 | Script 声明顺序规范                               | M  |
| 基础编码    | CR-RULE-016 | 避免手动操作 DOM，使用数据驱动                           | M  |
| 基础编码    | CR-RULE-017 | 使用解构赋值获取对象属性                                | M  |
| 基础编码    | CR-RULE-018 | 方法命名使用动词+名词驼峰                               | M  |
| 基础编码    | CR-RULE-019 | 事件方法使用 `on` / `handle` 开头                   | M  |
| 基础编码    | CR-RULE-020 | 接口注释使用 `@api`                               | M  |
| 基础编码    | CR-RULE-001 | 建议使用箭头函数，但不强制                              | R  |
| API 与数据 | CR-RULE-101 | 使用 `request` 封装，禁止直接 `axios`                | S  |
| API 与数据 | CR-RULE-102 | API 类型定义在 `api/{domain}/model.d.ts`         | S  |
| API 与数据 | CR-RULE-103 | 禁止组件内定义 API 类型                              | M  |
| API 与数据 | CR-RULE-104 | 缓存 key 使用 `constants/cache.ts`              | M  |
| API 与数据 | CR-RULE-105 | 纯 `return request(...)` 不需要 `async`         | M  |
| API 与数据 | CR-RULE-106 | 分页参数统一使用 `API.PageParamsQuery<T>`           | M  |
| API 与数据 | CR-RULE-107 | 接口字段命名统一规范                                  | M  |
| API 与数据 | CR-RULE-108 | 接口返回结构（errorCode/errorMsg/data/success）规范   | M  |
| 国际化     | CR-RULE-201 | 所有用户可见文案走 `$t()` / `t()`                    | S  |
| 国际化     | CR-RULE-202 | 固定配置翻译使用 `translateOptions()` 而非 `computed` | M  |
| 国际化     | CR-RULE-203 | 翻译需在生命周期 + `watch(locale)` 中执行  | M  |
| 国际化     | CR-RULE-204 | 表格列 `title` 使用国际化 key，不写死文案                 | S  |
| 样式      | CR-RULE-301 | 优先使用原子类框架                           | M  |
| 样式      | CR-RULE-302 | 自写样式必须 `scoped`                             | M  |
| 样式      | CR-RULE-303 | 颜色引用 Less 变量，禁止硬编码色值                        | M  |
| 样式      | CR-RULE-304 | 状态颜色复用全局配置                                  | M  |
| 样式      | CR-RULE-305 | CSS 选择器避免标签名和 ID 选择器                        | M  |
| 样式      | CR-RULE-306 | Less/SCSS 不超过 3 层嵌套                         | M  |
| 组件与模板   | CR-RULE-401 | 统一表格组件替代直接使用 UI 库 Table                  | M  |
| 组件与模板   | CR-RULE-402 | 弹窗 `visible` 由父组件 props 传入                  | M  |
| 组件与模板   | CR-RULE-403 | 按钮权限使用权限指令                          | M  |
| 组件与模板   | CR-RULE-404 | 分页逻辑使用统一表格组件体系                        | M  |
| 组件与模板   | CR-RULE-405 | 列定义集中到 `constants/tableColumns.ts`          | M  |
| 组件与模板   | CR-RULE-406 | 页面常量放在 `views/{Module}/constant.ts`         | M  |
| 组件与模板   | CR-RULE-407 | 弹窗组件关闭时只 emit，不直接修改 `visible`               | M  |
| 组件与模板   | CR-RULE-408 | 组件 Props 使用运行时声明 + PropType                 | M  |
| 组件与模板   | CR-RULE-409 | Emits 使用字符串数组声明                             | M  |
| 组件与模板   | CR-RULE-410 | 弹窗数据通过 props 传入（如 `orderId`）                | M  |
| 组件与模板   | CR-RULE-411 | 新增独立业务域应新建模块，不强行挂到旧目录                       | M  |
| 组件与模板   | CR-RULE-412 | 组件名多单词 + 大驼峰 + Fs 前缀                        | M  |
| 组件与模板   | CR-RULE-413 | 子组件以父组件名为前缀                                 | M  |
| 组件与模板   | CR-RULE-414 | Prop 定义详细（类型/注释/required/validator）         | M  |
| 组件与模板   | CR-RULE-415 | 特性元素多时主动换行                                  | M  |
| 组件与模板   | CR-RULE-416 | `v-show` 与 `v-if` 按场景选择                     | M  |
| 架构      | CR-RULE-501 | 路由跳转使用统一封装方法                   | M  |
| 架构      | CR-RULE-502 | Store 外不直接操作 `localStorage`           | S  |
| 架构      | CR-RULE-503 | 模块间依赖方向是否正确                                 | R  |
| 架构      | CR-RULE-504 | 新增功能是否复用现有组件/模式                             | R  |
| 架构      | CR-RULE-505 | 代码变更不超出需求范围                                 | M  |
| 流程      | CR-RULE-901 | 变更范围不超出需求                                   | M  |
| 流程      | CR-RULE-902 | 未使用的导入/变量已清理                                | M  |

***

## 整改优化项速查表

| 分类    | 优化项 ID     | 优化项简述                             | 优先级  |
| ----- | ---------- | --------------------------------- | ---- |
| 页面与交互 | CR-OPT-001 | 异步请求操作加 loading / 防重复提交 | 建议整改 |
| 页面与交互 | CR-OPT-002 | 表单状态初始化、回显、重置逻辑分离                 | 建议整改 |
| 页面与交互 | CR-OPT-003 | 生命周期只做调度，不堆业务逻辑                   | 建议整改 |
| 页面与交互 | CR-OPT-004 | 条件渲染和权限控制集中收敛                     | 建议整改 |
| 页面与交互 | CR-OPT-005 | 重复模板/样式/枚举映射需抽取复用                 | 建议整改 |
| 可维护性  | CR-OPT-101 | 模板复杂表达式和大段 slot 提升可读性             | 建议整改 |
| 可维护性  | CR-OPT-102 | 样式作用域、层级和无用样式清理                   | 建议整改 |
| 可维护性  | CR-OPT-103 | 国际化 key 结构统一，避免重复词条               | 建议整改 |
| 可维护性  | CR-OPT-104 | 方法职责拆分和可复用性提升                      | 建议整改 |

***

## 违规规则项详细说明

### CR-RULE-002：禁止 `any` 类型

- **类别**: 基础编码规范
- **等级**: 🔴 S（Blocking）
- **来源**: AGENTS.md → 类型
- **检查方式**: 人工审查 + `pnpm typecheck`
- **描述**: 所有参数、返回值、Props 禁止使用 `any`；`ref<any>` / `reactive<any>` 仅简单场景允许，但应优先使用具体类型

```ts
// ✅ 正确
const dataSource = ref<API.OrderListItem[]>([])

// ❌ 错误
const dataSource = ref<any>([])
```

***

### CR-RULE-003：`async` 配套 `try/catch/finally`

- **类别**: 基础编码规范
- **等级**: 🔴 S（Blocking）
- **来源**: AGENTS.md → async/await
- **检查方式**: 人工审查
- **描述**: 使用 `async` 的函数必须配套 `try/catch/finally`

```ts
// ✅ 正确
const getList = async () => {
  tableLoading.value = true
  try {
    const res = await getXxxServer(params)
    dataSource.value = res.records
  } catch (error) {
    console.log(error)
  } finally {
    tableLoading.value = false
  }
}

// ❌ 错误
const getList = async () => {
  const res = await getXxxServer(params)
  dataSource.value = res.records
}
```

***

### CR-RULE-004：Props 禁止 `Record<string, any>`

- **类别**: 基础编码规范
- **等级**: 🔴 S（Blocking）
- **来源**: AGENTS.md → 类型
- **检查方式**: 人工审查
- **描述**: 组件 Props 必须使用具体 `interface` 或 `type`，禁止 `Record<string, any>` 或 `any`

```ts
// ✅ 正确
interface Props {
  visible: boolean
  userId?: string
}
defineProps<Props>()

// ❌ 错误
defineProps<Record<string, any>>()
```

***

### CR-RULE-005：魔法值定义为具名常量

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: AGENTS.md → 魔法值
- **检查方式**: 人工审查
- **描述**: 禁止硬编码数字/字符串字面量；枚举值定义为具名常量对象

```ts
// ✅ 正确
export const APPLY_ROLE = { self: 1, agent: 2 }
if (role === APPLY_ROLE.agent)

// ❌ 错误
if (role === 2)
```

***

### CR-RULE-006：生命周期钩子内只调方法

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: AGENTS.md → 生命周期
- **检查方式**: 人工审查
- **描述**: 生命周期钩子（`onBeforeMount` / `onMounted` 等）内不要直接写业务逻辑，应调用抽取后的方法

```ts
// ✅ 正确
onBeforeMount(() => {
  getList()
  translateOptions()
})

// ❌ 错误
onBeforeMount(async () => {
  const res = await getXxxServer(params)
  dataSource.value = res.records
})
```

***

### CR-RULE-007：方法注释含 `@function` / `@param` / `@return`

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: AGENTS.md → 注释
- **检查方式**: 人工审查
- **描述**: 所有方法必须写 JSDoc 注释，包含 `@function`、`@param`（类型和描述）、`@return`

```ts
/**
 * @function 获取转账记录列表
 * @param { API.PageParamsQuery<API.XxxParams> } data 查询参数
 * @return { Promise<API.TableListResult<API.XxxItem>> } 分页列表
 */
export const getXxxListServer = (data) => request({ url: '/xxx', data })
```

***

### CR-RULE-008：HTML 区块用 `<!-- S -->` / `<!-- E -->` 标记

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: AGENTS.md → 注释
- **检查方式**: 人工审查
- **描述**: 模板中每个独立区块必须用 `<!-- S 区块名 -->` / `<!-- E 区块名 -->` 标记首尾

```vue
<!-- S 查询区域 -->
<a-form>...</a-form>
<!-- E 查询区域 -->
```

***

### CR-RULE-009：非直观逻辑必须补注释

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: AGENTS.md → 注释
- **检查方式**: 人工审查
- **描述**: 状态映射、轮询、草稿恢复、字段联动、驳回回填等非直观逻辑必须补充注释说明

***

### CR-RULE-010：禁止 Options API

- **类别**: 基础编码规范
- **等级**: 🔴 S（Blocking）
- **来源**: bug-fix.md → 反面模式 #5
- **检查方式**: 人工审查
- **描述**: 禁止在 `<script setup>` 中使用 Options API（`data`、`methods`、`computed` 选项）；必须使用 Composition API（`ref`、`reactive`、`computed`）

***

### CR-RULE-011：禁止使用拼音和中文命名

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 1.1.1
- **检查方式**: 人工审查
- **描述**: 变量、函数、文件、目录命名严禁使用拼音和中文，必须使用英文全拼或常识性缩写（如 DNA、CPU）

```ts
// ✅ 正确
const userList = []
const getDetail = () => {}

// ❌ 错误
const yongHuBiao = []
const getXiangQing = () => {}
```

***

### CR-RULE-012：`v-for` 必须设置 `key`

- **类别**: 基础编码规范
- **等级**: 🔴 S（Blocking）
- **来源**: 前端规范.md → 2.1.5
- **检查方式**: 人工审查
- **描述**: 使用 `v-for` 渲染列表时，必须为每个元素设置唯一的 `key` 值

```vue
<!-- ✅ 正确 -->
<div v-for="item in list" :key="item.id">{{ item.name }}</div>

<!-- ❌ 错误 -->
<div v-for="item in list">{{ item.name }}</div>
```

***

### CR-RULE-013：模板中使用简单表达式

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 2.1.2
- **检查方式**: 人工审查
- **描述**: 组件模板应该只包含简单的表达式，复杂的表达式必须重构为计算属性或方法

```ts
// ✅ 正确
const normalizedFullName = computed(() => {
  return fullName.value.split(' ').map(w => w[0].toUpperCase() + w.slice(1)).join(' ')
})

// ❌ 错误
// <p>{{ fullName.split(' ').map(function (w) { return w[0]... }).join(' ') }}</p>
```

***

### CR-RULE-014：标签顺序保持 `template` → `script` → `style`

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 2.1.4
- **检查方式**: 人工审查
- **描述**: 单文件组件标签顺序必须固定为 `<template>` → `<script>` → `<style>`

```vue
<!-- ✅ 正确 -->
<template>...</template>
<script lang="ts" setup>...</script>
<style lang="less" scoped>...</style>

<!-- ❌ 错误 -->
<template>...</template>
<style>...</style>
<script>...</script>
```

***

### CR-RULE-015：Script 声明顺序规范

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 2.2.4
- **检查方式**: 人工审查
- **描述**: `<script setup>` 内的声明顺序：导入 → Hook → 状态 → 参数 → onBeforeMount → watch → 方法

```ts
// ✅ 顺序
1. 第三方库导入
2. 项目内部导入 — 常量
3. 项目内部导入 — 组件
4. 项目内部导入 — API / Hooks / Utils
5. Hook 调用
6. 响应式状态声明（所有 ref / reactive）
7. 请求参数（commonParams + requestParam）
8. `onBeforeMount` 生命周期
9. `watch` 监听
10. 方法定义（按业务流程排序）
```

***

### CR-RULE-016：避免手动操作 DOM

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 2.2.3
- **检查方式**: 人工审查
- **描述**: 使用 Vue 的数据驱动机制更新 DOM，不到万不得已不要手动操作 DOM（增删改 dom 元素、直接修改样式、添加原生事件等）

***

### CR-RULE-017：使用解构赋值获取对象属性

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 1.6.7
- **检查方式**: 人工审查
- **描述**: 当需要使用对象的多个属性时，必须使用解构赋值

```ts
// ✅ 正确
const { firstName, lastName } = user

// ❌ 错误
const firstName = user.firstName
const lastName = user.lastName
```

***

### CR-RULE-018：方法命名使用动词+名词驼峰

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 1.1.5
- **检查方式**: 人工审查
- **描述**: 方法命名使用驼峰法，以动词+名词形式（如 `addUser`、`getDetail`、`deleteRecord`）

常用动词组合：add/remove、get/set、create/destroy、open/close、load/save、import/export、bind/unbind、select/mark、insert/delete、find/search、upload/download

***

### CR-RULE-019：事件方法使用 `on` / `handle` 开头

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 1.1.5
- **检查方式**: 人工审查
- **描述**: 事件处理方法使用 `on` 或 `handle` 开头命名

```ts
// ✅ 正确
const onSubmit = () => {}
const handleSizeChange = (val: number) => {}
const handlePageChange = (page: number, size: number) => {}

// ❌ 错误
const submit = () => {}
const sizeChange = (val: number) => {}
```

***

### CR-RULE-020：接口注释使用 `@api`

- **类别**: 基础编码规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 1.2.4
- **检查方式**: 人工审查
- **描述**: API 接口定义的注释使用 `@api` 标签，方法定义使用 `@function`

```ts
/**
 * @api 查询转账记录列表
 * @param { API.PageParamsQuery<API.XxxParams> } data
 * @return { Promise<API.TableListResult<API.XxxItem>> }
 */
export const getXxxListServer = (data) => request({ url: '/xxx', data })
```

***

### CR-RULE-001：箭头函数声明

- **类别**: 基础编码规范
- **等级**: 🔵 R（Suggestion）
- **来源**: AGENTS.md → 函数
- **检查方式**: 人工审查
- **描述**: 建议使用箭头函数 `const xxx = () => {}`，风格上更契合 Composition API 与 `<script setup>` 的书写习惯，但不强制要求；使用 `function` 声明不会造成 CR 阻塞，仅在风格不一致时作为建议提醒；API 层接口仍推荐箭头函数写法

```ts
// ✅ 推荐
export const getXxxServer = (data: API.XxxParams) => request({ url: '/xxx', data })

// 🔄 亦可接受
export function getXxxServer(data: API.XxxParams) {
  return request({ url: '/xxx', data })
}
```

***

### CR-RULE-101：使用 `request` 封装

- **类别**: API 与数据规范
- **等级**: 🔴 S（Blocking）
- **来源**: bug-fix.md → 反面模式 #1
- **检查方式**: 人工审查
- **描述**: 所有 HTTP 请求必须通过 `import { request } from '@/utils/http'`，禁止直接使用 `axios` 或 `fetch`

***

### CR-RULE-102：API 类型定义在 `model.d.ts`

- **类别**: API 与数据规范
- **等级**: 🔴 S（Blocking）
- **来源**: bug-fix.md → 反面模式 #7
- **检查方式**: 人工审查
- **描述**: API 入参和出参类型必须定义在 `api/{domain}/model.d.ts` 的 `API` 命名空间下

***

### CR-RULE-103：禁止组件内定义 API 类型

- **类别**: API 与数据规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #7
- **检查方式**: 人工审查
- **描述**: 禁止在组件文件内直接定义 API 请求/响应类型

***

### CR-RULE-104：缓存 key 使用常量

- **类别**: API 与数据规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #2
- **检查方式**: 人工审查
- **描述**: 所有缓存 key 必须使用 `@/constants/cache.ts` 中定义的常量，禁止硬编码字符串

***

### CR-RULE-105：纯 `return request(...)` 不需要 `async`

- **类别**: API 与数据规范
- **等级**: 🟡 M（Warning）
- **来源**: api-development.md → 0.3
- **检查方式**: 人工审查
- **描述**: 如果函数只是 `return request(...)` 且不处理结果，不需要 `async/await`

```ts
// ✅ 正确
export const getXxxServer = (data: API.XxxParams) => request({ url: '/xxx', data })

// ❌ 错误
export const getXxxServer = async (data: API.XxxParams) => request({ url: '/xxx', data })
```

***

### CR-RULE-106：分页参数统一使用 `API.PageParamsQuery<T>`

- **类别**: API 与数据规范
- **等级**: 🟡 M（Warning）
- **来源**: api-development.md → 3.1
- **检查方式**: 人工审查
- **描述**: 列表查询的分页参数必须使用 `API.PageParamsQuery<T>` 泛型类型

***

### CR-RULE-107：接口字段命名统一规范

- **类别**: API 与数据规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 3.1.1
- **检查方式**: 人工审查
- **描述**: 接口参数/响应字段命名遵守统一约定：分页使用 `page/pageSize/totalCount`、列表使用 `list`、时间使用 `time/startTime/endTime`、状态使用 `status`、结果使用 `success/errorCode/errorMsg/data`

| 场景   | 标准字段                                   |
| ---- | -------------------------------------- |
| 分页请求 | `page`、`pageSize`                      |
| 列表响应 | `list`、`totalCount`                    |
| 时间   | `time`（单个）、`startTime`/`endTime`（范围）   |
| 状态   | `status`（0 无业务含义）                      |
| 密码   | `password`、`oldPassword`、`newPassword` |

***

### CR-RULE-108：接口返回结构规范

- **类别**: API 与数据规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 3.1.2
- **检查方式**: 人工审查
- **描述**: 接口统一返回结构应包含 `errorCode`（错误码）、`errorMsg`（错误信息）、`data`（数据对象）、`success`（业务成功标志）

```ts
type ApiResponse<T = any> = {
  errorCode: number
  errorMsg: string
  data: T
  success: boolean
}
```

***

### CR-RULE-201：所有用户可见文案走 i18n

- **类别**: 国际化规范
- **等级**: 🔴 S（Blocking）
- **来源**: bug-fix.md → 反面模式 #4
- **检查方式**: 人工审查
- **描述**: 页面标题、按钮、提示、表格表头、下拉选项、空态文案等所有展示给用户的文本必须使用 `$t()` / `t()` 包裹

***

### CR-RULE-202：固定配置翻译使用统一翻译函数

- **类别**: 国际化规范
- **等级**: 🟡 M（Warning）
- **来源**: correction-rules.md → 国际化约束
- **检查方式**: 人工审查
- **描述**: 固定下拉选项、表格表头等翻译数据，禁止使用 `computed`，必须使用项目封装的翻译函数（如 `translateOptions()`），并在组件挂载前与 `watch(locale)` 中执行

***

### CR-RULE-203：翻译需在组件挂载前 + `watch(locale)` 中执行

- **类别**: 国际化规范
- **等级**: 🟡 M（Warning）
- **来源**: correction-rules.md → 国际化约束
- **检查方式**: 人工审查
- **描述**: 翻译函数必须在 `onBeforeMount` 中调用，并监听 `locale` 变化重新执行

***

### CR-RULE-204：表格列 `title` 使用国际化 key

- **类别**: 国际化规范
- **等级**: 🔴 S（Blocking）
- **来源**: correction-rules.md → 表格列定义规范
- **检查方式**: 人工审查
- **描述**: 所有表格列定义的 `title` 字段必须使用国际化 key，禁止直接写中文或英文文案

***

### CR-RULE-301：优先使用原子类框架

- **类别**: 样式规范
- **等级**: 🟡 M（Warning）
- **来源**: project-config.md → 二
- **检查方式**: 人工审查
- **描述**: 常规布局、间距、对齐、字号、边框等优先使用原子类框架，只有原子类无法表达时才写 `<style>`

***

### CR-RULE-302：自写样式必须 `scoped`

- **类别**: 样式规范
- **等级**: 🟡 M（Warning）
- **来源**: project-config.md → 二
- **检查方式**: 人工审查
- **描述**: 必须写 `<style>` 时必须写成 `<style lang="less" scoped>`

***

### CR-RULE-303：颜色引用 Less 变量

- **类别**: 样式规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #3
- **检查方式**: 人工审查
- **描述**: 颜色、间距等必须引用 `variables.less` 中定义的变量，禁止硬编码色值（如 `color: #2058e4`）

***

### CR-RULE-304：状态颜色复用全局配置

- **类别**: 样式规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #16
- **检查方式**: 人工审查
- **描述**: 状态展示优先复用全局状态类型颜色配置，`Tag` 的 `color` 优先取 `text`，状态文字颜色优先取 `color`

***

### CR-RULE-305：CSS 选择器避免标签名和 ID 选择器

- **类别**: 样式规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 1.4.2
- **检查方式**: 人工审查
- **描述**: CSS 选择器中避免使用标签名和 ID 选择器以防止污染全局样式；推荐使用 class 选择器

```less
// ✅ 正确
.my-header { padding-bottom: 0; }

// ❌ 错误
#header { padding-bottom: 0; }
span { color: red; }
```

***

### CR-RULE-306：Less/SCSS 不超过 3 层嵌套

- **类别**: 样式规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 1.5.2
- **检查方式**: 人工审查
- **描述**: Less/SCSS 嵌套层级不超过 3 层，超过时应考虑抽取为独立 class

```less
// ✅ 正确
.main-title {
  .name { color: #fff; }
}

// ❌ 错误（嵌套过深）
.main {
  .title {
    .name {
      .text { color: #fff; }
    }
  }
}
```

***

### CR-RULE-401：使用统一表格组件

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #6
- **检查方式**: 人工审查
- **描述**: 表格须使用项目封装的统一表格组件替代直接使用 UI 库 Table

***

### CR-RULE-402：弹窗 `visible` 由父组件传入

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #11
- **检查方式**: 人工审查
- **描述**: 弹窗组件的 `visible` 由父组件通过 props 传入，子组件不自行管理

***

### CR-RULE-403：按钮权限使用权限指令

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #14
- **检查方式**: 人工审查
- **描述**: 按钮级权限使用项目封装的权限指令（如 `v-auth="'模块:操作'"`），禁止在模板中用函数判断

***

### CR-RULE-404：分页逻辑使用统一表格组件体系

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #12
- **检查方式**: 人工审查
- **描述**: 分页逻辑使用项目封装的统一表格组件的 `getList` + 分页组件 + `resetCurrent` 体系，禁止自行实现

***

### CR-RULE-405：列定义集中到 `tableColumns.ts`

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: correction-rules.md → 表格列定义规范
- **检查方式**: 人工审查
- **描述**: 所有表格列定义必须统一放在 `@/constants/tableColumns.ts` 中集中管理，禁止在组件内部直接定义

***

### CR-RULE-406：页面常量放在 `views/{Module}/constant.ts`

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #8
- **检查方式**: 人工审查
- **描述**: 页面专属常量统一放在大模块根目录的 `views/{Module}/constant.ts`，同一模块子页面共享，不在子页面目录单独创建

***

### CR-RULE-407：弹窗关闭时只 emit

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: new-component.md → 七
- **检查方式**: 人工审查
- **描述**: 弹窗关闭时只 `emit('close')`，不直接修改 `visible`

***

### CR-RULE-408：Props 使用运行时声明

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: new-component.md → 三
- **检查方式**: 人工审查
- **描述**: 组件 Props 使用运行时声明（`defineProps({ ... })`），复杂类型使用 `PropType<T>` 断言

***

### CR-RULE-409：Emits 使用字符串数组

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: new-component.md → 四
- **检查方式**: 人工审查
- **描述**: Emits 使用字符串数组声明（`defineEmits(['close', 'submit'])`）

***

### CR-RULE-410：弹窗数据通过 props 传入

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: new-page.md → 五
- **检查方式**: 人工审查
- **描述**: 弹窗所需的数据（如 `orderId`、`record`）通过 props 传入，不在弹窗组件内自行获取

***

### CR-RULE-411：新模块应新建目录

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: new-page.md → 一
- **检查方式**: 人工审查
- **描述**: 新增独立业务域应新建 `views/{NewModule}` 模块，不强行挂到现有旧模块目录下

***

### CR-RULE-412：组件名多单词 + 大驼峰 + Fs 前缀

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 2.1.1 + new-component.md
- **检查方式**: 人工审查
- **描述**: 组件名由多个单词组成（≥2），使用大驼峰命名；基础组件使用约定前缀（如 `App`）；单文件组件在目录中以 `index.vue` 命名

```plain
// ✅ 正确
components/
  TodoList/
    index.vue
    index.scss
  TodoListItem.vue
  AppTable/
    index.vue

// ❌ 错误
components/
  TodoList.vue
  UProfOpts.vue  // 使用了缩写
```

***

### CR-RULE-413：子组件以父组件名为前缀

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 2.1.1
- **检查方式**: 人工审查
- **描述**: 和父组件紧密耦合的子组件应以父组件名作为前缀命名

```plain
// ✅ 正确
components/
  TodoList/
    index.vue
  TodoListItem.vue        // 以 TodoList 为前缀
  TodoListFooter.vue

// ❌ 错误
components/
  TodoList/
    index.vue
  Item.vue
  Footer.vue
```

***

### CR-RULE-414：Prop 定义详细（类型/注释/required/validator）

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 2.1.1
- **检查方式**: 人工审查
- **描述**: Prop 定义必须：使用 camelCase、指定类型、加注释说明含义、提供 `required` 或 `default`、有业务需要时加 `validator` 验证

```ts
// ✅ 正确
defineProps({
  status: {
    type: String,
    required: true,
    validator: (value: string) => ['succ', 'info', 'error'].includes(value)
  },
  userLevel: {
    type: String,
    default: 'normal'
  }
})

// ❌ 错误
defineProps({ status: String })
```

***

### CR-RULE-415：特性元素多时主动换行

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 2.1.1
- **检查方式**: 人工审查
- **描述**: 当组件特性元素（attributes）较多时，应主动换行以提高可读性

```vue
<!-- ✅ 正确 -->
<my-component
  foo="a"
  bar="b"
  baz="c"
/>

<!-- ❌ 错误 -->
<my-component foo="a" bar="b" baz="c" />
```

***

### CR-RULE-416：`v-show` 与 `v-if` 按场景选择

- **类别**: 组件与模板规范
- **等级**: 🟡 M（Warning）
- **来源**: 前端规范.md → 2.1.6
- **检查方式**: 人工审查
- **描述**: 运行时需要频繁切换使用 `v-show`；运行时条件很少改变使用 `v-if`

```vue
<!-- ✅ 频繁切换用 v-show -->
<div v-show="isVisible">频繁切换内容</div>

<!-- ✅ 条件少改变用 v-if -->
<div v-if="isAdmin">管理员可见</div>
```

***

### CR-RULE-501：路由跳转使用统一封装方法

- **类别**: 架构规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 反面模式 #13
- **检查方式**: 人工审查
- **描述**: 路由跳转使用项目封装的统一路由方法（如 `routeChange()`），禁止直接调用 `router.push`

***

### CR-RULE-502：Store 外不直接操作 `localStorage`

- **类别**: 架构规范
- **等级**: 🔴 S（Blocking）
- **来源**: bug-fix.md → 反面模式 #10
- **检查方式**: 人工审查
- **描述**: 禁止在状态管理 Store（如 Pinia）外直接操作 `localStorage`，应通过封装的工具方法并使用常量 key

***

### CR-RULE-503：模块间依赖方向正确

- **类别**: 架构规范
- **等级**: 🔵 R（Suggestion）
- **来源**: bug-fix.md → 代码审查要点
- **检查方式**: 人工审查
- **描述**: 模块间的依赖方向是否合理，是否存在跨模块耦合（如 A 模块引入了 B 模块的 API）

***

### CR-RULE-504：复用现有组件/模式

- **类别**: 架构规范
- **等级**: 🔵 R（Suggestion）
- **来源**: 人工审查经验
- **检查方式**: 人工审查
- **描述**: 新增功能时是否优先复用项目已有的组件和模式（如统一表格、弹窗等）

***

### CR-RULE-505：代码变更不超出范围

- **类别**: 架构规范
- **等级**: 🟡 M（Warning）
- **来源**: 人工审查经验
- **检查方式**: 人工审查
- **描述**: 变更只改被明确要求的部分，不扩大修改范围

***

### CR-RULE-901：变更范围不超出需求

- **类别**: 流程与管理规范
- **等级**: 🟡 M（Warning）
- **来源**: AGENTS.md → 回复与协作规则
- **检查方式**: 人工审查
- **描述**: 没有明确要求的内容不要写，只改被明确要求的部分

***

### CR-RULE-902：未使用的导入/变量已清理

- **类别**: 流程与管理规范
- **等级**: 🟡 M（Warning）
- **来源**: bug-fix.md → 检查清单
- **检查方式**: 人工审查 + ESLint（`no-unused-vars` 规则）
- **描述**: 提交前应清理未使用的导入和变量

***

## 整改优化项详细说明

### CR-OPT-001：异步流程统一 loading / disabled / 防重复提交

- **分类**: 页面与交互优化
- **优先级**: 建议整改
- **检查方式**: 人工审查

**适用范围**：仅针对**会发起 API 请求的操作**，包括新增/编辑/删除提交、导出下载、列表查询等。

**不适用场景**（无需 loading / 防重复）：
- 打开弹窗（`visible = true`）
- 表单步骤切换（上一步/下一步）
- Tab/标签页切换
- 展开/收起面板
- 纯客户端校验不涉及网络请求的操作

**判断信号**：
  - 点击提交/导出/下载（含异步请求）后仍可连续触发
  - 请求进行中没有 loading 或禁用态
  - 多个异步请求共用一个 loading 状态，导致界面反馈混乱
  - loading 变量命名不统一（项目中 `loading` / `submitLoading` / `btnLoading` / `tableLoading` 并存）
  - 批量操作、联动请求缺少独立的 loading 态

**整改要求**：
  - 带 API 请求的提交类操作必须防重复触发
  - 请求中按钮需 `:loading` 或 `:disabled`
  - 列表加载、保存提交、导出下载状态分开管理
  - loading 变量命名保持语义一致（如 `tableLoading`、`submitLoading`、`exportLoading`）

***

### CR-OPT-002：表单状态初始化、回显、重置逻辑分离

- **分类**: 页面与交互优化
- **优先级**: 建议整改
- **检查方式**: 人工审查
- **判断信号**:
  - 新增/编辑共用表单时状态相互污染
  - 弹窗关闭后再次打开仍残留上次数据
  - 编辑回显逻辑和默认值初始化混在一起
  - `watch` 中修改表单字段导致隐式数据联动
  - 校验时机混乱（提交时才加 `required`，但输入时没反馈）
  - 表格筛选条件重置后未清空关联选项
- **整改要求**:
  - 初始化、回显、重置三套逻辑明确分开
  - 弹窗关闭时清理状态（`onClose` / `afterClose`）
  - 新增/编辑切换时禁止复用脏数据
  - 筛选条件的级联联动（如省→市）重置时一并清空

***

### CR-OPT-003：生命周期只做调度，不堆业务逻辑

- **分类**: 页面与交互优化
- **优先级**: 建议整改
- **检查方式**: 人工审查
- **判断信号**:
  - `onMounted` / `onBeforeMount` 中直接写超过 5 行业务逻辑
  - 初始化顺序依赖阅读生命周期内部实现才能理解
  - 条件判断、多个请求、回显都堆在生命周期里
- **整改要求**:
  - 生命周期内只调用方法，不直接写业务逻辑（与 CR-RULE-006 一致，此处细化为行数阈值和拆分标准）
  - 初始化步骤拆为 `initXxx` / `fetchXxx` 之类的具名方法
  - 初始化顺序需可读、可追踪

***

### CR-OPT-004：条件渲染和权限控制集中收敛

- **分类**: 页面与交互优化
- **优先级**: 建议整改
- **检查方式**: 人工审查
- **判断信号**:
  - 模板中存在 `v-if="a && b && c"` 超过 2 个条件的组合
  - 按钮/区块的显隐规则散落在多个区域重复出现
  - 权限指令和 `v-if` 业务条件混用在同一元素上
  - 权限、状态、角色判断在不同页面重复拼接
- **整改要求**:
  - 复杂显隐逻辑收敛为具名 `computed`
  - 权限指令和 `v-if` 尽量分层使用，避免同一元素既判权限又判业务
  - 权限判断统一出口（使用项目封装的权限指令），保持一致使用
  - 状态驱动 UI 的规则集中管理

***

### CR-OPT-005：重复模板/样式/枚举映射需抽取复用

- **分类**: 页面与交互优化
- **优先级**: 建议整改
- **检查方式**: 人工审查
- **判断信号**:
  - 同模块或跨模块中重复 3 次以上的按钮区、状态展示、空态处理
  - 相同 tag/status 文案映射在不同页面重复定义
  - 相同表格列格式化逻辑反复出现
- **整改要求**:
  - 跨模块重复 2 次以上即评估抽取；同模块内重复 3 次以上评估抽取
  - 抽成常量、工具函数、公共组件或 composable
  - 抽取后命名保持业务语义，避免过度泛化

***

### CR-OPT-101：模板复杂表达式和大段 slot 提升可读性

- **分类**: 可维护性优化
- **优先级**: 建议整改
- **检查方式**: 人工审查
- **判断信号**:
  - 模板内联三元表达式嵌套超过 2 层
  - 复杂判断直接写在模板中（`v-if` 内含 3 个以上条件）
  - 大段 slot 内容导致模板层级过深
  - 链式调用过长（如 `item?.detail?.list?.filter(...)?.map(...)`）
  - 同一逻辑在多行模板中重复拼接
- **整改要求**:
  - 复杂表达式移出模板，收敛为 `computed` / 方法
  - 内联三元过多时拆分步骤
  - 大段 slot 内容评估拆分子组件
  - 链式调用拆为中间变量，命名有语义

***

### CR-OPT-102：样式作用域、层级和无用样式清理

- **分类**: 可维护性优化
- **优先级**: 建议整改
- **检查方式**: 人工审查
- **判断信号**:
  - `:deep()` 嵌套超过 2 层
  - 选择器对第三方 UI 组件内部 DOM 依赖过强（如 `.ant-table-body > .ant-table-row`）
  - 原子类 + Less 混用不一致（同类问题一处用原子类一处写 Less）
  - 存在明显无用样式未清理（class 在模板中无引用）
  - `!important` 使用过多，说明层级控制出现问题
- **整改要求**:
  - 控制 `:deep()` 嵌套层级，优先用组件 API override
  - 减少对第三方组件内部 DOM 的强依赖
  - 删除无用样式，保持作用域清晰（`scoped`）
  - 原子类和 Less 使用保持一致风格

***

### CR-OPT-103：国际化 key 结构统一，避免重复词条

- **分类**: 可维护性优化
- **优先级**: 建议整改
- **检查方式**: 人工审查
- **判断信号**:
  - 页面直接写中文字符串不走 `$t()` / `t()`
  - `$t()` 和 `t()` 混用（项目中两者并存）
  - 词条 key 命名混乱（驼峰、下划线、点号混用）
  - 同义词条重复存在（如 `'page.confirm'` 和 `'btn.confirm'`）
  - 页面文案 key 未按业务域归类（全部堆在 `page.*` 下）
- **整改要求**:
  - 用户可见文案必须走 i18n，禁止硬编码
  - `$t()` 和 `t()` 在同一文件内保持统一
  - 词条 key 按业务域归类（如 `vaAccount.xxx`、`payment.xxx`）
  - 合并重复词条，保持词条粒度合理

***

### CR-OPT-104：方法职责拆分和可复用性提升

- **分类**: 可维护性优化
- **优先级**: 建议整改
- **检查方式**: 人工审查
- **判断信号**:
  - 单个方法超过 50 行，职责不清晰
  - 方法副作用过重（同时改状态、发请求、弹提示）
  - 关键数据转换逻辑未抽成纯函数
  - 业务判断写死在页面内部，难复用
- **整改要求**:
  - 超过 50 行的方法评估拆分为多个职责单一的小方法
  - 关键数据转换逻辑抽为纯函数或独立方法
  - 复杂业务判断降低副作用耦合
  - 可复用判断逻辑从页面中抽离

***

## 等级说明

| 等级 | 标识            | 含义           | CR 处理方式      |
| -- | ------------- | ------------ | ------------ |
| S  | 🔴 Blocking   | 必须修复才能 Merge | 不修复则驳回       |
| M  | 🟡 Warning    | 建议修复         | 记录到报告，计入历史提醒 |
| R  | 🔵 Suggestion | 人工判断，可能需要讨论  | 记录到报告，不强制修复  |

***

## 等级分布统计

| 等级                   | 数量     | 说明          |
| -------------------- | ------ | ----------- |
| S（Severe / Blocking） | 11     | 阻塞级：违反即不可合并 |
| M（Medium / Warning）  | 48     | 警告级：建议修改    |
| R（Recommendation）    | 3      | 建议级：可延后处理   |
| **合计**               | **62** | <br />      |

***

## 整改优化项统计

| 优先级    | 数量    | 说明                     |
| ------ | ----- | ---------------------- |
| 建议整改   | 9     | 不阻塞合并，建议在后续迭代中处理     |
| **合计** | **9** | <br />                 |

