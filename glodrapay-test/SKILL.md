---
name: glodrapay-test
description: GlodraPay 商户后台（mso）项目级测试全流程 skill。当用户要求"写单测 / 跑测试 / 出测试报告 / /test"时使用。混合模式：挂载测试（零侵入 src，@vue/test-utils 直接 mount .vue 测行为）为主，纯逻辑单测为辅。执行顺序：定位 openspec 验收依据 → 代码走查（R1/R2/R3）→ 编写测试 → pnpm test → 生成 tests/report/<YYYYMMDD>/测试报告.md。
agent_created: true
---

# GlodraPay 测试全流程

## 测试策略（混合模式，2026-09-23 起生效）

`vitest.config.ts` 已启用 `@vitejs/plugin-vue` + AutoImport + Components 插件链（原 stub-vue 空模块方案已废弃），因此：

1. **挂载测试（首选）**：`.vue` 组件可直接 import + mount，**零侵入 src**——不抽函数、不改业务代码。测行为而非函数：断言按钮显隐 / 文案渲染 / 点击后交互 / mock 请求参数。
2. **纯逻辑单测（辅助）**：`constant.ts`、已导出纯函数、API 层等非组件逻辑照旧 `import` 直接测，风格沿用 `tests/mount/merchant-profit-logic.test.ts`。
3. 两者共存于同一配置，均跑 `pnpm test`（= `vitest run`，jsdom）。

## 执行流程

### 1. 定位被测对象与验收依据

- 若 openspec 工作流有对应变更（`openspec/changes/<change>/tasks.md`），读取并提取验收标准编号（如 1.6 / 3.5 / 6.4），作为测试依据写入报告。
- 若无 openspec 变更，向用户确认被测对象（模块 / 文件 / 变更名），报告"依据"字段记为用户描述或"无"。

### 2. 代码走查（先走查，后测试）

按五类问题扫被测代码，并对照 AGENTS.md 规范：

| 级别 | 含义 |
| --- | --- |
| R1 | 功能错误 |
| R2 | 响应性丢失 / 边界无兜底 / 竞态隐患 |
| R3 | 规范违反（注释格式、箭头函数、类型 any、魔法值、枚举位置等） |

重点检查项（历史踩坑）：
- 枚举/常量必须在 `constant.ts` 维护，不得出现在 `.d.ts` 或 `.vue` 内
- API 字段类型只出现在 `api/**/model.d.ts`
- 下拉等共享组件不得扩展 prop 类型（如 FsSelect 禁止加 Boolean 类型，规避 cast 告警）
- 判定逻辑必须有 null / undefined / 契约漂移值兜底
- 角色判定用 `role_code`，不得用角色 id 字面量

### 3. 编写测试

**挂载测试模板**（文件放 `tests/mount/<业务名>.test.ts`）：

```ts
import { describe, expect, it, vi } from 'vitest'
import { mount } from '@vue/test-utils'
import MyComponent from '@/views/<模块>/<组件>.vue'

describe('myComponent：行为测试', () => {
  it('正向：xxx 条件下按钮显示', () => {
    const wrapper = mount(MyComponent, {
      props: { /* ... */ },
      global: {
        // 组件依赖 router / pinia / i18n 时按需提供：
        // plugins: [router],
        // mocks: { $t: (k: string) => k },
        // stubs: { SvgIcon: true } // 未注册的自定义组件打桩
      }
    })
    expect(wrapper.find('.btn').exists()).toBe(true)
  })

  it('交互：点击后请求参数正确', async () => {
    const wrapper = mount(MyComponent, { /* 同上 */ })
    await wrapper.find('.btn').trigger('click')
    // 断言 mock 被调用时的参数
  })
})
```

挂载复杂组件的依赖处理原则（都是测试侧动作，不碰 src）：
- **i18n**：复用真实语言包 `createI18n({ legacy: false, locale: 'zh', messages: { zh: zhMessages, en: enMessages } })`（`@/locales/zh`、`@/locales/en`），比 mock `$t` 更有价值（间接验证 key 存在性）
- **pinia**：未装 `@pinia/testing` 时用 `setActivePinia(createPinia())` + `vi.mock` 对应 store 模块
- **router**：`createRouter` + `createMemoryHistory` 真实实例，或 stub `<RouterLink>`
- **axios/fetch 请求**：`vi.mock('@/api', async (importOriginal) => ({ ...(await importOriginal()), ... }))`——必须部分 mock 保留全量导出，`@/hooks` 等链路会 import 其他 API 方法
- **v-number-only 等自定义指令**：`global.directives: { 'number-only': {} }` 注册空指令
- **SvgIcon 等未注册组件**：`global.stubs` 打桩

挂载测试实战要点（add-audit-log 首跑踩坑沉淀，违反会假阳性/假失败）：
1. **teleport 组件（a-drawer/a-modal）**：内容渲染在 `document.body`，用 `document.body.textContent / querySelector` 断言；每个 describe 的 beforeEach 清空 `document.body.innerHTML = ''`，防跨用例残留串染
2. **watch 无 immediate 的开合组件**（如抽屉 watch visible）：不能以 `visible: true` 初始挂载（watch 不触发），须 `visible: false` 挂载后 `setProps({ visible: true })` 模拟真实翻转路径
3. **antd Button 两字中文自动插空格**（'导出'→'导 出'）：按钮查找必须归一化 `btn.text().replace(/\s+/g, '') === label`；点击类断言必须同时断言"调用次数变化"（如 1→2），否则没点中也可能侥幸通过（假阳性）
4. **`vi.clearAllMocks()` 不清除 mock implementation**：上一用例的 `mockResolvedValue` 缺陷数据会泄漏到下一用例，关键 describe 显式重设实现
5. **jsdom 缺 window.matchMedia**：antd 响应式栅格（a-row）挂载即崩，已由 `tests/setup.ts` polyfill 解决（vitest.config.ts setupFiles），新环境跑挂载测试先确认该文件在位
6. **捕获渲染崩溃证据**：组件渲染抛错时用 `global.config.errorHandler` 收集，比 it.fails 可靠（异步渲染崩溃 it.fails 捕不到）；用于固化"已知缺陷监测用例"，修复后该用例转红提示改写为正向断言

**纯逻辑单测**：文件放 `tests/mount/<业务名>-logic.test.ts`，describe 用中文，按 **正向 / 反向 / 边界兜底** 三组组织；契约漂移兜底（错误类型传入不报错、有合理默认值）必测。文件头加 `@description` 注释：被测对象、对应验收标准编号、覆盖范围。遵守 AGENTS.md：箭头函数、显式类型、无魔法值。

### 4. 执行测试

```bash
pnpm test
```

收集通过 / 失败数；失败先定位根因修复或确认是否为真实缺陷，不得隐藏失败。挂载真实视图组件若牵连全局副作用（router 守卫、store 初始化）报错，按第 3 步的依赖处理原则补 provider/stub，不改业务代码。

### 5. 生成测试报告

- 路径：`tests/report/<YYYYMMDD>/测试报告.md`（日期目录 8 位数字，如 20260922；**不要**用旧的 `tests/<日期>/` 路径）
- 结构与措辞严格遵循 `references/report-template.md`

### 6. 收尾

- 挂载测试仍覆盖不到的项（接口真实字段联调、真实双语言切换渲染、切换商户串号等需真实环境的场景）列入"遗留事项"，逐条给出场景编号
- 走查发现但未修复的问题进"R 级问题清单"；已随用户纠正解决的标明"已随纠正解决"

## 硬约束（违反即返工）

1. **零侵入 src**：禁止为可测性抽函数、改业务代码；测试能力不足时在测试侧补 mock/stub/provider
2. 报告路径统一 `tests/report/<YYYYMMDD>/`；同日多次运行覆盖同日报告并在文首标注轮次
3. 报告断言明细表每行必须有"证据"列（expect 表达式或走查证据描述）
4. 无法验证的断言记"无法验证"，不得标"通过"
5. 全部内容中文
