# CR 流程规范

> 本规范定义了从提交 PR/Commit/当前更改到 CR 审查完成的完整流程、角色职责和产出物要求。

---

## 一、触发条件

- **提交 Pull Request** 时触发 CR
- **直接 push 到主干分支**（不推荐）也应执行 CR
- **当前工作区未提交更改**，审查后再提交

---

## 二、流程总览

```
提交 PR / Commit / 当前更改
      │
      ▼
┌─────────────────────────────────────┐
│ Step 1: 前置检查                     │
│                                     │
│  pnpm lint          → 代码规范检查   │
│  pnpm typecheck     → 类型安全检查   │
│  pnpm lint:stylelint → 样式规范检查  │
│                                     │
│  结果：全过 → 继续                   │
│        有失败 → 阻塞，通知提交者修复  │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│ Step 2: 上下文感知                   │
│                                     │
│  ├─ 读取 package.json               │
│  │   → 确认技术栈与依赖无异常        │
│  ├─ 读取 README.md                  │
│  │   → 了解项目背景与启动方式        │
│  ├─ 检查构建配置 (vite.config.ts)    │
│  │   → 确认代理、环境变量等未误改    │
│  └─ 读取 OpenSpec                   │
│      → 关联哪个 change（如有）       │
│      → proposal.md → 变更范围       │
│      → design.md → 架构决策         │
│      → tasks.md → 任务完成度        │
│      → specs/*/spec.md → 场景覆盖  │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│ Step 3: 规则触发检查                 │
│                                     │
│  🔴 违规规则项                      │
│  明显违反某条 CR-RULE-XXX           │
│  输出：[规则ID] + [文件:行号]       │
│                                     │
│  🟠 整改优化项                      │
│  维护性 / 可读性 / 状态流问题        │
│  输出：[优化项ID] + [文件:行号]      │
│                                     │
│  ❓ 需人工审查项                     │
│  AI 确认不了是否需要整改的问题       │
│  输出：不确定原因描述                │
│                                     │
│  🟢 场景覆盖（OpenSpec 集成）        │
│  对照 spec.md 的 Gherkin 场景       │
│  逐条确认实现是否覆盖               │
│  输出：已覆盖 / 未覆盖 / 部分覆盖    │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│ Step 4: 历史重复触发提醒                 │
│                                     │
│  查 .trigger-history.json           │
│  → 按提交者索引                     │
│  → 列出该提交者所有未解决项         │
│                                     │
│  ⚠️ 同提交人触犯同规则 ≥3 次        │
│  报告中插入红色提醒：               │
│  「提交者 @xxx 已连续 3 次触发      │
│   CR-RULE-002，请注意此项问题」              │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│ Step 5: 生成报告                     │
│                                     │
│  填充 template → CR-YYYYMMDD-NNN.md │
│                → CR-YYYYMMDD-NNN.html│
│                                     │
│  写入 code-cr/YYYYMMDD/             │
│  更新 code-cr/index.md              │
│  更新 code-cr/.trigger-history.json │
└─────────────────┬───────────────────┘
                  │
                  ▼
┌─────────────────────────────────────┐
│ Step 6: 结论 & 闭环                 │
│                                     │
│  ✅ 批准  → Merge                   │
│  🔄 整改  → 修改后重审              │
│  ❌ 驳回  → 关闭 / 打回             │
└─────────────────────────────────────┘
```

---

## 三、重审机制（Re-review）

### 3.1 触发方式

CR 结论为 🔄 **需整改后重审** 时，提交者修改完毕后：

1. **本地验证**：运行 `pnpm lint && pnpm typecheck && pnpm lint:stylelint`，确认问题已修复
2. **再次调用 skill**：再次执行 `code-cr` skill，输入相同审查类型和范围，并在备注中注明"重审：CR-YYYYMMDD-NNN"

### 3.2 重审编号

重审报告沿用原 CR 编号，添加后缀 `-R{次数}`：

| 轮次 | 编号示例 | 说明 |
|------|----------|------|
| 首次审查 | `CR-20260610-001` | 原报告 |
| 第一次重审 | `CR-20260610-001-R1` | 重审报告，与原报告放在同日期目录下 |
| 第二次重审 | `CR-20260610-001-R2` | 以此类推 |

### 3.3 重审流程

skill 检测到是重审时，自动调整审查范围：

1. **前置检查**（完全重跑）：`pnpm lint` / `pnpm typecheck` / `pnpm lint:stylelint`
2. **规则检查**（增量检查）：优先复查原报告中触发的违规规则项 ID 和整改优化项 ID，确认全部已修复
3. **新触发规则**：仍可能发现原先未覆盖的新问题
4. **历史提醒**：正常读取 `.trigger-history.json`，但本次重审中新触发的不再重复警告（防同一次审查多次报警）

### 3.4 结论更新

- 原报告结论保持不变（记录原始审查结果，历史存档）
- 重审报告结论为最终状态：✅ 批准 / 🔄 仍需整改（进入下一轮重审） / ❌ 驳回

---

## 四、角色职责

### 提交者（Author）

1. 提交前确保 `pnpm lint` / `pnpm typecheck` / `pnpm lint:stylelint` 全通过
2. PR 描述（或变更说明）清晰说明变更内容、原因和影响范围
3. 如关联 OpenSpec 变更，在 PR 描述中标注 change 名称
4. 收到 CR 意见后及时回复和修改

### 审查者（Reviewer）

1. 按上述流程逐步骤执行
2. 使用 `./coding-standards.md` 中的规则 ID 标注问题
3. 给出 actionable（可操作）的修改意见
4. 在 CR 报告中区分填写“违规规则项”和“整改优化项”
5. 在 CR 报告中填写审查结论

### 合并责任人（Merge Owner）

1. 确认所有 S 级（Blocking）问题已修复
2. 确认 CR 报告已归档到 `code-cr/`
3. 执行 Merge 操作

---

## 五、CR 报告编号规则

```
CR-YYYYMMDD-NNN

YYYYMMDD = 审查日期（如 20260610）
NNN      = 当日序号（001、002、003...）
```

---

## 六、历史追踪

### 6.1 触发记录

每次 CR 完成后，将触发的违规规则项（`type: "rule"`）和整改优化项（`type: "optimization"`）统一记录到 `.trigger-history.json`：

```json
{
  "version": 1,
  "records": [
    {
      "crId": "CR-20260610-003",
      "submitter": "@alice",
      "date": "2026-06-11",
      "branch": "feature/xxx",
      "issues": [
        {
          "ruleId": "CR-RULE-002",
          "type": "rule",
          "desc": "src/views/XXX:45 ref<any>",
          "resolved": false
        },
        {
          "ruleId": "CR-OPT-003",
          "type": "optimization",
          "desc": "src/views/YYY:80 onMounted 堆业务逻辑",
          "resolved": false
        }
      ],
      "conclusion": "✅ 批准通过"
    }
  ],
  "stats": {}
}
```

### 6.2 重复触发警告

当 `.trigger-history.json` 中同一提交者同一规则 `totalCount >= 3`，CR 报告自动插入警告（按 `type` 区分文案）：

- `type: "rule"`：`⚠️ 提交者 @xxx 已连续 3 次触发 CR-RULE-002，建议修复历史遗留问题后再提交新变更。`
- `type: "optimization"`：`⚠️ 提交者 @xxx 已连续 3 次触发 CR-OPT-003，建议优先处理历史优化项后再做新变更。`

---

## 七、与 OpenSpec 的集成

当 CR 关联了 OpenSpec 变更时：

1. 读取 `openspec/changes/<name>/proposal.md` → 确认变更范围
2. 读取 `openspec/changes/<name>/design.md` → 确认架构决策被遵守
3. 读取 `openspec/changes/<name>/tasks.md` → 确认任务完成度
4. 读取 `openspec/changes/<name>/specs/*/spec.md` → 逐条确认场景覆盖

CR 报告的"场景覆盖检查"部分记录：
- 已覆盖的场景数 / 总场景数
- 未覆盖的场景列表

---

## 八、目录结构

```
code-cr/
├── template/
│   ├── template.md          ← CR 报告模板（Markdown）
│   └── template.html        ← CR 报告模板（HTML）
├── YYYYMMDD/                ← 按日期分类
│   ├── CR-YYYYMMDD-NNN.md   ← CR 报告正文
│   └── CR-YYYYMMDD-NNN.html
├── YYYYMMDD/
│   └── ...
├── index.md                 ← 目录索引（按时间倒序）
└── .trigger-history.json    ← 违规+优化触发记录（按提交者索引）
```
