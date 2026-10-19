---
name: glodrapay-front-test
description: 前端项目（Vue / React，Vitest / Jest / Playwright / Cypress 等任意测试栈）的测试全流程 skill：探测目标项目的测试栈与规范 → 代码走查（R1/R2/R3）→ 编写挂载/行为测试与纯逻辑单测（零侵入 src）→ 真实执行测试 → 生成 tests/report/<YYYYMMDD>/测试报告.md。当用户要求"写单测 / 跑测试 / 补测试 / 出测试报告 / 测这个变更 / /test / 验收测试"时使用；被 openspec/sflow 的 /sflow-verify 挂载做深度验证时也使用本 skill。只要任务是在前端仓库里验证行为、补测试或沉淀测试报告，就该用它，即使用户没有明确点名。
agent_created: true
---

# 前端项目测试全流程

本 skill 面向**任意前端仓库**（Vue / React + Vitest / Jest / Playwright / Cypress 等），不绑定某一个项目。
所有项目差异——测试命令、目录布局、组件库、代码规范——**一律从目标项目现场探测**，不假设、不套用别的项目的约定。
方法内核是固定的一步：**用真实执行的证据回答"这个改动是否达标"，而不是用"看着没问题"来回答。**

## 0. 先明确被测对象与验收依据

1. 若目录下有 openspec 工作流，先找对应变更：`openspec/changes/<change>/tasks.md`，提取**验收标准编号**（如 1.6 / 3.5 / 6.4）与 `specs/` 场景，作为测试依据写入报告。
2. 若无 openspec，向用户确认被测对象（模块 / 目录 / 文件 / 变更名）；报告"依据"一栏记用户原话，或记"无"。
3. 依据清单是**后面一切动作的靶子**：走查对照它，用例覆盖它，报告逐条回应它。没有依据就先去要，不要自己编一条"看起来合理"的标准。

## 1. 探测项目测试环境与规范（每个项目都要做，不可跳过）

在项目根目录跑一次探测，用结果决定后面所有细节：

```bash
node -e "const p=require('./package.json');console.log(JSON.stringify({scripts:p.scripts,testDeps:Object.keys({...p.dependencies,...p.devDependencies}).filter(d=>/vitest|jest|playwright|cypress|test-utils|testing-library|jsdom|happy-dom|msw/i.test(d))},null,2))"
ls vitest.config.* vite.config.* jest.config.* playwright.config.* cypress.config.* 2>/dev/null
ls -d tests test __tests__ src/__tests__ 2>/dev/null
```

按探测结果填出这张表（**后续每一步都引用它的结论**）：

| 探测项 | 怎么定 | 用途 |
| --- | --- | --- |
| 测试框架 | 配置文件名 + 依赖（`vitest` / `jest` / `playwright` / `cypress`） | 决定命令与 API（`vi` vs `jest`） |
| 执行命令 | `package.json` 的 `scripts.test`（无则按框架兜底，如 `npx vitest run`） | 第 4 步真实执行的命令 |
| 运行环境 | 配置里的 `environment`（`jsdom` / `happy-dom` / `node`）或 `testEnvironment` | 能否挂载组件（DOM 环境才行） |
| 组件挂载能力 | 是否装了 `@vue/test-utils` / `@testing-library/vue` / `@testing-library/react`，且配置启用了对应编译插件 | 决定走"挂载测试"还是"只能纯逻辑测试" |
| 测试目录与命名 | 配置里的 `include` / `testMatch`，以及 `tests/`、`__tests__/` 现状 | 新文件放哪、叫什么 |
| setup 文件 | 配置里的 `setupFiles` | 全局 polyfill 落点 |
| 代码规范 | **读项目根 `AGENTS.md`**（及 `docs/` 下它指向的规范文档、`docs/correction-rules.md`） | 走查判 R3 的唯一依据 |
| 报告路径 | 默认 `tests/report/<YYYYMMDD>/测试报告.md`；项目若在 AGENTS.md / workflow.config.md 另行声明则从其声明 | 第 5 步产物落点 |

关键原则：

- **规范只认目标项目自己的 `AGENTS.md`**。这个 skill 不内置任何项目的编码规范——注释格式、函数风格、类型约定、常量/枚举放哪、组件使用禁忌，全是项目私有的，必须现场读。读不到就按项目既有代码的既有模式（同目录同类文件怎么写的）推断，并在报告里写明"规范来源：项目既有模式"。
- **能力决定策略**，不是愿望决定策略。DOM 环境 + 挂载库可用 → 优先挂载测试；只有 node 环境 → 纯逻辑测试 + 走查。不要为了"测组件"而去改项目的测试配置（那属于改代码，见硬约束）。
- **项目没有测试框架时**（探测不到框架、也没有 `scripts.test`）：不要静默跳过，也不要擅自安装依赖改配置。向用户呈报三个选项，由用户定：① 用户同意后以最小改动引入框架（属于项目配置变更，需明说会新增哪些文件/依赖）；② 降级为"走查 + 报告"，报告如实写明"无测试框架，自动断言 0 条，结论基于走查证据"；③ 用户另指定命令。
  - 走①时的落地清单：**先在同 workspace 找已配好同栈框架的兄弟项目**，抄它的 `vitest.config.ts` / `jest.config.*` 与 `setup.ts` 作蓝本（比从零写可靠）；配置只保留测试必需插件（样式类插件如 WindiCSS / svg-icons / compression 无需带入）；新增 `package.json` 的 `scripts.test`；依赖用项目声明的包管理器装（看 `packageManager` / lock 文件）。
  - **先查 `.gitignore` 是否本就排除测试基建**：很多项目已把 `tests/*`、`vitest.config.ts` 列入忽略（视为本地草稿）。若如此，测试文件不会进版本库、也不参与 `pnpm lint`（`--ignore-path .gitignore`）。这属项目既有约定，**在报告"遗留事项"里说明即可，不要擅自改动忽略规则**。

## 2. 代码走查（先走查，后写测试）

测试之前先通读被测代码，对照第 1 步探测到的项目规范扫描问题。**走查先行的理由**：走查能发现测不到的缺陷（缺失兜底、竞态、契约漂移），也能反过来告诉你测试该盯哪几个分支。

按三级登记：

| 级别 | 含义 | 判定要点 |
| --- | --- | --- |
| R1 | 功能错误 | 实现与验收标准/接口契约不一致，会导致用户拿到错误结果 |
| R2 | 健壮性缺陷 | 响应性丢失、边界无兜底（null / undefined / 空数组 / 契约漂移值）、竞态、异常吞掉 |
| R3 | 规范违反 | 违反目标项目 `AGENTS.md`（或其指向的 docs）里**明文写的**规范 |

R2 的通用检查项（与项目无关，长期有效）：

- 判定/映射逻辑对非法输入有无兜底（未知状态值 → 有合理默认或显式空，而不是崩或误判）
- 异步链路有无竞态（快速切换筛选、重复点击、请求后到达覆盖先结果）
- 交互态开关（loading / disabled）是否覆盖失败分支
- 列表与详情的数据口径是否同源（同一状态枚举两处比较极易漂移）
- 权限/角色判定是否用了稳定标识（项目若规定了具体字段，以项目规范为准）

R3 **只引项目规范原文**：不把别的项目的规矩当通用真理。mso / ops 这类项目的规范样例见 `references/mount-testing.md` 末尾的"项目规范样例"，**仅供理解"R3 长什么样"，不作为判定依据**。

## 3. 编写测试

**先设计用例，再写代码**：通读 `references/test-design.md`，按"四个来源 + 四步法"先列出用例清单表（输入 / 期望 / 依据 / 类型），用其中的"停止准则"自检漏没漏，然后再逐条落地成 `.test.ts`。不要边想边写——清单表能暴露"只测了 happy path""期望值抄实现"这类问题。

文件位置与命名遵循第 1 步探测到的 `include` / 现有目录习惯。默认约定（项目无特别规定时）：

- 挂载/行为测试：`tests/mount/<业务名>.test.ts(x)`
- 纯逻辑单测：`tests/unit/<业务名>-logic.test.ts`
- 纯逻辑，`<业务名>` 用被测模块名；**一个业务一个文件**，不要把所有断言塞进一个巨型文件

### 挂载/行为测试（首选，能力允许时）

零侵入：直接 import 目标组件 + 挂载，断言**行为**而非函数——元素显隐、文案渲染、点击后的交互、发出请求的参数。

```ts
import { describe, expect, it, vi } from 'vitest'          // Jest 项目把 vi 换成 jest
import { mount } from '@vue/test-utils'                     // React 用 @testing-library/react 的 render
import MyComponent from '@/views/<模块>/<组件>.vue'

describe('myComponent：行为测试', () => {
  it('正向：xxx 条件下按钮显示', () => {
    const wrapper = mount(MyComponent, {
      props: { /* ... */ },
      global: {
        // 组件依赖 router / pinia / i18n 时按需提供；未注册组件打桩
        // plugins: [router], mocks: { $t: (k: string) => k }, stubs: { SvgIcon: true }
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

依赖处理与踩坑（teleport、watch 无 immediate、antd 中文按钮插空格、`vi.clearAllMocks()` 不清实现、jsdom 缺 `matchMedia`、渲染崩溃取证等）见 **`references/mount-testing.md`**——先读它再写挂载测试，能避开绝大多数假阳性/假失败。

### 纯逻辑单测（辅助）

`constant.ts`、已导出的纯函数、API 层等非组件逻辑直接 import 测。按 **正向 / 反向 / 边界兜底** 三组组织 `describe`，中文描述；**契约漂移兜底必测**（错误类型传入不报错、有合理默认值）。文件头加 `@description` 注释：被测对象、对应验收标准编号、覆盖范围。代码风格遵守目标项目规范。

判定分支是否"测到了"，用 `references/test-design.md` 的分支覆盖清单逐条对照（每个 `if` 一真一假、每个 `switch` case 全覆盖、每个 `??`/`||` 两侧都走）。写用例时同步记录**每条断言对应哪条验收标准**，第 5 步的报告要逐条回应，并补出"验收标准 → 用例"覆盖矩阵（防漏测，见 `report-template.md`）。

## 4. 执行测试

跑第 1 步探测到的真实命令（默认 `pnpm test` 或项目 `scripts.test`；包管理器以项目 `packageManager` / lock 文件为准）。**必须是真实执行并贴出输出**：

- 记录通过 / 失败 / 跳过数。
- 失败先定位根因：是测试写错（补 provider / stub / mock），还是真实缺陷。**不得隐藏失败、不得改断言迁就实现**。
- 挂载真实视图若牵连全局副作用（路由守卫、store 初始化）报错，按 `references/mount-testing.md` 在**测试侧**补 provider/stub，不改业务代码。
- 项目有更完整的验证链（lint / typecheck / test / build，见 `workflow.config.md`）时，本步至少跑完测试命令，必要时补跑相关检查。

### 执行成本控制（实测数据，别拿时间换形式）

**进程启动成本主要来自 import 图，不是用例条数。** fs-glodrapay-web-admin 2026-10-14 实测：全量 `run` **32.6s**、仅单元文件 **3.0s**、仅挂载文件 **28.6s**——挂载文件 import 到组件 + antd + store 图，单进程就要 ~26s，与里面几条用例无关。

因此：

- **同一次任务里，全量测试只跑必要的几次**：每批用例写完跑一次，全部改完再跑一次。不要每改一条断言就跑一次全量。
- **把反复验证的工作放到最便宜的文件上**：失败定位、变异抽查等需要"改一次跑一次"的环节，优先用**纯逻辑单测文件**（2~4s），别用挂载文件（25s+）。挂载文件的断言写完一次性验收即可。
- 需要反复单跑时用 `-t <用例名>` 定向到 `describe`/`it`，不要整个文件重跑。
- `npx vitest` 与直接调 `node node_modules/vitest/vitest.mjs` 只差约 0.3s，可忽略；reporter 用 `dot` 或 `verbose` 对耗时无影响，只为可读性选。
- **控制输出体积**：typecheck/lint 若被 `node_modules` 既有错误淹没（实测 1366 行里 1362 行来自 `@types/*`），先 `Select-String` 过滤 `^(src|tests)[\\/]` 再看，**不要把全量报错读进上下文**。需要贴进报告时只贴汇总行。

## 5. 生成测试报告

- 路径：默认 `tests/report/<YYYYMMDD>/测试报告.md`（日期目录 8 位数字，如 20260922）；项目另有声明则从其声明。
- 结构与措辞严格遵循 **`references/report-template.md`**。
- 报告"测试环境"栏如实写第 1 步探测结果（框架 / 环境 / 是否具备挂载能力），不要照抄模板里的示例。
- 报告须含**验收标准 → 用例覆盖矩阵**：每条验收标准对应被哪几条用例覆盖；无对应用例的验收标准显式标"未覆盖"，不许略过。
- 同日多次运行覆盖同日报告，并在文首标注轮次。

## 6. 变异抽查（关键分支必做）

**只要本次为某个判定分支写了断言，就对该分支做一次变异抽查**，证明断言不是假的（"把实现改坏，这条会不会红"）：

1. **按启动成本分流**（这是既快又不降级的关键）：逻辑层能覆盖的分支，变异时**只跑逻辑测试文件**（实测 3s vs 挂载 28s，差近 10 倍）；只有组件专属分支（渲染条件、事件绑定、prop 层拦截）才跑挂载文件。真正贵的不是变异，是被迫跑重型进程。
2. **控制规模**：一次抽查 **3~5 个**最高价值分支。超过时挑"最可能造假"的——期望值与实现同源的、只测了 happy path 的、新补的分支守卫。
3. 写个临时脚本循环：`备份 src → 变异 → 跑对应用例（-t 定向）→ 还原`。**判定只看日志里的 `Tests N failed|passed` 汇总行**：没有该行说明是收集/编译阶段出错，记 **UNKNOWN** 而不是 RED。
   - Windows 上别用 `execFileSync('npx.cmd', …)`——会静默失败（status 为 null、stdout 全空），必须用 `spawnSync(process.execPath, ['node_modules/vitest/vitest.mjs', 'run', …])` 直接调入口。
   - 变异前 `copyFileSync(原文件, .bak)`，逐个还原，最后 `git status --short <path>` 校验还原干净；**全部变异结束后只跑一次全量确认**，不要每变异一次就跑全量。
4. **失败 = 用例有效**；仍全绿 = 用例没真正覆盖该分支，须补强或判定为"等价变异"（见下）。
5. 判定为**等价变异**（变异点物理不可达或行为无差异，如 `disabled` 拦截导致内层 return 不可达，见 `mount-testing.md` 踩坑 3）时，在报告里如实写明原因与证据，**不得为了让它"能红"去改 src 或强点 disabled 元素**。注意：只有当"另一道防线是否生效"有证据时才判等价——例如切换态下按钮未禁用时，组件内守卫就是真防线，此时必须证红。
6. 结果写入报告第 5 节。纯走查、无自动断言的降级模式，本步可写"未执行"并说明关键分支已由走查证据覆盖。

## 7. 收尾

- 挂载测试覆盖不到的项（真实接口字段联调、真实语言切换、真实登录态/权限下发、跨端串号等需真实环境的场景）列入"遗留事项"，逐条给场景编号与"为什么不能在此环境验证"。
- 走查发现但未修复的问题进"R 级问题清单"；已随用户纠正解决的标明"已随纠正解决"。
- **验证链每条命令只跑一次**：`eslint` 加 `--cache`；typecheck / build 各跑一次即可。`build` 会重新生成 `components.d.ts` 等生成文件，跑完 `git checkout -- <生成文件>` 还原，别污染工作区。
- 若本次探测/踩坑得到**跨项目通用**的新结论（新框架适配、新的假阳性陷阱、新的耗时成因），优先补进 `references/mount-testing.md`，让下一个项目直接继承——这正是本 skill 保留价值的方式。

## 硬约束（违反即返工）

1. **零侵入 src**：禁止为可测性抽函数、改业务代码、改项目测试配置。测试能力不足时在测试侧补 mock/stub/provider，或按"无测试框架"三条路径走。
2. **规范来源必须是目标项目自己的 `AGENTS.md` / docs**，不得把别的项目的规范当依据。
3. 报告路径统一 `tests/report/<YYYYMMDD>/`（项目另有声明除外）；同日多次运行覆盖同日报告并标注轮次。
4. 报告**断言明细表每行必须有"证据"列**（`expect` 表达式或走查证据描述）。
5. **无法验证的断言记"无法验证"，不得标"通过"**；未真实执行的命令不得声称已执行。
6. **先出用例清单再写测试代码**；报告须含"验收标准 → 用例覆盖矩阵"，无对应用例的验收标准显式标"未覆盖"。
7. **为判定分支写了断言，就要做变异抽查**（纯走查降级模式除外），等价变异如实说明原因，不得靠改 src 让它"能红"。
8. **不重复执行同一验证命令**：同一次任务里全量测试 / typecheck / lint / build 各跑一次；反复验证（失败定位、变异抽查、逐条用例验收）一律用定向 `-t` 或单文件跑。挂载测试单进程启动 25s+，是主要时间黑洞。
9. 全部内容中文（以目标项目 AGENTS.md 的语言要求为准，默认中文）。

## 参考文件

- `references/test-design.md` —— 用例设计方法：四个来源、等价类/边界值/分支覆盖清单、停止准则、反模式。**写测试前必读。**
- `references/mount-testing.md` —— 组件挂载测试的依赖处理、框架差异速查、踩坑清单、项目规范样例。**写挂载测试前必读。**
- `references/report-template.md` —— 测试报告字段、五段结构与覆盖矩阵。**生成报告前必读。**
