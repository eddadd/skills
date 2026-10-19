# 组件挂载测试参考（框架无关方法论 + 各栈差异）

本文件是 `SKILL.md` 第 3 步的展开。**写挂载测试前通读一遍**，能避开绝大多数假阳性 / 假失败。
内容分四块：① 通用原则 ② 依赖注入速查（Vue / React）③ 踩坑清单（跨项目有效）④ 项目规范样例（仅供理解，不作依据）。

---

## 一、通用原则

1. **测行为，不测实现**：断言用户可感知的结果——元素显隐、渲染文案、点击后的交互、发出的请求参数——而不是断言内部方法被调用、私有函数返回值。
2. **零侵入**：任何为"好测"而改 src 的冲动都先怀疑。缺依赖就补 provider / stub / mock，缺数据就造数据。
3. **每条断言要有证据**：报告里能写出 `expect(...)` 表达式；写不出表达式的走查类结论才写证据描述。
4. **红要真红**：一个用例若"实现改坏了它也不失败"，它就是假的。关键分支写完后，心里过一遍"把实现反向改一下，这条断言会不会红"。

---

## 二、依赖注入速查

### Vue 3（Vitest/Jest + @vue/test-utils）

挂载时的 `global` 选项：

| 依赖 | 处理方式 |
| --- | --- |
| i18n | 优先复用**真实语言包**：`createI18n({ legacy: false, locale: 'zh', messages: { zh, en } })`——比 mock `$t` 更有价值，能间接验证 key 存在性 |
| pinia | 无 `@pinia/testing` 时用 `setActivePinia(createPinia())` + `vi.mock` 对应 store 模块 |
| router | `createRouter` + `createMemoryHistory` 真实实例，或 stub `<RouterLink>` |
| axios / fetch / 请求层 | `vi.mock('@/api', async (importOriginal) => ({ ...(await importOriginal()), ... }))`——**必须部分 mock 保留全量导出**，`@/hooks` 等链路会 import 其他 API 方法 |
| 自定义指令 | `global.directives: { 'number-only': {} }` 注册空指令 |
| 未注册的自定义组件 | `global.stubs` 打桩 |

### React（Vitest/Jest + @testing-library/react）

| 依赖 | 处理方式 |
| --- | --- |
| Provider（Redux / Zustand / Context / Query） | 写一个 `renderWithProviders(ui, { route, preloadedState })` 测试工具，包住 `render` |
| Router | `MemoryRouter` + `initialEntries` |
| 请求（axios / fetch） | `vi.mock('axios')`，或 `msw`（若项目已装 msw 优先用它，行为更真） |
| 国际化 | 复用真实语言包 provider，或 mock `useTranslation` |
| 副作用（定时器 / 动画） | `vi.useFakeTimers()`，用后 `vi.useRealTimers()` |

### 断言与查询选择器

- 优先按**可访问性语义**查询（`getByRole('button', { name: '导出' })`），比 class / test-id 更贴近用户视角，也更抗重构；项目已有 `data-testid` 约定时从约定。
- 文案断言要**归一化空白**（见踩坑 3），否则中文按钮插空格这类问题会误伤。

---

## 三、踩坑清单（跨项目有效）

1. **teleport / portal 组件**（Vue `a-drawer`、`a-modal`；React portal 弹层）：内容渲染到 `document.body`，`wrapper.find` 找不到，必须用 `document.querySelector` / `document.body.textContent` 断言；**每个用例前后清空 `document.body.innerHTML = ''`**，防跨用例残留串染。
   - **不要用 `stubs: { teleport: true }` 绕**（容易误以为更省事）：实测把 `<teleport>` 打桩后，`wrapper.html()` 只剩 `<teleport-stub to="[object HTMLDivElement]">[object Object]</teleport-stub>`，内容既不进 wrapper 也不进 body，反而完全取不到。正解是「照常挂载 + 断言走 `document`」。
   - 组件若带开合动画/过渡，`a-modal` 需先 `await flushPromises()` 再查询，否则内容尚未挂载。
2. **watch 无 immediate 的开合组件**：不能以 `visible: true` 初始挂载（watch 不触发）。须 `visible: false` 挂载后 `setProps({ visible: true })` 走真实翻转路径。
3. **antd Button 两字中文自动插空格**（`'导出'` → `'导 出'`）：按钮查找必须归一化 `btn.text().replace(/\s+/g, '') === label`；点击类断言**必须同时断言"调用次数变化"**（如 1→2），否则没点中也可能侥幸通过（假阳性）。
   - **`disabled` 组件在 jsdom 里点不动，会掩盖内层守卫**：antd `a-button`/`FsBtn` 的 `disabled` 是 **prop 层**拦截（`<button disabled>` 上加 `pointer-events` 语义），在 jsdom 中 `.click()` 不会触发 `@click` handler。因此组件里形如 `:disabled="x"` + `if (x) return` 的**双重防线**，其内层 `return` 在按钮路径下**不可达**。变异删除该 `return` 后测试仍全绿，是**等价变异**、不是用例弱点——报告里应按「冗余防线」如实记录，不要为了让它"能红"去 `removeAttribute('disabled')` 强点（实测仍不进 handler）。真正要测的守卫是另一条独立条件（如 `if (switching.value) return`，它不靠 disabled 拦截）。
4. **`vi.clearAllMocks()` 不清除 mock implementation**：上一用例的 `mockResolvedValue` 缺陷数据会泄漏到下一用例；关键 describe 显式重设实现（`mockReset` + 重设）。
5. **jsdom 缺 `window.matchMedia`**：antd 响应式栅格（`a-row`）挂载即崩。在 `setupFiles` 指定的 setup 文件里 polyfill（见文末模板）；新环境跑挂载测试先确认该文件在位。
6. **捕获渲染崩溃证据**：组件渲染抛错时用 `global.config.errorHandler`（Vue）或 `ErrorBoundary`（React）收集，比 `it.fails` 可靠（异步渲染崩溃 `it.fails` 捕不到）。用于固化"已知缺陷监测用例"，修复后该用例转红，提示改写为正向断言。
7. **AI 自动 import 的组件在测试里可能未注册**：项目若靠 `unplugin-vue-components` 自动注册 antd / 本地组件，测试环境未必同样生效；用 `stubs` 兜底，别为了让它真渲染去改配置。
8. **共享可变模块状态**：`constant.ts` 之外的模块级可变量、单例 store，在多个 test 文件间会串；必要时 `vi.resetModules()` + 动态 `import()`。
9. **jest vs vitest 的 API 差异**：`vi` ↔ `jest`、`vi.mock` ↔ `jest.mock`、`vi.useFakeTimers` ↔ `jest.useFakeTimers`；时间 mock、`importOriginal` 用法也有细微差别。第 1 步探测到 jest 就**抄项目里已有测试的写法**，别凭 vitest 习惯写。

### `window.matchMedia` polyfill 模板（放进项目 setup 文件）

```ts
// jsdom 缺失 API polyfill：响应式组件库依赖 window.matchMedia
if (!window.matchMedia) {
  Object.defineProperty(window, 'matchMedia', {
    writable: true,
    value: (query: string) => ({
      matches: false,
      media: query,
      onchange: null,
      addListener: () => {}, // 废弃 API，老组件库仍在用
      removeListener: () => {},
      addEventListener: () => {},
      removeEventListener: () => {},
      dispatchEvent: () => false
    })
  })
}
```

> 注意：polyfill 属于**项目测试基建**。若项目里还没有，且第 1 步确认未配 `setupFiles`，这属于"改项目配置"，按 SKILL.md 第 1 步的"无测试框架三条路径"先问用户，不要擅自新增。

---

## 四、项目规范样例（**仅供理解 R3 长什么样，不作判定依据**）

下面这些条目来自 GlodraPay 系（mso / ops / fs-admin）项目的 `AGENTS.md`，仅作为"项目规范可以细到什么程度"的示例。**判定 R3 时永远以当前目标项目的 `AGENTS.md` 为准**，不得套用此处。

样例条目（示例，非通用规则）：

- 枚举 / 常量必须在 `constant.ts` 维护，不得出现在 `.d.ts` 或 `.vue` 内
- API 字段类型只出现在 `api/**/model.d.ts`
- Props 禁止 `Record<string, any>` / `any`，必须具体 `interface` / `type`
- 共享组件不得扩展 prop 类型（如 FsSelect 禁止加 Boolean 类型，规避 cast 告警）
- 一律箭头函数，所有参数与返回值显式标注类型，禁止 `any`（API 层除外）
- `async` 必须配套 `try/catch/finally`
- 禁止硬编码魔法值；生命周期钩子内只调方法不写业务逻辑
- 角色判定用 `role_code`，不得用角色 id 字面量
- antd Form rules 需显式标注 `Record<string, RuleObject | RuleObject[]>`（避免字面量拓宽导致 `trigger` 类型不匹配）
- `auto-imports.d.ts` / `components.d.ts` 是生成文件，不得手改

**从样例能学到的**：R3 检查项应当**逐条可指认**（能指出违反了哪条规范原文），而不是"风格看着不太对"。若目标项目的 `AGENTS.md` 没有明文规定，则该问题不记为 R3，最多作为建议，或按"项目既有模式"提示。
