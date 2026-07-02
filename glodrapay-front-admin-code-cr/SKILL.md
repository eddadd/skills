---
name: glodrapay-front-admin-code-cr
description: 执行代码审查（Code Review）。按 6 步流程在 PR/Commit/当前更改上运行 CR，引用本 skill 内置 `coding-standards.md` 的规则，生成 `.md` + `.html` 报告并归档到项目根 `code-cr/`。支持 OpenSpec 集成和历史提醒。
license: MIT
compatibility: 需要项目根 `code-cr/` 输出目录存在（首次运行自动创建）。
metadata:
  author: syf
  version: "2.0"
---

# code-cr

代码审查 Skill。按 6 步流程对 PR/Commit/当前更改进行审查，引用本 skill 目录下的 `coding-standards.md` 规则 ID，生成 `.md` + `.html` 双格式报告归档到项目根 `code-cr/`，并更新历史索引和触发记录。

> 本 skill 所有资源自包含：规范库、流程文档、报告模板均在本 skill 目录内。
> 项目根 `code-cr/` 只存放运行时生成的报告和历史数据。

***

## 前导

### 环境

- 项目根：`.`（Git 仓库根目录，`pnpm` 所在位置）
- 运行时数据路径：`code-cr/`（项目根）
- 包管理：`pnpm`（遵循 `AGENTS.md` 声明）
- 质量命令：`pnpm lint` / `pnpm typecheck` / `pnpm lint:stylelint`

### 命名约定

| 项目    | 格式                              |
| ----- | ------------------------------- |
| CR 编号 | `CR-YYYYMMDD-NNN`               |
| 规则 ID | `CR-RULE-XXX`（3 位数字）            |
| 等级    | `S`（严重）/ `M`（中等）/ `R`（建议）       |
| 报告目录  | `code-cr/YYYYMMDD/`             |
| 历史文件  | `code-cr/.trigger-history.json`（违规+优化统一记录） |

### 目录结构

```
.xxx/skills/code-cr/
├── SKILL.md                   ← 本文件：skill 定义与执行入口
├── coding-standards.md        ← 62 条规则规范库
├── cr-process.md              ← 完整 CR 流程文档
├── template/
│   ├── template.md            ← CR 报告模板（Markdown）
│   └── template.html          ← CR 报告模板（HTML）

code-cr/                        ← 运行时数据（项目根）
├── index.md                   ← 报告目录索引
├── .trigger-history.json      ← 违规+优化触发记录
└── YYYYMMDD/
    ├── CR-YYYYMMDD-NNN.md
    └── CR-YYYYMMDD-NNN.html
```

***

## 触发条件

本 skill 在以下场景被调用：

- 用户明确请求执行 CR（"跑一下 CR"、"代码审查"、“代码CR”）
- 用户提交 PR 后要求审查
- 用户提问"帮我看看这段代码"

***

## 输入信息

运行 CR 前，通过 **AskUserQuestion** 收集以下信息（不完整时逐项追问）：

### 必填项

| 字段      | 说明                        | 缺省   |
| ------- | ------------------------- | ---- |
| 审查类型    | `pr`、`commit` 或 `current` | `pr` |
| 分支/提交信息 | 分支名、commit hash 或当前工作区    | —    |
| 提交者     | @用户名、@yun                     | @yun    |
| 审查者     | @用户名                      | @AI    |
| 变更摘要    | 一句话描述变更内容                 | —    |

### 可选项

| 字段          | 说明                |
| ----------- | ----------------- |
| PR 链接       | 可选（current 类型可省略） |
| OpenSpec 关联 | 关联的 change 名称，可选  |

### AskUserQuestion 格式

```markdown
<!-- 使用 AskUserQuestion 时遵循以下规则 -->
- 每次只问 1 个问题，不要堆叠
- 给默认值/推荐值（如审查类型默认 `pr`，可选 `commit` / `current`，提交者默认@yun，可选@用户名或其他）
- 选项不超过 5 个
- 释义清晰，不要求用户懂内部术语
```

***

## 写作风格

- 全部用中文回复，生成的文件内容也必须是中文。
- 每次回答前固定加称呼：`老板`。
- 简洁、直接、不铺垫。不写"这是你的报告"、"请查收"等废话。
- 报告内容使用客观、技术化的语气，避免主观评价。
- 代码示例使用正确的语法高亮（\`\`\`ts / \`\`\`vue / \`\`\`bash）。
- Markdown 表格对齐，编号连续。

***

## 混淆协议（Confusion Protocol）

当以下情况发生时，必须通过 AskUserQuestion 向用户澄清，不猜测：

1. **目标不明确**：不知道要审什么范围（整个 PR？单个文件？）
2. **选择不明确**：多个分支/变更同时存在
3. **缺少关键输入**：不知道提交者或审查者是谁
4. **先决条件未满足**：前置检查失败，不确定是否要继续

**禁止**：

- 猜测用户意图
- 假设文件路径或变更范围
- 自行决定审查粒度

***

## 完整性原则

- 不跳步：严格按 Step 0 → Step 6 顺序执行
- 不遗漏：报告必须包含所有步骤的输出
- 双格式：.md + .html 缺一不可
- 历史更新：每次 CR 必须更新 `.trigger-history.json`
- 编号不重复：从 `index.md` 读取最新序号

***

## 上下文恢复（Checkpoint）

当审查过程被打断或被要求保存状态时：

1. **保存检查点**：在 `code-cr/.checkpoint.json` 写入当前进度
   ```json
   {
     "stage": "step-3",
     "crId": "CR-20260610-001",
     "submitter": "@xxx",
     "branch": "feat/xxx",
     "findings": [...],
     "remainingRules": ["CR-RULE-005", "CR-RULE-010"],
     "updatedAt": "2026-06-10T14:30:00Z"
   }
   ```
2. **恢复检查点**：读取 `.checkpoint.json`，从保存的阶段继续
3. **重新验证**：恢复后重新运行前置检查（以防代码变更）

***

## 完成状态协议

CR 执行完成后，输出以下状态摘要：

```
CR 编号：CR-20260610-001
状态：   待结论（✅ 批准 / 🔄 整改 / ❌ 驳回）

报告路径：
  code-cr/20260610/CR-20260610-001.md
  code-cr/20260610/CR-20260610-001.html

触发规则：N 项（S:X, M:Y, R:Z）
遗留提醒：N 项未解决

OpenSpec 关联：change-name（场景覆盖：3/5 ✅）
```

- `pr` / `commit` 类型结论为"批准通过"，提示可合并 PR
- `current` 类型结论为"批准通过"，提示可提交变更
- 任何类型结论为"需整改"，提示修改后重审

***

## 设置与验证（Setup）

首次使用或报告异常时执行以下验证：

```bash
# 验证 skill 目录完整性，xxx 为当前所使用的AI工具名称
Test-Path -LiteralPath ".xxx/skills/code-cr/coding-standards.md"
Test-Path -LiteralPath ".xxx/skills/code-cr/cr-process.md"
Test-Path -LiteralPath ".xxx/skills/code-cr/template/template.md"
Test-Path -LiteralPath ".xxx/skills/code-cr/template/template.html"

# 验证运行时数据目录
Test-Path -LiteralPath "code-cr/"  # 不存在则 New-Item -ItemType Directory

# 验证历史文件
Test-Path -LiteralPath "code-cr/.trigger-history.json"  # 不存在则初始化为 { "submitter": [], "stats": {} }
```

***

## 审查步骤

***

### Step 0：前置准备

1. 确认项目根 `code-cr/` 目录存在（不存在则自动创建）。
2. 确认本 skill 目录下 `coding-standards.md` 存在。
3. **检测是否重审**：检查用户输入中是否包含 `重审：CR-YYYYMMDD-NNN` 或 `Re-review：CR-YYYYMMDD-NNN`
4. **生成 CR 编号**：
   - 首次审查：`CR-YYYYMMDD-NNN`，读取 `code-cr/index.md` 获取当天最新序号，+1
   - 重审：`CR-YYYYMMDD-NNN-R{次数}`，原编号 + 后缀
5. 确认提交者和审查者已填写（通过 AskUserQuestion 收集）。

***

### Step 1：前置检查

运行项目的质量检查命令，确认代码通过基础门禁：

```bash
pnpm lint
pnpm typecheck
pnpm lint:stylelint
```

**处理**：

- 全部通过 → 继续
- 任一失败 → 阻塞，通知提交者修复后再审。在报告中记录失败项。

***

### Step 2：上下文感知

读取项目上下文文件，了解变更背景：

1. **读取** **`package.json`** → 确认技术栈和依赖无异常
2. **读取** **`README.md`** → 了解项目背景
3. **检查构建配置**（`vite.config.ts`） → 确认代理/环境配置未误改
4. **如果是 OpenSpec 关联变更**：
   - 读取 `openspec/changes/<name>/proposal.md` → 确认变更范围
   - 读取 `openspec/changes/<name>/design.md` → 确认架构决策
   - 读取 `openspec/changes/<name>/tasks.md` → 确认任务完成度
   - 读取 `openspec/changes/<name>/specs/*/spec.md` → 准备场景覆盖检查

***

### Step 3：规则触发检查

对照本 skill 目录下的 `coding-standards.md` 规则逐项检查变更代码：

#### 🔴 违规规则项（明显违反规则）

逐文件检查代码，发现违反规则时记录：

| 规则 ID | 等级 | 描述 | 文件 | 行号 |
| ----- | -- | -- | -- | -- |

优先检查可脚本化的规则：

- CR-RULE-002（`any`）：搜索 `ref<any>`、`as any`、`reactive<any>`
- CR-RULE-005（魔法值）：检查硬编码数字/字符串
- CR-RULE-902（未使用导入）：检查 ESLint 输出

#### 🟠 整改优化项

对不一定构成直接违规、但会明显增加维护成本和后续缺陷概率的问题单独记录：

| 优化项 ID | 优先级 | 描述 | 文件 | 行号 | 说明 |
| -------- | ------ | ---- | ---- | ---- | ---- |

优先检查以下整改优化项：

- CR-OPT-001：异步流程统一 loading / disabled / 防重复提交
- CR-OPT-002：表单状态初始化、回显、重置逻辑分离
- CR-OPT-003：生命周期只做调度，不堆业务逻辑
- CR-OPT-004：条件渲染和权限控制集中收敛
- CR-OPT-005：重复模板/样式/枚举映射需抽取复用
- CR-OPT-101：模板复杂表达式和大段 slot 提升可读性
- CR-OPT-102：样式作用域、层级和无用样式清理
- CR-OPT-103：国际化 key 结构统一，避免重复词条
- CR-OPT-104：提升关键逻辑可测试性

#### ❓ 需人工审查项

AI 确认不了是否需要整改的问题，整理出来留给人工拍板。记录格式如下：

| 序号 | 描述 | 文件 | 行号 | 不确定原因 |
|------|------|------|------|------------|
| 1    | xxx 逻辑是否合理 | src/views/XXX.vue | 80 | 缺少业务上下文 |
| 2    | yyy 是否会造成性能问题 | src/components/YYY.vue | 120 | 无法判断调用频率 |

> 🔑 **核心判断标准**：只有在你无法确定"该不该整改"时才放入此栏。如果只是不确定违规等级（S/M/R）但确认有问题，仍放入"违规规则项"。

#### 🟢 场景覆盖检查（OpenSpec 关联时）

逐条读取 `specs/*/spec.md` 中的 Gherkin 场景，确认实现已覆盖：

| 场景          | 状态    |
| ----------- | ----- |
| VA 列表展示表头   | ✅ 已覆盖 |
| VA 申请表单字段联动 | ✅ 已覆盖 |
| ...         | ❌ 未覆盖 |

***

### Step 4：历史重复触发检查

1. 读取 `code-cr/.trigger-history.json`
2. 按当前提交者查询 `stats[submitter]` 中每条规则的 `totalCount`
3. 如果 `totalCount >= 3`，在报告中插入警告：
   - `type: "rule"`：`⚠️ 提交者 @xxx 已连续触发 CR-RULE-{id} 共 {N} 次，请注意此项问题。`
   - `type: "optimization"`：`⚠️ 提交者 @xxx 已连续触发 CR-OPT-{id} 共 {N} 次，请优先处理历史优化项。`

***

### Step 5：生成报告

1. **读取模板**（本 skill 目录下）
   - **`code-cr`**`/template/template.md`
   - **`code-cr`**`/template/template.html`
2. **替换占位符**，填入所有收集到的信息：
   - CR 编号、提交者、审查者、日期、分支
   - OpenSpec 关联信息
   - 前置检查结果
    - 违规规则项列表
    - 整改优化项列表
    - 需人工审查项列表
    - 场景覆盖状态
   - 历史重复触发提醒
   - 结论
3. **写入报告文件**
   ```bash
   code-cr/{YYYYMMDD}/CR-{YYYYMMDD}-{NNN}.md
   code-cr/{YYYYMMDD}/CR-{YYYYMMDD}-{NNN}.html
   ```
4. **更新** **`code-cr/index.md`**：在对应日期下追加一行记录
5. **更新** **`code-cr/.trigger-history.json`**，每条 issue 记录新增 `type` 字段区分违规规则项和整改优化项：
   ```json
   {
     "version": 1,
     "records": [
       {
         "crId": "CR-20260610-003",
         "submitter": "@xxx",
         "date": "2026-06-11",
         "branch": "feature/xxx",
         "issues": [
           { "ruleId": "CR-RULE-002", "type": "rule",
             "desc": "src/views/XXX:45 ref<any>", "resolved": false },
           { "ruleId": "CR-OPT-003", "type": "optimization",
             "desc": "src/views/YYY:80 onMounted 堆业务逻辑", "resolved": false }
         ],
         "conclusion": "✅ 批准通过"
       }
     ],
     "stats": {}
   }
   ```
   并更新 `stats[submitter][ruleId]` 计数（违规规则项和整改优化项均计入）。

***

### Step 6：结论 & 闭环

生成结论部分，包含三个选项（勾选其一）：

```markdown
- [ ] ✅ **批准通过** — 可合并
- [ ] 🔄 **需整改后重审** — 请在 {DEADLINE} 前完成修改
- [ ] ❌ **驳回** — 原因：{REJECT_REASON}
```

附审查意见，说明每个触发项的严重程度和修复建议。

---

## 重审机制（Re-review）

当结论为 🔄 **需整改后重审** 时：

### 提交者操作

1. 根据 CR 报告修复所有 S 级阻塞问题
2. 运行 `pnpm lint && pnpm typecheck && pnpm lint:stylelint` 本地验证
3. 再次调用本 skill，输入相同的审查类型/范围，备注中注明`重审：CR-YYYYMMDD-NNN`

### Skill 处理

1. 检测用户输入中包含"重审：CR-YYYYMMDD-NNN"时，自动进入重审模式
2. 重审编号：在原编号后加 `-R1`（第二次重审为 `-R2`，以此类推）
3. 优先复查原报告中触发的所有规则 ID
4. 重审报告与原始报告存放在同一日期目录下
5. 原报告结论归档不变，重审报告反映最终状态

***

## 护栏

- 不自动修改代码，只输出 CR 报告
- 不直接 Merge PR，只给出结论
- 不跳过任何步骤
- 前置检查未通过时阻塞，不进入后续步骤
- 必须先读取本 skill 目录下的 `coding-standards.md` 确认最新规则
- 必须双格式输出（.md + .html）
- 必须更新 `.trigger-history.json`
- CR 编号不允许重复
- 如果 OpenSpec 关联但找不到变更目录，给出警告继续审查
- 混淆时提问，不猜测
