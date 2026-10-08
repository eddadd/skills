---
name: openspec-spec-methodology
description: OpenSpec spec 增强方法论。当检测到 openspec 提案/变更生成类命令或编辑器展开的命令正文时自动触发，触发信号含三路——①命令变体：/opsx-propose、/opsx-new、/opsx-ff、/opsx-continue 及冒号变体 /opsx:propose、/opsx:new 等，openspec propose/new/ff/continue；②命令展开后的第一句话标题：「提议新变更 - 创建变更并一步生成所有产出物」「使用实验性产出物驱动方法启动新变更」「快速推进产出物创建 - 一次性生成开始实现所需的所有内容」「通过创建下一个产出物来继续处理变更」；③口语表述：「生成提案」「创建变更」「发一个变更」等。命中后自动对需求做一轮增强预检——复杂需求拆解、颗粒度校准（可验证阈值）、边界条件铺排——把增强结论作为上下文注入；产出物仍由 openspec 原样生成，本 skill 不新建、不修改任何产出物。适用于复杂真实需求，目标是「AI 依 spec 实现不跑偏」。本 skill 自动触发，无需用户手动调用。
license: MIT
compatibility: 需要已初始化的 openspec（`openspec-cn`）工作流（/opsx-* 命令存在）与 `spec-driven` schema 的 change。本 skill 纯增强、无外部依赖、无运行时文件。
metadata:
  author: syf
---

# openspec-spec-methodology

OpenSpec spec 增强方法论。检测到用户发起 openspec 提案/变更生成命令（如 `/opsx-propose`、`/opsx-new`、`/opsx-ff`）时自动触发，对需求做一轮「增强预检」，结果作为增强上下文注入对话，产出物仍全部由 openspec 生成。

> 核心原则：**产出物是 openspec 写的，方法论是它写的更稳。**
> 本 skill 不做任何文件输出，只提升进入 openspec 时的需求质量。

***

## 适用场景与触发方式

- **触发**：本 skill **自动触发**，不需要用户手动调用。触发信号分三路，任一路命中即自动生效，对当前需求做一轮增强预检。**完整触发清单见 `references/triggers.md`**（命令升级/改标题只更新该文件，本文件不随命令变动）。
  1. **命令字面变体**（含命名空间分隔符差异）：`/opsx-propose`、`/opsx:propose`、`openspec propose`（propose）；`/opsx-new`、`/opsx:new`、`openspec new`（new）；`/opsx-ff`、`/opsx:ff`、`/opsx-ff-change`、`openspec ff`（ff）；`/opsx-continue`、`/opsx:continue`、`openspec continue`（continue）。
  2. **命令正文第一句话标题**（编辑器把命令内容直接输出时，按开篇标题识别）：「提议新变更 - 创建变更并一步生成所有产出物」（propose）、「使用实验性产出物驱动方法启动新变更」（new）、「快速推进产出物创建 - 一次性生成开始实现所需的所有内容」（ff）、「通过创建下一个产出物来继续处理变更」（continue）。
  3. **口语表述**：「生成提案」「创建变更」「发一个变更」「用 openspec 让 AI 帮我写 spec / 设计 / 任务」「开始一个新变更」等表达生成变更产出物意图的句子。
- **判定信号**：对话出现「openspec / opsx / 提案 / proposal / 变更 / spec 生成」类关键词，且用户意图是**生成变更产出物**（而非归档/验证/同步/探索）。仅归档 / 验证 / 同步 / 探索 / onboard 类命令**不触发**本 skill（详见 `references/triggers.md` 第三节排除清单）。
- **适用对象**：中等以上复杂度需求——多模块动点、含隐性约束、有跨模块契约、状态时序敏感的变更。
- **不适用**：单文件、无交叉约束的小改动，跳过本 skill 直接跑 openspec（本 skill 会判断后不介入）。

> 触发后流程见下；增强预检在 openspec 实际生成产出物前完成。

***

## 执行流程（四步，全部在 openspec 生成前完成）

### 第一步：需求理解确认

- 把原始需求复述为**能力线**（本次要做什么）与**非目标线**（本次明确不做什么），向用户确认理解一致后再继续。
- 若需求本身模糊，先向用户澄清，禁止带着模糊进拆解。

### 第二步：复杂需求拆解（支柱一）

严格按 `references/methodology.md`「一、复杂需求拆解」执行：

1. 拆四层：能力 → 需求 → 场景 → 边界。
2. 强制识别三种隐性需求：逆向约束（禁止什么）、跨模块契约（路由/query/接口/常量/动态路由权限/缓存）、交互时序。
3. 决策点显式化：选 A 不选 B + 一句理由，后续 spec 全篇统一，禁止前后矛盾。
4. 契约与承载预检（对应 `references/methodology.md` 一、1.4~1.9）：字段关联性与取值形态、承载组件是否提供所需能力（如 `fs-table` 不透传 `expandedRowRender` → 用 `a-table`）、每个接口的返回结构形态与字段全集归属、用户粘贴契约的字段归属核对、未点名控件默认文本输入框。
5. 页面要素默认值（对应一、1.10）：新建页面/详情页时逐项确认用户可控要素（返回按钮、卡片结构、导出位置与入参、列表分页、展开行结构、时间列位置），把易返工项前置到同一轮对话。

### 第三步：颗粒度校准（支柱二）

严格按 `references/methodology.md`「二、颗粒度标准」执行：

- 每条需求拆成可独立验收的场景（当/那么/并且）。
- 每个验收点给出可核对量化标准（字段数、请求次数、枚举项数、逐项对照、全仓检索）。
- 全文自查并剔除自我验收措辞（`合理即可` / `正常展示` / `妥善处理` / `美观` / `友好提示` / `不报错` / `视情况` / `尽量`）。

### 第四步：边界条件铺排（支柱三）

严格按 `references/methodology.md`「三、边界条件表达法」执行：

- 过一遍强制边界场景清单（空值 / 非法值 / 失败态 / 空选项 / 时序 / 权限 404 / 串号缓存），凡适用即写**独立 `#### 场景`**。
- 边界遵循三原则：独立场景化、可触发前置、结果可观察。
- 每个模块验收标准挂 `边界：` 条目，且可量化。

### 检查关卡

四步完成后，逐条过验 `references/checklist.md`（A 拆解完整性 / B 颗粒度可验证 / C 边界覆盖 / D 与 openspec 配合）。任何「否」项修正后再进入下一步；全部通过后，把拆解结论作为**增强上下文**注入对话，提示模型「以下为增强约束上下文，产出物由 openspec 原样生成」。

***

## 硬约束（禁止事项）

- 本 skill **不新建、不修改**任何 openspec 产出物（proposal / design / spec / tasks / 变更文件）。拆解结论只在对话上下文生效。
- 产出物必须仍由 openspec（`/opsx-*` 工作流）原样生成，本 skill 只提升输入质量。
- **全部产出物必须使用中文书写**（代码与接口名等强制技术标识符除外），包括 proposal / design / spec / tasks 的正文、场景、验收标准，禁止混入英文叙述。
- 不把方法论结论写入项目文档（本文档及 references 除外）。
- **不做浏览器验证 / 端到端实测 / 测试用例生成**——那属测试 skill（如 `glodrapay-ai-testing-qc`）的职责。本 skill 只把验收标准写成「可转断言」的外部可观察行为，不实际执行验证、不在产出物中夹带浏览器实测动作。

***

## 目录结构

```
.agents/skills/openspec-spec-methodology/
├── SKILL.md                    ← 本文件：skill 定义与执行入口
└── references/
    ├── triggers.md             ← 自动触发清单（命令变体 / 开篇标题 / 排除清单）
    ├── methodology.md          ← 三支柱方法论文档（拆解/颗粒度/边界）
    └── checklist.md            ← 增强预检清单（A 拆解 / B 颗粒度 / C 边界 / D 配合）
```

## 效果对照（内部参考）

- **范本**：历史高难度 change（未归档 `add-merchant-detail-page`、已归档 `2026-07-07-voucher-management`）的 spec 多样性已体现三支柱，可作为产出质量的参照基线。
- **反例**：`add-payment-audit`（相对朴素）。验证时可用这两个 change 各跑一遍 propose 对比链路质量。