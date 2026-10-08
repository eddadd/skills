# OpenSpec 生成类命令触发清单

本文件是 `openspec-spec-methodology` skill 的**自动触发依据**。当对话中出现下列任一信号时，本 skill 自动对当前需求做增强预检，无需用户手动调用。

> 维护说明：openspec 若升级命令、改标题、增删命令，只改本清单即可，SKILL.md 主流程不动。新增的生成类命令应在此登记。

---

## 一、生成产出物类命令（触发本 skill）

以下命令的意图是**生成变更产出物**（proposal / spec / design / tasks），命中即触发增强预检。

### 1. propose —— 提议新变更并一步生成所有产出物

- **命令变体**：`/opsx-propose`、`/opsx:propose`、`/opsx propose`、`openspec propose`
- **命令开篇标题**：「提议新变更 - 创建变更并一步生成所有产出物」

### 2. new —— 启动新变更（实验性产出物工作流）

- **命令变体**：`/opsx-new`、`/opsx:new`、`/opsx new change`、`openspec new`、`openspec new change`
- **命令开篇标题**：「使用实验性产出物驱动方法启动新变更」

### 3. ff —— 快速推进产出物创建

- **命令变体**：`/opsx-ff`、`/opsx:ff`、`/opsx ff`、`/opsx-ff-change`、`openspec ff`
- **命令开篇标题**：「快速推进产出物创建 - 一次性生成开始实现所需的所有内容」

### 4. continue —— 继续创建下一个产出物

- **命令变体**：`/opsx-continue`、`/opsx:continue`、`/opsx continue`、`openspec continue`
- **命令开篇标题**：「通过创建下一个产出物来继续处理变更」

---

## 二、口语/意图表述（同样触发）

以下表述表达「生成变更产出物」意图，即便不含命令名也应触发：

- 「生成提案」「出一个提案」
- 「创建变更」「发一个变更」「新建变更」
- 「用 openspec 让 AI 帮我写 spec / 设计 / 任务」
- 「开始一个新变更」「为 X 出一份 spec + 设计 + 任务」

---

## 三、不触发本 skill 的命令（仅归档/验证/同步/探索/教学）

以下命令**不生成**变更产出物，命中本 skill **不触发**：

| 命令 | 意图 |
| --- | --- |
| `/opsx-apply` / `openspec apply` | 从变更中实现任务（代码阶段，非产出物） |
| `/opsx-verify` / `openspec verify` | 验证实现是否匹配产出物 |
| `/opsx-archive` / `openspec archive` | 归档已完成的变更 |
| `/opsx-bulk-archive` / `openspec bulk-archive` | 批量归档变更 |
| `/opsx-sync` / `openspec sync` | 增量 spec 同步到主 spec |
| `/opsx-explore` / `openspec explore` | 探索模式（变更前思考，不产出物） |
| `/opsx-onboard` / `openspec onboard` | 引导式教学入门 |

---

## 四、判定与排除

- **触发判定**：命中「一」或「二」任一信号，且用户意图是生成变更产出物 → 触发本 skill 增强预检。
- **排除**：命中「三」的命令 → 不触发；对话虽有「openspec/opsx/提案/spec」关键词，但意图明显是归档/验证/同步/探索 → 不触发。
- **粒度判断**：命中触发信号，但需求为单文件、无交叉约束的小改动 → skill 判断后不介入（跳过增强预检直接跑 openspec）；中等以上复杂度需求 → 完整执行增强预检。